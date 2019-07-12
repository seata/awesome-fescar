
Seata 0.7.0 发布

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

本次更新主要内容如下：


## 功能特性

- [[#1276](https://github.com/seata/seata/pull/1276)] 新的 RPC 通信协议
- [[#1266](https://github.com/seata/seata/pull/1266)] metrics 可配置 ([97](https://github.com/seata/seata/issues/97))
- [[#1236](https://github.com/seata/seata/pull/1236)] tc server 支持metrics
- [[#1214](https://github.com/seata/seata/pull/1214)] 添加 `shutdown.wait` ([1212](https://github.com/seata/seata/issues/1212))
- [[#1206](https://github.com/seata/seata/pull/1206)] 可以设置默认值
- [[#1174](https://github.com/seata/seata/pull/1174)] 添加nacos 初始化脚本 ([1172](https://github.com/seata/seata/issues/1172))
- [[#1145](https://github.com/seata/seata/pull/1145)] 修复lock模式和存储模式的关联
- [[#1125](https://github.com/seata/seata/pull/1125)] 支持 protostuff 作为 UndoLogParser 的序列化
- [[#1007](https://github.com/seata/seata/pull/1007)] 支持 Protobuf 作为序列化 ([97](https://github.com/seata/seata/issues/97))

## Bug 修复及优化

- [[#1286](https://github.com/seata/seata/pull/1286)] 排除 log 依赖 ([97](https://github.com/seata/seata/issues/97))
- [[#1278](https://github.com/seata/seata/pull/1278)] 传递 txId 到 TCC 拦截器
- [[#1274](https://github.com/seata/seata/pull/1274)] 优化 SQL join
- [[#1271](https://github.com/seata/seata/pull/1271)] @GlobalLock 修复报错 ([97](https://github.com/seata/seata/issues/97), [1224](https://github.com/seata/seata/issues/1224))
- [[#1270](https://github.com/seata/seata/pull/1270)] 打印异常信息
- [[#1269](https://github.com/seata/seata/pull/1269)] 修复 TMClinet 重连异常
- [[#1265](https://github.com/seata/seata/pull/1265)] 非全局事物，添加 addBatch
- [[#1264](https://github.com/seata/seata/pull/1264)] 更新ci配置 ([97](https://github.com/seata/seata/issues/97))
- [[#1263](https://github.com/seata/seata/pull/1263)] 添加贡献文档 ([97](https://github.com/seata/seata/issues/97))
- [[#1262](https://github.com/seata/seata/pull/1262)] 修复target class的寻找问题 ([97](https://github.com/seata/seata/issues/97))
- [[#1261](https://github.com/seata/seata/pull/1261)] 添加异常信息，当获取自增长的key时 (#1259) ([97](https://github.com/seata/seata/issues/97), [1259](https://github.com/seata/seata/issues/1259))
- [[#1258](https://github.com/seata/seata/pull/1258)] 优化 metrics 模块配置
- [[#1250](https://github.com/seata/seata/pull/1250)] 修复 protobuf 的配置 ([97](https://github.com/seata/seata/issues/97))
- [[#1245](https://github.com/seata/seata/pull/1245)] 重构 metrics
- [[#1242](https://github.com/seata/seata/pull/1242)] sql 优化
- [[#1239](https://github.com/seata/seata/pull/1239)] 修复 CME 在 ZK 服务发现的问题. ([97](https://github.com/seata/seata/issues/97))
- [[#1237](https://github.com/seata/seata/pull/1237)] 修复分支session 可能的 NPE ([97](https://github.com/seata/seata/issues/97))
- [[#1232](https://github.com/seata/seata/pull/1232)] 添加单测 io.seata.common.util CompressUtil, DurationUtil, ReflectionUtil
- [[#1230](https://github.com/seata/seata/pull/1230)] 优化全局🍜扫描器 #1227 ([97](https://github.com/seata/seata/issues/97), [1227](https://github.com/seata/seata/issues/1227))
- [[#1229](https://github.com/seata/seata/pull/1229)] 修复拼写错误 ([97](https://github.com/seata/seata/issues/97))
- [[#1225](https://github.com/seata/seata/pull/1225)] 优化 seata 配置环境信息. ([97](https://github.com/seata/seata/issues/97), [1209](https://github.com/seata/seata/issues/1209))
- [[#1222](https://github.com/seata/seata/pull/1222)] 修复 refresh cluster的bug  ([1160](https://github.com/seata/seata/issues/1160))
- [[#1221](https://github.com/seata/seata/pull/1221)] 修复sql的字段和数据库不一致的问题 ([1217](https://github.com/seata/seata/issues/1217))
- [[#1218](https://github.com/seata/seata/pull/1218)] containsPK 忽略大小写 ([1217](https://github.com/seata/seata/issues/1217))
- [[#1210](https://github.com/seata/seata/pull/1210)] 优化 arrayList 的并发问题
- [[#1207](https://github.com/seata/seata/pull/1207)] @Override 注解强制
- [[#1205](https://github.com/seata/seata/pull/1205)] 移除无用代码
- [[#1202](https://github.com/seata/seata/pull/1202)] 输出 branchRollback 失败日志 ([97](https://github.com/seata/seata/issues/97))
- [[#1200](https://github.com/seata/seata/pull/1200)] 修复 DefaultCoreTest.branchRegisterTest 测试 ([1199](https://github.com/seata/seata/issues/1199))
- [[#1198](https://github.com/seata/seata/pull/1198)] 检查三方依赖的 license ([1197](https://github.com/seata/seata/issues/1197))
- [[#1195](https://github.com/seata/seata/pull/1195)] TCC prepare 阶段晴空 上下文
- [[#1193](https://github.com/seata/seata/pull/1193)] 通过 storemode 关联 lockmode
- [[#1190](https://github.com/seata/seata/pull/1190)] 代码优化 ([97](https://github.com/seata/seata/issues/97), [540](https://github.com/seata/seata/issues/540))
- [[#1179](https://github.com/seata/seata/pull/1179)] jackson 内容存储
- [[#1177](https://github.com/seata/seata/pull/1177)] 修复 TransactionException 异常未能释放锁的问题. ([97](https://github.com/seata/seata/issues/97), [1154](https://github.com/seata/seata/issues/1154))
- [[#1169](https://github.com/seata/seata/pull/1169)] 禁止重复的listener ([1126](https://github.com/seata/seata/issues/1126))
- [[#1165](https://github.com/seata/seata/pull/1165)] 修复 INSERT_UNDO_LOG_SQL 缺失的占位符 ([1164](https://github.com/seata/seata/issues/1164))
- [[#1162](https://github.com/seata/seata/pull/1162)] destroy() 时 重置 initialized flag 和  instance [##1105 ([983](https://github.com/seata/seata/issues/983), [97](https://github.com/seata/seata/issues/97))
- [[#1159](https://github.com/seata/seata/pull/1159)] 修复 AT 模式  resourceId(row_key) 过长的问题 ([97](https://github.com/seata/seata/issues/97), [1158](https://github.com/seata/seata/issues/1158))
- [[#1150](https://github.com/seata/seata/pull/1150)] README.md 中更新seata 的版本 ([97](https://github.com/seata/seata/issues/97))
- [[#1148](https://github.com/seata/seata/pull/1148)] buffer 溢出bug 修复 
- [[#1146](https://github.com/seata/seata/pull/1146)] 修改包名称 ([97](https://github.com/seata/seata/issues/97))
- [[#1105](https://github.com/seata/seata/pull/1105)] 重构 TmRpcClient & RmClient. ([97](https://github.com/seata/seata/issues/97))
- [[#1075](https://github.com/seata/seata/pull/1075)] 多环境隔离
- [[#768](https://github.com/seata/seata/pull/768)] #751 添加事件机制

## 相关链接
- Seata: https://github.com/seata/seata 
- Seata-Samples: https://github.com/seata/seata-samples   
- Release：https://github.com/seata/seata/releases
