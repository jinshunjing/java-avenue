
## Dubbo 基础
### Dubbo角色
- provider：服务提供方
- consumer：服务消费方
- registry：注册中心
- moniotr：监控中心
- container: 服务运行容器

### 协议
- dubbo: 单一长连接，NIO异步通信
- rmi: 阻塞式短连接

### 负载均衡机制
- random: 随机选择，默认
- round robin: 轮询
- constant hash: 一致性Hash策略，用虚拟节点解决热点节点的问题
- least active: 最不活跃

### 集群容错方案
- failover: 失败自动切换，默认
- failfast: 失败直接报错
- failback: 失败自动恢复，定时重发
- failsafe: 失败安全，忽略异常
- forking: 并行调用多个，有一个成功就行
- broadcast: 广播逐个调用，忽略每个节点的异常

> 读操作采用failover, 默认重试两次
> 写操作采用failfast, 失败就报错

### 当一个接口有多个实现
- 通过group来分组，provider和consumer配置相同的group

### 版本兼容
- 通过version, 多个不同的版本可以注册到注册中心

### 调用方式
- 默认同步
- 支持异步调用，Reference(async = true)

### 服务降级
- 含义：告诉consumer，调用服务时要做哪些动作，不操作provider
- 设置consumer的mock参数
- 向注册中心写入：override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=consumer_app&mock=force:return+null
- force:return+null 消费方调用服务时直接返回null
- fail:return+null 消费方在调用失败后再返回null


## Dubbo 线程模型
### 原理
- Netty提供io线程处理请求，Dubbo提供业务/工作线程池
- SPI接口Dispatcher，提供方法dispatch(ChannelHandler)
- 默认采用AllDispatcher

### AllDispatcher
- 全部放入工作线程池

### DirectDispatcher
- 全部不放入工作线程池，在IO线程上处理

### ConnectionOrderedDispatcher
- Connected/Disconnected放入有序队列（线程池），Request和Response放入工作线程池

### MessageOnlyDispatcher
- 只有Request和Response事件放入工作线程池

### ExecutionDispatcher
- 只有Request事件放入工作线程池

## Dubbo 线程池
### ThreadPool
- 定义SPI接口ThreadPool
- 提供方法getExecutor获取线程池
- 默认采用FixedThreadPool，其他：LimitedThreadPool, CachedThreadPool, EagerThreadPool

### FixedThreadPool
- 线程池中线程的数量固定不变；线程池满了就放队列，队列满了就拒绝；
- 线程数通过参数threads配置，默认200；
- 等待队列大小通过参数queues配置，默认0，使用SynchronousQueue；正数使用固定容量的LinkedBlockingQueue；负数使用无容量限制的LinkedBlockingQueue；
- 拒绝策略是Abort，抛出异常；
- 拒绝策略有4种：Abort, CallerRuns, Discard, DiscardOldest

### LimitedThreadPool
- 线程池中线程的数量只能增加不能减少，但是有一个上限；先放队列，队列满了新建线程；线程池满了就拒绝；
- core线程数通过corethreads配置，默认为0；最大线程数通过threads配置，默认200；
- keepAlive无穷大，线程被创建之后就不会被回收；
- 等待队列与拒绝策略与FixedThreadPool相同；

### CachedThreadPool
- 线程池的容量无穷大，但是空闲线程会被过期回收；先放队列，队列满了新建线程；
- core线程数通过corethreads配置，默认为0；最大线程数通过threads配置，默认无穷大；
- keepAlive通过alive配置，默认1分钟；
- 等待队列与拒绝策略与FixedThreadPool相同；

### EagerThreadPool
- core线程都busy时，新建线程来处理task，而不是放入等待队列；有空闲线程时则放入等待队列，等待线程处理；
- 扩展出EagerThreadPoolExecutor和TaskQueue；
- core线程数通过corethreads配置，默认为0；最大线程数通过threads配置，默认无穷大；
- keepAlive通过alive配置，默认1分钟；
- 等待队列大小通过queues配置，默认为1；

### EagerThreadPoolExecutor
- 重载execute方法；
- 统计已提交的task数量；
- 被拒绝了，尝试直接放入等待队列；

### TaskQueue
- 重载offer方法；
- 如果有空闲线程(已提交的task数量小于线程池中线程的数量)，则把task插入队列，等待空闲线程来处理；
- 如果线程池中线程的数量小于线程池的上限，则task不插入队列，而是新建线程来处理；
- 如果线程池中线程的数量已经达到线程池的上限，则把task插入队列；


## Dubbo SPI
### Dubbo 自适应拓展机制原理与实例
Dubbo的拓展类(Extension)是通过SPI机制加载的：
- 对于某个SPI接口，加载指定目录下(META-INF/dubbo)名称为SPI接口全限名的配置文件
- 对于配置文件里的每一行键值对，加载SPI接口的实现类，也就是拓展类，然后创建实例

有时候我们不希望在Dubbo启动阶段就加载所有的拓展类，而是希望在用到某个拓展类时才加载，这就需要借助于自适应拓展机制。
- 对于某个SPI接口Car，生成一个自适应拓展类Car$Adaptive，并创建实例
- 调用该实例的方法时(SPI接口中包含Adaptive注解)，会加载拓展类并创建拓展实例，然后调用它的方法

### Dubbo Filter 原理
Dubbo的Filter职责链有点绕：
- 在Invoker#invoke方法里调用Filter#invoke
- 在Filer#invoke方法的最后一步调用下一个Invoker的inovke方法
- 如此递归调用
- 在Invoker#invoke的最后一步调用Filter#onResponse
- 如此递归返回

> Filter#invoke：调用的准备工作，需要执行inovker.inovke(invocation)
> Filter#onRespone：调用的结束工作，需要返回result

## Dubbo 反射和代理
### Dubbo Wrapper
Dubbo Wrapper 可以认为是一种反射机制。它既可以读写目标实例的字段，也可以调用目标实例的方法。比如
- Car 是接口；RaceCar 是实现类，实现了 Car；ferrari 和 porsche 是 RaceCar 的实例
- 我们可以为接口 Car 生成一个 Warpper 子类，比如 Wrapper0；然后创建 Wrapper0 的实例 wrapper0
- 可以通过 wrapper0#getPropertyValue 来读取 ferrari 的字段，也可以读取 porsche 的字段
- 可以通过 wrapper0#setPropertyValue 来修改 ferrari 的字段，也可以修改 porsche 的字段
- 可以通过 wrapper0#invokeMethod 来调用 ferrari 的方法，也可以调用 porsche 的方法
- 优点：通过一个 Wrapper 实例就可以操作目标接口的所有实例

### Dubbo Proxy 原理与实例
Dubbo代理机制与JDK的代理机制不同。比如我们有一个接口Car，
- JDK通过 Proxy#newProxyInstance(ClassLoader, Class[], InvocationHandler) 创建Car的一个实例，所有的方法都要通过InvocationHandler

Dubbo则是分成了两个步骤：
- 首先生成了Car的实现类proxy0。这是一个代理类，所有的方法都要通过InvocationHandler
- 然后生成了Proxy的子类Proxy0，并创建了实例。通过newInstance可以创建代理类proxy0的实例。
- 优点：Proxy类就像一个工厂类，可以创建N个接口的不同实例

