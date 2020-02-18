
## 角色
- leader
- follower 可读可写
- observer 只读，写转发给leader
- client

## Zookeeper集群
- 一个leader,多个follower
- server通过host而非ip， host与ip的绑定关系在/etc/hosts，特别注意本机的ip需要写出0.0.0.0，否则监听不了
- 在dataDir里添加myid文件，从1开始
- 按顺序启动，通过zkServer.sh status来查看状态

## server工作状态
- LOOKING 不知道谁是leader
- LEADING 本机是leader
- FOLLOWING 本机是follower
- OBSERVING 本机是observer

## zxid
- 64位，高32位是epoch，低32位自增counter
- epoch代表leader

## 选主流程 （basic paxos）
- myid/sid: 机器ID
- zxid: 事务ID
- electionEpoch: 选举周期
- 发送投票(myid, zxid)给所有的server
- 接收到的投票与自己的投票PK，选举轮次相同，如果zxid更大，或者myid更大，则重新投票
- 统计某个投票超过半数同意，则选出leader

## 同步流程
- follower发送zxid给leader
- leader确定同步点
- 完成同步后，通知follower已经成为uptodate状态
- follower可以接收client请求

## 原子广播 （ZAB协议 数据一致性）
- 在Paxos算法上做了改进
- leader处理client的事务请求，采用ZAB广播到follower
- 恢复模式（选举），广播模式（数据同步）
- leader计算zxid并发送事务proposal
- leader激活：该阶段，leader建立并准备发送proposal
- 消息激活：该阶段，leader协调消息传输
- 第一阶段：leader广播事务proposal给所有的follower, follower顺序ack；当leader接收到超过法定数量到ack,进入第二阶段
- 第二阶段：leader顺序commit事务给follower，follower接收到之后即可以发送给client
- 顺序一致性
- 原子性
- 单一系统映象：新加入的follower只有在跟上进度后才可以接受client的读操作
- 类似与两阶段提交，强一致

## session
- 状态：connecting, connected, reconnecting, reconnected, close
- Session四个属性：sessionID, timeOut, tickTime, isClosing
- 管理：分桶策略: (time/expirationInterval + 1) * expirationInterval
- 激活：(1) client发送读写操作都会触发激活请求；(2) client发现在timeOut/3时间内未和server通信，主动发送PING请求
- 清理：发起session关闭请求，收集需要清理的临时节点，删除临时节点，移除session
- 重连：connection_loss, 重连之前的server；session_moved, 重连到另一台server; session_expired，session已经被清理

## watch 机制
- 实现Watcher接口，一次性触发
- implements Watcher, 重载process(WatchedEvent)
- getData(), exists(), getChildren() 可以设置watch
- client向某个节点注册监听事件，当这个节点或者子节点发生变更时，ZK会发送通知给client
- client串行执行，保证了顺序
- 轻量，只通知event，但不提供详情

- client注册Watcher: client向服务器注册Watcher，等待服务器返回，然后把Watcher添加到client的ZKWatchManager
- 服务器处理Watcher: znode有变动时，发送event给client
- client回调Watcher: 找出Watcher，执行回调

- ClientCnxn
- sendThread: 发送命令，解析response，把event放入队列。#readResponse
- eventThread: 读队列，处理WatchedEvent。#queueEvent, #run, #processEvent


## znode类型
- persistent
- persistent_sequential
- ephemeral：一旦client与ZK之间的会话失效，临时节点会自动删除
- ephemeral_sequential：节点序号自动增加


## 应用场景

### 配置管理
- 配置更新了通知所有节点

### 集群管理
- 注册中心：client在ZK上建立临时顺序节点
- 选出master/leader：编号最小的临时节点选举为master

### 分布式锁
- 在节点locks下创建临时顺序节点node_n
- 查询locks下所有的子节点，如果是最小的，则锁获取成功
- 如果不是，则监听比它小的那个节点的删除事件
- 监听到删除事件之后从第二步重复


## 安装
- 官网下载压缩包
- 添加zoo.cfg
- 启动：bin/zkServer.sh start
- 连接：bin/zkCli.sh -server 127.0.0.1:2181


