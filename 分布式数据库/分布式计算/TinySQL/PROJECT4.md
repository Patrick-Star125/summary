# 优化器

从 Parser 往后，我们能够看到 Validator 和 Type Infer 两个流程。这两部分对应的是 SQL 合法性验证和类型系统，在实际的、用于生产环境的数据库系统中，这两部分十分重要，我们首先需要验证 SQL 的合法性，并且能够在预先定义的类型系统中成功识别和转换，以方便接下来的处理。不过由于这部分的内容略微枯燥，因此不作为 TinySQL 的课程内容，有兴趣的同学可以自行了解。

在 TinySQL 中存在两个优化器框架，我们能够看到进入优化器的入口：

~~~go
	// Handle the logical plan statement, use cascades planner if enabled.
	if sctx.GetSessionVars().EnableCascadesPlanner {
		finalPlan, err := cascades.DefaultOptimizer.FindBestPlan(sctx, logic)
		return finalPlan, names, err
	}
	finalPlan, err := plannercore.DoOptimize(ctx, builder.GetOptFlag(), logic)
~~~

这里实际上表明了在 TinySQL 中存在两个优化器框架，Cascades 和我们提到的 System R。[最初的论文](https://15721.courses.cs.cmu.edu/spring2018/papers/15-optimizer1/graefe-ieee1995.pdf)里对 Cascades 架构的设计理念做了一个比较细致的讲解。

从类型推导之后的处理逻辑如下面的代码所示

~~~go
// DoOptimize optimizes a logical plan to a physical plan.
func DoOptimize(ctx context.Context, flag uint64, logic LogicalPlan) (PhysicalPlan, error) {
	logic, err := logicalOptimize(ctx, flag, logic)
	if err != nil {
		return nil, err
	}
	physical, err := physicalOptimize(logic)
	if err != nil {
		return nil, err
	}
	finalPlan := postOptimize(physical)
	return finalPlan, nil
}
~~~

该函数的逻辑十分简单，只是依次调用了 `logicalOptimize`， `physicalOptimize` 和 `postOptimize`。其中，前两者就是最开始图中提到的优化流程，最后的 `postOptimize` 实际上一些优化之后对执行计划的一些调整，例如去掉多余的 projection 算子等。

一般优化器分两个阶段进行优化，即基于规则的优化（Rule-Based-Optimization，简称 RBO）和基于代价的优化（CBO）。

TiDB 主要分为两个模块对计划进行优化：

- 逻辑优化，主要依据关系代数的等价交换规则做一些逻辑变换。
- 物理优化，主要通过对查询的数据读取、表连接方式、表连接顺序、排序等技术进行优化。

我们可以知道同一个逻辑算子可能因为它的数据读取、计算方式等不同会生成多个不同的物理算子，例如逻辑上的 Join 算子转换成物理算子可以选择 HashJoin、SortMergeJoin、IndexLookupJoin。

这里会简单介绍一些逻辑算子可选择的物理算子。例如语句：`select sum(*) from t join s on t.c = s.c group by a`。此语句中逻辑算子有 DataSource、Aggregation、Join 和 Projection，接下来会对其中几个典型的逻辑算子对应的物理算子进行一个简单介绍，如下表：

![](http://pic.netpunk.top/images/2022/08/03/20220803154047.png)



## RBO逻辑优化

逻辑优化，即 logical optimize, 入口位于 `planner/core/optimizer.go`。逻辑优化做的事情实际上并不复杂：

```go
func logicalOptimize(ctx context.Context, flag uint64, logic LogicalPlan) (LogicalPlan, error) {
	var err error
	for i, rule := range optRuleList {
		// The order of flags is same as the order of optRule in the list.
		// We use a bitmask to record which opt rules should be used. If the i-th bit is 1, it means we should
		// apply i-th optimizing rule.
		if flag&(1<<uint(i)) == 0 {
			continue
		}
		logic, err = rule.optimize(ctx, logic)
		if err != nil {
			return nil, err
		}
	}
	return logic, err
}
```

在逻辑优化中，我们会依次遍历优化规则列表，并对当前的执行计划树尝试优化。在 TinySQL 中，我们有

```go
var optRuleList = []logicalOptRule{
	&columnPruner{},
	&buildKeySolver{},
	&aggregationEliminator{},
	&projectionEliminator{},
	&maxMinEliminator{},
	&ppdSolver{},
	&outerJoinEliminator{},
	&aggregationPushDownSolver{},
	&pushDownTopNOptimizer{},
	&joinReOrderSolver{},
}
```

这些规则，我们接下来会对 `columnPruner` 即列裁剪作为例子进行讲解。

每一个优化规则的相关代码都放在了 `planner/core` 下的一个单独文件中，具体来讲，`columnPruner` 的具体实现位于 `planner/core/rule_column_pruning.go` 中。

列裁剪的目的是为了裁掉不需要读取的列，以节约 IO 资源。例如，对于有 a, b, c, d 四列的表 t 来说：

```sql
select a from t where b > 5;
```

这个查询实际上只需要用到 a, b 两列，所以 c, d 两列是不需要读取的，因此可以将他们裁剪掉。

列裁剪的算法实现是自顶向下地把算子过一遍。某个节点需要用到的列，等于它自己需要用到的列，加上它的父节点需要用到的列。可以发现，由上往下的节点，需要用到的列将越来越多。

列裁剪主要影响的算子是 Projection，DataSource，Aggregation，因为它们跟列直接相关。Projection 里面会裁掉用不上的列，DataSource 里面也会裁剪掉不需要使用的列。

Aggregation 算子会涉及哪些列？group by 用到的列，以及聚合函数里面引用到的列。比如 `select avg(a), sum(b) from t group by c d`，这里面 group by 用到的 c 和 d 列，聚合函数用到的 a 和 b 列。所以这个 Aggregation 使用到的就是 a b c d 四列。

Selection 做列裁剪时，要看它父节点要哪些列，然后它自己的条件里面要用到哪些列。Sort 就看 order by 里面用到了哪些列。Join 则要把连接条件中用到的各种列都算进去。具体的代码里面，各个算子都是实现 PruneColumns 接口：

```go
func (p *LogicalPlan) PruneColumns(parentUsedCols []*expression.Column)
```

通过列裁剪这一步操作之后，查询计划里面各个算子，只会记录下它实际需要用到的那些列。

回到代码中，我们在逻辑优化入口处可以得知， `rule.optimize` 是每个规则的入口，对应在这里就是

```go
func (s *columnPruner) optimize(ctx context.Context, lp LogicalPlan) (LogicalPlan, error) {
	err := lp.PruneColumns(lp.Schema().Columns)
	return lp, err
}
```

其中 `lp` 是计划树的根节点，从根节点开始遍历，即自顶向下进行上面描述的算法。

## CBO物理优化

物理优化，即 physical optimize, 入口也是在 `/planner/core/optimizer.go` 中。

[physicalOptimize](https://github.com/pingcap-incubator/tinysql/blob/df75611ce926442bd6074b0f32b1351ca4aad925/planner/core/optimizer.go#L112) 是物理优化的入口。这个过程实际上是一个记忆化搜索的过程。

其记忆化搜索的过程大致可以用如下伪代码表示：

```go
// The OrderProp tells whether the output data should be ordered by some column or expression. (e.g. For select * from t order by a, we need to make the data ordered by column a, that is the exactly information that OrderProp should store)
func findBestTask(p LogicalPlan, prop OrderProp) PhysicalPlan {

	if retP, ok := dpTable.Find(p, prop); ok {
		return retP
	}
	
	selfPhysicalPlans := p.getPhysicalPlans()
	
	bestPlanTree := a plan with maximum cost
	
	for _, pp := range selfPhysicalPlans {
	
		childProps := pp.GetChildProps(prop)
		childPlans := make([]PhysicalPlan, 0, len(p.children))
		for i, cp := range p.children {
			childPlans = append(childPlans, findBestTask(cp, childProps[i])
		}
		physicalPlanTree, cost := connect(pp, childPlans)
		
		if physicalPlanTree.cost < bestPlanTree.cost {
			bestPlanTree = physicalPlanTree
		}
	}
	return bestPlanTree
}
```

实际的执行代码可以在[findBestTask](https://github.com/pingcap-incubator/tinysql/blob/df75611ce926442bd6074b0f32b1351ca4aad925/planner/core/find_best_task.go#L95) 中查看，其逻辑和上述伪代码基本一致。

## 作业

完成 `rule_predicate_push_down.go` 中的 TODO 内容。

## 评分

通过 package `core` 下 `TestPredicatePushDown` 的测试。

你可以在根目录通过命令 `make test-proj4-1` 执行测试。

## 额外内容

如果你对 Cascades 优化器感兴趣，我们也提供了一个不是很完善的作业，你可以尝试完成 `transformation_rules.go` 中的 TODO 内容，其对应的测试在 `transformation_rules_test.go` 中。

# 代价估计和路径选择

## 描述数据分布的数据结构

当结束启发式规则的筛选之后，我们仍然可能剩余多组索引等待筛选我们就需要知道每个索引究竟会过滤多少行数据。在 TiDB 中我们使用直方图和 Count-Min Sketch 来存储的统计信息，[TiDB 源码阅读系列文章（十二）统计信息（上）](https://pingcap.com/blog-cn/tidb-source-code-reading-12/) 中，我们对直方图和 Count-Min Sketch 的实现原理做了比较详细的介绍。

**作业**

这里我们需要完成 `cmsketch.go` 中的 TODO 内容，并通过 `cmsketch_test.go` 中的测试

## Join路径

如果我们不加修改的直接执行用户输入的 Join 的顺序，假设用户输入了 `select * from t1 joni t2 on ... join t3 on ...`，我们会按照先扫描 `t1`，然后和 `t2` 做 Join，然后和 `t3` 做 Join。这个顺序会在很多情况下表现的并不是特别好（一个简单的场景，`t1`, `t2` 特别大，而 `t3` 特别小，那么只要和 `t3` join 时可以过滤部分数据，先 Join `t1` 和 `t2` 势必没有先让 `t3` 和其中一个表 join 在性能上占优势）。

**作业**

你需要实现一个基于 DP 的 Join Reorder 算法，并通过 `rule_join_reorder_dp_test.go` 中的测试。实现的位置在文件 `rule_join_reorder_dp.go` 中。

这里我们简单的表述一下这个算法：

- 使用数字的二进制表示来代表当前参与 Join 的节点情况。11（二进制表示为 1011）表示当前的 Join Group 包含了第 3 号节点，第 1 号节点和第 0 号节点（节点从 0 开始计数）。
- f[11] 来表示包含了节点 `3, 1, 0` 的最优的 Join Tree。
- 转移方程则是 `f[group] = min{join{f[sub], f[group ^ sub])}` 这里 `sub` 是 `group` 二进制表示下的任意子集。

## Skyline pruning

在实际场景中，一个表可能会有多个索引，一个查询中的查询条件也可能涉及到某个表的多个索引。因此就会需要我们去决定选择使用哪个索引。你会需要实现这个过程的某些代码

你可以在 [TiDB proposal](https://github.com/pingcap/tidb/blob/master/docs/design/2019-01-25-skyline-pruning.md) 以及 [Skyline pruning operator](http://www.cs.ust.hk/~dimitris/PAPERS/SIGMOD03-Skyline.pdf) 来了解其理论基础。

**作业**

这是一个启发式规则的筛选，用来筛除一些一定会差的选择分支。具体的筛选要求在 TiDB proposal 以及 `TODO` 注释的解释中有更详细的说明。你需要实现并通过 `TestSkylinePruning` 中的所有测试。实现的位置为 `find_best_task.go` 的 TODO 内容。
