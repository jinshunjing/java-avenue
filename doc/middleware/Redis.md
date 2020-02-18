
## 持久化
- RDB容易丢失数据，AOF通过合适的fsync策略丢失更少的数据
- AOF体积大，但是可以自动重写
- RDB适合数据备份

### RDB
- 把内存里的key-value快照写到磁盘文件里
- fork一个子进程来生成RDB文件
- RDB文件是二进制文件
- 在一定的时间段内执行一定数量的写操作，会触发BGRDB

### AOF
- 将执行过的写操作记录到AOF文件里
- AOF是一个文本文件
- 先写缓存，然后同步到AOF文件里。同步策略有3种，比如其中一种是每隔1秒同步一次AOF文件
- AOF文件太大了之后会触发AOF重写。重写是根据当时的key-value来重新生成写操作，所以跟已经存在的AOF文件不相关
- AOF重写过程中的写操作记录在缓存，最后会追加到AOF文件里


## 数据库
- 默认16个数据库，编号从0开始。集群里只能使用0号数据库
- 数据库内部是一个dict结构
- __keyspace@{dbid}__:{key} 频道通知key的操作
- __keyevents@{dbid}__:{ops} 频道通知操作的key

## 过期
- expires字典（dict）记录key的过期时间
- 惰性删除策略，操作的时候判断key是否过期，过期则删除
- 定期删除策略，启一个job，随机删除N个过期的key

## LRU
- HashMap + 双向链表，每次访问之后移到链头
- 近似淘汰算法：随机取出若干个key，按照访问时间排序，淘汰最不经常使用的

### 淘汰策略
- noeviction 不淘汰
- volatile-lru 设置了过期时间的key参与LRU
- allkeys-lru 所有的key参与LRU
- volatile-random 随机淘汰设置了过期时间的key
- allkeys-random 随机淘汰所有的key
- volatile-ttl 设置了过期时间的key中存活时间最短的


## 主从复制
- 角色：主节点，从节点
- 从节点第一次连上主节点，发送命令PSYNC，主节点会执行全局复制：生成RDB文件，并发送给从节点（同时会把生成RDB文件过程中的写操作也发送给从节点）
- 同步完成之后，主节点会把写命令传播给从节点（命令传播）
- 从节点断线重连之后，发送命令PSYNC给主节点，主节点会执行部分复制：根据从节点的offset，从replication backlog里读取未同步的命令，发送给从节点
- replication backlog是一个固定大小的先进先出队列，每个位置存储了一个字节
- 从节点定时发送心跳检查给主节点，主节点检查到从节点的offset小于自己的offset，也会把未同步的命令补偿给从节点，整个过程与部分复制类似

## Sentinel
### 建立
- 角色：哨兵，主节点，从节点
- 哨兵和主节点建立命令和订阅链接
- 哨兵每10秒向主节点发送INFO命令，从返回消息里了解主节点，并发现从节点
- 哨兵每10秒向从节点发送INFO命令，更新从节点和主节点的信息
- 哨兵每2秒向主从节点发送消息，包括哨兵和主节点的信息，使其他哨兵发现自己，然后哨兵之间建立命令链接

### 故障转移
- 哨兵每秒向所有节点发送PING命令，返回PONG
- 在一个时间范围内，连续返回无效回复，则标识为主观下线
- 哨兵检测到主节点主观下线后，发送命令SENTINEL is-master-down-by-addr，回复是否主观下线
- 哨兵收集到一定数量的主观下线，则把节点标识为客观下线
- 选举领头哨兵
- 从节点中挑选一个节点，转换为主节点
- 其他从节点改为复制新的主节点
- 已下线的主节点设置为从节点


## 集群
### 建立集群
- 客户端发送CLUSTER MEET 命令把节点B加入节点A的集群
- 节点A发送MEET命令给B，B发送PONG命令给A，A再发送PING给B，握手完成

### 槽指派
- 集群的整个数据库被分成16384个槽slot, 每个节点指派一部分槽
- 客户端发送 CLUSTER AddSlots 命令把槽指派给节点
- 节点发送消息给其他节点，告知自己负责哪些槽

### 执行命令
- 节点计算key使用哪个槽（CRC16），如果是自己则执行命令，如果不是则返回MOVED错误，指引客户端转向正确的节点

### 重新分片
- 向目标节点发送 CLUSTER SETSLOT slot IMPORTING source 命令，让目标节点准备好从源节点导入槽
- 向源节点发送 CLUSTER SETSLOT slot MIGRATING target 命令，让源节点准备好迁移槽
- 向源节点发送 CLUSTER GetKeysInSlot slot count 命令，返回key name列表
- 对每个key，向源节点发送 Migrate target key 命令，迁移
- 向任意节点发送 CLUSTER SetSlot slot NODE target 命令，告知槽已经重新指派

> 在重新分片过程中，如果源节点不存在key，则返回ASK错误
> 客户端发送 ASKING 命令给目标节点，再发送原来的命令；如果不发送ASKING命令，则目标节点会返回Moved错误

### 故障转移
- 通过发送PING消息来检测对方是否在线，如果在规定的时间里没有返回PONG，则标识为 疑似下线PFail
- 如果半数以上的节点都标记为 疑似下线，则标记为 已下线Fail
- 选择一个从节点成为新的主节点

## 数据结构
- string：int, embstr, raw,
- list：双向链表, ziplist, linked list
- hash: 映射表。数据较少时通过ziplist实现，数据较多时通过dict实现
- set: 集合，数据少时intset, 通过hash实现
- zset: 有序集合，ziplist, skiplist

### Sorted Set
- 数据较少时，通过ziplist实现。压缩链表，元素压缩编码
- 数据较多时，通过zset实现，包含一个dict和一个skiplist。dict用来查询数据到score的映射关系，skiplist用来根据score查询数据（支持范围查询）。
- dict: 字典，链表解决碰撞
- skiplist: 
> 为什么不用平衡树而用skiplist? 内存占用，范围操作，实现简单。


## 发布订阅
- 角色：Channel，Publisher，Subscriber

命令：
+ PUBLISH
+ SUBSCRIBE
+ UNSUBSCRIBE
+ PSUBSCRIBE
+ UNPSUBSCRIBE

### 订阅Channel
+ redisServer.pubsub_channels 字典
+ 把client添加到channel，一个channel对应多个client
+ 把channel添加到client

### 订阅模式
+ redisServer.pubsub_patterns 链表
+ 把{client,pattern}添加到链表里

### 发布消息
+ 在字典里找出channel，遍历上面的client列表，发送信息给client
+ 遍历模式列表，如果模式能够匹配channel，发送信息给client



