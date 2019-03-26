Alibaba Fescar 0.3.1 发布

Fescar 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新内容如下：

## 特性

- [[#557](https://github.com/alibaba/fescar/issues/557)] 增加事务处理各阶段用户自定义 hook 接入点支持
- [[#594](https://github.com/alibaba/fescar/pull/594)] 增加 ZooKeeper 注册中心支持   

## Bug 修复

- [[#569](https://github.com/alibaba/fescar/issues/569)] 修复 Eureka renew 问题
- [[#551](https://github.com/alibaba/fescar/pull/551)] 修复 ConfigType NPE 问题   
- [[#489](https://github.com/alibaba/fescar/issues/489)] 修复 GlobalRollback 请求时未收到分支 branchReport 问题
- [[#598](https://github.com/alibaba/fescar/pull/598)] 修复 p3c 扫描出不符合规范的若干问题；



## 相关链接
- Fescar: https://github.com/alibaba/fescar   
- Fescar-Samples: https://github.com/fescar-group/fescar-samples   
- Release：https://github.com/alibaba/fescar/releases
