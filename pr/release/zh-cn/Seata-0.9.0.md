## Seata 0.9.0 

Seata 0.9.0 正式发布。

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

此版本更新如下：


### feature：
- [[#1608](https://github.com/seata/seata/pull/1608)] 长事务解决方案: Saga 模式（基于状态机实现）
- [[#1625](https://github.com/seata/seata/pull/1625)] 支持自定义配置和注册中心类型
- [[#1656](https://github.com/seata/seata/pull/1656)] 支持 spring cloud config 配置中心
- [[#1689](https://github.com/seata/seata/pull/1689)] 支持 -e 启动参数，用于指定环境名称
- [[#1739](https://github.com/seata/seata/pull/1739)] 支持 TM commit 或rollback 失败时的重试


### bugfix：
- [[#1605](https://github.com/seata/seata/pull/1605)] 修复对象锁和全局锁可能造成的死锁和优化锁的粒度
- [[#1685](https://github.com/seata/seata/pull/1685)] 修复db存储类异常被忽略的问题
- [[#1691](https://github.com/seata/seata/pull/1691)] 修复 DruidDataSourceWrapper 反射问题
- [[#1699](https://github.com/seata/seata/pull/1699)] 修复 mysql 和 oracle 中 'in' 和 'between' 在 where 条件的支持
- [[#1713](https://github.com/seata/seata/pull/1713)] 修复 LockManagerTest.concurrentUseAbilityTest 中的测试条件
- [[#1720](https://github.com/seata/seata/pull/1720)] 修复了不能获取 oracle tableMeta 问题
- [[#1729](https://github.com/seata/seata/pull/1729)] 修复 oracle 的批量获取问题
- [[#1735](https://github.com/seata/seata/pull/1735)] 修复当 TM commit 或 rollback 出现网络异常无法清除 xid 的问题
- [[#1749](https://github.com/seata/seata/pull/1749)] 修复无法获取 oracle tableMeta cache 问题
- [[#1751](https://github.com/seata/seata/pull/1751)] 修复文件存储模式下由于hash冲突导致的锁无法释放问题
- [[#1761](https://github.com/seata/seata/pull/1761)] 修复 oracle 在回滚时 Blob 或 Clob null 值回滚失败问题
- [[#1759](https://github.com/seata/seata/pull/1759)] 修复 saga 模式下 service method 不支持接口类型参数问题
- [[#1401](https://github.com/seata/seata/pull/1401)] 修复 RM 启动时第一次注册 resource 为 null 的问题



### optimize： 
- [[#1701](https://github.com/seata/seata/pull/1701)] 移除无用的 imports
- [[#1705](https://github.com/seata/seata/pull/1705)] 优化了一些基于 java5 的语法结构
- [[#1706](https://github.com/seata/seata/pull/1706)] 将内部类声明为 static
- [[#1707](https://github.com/seata/seata/pull/1707)] 使用 StandardCharsets.UTF_8 代替 utf-8 编码
- [[#1712](https://github.com/seata/seata/pull/1712)] 抽象 undologManager 的通用方法
- [[#1722](https://github.com/seata/seata/pull/1722)] 简化代码提高代码的可读性
- [[#1726](https://github.com/seata/seata/pull/1726)] 格式化日志输出
- [[#1738](https://github.com/seata/seata/pull/1738)] 增加 seata-server jvm 参数
- [[#1743](https://github.com/seata/seata/pull/1743)] 提高批量打印日志的性能
- [[#1747](https://github.com/seata/seata/pull/1747)] 使用基本类型避免数据装箱
- [[#1750](https://github.com/seata/seata/pull/1750)] 抽象 tableMetaCache 方法
- [[#1755](https://github.com/seata/seata/pull/1755)] 提高 seata-common 模块的单测覆盖率
- [[#1756](https://github.com/seata/seata/pull/1756)] 升级 jackson 版本防止潜在的安全漏洞
- [[#1657](https://github.com/seata/seata/pull/1657)] 优化文件存储模式下文件 rolling 时占用较大 direct buffer的问题



非常感谢以下 contributors 的代码贡献。若有无意遗漏，请报告。

- [slievrly](https://github.com/slievrly)
- [long187](https://github.com/long187)
- [ggndnn](https://github.com/ggndnn)
- [xingfudeshi](https://github.com/xingfudeshi)
- [BeiKeJieDeLiuLangMao](https://github.com/BeiKeJieDeLiuLangMao)
- [zjinlei](https://github.com/zjinlei)
- [cmonkey](https://github.com/cmonkey)
- [jsbxyyx](https://github.com/jsbxyyx)
- [zaqweb](https://github.com/zaqweb)
- [tjnettech](https://github.com/tjnettech)
- [l81893521](https://github.com/l81893521)
- [abel533](https://github.com/abel533)
- [suhli](https://github.com/suhli)
- [github-ygy](https://github.com/github-ygy)
- [worstenemy](https://github.com/worstenemy)
- [caioguedes](https://github.com/caioguedes)

同时，我们收到了社区反馈的很多有价值的issue和建议，非常感谢大家。


### 常用链接
- **Seata:** https://github.com/seata/seata  
- **Seata-Samples:** https://github.com/seata/seata-samples   
- **Release:** https://github.com/seata/seata/releases
