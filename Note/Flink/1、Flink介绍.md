## (一)Flink介绍
<!-- TOC -->

- [(一)Flink介绍](#一flink介绍)
  - [Flink是什么](#flink是什么)
  - [为什么选择Flink](#为什么选择flink)
  - [Flink流处理的特征](#flink流处理的特征)
  - [Flink的基石](#flink的基石)
    - [Checkpoint](#checkpoint)
    - [State](#state)
    - [Time](#time)
    - [Window](#window)
  - [组件栈](#组件栈)
  - [Flink应用场景](#flink应用场景)
    - [Event-driven Applications【事件驱动】](#event-driven-applications事件驱动)
    - [Data Analytics Applications【数据分析】](#data-analytics-applications数据分析)
    - [Data Pipeline Applications【数据管道】](#data-pipeline-applications数据管道)
  - [哪些行业需要处理流数据](#哪些行业需要处理流数据)
  - [传统数据处理架构](#传统数据处理架构)
  - [状态化流处理](#状态化流处理)
  - [一些概念](#一些概念)
    - [Dataflow 图](#dataflow-图)
    - [数据并行和任务并行](#数据并行和任务并行)
    - [数据交换策略](#数据交换策略)
    - [并行流处理](#并行流处理)
    - [数据流上的操作](#数据流上的操作)
    - [时间语义](#时间语义)
  - [Flink 的主要特点](#flink-的主要特点)
    - [**事件驱动（Event-driven）**](#事件驱动event-driven)
    - [流与批的世界观](#流与批的世界观)
    - [分层 api](#分层-api)
    - [Flink其他的特点](#flink其他的特点)
    - [Flink vs Spark Streaming](#flink-vs-spark-streaming)
  - [为什么选择Flink](#为什么选择flink-1)
  - [流处理和批处理](#流处理和批处理)
  - [流处理和批处理统一](#流处理和批处理统一)
  - [计算框架发展史](#计算框架发展史)

<!-- /TOC -->

### Flink是什么

Apache Flink 是一个框架和**分布式处理引擎**，用于对**无界和有界**数据流进行状态计算。

### 为什么选择Flink

流数据更真实地反映了我们的生活方式。

传统的数据架构是基于有限数据集的。

我们的目标

- **低延迟**
- **高吞吐**
- **结果的准确性和良好的容错性**

### Flink流处理的特征

Flink 流处理特性：

- 支持**高吞吐、低延迟、高性能**的流处理
- 支持带有**事件时间**的窗口（Window）操作，时间语义丰富。
- 支持**有状态计算的 Exactly-once**语义
- 支持高度灵活的窗口（Window）操作，支持基于 time、count、session，以及 data-driven 的窗口操作
- 支持具有**Backpressure**功能的持续流模型
- 支持基于**轻量级分布式快照**（Snapshot）实现的容错
- 一个运行时同时支持 Batch on Streaming 处理和 Streaming 处理
- Flink 在 JVM 内部实现了自己的**内存管理**
- 支持**迭代计算**
- 支持程序自动优化：避免特定情况下 Shuffle、排序等昂贵操作，中间结果有必要进行缓存。


### Flink的基石

Flink之所以能这么流行，离不开它最重要的四个基石：

- **Checkpoint（检查点）**
- **State（状态）**
- **Time（时间语义）**
- **Window（窗口函数）**

![20211107153219](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211107153219.png)

#### Checkpoint

这是Flink最重要的一个特性。

Flink基于**Chandy-Lamport**算法实现了一个分布式的**轻量级一致性**的快照，从而提供了一致性的语义。

Chandy-Lamport算法实际上在1985年的时候已经被提出来，但并没有被很广泛的应用，而Flink则把这个算法发扬光大了。

Spark最近在实现Continue streaming，Continue streaming的目的是为了降低处理的延时，其也需要提供这种一致性的语义，最终也采用了Chandy-Lamport这个算法，说明Chandy-Lamport算法在业界得到了一定的肯定。

#### State 

提供了一致性的语义之后，Flink为了让用户在编程时能够更轻松、更容易地去管理状态，还提供了一套非常简单明了的State API，包括里面的有ValueState、ListState、MapState，近期添加了BroadcastState，使用State API能够自动享受到这种一致性的语义。

#### Time

除此之外，Flink还实现了Watermark的机制，能够支持基于**事件时间**的处理，能够容忍**迟到/乱序**的数据。

#### Window

另外流计算中一般在对流数据进行操作之前都会先进行开窗，即基于一个什么样的窗口上做这个计算。Flink提供了开箱即用的各种窗口，比如**滑动窗口、滚动窗口、会话窗口以及非常灵活的自定义的窗口**。

### 组件栈

![1621563860910](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/07/153427-665799.png)

**各层详细介绍：**

![20211107153025](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211107153025.png)

- 物理部署层：Flink 支持本地运行、能在独立集群或者在被 YARN 管理的集群上运行， 也能部署在云上，该层主要涉及Flink的部署模式，目前Flink支持多种部署模式：本地、集群(Standalone、YARN)、云(GCE/EC2)、Kubenetes。Flink能够通过该层能够支持不同平台的部署，用户可以根据需要选择使用对应的部署模式。
- Runtime核心层：Runtime层提供了支持Flink计算的全部核心实现，为上层API层提供基础服务，该层主要负责对上层不同接口提供基础服务，也是Flink分布式计算框架的核心实现层，支持分布式Stream作业的执行、JobGraph到ExecutionGraph的映射转换、任务调度等。将DataSteam和DataSet转成统一的可执行的Task Operator，达到在流式引擎下同时处理批量计算和流式计算的目的。
- API&Libraries层：Flink 首先支持了 Scala 和 Java 的 API，Python 也正在测试中。DataStream、DataSet、Table、SQL API，作为分布式数据处理框架，Flink同时提供了支撑计算和批计算的接口，两者都提供给用户丰富的数据处理高级API，例如Map、FlatMap操作等，也提供比较低级的Process Function API，用户可以直接操作状态和时间等底层数据。
- 扩展库：Flink 还包括用于复杂事件处理的CEP，机器学习库FlinkML，图处理库Gelly等。Table 是一种接口化的 SQL 支持，也就是 API 支持(DSL)，而不是文本化的SQL 解析和执行。


### Flink应用场景

#### Event-driven Applications【事件驱动】

事件驱动型应用是一类具有**状态**的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作。

事件驱动型应用是在计算存储分离的传统应用基础上进化而来。
在传统架构中，应用需要读写远程事务型数据库。

相反，事件驱动型应用是基于状态化流处理来完成。在该设计中，数据和计算不会分离，应用只需访问本地(内存或磁盘)即可获取数据。

系统容错性的实现依赖于定期向远程持久化存储写入 checkpoint。下图描述了传统应用和事件驱动型应用架构的区别。

![20211107154139](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211107154139.png)


从某种程度上来说，所有的实时的数据处理或者是流式数据处理都应该是属于Data Driven，流计算本质上是Data Driven 计算。应用较多的如风控系统，当风控系统需要处理各种各样复杂的规则时，Data Driven 就会把处理的规则和逻辑写入到Datastream 的API 或者是ProcessFunction 的API 中，然后将逻辑抽象到整个Flink 引擎，当外面的数据流或者是事件进入就会触发相应的规则，这就是Data Driven 的原理。在触发某些规则后，Data Driven 会进行处理或者是进行预警，这些预警会发到下游产生业务通知，这是Data Driven 的应用场景，Data Driven 在应用上更多应用于复杂事件的处理。

**典型实例：**

- 欺诈检测(Fraud detection)
- 异常检测(Anomaly detection)
- 基于规则的告警(Rule-based alerting)
- 业务流程监控(Business process monitoring)
- Web应用程序(社交网络)

#### Data Analytics Applications【数据分析】

数据分析任务需要从原始数据中提取有价值的信息和指标。

如下图所示，Apache Flink 同时支持流式及批量分析应用。

![20211107154344](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211107154344.png)


Data Analytics Applications包含Batch analytics(批处理分析)和Streaming analytics(流处理分析)
Batch analytics可以理解为周期性查询：Batch Analytics 就是传统意义上使用类似于Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表。比如Flink应用凌晨从Recorded Events中读取昨天的数据，然后做周期查询运算，最后将数据写入Database或者HDFS，或者直接将数据生成报表供公司上层领导决策使用。

Streaming analytics可以理解为连续性查询：比如实时展示双十一天猫销售GMV(Gross Merchandise Volume成交总额)，用户下单数据需要实时写入消息队列，Flink 应用源源不断读取数据做实时计算，然后不断的将数据更新至Database或者K-VStore，最后做大屏实时展示。

**典型实例**

- 电信网络质量监控
- 移动应用中的产品更新及实验评估分析
- 消费者技术中的实时数据即席分析
- 大规模图分析

#### Data Pipeline Applications【数据管道】
什么是数据管道？

提取-转换-加载(ETL)是一种在存储系统之间进行数据转换和迁移的常用方法。

ETL 作业通常会周期性地触发，将数据从事务型数据库拷贝到分析型数据库或数据仓库。

数据管道和 ETL 作业的用途相似，都可以转换、丰富数据，并将其从某个存储系统移动到另一个。

但数据管道是以持续流模式运行，而非周期性触发。

因此数据管道支持从一个不断生成数据的源头读取记录，并将它们以低延迟移动到终点。

例如：数据管道可以用来监控文件系统目录中的新文件，并将其数据写入事件日志；另一个应用可能会将事件流物化到数据库或增量构建和优化查询索引。

和周期性 ETL 作业相比，持续数据管道可以明显降低将数据移动到目的端的延迟。

此外，由于它能够持续消费和发送数据，因此用途更广，支持用例更多。

下图描述了周期性ETL作业和持续数据管道的差异。

![20211107154555](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211107154555.png)

Periodic ETL：比如每天凌晨周期性的启动一个Flink ETL Job，读取传统数据库中的数据，然后做ETL，最后写入数据库和文件系统。

Data Pipeline：比如启动一个Flink 实时应用，数据源(比如数据库、Kafka)中的数据不断的通过Flink Data Pipeline流入或者追加到数据仓库(数据库或者文件系统)，或者Kafka消息队列。
Data Pipeline 的核心场景类似于数据搬运并在搬运的过程中进行部分数据清洗或者处理，而整个业务架构图的左边是Periodic ETL，它提供了流式ETL 或者实时ETL，能够订阅消息队列的消息并进行处理，清洗完成后实时写入到下游的Database或File system 中。

**典型实例**

- 电子商务中的持续 ETL(实时数仓)
当下游要构建实时数仓时，上游则可能需要实时的Stream ETL。这个过程会进行实时清洗或扩展数据，清洗完成后写入到下游的实时数仓的整个链路中，可保证数据查询的时效性，形成实时数据采集、实时数据处理以及下游的实时Query。
- 电子商务中的实时查询索引构建(搜索引擎推荐)
搜索引擎这块以淘宝为例，当卖家上线新商品时，后台会实时产生消息流，该消息流经过Flink 系统时会进行数据的处理、扩展。然后将处理及扩展后的数据生成实时索引，写入到搜索引擎中。这样当淘宝卖家上线新商品时，能在秒级或者分钟级实现搜索引擎的搜索。

### 哪些行业需要处理流数据

![1621659568517](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/143814-152431.png)

电商和市场营销

- 数据报表、广告投放、业务流程需要

物联网（IOT）

- 传感器实时数据采集和显示、实时报警，交通运输业

电信业

- 基站流量调配

银行和金融业

- 实时结算和通知推送，实时检测异常行为

### 传统数据处理架构

**事务处理**

![1614254351391](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/195911-165114.png)

上面一层是计算层，下面层是存储层，**数据计算和数据存储**分开，事务处理就是当前应对的是一个一个的事务，但是事务性数据处理应对不了数据量很大的情况。但是这种情况实时性比较好，但是不能应对高并发，大数据量的情况。

**分析处理**

将数据从业务数据库复制到数仓，再进行分析和查询

![1614254709920](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/200511-798848.png)

将数据从数据库中取出先做ETL数据清洗，然后在放到数据仓库中，然后用分析计算的引擎计算处理，数据的来源可以从不同的数据库，统一处理后放到数据仓库，但是这个过程很慢，需要离线处理，不能做到事实处理。可以做到应对大数据量，但是不能做到实时处理。

**第一代：有状态的流式处理**

比如storm，低延迟做的很好，但是吞吐量不是很大，保证不了正确，数据乱序结果不能保证。

Spark Streaming:可以保证高吞吐，在高压下保证结果正确，容错性也很好，但是不能保证低延迟，对乱序数据处理不是很好。

![1614255372417](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082406-103256.png)

这种数据的处理方式无法保证数据的来的顺序关系，所以产生第二代流式处理。

**第二代：lambda 架构**

用两套系统，同时保证低延迟和结果准确

![1614255489481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/201809-596238.png)

事件日志，每一条日志对应一个事务的发生，Batch Layer是批处理层，Stream Processor是流处理层。用这两套系统同时保证结果的低延迟和准确性。Stream Processor层保证的是速度，但是存在的问题就是系统太麻烦，一个处理需要实现两套系统，代价很高。

**第三代流处理**

![1614255866157](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082410-502959.png)

flink可以保证低延迟，高吞吐量，容错性比较好。

### 状态化流处理

几乎所有数据都是以连续事件流的形式产生。请考虑一下，无论是网站或移动应用中的用户交互或订单下达，还是服务器日志或传感器测量结果，这些数据本质上都是事件流。事实上，现实世界中很难找到那种瞬间就生成完整数据集的例子。作为一类面向无限事件流的应用设计模式，状态化流处理适用于企业 IT 基础设施中的很多应用场景。 

任何一个处理事件流的应用，**如果要支持跨多条记录的转换操作，都必须是有状态的，即能够存储和访问中间结果**。应用收到事件后可以执行包括读写状态在内的任意计算。原则上，需要在应用中访问的状态有多种可选的存储位置，例如:程序变量、本地文件、嵌入式或外部数据库等。 

Apache Flink 会将应用状态存储在**本地内存或嵌入式数据库**中。由 于采用的是分布式架构，Flink需要对本地状态予以保护，以避免因应用或机器故障导致数据丢失。为了实现该特性， **Flink 会定期将应用状态的一致性检查点(checkpoint) 写入远程持久化存储**。

![1614424052530](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/190737-12284.png)

有状态的流处理应用通常会从事件日志中读取事件记录。事件日志负责存储事件流并将其分布式化。由于事件只 能以追加的形式写入持久化日志中，所以其顺序无法在后期改变 。写入事件日志的数据流可以被相同或不同 的消费者重复读取。得益于日志的追加特性，无论向消费者发布几次，事件的顺序都能保持一致。有不少事件日志系统都是开惊软件，其中最流行的当属Apache Kafka ，也有部分系统会以云计算提供商集成服务的形式提供。 

出于很多原因，将运行在 Flink 之上的有状态的流处理应用和事件日志系统相连会很有意义。在该架构下，事件日志系统可以持久化输入事件并以确定的顺序将其重放 。一旦出现故障， Flink 会利用之前的检查点恢复状态并重置事件日志的读取位置，以此来使有状态的流处理应用恢复正常。随后应用会从 事件日志中读取井(快速)重放输入事件，直到追赶上数据流当前的进度。 

### 一些概念

#### Dataflow 图

Dataflow 程序描述了数据如何在不同操作之间流动。Dataflow 程序通常表示为有向图。图中**顶点称为算子，表示计算**， **而边表示数据依赖关系**。算子是Dataflow 程序的基本功能单元，它们从输入获取数据，对其进行计算，然后产生数据并发往输出以供后续处理。没有输入端的算子称为数据源，没有输出端的算子称为数据汇。一个Dataflow 图至少要有一个数据源和一个数据汇

**逻辑图**

![1614346618229](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082415-248304.png)

类似图2- 1 的Dataflow 图被称作逻辑图，因为它们表达了高层视角下的计算逻辑。为了执行Dataflow 程序，需要将逻辑图转化为物理Dataflow 图，后者会指定程序的执行细节。例如: 当我们使用分布式处理引擎时，**每个算子可能会在不同物理机器上运行多个并行任务**。图2-2 展示了图2-1 中逻辑图所对应的物理Dataflow 图。在逻辑Dataflow 图中，**顶点代表算子;在物理Dataflow 图中，顶点代表任务**。**"抽取主题标签"和"计数"算子都包含两个并行算子任务，每个任务负责计算一部分输入数据。**

也就是说一个算子可以分解为多个并行执行的任务。

**物理DataFlow图**

![1614346702527](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/213823-83256.png)

#### 数据并行和任务并行

Dataflow 图的并行性可以通过多种方式加以利用。首先，你可以将输入**数据分组**，让同一操作的多个任务并行执行在不同数据子集上，这种并行称为数据并行(data parallelism) 。数据井行非常有用，因为它能够将计算负载分配到多个节点上从而允许处理大规模的数据。再者，你可以让不同算子的任务(基于相同或不同的数据)并行计算，这种并行称为任务并行( task parallelism) 。通过任务并行，可以更好地利用集群的计算资源。

#### 数据交换策略

数据交换策略定义了如何将数据项分配给物理Dataflow 图中的不同任务。这些策略可以由执行引擎根据算子的语义自动选择，也可以由Dataflow 编程人员显式指定。接下来，我们结合图2-3 来简单了解一下常见的数据交换策略。

![1614346967396](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/082416-355216.png)

- 转发策略(forward strategy) 在发送端任务和接收端任务之间一对一地进行数据传输。如果两端任务运行在同一物理机器上(通常由任务调度器决定) ，该交换策略可以避免网络通信。
- 广播策略(broadcast strategy) 会把每个数据项发往下游算子的全部并行任务。该策略会把数据复制多份且涉及网络通信，因此代价十分昂贵。
- 基于键值的策略(key-based strategy) 根据某一键值属性对数据分区，并保证键值相同的数据项会交由同一任务处理。图2- 2 中， "抽取主题标签"算子的输出就是按照键值(主题标签)划分的，因此下游的计数算子可以正确计算出每个主题标签的出现次数。
- 随机策略(random strategy) 会将数据均句分配至算子的所有任务，以实现计算任务的负载均衡。

#### 并行流处理

数据流的定义: 数据流是一个可能无限的事件序列。数据流中的事件可以表示监控数据、传感器测量值、信用卡交易、气象站观测数据、在线用户交互，以及网络搜索等。

**延迟**

表示处理一个事件所需要的时间，在流式应用中，延时是以时间片为单位测量。

**吞吐**

吞吐量是用来衡量系统处理能力的指标，他告诉我们系统每单位时间可以处理多少事件。

> 其实上面的两个指标相互影响

#### 数据流上的操作

流处理引擎通常会提供一系列内置操作来实现数据流的获取、转换，以及输出。这些算子可以组合生成 Dataflow 处理图，从而实现流式应用所需的逻辑。 

- 数据接入和数据输出
- 转换操作
  - 转换操作是一类"只过一次"的操作，它们会分别处理每个事件。这些操作逐个读取事件，对其应用某些转换并产生一条新的输出流。
- 滚动聚合
  - 滚动聚合(如求和、求最小值和求最大值)会根据每个到来的事件持续更新结果。聚合操作都是有状态的，它们通过将新到来的事件合并到已有状态来生成更新后的聚合值
- 窗口操作

#### 时间语义

**处理时间**

处理时间是当前流处理算子所在机器上的本地时钟时间 。

**事件时间**

事件时间是数据流中事件实际发生的时间，它以附加在数据流中事件的时间戳为依据 。
**摄入时间**

摄入时间是当前事件进入Flink系统的时间。

### Flink 的主要特点

#### **事件驱动（Event-driven）**

Flink处理数据和传统的事件驱动型数据处理很像，先读取事件日志，然后存储到本地状态上面，如果要保证容错性，还可以定期进行存盘保存在磁盘中，最后输出到日志文件或者触发操作。

事件驱动型应用是一类具有**状态**的应用，它从一个或多个事件流提取数据，并根据到来的事件触发计算、状态更新或其他外部动作。比较典型的就是以 kafka 为代表的消息队列几乎都是事件驱动型应用。与之不同的就是 SparkStreaming 微批次，如图：

![1614327856778](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222645-348669.png)

**事件驱动型：**

![1614256146967](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/202908-84379.png)

#### 流与批的世界观

批处理的特点是**有界、持久、大量**，非常适合需要访问全套记录才能完成的计算工作，一般用于离线统计。

流处理的特点是**无界、实时**, 无需针对整个数据集执行操作，而是对通过系统传输的每个数据项执行操作，一般用于实时统计。

在 spark 的世界观中，一切都是由批次组成的，**离线数据是一个大批次，而实时数据是由一个一个无限的小批次组成的**。

而在 flink 的世界观中，**一切都是由流组成的，离线数据是有界限的流，实时数据是一个没有界限的流，这就是所谓的有界流和无界流**。

**无界数据流：**

无界数据流有一个开始但是没有结束，它们不会在生成时终止并提供数据，必须连续处理无界流，也就是说必须在获取后立即处理 event。对于无界数据流我们无法等待所有数据都到达，因为输入是无界的，并且在任何时间点都不会完成。处理无界数据通常要求以特定顺序（例如事件发生的顺序）获取 event，以便能够推断结果完整性。

 **有界数据流：**

有界数据流有明确定义的开始和结束，可以在执行任何计算之前通过获取所有数据来处理有界流，处理有界流不需要有序获取，因为可以始终对有界数据集进行排序，有界流的处理也称为批处理。

![1614331802959](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222640-998093.png)

这种以流为世界观的架构，获得的最大好处就是具有极低的延迟。

#### 分层 api

- 越顶层越抽象，表达含义越简明，使用越方便
- 越底层越具体，表达能力越丰富，使用越灵活

![1614331948887](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/173514-873064.png)

最底层级的抽象仅仅提供了有状态流，它将通过过程函数（Process Function）被嵌入到 DataStream API 中。底层过程函数（Process Function） 与 DataStream API相集成，使其可以对某些特定的操作进行底层的抽象，它允许用户可以自由地处理来自一个或多个数据流的事件，并使用一致的容错的状态。除此之外，用户可以注册事件时间并处理时间回调，从而使程序可以处理复杂的计算。

实际上，大多数应用并不需要上述的底层抽象，而是针对核心 API（Core APIs）进行编程，比如 DataStream API（有界或无界流数据）以及 DataSet API（有界数据集）。这些 API 为数据处理提供了通用的构建模块，比如由用户定义的多种形式的转换（transformations），连接（joins），聚合（aggregations），窗口操作（windows）等等。DataSet API 为有界数据集提供了额外的支持，例如循环与迭代。这些 API处理的数据类型以类（classes）的形式由各自的编程语言所表示。

Table API 是以表为中心的声明式编程，其中表可能会动态变化（在表达流数据时）。Table API 遵循（扩展的）关系模型：表有二维数据结构（schema）（类似于关系数据库中的表），同时 API 提供可比较的操作，例如 select、project、join、group-by、aggregate 等。Table API 程序声明式地定义了什么逻辑操作应该执行，而不是准确地确定这些操作代码的看上去如何。

尽管 Table API 可以通过多种类型的用户自定义函数（UDF）进行扩展，其仍不如核心 API 更具表达能力，但是使用起来却更加简洁（代码量更少）。除此之外，Table API 程序在执行之前会经过内置优化器进行优化。你可以在表与 DataStream/DataSet 之间无缝切换，以允许程序将 Table API 与DataStream 以及 DataSet 混合使用。

Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与Table API 类似，但是是以 SQL 查询表达式的形式表现程序。SQL 抽象与 Table API交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

目前 Flink 作为批处理还不是主流，不如 Spark 成熟，所以 DataSet 使用的并不是很多。Flink Table API 和 Flink SQL 也并不完善，大多都由各大厂商自己定制。所以我们主要学习 DataStream API 的使用。实际上 Flink 作为最接近 Google DataFlow模型的实现，是流批统一的观点，所以基本上使用 DataStream 就可以了。

**Flink 几大模块**

- Flink Table & SQL(还没开发完)
- Flink Gelly(图计算)
- Flink CEP(复杂事件处理)

#### Flink其他的特点

- 支持事件时间（event-time）和处理时间（processing-time）和摄入时间三大语义。
- 精确一次（exactly-once）的状态一致性保证，出现故障之后可以恢复到原来的状态。
- 低延迟，每秒处理数百万个事件，毫秒级延迟，低延迟，高吞吐量
- 与众多常用存储系统的连接
- 高可用，动态扩展，实现7*24小时全天候运行

#### Flink vs Spark Streaming

流（stream）和微批（micro-batching）

![1614332595714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/174318-581777.png)

sparking streaming是批处理，只是每一批数据积攒的很小，而flink是真正意义上的流式处理，从架构上来说，spark就是批处理的，所以存在延迟，不管批次有多小，总存在延迟发生，但是flink底层就是流处理，两个框架的底层架构不同

**底层数据模型的区别**

- 数据模型
  - spark 采用 RDD 模型，spark streaming 的 DStream 实际上也就是一组组小批数据 RDD 的集合
  - flink 基本数据模型是数据流（dataflow），以及事件（Event）序列,处理的是一条一条的数据
- 运行时架构
  - spark 是批计算，将 DAG 划分为不同的 stage，一个完成后才可以计算下一个，所以阶段之间存在依赖性，上一个阶段如果没有完成的话，就不能进行下一个阶段。
  - flink是标准的流执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理，中间没有阶段的划分，所以也没有延迟。

### 为什么选择Flink

**主要原因**

1. Flink 具备统一的框架处理有界和无界两种数据流的能力
2. 部署灵活，Flink 底层支持多种资源调度器，包括Yarn、Kubernetes 等。Flink 自身带的Standalone 的调度器，在部署上也十分灵活。
3. 极高的可伸缩性，可伸缩性对于分布式系统十分重要，阿里巴巴双11大屏采用Flink 处理海量数据，使用过程中测得Flink 峰值可达17 亿条/秒。
4. 极致的流式处理性能。Flink 相对于Storm 最大的特点是将状态语义完全抽象到框架中，支持本地状态读取，避免了大量网络IO，可以极大提升状态存取的性能。

 **其他更多的原因:**

1. 同时支持高吞吐、低延迟、高性能
2. Flink 是目前开源社区中唯一一套集**高吞吐、低延迟、高性能**三者于一身的分布式流式数据处理框架。
3. Spark 只能兼顾高吞吐和高性能特性，无法做到低延迟保障,因为Spark是用批处理来做流处理
4. Storm 只能支持低延时和高性能特性，无法满足高吞吐的要求

**支持事件时间(Event Time)概念**

在流式计算领域中，窗口计算的地位举足轻重，但目前大多数框架窗口计算采用的都是系统时间(Process Time)，也就是事件传输到计算框架处理时，系统主机的当前时间。

Flink 能够支持基于事件时间(Event Time)语义进行窗口计算。

这种基于事件驱动的机制使得事件即使乱序到达甚至延迟到达，流系统也能够计算出精确的结果，保持了事件原本产生时的时序性，尽可能避免网络传输或硬件系统的影响。

![1621660006900](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165452-629379.png)

**支持有状态计算**

Flink1.4开始支持有状态计算

所谓状态就是在流式计算过程中将算子的中间结果保存在内存或者文件系统中，等下一个事件进入算子后可以从之前的状态中获取中间结果，计算当前的结果，从而无须每次都基于全部的原始数据来统计结果，极大的提升了系统性能，状态化意味着应用可以维护随着时间推移已经产生的数据聚合

![1621660055700](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165454-678728.png)

**支持高度灵活的窗口(Window)操作**

Flink 将窗口划分为基于 Time 、Count 、Session、以及Data-Driven等类型的窗口操作，窗口可以用灵活的触发条件定制化来达到对复杂的流传输模式的支持，用户可以定义不同的窗口触发机制来满足不同的需求

**基于轻量级分布式快照(Snapshot/Checkpoints)的容错机制**

Flink 能够分布运行在上千个节点上，通过基于分布式快照技术的Checkpoints，将执行过程中的状态信息进行持久化存储，一旦任务出现异常停止，Flink 能够从 Checkpoints 中进行任务的自动恢复，以确保数据处理过程中的一致性

Flink 的容错能力是轻量级的，允许系统保持高并发，同时在相同时间内提供强一致性保证。

![1621660134121](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165455-500063.png)

**基于 JVM 实现的独立的内存管理**

Flink 实现了自身管理内存的机制，通过使用散列，索引，缓存和排序有效地进行内存管理，通过序列化/反序列化机制将所有的数据对象转换成二进制在内存中存储，降低数据存储大小的同时，更加有效的利用空间。使其独立于 Java 的默认垃圾收集器，尽可能减少 JVM GC 对系统的影响。

 **SavePoints 保存点**

对于 7 * 24 小时运行的流式应用，数据源源不断的流入，在一段时间内应用的终止有可能导致数据的丢失或者计算结果的不准确。

比如集群版本的升级，停机运维操作等。

值得一提的是，Flink 通过SavePoints 技术将任务执行的快照保存在存储介质上，当任务重启的时候，可以从事先保存的 SavePoints 恢复原有的计算状态，使得任务继续按照停机之前的状态运行。

Flink 保存点提供了一个状态化的版本机制，使得能以无丢失状态和最短停机时间的方式更新应用或者回退历史数据。

![1621660228183](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/142956-940682.png)

**灵活的部署方式，支持大规模集群**

Flink 被设计成能用上千个点在大规模集群上运行

除了支持独立集群部署外，Flink 还支持 YARN 和Mesos 方式部署。

**Flink 的程序内在是并行和分布式的**

数据流可以被分区成 stream partitions，

operators 被划分为operator subtasks; 

这些 subtasks 在不同的机器或容器中分不同的线程独立运行；

operator subtasks 的数量就是operator的并行计算数，不同的 operator 阶段可能有不同的并行数；

如下图所示，source operator 的并行数为 2，但最后的 sink operator 为1；

![1621660321711](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165458-516420.png)

### 流处理和批处理

**数据的时效性**

日常工作中，我们一般会先把数据存储在**表**，然后对表的数据进行**加工、分析**。既然先存储在表中，那就会涉及到时效性概念。

如果我们处理以年，月为单位的级别的数据处理，进行**统计分析，个性化推荐**，那么数据的的最新日期离当前有**几个甚至上月**都没有问题。但是如果我们处理的是**以天为级别**，或者**一****小时**甚至**更小粒度**的数据处理，那么就要求数据的时效性更高了。比如：

- 对网站的实时监控
- 对异常日志的监控

这些场景需要工作人员**立即响应**，这样的场景下，传统的统一收集数据，再存到数据库中，再取出来进行分析就无法满足高时效性的需求了。

**流式计算和批量计算**

![1621660594100](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165502-264342.png)

- Batch Analytics，右边是 Streaming Analytics。批量计算: 统一收集数据->存储到DB->对数据进行批量处理，就是传统意义上使用类似于 Map Reduce、Hive、Spark Batch 等，对作业进行分析、处理、生成离线报表
- Streaming Analytics 流式计算，顾名思义，就是对数据流进行处理，如使用流式分析引擎如 Storm，Flink 实时处理分析数据，应用较多的场景如实时大屏、实时报表。

**它们的主要区别是：**

- 与批量计算那样慢慢积累数据不同，流式计算立刻计算，数据持续流动，计算完之后就丢弃。
- 批量计算是维护一张表，对表进行实施各种计算逻辑。流式计算相反，是必须先定义好计算逻辑，提交到流式计算系统，这个计算作业逻辑在整个运行期间是不可更改的。
- 计算结果上，批量计算对全部数据进行计算后传输结果，流式计算是每次小批量计算后，结果可以立刻实时化展现。

**对比**

![1621749815430](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/140337-942786.png)

### 流处理和批处理统一

在大数据处理领域，批处理任务与流处理任务一般被认为是两种不同的任务，一个大数据框架一般会被设计为只能处理其中一种任务：

- MapReduce只支持批处理任务；
- Storm只支持流处理任务；
- Spark Streaming采用micro-batch架构，本质上还是基于Spark批处理对流式数据进行处理
- Flink通过灵活的执行引擎，能够同时支持批处理任务与流处理任务

![1621660852208](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/165504-865466.png)

在执行引擎这一层，流处理系统与批处理系统最大不同在于节点间的数据传输方式：

1. 对于一个流处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，然后立刻通过网络传输到下一个节点，由下一个节点继续处理
2. 对于一个批处理系统，其节点间数据传输的标准模型是：当一条数据被处理完成后，序列化到缓存中，并不会立刻通过网络传输到下一个节点，当缓存写满，就持久化到本地硬盘上，当所有数据都被处理完成后，才开始将处理后的数据通过网络传输到下一个节点

这两种数据传输模式是两个极端，对应的是流处理系统对低延迟的要求和批处理系统对高吞吐量的要求

Flink的执行引擎采用了一种十分灵活的方式，同时支持了这两种数据传输模型：

Flink以固定的缓存块为单位进行网络数据传输，用户可以通过设置缓存块超时值指定缓存块的传输时机。

如果缓存块的超时值为0，则Flink的数据传输方式类似上文所提到流处理系统的标准模型，此时系统可以获得最低的处理延迟

如果缓存块的超时值为无限大，则Flink的数据传输方式类似上文所提到批处理系统的标准模型，此时系统可以获得最高的吞吐量

同时缓存块的超时值也可以设置为0到无限大之间的任意值。缓存块的超时阈值越小，则Flink流处理执行引擎的数据处理延迟越低，但吞吐量也会降低，反之亦然。通过调整缓存块的超时阈值，用户可根据需求灵活地权衡系统延迟和吞吐量

默认情况下，流中的元素并不会一个一个的在网络中传输，而是缓存起来伺机一起发送(默认为32KB，通过taskmanager.memory.segment-size设置),这样可以避免导致频繁的网络传输,提高吞吐量，但如果数据源输入不够快的话会导致后续的数据处理延迟，所以可以使用env.setBufferTimeout(默认100ms)，来为缓存填入设置一个最大等待时间。等待时间到了之后，即使缓存还未填满，缓存中的数据也会自动发送。 

- timeoutMillis > 0 表示最长等待 timeoutMillis 时间，就会flush
- timeoutMillis = 0 表示每条数据都会触发 flush，直接将数据发送到下游，相当于没有Buffer了(避免设置为0，可能导致性能下降)
- timeoutMillis = -1 表示只有等到 buffer满了或 CheckPoint的时候，才会flush。相当于取消了 timeout 策略

**总结:**

Flink以缓存块为单位进行网络数据传输,用户可以设置缓存块超时时间和缓存块大小来控制缓冲块传输时机,从而控制Flink的延迟性和吞吐量

### 计算框架发展史

![1621660996990](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/07/154941-974650.png)

这几年大数据的飞速发展，出现了很多热门的开源社区，其中著名的有Hadoop、Storm，以及后来的 Spark，他们都有着各自专注的应用场景。Spark 掀开了内存计算的先河，也以内存为赌注，赢得了内存计算的飞速发展。Spark 的火热或多或少的掩盖了其他分布式计算的系统身影。就像Flink，也就在这个时候默默的发展着。

在国外一些社区，有很多人将大数据的计算引擎分成了 4 代，当然，也有很多人不会认同。我们先姑且这么认为和讨论。

1. 第1代——Hadoop MapReduce

首先第一代的计算引擎，无疑就是 Hadoop 承载的 MapReduce。它将计算分为两个阶段，分别为 Map 和 Reduce。对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个 Job 的串联，以完成一个完整的算法，例如迭代计算。

- 批处理
- Mapper、Reducer

2. 第2代——DAG框架（Tez） + MapReduce

由于这样的弊端，催生了支持 DAG 框架的产生。因此，支持 DAG 的框架被划分为第二代计算引擎。如 Tez 以及更上层的 Oozie。这里我们不去细究各种 DAG 实现之间的区别，不过对于当时的 Tez 和 Oozie 来说，大多还是批处理的任务。

- 批处理
- 1个Tez = MR(1) + MR(2) + ... + MR(n)
- 相比MR效率有所提升

![1621661167571](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/07/154943-879952.png)

3. 第3代——Spark

接下来就是以 Spark 为代表的第三代的计算引擎。第三代计算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及强调的实时计算。在这里，很多人也会认为第三代计算引擎也能够很好的运行批处理的 Job。

- 批处理、流处理、SQL高层API支持
- 自带DAG
- 内存迭代计算、性能较之前大幅提升

4. 第4代——Flink

随着第三代计算引擎的出现，促进了上层应用快速发展，例如各种迭代计算的性能以及对流计算和 SQL 等的支持。Flink 的诞生就被归在了第四代。这应该主要表现在 Flink 对流计算的支持，以及更一步的实时性上面。当然 Flink 也可以支持 Batch 的任务，以及 DAG 的运算。

- 批处理、流处理、SQL高层API支持
- 自带DAG
- 流式计算性能更高、可靠性更高