# Lab3-执行器

在第三个编程项目中，你将在你的数据库系统中添加对查询执行的支持。你将实现负责获取查询计划节点并执行的执行器。你将创建执行器来执行以下操作。

* 访问方法：顺序扫描
* 修改：插入、更新、删除
* 杂项：嵌套循环连接，哈希连接，聚合，限制，区别对待

因为DBMS不支持SQL（还没有），你的实现将直接在手写的查询计划上操作。

我们将使用迭代器查询处理模型（即Volcano模型）。回顾一下，在这个模型中，每个查询计划执行者都实现了一个Next函数。当DBMS调用执行者的Next函数时，执行者会返回(1)一个元组或(2)一个没有更多元组的指标。通过这种方法，每个执行器实现了一个循环，继续在其子代上调用Next，以检索tuple并逐一处理。

在BusTub的迭代器模型的实现中，每个执行器的Next函数除了返回一个tuple外，还返回一个记录标识符（RID）。记录标识符作为元组相对于它所属的表的唯一标识符。

这个项目是由一个单一的任务组成的，在这个任务中，你根据下面提供的规格来实现每个单独的执行器。

**PROJECT SPECIFICATION**

和以前的项目一样，我们为你提供了定义API的类，你必须实现这些API。你不应该修改ExecutionEngine类中预定义的函数的签名，或者各个执行器的公共API中的任何成员函数。修改这些API会引入构建错误，导致自动生成器在评估你的提交时失败，你将不会得到项目的奖励。此外，如果一个类已经包含成员变量，你不应该删除它们。除了这些要求之外，你可以根据你的需要添加数据成员和成员函数。

本项目的正确性取决于你对以前项目的实现的正确性，特别是项目1中缓冲池管理器的实现和项目2中可扩展哈希表的实现。我们将不提供解决方案或二进制文件。在你开始实施之前，你必须先了解这个项目的主要概念，这一点很重要。

## TASK #1 - EXECUTORS

在这个任务中，你将实现九个执行器，每个执行器负责以下操作：sequential scan, insert, update, delete, nested loop join, hash join, aggregation, limit, 和 distinct。对于每个查询计划操作类型，都有一个相应的执行器对象，实现Init和Next方法。Init方法初始化操作符的内部状态（例如，检索要扫描的相应表）。Next方法提供了迭代器接口，在每次调用时返回一个元组和相应的RID（或指示器，表明执行器已用尽）。

你要实现的执行器被定义在以下头文件中：

- `src/include/execution/executors/seq_scan_executor.h`
- `src/include/execution/executors/insert_executor.h`
- `src/include/execution/executors/update_executor.h`
- `src/include/execution/executors/delete_executor.h`
- `src/include/execution/executors/nested_loop_join_executor.h`
- `src/include/execution/executors/hash_join_executor.h`
- `src/include/execution/executors/aggregation_executor.h`
- `src/include/execution/executors/limit_executor.h`
- `src/include/execution/executors/distinct_executor.h`

仓库中存在其他执行器的头文件（.h）和执行文件（.cpp）（例如index_scan_executor和nested_index_join_executor）。你不需要负责实现这些执行器。

每个执行器负责处理单一的计划节点类型。计划节点是组成查询计划的单个元素。每个计划节点可以定义它所代表的操作者的特定信息。例如，连续扫描的计划节点必须定义执行扫描的表的标识符，而LIMIT操作符的计划节点不需要这个信息。你要实现的执行器的计划节点在以下头文件中声明：

- `src/include/execution/plans/seq_scan_plan.h`
- `src/include/execution/plans/insert_plan.h`
- `src/include/execution/plans/update_plan.h`
- `src/include/execution/plans/delete_plan.h`
- `src/include/execution/plans/nested_loop_join_plan.h`
- `src/include/execution/plans/hash_join_plan.h`
- `src/include/execution/plans/aggregation_plan.h`
- `src/include/execution/plans/limit_plan.h`
- `src/include/execution/plans/distinct_plan.h`

这些计划节点已经拥有实现所需执行者的所有数据和功能。你不应该修改这些计划节点中的任何一个。

执行者接受来自其子代（可能是多个）的tuple，并向其父代输出tuple。它们可能依赖于关于子代排序的约定，我们将在下面描述。你也可以自由地添加private辅助函数和你认为合适的类成员。

在这个项目中，我们假设执行器是在单线程环境下运行的。你不需要采取措施来确保你的执行器在被多个线程同时执行时行为正确。

我们提供ExecutionEngine辅助类。它将输入的查询计划转换为查询执行器，并执行它直到产生所有结果。你必须修改ExecutionEngine来捕捉你的执行器在查询执行过程中由于失败而抛出的任何异常。正确处理执行引擎中的失败，对于未来项目的成功非常重要。

要了解在查询执行期间如何在运行时创建执行器，请参考ExecutorFactory辅助类。此外，每个执行器都可以访问它所执行的ExecutorContext。

你的一些执行器将执行表的修改（插入、更新和删除）。为了使表的索引与底层表保持一致，这些执行器也需要更新被执行修改的表的所有索引。你将使用项目2中的可扩展哈希表实现作为本项目中所有索引的基础数据结构。

最后，我们在test/execution/executor_test.cpp中为你提供了一些执行器的测试样本。这些测试远非详尽无遗，但它们应该给你一些想法，让你在为你的执行器编写自己的测试时可以从哪里开始。

### SEQUENTIAL SCAN

SeqScanExecutor遍历一个表，并逐一返回其tuple。顺序扫描由一个SeqScanPlanNode指定。该计划节点指定了要进行扫描的表。该计划节点也可以包含一个谓词；如果一个元组不满足该谓词，那么它就不会被扫描产生。

提示：使用TableIterator对象时要小心。请确保你理解预增加和后增加操作符之间的区别。你可能会发现自己通过在++iter和iter++之间切换得到奇怪的输出。

提示：你会想利用顺序扫描计划节点中的谓词。特别是，看一下AbstractExpression::Evaluate。注意，它返回一个Value，你可以调用`GetAs<bool>()`来评估结果为一个本地C++布尔类型。

提示：顺序扫描的输出是每个匹配元组的副本和它的原始记录标识符（RID）

### INSERT

InsertExecutor向表插入tuple并更新索引。有两种类型的插入操作是你的执行器必须支持的。

第一种类型是插入操作，其中要插入的值直接嵌入到计划节点本身。我们把这些称为原始插入。

第二种类型是插入操作，它从一个子执行器中获取要插入的值。例如，你可能有一个InsertPlanNode和一个SeqScanPlanNode作为子节点来实现INSERT INTO ... SELECT ...

你可以假设InsertExecutor总是在它所出现的查询计划的根部。InsertExecutor不应该修改其结果集。

提示：你需要在执行器初始化期间为插入的目标查询表信息。有关访问目录的其他信息，请参见下面的系统目录部分。

提示：你将需要更新插入tuple的表的所有索引。更多细节请参见下面的索引更新部分。

提示：你将需要使用TableHeap类来执行表的修改。

### UPDATE

UpdateExecutor修改指定表中的现有tuple并更新其索引。子执行器将提供更新执行器所要修改的tuple（连同其RID）。

与InsertExecutor不同，UpdateExecutor总是从子执行器中提取tuple来执行更新。例如，UpdatePlanNode将有一个SeqScanPlanNode作为其子节点。

你可以假设UpdateExecutor总是在它出现的查询计划的根部。UpdateExecutor不应该修改其结果集。

提示：我们为你提供了一个GenerateUpdatedTuple，它根据计划节点中提供的更新属性为你构造一个更新元组。

提示：你将需要在执行器初始化期间为更新的目标查找表信息。有关访问目录的其他信息，请参见下面的系统目录部分。

提示：你将需要使用TableHeap类来执行表的修改。

提示：你将需要更新执行更新的表的所有索引。更多细节请参见下面的索引更新部分。

### DELETE

DeleteExecutor从表中删除tuple，并从所有表的索引中删除其条目。像更新一样，要删除的tuple是从一个子执行器（如SeqScanExecutor实例）中提取的。

你可以假设DeleteExecutor总是在它所出现的查询计划的根部。DeleteExecutor不应该修改其结果集。

提示：你只需要从子执行器中获得RID并调用TableHeap::MarkDelete()来有效地删除元组。所有的删除都将在事务提交时应用。

提示：你将需要更新被删除元组的表的所有索引。更多细节请参见下面的索引更新部分。

### NESTED LOOP JOIN

NestedLoopJoinExecutor实现了一个基本的嵌套循环连接，将其两个子执行器的tuple结合在一起。

这个执行器应该实现讲座中介绍的简单嵌套循环连接算法。也就是说，对于连接外表中的每个元组，你应该考虑连接内表中的每个元组，并在连接谓词得到满足时发出一个输出元组。

本项目的学习目标之一是让你了解逻辑运算符的不同物理实现的优点和缺点。由于这个原因，重要的是你的NestedLoopJoinExecutor真正实现了嵌套循环连接算法，而不是，例如，重复使用你对哈希连接执行器的实现（见下文）。为此，我们将测试你的 NestedLoopJoinExecutor 的 IO 成本，以确定它是否正确实现了该算法。

提示：你将想利用嵌套循环连接计划节点中的谓词。特别是，看看AbstractExpression::EvaluateJoin，它处理左元组和右元组以及它们各自的模式。注意，它返回一个值，在这个值上你可以调用GetAs<bool>()来将结果评估为本地C++布尔类型。

### HASH JOIN

HashJoinExecutor实现了一个散列连接操作，将来自其两个子执行器的tuple结合在一起。

正如其名称所暗示的那样（正如在讲座中所讨论的那样），哈希连接操作是在哈希表的帮助下实现的。在这个项目中，我们做了一个简化的假设，即连接哈希表完全在内存中。这意味着在你的实现中，你不需要担心构建方表的临时分区溢出到磁盘上。

与 NestedLoopJoinExecutor 一样，我们将测试你的 HashJoinExecutor 的 IO 成本，以确定它是否正确地实现了散列连接算法。

提示：你的实现应该正确处理多个tuple（在连接的任何一边）共享一个共同的连接键的情况。

提示：HashJoinPlanNode定义了GetLeftJoinKey()和GetRightJoinKey()成员函数；你应该使用这些访问器返回的表达式来分别构建连接的左边和右边的连接键。

提示：你将需要一种方法来散列一个具有多个属性的元组，以便构造一个唯一的键。作为一个起点，看看AggregationExecutor中的SimpleAggregationHashTable是如何实现这个功能的。

提示：回顾一下，在查询计划的上下文中，哈希连接的构建方是一个管道中断器。这可能会影响你在实现中使用 HashJoinExecutor::Init() 和 HashJoinExecutor::Next() 函数的方式。特别是要考虑连接的构建阶段是否应该在 HashJoinExecutor::Init() 或 HashJoinExecutor::Next() 中执行。

### AGGREGATION

该执行器将单个子执行器的多个元组结果合并为一个元组。在这个项目中，我们要求你实现COUNT、SUM、MIN和MAX的聚合。你的AggregationExecutor还必须支持GROUP BY和HAVING条款。

正如在讲座中所讨论的，实现聚合的一个常见策略是使用哈希表。这就是你在这个项目中要使用的方法，然而，我们做了一个简化的假设，即聚合哈希表完全适合在内存中。这意味着你不需要担心实现哈希聚合的两阶段（Partition, Rehash）策略，而可以假设所有的聚合结果都可以存放在内存中的哈希表中。

此外，我们为你提供了SimpleAggregationHashTable数据结构，它暴露了一个内存哈希表（std::unordered_map），但有一个为计算聚合而设计的接口。该类还公开了SimpleAggregationHashTable::Iterator类型，可用于在哈希表中进行迭代。

提示：你会想要聚合结果并利用HAVING子句进行约束。特别是，看看AbstractExpression::EvaluateAggregate，它处理不同类型表达式的聚合评估。注意，它返回一个值，你可以调用GetAs<bool>()来将结果评估为本地C++布尔类型。

提示：回顾一下，在查询计划的上下文中，聚合是管道中断器。这可能会影响你在实现中使用AggregationExecutor::Init()和AggregationExecutor::Next()函数的方式。特别是，要考虑聚合的构建阶段是否应该在AggregationExecutor::Init()或AggregationExecutor::Next()中执行。

### LIMIT

LimitExecutor限制了来自其子执行器的输出tuple的数量。如果其子执行器产生的tuple数量少于LimitPlanNode中指定的限制，则该执行器没有任何作用，并产生它所收到的所有tuple。

### DISTINCT

DistinctExecutor可以消除从其子执行器中收到的重复元组。你的DistinctExecutor在确定唯一性时应该考虑输入元组的所有列。

像聚合一样，distinct操作符通常在哈希表的帮助下实现（这是一个哈希distinct操作）。这就是你在这个项目中要使用的方法。就像在AggregationExecutor和HashJoinExecutor中一样，你可以假设你用来实现DistinctExecutor的哈希表完全适合在内存中。

提示：你将需要一种方法来对具有潜在的许多属性的元组进行散列，以便构建一个唯一的键。作为一个起点，看看AggregationExecutor中的SimpleAggregationHashTable是如何实现这个功能的。

# ADDITIONAL INFORMATION

本节提供了一些关于其他系统组件的额外信息，为了完成这个项目，你将需要与这些组件进行互动。

## SYSTEM CATALOG

一个数据库维护一个内部目录，以跟踪关于数据库的元数据。在这个项目中，你将与系统目录交互，以查询有关表、索引及其模式的信息。

目录的全部实现都在 src/include/catalog.h 中。你应该特别注意成员函数 Catalog::GetTable() 和 Catalog::GetIndex() 。你将在执行器的实现中使用这些函数来查询目录中的表和索引。

## INDEX UPDATES

对于表的修改执行器（InsertExecutor、UpdateExecutor和DeleteExecutor），你必须修改操作所针对的表的所有索引。你可能会发现Catalog::GetTableIndexes()函数对于查询为某个特定表定义的所有索引很有用。一旦你拥有表的每个索引的 IndexInfo 实例，你就可以在底层索引结构上调用索引修改操作。

在这个项目中，我们使用你在项目2中对可扩展哈希表的实现作为所有索引操作的底层数据结构。因此，本项目的成功完成依赖于可扩展哈希表的工作实现。







