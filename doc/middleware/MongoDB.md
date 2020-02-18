
# MongoDB

## 使用
* 默认的数据库路径 -dbpath=/data/db

## 副本集Replica Set模式
- 角色：主节点(primary)，副本节点(secondary)，仲裁者(Arbiter)
- 仲裁者参与选主，但是如果仲裁者挂了就选不了主，推荐使用没有仲裁者的方案
- priority高的是主节点，仲裁者设置arbiterOnly=true
- 主节点写oplog
- 从节点定期从主机读取oplog，并在本地执行
- 配置文件加上Replica Set的名字：replSet=
- 初始化：连上任意一个点击，然后设置rs.initiate(cfg)


## 设计原则
* 需要单独访问的对象不要内嵌
* 需要经常更新的字段不要冗余
* 一对很少：直接内嵌
* 一对许多：比如一个block包含多个tx，在block加入tx主键的数组，也可以加入一些tx的冗余字段
* 一对非常多：比如一个公司有非常多的员工，在员工加入公司的主键，也可以加入一些公司的冗余字段
* 多对多：使用双向关联，加入主键的数组


## BSON: Binary JSON

### _id
"_id" : ObjectId("5ba1e274d276de40947b06b6")
4字节时间戳：5ba1e274
3字节机器标识码：d276de
2字节进程ID：4094
3字节随机数：7b06b6

### snowflake 雪花
https://github.com/apache/incubator-shardingsphere/blob/d47354341bf87bf032ba84e29a0cb7dac127083c/sharding-core/src/main/java/org/apache/shardingsphere/core/keygen/generator/impl/SnowflakeShardingKeyGenerator.java
1位: 0
41位: 时间戳
10位: 机器标识码
12位: 随机数

### Double, NumberInt, NumberLong
默认是Double类型

## MMAPv1文件结构
* http://www.mongoing.com/archives/1484
* MMAP内存映射
* 每个数据库database有独立的文件: .ns, .0, .1, ...
* 数据文件大小每次翻倍
* 名字空间文件.ns：namespace，对应collection；每个namespace包含多个extent；每个extent包含多条record，对应document;
* 数据文件内部分成块extent，每个块extent只保留一个名字空间namespace的数据，同一个namespace的所有extent之间通过双向链表; 
* 每个块extent中保存多条记录record（document，BSON），记录之间通过双向链表
* 删除记录：放入DeleteRecord单向链表，根据记录的大小分成多个链表
* 写入记录：先查看是否有足够大小的DeleteRecord；然后查看空闲的extent；最后创建新的extent，有可能要创建新的数据文件
* 更新记录：如果记录变小，直接修改；如果记录变大，先删除再添加

## 索引
* 索引也在数据文件中，是B-Tree
* Document插入之后会返回一个位置信息DiskLoc，索引的值就是这个位置信息
* 单字段索引
* 复合索引，索引前缀
* 多key索引，数组

### 索引优化
* 查询计划explain()

## 数据库备份与导入
mongodump -h dbhost -d dbname -o dbdirectory
mongorestore -h dbhost -d dbname --dir dbdirectory

mongodump -h 47.98.96.138:27017 -u fingo -p 186907189e58f62fd3dc889b0ac6b2be -d fingo -o /opt/ops/mongodb/
tar -zcvf fingo.tar.gz fingo/
mongorestore -h 47.99.196.196:27017 -u fingo -p 186907189e58f62fd3dc889b0ac6b2be -d fingo --dir /opt/fingo/

