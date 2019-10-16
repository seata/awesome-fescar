## Seata 0.8.1

Seata 0.8.1 Released.

Seata is an easy-to-use, high-performance, open source distributed transaction solution.

The version is updated as follows:

### feature：
- [[#1598](https://github.com/seata/seata/pull/1598)] support profile to use absolute path
- [[#1617](https://github.com/seata/seata/pull/1617)] support profile’s（registry.conf） name configurable
- [[#1418](https://github.com/seata/seata/pull/1418)] support undo_log kryo serializer
- [[#1489](https://github.com/seata/seata/pull/1489)] support protobuf maven plugin
- [[#1437](https://github.com/seata/seata/pull/1437)] support kryo codec
- [[#1478](https://github.com/seata/seata/pull/1478)] support db mock
- [[#1512](https://github.com/seata/seata/pull/1512)] extended support for mysql and oracle multiple insert batch syntax
- [[#1496](https://github.com/seata/seata/pull/1496)] support auto proxy of DataSource 


### bugfix：
- [[#1646](https://github.com/seata/seata/pull/1646)] fix selectForUpdate lockQuery exception in file mode
- [[#1572](https://github.com/seata/seata/pull/1572)] fix get tablemeta fail in oracle when table name was lower case 
- [[#1663](https://github.com/seata/seata/pull/1663)] fix get tablemeta fail when table name was keyword
- [[#1666](https://github.com/seata/seata/pull/1666)] fix restore connection's autocommit
- [[#1643](https://github.com/seata/seata/pull/1643)] fix serialize and deserialize in java.sql.Blob, java.sql.Clob
- [[#1628](https://github.com/seata/seata/pull/1628)] fix oracle support ROWNUM query
- [[#1552](https://github.com/seata/seata/pull/1552)] fix BufferOverflow when BranchSession size too large
- [[#1609](https://github.com/seata/seata/pull/1609)] fix thread unsafe of oracle keyword checker
- [[#1599](https://github.com/seata/seata/pull/1599)] fix thread unsafe of mysql keyword checker
- [[#1607](https://github.com/seata/seata/pull/1607)] fix NoSuchMethodError when the version of druid used < 1.1.3 
- [[#1581](https://github.com/seata/seata/pull/1581)] fix missing some length in GlobalSession and FileTransactionStoreManager 
- [[#1594](https://github.com/seata/seata/pull/1594)] fix nacos's default namespace
- [[#1550](https://github.com/seata/seata/pull/1550)] fix calculate BranchSession size missing xidBytes.length
- [[#1558](https://github.com/seata/seata/pull/1558)] fix NPE when the rpcMessage's body is null
- [[#1505](https://github.com/seata/seata/pull/1505)] fix bind public network address listen failed
- [[#1539](https://github.com/seata/seata/pull/1539)] fix nacos namespace setting does not take effect
- [[#1537](https://github.com/seata/seata/pull/1537)] fix nacos-config.txt missing store.db.driver-class-name property
- [[#1522](https://github.com/seata/seata/pull/1522)] fix ProtocolV1CodecTest testAll may be appears test not pass 
- [[#1525](https://github.com/seata/seata/pull/1525)] fix when getAfterImage error, trx autocommit 
- [[#1518](https://github.com/seata/seata/pull/1518)] fix EnhancedServiceLoader may be appears load class error
- [[#1514](https://github.com/seata/seata/pull/1514)] fix when lack serialization dependence can't generate undolog and report true
- [[#1445](https://github.com/seata/seata/pull/1445)] fix DefaultCoordinatorMetricsTest UT failed
- [[#1481](https://github.com/seata/seata/pull/1481)] fix TableMetaCache refresh problem in multiple datasource


### optimize： 
- [[#1629](https://github.com/seata/seata/pull/1629)] optimize the watcher efficiency of etcd3
- [[#1661](https://github.com/seata/seata/pull/1661)] optimize global_table insert transaction_name size 
- [[#1633](https://github.com/seata/seata/pull/1633)] optimize branch transaction repeated reporting false 
- [[#1654](https://github.com/seata/seata/pull/1654)] optimize wrong usage of slf4j  
- [[#1593](https://github.com/seata/seata/pull/1593)] optimize and standardize server log 
- [[#1648](https://github.com/seata/seata/pull/1648)] optimize transaction_name length when building the table
- [[#1576](https://github.com/seata/seata/pull/1576)] eliminate the impact of instructions reordering on session async committing task 
- [[#1618](https://github.com/seata/seata/pull/1618)] optimize undolog manager and fix delete undolog support oracle
- [[#1469](https://github.com/seata/seata/pull/1469)] reduce the number of lock conflict exception  
- [[#1619](https://github.com/seata/seata/pull/1416)] replace StringBuffer with StringBuilder
- [[#1580](https://github.com/seata/seata/pull/1580)] optimize LockKeyConflictException and change register method
- [[#1574](https://github.com/seata/seata/pull/1574)] optimize once delete GlobalSession locks for db mode when commit success 
- [[#1601](https://github.com/seata/seata/pull/1601)] optimize typo
- [[#1602](https://github.com/seata/seata/pull/1602)] upgrade fastjson version to 1.2.60 for security issue 
- [[#1583](https://github.com/seata/seata/pull/1583)] optimize get oracle primary index
- [[#1575](https://github.com/seata/seata/pull/1575)] add UT for RegisterTMRequest 
- [[#1559](https://github.com/seata/seata/pull/1559)] optimize delay to delete the expired undo log
- [[#1547](https://github.com/seata/seata/pull/1547)] TableRecords delete jackson annotation 
- [[#1542](https://github.com/seata/seata/pull/1542)] optimize  AbstractSessionManager debug log
- [[#1535](https://github.com/seata/seata/pull/1535)] remove H2 and pgsql get primary index code and close resultSet
- [[#1541](https://github.com/seata/seata/pull/1541)] code clean
- [[#1544](https://github.com/seata/seata/pull/1544)] remove Chinese comment
- [[#1533](https://github.com/seata/seata/pull/1533)] refactor of the logics of Multi-configuration Isolation
- [[#1493](https://github.com/seata/seata/pull/1493)] add table meta checker switch
- [[#1530](https://github.com/seata/seata/pull/1530)] throw Exception when no index in the table
- [[#1444](https://github.com/seata/seata/pull/1444)] simplify operation of map
- [[#1497](https://github.com/seata/seata/pull/1497)] add seata-all dependencies
- [[#1490](https://github.com/seata/seata/pull/1490)] remove unnecessary code



Thanks to these contributors for their code commits. Please report an unintended omission.  

- [slievrly](https://github.com/slievrly)
- [BeiKeJieDeLiuLangMao](https://github.com/BeiKeJieDeLiuLangMao)
- [jsbxyyx](https://github.com/jsbxyyx)
- [ldcsaa](https://github.com/ldcsaa)
- [zjinlei](https://github.com/zjinlei)
- [l81893521](https://github.com/l81893521)
- [ggndnn](https://github.com/ggndnn)
- [github-ygy](https://github.com/github-ygy)
- [chenxi-null](https://github.com/chenxi-null)
- [tq02ksu](https://github.com/tq02ksu)
- [AjaxXu](https://github.com/AjaxXu)
- [finalcola](https://github.com/finalcola)
- [lovepoem](https://github.com/lovepoem)
- [cmonkey](https://github.com/cmonkey)
- [xingfudeshi](https://github.com/xingfudeshi)
- [andyqian](https://github.com/andyqian)
- [tswstarplanet](https://github.com/tswstarplanet)
- [zhengyangyong](https://github.com/zhengyangyong)

Also, we receive many valuable issues, questions and advices from our community. Thanks for you all.

### Link
- **Seata:** https://github.com/seata/seata  
- **Seata-Samples:** https://github.com/seata/seata-samples   
- **Release:** https://github.com/seata/seata/releases
