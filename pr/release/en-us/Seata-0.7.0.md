
Seata 0.7.0 Release

🔥 Seata is an easy-to-use, high-performance, open source distributed transaction solution.


What the current update is：


## Feature

- [[#1276](https://github.com/seata/seata/pull/1276)] New RPC protocol
- [[#1266](https://github.com/seata/seata/pull/1266)] add enabled configuration for metrics ([97](https://github.com/seata/seata/issues/97))
- [[#1236](https://github.com/seata/seata/pull/1236)] support metrics for tc server
- [[#1214](https://github.com/seata/seata/pull/1214)] add config `shutdown.wait` and update version to 0.7.0-SNAPSHOT ([1212](https://github.com/seata/seata/issues/1212))
- [[#1206](https://github.com/seata/seata/pull/1206)] setting default values using trinomial operators
- [[#1174](https://github.com/seata/seata/pull/1174)] add nacos config initialization python script ([1172](https://github.com/seata/seata/issues/1172))
- [[#1145](https://github.com/seata/seata/pull/1145)] Change LockMode from MEMORY to DB when the StoreMode is DB
- [[#1125](https://github.com/seata/seata/pull/1125)] Add protostuff as serializer of UndoLogParser.
- [[#1007](https://github.com/seata/seata/pull/1007)] support protobuf feature ([97](https://github.com/seata/seata/issues/97))


## Bugfix & Optimize

- [[#1286](https://github.com/seata/seata/pull/1286)] bugfix: add some configuration and exclude log dependency ([97](https://github.com/seata/seata/issues/97))
- [[#1278](https://github.com/seata/seata/pull/1278)] bugfix: pass txId into TCC interceptor
- [[#1274](https://github.com/seata/seata/pull/1274)] 1. optimization SQL join
- [[#1271](https://github.com/seata/seata/pull/1271)] bugfix: @GlobalLock get error with Response ([97](https://github.com/seata/seata/issues/97), [1224](https://github.com/seata/seata/issues/1224))
- [[#1270](https://github.com/seata/seata/pull/1270)] bugfix: print error exception
- [[#1269](https://github.com/seata/seata/pull/1269)] bugfix: fix TMClinet reconnect exception
- [[#1265](https://github.com/seata/seata/pull/1265)] Invoke addBatch of targetStatement if not in global transaction
- [[#1264](https://github.com/seata/seata/pull/1264)] configuration:update ignore and coverage ([97](https://github.com/seata/seata/issues/97))
- [[#1263](https://github.com/seata/seata/pull/1263)] docs: add doc about contribution ([97](https://github.com/seata/seata/issues/97))
- [[#1262](https://github.com/seata/seata/pull/1262)] bugfix: fix find target class issue if scan the web scope bean such a… ([97](https://github.com/seata/seata/issues/97))
- [[#1261](https://github.com/seata/seata/pull/1261)] add warn log when fail to get auto-generated keys. (#1259) ([97](https://github.com/seata/seata/issues/97), [1259](https://github.com/seata/seata/issues/1259))
- [[#1258](https://github.com/seata/seata/pull/1258)] move metrics config keys and simplify metrics modules dependency
- [[#1250](https://github.com/seata/seata/pull/1250)] fix codecov for protobuf ([97](https://github.com/seata/seata/issues/97))
- [[#1245](https://github.com/seata/seata/pull/1245)] refactor metrics let it initialize by configuration
- [[#1242](https://github.com/seata/seata/pull/1242)] perfect sql
- [[#1239](https://github.com/seata/seata/pull/1239)] bugfix:fix CME in ZK discovery implementation. ([97](https://github.com/seata/seata/issues/97))
- [[#1237](https://github.com/seata/seata/pull/1237)] bugfix:server start  and handle remain branch session may cause NPE ([97](https://github.com/seata/seata/issues/97))
- [[#1232](https://github.com/seata/seata/pull/1232)] Add unit tests for io.seata.common.util CompressUtil, DurationUtil, ReflectionUtil
- [[#1230](https://github.com/seata/seata/pull/1230)] prioritize global transaction scanner #1227 ([97](https://github.com/seata/seata/issues/97), [1227](https://github.com/seata/seata/issues/1227))
- [[#1229](https://github.com/seata/seata/pull/1229)] fix a typo ([97](https://github.com/seata/seata/issues/97))
- [[#1225](https://github.com/seata/seata/pull/1225)] optimize the name of seata config environment. ([97](https://github.com/seata/seata/issues/97), [1209](https://github.com/seata/seata/issues/1209))
- [[#1222](https://github.com/seata/seata/pull/1222)] fix bug of refresh cluster ([1160](https://github.com/seata/seata/issues/1160))
- [[#1221](https://github.com/seata/seata/pull/1221)] bugfix: fix in which SQL and database field names are inconsistent#1217 ([1217](https://github.com/seata/seata/issues/1217))
- [[#1218](https://github.com/seata/seata/pull/1218)] bugfix:containsPK ignoreCase ([1217](https://github.com/seata/seata/issues/1217))
- [[#1210](https://github.com/seata/seata/pull/1210)] 1. optimize arrayList single value
- [[#1207](https://github.com/seata/seata/pull/1207)] All overriding methods must be preceded by @Override annotations.
- [[#1205](https://github.com/seata/seata/pull/1205)] remove useless code
- [[#1202](https://github.com/seata/seata/pull/1202)] output branchRollback failed log ([97](https://github.com/seata/seata/issues/97))
- [[#1200](https://github.com/seata/seata/pull/1200)] bugfix:DefaultCoreTest.branchRegisterTest ([1199](https://github.com/seata/seata/issues/1199))
- [[#1198](https://github.com/seata/seata/pull/1198)] check the third-party dependencies license ([1197](https://github.com/seata/seata/issues/1197))
- [[#1195](https://github.com/seata/seata/pull/1195)] Clear the transaction context in TCC prepare methed
- [[#1193](https://github.com/seata/seata/pull/1193)] Get lockmode by the storemode
- [[#1190](https://github.com/seata/seata/pull/1190)] remove unused semicolons ([97](https://github.com/seata/seata/issues/97), [540](https://github.com/seata/seata/issues/540))
- [[#1179](https://github.com/seata/seata/pull/1179)] fix jackson default content
- [[#1177](https://github.com/seata/seata/pull/1177)] write session may be failed，throw TransactionException but hold lock. ([97](https://github.com/seata/seata/issues/97), [1154](https://github.com/seata/seata/issues/1154))
- [[#1169](https://github.com/seata/seata/pull/1169)] bugfix: use Set to avoid duplicate listeners. fixes #1126 ([1126](https://github.com/seata/seata/issues/1126))
- [[#1165](https://github.com/seata/seata/pull/1165)] add a missing placeholder in INSERT_UNDO_LOG_SQL ([1164](https://github.com/seata/seata/issues/1164))
- [[#1162](https://github.com/seata/seata/pull/1162)] Reset initialized flag & instance while destroy(). split [##1105 ([983](https://github.com/seata/seata/issues/983), [97](https://github.com/seata/seata/issues/97))
- [[#1159](https://github.com/seata/seata/pull/1159)] bugfix: AT mode resourceId(row_key) too long ([97](https://github.com/seata/seata/issues/97), [1158](https://github.com/seata/seata/issues/1158))
- [[#1150](https://github.com/seata/seata/pull/1150)] updates seata's version in README.md ([97](https://github.com/seata/seata/issues/97))
- [[#1148](https://github.com/seata/seata/pull/1148)] bugfix:the buffer may cause overflows when sql statement is long
- [[#1146](https://github.com/seata/seata/pull/1146)] revise the package name of the module ([97](https://github.com/seata/seata/issues/97))
- [[#1105](https://github.com/seata/seata/pull/1105)] refactor TmRpcClient & RmClient for common use. ([97](https://github.com/seata/seata/issues/97))
- [[#1075](https://github.com/seata/seata/pull/1075)] Multiple environmental isolation
- [[#768](https://github.com/seata/seata/pull/768)] #751 add event bus mechanism and apply it in tc


## Link
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
