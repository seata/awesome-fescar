Alibaba Fescar 0.2.0 发布

Fescar 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新内容如下：

## 特性

- 支持 MySQL 分布式事务自动模式（AT）
- 支持 Dubbo 无缝集成
- 支持 分布式事务 API
- 支持 Spring 事务注解
- 支持 Mybatis、JPA
- 支持 Nacos 服务注册和配置中心
- 增加 server 重启时从文件自动恢复未完成事务操作至内存
- 支持 多 IP 环境下，启动 server 指定 IP 参数

## Bug 修复

- 修复 server 重启可能导致 XID 重复问题
- 修复 Windows 启动脚本 $EXTRA_JVM_ARGUMENTS 参数报错
- 修复分布式事务本地嵌套内层事务提交/回滚导致外层事务异常问题
- 修复本地事务提交时异常，本地事务不回滚问题
- 修复 MySQL 表别名解析问题

## 其他
- 升级依赖 JDK 版本至 1.8
- 将依赖 Alibaba Dubbo 升级至 Apache Dubbo 2.7.0
- 优化相关依赖引用


## 相关链接
- Fescar: https://github.com/alibaba/fescar   
- Fescar-Samples: https://github.com/fescar-group/fescar-samples   
- Release：https://github.com/alibaba/fescar/releases
