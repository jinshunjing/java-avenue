# AQS框架

## AbstractQueuedSynchronizer
- state: 同步状态
- 两种资源共享方式：exclusive/shared
- 同步队列 sync queue
- 条件/等待队列 condition queue
- 获取锁/释放锁：tryAcquire/tryRelease
- Node等待状态：已取消，唤醒后继，传播


### Node 节点
- thread: 阻塞的线程
- waitStatus：等待状态，0, signal, cancelled, propagate, condition
- prev/next：同步队列，双向链表
- nextWaiter：条件等待队列，单向链表

### 同步队列 sync queue
- CLH 队列：自旋，CAS原子操作
- head/tail: 队首/队尾节点
- 独占式：acquire, release
- FIFO，新节点添加到队尾 addWaiter/enq
- 新节点加入队尾后，执行自旋 acquireQueued
- 当前序节点不是队首节点时，只有在前序节点的等待状态为signal时，当前节点才会被阻塞 shouldParkAfterFailedAcquire/parkAndCheckInterrupt
- 如果前序节点是队首节点，则再次尝试获取锁资源 tryAcquire，如果获取成功，则把当前节点设置为队首节点 setHead，退出自旋
- 队首节点获得了锁资源
- 释放队首节点时 release，唤醒后继节点 unparkSuccessor
- 节点被唤醒之后，会继续执行自旋


### ConditionObject
- firstWaiter/lastWaiter：等待队列
- signal/signalAll: 条件已经达成，通知
- await: 等待条件

### Unsafe
- objectFieldOffset: 字段的内存位置
- compareAndSetXX：CAS操作

### LockSupport
- unpark: 唤醒线程
- park: 阻塞线程

### Basic
- head/tail: 阻塞队列
- state: 同步状态
- enq/addWaiter: 以自旋的方式入队列



## ReentrantLock
- tryLock/unlock：锁的获取与释放
- getQueueThreads: 等待锁
- getWaitingThreads：等待条件
- Sync, FairSync, NonfairSync: AQS实现类

## ReentrantReadWriteLock
- ReadLock/WriteLock:
- Sync, FairSync, NonfairSync: AQS实现类

## CyclicBarrier
- 强调线程相互等待
- 可以重复使用

## CountDownLatch
- 强调一个线程等待其他N个线程完成
- 不可以重复使用

## Mutx
- 锁

- CyclickBarrier: 使用ReentrantLock，await条件，等到N个线程await之后一起trip。每个工作线程里await，等待N个工作线程完成工作之后执行一个回调。可以重复使用。强调一个线程需要等待另外N个线程。
- CountDownLatch: 使用AQS实现，countDown状态减1, await状态为0。每个工作线程里countDown，在最外层await N个工作线程完成工作。一次性。强调多个线程相互等待。
- Semaphore: 使用AQS实现



