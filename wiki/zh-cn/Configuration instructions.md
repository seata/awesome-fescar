
###公共部分

| key         | desc         | remark|
|---------------|--------------|----|
| transport.serialization            | client和server通信编解码方式   |seata、protobuf |
| transport.heartbeat            | client和server通信心跳检测开关   |默认true开启 |


### server端

| key         | desc         | remark|
|---------------|--------------|----|
| transaction.undo.log.save.days            | undo保留天数   |默认7天,log_status=1和未正常清理的undo |
| transaction.undo.log.delete.period          | undo清理线程间隔时间 |默认86400000，单位毫秒    |
| service.max.commit.retry.timeout          | 二阶段提交重试超时时长 | 单位毫秒，默认-1表示无限重试。公式: timeout>=now-globalTransactionBeginTime,true表示超时则不再重试   |
| service.max.rollback.retry.timeout         | 二阶段回滚重试超时时长 |  同commit  |
| recovery.committing-retry-period         | 二阶段提交未完成状态全局事务重试提交线程间隔时间 |默认1000，单位毫秒    |
| recovery.asyn-committing-retry-period         | 二阶段异步提交状态重试提交线程间隔时间 |默认1000，单位毫秒    |
| recovery.rollbacking-retry-period         |二阶段回滚状态重试回滚线程间隔时间 |默认1000，单位毫秒    |
| recovery.timeout-retry-period         | 超时状态检测重试线程间隔时间 |默认1000，单位毫秒，检测出超时将全局事务置入回滚会话管理器    |
| store.mode         |事务会话信息存储方式 |file本地文件(不支持HA)，db数据库(支持HA)    |
| store.file.dir         |file模式文件存储文件夹名 |默认sessionStore    |
| store.db.datasource    |db模式数据源类型 |默认dbcp    |
| store.db.db-type    |db模式数据库类型 |默认mysql    |
| store.db.driver-class-name    |db模式数据库驱动 |默认com.mysql.jdbc.Driver    |
| store.db.url    |db模式数据源库url | 默认jdbc:mysql://127.0.0.1:3306/seata   |
| store.db.user    |db模式数据库账户 |默认mysql    |
| store.db.min-conn    |db模式数据库初始连接数 |默认1    |
| store.db.max-conn    |db模式数据库最大连接数|默认3    |
| store.db.global.table    |db模式全局事务表名 |默认global_table    |
| store.db.branch.table    |db模式分支事务表名 |默认branch_table    |
| store.db.lock-table    |db模式全局锁表名 |默认lock_table    |
| store.db.query-limit    |db模式查询全局事务一次的最大条数 |默认1000    |

###client端

| key      | role     | desc    | Key|
|----------|--------|--------------|----|
| service.vgroup_mapping.my_test_tx_group       |  TM,RM    | 事务群组   |my_test_tx_group为分组，配置项值为TC集群名 |
| service.default.grouplist     |   TM,RM   | TC服务列表 |  仅注册中心为file时使用  |
| service.disableGlobalTransaction    |  TM,RM    | 全局事务开关 |  默认false。false为开启，true为关闭  |
| service.enableDegrade    |  TM    | 降级开关 |  默认false。用途未知  |
| client.async.commit.buffer.limit    |  RM    | 异步提交缓存队列长度 | 默认10000。 二阶段提交成功，RM异步清理undo队列  |
| client.lock.retry.internal    |  RM    | 校验或占用全局锁重试间隔(单位毫秒) |  默认10  |
| client.lock.retry.times    |  RM    | 校验或占用全局锁重试次数 |  默认30  |
| client.report.retry.count    |  TM,RM    | 一阶段结果上报TC重试次数 |  默认5次  |
| client.tm.commit.retry.count    |  TM    | 一阶段全局提交结果上报TC重试次数 |  默认1次，建议大于1  |
| client.tm.rollback.retry.count    |  TM    | 一阶段全局回滚结果上报TC重试次数 |  默认1次，建议大于1  |
| transaction.undo.data.validation    |  RM    | 二阶段回滚镜像校验 |  默认true开启，false关闭 |
| transaction.undo.log.serialization    |  RM    | undo序列化方式 |  默认jackson  |
| transaction.undo.log.table    |  RM    | 自定义undo表名 |  默认undo_log  |
| support.spring.datasource.autoproxy    |  RM    | 数据源自动代理开关 |  默认false关闭  |


##未使用
lock {
  // the lock store mode: local、remote
  mode = "remote"

  local {
    // store locks in user's database
  }

  remote {
    // store locks in the seata's server
  }
}

## metrics settings 未知
metrics {
  enabled = false
  registry-type = "compact"
  //multi exporters use comma divided
  exporter-list = "prometheus"
  exporter-prometheus-port = 9898
}