## Seata 0.8.0 
Seata 0.8.0 正式发布。

Seata 是一款开源的分布式事务解决方案，提供高性能和简单易用的分布式事务服务。

### feature：
- [[#902](https://github.com/seata/seata/pull/902)] 支持 oracle 数据库的 AT 模式
- [[#1447](https://github.com/seata/seata/pull/1447)] 支持 oracle 数据库的批量操作
- [[#1392](https://github.com/seata/seata/pull/1392)] 支持 undo_log 表名可配置 
- [[#1353](https://github.com/seata/seata/pull/1353)] 支持 mysql 数据库的批量更新和删除操作
- [[#1379](https://github.com/seata/seata/pull/1379)] 配置中心所有配置项支持-D参数传入
- [[#1365](https://github.com/seata/seata/pull/1365)] 支持定时更新mysql的表结构，可不停机更改表结构
- [[#1371](https://github.com/seata/seata/pull/1371)] 支持 mysql preparedStatement 自增批量插入
- [[#1337](https://github.com/seata/seata/pull/1337)] 支持 mysql preparedStatement 非自增批量插入
- [[#1235](https://github.com/seata/seata/pull/1453)] 支持兜底定时删除 undolog 使用protobuf codec 
- [[#1235](https://github.com/seata/seata/pull/1235)] 支持兜底定时删除 undolog 使用 seata codec
- [[#1323](https://github.com/seata/seata/pull/1323)] 支持db driver class 可配置


### bugfix：
- [[#1456](https://github.com/seata/seata/pull/1456)] 修复 xid 在 db 模式可重复的问题
- [[#1454](https://github.com/seata/seata/pull/1454)] 修复 DateCompareUtils 不能比对 byte array 的问题
- [[#1452](https://github.com/seata/seata/pull/1452)] 修复 select for update 重试获取到脏数据的问题
- [[#1443](https://github.com/seata/seata/pull/1443)] 修复 timestamp 反序列化丢失纳秒精度的问题
- [[#1374](https://github.com/seata/seata/pull/1374)] 修复 store.mode 启动参数与获取锁配置不一致的问题
- [[#1409](https://github.com/seata/seata/pull/1409)] 修复 map.toString() 错误
- [[#1344](https://github.com/seata/seata/pull/1344)] 修复 ByteBuffer 分配固定长度, 导致 BufferOverflowException 的问题
- [[#1419](https://github.com/seata/seata/pull/1419)] 修复数据库连接默认autocommit=false 无法删除undolog的问题
- [[#1370](https://github.com/seata/seata/pull/1370)] 修复begin事务失败释放channel和继续进行事务的问题
- [[#1396](https://github.com/seata/seata/pull/1396)] 修复 Nacos config SPI 加载 class not found 的问题
- [[#1395](https://github.com/seata/seata/pull/1395)] 修复获取 channel 检测逻辑
- [[#1385](https://github.com/seata/seata/pull/1385)] 在rollback重试时修复获取 SessionManager 错误的问题
- [[#1378](https://github.com/seata/seata/pull/1378)] 修复 eureka注册中心clusterAddressMap 在实例下线列表不清除的问题
- [[#1332](https://github.com/seata/seata/pull/1332)] 修复 nacos 配置初始化脚本初始化含 ’=‘ 配置值错误的问题
- [[#1341](https://github.com/seata/seata/pull/1341)] 修复同一个本地事务中对同一数据反复修改回滚错误的问题
- [[#1339](https://github.com/seata/seata/pull/1339)] 修复数据镜像是 EmptyTableRecords, 回滚失败的问题
- [[#1314](https://github.com/seata/seata/pull/1314)] 修复不指定db模式启动参数，配置文件不生效的问题
- [[#1342](https://github.com/seata/seata/pull/1342)] 修复 ByteBuffer 长度分配错误
- [[#1333](https://github.com/seata/seata/pull/1333)] 修复 netty 内存泄露问题
- [[#1338](https://github.com/seata/seata/pull/1338)] 修复db模式下可重入锁后不再获取其他所的问题
- [[#1334](https://github.com/seata/seata/pull/1334)] 修复使用 protobuf 时 tcc 模式下lock key NPE 的问题
- [[#1313](https://github.com/seata/seata/pull/1313)] 修复 DefaultFailureHandler 检查 status NPE 的问题


### optimize： 
- [[#1474](https://github.com/seata/seata/pull/1474)] 优化数据镜像比对日志
- [[#1446](https://github.com/seata/seata/pull/1446)] 优化了 server 的 schedule tasks 
- [[#1448](https://github.com/seata/seata/pull/1448)] 重构了 executor 类删除了多余的重复代码
- [[#1408](https://github.com/seata/seata/pull/1408)] 更改 TmRpcClientTest 类中的 ChannelFactory package路径
- [[#1432](https://github.com/seata/seata/pull/1432)] 实现了作为 hash key类型对象的equals 和 hashcode 方法 
- [[#1429](https://github.com/seata/seata/pull/1429)] 删除了无用的 imports 
- [[#1426](https://github.com/seata/seata/pull/1426)] 修复语法错误 
- [[#1425](https://github.com/seata/seata/pull/1425)] 修复 typo 
- [[#1356](https://github.com/seata/seata/pull/1356)] 优化 sql 拼接语法 
- [[#1416](https://github.com/seata/seata/pull/1416)] 优化 javadoc 和注释
- [[#1417](https://github.com/seata/seata/pull/1417)] 梳理优化了 oracle 的关键字
- [[#1404](https://github.com/seata/seata/pull/1404)] 优化了 BranchStatus 的注释
- [[#1414](https://github.com/seata/seata/pull/1414)] 梳理优化了 mysql 的关键字
- [[#1407](https://github.com/seata/seata/pull/1407)] 禁用了不稳定的单元测试
- [[#1398](https://github.com/seata/seata/pull/1398)] 优化了 eureka 注册中心 serviceUrl 默认值使用默认端口
- [[#1364](https://github.com/seata/seata/pull/1364)] 优化 table 列字段名称定义为常量 
- [[#1389](https://github.com/seata/seata/pull/1389)] 增加 oracle 支持提示信息
- [[#1375](https://github.com/seata/seata/pull/1375)] 增加 compareRows 比对失败日志
- [[#1358](https://github.com/seata/seata/pull/1358)] 运行完成单测用例时清理临时文件
- [[#1355](https://github.com/seata/seata/pull/1355)] 增加 rpc protocol 的单测
- [[#1357](https://github.com/seata/seata/pull/1357)] 优化 Consul&Etcd 配置中心代码
- [[#1345](https://github.com/seata/seata/pull/1345)] 代码清理和调整日志级别
- [[#1329](https://github.com/seata/seata/pull/1329)] 增加 `STORE_FILE_DIR` 配置项的默认值


非常感谢以下 contributors 的代码贡献。若有无意遗漏，请报告.  

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

同时，我们收到了社区反馈的很多有价值的issue和建议，非常感谢大家。


### 常用链接
- **Seata:** https://github.com/seata/seata  
- **Seata-Samples:** https://github.com/seata/seata-samples   
- **Release:** https://github.com/seata/seata/releases
