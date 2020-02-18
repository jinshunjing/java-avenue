# LevelDB

### 系统架构
- 适用场景：写多读少，写的快，读的慢
- 内存：MemTable, ImmutableMemTable
- 磁盘：.stt SSTable
- 日志文件: .log, manifest, current

磁盘文件
- 分层: level 0, level 1, level 2, ...
- sst文件内部的key有序
- level 0 的两个文件可能有key重叠，其他层的文件不会有key重叠

- current文件：保存当前manifest文件(可能有多个manifest文件)
- manifest文件: level x, x.sst, key min, key max

- 日志文件：物理上分成多个block，每块32k。记录有4种类型：FULL-记录完整的存储在一个block里，其他情况FIRST/MIDDLE/LAST
- 日志文件块：类型和数据的checksum, 数据长度，类型，数据（键值对）

- sst文件：物理上分成多个块，逻辑上也分成多个块
- sst文件物理块：数据，压缩类型，校验CRC
- sst逻辑块：数据块，索引块
- 索引块：数据块最大的key，数据块的开始位置，数据块的长度
- 数据块：key有序的记录，重启点（相邻的两个key会有重复的部分，后一个可以省略重复的部分前缀；重启点就是完整的key）
- 记录：key共享长度，key非共享长度，value长度，key非共享内容，value内容

内存
- MemTable
- Lazy删除
- 插入保持key有序
- 背后是一个SkipList


写入：
- 顺序写入日志
- 成功之后，写入内存SkipList

删除
- 写入一个{key，删除标记}
- 通过后面的compaction来删除

写磁盘：
- MemTable 变成 Immutable MemTable，通过compaction，写磁盘文件SSTable

读取
- 先读MemTable, Immutable MemTable
- 先查询level 0，接着查询level 1, level 2, ...
- level L 和 level L+1 都存在key，那么 level L要比level L+1新
- level L+1是level L经过compaction之后得到的
- level 0可能存在多个块文件保存了相同的key，查找的时候需要把所有包含key的文件都读出来，然后挑文件最新的

压缩compaction
- minor: 从Immutable MemTable到SSTable
- major: 从level L到level L+1
- full: 所有的SSTable

major compaction
- 如果是level 0，那么要把有重叠的多个文件都选出来；如果不是level 0，则只需要选出一个文件
- level L+1 层需要选出所有有重叠的文件
- 通过多路归并排序的方式，依次找出最小的key，生成新的level L+1 SSTable文件
- 原来的文件全部删除
- 如果一个key在更小的level中存在，那么compaction的时候可以丢弃

缓存cache
- Table Cache: SSTable文件的信息
- Block Cache：Block内容







