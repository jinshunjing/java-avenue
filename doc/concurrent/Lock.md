# Java锁

## Synchronized
- 通过monitorenter/monitorexit指令实现
- 存放在对象头Object Header的Mark Word
- 有4种锁：无锁，偏向锁，轻量级锁，重量级锁
- 偏向锁记录了获取锁的线程ID
- 轻量级锁记录了 栈帧中锁记录的指针
- 重量级锁记录了 互斥量的指针，通过C++ ObjectMonitor 实现

锁升级
https://www.jianshu.com/writer#/notebooks/33672046/notes/40834252

## Monitor 实现原理
- monitorenter：线程获得monitor，count加一，重复进入重复加一。其他线程阻塞等待count变为0
- monitorexit：线程释放monitor, count减一。count为0时，唤醒其他线程

- ObjectMonitor.hpp
- owner: 获得了monitor的线程
- cxq: 竞争队列
- EntryList: cxq中有资格成为候选的线程进入该队列
- WaitSet: 被wait调用阻塞的线程

- 竞争队列ContentionList: 所有等待锁的线程，是一个双向列表，新线程通过CAS操作放入头部，锁的Owner从尾部取线程放入EntryList
- 候选队列EntryList: 
- OnDeck: 只有一个线程在竞争锁
- Owner: 持有锁的线程
- WaitSet: 调用wait的线程

## sun.misc.Unsafe
- 内存操作：allocateMemory/reallocateMemory/freeMemory

- 字段在主存的偏移：staticFieldOffset/objectFieldOffset
- 存取字段的值：getInt/putInt
- 相等才替换：compareAndSwapXX 

- park: 阻塞当前线程
- unpark: 唤醒某个线程

- monitorEnter 锁住对象
- monitorExit 解锁对象


## 锁 Lock

### Lock
- 接口
- 替换synchronized
- 通过AQS框架实现

### ReentantLock 可重入锁
- 可重入锁

### ReadWriteLock 读写锁
- 接口

### StampedLock

### Condition
- 接口
- 替换Object monitor，wait/notify
- 关联一个Lock
- 维护一个等待队列，通常是一个双向链表，存储的是线程对象
- 主要功能是阻塞当前线程与唤醒线程
- 阻塞当前线程：线程加入等待队列，然后执行Unsafe.park
- 唤醒某个线程：线程移出等待队列，然后执行Unsafe.unpark

### LockSupport
- park, unpark


## 同步 Synchronizer

### Semaphore 信号量
- 使用AQS

### CountDownLatch 倒计数锁
- CountDownLatch强调一个线程需要等待N个其他线程。不可重用。
- CyclicBarrier强调多个线程相互等待。可以重用。

### CyclicBarrier 栅栏锁
- 使用ReentrantLock
- 在barrier上等待的线程数目达到设定的数值则执行某个操作，然后唤醒所有的线程
- 适用于map reduce的应用场景

### Phaser

### Exchanger


