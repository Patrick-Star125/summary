# BigTable

BigTable是一个分布式存储系统，是早期Google牛逼的kv存储方案，现在对标BigTable的是HBase。

## 简介

BigTable被设计成可以扩展到PB的数据和上千个机器，BigTable已经达到了几个目标：广泛应用性（接口多）、可扩展性（成本低）、高性能和高可用性（容错强）。被应用在包括Google Analytics和Google Finance，Orkut，Personalized Search，Writely和Google Earth等产品上。这些产品所使用的BigTable的簇，涵盖了多种配置，从几个到几千个服务器，并且存储了几百TB的数据。

在许多方面，BigTable都和数据库很相似，它具有和数据库相同的实施策略。但是有一些不同，BigTable不能支持完整的关系型数据模型，相反，它为客户提供了一个简单数据模型，该数据模型可以支持针对数据部署和格式的动态控制，并且可以允许用户去推理底层存储所展现的数据的位置属性(比如具有相同前缀key的数据位置很接近，读取时候可进行一定的预取来进行优化)。

BigTable使用行和列名称对数据进行索引，这些名称可以是任意字符串。BigTable把数据视为未经解释的字符串，客户可能经常把不同格式的结构化数据和非结构化数据都序列化成字符串。最后，BigTable模式参数允许用户动态地控制，是从磁盘获得数据还是从内存获得数据。

## 数据模型

BigTable是一个稀疏的、分布式的、持久化存储的多维排序Map表，表中的数据通过一个行关键字（Row Key）、一个列关键字（Column Key）以及一个时间戳（Time Stamp）进行索引. 在Bigtable中一共有三级索引. 行关键字为第一级索引，列关键字为第二级索引，时间戳为第三级索引；Map中的每个value都是一个未经解析的byte数组，例如：

~~~go
(row:string,column:string,time:int64)->byte-array(string)
~~~

![](http://pic.netpunk.space/images/2022/09/19/20220919105434.png)

### **row 的特点**

- Bigtable 的行关键字可以是任意的字符串，但是大小不能够超过 64KB
- 表中数据都是根据行关键字进行排序的，排序使用的是**词典序**
- 同一地址域的网页会被存储在表中的连续位置
- **倒排**便于数据压缩，可以大幅提高压缩率

需要特别注意的是对于一个网站 [www.cnn.com](https://link.zhihu.com/?target=http%3A//www.cnn.com/) 存储在 Bigtable 中的格式是 com.cnn.www

这样倒排的好处是,对于同一域名下的内容,我们可以进行更加快速的索引.

### **column 的特点**

- 将其组织成列族（Column Family）
- 族名必须有意义，限定词则可以任意选定, 比如 “contents”, “title” 等等.
- 组织的数据结构清晰明了，含义也很清楚
- 族同时也是 Bigtable 中访问控制（Access Control）的基本单元

我们从 Bigtable 中读取数据先找到哪一行 然后再去选择读取那个一个 column.

### **time 的特点**

- Google的很多服务比如网页检索和用户的个性化设置等都需要保存不同时间的数据，这些不同的数据版本必须通过时间戳来区分。
- Bigtable中的时间戳是64位整型数，具体的赋值方式可以用户自行定义

选定了 row 和 column 之后,我们就会选择读取哪一个版本的,不同的时间戳代表着不同的数据版本.

## 组件

大概就是这样，bigtable在实际应用中和MapReduce、GFS、Chubby一起构成谷歌云服务基本架构

![](http://pic.netpunk.space/images/2022/09/19/20220919105623.png)



## 实现



## 优化



## 总结

