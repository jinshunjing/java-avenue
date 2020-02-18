# JVM

## Java内存模型
- 所有变量都存储在主内存中，每条线程有自己的工作内存
- 保证原子性，可见性，有序性

- 变量从主内存到工作内存，执行read+load操作
- 变量从工作内存到主内存，执行store+write操作
- assign: 从执行引擎传递给工作内存，use: 从工作内存传递给执行引擎
- 主内存变量是否线程独占：lock与unlock

### volatile
- 保证共享变量对所有线程可见：写操作会强制刷新主内存，并且使其他线程的缓存无效
- 禁止指令重排优化
- 内存屏障指令：LoadLoad, StoreStore, LoadStore, StoreLoad

### happens-before规则
- A happens-before B, A的结果B可见

## JVM内存区域
- 程序计数器：线程私有。如果指向的是本地方法，存放的是undefined。不会导致OOO。
- Java栈：线程私有。为方法服务，存放方法参数，局部变量等信息。栈深度溢出SOE，栈的动态扩展OOO, -Xss。
- 本地方法栈：线程私有。为本地方法服务
- Java堆：线程共享。垃圾回收的主要对象。堆的动态扩展，-Xms, -Xmx, 内存溢出，内存泄露
- 直接内存/堆外内存：库函数可直接分配堆外内存，也会导致OOO
- 方法区：线程共享。存放类加载子系统加载的类信息，同时包括运行时常量池。垃圾回收的次要对象。-XX:PermSize, -XX:MaxPermSize，运行时常量池溢出

### 运行时常量池
- 存放字面量，符号引用量
- 不仅包括编译时生成的常量，也包括运行时生成的常量

### 栈帧
- 局部变量表：方法参数 + 局部变量, variable slot 可以复用
- 操作数栈：进行算术运算
- 动态连接：运行时常量池？
- 返回地址：正常返回PC计数器，异常返回异常处理器表
- 方法返回时：1. 恢复上层方法的局部变量表和操作数栈；2. 返回值压入调用者栈帧的操作数栈；3. 调整PC计数器

- 存储锁记录：复制对象头的Mark Word，叫做Displaced Mark Word


### OOM的实例和解决办法
- 持有大数组，大集合类
- SoftReference：内存不够时会被回收
- WeakReference: 不管内存够不够都会被回收


## 垃圾回收

### 分代收集算法
年轻代用复制算法，年老代用标记-清除算法
年轻代默认用串行GC
Client模式年老代默认用串行GC，Server模式年老代默认用并行GC

### 复制算法
年轻代按照8:1:1分成3块，整理Eden区的时候，把存活对象复制到Survivor区，然后清除Eden区。

### 根搜索算法
GC root对象作为起点，搜索所有走过的路径（引用链），不可达的对象可以被回收。
GC root: 方法区的静态变量，方法区的常量，Java栈的变量，本地方法栈的变量

### Serial GC串行GC/Parallel GC平行GC 标记-清理-压缩
* 标记：标记年老代的存活对象
* 清理：从头检查，只留下存活对象
* 压缩：从头开始，顺序填满
* JDK默认是P GC

### CMS GC Concurrent-Mask-Sweep 并发标记-清除算法
* 目的是为了缩短停机时间
* 初始化标记，只查找距离类加载器最近的幸存对象，停机时间很短
* 并发标记，查找幸存对象的引用，此时其他线程仍在运行
* 重新标记，修正并发标记期间的变动， 停机时间很短
* 并行清除，清除对象，此时其他线程仍在运行
* 问题：高CPU，不能清理干净，没有压缩导致内存碎片

### G1 GC
* 目的是既要停机时间短，又要内存碎片少
* Java堆划分成大小相等的区域Region
* 年轻代包含几个区域，年老代也包含几个区域
* 跟踪各个Region的垃圾收集情况：回收空间大小，回收耗时
* 维护一个优先队列，优先回收高价值的Region
* Region之间对象的引用关系记录在Remembered Set数据结构中


### 年轻代
* Eden, From, To
* 新建对象放Eden，放不下触发Minor GC，整理Eden, 存活对象放入From
* 如果From放不下，则继续整理From，存活对象放入To，然后交换From与To
* 如果To放不下，则放入年老代

### 年老代
* 年老代放不下对象，则触发Full GC
* 对象体积大于-XX:PretenureSizeThreshold，直接放入年老代
* 对象年龄大于XX:MaxTenuringThreshold，晋升到年老代
* Survivor相同年龄的对象超过空间的一半，则把年龄大于等于的对象晋升到年老代

### 持久代
* 大小-XX:MaxPermSize


## 类加载

### 类加载器
* Bootstrap ClassLoader: 核心类库，-Xbootclasspath，-Dsun.boot.class.path
* Extension ClassLoader: 扩展类，-Djava.ext.dirs
* Application ClassLoader: 应用类，-Djava.class.path

### 类加载器父类
* 自定义类加载器的父类是AppClassLoader
* AppClassLoader的父类是ExtClassLoader
* ExtClassLoader没有父类

### 双亲委托
* 委托父类加载器加载
* 为了解决类加载过程中的安全性问题，比如加载自己的java.lang.Object
* findLoadedClass() 检查该类是否已经加载过
* parent.loadClass() 委托父类加载
* findBootstrapClassOrNull() 委托Bootstrap ClassLoader加载
* findClass() 自己加载

### autowired
* 使@Lazy标签无效

### JVM性能调优
- jmap -heap pid
- jmap -histo pid | grep dapp | sort -k 2 -g -r | less
- jmap -dump:live,format=b,file=d.log pid
- jhat -port 3000 d.log
- jstat -gc pid


## 监控
### jmap
heap的使用情况
jmap -heap pid

导出dump，用jhat分析
jmap -dump:format=b,file=outfile pid

统计对象的柱状图
jmp histo pid

统计class loader (慎用，非常耗时)
jmap -clstats pid

等待finalization的对象
jmap -finalizerinfo pid

### jstat
GC统计
jstat -gc pid

元数据空间统计
jstat -gcmetacapacity pid

类加载统计
jstat -class pid

### jstack
查看运行栈
jstack pid


## 配置
### 内存
-server -Xms8192m -Xmx8192m -Xmn1890m

### GC
-verbose:gc -XX:+UseConcMarkSweepGC -XX:MaxTenuringThreshold=5 -XX:+ExplicitGCInvokesConcurrent -XX:GCTimeRatio=19 -XX:CMSInitiatingOccupancyFraction=70 -XX:CMSFullGCsBeforeCompaction=0 -Xnoclassgc -XX:SoftRefLRUPolicyMSPerMB=0

## CPU 问题排查
top 查找最耗CPU的进程
top -Hp pid 查找最耗CPU的线程
printf "%x\n" pid 转成16进制
jstack 29079 | grep '0x71ec' -A20 找到线程的堆栈











