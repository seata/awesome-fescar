## Seata 0.8.0 
Seata 0.8.0 Released.

Seata is an easy-to-use, high-performance, open source distributed transaction solution.

The version is updated as follows:

### feature：
- [[#902](https://github.com/seata/seata/pull/902)] support oracle database in AT mode
- [[#1447](https://github.com/seata/seata/pull/1447)] support oracle batch operation
- [[#1392](https://github.com/seata/seata/pull/1392)] support undo log table name configurable 
- [[#1353](https://github.com/seata/seata/pull/1353)] support mysql batch update and batch delete
- [[#1379](https://github.com/seata/seata/pull/1379)] support -Dkey=value SysConfig
- [[#1365](https://github.com/seata/seata/pull/1365)] support schedule check table mata
- [[#1371](https://github.com/seata/seata/pull/1371)] support mysql preparedStatement batch self-increment primary keys
- [[#1337](https://github.com/seata/seata/pull/1337)] support mysql batch insert for non-self-inc primary keys
- [[#1235](https://github.com/seata/seata/pull/1453)] support delete expired undolog use protobuf codec
- [[#1235](https://github.com/seata/seata/pull/1235)] support to delete undolog in back task use seata codec
- [[#1323](https://github.com/seata/seata/pull/1323)] support database driver class configuration item


### bugfix：
- [[#1456](https://github.com/seata/seata/pull/1456)] fix xid would be duplicate in cluster mode
- [[#1454](https://github.com/seata/seata/pull/1454)] fix DateCompareUtils can not compare byte array 
- [[#1452](https://github.com/seata/seata/pull/1452)] fix select for update retry get dirty value
- [[#1443](https://github.com/seata/seata/pull/1443)] fix serialize the type of timestamp lost nano value
- [[#1374](https://github.com/seata/seata/pull/1374)] fix store.mode get configuration inconsistent
- [[#1409](https://github.com/seata/seata/pull/1409)] fix map.toString() error
- [[#1344](https://github.com/seata/seata/pull/1344)] fix ByteBuffer allocates a fixed length, which cause BufferOverflowException
- [[#1419](https://github.com/seata/seata/pull/1419)] fix if the connection is autocommit=false will cause fail to delete
- [[#1370](https://github.com/seata/seata/pull/1370)] fix begin failed not release channel and throw exception
- [[#1396](https://github.com/seata/seata/pull/1396)] fix ClassNotFound problem for Nacos config implementation
- [[#1395](https://github.com/seata/seata/pull/1395)] fix check null channel
- [[#1385](https://github.com/seata/seata/pull/1385)] fix get SessionManager error when rollback retry timeout
- [[#1378](https://github.com/seata/seata/pull/1378)] fix clusterAddressMap did not remove the instance after the instance was offline
- [[#1332](https://github.com/seata/seata/pull/1332)] fix nacos script initialization the configuration value contains ’=‘ failed
- [[#1341](https://github.com/seata/seata/pull/1341)] fix multiple operations on the same record in the same local transaction, rollback failed
- [[#1339](https://github.com/seata/seata/pull/1339)] fix when image is EmptyTableRecords, rollback failed
- [[#1314](https://github.com/seata/seata/pull/1314)] fix if don't specify the startup parameters, db mode don't take effect
- [[#1342](https://github.com/seata/seata/pull/1342)] fix ByteBuffer allocate len error
- [[#1333](https://github.com/seata/seata/pull/1333)] fix netty memory leak
- [[#1338](https://github.com/seata/seata/pull/1338)] fix lock is not acquired when multiple branches have cross locks
- [[#1334](https://github.com/seata/seata/pull/1334)]  fix lock key npe bug, when tcc use protobuf
- [[#1313](https://github.com/seata/seata/pull/1313)] fix DefaultFailureHandler check status NPE


### optimize： 
- [[#1474](https://github.com/seata/seata/pull/1474)] optimize data image compare log
- [[#1446](https://github.com/seata/seata/pull/1446)] optimize the server's schedule tasks 
- [[#1448](https://github.com/seata/seata/pull/1448)] refactor executor class remove the duplicate code 
- [[#1408](https://github.com/seata/seata/pull/1408)] change ChannelFactory package in TmRpcClientTest 
- [[#1432](https://github.com/seata/seata/pull/1432)] implement equals and hashcode of the object that is used as the hash key 
- [[#1429](https://github.com/seata/seata/pull/1429)] remove unused imports 
- [[#1426](https://github.com/seata/seata/pull/1426)] fix syntax error 
- [[#1425](https://github.com/seata/seata/pull/1425)] fix typo 
- [[#1356](https://github.com/seata/seata/pull/1356)] optimize sql join 
- [[#1416](https://github.com/seata/seata/pull/1416)] optimize some javadoc comments
- [[#1417](https://github.com/seata/seata/pull/1417)] optimize oracle keyword
- [[#1404](https://github.com/seata/seata/pull/1404)] optimize BranchStatus comments
- [[#1414](https://github.com/seata/seata/pull/1414)] optimize mysql keywords
- [[#1407](https://github.com/seata/seata/pull/1407)] disable unstable unit tests
- [[#1398](https://github.com/seata/seata/pull/1398)] optimize eureka registry serviceUrl with default port
- [[#1364](https://github.com/seata/seata/pull/1364)] optimize table columns name defined as constants 
- [[#1389](https://github.com/seata/seata/pull/1389)] add the oracle support prompt information
- [[#1375](https://github.com/seata/seata/pull/1375)] add compareRows failed log
- [[#1358](https://github.com/seata/seata/pull/1358)] clean temporary file file runs when UT is finished
- [[#1355](https://github.com/seata/seata/pull/1355)] add test case for rpc protocol
- [[#1357](https://github.com/seata/seata/pull/1357)] code clean of Consul&Etcd config center implementations
- [[#1345](https://github.com/seata/seata/pull/1345)] code clean and modify log level
- [[#1329](https://github.com/seata/seata/pull/1329)] add `STORE_FILE_DIR` default value


Thanks to these contributors for their code commits. Please report an unintended omission.  

- [slievrly](https://github.com/slievrly)
- [Justice-love](https://github.com/Justice-love)
- [l81893521](https://github.com/l81893521)
- [ggndnn](https://github.com/ggndnn)
- [zjinlei](https://github.com/zjinlei)
- [andyqian](https://github.com/andyqian)
- [cmonkey](https://github.com/cmonkey)
- [wangjin](https://github.com/wangjin)
- [Arlmls](https://github.com/Arlmls)
- [lukairui](https://github.com/lukairui)
- [kongwang](https://github.com/kongwang)
- [lightClouds917](https://github.com/lightClouds917)
- [xingfudeshi](https://github.com/xingfudeshi)
- [alicexiaoshi](https://github.com/alicexiaoshi)
- [itxingqing](https://github.com/itxingqing)
- [wanghuizuo](https://github.com/wanghuizuo)
- [15168326318](https://github.com/15168326318)
- [github-ygy](https://github.com/github-ygy)
- [ujjboy](https://github.com/ujjboy)
- [leizhiyuan](https://github.com/leizhiyuan)
- [vikenlove](https://github.com/vikenlove)

Also, we receive many valuable issues, questions and advices from our community. Thanks for you all.

### Link
- **Seata:** https://github.com/seata/seata  
- **Seata-Samples:** https://github.com/seata/seata-samples   
- **Release:** https://github.com/seata/seata/releases
