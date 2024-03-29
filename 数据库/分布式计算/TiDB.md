TiDB最强大的并不在于性能（它性能挺一般的），就像很多产品一样，它的知名度是因为其全面的生态和拓展性，它标称自己为HTAP数据库并整合了一系列的生态，目的是提供统一的OLTP和OLAP服务，即在不同的场景中同时支持高并发、低延时、高吞吐，在这里我想记录一下TiDB是如何做到的。

作为开发人员，需要知道各种数据库中各种可能支持的功能，明白其设计理念，利弊权衡和实现方法。从对TiDB的分析中可以发现，要打造一个完整的分布式关系型数据库的计算部分，需要支持至少以下的功能和特性：

|    模块    |                             特性                             |
| :--------: | :----------------------------------------------------------: |
|  用户友好  | 标准ANSI SQL支持、监控运维支持、多种调优方法、支持多种数据源 |
|    部署    |            跨平台支持、缩小内存占用、快速集群部署            |
|    开发    |              多语言SDK支持、MySQL兼容、故障诊断              |
| 逻辑优化器 |                     SQL重写、谓词下推、                      |
| 物理优化器 |                          统计信息、                          |
|  执行算子  |                          并行计算、                          |
| 存储一致性 |         分布式一致性实现、动态扩容、多种数据类型支持         |
|   可用性   |                    数据同步复制、数据迁移                    |
|    工具    | 自动运维、性能可视化、文件导入导出、强化数据迁移、实时日志、智能诊断、慢查询分析 |