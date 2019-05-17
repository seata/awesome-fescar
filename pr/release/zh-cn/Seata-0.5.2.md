Seata 0.5.2 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：


## 功能特性

- [[#988](https://github.com/seata/seata/pull/988)] 增加配置中心Consul支持
- [[#1043](https://github.com/seata/seata/pull/1043)] 增加sofa-rpc支持


## Bug 修复及优化

- [[#987](https://github.com/seata/seata/pull/987)] 优化同事务内并发使用 reentrantLock 代替 spinlock
- [[#943](https://github.com/seata/seata/pull/943)] 修复无相应文件配置项时取配置等待超时问题
- [[#965](https://github.com/seata/seata/pull/965)] 修复PreparedStatement 时where语句中 in、between 报错问题
- [[#929](https://github.com/seata/seata/pull/929)] 优化GlobalSession第一次取锁等待问题
- [[#967](https://github.com/seata/seata/pull/967)] 优化部分日志描述
- [[#970](https://github.com/seata/seata/pull/970)] 修复无法读取flush-disk-mode配置项问题
- [[#916](https://github.com/seata/seata/pull/916)] 优化解码时readable index
- [[#979](https://github.com/seata/seata/pull/979)] 优化copyright
- [[#981](https://github.com/seata/seata/pull/981)] 优化pom依赖，使用 caffine 代替 guava cache，junit升级至junit5，使用junit5改造原有testng单元测试
- [[#991](https://github.com/seata/seata/pull/991)] 优化core模块的文件头import
- [[#996](https://github.com/seata/seata/pull/996)] 修复maven-surefire-plugin在mac环境下编译错误问题
- [[#994](https://github.com/seata/seata/pull/994)] 修复ByteBuffer多次flip问题
- [[#999](https://github.com/seata/seata/pull/999)] 更改社区邮件订阅地址
- [[#861](https://github.com/seata/seata/pull/861)] 优化FailureHandler定时获取重试的事务结果，并将成功结果打印
- [[#802](https://github.com/seata/seata/pull/802)] 优化GlobalTransactionalInterceptor中lambda代码风格
- [[#1026](https://github.com/seata/seata/pull/1026)] 修复错误排除data*代码文件问题，增加本地事务文件排除路径
- [[#1024](https://github.com/seata/seata/pull/1024)] 修复Consul模块SPI配置文件路径错误问题
- [[#1023](https://github.com/seata/seata/pull/1023)] 增加seata-all客户端依赖jar包
- [[#1029](https://github.com/seata/seata/pull/1029)] 修复回滚中客户端宕机重启回滚时无channel导致的延迟回滚问题
- [[#1027](https://github.com/seata/seata/pull/1027)] 修复release-seata无法生成压缩包问题
- [[#1033](https://github.com/seata/seata/pull/1033)] 修复createDependencyReducedPom生成多余xml问题
- [[#1035](https://github.com/seata/seata/pull/1035)] 修复TCC模式中branchCommit/branchRollback，branchId为null问题
- [[#1040](https://github.com/seata/seata/pull/1040)] 重构exceptionHandleTemplate,修复GlobalRollback 分支异常时无法返回状态问题
- [[#1036](https://github.com/seata/seata/pull/1036)] 替换中文注释为相应英文注释
- [[#1051](https://github.com/seata/seata/pull/1051)] 优化回滚时校验数据变化，若无变化停止回滚
- [[#1017](https://github.com/seata/seata/pull/1017)] 优化mysql undo executor构造undo sql逻辑处理
- [[#1063](https://github.com/seata/seata/pull/1063)] 修复server重启后事务恢复后，可能造成新事务id冲突失败问题




## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
