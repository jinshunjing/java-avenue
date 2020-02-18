# MySQL

## InnoDB vs MyISAM
- InnoDB是聚簇索引（叶子节点存数据），MyISAM是非聚簇索引（叶子节点存指针）
- MyISAM不支持事务，不支持外键
- MyISAM只支持表锁，不支持行锁
- MyISAM支持全文索引


## MySQL 日志
- bin log

## InnoDB 日志
- redo log
- undo log

## 主从复制
- 主：binlog dump线程 - SQL更新语句记录在binlog
- 从：io线程 - 拉取master的binlog，写入自己的relay log
- 从：SQL执行线程 - 执行relay log里的语句

- 基于SQL语句的复制：binlog小，但是有些语句无法被复制，存储过程，触发器
- 基于行的复制：可靠性高，任何情况都可以复制，binlog大
- 混合复制：两种方式都可以用

## bin log vs redo log
- bin log 是逻辑的，SQL语句或记录行；redo log是物理的，对页的修改
- redo log 操作是幂等性，bin log 不是
- bin log 在事务提交时一次性写入；一个一个事务，顺序写入；
- redo log 可以写多次到 redo log buffer，提交时写入日志；并发写入，不同事务的多个操作会混合写入

- 事务提交时，先写 bin log，再写 redo log
- 将事务放入队列，第一个事务称为leader，其他事务称为follower。分成3个阶段：flush/sync/commit
- fulsh阶段：先写bin log内存，再写redo log内存。flush等待一段时间之后才进入sync
- sync阶段：bin log刷盘，可能刷一个事务也可能刷多个事务
- commit阶段：leader根据顺序提交事务，可以使用innodb的group commit

- redo log 有两部分：内存中的 redo log buffer，磁盘里的 redo log file
- redo log block 大小为 512 字节，不需要 double write
- 先写 redo log buffer（用户空间），再写 os buffer（内核空间），然后 fsync 到磁盘
- innodb_flush_log_at_trx_commit: 0 事务提交时不进行写入操作，1 事务提交时必须调用一次fsync， 2 事务提交时仅写入文件系统的缓存，不进行fsync

- 事务提交 会刷日志文件（bin log + redo log），但不会刷数据文件
- checkpoint 会同时刷日志文件和数据文件，使其有相同的 LSN (Log Sequence Number)
- 主线程 每秒会刷日志文件(redo log)

## undo log
- 不是物理日志，是逻辑日志，存放在 共享表空间 的 undo段 (undo segment)
- 共享表空间中有N个回滚段 rollback segment, 有 1024个 undo log segment
- undo log 里存放的是记录，不是SQL语句
- 逻辑的回滚到原来的状态，但是物理页已经修改了
- insert undo log 在事务提交后直接删除，不需要进行purge操作
- update undo log 提交时放入undo log链表，等待purge线程进行最后的删除


## 提升性能的关键特性
- 插入缓冲
- 两次写
- 自适应哈希索引

### 插入缓冲 insert buffer
- 解决非聚簇索引的随机插入的性能问题，非唯一索引
- 先判断索引页是否在缓存中
- 如果不在则放入插入缓冲区
- 每隔一段时间执行插入缓冲区和非聚簇索引页的合并操作
- Insert Buffer 是一棵B+树，存放在共享表空间

### 两次写 double write
- 提高写的安全性，解决数据丢失（写失效）问题：16K的页只写了4K
- 两个部分：内存里 double write buffer （2MB），磁盘上 共享表空间 （连续 128页 2MB）
- 先把脏页拷贝到内存的doublewrite buffer
- 分两次，每次写1MB到共享表空间，然后同步磁盘。此时是顺序写
- 然后再写入各个 表空间，此时是随机写
- 恢复过程，从 共享表空间 找到副页，然后写入 表空间

### 自适应哈希索引 Adaptive Hash Index
- 只能用于等值的查询
- 根据数据的访问频率和模式，热点数据
- 通过缓冲池的B+树来创建自适应哈希索引，不用查磁盘上的索引页，所以很快


## 索引
- 索引页：16K，留出1/16空闲页用于update/insert
- 聚集索引按照主键顺序插入
- 二级索引随机插入：先检查索引页是否在内存中，如果不在内存中则插入insert buffer，每隔一段时间合并属于同一个索引页
- 自适应哈希索引：为经常被访问的索引页建立自适应哈希索引


## MySQL事务

### 事务的ACID属性
- Atomicity原子性：全部执行或者全部不执行
- Consistency一致性：开始和完成时，数据保持一致性
- Isolation隔离性：独立执行
- Durability持久性：数据的修改是永久性的

- 事务的ACID是通过日志和锁来保证
- 隔离性是通过锁机制来实现
- 持久性通过redo log来实现
- 原子性和一致性通过undo log来实现

- InnoDB通过 Force Log At Commit 来实现 持久性：commit时，必须将事务的所有日志写入redo log

- 事务回滚，通过undo log，insert 对应 delete， update 对应反向的 update来实现原子性
- MVCC的实现就是靠undo log，通过undo读取之前的行版本信息: RR隔离级别下，总是读事务开始的行数据；RC隔离级别下，总是读最新的快照

### 并发事务处理的问题
- 脏读：一个事务查询了另一个事务未提交的数据更新
- 不可重复读：一个事务重新查询，发现了另一个事务更新的数据
- 幻读：一个事务重新查询，发现了另一个事务插入的数据
- 更新丢失：一个事务覆盖了另一个事务的数据更新

### 事务隔离级别（读数据一致性）：
- 读未提交 read uncommited：脏读，不可重复读，幻读
- 读已提交 read commited：不可重复读，幻读
- 可重复读 repeatable read：幻读
- 串行化 serializable

### MySQL MVCC 多版本并发控制
- 记录增加两个隐藏列，创建事务版本号，删除事务版本号。
- 更新的时候删除旧记录，创建新记录。
- 查询的时候需满足：
  - 创建版本号小于等于事务版本号
  - 删除版本号大于事务版本号

### Spring事务的传播行为

|传播行为|说明|
|------|------|
|PROPAGATION_REQUIRED|默认值。如果没有则新建事务，如果有则加入当前事务|
|PROPAGATION_REQUIRES_NEW|如果没有则新建事务，如果有则挂起当前事务|
|PROPAGATION_NESTED|如果没有则新建事务，如果有则新建当前事务的子事务|

|PROPAGATION_SUPPORTS|如果没有则非事务，如果有则加入当前事务|
|PROPAGATION_NOT_SUPPORTED|如果没有则非事务，如果有则挂起当前事务|
|PROPAGATION_NEVER|如果没有则非事务，如果有则抛出异常|

|PROPAGATION_MANDATORY|如果没有则抛出异常，如果有则加入当前事务|


## InnoDB锁机制

### 乐观锁
读取数据的时候不加锁，更新数据的时候会判断数据是否被修改。
一般通过版本号或CAS实现。

### 悲观锁
读取数据的时候会加锁。
表锁，行锁，共享锁，排他锁，都是悲观锁

### 表锁
- 意向共享锁IS：事务给一个数据行加共享锁之前必须先取得该表的意向共享锁
- 意向排他锁IX：事务给一个数据行加排他锁之前必须先取得该表的意向排他锁

### 行锁 Row Lock
* 共享锁S：允许事务去读一行，阻止其他事务获取该数据集的排他锁
* 排他锁X：允许事务更新数据，阻止其他事务获取该数据集的共享读锁与排他写锁

> 要点：行锁通过给索引项加锁实现，而不是给记录加锁。只有通过索引检索才使用行锁，否则使用表锁。

INSERT,UPDATE,DELETE语句自动加排他锁。

SELECT语句需要手动加锁：
- 共享锁：SELECT ... LOCK IN SHARE MODE
- 排他锁：SELECT ... FOR UPDATE

### 间隙锁 Gap Lock
- 使用范围条件检索，不存在的记录也会被加锁
- 使用相等条件检索也会给不存在的记录加锁
- 能够将左右两边不存在的记录加锁：SELECT MAX(...) ... FOR UPDATE

> 要点1：唯一索引只有行锁，非唯一索引才有间隙锁
> 要点2：间隙锁锁住了左边和右边不存在的记录，比如{2, 4, 6}，如果查询条件是4，那么间隙锁锁住的是左边[3, 4)，右边[5，6）

### Next-Key Lock
- 非唯一索引，包含行锁和间隙锁，用于防止幻读
- 唯一索引，降级为行锁

InnoDB会加Netx-Key Lock，包括行锁(Record Lock)和间隙锁(Gap Lock)。间隙锁会锁住后面没有的记录，可以用来解决幻读的问题。
比如事务A一开始使用select ... for update读出3条记录，此时由于间隙锁的存在，事务B将不能插入ID为4的记录，所以就不存在幻读的问题。




