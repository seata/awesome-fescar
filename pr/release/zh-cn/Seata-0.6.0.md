Seata 0.6.0 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：


## 功能特性

- [[#942](https://github.com/seata/seata/pull/942)] 服务端使用数据库存储事务日志，支持服务端集群部署
- [[#1014](https://github.com/seata/seata/pull/1014)] 支持 etcd3 作为配置中心
- [[#1060](https://github.com/seata/seata/pull/1060)] 添加事务回滚时脏写校验

## Bug 修复及优化

- [[#1064](https://github.com/seata/seata/pull/1064)] 修复 xid 和 branchId 长度错误
- [[#1074](https://github.com/seata/seata/pull/1074)] 修复一些拼写错误，并用lambda替换匿名类 
- [[#824](https://github.com/seata/seata/pull/824)] 添加事务恢复重试超时时间限制
- [[#1082](https://github.com/seata/seata/pull/1082)] 添加配置中心单实例缓存
- [[#1084](https://github.com/seata/seata/pull/1084)] 重构字符集和blob工具类
- [[#1080](https://github.com/seata/seata/pull/1080)] 升级fastjson和nacos-client版本



## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
