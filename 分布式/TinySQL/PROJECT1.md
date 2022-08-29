# 理论

在这一个项目中我们要实现`Codec`，作用是将关系表上的数据映射到TiKV的数据空间上，换言之，我们要实现关系模型到 Key-Value 模型的映射。

为什么要这样做呢？TiDB并不单指一个数据库，它的分层架构嵌套了多种结构，其中TiKV 是一个分布式事务型的键值数据库，它同时是 TiDB 的存储层，为用户写入 TiDB 的数据提供了持久化以及读写服务，同时还存储了 TiDB 的统计信息数据。也就是说，TiDB的底层数据是以K-V形式存储的。

具体TiKV怎么映射，看[文章](https://pingcap.com/zh/blog/tidb-internal-2#关系模型到-key-value-模型的映射)即可。

SQL如何查询KV数据？TiDB 的整体架构如下图所示

![](http://pic.netpunk.space/images/2022/07/12/20220712175951.png)

TiKV Cluster 主要作用是作为 KV 引擎存储数据，而TiDB Servers就是SQL层，这一层的节点都是无状态的节点，本身并不存储数据，节点之间完全对等。TiDB Server 这一层最重要的工作是处理用户请求，执行 SQL 运算逻辑，在这一层需要将计算尽量靠近存储节点，以避免大量的 RPC 调用。其次，我们需要将 Filter 也下推到存储节点进行计算，这样只需要返回有效的行，避免无意义的网络传输。最后，我们可以将聚合函数、GroupBy 也下推到存储节点，进行预聚合，每个节点只需要返回一个 Count 值即可，再由 tidb-server 将 Count 值 Sum 起来。 

<img src="http://pic.netpunk.space/images/2022/07/12/20220712180155.png" style="zoom:80%;" />

## 实现

### 存储分析

从上一节介绍的数据处理看，我们需要存储具有怎样的性质呢：

- 首先，从单表操作的角度看，同一张表的数据应当是存放在一起的，这样我们可以避免在处理某一张表时读取到其他表上的数据
- 其次，对于同一张表内的数据，我们该如何排列呢？从 SQL 过滤条件的角度来看
  - 如果基本上没有过滤条件，那么无论怎么排列都没关系
  - 如果类似 person = “xx” 这样的等值查询比较多，那么一个类似 hash 表的排列是比较优的，当然一个有序数组也可以
  - 如果类似 number >= “xx” 这样的查询比较多，那么一个类似有序数组的排列是比较优的，因为这样我们可以避免读取较多的无用数据
- 对于同一行上的数据，我们是分开存还是存在一起比较好呢？再一次，这取决于数据访问的模式，如果同一行上的数据总是需要同时被读取，那么存在一起是更好的，在 TinySQL 中我们选择了将同一行上的数据存放在一起。

从上面的角度看，一个类似有序的数组的排列可能是最简单的方式，因为几乎它的可以用一个统一的方式满足所有的要求。

### 编码规则

从最简单的角度看，我们可以将它看做一个提供了如下性质的 KV 引擎：

- Key 和 Value 都是 bytes 数组，也就是说无论原先的类型是什么，我们都要序列化后再存入
- Scan(startKey)，任意给定一个 Key，这个接口可以按顺序返回所有大于等于这个 startKey 数据。
- Set(key, value)，将 key 的值设置为 value。

结合上面的讨论，数据的存储方式就呼之欲出了：

- 由于同一张表的需要存放在一起，那么表的唯一标示应该放在 Key 的最前面，这样同一张表的 Key 就是连续的
- 对于某一张表，将需要排序的列放在表的唯一标示后面，编码在 Key 里
- Value 中存放某一行上所有其他的列

具体来说，我们会对每个表分配一个 TableID，每一行分配一个 RowID（如果表有整数型的 Primary Key，那么会用 Primary Key 的值当做 RowID），其中 TableID 在整个集群内唯一，RowID 在表内唯一，这些 ID 都是 int64 类型。 每行数据按照如下规则进行编码成 Key-Value pair：

```
Key： tablePrefix_tableID_recordPrefixSep_rowID
Value: [col1, col2, col3, col4]
```

其中 Key 的 tablePrefix/recordPrefixSep 都是特定的字符串常量，用于在 KV 空间内区分其他数据。对于索引，会为每一个索引分配表内唯一的 indexID，然后按照如下规则编码成 Key-Value pair：

```
Key: tablePrefix_tableID_indexPrefixSep_indexID_indexColumnsValue
Value: rowID
```

当然，我们还需要考虑非唯一索引，这个时候上面的方法就行不通了，我们需要将 rowID 也编码进 Key 里使之成为唯一的：

```
Key: tablePrefix_tableID_indexPrefixSep_indexID_ColumnsValue_rowID
Value：null
```

### 实现DecodeRecordKey方法

通过面向测试编程（x）我认为该函数的功能是将

* rowkey：行索引，36个16进制数，由table ID和最大32位无符号数（行ID）拼接组成

分解为

* tableID：
* Handle：
* err：

**参考实现**

<img src="http://pic.netpunk.space/images/2022/07/14/20220714110702.png" style="zoom:80%;" />



