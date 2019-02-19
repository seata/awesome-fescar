[//]: #

(title: API Guide)

[//]: #

(author: sharajava, mingisme)

[//]: #

(keywords: API Guide)

[//]: #

(date: 02/18/2019)

# Transaction Context

Transaction context of Fescar is managed by RootContext.

When application begins a global transaction, RootContext will bind the XID of the transaction automatically, at the end of transaction(commit or rollback), RootContext will unbind the XID automatically.

```java
// Bind XID
RootContext.bind(xid);

// Unbind XID
String xid = RootContext.unbind();
```

Application retrieve the global transaction XID through the API of RootContext.

```java
// Retrieve XID
String xid = RootContext.getXID();
```
Whether application is running a global transaction, just check if an XID bound to RootContext.

```java
    public static boolean inGlobalTransaction() {
        return CONTEXT_HOLDER.get(KEY_XID) != null;
    }
```

# Transaction propagation

The mechanism of the global transaction of Fescar is the propagation of transaction context,  primarily, it's the propagation way of XID in runtime.

*1. The propagation of transaction in the service*

By default, RootContext is based on ThreadLocal, which is the XID is bound in the context of thread.

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

So the inner XID of service is tracing by the same thread naturally, do nothing to propagate the transaction by default.

If it hopes to hung up the transaction context, implement it by the API of RootContext:

```java
// Hung up(pause)
String xid = RootContext.unbind();

// TODO: Logic running out of the global transaction scope

// recover the global transaction
RootContext.bind(xid);

```

*2. Transactional propagation across service calls*

It's easy to know by the basic idea preceding:

> The transaction propagation across service calls, essentially, propagate the XID via service call to service provider, and bind it to RootContext.

As long as it can be done, Fescar can support any microservice framework in theory.

# Interpretation of supporting Dubbo

Let's interpret the inner support for Dubbo RPC to illustrate how Fescar supports a specific microservice framework in follows:

We use the org.apache.dubbo.rpc.Filter of Dubbo to support propagation of transaction.

```java
/**
 * The type Transaction propagation filter.
 */
@Activate(group = { Constants.PROVIDER, Constants.CONSUMER }, order = 100)
public class TransactionPropagationFilter implements Filter {

    private static final Logger LOGGER = LoggerFactory.getLogger(TransactionPropagationFilter.class);

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        String xid = RootContext.getXID(); // Get XID of current transaction
        String rpcXid = RpcContext.getContext().getAttachment(RootContext.KEY_XID); // Acquire the XID from RPC invoke
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("xid in RootContext[" + xid + "] xid in RpcContext[" + rpcXid + "]");
        }
        boolean bind = false;
        if (xid != null) { // Consumer：Put XID into the attachment of RPC
            RpcContext.getContext().setAttachment(RootContext.KEY_XID, xid);
        } else {
            if (rpcXid != null) { // Provider：Bind the XID propagated by RPC to current runtime
                RootContext.bind(rpcXid);
                bind = true;
                if (LOGGER.isDebugEnabled()) {
                    LOGGER.debug("bind[" + rpcXid + "] to RootContext");
                }
            }
        }
        try {
            return invoker.invoke(invocation); // Business method invoke

        } finally {
            if (bind) { // Provider：Clean up XID after invoke
                String unbindXid = RootContext.unbind();
                if (LOGGER.isDebugEnabled()) {
                    LOGGER.debug("unbind[" + unbindXid + "] from RootContext");
                }
                if (!rpcXid.equalsIgnoreCase(unbindXid)) {
                    LOGGER.warn("xid in change during RPC from " + rpcXid + " to " + unbindXid);
                    if (unbindXid != null) { // if there is new transaction begin, can't do clean up
                        RootContext.bind(unbindXid);
                        LOGGER.warn("bind [" + unbindXid + "] back to RootContext");
                    }
                }
            }
        }
    }
}
```









