# 执行模型

在这一章节我们将介绍执行层最核心的执行模型。

## 执行模型介绍

在介绍具体模型之前，我们先思考一下为什么需要模型。模型是对现实事物核心特质的简化，可以用来帮助人们理解问题。在前面的 SQL 部分中，可以看到 SQL 的可以表达语义复杂多变，但同时它也具有一定的特性：

- SQL 是由不同的部分组成的，每个部分有固定的语义
- 部分与部分之间是有一定关系的，每个部分都是对前一部分结果的进一步处理

### 火山模型

让我们先来看一看火山模型。在火山模型中，由不同的执行器组成，每个执行器对应的是 SQL 中的某个部分，例如过滤，聚合等；执行器与执行器之间组成了类似树状的关系，每个算子都实现了三个接口：

- Open，对当前执行器所需的资源进行初始化
- Next，从孩子节点（如果存在）取必需的数据，计算并返回一条结果
- Close，对执行器所需的资源进行释放

[![Volcano Execution Model](https://github.com/talent-plan/tinysql/raw/course/courses/imgs/proj5-part1-1.png)](https://github.com/talent-plan/tinysql/blob/course/courses/imgs/proj5-part1-1.png)

从这里也可以看到，火山模型是符合我们对模型的设想的，每个执行器负责特定的语义，并通过树型结构灵活地组合起来。那么它的缺点是什么呢？如果处理的数据量多，那么每个算子输出的每一行都对应一次 `Next` 调用，框架上的函数调用开销将会非常大。

### 向量化

减小函数调用的一个直观想法就是每次 `Next` 返回一批数据，而不是只返回一行。为了支持返回多行的操作，TinySQL 还使用了 `Chunk` 来表示这些行，用于减小内存分配开销、降低内存占用以及实现内存使用量统计/控制。

在结果按批返回后，也为计算的向量化带来了可能性，但首先，我们先来了解一下表达式及其计算。

## 代码

**向量化表达式**

在 [builtin_string_vec.go](https://github.com/tidb-incubator/tinysql/blob/course/expression/builtin_string_vec.go) 有三个较为简单的向量化的 string 类型函数，可以结合[向量化的进行计算](https://docs.google.com/document/d/1JKP9YS3wYsuXsYhDgVepJt5y72K6_WxhUVfOLyjpAjc/edit#heading=h.66r4twnr3b1c)阅读。

**火山模型**

我们以 Selection 为例来介绍代码。

在 [executor.go#L346](https://github.com/tidb-incubator/tinysql/blob/course/executor/executor.go#L346) 实现了一个较为简单的执行器 `Selection`，它的作用就是根据 `filters` 过滤掉不需要的行并返回给父亲，可以看到它也实现了常见的 `Open`, `Next` 和 `Close` 接口。可以通过阅读 unBatchedNext 理解一下它的功能。

## 作业

- 实现向量化表达式 [vecEvalInt](https://github.com/tidb-incubator/tinysql/blob/course/expression/builtin_string_vec.go#L89) ，并将 [vectorized](https://github.com/tidb-incubator/tinysql/blob/course/expression/builtin_string_vec.go#L84) 的返回值改为 `true`
- 实现向量化 selection 的 [Next](https://github.com/tidb-incubator/tinysql/blob/course/executor/executor.go#L380) 函数。

**测试**

- 通过 `expression` 下的所有测试
- 通过 `executor/executor_test.go` 中 `TestSelectExec`。

你可以直接通过 `make test-proj5-1` 来运行测试，也可以通过 `go test package_path -check.f func_name` 来跑一个具体的函数。以 `TestSelectExec` 为例，你可以使用 `go test ./executor -check.f "TestSelectExec"` 来跑这个具体的函数。

**评分**

`expression` 和 `executor` 各占 50%。

# Hash Join

在这一小节我们将学习 Hash Join 及其实现，并且从这一小节开始我们将接触并发计算 。Hash Join 是实现 Join 的一种常见方式，除此之外 TinySQL 还实现了与 Merge Sort 思想较类似的 [Merge Join](https://github.com/pingcap-incubator/tinysql/blob/df75611ce926442bd6074b0f32b1351ca4aad925/executor/merge_join.go#L24)，感兴趣可以自行阅读。

## Hash Join 算法简介

简单来说，对于两张表的 Hash Join，我们会选择选择一个内表来构造哈希表，然后对外表的每一行数据都去这个哈希表中查找是否有匹配的数据。那怎样提高 Hash Join 的效率呢？在建立好哈希表后，实际上哈希表就是只读的了，那么查找匹配的过程其实是可以并行起来的，也就是说我们可以用多个线程同时查哈希表：

[![Hash Join 1](https://github.com/talent-plan/tinysql/raw/course/courses/imgs/proj5-part2-1.png)](https://github.com/talent-plan/tinysql/blob/course/courses/imgs/proj5-part2-1.png)

这样可以大大提高 Hash Join 的效率。

## 理解代码

从上图也可以看到，其包含的主要过程如下：

- Main Thread：一个，执行下列任务：
  1. 读取所有的 Inner 表数据并构造哈希表
  2. 启动 Outer Fetcher 和 Join Worker 开始后台工作，生成 Join 结果。各个 goroutine 的启动过程由 fetchAndProbeHashTable 这个函数完成；
  3. 将 Join Worker 计算出的 Join 结果返回给 NextChunk 接口的调用方。
- Outer Fetcher：一个，负责读取 Outer 表的数据并分发给各个 Join Worker；
- Join Worker：多个，负责查哈希表、Join 匹配的 Inner 和 Outer 表的数据，并把结果传递给 Main Thread。

接下来我们细致的介绍 Hash Join 的各个阶段。

### Main Thread 读内表数据并构造哈希表

读 Inner 表数据的过程由 fetchAndBuildHashTable 这个函数完成。这个过程会不断调用 Child 的 NextChunk 接口，把每次函数调用所获取的 Chunk 存储到 hashRowContainer 中供接下来的计算使用。

我们这里使用的哈希表本质上是一个链表，将具有相同 Key 哈希值的类似链表的方式连在一起，这样后续查找具有相同 Key 的值时只需要遍历链表即可。

### Outer Fetcher

Outer Fetcher 是一个后台 goroutine，他的主要计算逻辑在 fetchOuterSideChunks 这个函数中。它会不断的读大表的数据，并将获得的 Outer 表的数据分发给各个 Join Worker。这里多线程之间的资源交互可以用下图表示：

![proj5-part2-2.jpg](https://github.com/talent-plan/tinysql/blob/course/courses/imgs/proj5-part2-2.jpg?raw=true)

上图中涉及到了两个 channel：

1. outerResultChs[i]：每个 Join Worker 一个，Outer Fetcher 将获取到的 Outer Chunk 写入到这个 channel 中供相应的 Join Worker 使用
2. outerChkResourceCh：当 Join Worker 用完了当前的 Outer Chunk 后，它需要把这个 Chunk 以及自己对应的 outerResultChs[i] 的地址一起写入到 outerChkResourceCh 这个 channel 中，告诉 Outer Fetcher 两个信息：
   1. 我提供了一个 Chunk 给你，你直接用这个 Chunk 去拉 Outer 数据吧，不用再重新申请内存了；
   2. 我的 Outer Chunk 已经用完了，你需要把拉取到的 Outer 数据直接传给我，不要给别人了；

所以，整体上 Outer Fetcher 的计算逻辑是：

1. 从 outerChkResourceCh 中获取一个 outerChkResource，存储在变量 outerResource 中
2. 从 Child 那拉取数据，将数据写入到 outerResource 的 chk 字段中
3. 将这个 chk 发给需要 Outer 表的数据的 Join Worker 的 outerResultChs[i] 中去，这个信息记录在了 outerResource 的 dest 字段中

### Join Worker

每个 Join Worker 都是一个后台 goroutine，主要计算逻辑在 runJoinWorker 这个函数中。

![Hash Join 3](https://github.com/talent-plan/tinysql/raw/course/courses/imgs/proj5-part2-3.jpg)

上图中涉及到两个 channel：

1. joinChkResourceCh[i]：每个 Join Worker 一个，用来存 Join 的结果
2. joinResultCh：Join Worker 将 Join 的结果 Chunk 以及它的 joinChkResourceCh 地址写入到这个 channel 中，告诉 Main Thread 两件事：
   1. 我计算出了一个 Join 的结果 Chunk 给你，你读到这个数据后可以直接返回给你 Next 函数的调用方
   2. 你用完这个 Chunk 后赶紧还给我，不要给别人，我好继续干活

所以，整体上 Join Worker 的计算逻辑是：

1. 获取一个 Join Chunk Resource
2. 获取一个 Outer Chunk
3. 查哈希表，将匹配的 Outer Row 和 Inner Rows 写到 Join Chunk 中
4. 将写满了的 Join Chunk 发送给 Main Thread

### Main Thread

主线程的计算逻辑由 NextChunk 这个函数完成。主线程的计算逻辑非常简单：

1. 从 joinResultCh 中获取一个 Join Chunk
2. 将调用方传下来的 chk 和 Join Chunk 中的数据交换
3. 把 Join Chunk 还给对应的 Join Worker

## 作业

实现 [runJoinWorker](https://github.com/tidb-incubator/tinysql/blob/course/executor/join.go#L243) 以及 [fetchAndBuildHashTable](https://github.com/tidb-incubator/tinysql/blob/course/executor/join.go#L148) 。

**测试**

- 通过 `join_test.go` 下除 `TestJoin` 以外的所有测试用例。
- `TestJoin` 测试用例涉及到 project5-3 中 `aggregate` 相关方法，可以在完成整个 project5 后再测试，该测试不会计算到分数之中。

你可以通过 `make test-proj5-2` 来运行测试。

**评分**

全部通过可得 100 分。若有测试未通过按比例扣分。

# Hash Aggregate

在这一小节，我们将深入讲解 Hash Agg 的原理和实现。

## Hash Agg 简介

在理解 Hash 聚合之前，我们先回顾下聚合。以 `select sum(b) from t group by a` 为例，它的意思就是求那些具有相同 a 列值的 b 列上的和，也就是说最终每个不同的 a 只会输出一行，那么其实最终的结果可以认为是一个类似 map<a, sum(b)> 这样的 map，那么而更新 map 的值也比较简单。

怎样进行优化呢？一个观察是 sum 这样的函数是满足交换律和结合律的，也就是说相加的顺序没有关系，例如假设某个具体的 a 值具有 10 行，那么我们可以先计算其中 5 行的 sum(b) 值，再计算另外 5 行 sum(b) 值，再把这样的结果合并起来。

对于 avg 这样的函数怎样优化呢？我们可以将它拆成是两个聚合函数，即 sum(b) 和 count(b)，两个函数都满足交换律和结合律，因此也可以用上面的方法去优化。

那么一个可行的优化就是我们引入一个中间状态，这个时候每个 partial worker 从孩子节点取一部分数据并预先计算，注意这个时候得到的结果可能是不正确的，然后再将结果交给一个合并 partial worker 结果的 final worker，这样就可以提高整体的执行效率。

但上面的模型可能的一个结果就是有可能 a 列上不同值的数目较多，导致 final worker 成为瓶颈。怎样优化这个瓶颈呢？一个想法是将 fianl woker 拆成多个，但这个时候怎样保证正确性呢？即怎样让具有相同 a 值的、由 partial worker 所计算的中间结果映射到同一个 final worker 上？这个时候哈希函数又可以排上用场了，我们可以将 a 上的值进行哈希，然后将具有相同哈希值的中间结果交给同一个 final worker，即：

这样我们就得到了 TinySQL 中 Hash Agg 的执行模型。 为了适应上述并行计算的，TiDB 对于聚合函数的计算阶段进行划分，相应定义了 4 种计算模式：CompleteMode，FinalMode，Partial1Mode，Partial2Mode。不同的计算模式下，所处理的输入值和输出值会有所差异，如下表所示：

| AggFunctionMode | 输入值   | 输出值               |
| --------------- | -------- | -------------------- |
| CompleteMode    | 原始数据 | 最终结果             |
| FinalMode       | 中间结果 | 最终结果             |
| Partial1Mode    | 原始数据 | 中间结果             |
| Partial2Mode    | 中间结果 | 进一步聚合的中间结果 |

以上文提到的 `select avg(b) from t group by a` 为例，通过对计算阶段进行划分，可以有多种不同的计算模式的组合，如：

- CompleteMode

此时 AVG 函数 的整个计算过程只有一个阶段，如图所示：

![](http://pic.netpunk.space/images/2022/08/04/20220804142336.png)

- Partial1Mode --> FinalMode

此时我们将 AVG 函数的计算过程拆成两个阶段进行，如图所示：

![](http://pic.netpunk.space/images/2022/08/04/20220804142353.png)

除了上面的两个例子外，还可能聚合被下推到 TinyKV 上进行计算（Partial1Mode），并返回经过预聚合的中间结果。为了充分利用 TinySQL 所在机器的 CPU 和内存资源，加快 TinySQL 层的聚合计算，TinySQL 层的聚合函数计算可以这样进行：Partial2Mode --> FinalMode。

## 理解代码

TiDB 的并行 Hash Aggregation 算子执行过程中的主要线程有：Main Thead，Data Fetcher，Partial Worker，和 Final Worker：

- Main Thread 一个：
  - 启动 Input Reader，Partial Workers 及 Final Workers
  - 等待 Final Worker 的执行结果并返回
- Data Fetcher 一个：
  - 按 batch 读取子节点数据并分发给 Partial Worker
- Partial Worker 多 个：
  - 读取 Data Fetcher 发送来的数据，并做预聚合
  - 将预聚合结果根据 Group 值 shuffle 给对应的 Final Worker
- Final Worker 多 个：
  - 读取 PartialWorker 发送来的数据，计算最终结果，发送给 Main Thread

Hash Aggregation 的执行阶段可分为如下图所示的 5 步：

![](http://pic.netpunk.space/images/2022/08/04/20220804142419.png)

1. 启动 Data Fetcher，Partial Workers 及 Final Workers

   这部分工作由 prepare4ParallelExec 函数完成。该函数会启动一个 Data Fetcher，多个 Partial Worker 以及多个 Final Worker。

2. DataFetcher 读取子节点的数据并分发给 Partial Workers

   这部分工作由 fetchChildData 函数完成。

3. Partial Workers 预聚合计算，及根据 Group Key shuffle 给对应的 Final Workers

   这部分工作由 HashAggPartialWorker.run 函数完成。该函数调用 updatePartialResult 函数对 DataFetcher 发来数据执行预聚合计算，并将预聚合结果存储到 partialResultMap 中。其中 partialResultMap 的 key 为根据 Group-By 的值 encode 的结果，value 为 PartialResult 类型的数组，数组中的每个元素表示该下标处的聚合函数在对应 Group 中的预聚合结果。shuffleIntermData 函数完成根据 Group 值 shuffle 给对应的 Final Worker。

4. Final Worker 计算最终结果，发送给 Main Thread

   这部分工作由 HashAggFinalWorker.run 函数完成。该函数调用 consumeIntermData 函数接收 PartialWorkers 发送来的预聚合结果，进而合并得到最终结果。getFinalResult 函数完成发送最终结果给 Main Thread。

5. Main Thread 接收最终结果并返回

## 作业描述

- 补充完整 `aggregate.go` 中的 TODO 代码，包括 `consumeIntermData` 和 `shuffleIntermData` 两个方法。

**测试**

- 完成 `aggregate_test.go` 中的所有测试

你可以通过 `make test-proj5-3` 来运行测试

**评分**

全部通过可得 100 分。若有测试未通过按比例扣分。



