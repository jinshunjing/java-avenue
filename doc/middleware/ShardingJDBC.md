# Sharding JDBC

## 基础
- 逻辑表，物理表
- 先分库再分表
- DataNode
- ShardingColumn: 分片字段，支持多分片字段
- ShardingAlgorith: 分片算法

- TableRule
- ShardingRule

- SQL解析
- 分库分表路由
- SQL改写
- 结果归并

### 分片策略
- StandardShardingStrategy
- ComplexShardingStrategy
- InlineShardingStrategy: Groovy inline 表达式
- HintShardingStrategy：强制路由
- NoneShardingStrategy

### 分片算法
- PreciseShardingAlgorithm
- RangeShardingAlgorithm
- HintShardingAlgorithm
- ComplexKeyShardingAlgorithm

### 强制路由
- HintManager
- masterRouteOnly: 强制路由到主数据库
- databaseShardingOnly: 
- 强制路由到主库：HintManager#setMasterRouteOnly();
- 强制路由到库：HintManager#addDatabaseShardingValue("t_order", "user_id", 10);
- 强制路由到表：HintManager#addTableShardingValue("t_order", "order_id", 1000);

### 分页查询
- SQL改写limit 0, 10010
- 归并排序
- 落至单分片的查询不需要改写SQL

### KeyGenerator
- DefaultKeyGenerator: Snowflake 
- 41 时间戳，10  worker ID，12 随机数
- 关键是worker ID
- HostNameKeyGenerator
- IPKeyGenerator
- IPSectionKeyGenerator


## 分布式事务
- 柔性事务 SoftTransactionManager
- 最大努力型事务 BestEffortsDelivery BEDSoftTransaction
- TCC型事务 TCCSoftTransaction

### 事务日志存储
- RdbTransactionLogStorage
- MemoryTransactionLogStorage

### 事务监听器
- BestEffortsDeliveryListener
- 执行事件类型：Before_Execute, Execute_Success，Execute_Failure 
- Before_Execute: 写入事务日志
- Execute_Success：删除事务日志
- Execute_Failure：重试N次

### 异步送达JOB
- 事务监听器多次监听失败后，把任务交给异步Job
- BestEffortsDeliveryJob
- 从事务日志里取N条重试




