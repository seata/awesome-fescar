---
title: 分布式事务之Seata-Client原理及流程详解
author: fangliangsheng
date: 2019/04/15
keywords: fescar、seata、分布式事务
---

## 前言
在分布式系统中，分布式事务是一个必须要解决的问题，目前使用较多的是最终一致性方案。自年初阿里开源了Fescar（四月初更名为Seata）后，该项目受到了极大的关注度，目前已接近8000Star。[Seata](https://github.com/seata/seata)以高性能和零侵入的方式为目标解决微服务领域的分布式事务难题，目前正处于快速迭代中，近期小目标是生产可用的Mysql版本。关于Seata的总体介绍，可以查看[官方WIKI](https://github.com/seata/seata/wiki/%E6%A6%82%E8%A7%88)获得更多更全面的内容介绍。

本文主要基于spring cloud+spring jpa+spring cloud alibaba fescar+mysql+seata的结构，搭建一个分布式系统的demo，通过seata的debug日志和源代码，从client端（RM、TM）的角度分析说明其工作流程及原理。

文中代码基于fescar-0.4.1，由于项目刚更名为seata不久，例如一些包名、类名、jar包名称还都是fescar的命名，故下文中仍使用fescar进行表述。

示例项目：[https://github.com/fescar-group/fescar-samples/tree/master/springcloud-jpa-seata](https://github.com/fescar-group/fescar-samples/tree/master/springcloud-jpa-seata)
## 相关概念
- XID：全局事务的唯一标识，由ip:port:sequence组成
- Transaction Coordinator (TC)：事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚
- Transaction Manager (TM )：控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议
- Resource Manager (RM)：控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚
## 分布式框架支持
Fescar使用XID表示一个分布式事务，XID需要在一次分布式事务请求所涉的系统中进行传递，从而向feacar-server发送分支事务的处理情况，以及接收feacar-server的commit、rollback指令。
Fescar官方已支持全版本的dubbo协议，而对于spring cloud（spring-boot）的分布式项目社区也提供了相应的实现
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-fescar</artifactId>
    <version>2.1.0.BUILD-SNAPSHOT</version>
</dependency>
```
该组件实现了基于RestTemplate、Feign通信时的XID传递功能。
## 业务逻辑
业务逻辑是经典的下订单、扣余额、减库存流程。
根据模块划分为三个独立的服务，且分别连接对应的数据库

 - 订单：order-server
 - 账户：account-server
 - 库存：storage-server

另外还有发起分布式事务的业务系统

 - 业务：business-server
 
项目结构如下图
![在这里插入图片描述](https://github.com/fescar-group/awesome-fescar/blob/master/img/20190410114411366.png)
 
**正常业务**
   1. business发起购买请求
   2. storage扣减库存
   3. order创建订单
   4. account扣减余额
   
**异常业务**
   1. business发起购买请求
   2. storage扣减库存
   3. order创建订单
   4. account`扣减余额异常`
   
正常流程下2、3、4步的数据正常更新全局commit，异常流程下的数据则由于第4步的异常报错全局回滚。

## 配置文件
fescar的配置入口文件是[registry.conf](https://github.com/seata/seata/blob/develop/config/src/main/resources/registry.conf),查看代码[ConfigurationFactory](https://github.com/seata/seata/blob/develop/config/src/main/java/com/alibaba/fescar/config/ConfigurationFactory.java)得知目前还不能指定该配置文件，所以配置文件名称只能为registry.conf
```java
private static final String REGISTRY_CONF = "registry.conf";
public static final Configuration FILE_INSTANCE = new FileConfiguration(REGISTRY_CONF);
```
在`registry`中可以指定具体配置的形式，默认使用file类型，在file.conf中有3部分配置内容

 1. transport
     transport部分的配置对应[NettyServerConfig](https://github.com/seata/seata/blob/develop/core/src/main/java/com/alibaba/fescar/core/rpc/netty/NettyServerConfig.java)类，用于定义Netty相关的参数，TM、RM与fescar-server之间使用Netty进行通信
 2. service
	 ```js
	 service {
	  #vgroup->rgroup
	  vgroup_mapping.my_test_tx_group = "default"
	  #配置Client连接TC的地址
	  default.grouplist = "127.0.0.1:8091"
	  #degrade current not support
	  enableDegrade = false
	  #disable
	  是否启用seata的分布式事务
	  disableGlobalTransaction = false
	}
	 ```
 3. client
    ```js
	client {
	  #RM接收TC的commit通知后缓冲上限
	  async.commit.buffer.limit = 10000
	  lock {
	    retry.internal = 10
	    retry.times = 30
	  }
	}
    ```
## 数据源Proxy
除了前面的配置文件，fescar在AT模式下稍微有点代码量的地方就是对数据源的代理指定，且目前只能基于`DruidDataSource`的代理。
注：在最新发布的0.4.2版本中已支持任意数据源类型
```java
@Bean
@ConfigurationProperties(prefix = "spring.datasource")
public DruidDataSource druidDataSource() {
    DruidDataSource druidDataSource = new DruidDataSource();
    return druidDataSource;
}

@Primary
@Bean("dataSource")
public DataSourceProxy dataSource(DruidDataSource druidDataSource) {
    return new DataSourceProxy(druidDataSource);
}
```
使用`DataSourceProxy`的目的是为了引入`ConnectionProxy`,fescar无侵入的一方面就体现在`ConnectionProxy`的实现上，即分支事务加入全局事务的切入点是在本地事务的`commit`阶段，这样设计可以保证业务数据与`undo_log`是在一个本地事务中。

`undo_log`是需要在业务库上创建的一个表，fescar依赖该表记录每笔分支事务的状态及二阶段`rollback`的回放数据。不用担心该表的数据量过大形成单点问题，在全局事务`commit`的场景下事务对应的`undo_log`会异步删除。
```sql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```
## 启动Server
前往[https://github.com/seata/seata/releases](https://github.com/seata/seata/releases) 下载与Client版本对应的fescar-server,避免由于版本的不同导致的协议不一致问题
进入解压之后的 bin 目录，执行
```shell
./fescar-server.sh 8091 ../data
```
启动成功输出
```shell
2019-04-09 20:27:24.637 INFO [main]c.a.fescar.core.rpc.netty.AbstractRpcRemotingServer.start:152 -Server started ... 
```
## 启动Client
fescar的加载入口类位于[GlobalTransactionAutoConfiguration](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/blob/finchley/spring-cloud-alibaba-fescar/src/main/java/org/springframework/cloud/alibaba/fescar/GlobalTransactionAutoConfiguration.java)，对基于spring boot的项目能够自动加载，当然也可以通过其他方式示例化`GlobalTransactionScanner`
```java
@Configuration
@EnableConfigurationProperties({FescarProperties.class})
public class GlobalTransactionAutoConfiguration {
    private final ApplicationContext applicationContext;
    private final FescarProperties fescarProperties;

    public GlobalTransactionAutoConfiguration(ApplicationContext applicationContext, FescarProperties fescarProperties) {
        this.applicationContext = applicationContext;
        this.fescarProperties = fescarProperties;
    }

	/**
	* 示例化GlobalTransactionScanner
	* scanner为client初始化的发起类
	*/
    @Bean
    public GlobalTransactionScanner globalTransactionScanner() {
        String applicationName = this.applicationContext.getEnvironment().getProperty("spring.application.name");
        String txServiceGroup = this.fescarProperties.getTxServiceGroup();
        if (StringUtils.isEmpty(txServiceGroup)) {
            txServiceGroup = applicationName + "-fescar-service-group";
            this.fescarProperties.setTxServiceGroup(txServiceGroup);
        }
		
        return new GlobalTransactionScanner(applicationName, txServiceGroup);
    }
}
```
可以看到支持一个配置项FescarProperties，用于配置事务分组名称
```json
spring.cloud.alibaba.fescar.tx-service-group=my_test_tx_group
```
如果不指定服务组，则默认使用spring.application.name+ -fescar-service-group生成名称，所以不指定spring.application.name启动会报错
```java
@ConfigurationProperties("spring.cloud.alibaba.fescar")
public class FescarProperties {
    private String txServiceGroup;

    public FescarProperties() {
    }

    public String getTxServiceGroup() {
        return this.txServiceGroup;
    }

    public void setTxServiceGroup(String txServiceGroup) {
        this.txServiceGroup = txServiceGroup;
    }
}
```
获取applicationId和txServiceGroup后，创建[GlobalTransactionScanner](https://github.com/seata/seata/blob/develop/spring/src/main/java/com/alibaba/fescar/spring/annotation/GlobalTransactionScanner.java)对象，主要看类中initClient方法
```java
private void initClient() {
    if (StringUtils.isNullOrEmpty(applicationId) || StringUtils.isNullOrEmpty(txServiceGroup)) {
        throw new IllegalArgumentException(
            "applicationId: " + applicationId + ", txServiceGroup: " + txServiceGroup);
    }
    //init TM
    TMClient.init(applicationId, txServiceGroup);

    //init RM
    RMClient.init(applicationId, txServiceGroup);
  
}
```
方法中可以看到初始化了`TMClient`和`RMClient`，对于一个服务既可以是TM角色也可以是RM角色，至于什么时候是TM或者RM则要看在一次全局事务中`@GlobalTransactional`注解标注在哪。
Client创建的结果是与TC的一个Netty连接，所以在启动日志中可以看到两个Netty Channel，其中标明了transactionRole分别为`TMROLE`和`RMROLE`
```java
2019-04-09 13:42:57.417  INFO 93715 --- [imeoutChecker_1] c.a.f.c.rpc.netty.NettyPoolableFactory   : NettyPool create channel to {"address":"127.0.0.1:8091","message":{"applicationId":"business-service","byteBuffer":{"char":"\u0000","direct":false,"double":0.0,"float":0.0,"int":0,"long":0,"readOnly":false,"short":0},"transactionServiceGroup":"my_test_tx_group","typeCode":101,"version":"0.4.1"},"transactionRole":"TMROLE"}
2019-04-09 13:42:57.505  INFO 93715 --- [imeoutChecker_1] c.a.f.c.rpc.netty.NettyPoolableFactory   : NettyPool create channel to {"address":"127.0.0.1:8091","message":{"applicationId":"business-service","byteBuffer":{"char":"\u0000","direct":false,"double":0.0,"float":0.0,"int":0,"long":0,"readOnly":false,"short":0},"transactionServiceGroup":"my_test_tx_group","typeCode":103,"version":"0.4.1"},"transactionRole":"RMROLE"}
2019-04-09 13:42:57.629 DEBUG 93715 --- [lector_TMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:RegisterTMRequest{applicationId='business-service', transactionServiceGroup='my_test_tx_group'}
2019-04-09 13:42:57.629 DEBUG 93715 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:RegisterRMRequest{resourceIds='null', applicationId='business-service', transactionServiceGroup='my_test_tx_group'}
2019-04-09 13:42:57.699 DEBUG 93715 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null,messageId:1
2019-04-09 13:42:57.699 DEBUG 93715 --- [lector_TMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null,messageId:2
2019-04-09 13:42:57.701 DEBUG 93715 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : com.alibaba.fescar.core.rpc.netty.RmRpcClient@3b06d101 msgId:1, future :com.alibaba.fescar.core.protocol.MessageFuture@28bb1abd, body:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null
2019-04-09 13:42:57.701 DEBUG 93715 --- [lector_TMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : com.alibaba.fescar.core.rpc.netty.TmRpcClient@65fc3fb7 msgId:2, future :com.alibaba.fescar.core.protocol.MessageFuture@9a1e3df, body:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null
2019-04-09 13:42:57.710  INFO 93715 --- [imeoutChecker_1] c.a.fescar.core.rpc.netty.RmRpcClient    : register RM success. server version:0.4.1,channel:[id: 0xe6468995, L:/127.0.0.1:57397 - R:/127.0.0.1:8091]
2019-04-09 13:42:57.710  INFO 93715 --- [imeoutChecker_1] c.a.f.c.rpc.netty.NettyPoolableFactory   : register success, cost 114 ms, version:0.4.1,role:TMROLE,channel:[id: 0xd22fe0c5, L:/127.0.0.1:57398 - R:/127.0.0.1:8091]
2019-04-09 13:42:57.711  INFO 93715 --- [imeoutChecker_1] c.a.f.c.rpc.netty.NettyPoolableFactory   : register success, cost 125 ms, version:0.4.1,role:RMROLE,channel:[id: 0xe6468995, L:/127.0.0.1:57397 - R:/127.0.0.1:8091]

```
日志中可以看到
1. 创建Netty连接
2. 发送注册请求
3. 得到响应结果
4. `RmRpcClient`、`TmRpcClient`成功实例化
## TM处理流程
在本例中，TM的角色是business-service,BusinessService的purchase方法标注了`@GlobalTransactional`注解
```java
@Service
public class BusinessService {

    @Autowired
    private StorageFeignClient storageFeignClient;
    @Autowired
    private OrderFeignClient orderFeignClient;

    @GlobalTransactional
    public void purchase(String userId, String commodityCode, int orderCount){
        storageFeignClient.deduct(commodityCode, orderCount);

        orderFeignClient.create(userId, commodityCode, orderCount);
    }
}
```
方法调用后将会创建一个全局事务，首先关注`@GlobalTransactional`注解的作用，在[GlobalTransactionalInterceptor](https://github.com/seata/seata/blob/develop/spring/src/main/java/com/alibaba/fescar/spring/annotation/GlobalTransactionalInterceptor.java)中被拦截处理
```java
/**
 * AOP拦截方法调用
 */
@Override
public Object invoke(final MethodInvocation methodInvocation) throws Throwable {
    Class<?> targetClass = (methodInvocation.getThis() != null ? AopUtils.getTargetClass(methodInvocation.getThis()) : null);
    Method specificMethod = ClassUtils.getMostSpecificMethod(methodInvocation.getMethod(), targetClass);
    final Method method = BridgeMethodResolver.findBridgedMethod(specificMethod);

	//获取方法GlobalTransactional注解
    final GlobalTransactional globalTransactionalAnnotation = getAnnotation(method, GlobalTransactional.class);
    final GlobalLock globalLockAnnotation = getAnnotation(method, GlobalLock.class);
    
    //如果方法有GlobalTransactional注解，则拦截到相应方法处理
    if (globalTransactionalAnnotation != null) {
        return handleGlobalTransaction(methodInvocation, globalTransactionalAnnotation);
    } else if (globalLockAnnotation != null) {
        return handleGlobalLock(methodInvocation);
    } else {
        return methodInvocation.proceed();
    }
}
```
`handleGlobalTransaction`方法中对[TransactionalTemplate](https://github.com/seata/seata/blob/develop/tm/src/main/java/com/alibaba/fescar/tm/api/TransactionalTemplate.java)的execute进行了调用，从类名可以看到这是一个标准的模版方法，它定义了TM对全局事务处理的标准步骤，注释已经比较清楚了
```java
public Object execute(TransactionalExecutor business) throws TransactionalExecutor.ExecutionException {
    // 1. get or create a transaction
    GlobalTransaction tx = GlobalTransactionContext.getCurrentOrCreate();

    try {
        // 2. begin transaction
        try {
            triggerBeforeBegin();
            tx.begin(business.timeout(), business.name());
            triggerAfterBegin();
        } catch (TransactionException txe) {
            throw new TransactionalExecutor.ExecutionException(tx, txe,
                TransactionalExecutor.Code.BeginFailure);
        }
        Object rs = null;
        try {
            // Do Your Business
            rs = business.execute();
        } catch (Throwable ex) {
            // 3. any business exception, rollback.
            try {
                triggerBeforeRollback();
                tx.rollback();
                triggerAfterRollback();
                // 3.1 Successfully rolled back
                throw new TransactionalExecutor.ExecutionException(tx, TransactionalExecutor.Code.RollbackDone, ex);
            } catch (TransactionException txe) {
                // 3.2 Failed to rollback
                throw new TransactionalExecutor.ExecutionException(tx, txe,
                    TransactionalExecutor.Code.RollbackFailure, ex);
            }
        }
        // 4. everything is fine, commit.
        try {
            triggerBeforeCommit();
            tx.commit();
            triggerAfterCommit();
        } catch (TransactionException txe) {
            // 4.1 Failed to commit
            throw new TransactionalExecutor.ExecutionException(tx, txe,
                TransactionalExecutor.Code.CommitFailure);
        }
        return rs;
    } finally {
        //5. clear
        triggerAfterCompletion();
        cleanUp();
    }
}
```
通过[DefaultGlobalTransaction](https://github.com/seata/seata/blob/develop/tm/src/main/java/com/alibaba/fescar/tm/api/DefaultGlobalTransaction.java)的begin方法开启全局事务
```java
public void begin(int timeout, String name) throws TransactionException {
    if (role != GlobalTransactionRole.Launcher) {
        check();
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Ignore Begin(): just involved in global transaction [" + xid + "]");
        }
        return;
    }
    if (xid != null) {
        throw new IllegalStateException();
    }
    if (RootContext.getXID() != null) {
        throw new IllegalStateException();
    }
    //具体开启事务的方法，获取TC返回的XID
    xid = transactionManager.begin(null, null, name, timeout);
    status = GlobalStatus.Begin;
    RootContext.bind(xid);
    if (LOGGER.isDebugEnabled()) {
        LOGGER.debug("Begin a NEW global transaction [" + xid + "]");
    }
}
```
方法开头处`if (role != GlobalTransactionRole.Launcher)`对role的判断有关键的作用，表明当前是全局事务的发起者（Launcher）还是参与者（Participant）。如果在分布式事务的下游系统方法中也加上`@GlobalTransactional`注解，那么它的角色就是Participant，会忽略后面的begin直接return，而判断是Launcher还是Participant是根据当前上下文是否已存在XID来判断，没有XID的就是Launcher，已经存在XID的就是Participant.
由此可见，全局事务的创建只能由Launcher执行，而一次分布式事务中也只有一个Launcher存在。

[DefaultTransactionManager](https://github.com/seata/seata/blob/develop/tm/src/main/java/com/alibaba/fescar/tm/DefaultTransactionManager.java)负责TM与TC通讯，发送begin、commit、rollback指令
```java
@Override
public String begin(String applicationId, String transactionServiceGroup, String name, int timeout)
    throws TransactionException {
    GlobalBeginRequest request = new GlobalBeginRequest();
    request.setTransactionName(name);
    request.setTimeout(timeout);
    GlobalBeginResponse response = (GlobalBeginResponse)syncCall(request);
    return response.getXid();
}
```
至此拿到fescar-server返回的XID表示一个全局事务创建成功，日志中也反应了上述流程
```java
2019-04-09 13:46:57.417 DEBUG 31326 --- [nio-8084-exec-1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : offer message: timeout=60000,transactionName=purchase(java.lang.String,java.lang.String,int)
2019-04-09 13:46:57.417 DEBUG 31326 --- [geSend_TMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : write message:FescarMergeMessage timeout=60000,transactionName=purchase(java.lang.String,java.lang.String,int), channel:[id: 0xa148545e, L:/127.0.0.1:56120 - R:/127.0.0.1:8091],active?true,writable?true,isopen?true
2019-04-09 13:46:57.418 DEBUG 31326 --- [lector_TMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:FescarMergeMessage timeout=60000,transactionName=purchase(java.lang.String,java.lang.String,int)
2019-04-09 13:46:57.421 DEBUG 31326 --- [lector_TMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:MergeResultMessage com.alibaba.fescar.core.protocol.transaction.GlobalBeginResponse@2dc480dc,messageId:1196
2019-04-09 13:46:57.421 DEBUG 31326 --- [nio-8084-exec-1] c.a.fescar.core.context.RootContext      : bind 192.168.224.93:8091:2008502699
2019-04-09 13:46:57.421 DEBUG 31326 --- [nio-8084-exec-1] c.a.f.tm.api.DefaultGlobalTransaction    : Begin a NEW global transaction [192.168.224.93:8091:2008502699]
```
全局事务创建后，就开始执行business.execute()，即业务代码`storageFeignClient.deduct(commodityCode, orderCount)`进入RM处理流程，此处的业务逻辑为调用storage-service的扣减库存接口。
## RM处理流程
```java
@GetMapping(path = "/deduct")
public Boolean deduct(String commodityCode, Integer count){
    storageService.deduct(commodityCode,count);
    return true;
}

@Transactional
public void deduct(String commodityCode, int count){
    Storage storage = storageDAO.findByCommodityCode(commodityCode);
    storage.setCount(storage.getCount()-count);

    storageDAO.save(storage);
}
```
storage的接口和service方法并未出现fescar相关的代码和注解，体现了fescar的无侵入。那它是如何加入到这次全局事务中的呢？答案在[ConnectionProxy](https://github.com/seata/seata/blob/develop/rm-datasource/src/main/java/com/alibaba/fescar/rm/datasource/ConnectionProxy.java)中，这也是前面说为什么必须要使用`DataSourceProxy`的原因，通过DataSourceProxy才能在业务代码的本地事务提交时，fescar通过该切入点，向TC注册分支事务并发送RM的处理结果。

由于业务代码本身的事务提交被`ConnectionProxy`代理实现，所以在提交本地事务时，实际执行的是ConnectionProxy的commit方法
```java
public void commit() throws SQLException {
	//如果当前是全局事务，则执行全局事务的提交
	//判断是不是全局事务，就是看当前上下文是否存在XID
    if (context.inGlobalTransaction()) {
        processGlobalTransactionCommit();
    } else if (context.isGlobalLockRequire()) {
        processLocalCommitWithGlobalLocks();
    } else {
        targetConnection.commit();
    }
}
    
private void processGlobalTransactionCommit() throws SQLException {
    try {
    	//首先是向TC注册RM，拿到TC分配的branchId
        register();
    } catch (TransactionException e) {
        recognizeLockKeyConflictException(e);
    }

    try {
        if (context.hasUndoLog()) {
        	//写入undolog
            UndoLogManager.flushUndoLogs(this);
        }

		//提交本地事务，写入undo_log和业务数据在同一个本地事务中
        targetConnection.commit();
    } catch (Throwable ex) {
    	//向TC发送RM的事务处理失败的通知
        report(false);
        if (ex instanceof SQLException) {
            throw new SQLException(ex);
        }
    }
	//向TC发送RM的事务处理成功的通知
    report(true);
    context.reset();
}
    
private void register() throws TransactionException {
	//注册RM，构建request通过netty向TC发送注册指令
    Long branchId = DefaultResourceManager.get().branchRegister(BranchType.AT, getDataSourceProxy().getResourceId(),
            null, context.getXid(), null, context.buildLockKeys());
    //将返回的branchId存在上下文中
    context.setBranchId(branchId);
}
```
通过日志印证一下上面的流程
```java
2019-04-09 21:57:48.341 DEBUG 38933 --- [nio-8081-exec-1] o.s.c.a.f.web.FescarHandlerInterceptor   : xid in RootContext null xid in RpcContext 192.168.0.2:8091:2008546211
2019-04-09 21:57:48.341 DEBUG 38933 --- [nio-8081-exec-1] c.a.fescar.core.context.RootContext      : bind 192.168.0.2:8091:2008546211
2019-04-09 21:57:48.341 DEBUG 38933 --- [nio-8081-exec-1] o.s.c.a.f.web.FescarHandlerInterceptor   : bind 192.168.0.2:8091:2008546211 to RootContext
2019-04-09 21:57:48.386  INFO 38933 --- [nio-8081-exec-1] o.h.h.i.QueryTranslatorFactoryInitiator  : HHH000397: Using ASTQueryTranslatorFactory
Hibernate: select storage0_.id as id1_0_, storage0_.commodity_code as commodit2_0_, storage0_.count as count3_0_ from storage_tbl storage0_ where storage0_.commodity_code=?
Hibernate: update storage_tbl set count=? where id=?
2019-04-09 21:57:48.673  INFO 38933 --- [nio-8081-exec-1] c.a.fescar.core.rpc.netty.RmRpcClient    : will connect to 192.168.0.2:8091
2019-04-09 21:57:48.673  INFO 38933 --- [nio-8081-exec-1] c.a.fescar.core.rpc.netty.RmRpcClient    : RM will register :jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false
2019-04-09 21:57:48.673  INFO 38933 --- [nio-8081-exec-1] c.a.f.c.rpc.netty.NettyPoolableFactory   : NettyPool create channel to {"address":"192.168.0.2:8091","message":{"applicationId":"storage-service","byteBuffer":{"char":"\u0000","direct":false,"double":0.0,"float":0.0,"int":0,"long":0,"readOnly":false,"short":0},"resourceIds":"jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false","transactionServiceGroup":"hello-service-fescar-service-group","typeCode":103,"version":"0.4.0"},"transactionRole":"RMROLE"}
2019-04-09 21:57:48.677 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:RegisterRMRequest{resourceIds='jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false', applicationId='storage-service', transactionServiceGroup='hello-service-fescar-service-group'}
2019-04-09 21:57:48.680 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null,messageId:9
2019-04-09 21:57:48.680 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : com.alibaba.fescar.core.rpc.netty.RmRpcClient@7d61f5d4 msgId:9, future :com.alibaba.fescar.core.protocol.MessageFuture@186cd3e0, body:version=0.4.1,extraData=null,identified=true,resultCode=null,msg=null
2019-04-09 21:57:48.680  INFO 38933 --- [nio-8081-exec-1] c.a.fescar.core.rpc.netty.RmRpcClient    : register RM success. server version:0.4.1,channel:[id: 0xd40718e3, L:/192.168.0.2:62607 - R:/192.168.0.2:8091]
2019-04-09 21:57:48.680  INFO 38933 --- [nio-8081-exec-1] c.a.f.c.rpc.netty.NettyPoolableFactory   : register success, cost 3 ms, version:0.4.1,role:RMROLE,channel:[id: 0xd40718e3, L:/192.168.0.2:62607 - R:/192.168.0.2:8091]
2019-04-09 21:57:48.680 DEBUG 38933 --- [nio-8081-exec-1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : offer message: transactionId=2008546211,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,lockKey=storage_tbl:1
2019-04-09 21:57:48.681 DEBUG 38933 --- [geSend_RMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : write message:FescarMergeMessage transactionId=2008546211,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,lockKey=storage_tbl:1, channel:[id: 0xd40718e3, L:/192.168.0.2:62607 - R:/192.168.0.2:8091],active?true,writable?true,isopen?true
2019-04-09 21:57:48.681 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:FescarMergeMessage transactionId=2008546211,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,lockKey=storage_tbl:1
2019-04-09 21:57:48.687 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:MergeResultMessage BranchRegisterResponse: transactionId=2008546211,branchId=2008546212,result code =Success,getMsg =null,messageId:11
2019-04-09 21:57:48.702 DEBUG 38933 --- [nio-8081-exec-1] c.a.f.rm.datasource.undo.UndoLogManager  : Flushing UNDO LOG: {"branchId":2008546212,"sqlUndoLogs":[{"afterImage":{"rows":[{"fields":[{"keyType":"PrimaryKey","name":"id","type":4,"value":1},{"keyType":"NULL","name":"count","type":4,"value":993}]}],"tableName":"storage_tbl"},"beforeImage":{"rows":[{"fields":[{"keyType":"PrimaryKey","name":"id","type":4,"value":1},{"keyType":"NULL","name":"count","type":4,"value":994}]}],"tableName":"storage_tbl"},"sqlType":"UPDATE","tableName":"storage_tbl"}],"xid":"192.168.0.2:8091:2008546211"}
2019-04-09 21:57:48.755 DEBUG 38933 --- [nio-8081-exec-1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : offer message: transactionId=2008546211,branchId=2008546212,resourceId=null,status=PhaseOne_Done,applicationData=null
2019-04-09 21:57:48.755 DEBUG 38933 --- [geSend_RMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : write message:FescarMergeMessage transactionId=2008546211,branchId=2008546212,resourceId=null,status=PhaseOne_Done,applicationData=null, channel:[id: 0xd40718e3, L:/192.168.0.2:62607 - R:/192.168.0.2:8091],active?true,writable?true,isopen?true
2019-04-09 21:57:48.756 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:FescarMergeMessage transactionId=2008546211,branchId=2008546212,resourceId=null,status=PhaseOne_Done,applicationData=null
2019-04-09 21:57:48.758 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:MergeResultMessage com.alibaba.fescar.core.protocol.transaction.BranchReportResponse@582a08cf,messageId:13
2019-04-09 21:57:48.799 DEBUG 38933 --- [nio-8081-exec-1] c.a.fescar.core.context.RootContext      : unbind 192.168.0.2:8091:2008546211
2019-04-09 21:57:48.799 DEBUG 38933 --- [nio-8081-exec-1] o.s.c.a.f.web.FescarHandlerInterceptor   : unbind 192.168.0.2:8091:2008546211 from RootContext
```

 1. 获取business-service传来的XID
 2. 绑定XID到当前上下文中
 3. 执行业务逻辑sql
 4. 向TC创建本次RM的Netty连接
 5. 向TC发送分支事务的相关信息
 6. 获得TC返回的branchId
 7. 记录Undo Log数据
 8. 向TC发送本次事务PhaseOne阶段的处理结果
 9. 从当前上下文中解绑XID

其中第1步和第9步，是在[FescarHandlerInterceptor](https://github.com/dongsheep/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-fescar/src/main/java/org/springframework/cloud/alibaba/fescar/web/FescarHandlerInterceptor.java)中完成的，该类并不属于fescar，是前面提到的spring-cloud-alibaba-fescar,它实现了基于feign、rest通信时将xid bind和unbind到当前请求上下文中。到这里RM完成了PhaseOne阶段的工作，接着看PhaseTwo阶段的处理逻辑。
## 事务提交
各分支事务执行完成后，TC对各RM的汇报结果进行汇总，给各RM发送commit或rollback的指令
```java
2019-04-09 21:57:49.813 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Receive:xid=192.168.0.2:8091:2008546211,branchId=2008546212,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,applicationData=null,messageId:1
2019-04-09 21:57:49.813 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.AbstractRpcRemoting    : com.alibaba.fescar.core.rpc.netty.RmRpcClient@7d61f5d4 msgId:1, body:xid=192.168.0.2:8091:2008546211,branchId=2008546212,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,applicationData=null
2019-04-09 21:57:49.814  INFO 38933 --- [atch_RMROLE_1_8] c.a.f.core.rpc.netty.RmMessageListener   : onMessage:xid=192.168.0.2:8091:2008546211,branchId=2008546212,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false,applicationData=null
2019-04-09 21:57:49.816  INFO 38933 --- [atch_RMROLE_1_8] com.alibaba.fescar.rm.AbstractRMHandler  : Branch committing: 192.168.0.2:8091:2008546211 2008546212 jdbc:mysql://127.0.0.1:3306/db_storage?useSSL=false null
2019-04-09 21:57:49.816  INFO 38933 --- [atch_RMROLE_1_8] com.alibaba.fescar.rm.AbstractRMHandler  : Branch commit result: PhaseTwo_Committed
2019-04-09 21:57:49.817  INFO 38933 --- [atch_RMROLE_1_8] c.a.fescar.core.rpc.netty.RmRpcClient    : RmRpcClient sendResponse branchStatus=PhaseTwo_Committed,result code =Success,getMsg =null
2019-04-09 21:57:49.817 DEBUG 38933 --- [atch_RMROLE_1_8] c.a.f.c.rpc.netty.AbstractRpcRemoting    : send response:branchStatus=PhaseTwo_Committed,result code =Success,getMsg =null,channel:[id: 0xd40718e3, L:/192.168.0.2:62607 - R:/192.168.0.2:8091]
2019-04-09 21:57:49.817 DEBUG 38933 --- [lector_RMROLE_1] c.a.f.c.rpc.netty.MessageCodecHandler    : Send:branchStatus=PhaseTwo_Committed,result code =Success,getMsg =null
```
从日志中可以看到
1. RM收到XID=192.168.0.2:8091:2008546211,branchId=2008546212的commit通知
2. 执行commit动作
3. 将commit结果发送给TC，branchStatus为PhaseTwo_Committed

具体看下二阶段commit的执行过程，在[AbstractRMHandler](https://github.com/seata/seata/blob/develop/rm/src/main/java/com/alibaba/fescar/rm/AbstractRMHandler.java)类的doBranchCommit方法
```java
/**
 * 拿到通知的xid、branchId等关键参数
 * 然后调用RM的branchCommit
 */
protected void doBranchCommit(BranchCommitRequest request, BranchCommitResponse response) throws TransactionException {
    String xid = request.getXid();
    long branchId = request.getBranchId();
    String resourceId = request.getResourceId();
    String applicationData = request.getApplicationData();
    LOGGER.info("Branch committing: " + xid + " " + branchId + " " + resourceId + " " + applicationData);
    BranchStatus status = getResourceManager().branchCommit(request.getBranchType(), xid, branchId, resourceId, applicationData);
    response.setBranchStatus(status);
    LOGGER.info("Branch commit result: " + status);
}
```
最终会将branchCommit的请求调用到[AsyncWorker](https://github.com/seata/seata/blob/develop/rm-datasource/src/main/java/com/alibaba/fescar/rm/datasource/AsyncWorker.java)的branchCommit方法。AsyncWorker的处理方式是fescar架构的一个关键部分，因为大部分事务都是会正常提交的，所以在PhaseOne阶段就已经结束了，这样就可以将锁最快的释放。PhaseTwo阶段接收commit的指令后，异步处理即可。将PhaseTwo的时间消耗排除在一次分布式事务之外。
```java
private static final List<Phase2Context> ASYNC_COMMIT_BUFFER = Collections.synchronizedList( new ArrayList<Phase2Context>());
        
/**
 * 将需要提交的XID加入list
 */
@Override
public BranchStatus branchCommit(BranchType branchType, String xid, long branchId, String resourceId, String applicationData) throws TransactionException {
    if (ASYNC_COMMIT_BUFFER.size() < ASYNC_COMMIT_BUFFER_LIMIT) {
        ASYNC_COMMIT_BUFFER.add(new Phase2Context(branchType, xid, branchId, resourceId, applicationData));
    } else {
        LOGGER.warn("Async commit buffer is FULL. Rejected branch [" + branchId + "/" + xid + "] will be handled by housekeeping later.");
    }
    return BranchStatus.PhaseTwo_Committed;
}
	
/**
 * 通过定时任务消费list中的XID
 */
public synchronized void init() {
    LOGGER.info("Async Commit Buffer Limit: " + ASYNC_COMMIT_BUFFER_LIMIT);
    timerExecutor = new ScheduledThreadPoolExecutor(1,
        new NamedThreadFactory("AsyncWorker", 1, true));
    timerExecutor.scheduleAtFixedRate(new Runnable() {
        @Override
        public void run() {
            try {
                doBranchCommits();
            } catch (Throwable e) {
                LOGGER.info("Failed at async committing ... " + e.getMessage());
            }
        }
    }, 10, 1000 * 1, TimeUnit.MILLISECONDS);
}
	
private void doBranchCommits() {
    if (ASYNC_COMMIT_BUFFER.size() == 0) {
        return;
    }
    Map<String, List<Phase2Context>> mappedContexts = new HashMap<>();
    Iterator<Phase2Context> iterator = ASYNC_COMMIT_BUFFER.iterator();
    
    //一次定时循环取出ASYNC_COMMIT_BUFFER中的所有待办数据
    //以resourceId作为key分组待commit数据，resourceId是一个数据库的连接url
    //在前面的日志中可以看到，目的是为了覆盖应用的多数据源创建
    while (iterator.hasNext()) {
        Phase2Context commitContext = iterator.next();
        List<Phase2Context> contextsGroupedByResourceId = mappedContexts.get(commitContext.resourceId);
        if (contextsGroupedByResourceId == null) {
            contextsGroupedByResourceId = new ArrayList<>();
            mappedContexts.put(commitContext.resourceId, contextsGroupedByResourceId);
        }
        contextsGroupedByResourceId.add(commitContext);

        iterator.remove();

    }

    for (Map.Entry<String, List<Phase2Context>> entry : mappedContexts.entrySet()) {
        Connection conn = null;
        try {
            try {
            	//根据resourceId获取数据源以及连接
                DataSourceProxy dataSourceProxy = DataSourceManager.get().get(entry.getKey());
                conn = dataSourceProxy.getPlainConnection();
            } catch (SQLException sqle) {
                LOGGER.warn("Failed to get connection for async committing on " + entry.getKey(), sqle);
                continue;
            }
            List<Phase2Context> contextsGroupedByResourceId = entry.getValue();
            for (Phase2Context commitContext : contextsGroupedByResourceId) {
                try {
                	//执行undolog的处理，即删除xid、branchId对应的记录
                    UndoLogManager.deleteUndoLog(commitContext.xid, commitContext.branchId, conn);
                } catch (Exception ex) {
                    LOGGER.warn(
                        "Failed to delete undo log [" + commitContext.branchId + "/" + commitContext.xid + "]", ex);
                }
            }

        } finally {
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException closeEx) {
                    LOGGER.warn("Failed to close JDBC resource while deleting undo_log ", closeEx);
                }
            }
        }
    }
}
```
所以对于commit动作的处理，RM只需删除xid、branchId对应的undo_log即可。
## 事务回滚
对于rollback场景的触发有两种情况
 1. 分支事务处理异常，即[ConnectionProxy](https://github.com/seata/seata/blob/develop/rm-datasource/src/main/java/com/alibaba/fescar/rm/datasource/ConnectionProxy.java)中`report(false)`的情况
 2. TM捕获到下游系统上抛的异常，即发起全局事务标有`@GlobalTransactional`注解的方法捕获到的异常。在前面[TransactionalTemplate](https://github.com/seata/seata/blob/develop/tm/src/main/java/com/alibaba/fescar/tm/api/TransactionalTemplate.java)类的execute模版方法中，对business.execute()的调用进行了catch，catch后会调用rollback，由TM通知TC对应XID需要回滚事务
 ```java
 public void rollback() throws TransactionException {
    //只有Launcher能发起这个rollback
    if (role == GlobalTransactionRole.Participant) {
        // Participant has no responsibility of committing
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Ignore Rollback(): just involved in global transaction [" + xid + "]");
        }
        return;
    }
    if (xid == null) {
        throw new IllegalStateException();
    }

    status = transactionManager.rollback(xid);
    if (RootContext.getXID() != null) {
        if (xid.equals(RootContext.getXID())) {
            RootContext.unbind();
        }
    }
}
 ```
TC汇总后向参与者发送rollback指令，RM在[AbstractRMHandler](https://github.com/seata/seata/blob/develop/rm/src/main/java/com/alibaba/fescar/rm/AbstractRMHandler.java)类的doBranchRollback方法中接收这个rollback的通知
```java
protected void doBranchRollback(BranchRollbackRequest request, BranchRollbackResponse response) throws TransactionException {
    String xid = request.getXid();
    long branchId = request.getBranchId();
    String resourceId = request.getResourceId();
    String applicationData = request.getApplicationData();
    LOGGER.info("Branch rolling back: " + xid + " " + branchId + " " + resourceId);
    BranchStatus status = getResourceManager().branchRollback(request.getBranchType(), xid, branchId, resourceId, applicationData);
    response.setBranchStatus(status);
    LOGGER.info("Branch rollback result: " + status);
}
```
然后将rollback请求传递到`DataSourceManager`类的branchRollback方法
```java
public BranchStatus branchRollback(BranchType branchType, String xid, long branchId, String resourceId, String applicationData) throws TransactionException {
    //根据resourceId获取对应的数据源
    DataSourceProxy dataSourceProxy = get(resourceId);
    if (dataSourceProxy == null) {
        throw new ShouldNeverHappenException();
    }
    try {
        UndoLogManager.undo(dataSourceProxy, xid, branchId);
    } catch (TransactionException te) {
        if (te.getCode() == TransactionExceptionCode.BranchRollbackFailed_Unretriable) {
            return BranchStatus.PhaseTwo_RollbackFailed_Unretryable;
        } else {
            return BranchStatus.PhaseTwo_RollbackFailed_Retryable;
        }
    }
    return BranchStatus.PhaseTwo_Rollbacked;
}
```
最终会执行[UndoLogManager](https://github.com/seata/seata/blob/develop/rm-datasource/src/main/java/com/alibaba/fescar/rm/datasource/undo/UndoLogManager.java)类的undo方法，因为是纯jdbc操作代码比较长就不贴出来了，可以通过连接到github查看源码，说一下undo的具体流程

 1. 根据xid和branchId查找PhaseOne阶段提交的undo_log
 2. 如果找到了就根据undo_log中记录的数据生成回放sql并执行，即还原PhaseOne阶段修改的数据
 3. 第2步处理完后，删除该条undo_log数据
 4. 如果第1步没有找到对应的undo_log，就插入一条状态为`GlobalFinished`的undo_log.
 出现没找到的原因可能是PhaseOne阶段的本地事务异常了，导致没有正常写入。
 因为xid和branchId是唯一索引，所以第4步的插入，可以防止PhaseOne阶段恢复后的成功写入，那么PhaseOne阶段就会异常，这样一来业务数据也就不会提交成功，数据达到了最终回滚了的效果
## 总结
本地结合分布式业务场景，分析了fescar client侧的主要处理流程，对TM和RM角色的主要源码进行了解析，希望能对大家理解fescar的工作原理有所帮助。

随着fescar的快速迭代以及后期的Roadmap规划，假以时日相信fescar能够成为开源分布式事务的标杆解决方案。
