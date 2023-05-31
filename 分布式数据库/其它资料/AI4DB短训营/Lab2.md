## LAB2

### Introduction

在 lab2 的实验中你需要分别用上述两个方法实现 Cost Model，在 `plan.py` 中，实现了 Plan 结构体，能帮你解析如下的 TiDB Plan：

![](http://pic.netpunk.top/images/2022/11/24/20221124173124.png)

为了简单，在 lab2 的实验数据中只会出现 HashAgg, HashJoin, Sort, Selection, Projection, TableReader, TableScan(包含 TableRangeScan, TableFullScan 和 TableRowIDScan), IndexReader, IndexScan(包含 IndexRangeScan 和 IndexFullScan), IndexLookup 这些算子。

在生成 LAB2 的实验数据时，关闭了并发和 Cache，所以设计算子代价公式时只需要考虑算子本身的执行逻辑即可；如为 Projection 设计代价公式时，你可以设计为 `input_rows * cpu_factor` 而不用为 `(input_rows * cpu_factor) / concurrency`。

执行计划及上述算子的解释，可见文档：https://docs.pingcap.com/zh/tidb/dev/explain-overview

训练用和测试用的执行计划被分别放在 `train_plans.json` 和 `test_plans.json` 中。

### The Code

在此实验中请补全 `cost_learning.py` 和 `cost_calibration.py` 中 `YOUR CODE HERE` 的部分。

在 `cost_learning.py` 中，你需要从执行计划中抽取特性，选择合适的机器学习模型，定义损失函数，训练模型预测执行计划的执行时间。

在 `cost_calibration.py` 的注释中定义了一套简单的算子代价公式，你需要实现他们，然后对这套公式进行回归校准；目前为了简单，只考虑 cpu, scan, net, seek 代价。

对于 Plan 的解析代码，都已经被实现在 `plan.py` 中了，你可以直接使用。

lab2 使用 TiDB 校准前的 cost model 作为 baseline，完成所有代码后，请执行 `evaluation.py` 对你的模型进行评估。

评估使用 `est_cost` 和 `act_exec_time` 的相关性作为指标，好的 cost model 会让他俩呈现比较好的正相关，如下：

![](http://pic.netpunk.top/images/2022/11/24/20221124173156.png)

同时会产生一个 `/eval/results.json` 文件，请提交这个文件，classroom 会根据这个文件做一个简单的打分。