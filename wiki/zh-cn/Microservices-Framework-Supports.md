# 事务上下文

Fescar 的事务上下文由 RootContext 来管理。

应用开启一个全局事务后，RootContext 会自动绑定该事务的 XID，事务结束（提交或回滚完成），RootContext 会自动解绑 XID。

```java
// 绑定 XID
RootContext.bind(xid);

// 解绑 XID
String xid = RootContext.unbind();
```

应用可以通过 RootContext 的 API 接口来获取当前运行时的全局事务 XID。

```java
// 获取 XID
String xid = RootContext.getXID();
```
应用是否运行在一个全局事务的上下文中，就是通过 RootContext 是否绑定 XID 来判定的。

```java
    public static boolean inGlobalTransaction() {
        return CONTEXT_HOLDER.get(KEY_XID) != null;
    }
```

# 事务传播

Fescar 全局事务的传播机制就是指事务上下文的传播，根本上，就是 XID 的应用运行时的传播方式。

*1. 服务内部的事务传播*

默认的，RootContext 的实现是基于 *ThreadLocal* 的，即 XID 绑定在当前线程上下文中。

```java
public class ThreadLocalContextCore implements ContextCore {

    private ThreadLocal<Map<String, String>> threadLocal = new ThreadLocal<Map<String, String>>() {
        @Override
        protected Map<String, String> initialValue() {
            return new HashMap<String, String>();
        }

    };

    @Override
    public String put(String key, String value) {
        return threadLocal.get().put(key, value);
    }

    @Override
    public String get(String key) {
        return threadLocal.get().get(key);
    }

    @Override
    public String remove(String key) {
        return threadLocal.get().remove(key);
    }
}
```

所以服务内部的 XID 传播通常是天然的通过同一个线程的调用链路串连起来的。默认不做任何处理，事务的上下文就是传播下去的。

如果希望挂起事务上下文，则需要通过 RootContext 提供的 API 来实现：

```java
// 挂起（暂停）
String xid = RootContext.unbind();

// TODO: 运行在全局事务外的业务逻辑

// 恢复全局事务上下文
RootContext.bind(xid);

```

*2. 跨服务调用的事务传播*

通过上述基本原理，我们可以很容易理解：

> 跨服务调用场景下的事务传播，本质上就是要把 XID 通过服务调用传递到服务提供方，并绑定到 RootContext 中去。

只要能做到这点，理论上 Fescar 可以支持任意的微服务框架。

# 对 Dubbo 支持的解读

下面，我们通过内置的对 Dubbo RPC 支持机制的解读，来说明 Fescar 在实现对一个特定微服务框架支持的机制。

对 Dubbo 的支持，我们利用了 Dubbo 框架的 _org.apache.dubbo.rpc.Filter_ 机制。

```java
/**
 * The type Transaction propagation filter.
 */
@Activate(group = { Constants.PROVIDER, Constants.CONSUMER }, order = 100)
public class TransactionPropagationFilter implements Filter {

    private static final Logger LOGGER = LoggerFactory.getLogger(TransactionPropagationFilter.class);

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        String xid = RootContext.getXID(); // 获取当前事务 XID
        String rpcXid = RpcContext.getContext().getAttachment(RootContext.KEY_XID); // 获取 RPC 调用传递过来的 XID
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("xid in RootContext[" + xid + "] xid in RpcContext[" + rpcXid + "]");
        }
        boolean bind = false;
        if (xid != null) { // Consumer：把 XID 置入 RPC 的 attachment 中
            RpcContext.getContext().setAttachment(RootContext.KEY_XID, xid);
        } else {
            if (rpcXid != null) { // Provider：把 RPC 调用传递来的 XID 绑定到当前运行时
                RootContext.bind(rpcXid);
                bind = true;
                if (LOGGER.isDebugEnabled()) {
                    LOGGER.debug("bind[" + rpcXid + "] to RootContext");
                }
            }
        }
        try {
            return invoker.invoke(invocation); // 业务方法的调用

        } finally {
            if (bind) { // Provider：调用完成后，对 XID 的清理
                String unbindXid = RootContext.unbind();
                if (LOGGER.isDebugEnabled()) {
                    LOGGER.debug("unbind[" + unbindXid + "] from RootContext");
                }
                if (!rpcXid.equalsIgnoreCase(unbindXid)) {
                    LOGGER.warn("xid in change during RPC from " + rpcXid + " to " + unbindXid);
                    if (unbindXid != null) { // 调用过程有新的事务上下文开启，则不能清除
                        RootContext.bind(unbindXid);
                        LOGGER.warn("bind [" + unbindXid + "] back to RootContext");
                    }
                }
            }
        }
    }
}
```









