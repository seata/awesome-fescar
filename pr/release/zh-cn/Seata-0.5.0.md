Seata 0.5.0 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：

### 兼容性变更

- [[#809](https://github.com/seata/seata/pull/809)] 更改 groupid、artifactid和包路径
- [[#815](https://github.com/seata/seata/pull/815)] 添加maven 插件，以支持使用 groupId “io.seata” 发包
- [[#790](https://github.com/seata/seata/pull/790)] 修改服务器的启动参数以支持数据库存储模式
- [[#769](https://github.com/seata/seata/pull/769)] 重构RPC协议，在客户端中去掉XID的解析，使得服务端变成无状态

## 功能特性

- [[#774](https://github.com/seata/seata/pull/774)] 优化配置中心和注册中心的结构
- [[#783](https://github.com/seata/seata/pull/783)] 允许用户自定义分支事务记录报告重试次数
- [[#791](https://github.com/seata/seata/pull/791)] 用状态枚举替换超时状态的模糊判断
- [[#836](https://github.com/seata/seata/pull/836)] 添加maven插件，管理工程版本号
- [[#820](https://github.com/seata/seata/pull/820)] 添加按异常回滚事务的特性


## Bug 修复

- [[#772](https://github.com/seata/seata/pull/772)] 修复文件配置中心监听器问题
- [[#807](https://github.com/seata/seata/pull/807)] 优化服务端文件存储器的文件路径
- [[#804](https://github.com/seata/seata/pull/804)] 修复分支提交不断重试问题



## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/fescar-group/fescar-samples   
- Release：https://github.com/seata/seata/releases
