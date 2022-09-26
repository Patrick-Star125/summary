# Flink

Apache Flink是一个框架和**分布式处理引擎**，用于对**无界和有界数据流**进行**有状态计算**。Flink被设计成可以在所有常见的**集群环境**中运行，以**内存速度**和**任何规模**进行计算。

也就是说，Flink是一个支持多端部署、流批一体、超大规模和内存级速度的分布式计算系统

## 原理简述

**FlinkClient：** Client 为提交 Job 的客户端，可以是运行在任何机器上（与 JobManager 环境连通即可）。提交 Job 后，Client 可以结束进程（Streaming的任务），也可以不结束并等待结果返回（**MapReduce Client：yarn jar hadoop-mapreduce.jar WordCount input ouput）**

**JobManager：**主要负责调度 Job 并协调 Task 做 checkpoint，从 Client 处接收到 Job 和JAR 包等资源后，会生成优化后的执行计划，并以 Task 为单元调度到各个 TaskManager 去执行。**（ResourceManager和ApplicationMaster或JobTracker）**
**TaskManager：**在启动的时候就设置好了槽位数（Slot），每个 slot 能启动一个 Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立连接，接收数据**（NodeManager或TaskTracker）**



## 特性



## 安装



