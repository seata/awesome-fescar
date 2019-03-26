Alibaba Fescar 0.4.0 发布

Fescar 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新内容如下：

## 特性

- [[#583](https://github.com/alibaba/fescar/pull/583)] 新增蚂蚁金服的TCC模式，自动代理Dubbo服务和SOFARPC服务，使fescar支持除数据库以外的其他资源（RPC服务、restful服务、消息以及NoSQL等）作为分布式事务资源
- [[#594](https://github.com/alibaba/fescar/pull/611)] 新增 p3c pmd Maven插件，自动进行代码扫描并找出不规范的代码格式
- [[#627](https://github.com/alibaba/fescar/pull/627)] Maven依赖优化


## 相关链接
- Fescar: https://github.com/alibaba/fescar   
- Fescar-Samples: https://github.com/fescar-group/fescar-samples   
- Release：https://github.com/alibaba/fescar/releases
