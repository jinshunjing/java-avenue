# Netty

https://www.cnblogs.com/lighten/

## 架构
* Channel, ChannelHandler, ChannelHandlerContext, ChannelPipeline
* ChannelFuture, ChannelFutureListener
* EventLoopGroup, EventLoop
* ByteBuf
* ByteToMessageDecoder, MessageToByteEncoder

### 高性能
- 串行无锁化
- volatile，CAS的大量使用
- TCP参数：SO_RCVBUF, SO_SNDBUF，批量发送

### Reactor线程模型

### 粘包的策略
- 定长: FixedLengthFrameDecoder
- 分隔符: LineBasedFrameDecoder, DelimiterBasedFrameDecoder
- 消息头和消息体：消息头保护消息的总长度 LengthFieldBasedFrameDecoder

### 序列化


### 零拷贝
- 堆外内存/直接内存

### 空轮询
- CPU 100%
- 统计空select的次数，在某个周期内连续发生N次，则重建selector，注册原理的SocketChannel

### 流量整形
- GlobalTrafficShapingHandler
- 令牌桶


## Netty多线程

### 异步
- Future: 重写JDK Future，#get
- Promise: 主线程可写，#addListener

- AbstractFuture
- DefaultPromise
- DefaultChannelPromise


### 接口
- EventExecutorGroup：通过next管理多个EventExecutor；通过submit提交一个任务；通过schedule调度一个任务
- EventExecutor：通过inEventLoop检查当前线程是否在EventLoop中
- OrderedEventExecutor：顺序处理任务

- EventLoopGroup：通过next管理多个EventLoop；通过register注册一个Channel
- EventLoop：事件循环

### 实现类
- AbstractEventExecutor：safeExecute执行任务
- AbstractScheduledEventExecutor：维护一个优先队列
- SingleThreadEventExecutor：runAllTasks运行任务
- SingleThreadEventLoop： 
- NioEventLoop：Selector, run, 

### EventLoop 线程功能
- 串行化，在同一个线程里处理，减少多线程竞争锁
- 每一批次处理，可以设置I/O事件和任务处理的比率
- 每一批次处理，如果数据量太大，可以新建任务推迟到下一批次
- 每一批次处理，如果输出缓冲区已经写满，可以设置I/O写事件推迟到下一批次
- 处理I/O事件，如果一次没有处理完，可以设置OPS标记位继续处理
- 处理定时任务
- 处理任务。为了高性能，每次可能处理不了所有的数据，此时可以创建任务在下一次继续处理


## Netty内存管理

Netty的内存分成Pooled和Unpooled。池化的好处是减少内存的创建和销毁，提高性能。
- PooledByteBufAllocator
- UnpooledByteBufAllocator

又分成heap和direct。Direct的好处是可以减少数据在内核态和用户态之间的拷贝。
- PooledHeapByteBuf
- PooledUnsafeHeapByteBuf
- PooledDirectByteBuf
- PooledUnsafeDirectByteBuf

- UnpooledHeapByteBuf
- UnpooledUnsafeHeapByteBuf
- UnpooledDirectByteBuf
- UnpooledUnsafeDirectByteBuf
- UnpooledUnsafeNoCleanerDirectByteBuf

内存区域：减少多线程的锁竞争；减少内存碎片。
- PoolThreadCache: ThreadLocal
- PoolArena: cpu核数
- PoolChunk: 16M，完全二叉树
- PoolSubpage: 8K, 位图。tiny: 16B * 32; small: 512B, 1024B, 2048B, 4096B

为了实现零拷贝衍生出：
- PooledDuplicatedByteBuf
- PooledSlicedByteBuf：分成多个ByteBuf

- CompositeByteBuf：组合多个ByteBuf
- FixedCompositeByteBuf
- WrappedCompositeByteBuf

- WrappedByteBuf：包裹各种byte数组，ByteBuf

内存泄露：
通过计数器来释放内存：retain增加计数，release减少计数，为0则调用deallocate
- AbstractReferenceCountedByteBuf
- ReferenceCountUtil

PooledByteBuf被GC释放掉，但是没有调用release把DirectByteBuf归还给池。
Netty提供了内存泄露检测机制，默认抽样1%的ByteBuf进行跟踪。
命中了就创建一个虚引用PhantomReference，然后创建一个WrappedByteBuf。在GC发生的时候放入ReferenceQueue。

4个泄露检测级别：禁用Disabled，简单Simple，高级Advanced，偏执Paranoid
- SimpleLeakAwareByteBuf
- AdvancedLeakAwareByteBuf
- ResourceLeakDetector
- ResourceLeakTracker




