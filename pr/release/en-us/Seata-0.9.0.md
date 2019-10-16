## Seata 0.9.0

Seata 0.9.0 Released.

Seata is an easy-to-use, high-performance, open source distributed transaction solution.

The version is updated as follows:

### feature：
- [[#1608](https://github.com/seata/seata/pull/1608)] Saga implementation base on state machine
- [[#1625](https://github.com/seata/seata/pull/1625)] support custom config and registry type
- [[#1656](https://github.com/seata/seata/pull/1656)] support spring cloud config
- [[#1689](https://github.com/seata/seata/pull/1689)] support new parameter option "-e" used for setting name of configuration
- [[#1739](https://github.com/seata/seata/pull/1739)] support retry when tm commit or rollback failed


### bugfix：
- [[#1605](https://github.com/seata/seata/pull/1605)] fix deadlock and lock optimization
- [[#1685](https://github.com/seata/seata/pull/1685)] fix pk too long in lock table on db mode and optimize error log
- [[#1691](https://github.com/seata/seata/pull/1691)] fix can not access a member of DruidDataSourceWrapper
- [[#1699](https://github.com/seata/seata/pull/1699)] fix use 'in' and 'between' in where condition for Oracle and Mysql
- [[#1713](https://github.com/seata/seata/pull/1713)] correct LockManagerTest.concurrentUseAbilityTest assertion condition
- [[#1720](https://github.com/seata/seata/pull/1720)] fix can't refresh table meta data for oracle
- [[#1729](https://github.com/seata/seata/pull/1729)] fix oracle batch insert error
- [[#1735](https://github.com/seata/seata/pull/1735)] clean xid when tm commit or rollback failed
- [[#1749](https://github.com/seata/seata/pull/1749)] fix undo support oracle table meta cache
- [[#1751](https://github.com/seata/seata/pull/1751)] fix memory lock is not released due to hash conflict
- [[#1761](https://github.com/seata/seata/pull/1761)] fix oracle rollback failed when the table has null Blob Clob value
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


Thanks to these contributors for their code commits. Please report an unintended omission.  

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

Also, we receive many valuable issues, questions and advices from our community. Thanks for you all.

### Link
- **Seata:** https://github.com/seata/seata  
- **Seata-Samples:** https://github.com/seata/seata-samples   
- **Release:** https://github.com/seata/seata/releases
