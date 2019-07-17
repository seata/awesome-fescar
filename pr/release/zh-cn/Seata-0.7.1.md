Seata 0.7.1 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

0.7.1 版本是针对0.7.0 版本问题的紧急修复，本次更新主要内容如下：


## Bug 修复及优化

- [[#1297](https://github.com/seata/seata/pull/1297)] 兼容seata-spring独立依赖用法，对seata-spring添加了seata-codec-all依赖
- [[#1305](https://github.com/seata/seata/pull/1305)] 修复GlobalTransactionScanner 切面优先级导致的Spring Cloud 的AutoConfiguration无法初始化问题
- 修复了0.7.0 因mvn插件过低导致的版本号无替换，无法从中央仓库拉取依赖的问题。


## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases