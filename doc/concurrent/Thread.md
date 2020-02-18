# Java线程

## Thread
### 状态
|状态|说明|
|------|------|
|NEW||
|RUNNABLE|已经start|
|BLOCKED|等待monitor|
|WAITING| 无期限等待|
|TIMED_WAITING|有期限等待|
|TERMINATED|被中断|

### 静态方法
- currentThread()
- holdsLock(Object)
- sleep(long)
- yield()

### 方法
- getId(), getName(), getPriority(), isDaemon()
- getState(), isAlive(), isInterrupted()
- getThreadGroup()
- getUncaughtExceptionHandler()
- start(), run()
- interrupt()
- join(long)


## 异步
### Runnable
- void run()

### Callable<V>
- V call() throws Exception

### Future<V>
- 存储异步执行的返回值

### Executor
- 执行Runnable任务

### ExecutorService
- submit任务
- invoke任务
- shutdown

### ThreadPoolExecutor
构造函数的参数
|参数|说明|
|------|------|
|corePoolSize|核心线程数，一直存活|
|maxPoolSize|最大线程数，任务队列满了之后，还能创建新的线程；线程数达到最大值，则采取拒绝策略|
|blockingQueueCapacity|任务队列容量，当核心线程数达到最大值，任务先放入队列|
|rejectedExecutionHandler|任务拒绝处理器|
|keepAliveTime|线程空闲时间，线程空闲超时，退出线程池|
|allowCoreThreadTimeout|是否运行核心线程退出|

拒绝策略
|策略|说明|
|------|------|
|AbortPolicy|丢弃策略，抛出异常|
|CallerRunsPolicy|调用者运行策略|
|DiscardPolicy|忽略|
|DiscardOldestPolicy|移除最先进入队列的任务|

线程池Executors
- newSingleThreadExecutor(1, 1, LinkedBlockingQueue)
- newFixedThreadPool(n, n, LinkedBlockingQueue)
- newCacheThreadPool(0, max, SynchronousQueue)

### ForkJoinTask<V>
- 类似与线程的实体对象
- 实现Future接口
- RecursiveAction没有返回值，RecursiveTask有返回值
- CountedCompleter
- fork - 异步计算
- join - 等待计算结果
- compute - 递归计算

### ForkJoinPool
- 执行ForkJoinTask的线程池

### CompletionStage (since 1.8)
- 一个CompletionStage完成之后再执行下一个CompletionStage
- 3个重载：同步，异步（使用ForkJoinPool），传入Executor的异步
- 触发的操作：Runnable, Consumer, Function
- exceptionally(Function): 该CompletionStage执行出错
- whenComplete(BiConsumer): 该CompletionStage执行完成，不管出错还是正常
- handle(BiFunction): 该CompletionStage执行完成，不管出错还是正常
- thenRun(Runnable), thenAccept(Consumer), thenApply(Function), thenCompose(Function): 该CompletionStage执行成功
- runAfterEither(Runnable), acceptEither(Consumber), applyToEither(Function): 该CompletionStage与另一个CompletionStage有一个执行成功
- runAfterBoth(Runnable), thenAcceptBoth(BiConsumer), thenCombine(BiFunction): 该CompletionStage与另一个CompletionStage都执行成功
- toCompletableFuture

### CompletableFuture
- runAsync(Runnalbe): 执行任务
- supplyAsync(Supplier): 结果从Supplier中获取
- completedFuture(V): 封装已知结果



