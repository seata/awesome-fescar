Seata 0.5.1 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：


## 功能特性

- [[#774](https://github.com/seata/seata/pull/869)] 增加注册中心Etcd3支持
- [[#793](https://github.com/seata/seata/pull/793)] 增加注册中心sofa-registry支持
- [[#856](https://github.com/seata/seata/pull/856)] 增加批量删除undolog处理
- [[#786](https://github.com/seata/seata/pull/786)] 增加全局事务内分支事务并发支持



## Bug 修复及优化

- [[#879](https://github.com/seata/seata/pull/879)] 修复批量删除undolog PreparedStatement不关闭问题
- [[#945](https://github.com/seata/seata/pull/945)] 增加LockManager中releaseLock接口，优化调用逻辑
- [[#938](https://github.com/seata/seata/pull/938)] 优化TransactionManager服务加载逻辑
- [[#913](https://github.com/seata/seata/pull/938)] 优化与RPC集成框架的模块结构
- [[#795](https://github.com/seata/seata/pull/795)] 优化server节点写文件的性能
- [[#921](https://github.com/seata/seata/pull/921)] 修复select for update时的NPE异常
- [[#925](https://github.com/seata/seata/pull/925)] 优化server启动时复用同一DefaultCoordinator实例
- [[#930](https://github.com/seata/seata/pull/930)] 优化字段访问修饰符
- [[#907](https://github.com/seata/seata/pull/907)] 修复hostname can't be null异常
- [[#923](https://github.com/seata/seata/pull/923)] 修复nettyClientKeyPool连接销毁时Key未format问题
- [[#891](https://github.com/seata/seata/pull/891)] 修复select union all时NPE异常
- [[#888](https://github.com/seata/seata/pull/888)] 修复copyright checkstyle验证问题
- [[#901](https://github.com/seata/seata/pull/901)] 修复Zookeeper 注册时父节点路径不存在问题
- [[#904](https://github.com/seata/seata/pull/904)] 优化UpdateExecutort后镜像数据查询
- [[#802](https://github.com/seata/seata/pull/802)] 优化checkstyle，增加插件校验
- [[#882](https://github.com/seata/seata/pull/882)] 更改copyright，增加copyright自动插件
- [[#874](https://github.com/seata/seata/pull/874)] 增加通讯传输层默认配置值
- [[#866](https://github.com/seata/seata/pull/866)] 修复无法生成dubbo:reference代理类问题
- [[#877](https://github.com/seata/seata/pull/877)] 修复批量删除undolog时concurrentModifyException异常
- [[#855](https://github.com/seata/seata/pull/855)] 优化AT模式时globalCommit时始终返回committed给用户
- [[#875](https://github.com/seata/seata/pull/875)] 修复select for update，Boolean转型ResultSet失败问题
- [[#830](https://github.com/seata/seata/pull/830)] 修复RM延迟注册问题
- [[#872](https://github.com/seata/seata/pull/872)] 修复RegisterRMRequest解码消息长度校验不准确问题
- [[#831](https://github.com/seata/seata/pull/831)] 优化MessageFuture中CountDownLatch，使用CompletableFuture替代
- [[#834](https://github.com/seata/seata/pull/834)] 修复ExecuteTemplate中非SQLException异常不抛出问题




## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
