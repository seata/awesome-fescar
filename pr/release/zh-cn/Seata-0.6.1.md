
Seata 0.6.1 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：


## 功能特性

- [[#1119](https://github.com/seata/seata/pull/1119)] 支持 weibo/motan 上下文透传
- [[#1075](https://github.com/seata/seata/pull/1075)] 支持多环境配置隔离

## Bug 修复及优化

- [[#1099](https://github.com/seata/seata/pull/1099)] 将UndoLogParser修改成SPI形式
- [[#1113](https://github.com/seata/seata/pull/1113)] 优化代码格式
- [[#1087](https://github.com/seata/seata/pull/1087)] 去掉无用的字节复制
- [[#1090](https://github.com/seata/seata/pull/1090)] 修改UndoLogParser的方法的返回格式，便于后续扩展
- [[#1120](https://github.com/seata/seata/pull/1120)] 修复分支事务提交和回滚时 xid使用错误的问题
- [[#1135](https://github.com/seata/seata/pull/1135)] 升级zookeeper以修复安全漏洞
- [[#1138](https://github.com/seata/seata/pull/1138)] 修复windows下seata-server.bat classpath过长的问题
- [[#1117](https://github.com/seata/seata/pull/1117)] 修复脏写校验时时间类型数据校验失败问题
- [[#1115](https://github.com/seata/seata/pull/1115)] 配置 seata-all 和 seata-bom 打包发布环境


## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
