
## 问题
- 自动生成文档ID的算法？UUID? Hash?
- 多个字段共享一个索引？还是每个字段有自己的索引？
- 为什么要移除 type？ 同一个index下面的不同type里的同名的field，Lucene看成是一个字段。一个index只包含一个type
- 如何定义会被索引的字段？字段的值存储？
- sum返回的是duoble,怎么聚合big decimal？

## Lucene
### 倒排索引 inverted index
由属性值来确定记录的位置
- 取得关键词：分词，过滤无意义的词语和标点符号，统一大小写，统一时态
- 建立倒排索引：关键词，出现位置，出现频率。关键词按字母序排列，二分查找
- 词典文件 term dictionary，频率文件 frequencies，位置文件 positions （文档ID）
- 压缩算法：关键词前缀压缩，数字差值压缩

## 概念
- 索引词 term
- cluster, node,
- 索引 index：一个索引默认有5个主分片和1个副本
- 分片 shard：主分片，副本分片，每个分片是一个Lucene实例
- 复制 replica
- 类型 type / mapping type: 类型是索引的逻辑分区
- 文档 document: JSON格式，存储N个字段（键值对），原始的JSON存储在_source字段
- 映射 mapping: 每个索引都有一个映射，定义了字段类型
- ID: 文档的唯一标识

- 分段 segment: 一个分片有多个分段，每个分段是一个倒排索引。每秒更新到磁盘，类似于MySQL?
- 索引由segment 和 commit point 文件组成


### 查询性能
- Term Dictionary 放入内存，二分查找
- Term Index：term dictionary 太大；term index是一棵前缀树，指向term dictionary的block；term index 放入内存
- FST 压缩技术，Frame of Reference 压缩ID，Roaring Bitmaps 压缩ID
- 减少磁盘的随机访问，利用磁盘的顺序访问

### 联合索引
- 跳表：多个positions的交集
- bitset: 多个positions直接按位取与


## Java API
### 字段
- source 存储原始的JSON文档


### 索引分析
- char_filters 字符过滤器
- tokenizer 分词器
- token_filters 标记过滤器













