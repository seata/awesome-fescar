Seata 0.4.2 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：

## 特性

- [[#704](https://github.com/seata/seata/pull/704)] 增加 本地文件写入时 ByteBuffer 池
- [[#679](https://github.com/seata/seata/issues/679)] 增加 现有注册中心增加了 close 接口实现，优化了 server 优雅下线 
- [[#713](https://github.com/seata/seata/pull/713)] 增加 本地文件写入对超过配置大小的消息启用压缩功能  
- [[#587](https://github.com/seata/seata/issues/587)] 增加 MySQL DDL 语句支持 
- [[#717](https://github.com/seata/seata/pull/717)] 增加 Nacos 初始化配置脚本配置和补全程序配置文件
- [[#726](https://github.com/seata/seata/pull/726)] 增加 DBCP, C3P0, BoneCP, HikariCP 和 Tomcat-JDBC 连接池的支持
- [[#744](https://github.com/seata/seata/pull/744)] 增加 ZooKeeper 断线重连时重新注册和订阅
- [[#728](https://github.com/seata/seata/pull/728)] 增加 Consul 注册中心支持

## Bug 修复

- [[#569](https://github.com/seata/seata/pull/695)] 修复 已是jdk代理且无 target 只遍历第一个实现接口的问题
- [[#721](https://github.com/seata/seata/pull/721)] 修复 ConfigFuture 构造方法超时参数不起作用的问题
- [[#725](https://github.com/seata/seata/pull/725)] 修复 MergedSendRunnable channel被意外关闭问题，增加 fail-fast 机制
- [[#723](https://github.com/seata/seata/pull/723)] 修复 defaultServerMessageListener 未初始化的问题
- [[#746](https://github.com/seata/seata/pull/746)] 修复 DataSourceManager SPI 导致的test module 集测用例全部失效问题
- [[#754](https://github.com/seata/seata/pull/754)] 优化 Eureka 注册中心实现
- [[#750](https://github.com/seata/seata/pull/750)] 修复 DataSourceManager SPI 导致的 undolog 无法删除问题
- [[#747](https://github.com/seata/seata/pull/747)] 删除 MT 模式，之后将被 TCC 模式代替 
- [[#757](https://github.com/seata/seata/pull/757)] 修复 BranchRollback 异常后回滚重试被终止问题
- [[#776](https://github.com/seata/seata/pull/776)] 修复 连接池创建 channel 时 toString 异常导致的连接创建失败问题



## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/fescar-group/fescar-samples   
- Release：https://github.com/seata/seata/releases
