# Kafka

## 工作原理
- 角色：broker, producer, consumer
- Producer发送消息到broker，Consumer从broker拉取消息
- Producer按照规则把消息发送给某个partition
- Consumer按照规则从某个或者多个partition拉取消息

## 消息管理
- topic，partition，consumer group
- 一个topic可以配置多个partition
- 一个partition有多个副本（副本因子），其中一个是leader，其余是follower/replica
- 一个partition可以被多个consumer group消费，但是同一个consumer group只有一个consumer能消费

### partition与broker
- 每个副本，分区0随机分配一个broker，后续分区顺序分配broker
- partition的数量大于broker的数量，可以提高吞吐率
- partition的replica尽量分散到不同的broker，高可用
- leader的选举：维护ISR(in-sync replica)，从ISR中选择一个副本作为leader

### partition与consumer
- partition数量大于consumer数量，每个consumer会处理多个partition
- partition数量小于consumer数量，每个consumer最多只处理一个partition,并且会有空闲consumer
- 一个partition只能被一个consumer处理
- 一个consumer可以处理多个partition

## 连接
### consumer与broker
- 建立3个TCP连接
- 首次请求元数据时创建第一个Socket连接
- 连接任意broker去发现coordinator，创建第二个Socket连接，之后所有的 组协调请求都走该连接
- 发送FETCH请求时创建第三个Socket连接，第一个Socket连接会被废弃


## 管理者
- Broker Controller：broker中有一个是控制器，负责分区管理和副本管理，也执行重分配。通过ZK的临时节点来选举controller
- Consumer Leader: 同一个组中有一个consumer是leader

## 协调器
- GroupCoordinator/ConsumerCoordinator: 消费组协调器，consumer与之通信，metadata数据，提交的位移(offset)写入特殊的topic (comsumer_offsets)
- TransactionCoordinator: 事务协调器，producer与之通信, 特殊的topic (transaction_state)


## Zookeeper的使用
- 问题：ZK不适合大批量的写操作
- broker, consumer
- partition与broker的配置存储在ZK
- partition与consumer的配置存储在ZK
- producer和consumer连接broker(列表), broker连接ZK

## Broker在ZK上的配置
- /brokers/ids/{0}: "host":"","port":9092，每个broker
- /brokers/topics/{topic}: "partitions":{"0":[0,1,2]}，每个topic
- /brokers/topics/{topic}/partitions/{0}/state: "leader":0, "isr":[0,1,2]，每个partition

## Consumer在ZK上的配置
- /consumers/{group}/ids/{consumer id}{进程ID}: "subscribtion":{"{topic}":{partition}}，每个consumer订阅的partition 
- /consumers/{group}/owners/{topic}/{partition}: consumer id + 进程ID，每个partition的consumer 
- /consumers/{group}/offsets/{topic}/{partition}: offset，每个partition的offset

## Spring使用Kafka
- yml里配置bootstrap-servers，连接多个broker，避免单点故障
- producer和consumer只连接broker
- broker配置ZK节点列表
- producer和consumer会记住要连接的broker，如果不是目标broker，会自动路由到目标broker
- producer先连接任意一个broker，broker查询ZK，转发到partition的leader。维护多个broker的信息
- consumer先连接任意一个broker，broker查询ZK，到partition的leader拉取消息

## producer发送消息
- 消息发送分异步和同步两种方式
- 异步时，先在客户端缓存然后批量发送
- 通过配置acks来确定消息送达
- acks=0不确认
- acks=1表示leader接收成功就确认
- acks=all表示要求所有的副本都接收成功才确认


## consumer拉取消息
- 轮询
- 可以阻塞
- fetch.min.bytes/fetch.max.bytes
- fetch.max.wait.ms

### Consumer Polling
- 通过poll()的心跳机制来维持连接
- max.poll.interval.ms: 超过该时间不调用poll，broker会认为consumer已经挂了，然后触发rebalance
- max.poll.records：限制每次poll返回的消息数目，太大可能会导致处理时间过长，导致超时触发rebalance

### Consumer Offset
- 提交给特殊的topic(_consumers_offsets)
- enable.auto.commit: 自动或者手动提交offset
- auto.commit.interval.ms
- consumer.commitSync()：手动提交offset。使用场景：需要累积到一定量的消息之后，才批处理。批处理成功之后才commit
- 手动提交可能会失败，导致重复消费消息。所以consumer要幂等


## Consumer Partition
- 手动配置partition: consumer.assign()

### Consumer rebalance
参考文章：https://www.cnblogs.com/yoke/p/11405397.html
- 动态维持consumer group内consumer与partition的关联
- 两种分配策略：range 和 round-robin
- 由consumer group和coordinator共同完成rebalance
- coordinator的确定：1. 确认consumer group的位移写入主题_consumers_offsets的哪个分区；2. 该分区的leader就是coordinator
- rebalance第1步 Join: 所有consumer向coordinator发送JoinGroup请求；coordinator选择一个consumer担任leader；coordinator把组员信息发送给leader；
- rebalance第2步 Sync：leader完成分配方案，发送SyncGroup请求给coordinator；其他consumer请求SyncGroup时，coordinator把分配方案返回 


## RocketMQ延迟消息
- 类似hash：预定义了18个level，每个level的延迟时间相同，有自己的定时器，所以可以顺序处理，避免了排序
- 用优先队列，每次取根，判断是否投递

## 持久化日志
* 日志目录：如果通过log.dirs配置了多个日志目录，新的分区目录会写到分区目录数量最少的那个日志目录下，分区目录格式为{topic-分区ID}
* 日志文件：topic/partition/segment ({偏移量}.log, .index, .timeindex，文件大小为1G)
* 索引文件：偏移量索引文件.index，时间戳索引文件.timeindex
* 利用磁盘的顺序读写性能

## 事务
- kafka EOS 精确一次
- producer 幂等性：通过producer id, sequence, acks = -1
- consume-transfer-produce: 消费，offset 提交，生产的原子性，producer.initTx, beginTx, sendOffsets, commitTx, abortTx

## 延时操作
- acks = -1， 延时生产，超时报错
- 延时fetch，超时返回

## 重新分区和再平衡
- 重新分配分区：reassign_partitions
- 先通过控制器为每个分区添加新副本，新的副本从leader那里复制数据，完成之后控制器将旧副本移除
- 优先副本选举：prefferred_replica_elction，把优先副本选举成leader
- 定时任务计算分区不平衡率，超过阈值则触发优先副本的选举
- 生产者选择分区：Partitioner
- 消费者分配分区策略：PartitionAssignor
- 消费者分区再平衡：GroupCoordinator, find/join/sync/heartbeat；join之后返回多次消费者选择出的分配策略，由leader计算分配方案；通过sync通知给所有的消费者

