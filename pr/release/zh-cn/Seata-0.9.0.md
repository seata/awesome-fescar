## Seata 0.9.0 

Seata 0.9.0 正式发布。

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

此版本更新如下：


### feature：
- [[#1608](https://github.com/seata/seata/pull/1608)] 长事务解决方案: Saga模式（基于状态机实现）
- [[#1625](https://github.com/seata/seata/pull/1625)] 支持自定义配置和注册中心类型
- [[#1656](https://github.com/seata/seata/pull/1656)] 支持spring cloud配置
- [[#1689](https://github.com/seata/seata/pull/1689)] 支持新的参数选项"-e", 用于设置配置的名称
- [[#1739](https://github.com/seata/seata/pull/1739)] 支持当commit或rollback失败的时候重试


### bugfix：
- [[#1605](https://github.com/seata/seata/pull/1605)] 修复死锁问题和优化锁的实现在获取锁之前先用主键进行排序
- [[#1685](https://github.com/seata/seata/pull/1685)] fix pk too long in lock table on db mode and optimize error log
- [[#1691](https://github.com/seata/seata/pull/1691)] fix can not access a member of DruidDataSourceWrapper
- [[#1699](https://github.com/seata/seata/pull/1699)] fix use 'in' and 'between' in where condition for Oracle and Mysql
- [[#1713](https://github.com/seata/seata/pull/1713)] correct LockManagerTest.concurrentUseAbilityTest assertion condition
- [[#1720](https://github.com/seata/seata/pull/1720)] fix can't refresh table meta data for oracle
- [[#1729](https://github.com/seata/seata/pull/1729)] fix oracle batch insert error
- [[#1735](https://github.com/seata/seata/pull/1735)] clean xid when tm commit or rollback failed
- [[#1749](https://github.com/seata/seata/pull/1749)] fix undo support oracle table meta cache
- [[#1751](https://github.com/seata/seata/pull/1751)] fix memory lock is not released due to hash conflict
- [[#1761](https://github.com/seata/seata/pull/1761)] fix racle rollback failed when the table has null Blob Clob value
- [[#1759](https://github.com/seata/seata/pull/1759)] fix saga service method not support interface type parameter
- [[#1401](https://github.com/seata/seata/pull/1401)] fix rm channel register null resource
- [[#1761](https://github.com/seata/seata/pull/1761)] fix oracle rollback failed when the table has null Blob Clob value



### optimize： 
- [[#1701](https://github.com/seata/seata/pull/1701)] remove unused imports
- [[#1705](https://github.com/seata/seata/pull/1705)] Based on Java 5 optimization 
- [[#1706](https://github.com/seata/seata/pull/1706)] inner class may be static
- [[#1707](https://github.com/seata/seata/pull/1707)] default charset use StandardCharsets.UTF_8 
- [[#1712](https://github.com/seata/seata/pull/1712)] abstract common undolog manager method
- [[#1722](https://github.com/seata/seata/pull/1722)] simplify to make codes more readable
- [[#1726](https://github.com/seata/seata/pull/1726)] formating log messages
- [[#1738](https://github.com/seata/seata/pull/1738)] add some server's jvm parameters
- [[#1743](https://github.com/seata/seata/pull/1743)] improve the efficiency of the batch log
- [[#1747](https://github.com/seata/seata/pull/1747)] use raw types instead of boxing types
- [[#1750](https://github.com/seata/seata/pull/1750)] abstract table meta cache
- [[#1755](https://github.com/seata/seata/pull/1755)] test: enhance test coverage of seata common
- [[#1756](https://github.com/seata/seata/pull/1756)] security: upgrade jackson to avoid security vulnerabilities
- [[#1657](https://github.com/seata/seata/pull/1657)] eliminate the possibility of allocating too much direct memory



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
