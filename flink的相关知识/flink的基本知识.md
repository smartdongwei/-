## 1：flink是什么

​    Flink 是一个针对流数据和批数据的分布式处理引擎，代码主要是由 Java 实现，部分代码是 Scala。它可以处理有界的批量数据集、也可以处理无界的实时数据集。对 Flink 而言，其所要处理的主要场景就是流数据，批数据只是流数据的一个极限特例而已，所以 Flink 也是一款真正的流批统一的计算引擎。

​    Flink 提供了 State、Checkpoint、Time、Window 等，它们为 Flink 提供了基石。



## 2：Flink 整体架构

1. 部署：Flink 支持本地运行（IDE 中直接运行程序）、能在独立集群（Standalone 模式）或者在被 YARN、Mesos、K8s 管理的集群上运行，也能部署在云上。
2. 运行：Flink 的核心是分布式流式数据引擎，意味着数据以一次一个事件的形式被处理。
3. API：DataStream、DataSet、Table、SQL API。
4. 扩展库：Flink 还包括用于 CEP（复杂事件处理）、机器学习、图形处理等场景。

作为一个计算引擎，如果要做的足够完善，除了它自身的各种特点要包含，还得支持各种生态圈，比如部署的情况，Flink 是支持以 Standalone、YARN、Kubernetes、Mesos 等形式部署的。

- Local：直接在 IDE 中运行 Flink Job 时则会在本地启动一个 mini Flink 集群
- Standalone：在 Flink 目录下执行 `bin/start-cluster.sh` 脚本则会启动一个 Standalone 模式的集群
- YARN：YARN 是 Hadoop 集群的资源管理系统，它可以在群集上运行各种分布式应用程序，Flink 可与其他应用并行于 YARN 中，Flink on YARN 的架构如下：

![image-20220105235254682](E:\study\images\image-20220105235254682.png)

- Kubernetes：Kubernetes 是 Google 开源的容器集群管理系统，在 Docker 技术的基础上，为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能，提高了大规模容器集群管理的便捷性，Flink 也支持部署在 Kubernetes 上，在 [GitHub](https://github.com/Aleksandr-Filichkin/flink-k8s/blob/master/flow.jpg) 看到有下面这种运行架构的。



## 3：flink的分布式运行

  Flink 作业提交架构流程可见下图：

![image-20220105235534941](E:\study\images\image-20220105235534941.png)

1. Program Code：我们编写的 Flink 应用程序代码。
2. Job Client：Job Client 不是 Flink 程序执行的内部部分，但它是任务执行的起点。Job Client 负责接受用户的程序代码，然后创建数据流，将数据流提交给 Job Manager 以便进一步执行。执行完成后，Job Client 将结果返回给用户。
3. Job Manager：主进程（也称为作业管理器）协调和管理程序的执行。它的主要职责包括安排任务、管理 checkpoint 、故障恢复等。机器集群中至少要有一个 master，master 负责调度 task、协调 checkpoints 和容灾，高可用设置的话可以有多个 master，但要保证一个是 leader，其他是 standby。Job Manager 包含 Actor system、Scheduler、Check pointing 三个重要的组件。
4. Task Manager：从 Job Manager 处接收需要部署的 Task。Task Manager 是在 JVM 中的一个或多个线程中执行任务的工作节点。任务执行的并行性由每个 Task Manager 上可用的任务槽（Slot 个数）决定。每个任务代表分配给任务槽的一组资源。例如，如果 Task Manager 有四个插槽，那么它将为每个插槽分配 25％ 的内存。可以在任务槽中运行一个或多个线程。同一插槽中的线程共享相同的 JVM。

同一 JVM 中的任务共享 TCP 连接和心跳消息。Task Manager 的一个 Slot 代表一个可用线程，该线程具有固定的内存，注意 Slot 只对内存隔离，没有对 CPU 隔离。默认情况下，Flink 允许子任务共享 Slot，即使它们是不同 task 的 subtask，只要它们来自相同的 job。这种共享可以有更好的资源利用率。

## 4：flink api

Flink 提供了不同的抽象级别的 API 以开发流式或批处理应用。

- 最底层提供了有状态流。它将通过 Process Function 嵌入到 DataStream API 中。它允许用户可以自由地处理来自一个或多个流数据的事件，并使用一致性、容错的状态。除此之外，用户可以注册事件时间和处理事件回调，从而使程序可以实现复杂的计算。
- DataStream/DataSet API 是 Flink 提供的核心 API，DataSet 处理有界的数据集，DataStream 处理有界或者无界的数据流。用户可以通过各种方法（map/flatmap/window/keyby/sum/max/min/avg/join 等）将数据进行转换或者计算。
- Table API 是以表为中心的声明式 DSL，其中表可能会动态变化（在表达流数据时）。Table API 提供了例如 select、project、join、group-by、aggregate 等操作，使用起来却更加简洁（代码量更少）。 你可以在表与 DataStream/DataSet 之间无缝切换，也允许程序将 Table API 与 DataStream 以及 DataSet 混合使用。
- Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与 Table API 类似，但是是以 SQL查询表达式的形式表现程序。SQL 抽象与 Table API 交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。 Flink 除了 DataStream 和 DataSet API，它还支持 Table/SQL API，Flink 也将通过 SQL API 来构建统一的大数据流批处理引擎，因为在公司中通常会有那种每天定时生成报表的需求（批处理的场景，每晚定时跑一遍昨天的数据生成一个结果报表），但是也是会有流处理的场景（比如采用 Flink 来做实时性要求很高的需求），于是慢慢的整个公司的技术选型就变得越来越多了，这样开发人员也就要面临着学习两套不一样的技术框架，运维人员也需要对两种不一样的框架进行环境搭建和作业部署，平时还要维护作业的稳定性。

## 5：Flink 程序与数据流结构

![image-20220106000711047](E:\study\images\image-20220106000711047.png)

一个完整的 Flink 应用程序结构就是如上两图所示：

1. Source：数据输入，Flink 在流处理和批处理上的 source 大概有 4 类：基于本地集合的 source、基于文件的 source、基于网络套接字的 source、自定义的 source。自定义的 source 常见的有 Apache kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可以定义自己的 source。
2. Transformation：数据转换的各种操作，有 Map/FlatMap/Filter/KeyBy/Reduce/Fold/ Aggregations/Window/WindowAll/Union/Window join/Split/Select/Project 等，操作很多，可以将数据转换计算成你想要的数据。
3. Sink：数据输出，Flink 将转换计算后的数据发送的地点，你可能需要存储下来，Flink 常见的 Sink 大概有如下几类：写入文件、打印出来、写入 socket、自定义的 sink 。自定义的 Sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 sink。