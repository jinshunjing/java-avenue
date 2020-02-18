
# Hystrix


## 主要功能
- 资源隔离/依赖隔离：线程隔离，信号量隔离
- 降级策略：超时，失败
- 熔断机制

- 实时监控

- 请求缓存
- 请求批处理

## 主要概念
- service dependency 服务依赖
- latency tolerance 延迟
- fault tolerance 容错
- isolating points of access
- fallback options


## 代码
hystrix-core


[hystrix-contrib]
hystrix-javanica: annotation support


## HystrixCommand注解
- groupKey
- commandKey
- threadPoolKey

- fallbackMethod：fallback 方法需要有相同的参数，但是可以扩展一个Throwable参数
- defaultFallback: 

- ignoreExceptions: 传播异常，不触发fallback
- raiseHystrixExceptions: ???

- 超时：会中断当前任务

- commandProperties: 命令属性
- threadPoolProperties: 工作线程池配置属性
- HystrixProperty: name, value

## DefaultProperties注解
- groupKey
- threadPoolKey

- defaultFallback:

- ignoreExceptions:
- raiseHystrixExceptions:

- commandProperties:
- threadPoolProperties:

## HystrixCollapser注解


## Cache注解
- CacheResult: 使用默认cacheKeyMethod
- CacheRemove: 使用默认cacheKeyMethod，指定commandKey
- CacheKey: 指定某个参数为key，也可以是对象的字段
- get-set-get pattern


## 熔断器属性
- circuitBreaker.enabled：熔断器是否打开
- circuitBreaker.requestVolumeThreshold：至少N个请求才开始熔断错误比率计算
- circuitBreaker.errorThresholdPercentage：错误率超过N%后熔断器启动
- circuitBreaker.sleepWindowInMilliseconds：熔断器工作时间，超过改时间，放一个请求进去，如果成功则关闭熔断器，否则继续等待
- 

## 线程池属性
- coreSize： 核心线程数，默认10
- maximumSize: 最大线程数，默认10
- maxQueueSize：LinkedBlockingQueue的最大队列，默认-1使用SynchronousQueue
- queueSizeRejectionThreshold：默认5，队列的线程数大于这个配置则开始拒绝
- keepAliveTimeMinutes：默认1分钟，线程的idle时间
- allowMaximumSizeToDivergeFromCoreSize：默认false，设为true才能用maximumSize和keepAliveTimeMinutes


- 不在tomcat的线程池里处理command，在工作线程池里异步处理command
- 一般是一个group一个线程池，也就是一个类一个线程池

## 统计属性
- metrics.rollingStats.numBuckets：滑动统计的桶数量，默认为10
- metrics.rollingStats.timeInMilliseconds：滑动窗口的统计时间


## 主要流程
- Command#execute, #queue
- Circuit breaker是否打开
- 线程池/信号量/队列是否已经满了
- Command#run
- 是否超时
- 是否成功
- 包括运行状态（成功，失败，拒绝，超时）
- fallback 降级处理

## 容错框架
- 线程池隔离
- 信号量隔离
- 降级策略
- 熔断机制

## 命令模式
- 把业务请求封装成命令请求


## 资源隔离模式
- 默认采用线程池隔离机制
- 信号量隔离机制

### 线程池隔离机制
- 每个command对应一个线程池
- #execute 同步调用
- #queue 异步调用

## 降级策略
- fallback
- 超时降级

## 熔断机制
- circutBreaker
- 统计失败率
- 熔断开启后自动重试

## 请求缓存
- 直接从缓存取结果

## 请求批处理
- 多个请求批量处理
