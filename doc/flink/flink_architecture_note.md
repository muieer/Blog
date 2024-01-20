# Flink 架构学习笔记

Flink 是一个框架和分布式计算引擎，用于在无界和有界数据流上进行有状态计算，需要管理和分配计算资源，所以有诸如 Flink Yarn、Flink K8s、Flink Standalone、Flink Local 类型的集群模式。 

## Flink 集群的主要组件

Flink 集群由一个 JobManager 和至少一个 TaskManager 组成。JobManager 的职责是分配和管理计算资源、任务的调度与管理。TaskManager 职责是执行 JobManager 分配的任务。

## 任务和 Operator Chains（算子链）

算子链的每个节点包含一个或多个算子，每个节点的任务数和该节点的并行度一致。算子链的好处是减少缓存和线程切换的负载，增加吞吐和降低延迟。

我觉得算子链可以等价于 Spark 中的 Job，每个节点可以等价于 Spark 中的 Stage，Stage 中任务数与分区数量相关，最大并行度与执行机数量相关。

## JobManager Task Slots（任务槽）和资源的关系

每个 JobManager 有至少一个 slot，slot 均分 JobManager 的内存资源，算子链每个节点的子任务都是分配到 slot 中执行。

Flink 允许 slot 共享，这意味着来自同一个作业的算子链节点对应的子任务都可能在同一个 slot 中执行。slot 共享好处是：提高资源利用率；需要的 slot 数量和最大并行度一致，方便计算需要分配的资源数量。

## Flink 执行时的集群类型
分为 Flink Application Cluster（应用集群）和 Flink Session Cluster（会话集群）。应用集群生命周期和业务相关，资源隔离。<wbr>
会话集群资源共享，一个作业结束后还会继续存活，可以应用在 Jupyter 这种交互式计算产品中。

# 参考链接

- [Flink Architecture](https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/concepts/flink-architecture/), by Flink Documentation