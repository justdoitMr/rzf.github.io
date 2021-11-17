## Flink学习--Flink on Yarn

### Flink整体架构

Flink 的整体架构如下图所示。Flink 是可以运行在多种不同的环境中的，例如，它可以通过单进程多线程的方式直接运行，从而提供调试的能力。它也可以运行在 Yarn 或者 K8S 这种资源管理系统上面，也可以在各种云环境中执行。

![1637128195593](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/134957-207852.png)

> 其中 Runtime 层对不同的执行环境提供了一套统一的分布式执行引擎。

**针对不同的执行环境，Flink 提供了一套统一的分布式作业执行引擎**，也就是 Flink Runtime 这层。Flink 在 Runtime 层之上提供了 DataStream 和 DataSet 两套 API，分别用来编写流作业与批作业，以及一组更高级的 API 来简化特定作业的编写。本文主要介绍 Flink Runtime 层的整体架构。

Flink Runtime 层的主要架构如下图 所示，它展示了一个 Flink 集群的基本结构。Flink Runtime 层的整个架构主要是在 FLIP-6 中实现的，整体来说，它采用了标准 master-slave 的结构，其中左侧白色圈中的部分即是 **master**，它负责管理整个集群中的资源和作业；而右侧的两个 **TaskExecutor** 则是 Slave，负责提供具体的资源并实际执行作业。

![1637128348362](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/135229-719647.png)

其中，Master 部分又包含了三个组件，即

- Dispatcher
- ResourceManager 
- JobManager

其中，**Dispatcher 负责接收用户提供的作业，并且负责为这个新提交的作业拉起一个新的 JobManager 组件。ResourceManager 负责资源的管理，在整个 Flink 集群中只有一个 ResourceManager。JobManager 负责管理作业的执行，在一个 Flink 集群中可能有多个作业同时执行，每个作业都有自己的 JobManager 组件**。这三个组件都包含在 AppMaster 进程中，这三个组件在一起是Flink中的Master组件。

基于上述结构，当用户提交作业的时候，提交脚本会首先启动一个 Client进程负责作业的编译与提交。它首先将用户编写的代码编译为一个 JobGraph，在这个过程，它还会进行一些检查或优化等工作，例如判断哪些 Operator 可以 Chain 到同一个 Task 中。然后，Client 将产生的 JobGraph 提交到集群中执行。此时有两种情况：

1. 一种是类似于 Standalone 这种 Session 模式，AM 会预先启动，此时 Client 直接与 Dispatcher 建立连接并提交作业

2. 另一种是 Per-Job 模式，AM 不会预先启动，此时 Client 将首先向资源管理系统 （如Yarn、K8S）申请资源来启动 AM，然后再向 AM 中的 Dispatcher 提交作业。

当作业到 Dispatcher 后，Dispatcher 会首先启动一个 JobManager 组件，然后 JobManager 会向 Flink ResourceManager 申请资源来启动作业中具体的任务。这时根据 Session 和 Per-Job 模式的区别， TaskExecutor 可能已经启动或者尚未启动：

- 如果是Session模式，此时 ResourceManager 中已有记录了 TaskExecutor 注册的资源，可以直接选取空闲资源进行分配。
- 否则如果是Per-Job模式，Flink ResourceManager 也需要首先向外部资源管理系统申请资源来启动 TaskExecutor，然后等待 TaskExecutor 注册相应资源后再继续选择空闲资源进程分配。

目前 Flink 中 TaskExecutor 的资源是通过 Slot 来描述的，一个 Slot 一般可以执行一个具体的 Task，但在一些情况下也可以执行多个相关联的 Task，这部分内容将在下文进行详述。ResourceManager 选择到空闲的 Slot 之后，就会通知相应的 TM “将该 Slot 分配分 JobManager XX ”，然后 TaskExecutor 进行相应的记录后，会向 JobManager 进行注册。JobManager 收到 TaskExecutor 注册上来的 Slot 后，就可以实际提交 Task 了。

TaskExecutor 收到 JobManager 提交的 Task 之后，会启动一个新的线程来执行该 Task。Task 启动后就会开始进行预先指定的计算，并通过数据 Shuffle 模块互相交换数据。

以上就是 Flink Runtime 层执行作业的基本流程。可以看出，Flink 支持两种不同的模式，即 Per-job 模式与 Session 模式。，Per-job 模式下整个 Flink 集群只执行单个作业，即每个作业会独享 Dispatcher 和 ResourceManager 组件。此外，Per-job 模式下 AppMaster 和 TaskExecutor 都是按需申请的。因此，Per-job 模式更适合运行执行时间较长的大作业，这些作业对稳定性要求较高，并且对申请资源的时间不敏感。与之对应，在 Session 模式下，Flink 预先启动 AppMaster 以及一组 TaskExecutor，然后在整个集群的生命周期中会执行多个作业。可以看出，Session 模式更适合规模小，执行时间短的作业。

![1637128980581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/140301-936697.png)

**资源管理和作业调度**

本节对 Flink 中资源管理与作业调度的功能进行更深入的说明。实际上，作业调度可以看做是对资源和任务进行匹配的过程。如上节所述，在 Flink 中，资源是通过 Slot 来表示的，每个 Slot 可以用来执行不同的 Task。而在另一端，任务即 Job 中实际的 Task，它包含了待执行的用户逻辑。调度的主要目的就是为了给 Task 找到匹配的 Slot。逻辑上来说，每个 Slot 都应该有一个向量来描述它所能提供的各种资源的量，每个 Task 也需要相应的说明它所需要的各种资源的量。但是实际上在 1.9 之前，Flink 是不支持细粒度的资源描述的，而是统一的认为每个 Slot 提供的资源和 Task 需要的资源都是相同的。从 1.9 开始，Flink 开始增加对细粒度的资源匹配的支持的实现，但这部分功能目前仍在完善中。

　　作业调度的基础是首先提供对资源的管理，因此我们首先来看下 Flink 中资源管理的实现。如上文所述，Flink 中的资源是由 TaskExecutor 上的 Slot 来表示的。如图 4 所示，在 ResourceManager 中，有一个子组件叫做 SlotManager，它维护了当前集群中所有 TaskExecutor 上的 Slot 的信息与状态，如该 Slot 在哪个 TaskExecutor 中，该 Slot 当前是否空闲等。当 JobManger 来为特定 Task 申请资源的时候，根据当前是 Per-job 还是 Session 模式，ResourceManager 可能会去申请资源来启动新的 TaskExecutor。当 TaskExecutor 启动之后，它会通过服务发现找到当前活跃的 ResourceManager 并进行注册。在注册信息中，会包含该 TaskExecutor中所有 Slot 的信息。 ResourceManager 收到注册信息后，其中的 SlotManager 就会记录下相应的 Slot 信息。当 JobManager 为某个 Task 来申请资源时， SlotManager 就会从当前空闲的 Slot 中按一定规则选择一个空闲的 Slot 进行分配。当分配完成后，如第 2 节所述，RM 会首先向 TaskManager 发送 RPC 要求将选定的 Slot 分配给特定的 JobManager。TaskManager 如果还没有执行过该 JobManager 的 Task 的话，它需要首先向相应的 JobManager 建立连接，然后发送提供 Slot 的 RPC 请求。在 JobManager 中，所有 Task 的请求会缓存到 SlotPool 中。当有 Slot 被提供之后，SlotPool 会从缓存的请求中选择相应的请求并结束相应的请求过程。



![1637130632024](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/143046-241568.png)

当 Task 结束之后，无论是正常结束还是异常结束，都会通知 JobManager 相应的结束状态，然后在 TaskManager 端将 Slot 标记为已占用但未执行任务的状态。JobManager 会首先将相应的 Slot 缓存到 SlotPool 中，但不会立即释放。这种方式避免了如果将 Slot 直接还给 ResourceManager，在任务异常结束之后需要重启时，需要立刻重新申请 Slot 的问题。通过延时释放，Failover 的 Task 可以尽快调度回原来的 TaskManager，从而加快 Failover 的速度。当 SlotPool 中缓存的 Slot 超过指定的时间仍未使用时，SlotPool 就会发起释放该 Slot 的过程。与申请 Slot 的过程对应，SlotPool 会首先通知 TaskManager 来释放该 Slot，然后 TaskExecutor 通知 ResourceManager 该 Slot 已经被释放，从而最终完成释放的逻辑。

　　除了正常的通信逻辑外，在 ResourceManager 和 TaskExecutor 之间还存在定时的心跳消息来同步 Slot 的状态。在分布式系统中，消息的丢失、错乱不可避免，这些问题会在分布式系统的组件中引入不一致状态，如果没有定时消息，那么组件无法从这些不一致状态中恢复。此外，当组件之间长时间未收到对方的心跳时，就会认为对应的组件已经失效，并进入到 Failover 的流程。

　　在 Slot 管理基础上，Flink 可以将 Task 调度到相应的 Slot 当中。如上文所述，Flink 尚未完全引入细粒度的资源匹配，默认情况下，每个 Slot 可以分配给一个 Task。但是，这种方式在某些情况下会导致资源利用率不高。如图 5 所示，假如 A、B、C 依次执行计算逻辑，那么给 A、B、C 分配分配单独的 Slot 就会导致资源利用率不高。为了解决这一问题，Flink 提供了 Share Slot 的机制。如图 5 所示，基于 Share Slot，每个 Slot 中可以部署来自不同 JobVertex 的多个任务，但是不能部署来自同一个 JobVertex 的 Task。如图5所示，每个 Slot 中最多可以部署同一个 A、B 或 C 的 Task，但是可以同时部署 A、B 和 C 的各一个 Task。当单个 Task 占用资源较少时，Share Slot 可以提高资源利用率。 此外，Share Slot 也提供了一种简单的保持负载均衡的方式。

![1637130672662](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/143113-642637.png)

使用 Share Slot 可以在每个 Slot 中部署来自不同 JobVertex 的多个 Task。

　　基于上述 Slot 管理和分配的逻辑，JobManager 负责维护作业中 Task执行的状态。如上文所述，Client 端会向 JobManager 提交一个 JobGraph，它代表了作业的逻辑结构。JobManager 会根据 JobGraph 按并发展开，从而得到 JobManager 中关键的 ExecutionGraph。ExecutionGraph 的结构如图 5 所示，与 JobGraph 相比，ExecutionGraph 中对于每个 Task 与中间结果等均创建了对应的对象，从而可以维护这些实体的信息与状态。

![1637130696517](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/143146-56021.png)

ExecutionGraph 是 JobGraph 按并发展开所形成的，它是 JobMaster 中的核心数据结构。

　　在一个 Flink Job 中是包含多个 Task 的，因此另一个关键的问题是在 Flink 中按什么顺序来调度 Task。如图 7 所示，目前 Flink 提供了两种基本的调度逻辑，即 Eager 调度与 Lazy From Source。Eager 调度如其名子所示，它会在作业启动时申请资源将所有的 Task 调度起来。这种调度算法主要用来调度可能没有终止的流作业。与之对应，Lazy From Source 则是从 Source 开始，按拓扑顺序来进行调度。简单来说，Lazy From Source 会先调度没有上游任务的 Source 任务，当这些任务执行完成时，它会将输出数据缓存到内存或者写入到磁盘中。然后，对于后续的任务，当它的前驱任务全部执行完成后，Flink 就会将这些任务调度起来。这些任务会从读取上游缓存的输出数据进行自己的计算。这一过程继续进行直到所有的任务完成计算。

![1637130719295](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/143200-48437.png)

### Flink架构概述

#### Flink 架构概览–Job

![1637122928672](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/122210-736072.png)

用户通过 DataStream API、DataSet API、SQL 和 Table API 编写 Flink 任务，它会生成一个JobGraph。JobGraph 是由 source、map()、keyBy()/window()/apply() 和 Sink 等算子组成的。当 JobGraph 提交给 Flink 集群后，能够以 Local、Standalone、Yarn 和 Kubernetes 四种模式运行。

#### Flink 架构概览–JobManager

![1637123008194](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/122328-391475.png)

JobManager的组件和功能主要有：

- 将 JobGraph 转换成 **Execution Graph**，最终将 Execution Graph 拿来运行。
- **Scheduler** 组件负责 Task 的调度，因为在JobManager上会生成执行图，执行图中包含一个一个的Task。
- **Checkpoint Coordinator** 组件负责协调整个任务的 Checkpoint，包括 Checkpoint 的开始和完成
- 通过 **Actor System** 与 TaskManager 进行通信
- 其它的一些功能，例如 Recovery Metadata，用于进行故障恢复时，可以从 Metadata 里面读取数据。

#### Flink 架构概览–TaskManager

![1637123149538](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/122550-160752.png)

TaskManager 是负责具体任务的执行过程，在 JobManager 申请到资源之后开始启动。TaskManager 里面的主要组件有：

- Memory & I/O Manager，即内存 I/O 的管理
- Network Manager，用来对网络方面进行管理
- Actor system，用来负责网络的通信

TaskManager 被分成很多个 TaskSlot，每个任务(Task)都要运行在一个 TaskSlot 里面，TaskSlot 是调度资源里的最小单位。

![1637123236912](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/122718-95221.png)

在介绍 Yarn 之前先简单的介绍一下 Flink Standalone 模式，这样有助于更好地了解 Yarn 和 Kubernetes 架构。

- 在 Standalone 模式下，Master 和 TaskManager 可以运行在同一台机器上，也可以运行在不同的机器上。
- 在 Master 进程中，**Standalone ResourceManager 的作用是对资源进行管理**。当用户通过 Flink Cluster Client 将 JobGraph 提交给 Master 时，JobGraph 先经过 Dispatcher。
- 当 Dispatcher 收到客户端的请求之后，生成一个 JobManager。**接着 JobManager 进程向 Standalone ResourceManager 申请资源，最终再启动 TaskManager**。
- TaskManager 启动之后，会有一个注册的过程,这个注册是到JobManager进程处注册，注册之后 JobManager 再将具体的 Task 任务分发给这个 TaskManager 去执行。

以上就是一个 Standalone 任务的运行过程。

### Flink 集群运行时相关组件

一个 Flink Cluster 是由一个 **Flink Master** 和多个 **Task Manager** 组成的，Flink Master 和 Task Manager 是进程级组件，其他的组件都是进程内的组件，可以使用下面这张图进行概括：是一个标准的master-slave结构。

而Flink Master又包含三个组件：

- Dispatcher
- ResourceManager 
- JobManager

![1637125634108](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/130752-473832.png)

一个FlinkMaster中有一个**ResourceManager**和多个JobManager,

Flink Master中每一个JobManager单独管理一个具体的Job

JobManager中的Scheduler组件负责调度执行该Job的DAG中所有Task，发出资源请求，即整个资源调度的起点；JobManager中Slot Pool组件持有该 Job 的所有资源。

FlinkMaster中唯一的ResourceManager负责整个 Flink Cluster 的资源调度以及与外部调度系统对接，这里的外部调度系统指的是 Kubernetes、Mesos、Yarn 等资源管理系统。

Task Manager 负责 Task 的执行，其中的 Slot 是 Task Manager 资源的一个子集，也是 Flink 资源管理的基本单位，Slot 的概念贯穿资源调度过程的始终。

> **Client：**用户通过 SQL 或者 API 的方式进行任务的提交，提交后会生成一个 JobGraph。
>
> **JobManager：**JobManager 接受到用户的请求之后，会对任务进行调度，并且申请资源启动 TaskManager。一个Job对应一个JobManager。
>
> **TaskManager：**它负责一个具体 Task 的执行。TaskManager 向 JobManager 进行注册，当 TaskManager 接收到 JobManager 分配的任务之后，开始执行具体的任务。

#### 逻辑层次

介绍完相关组件，我们需要了解一下这些组件之间的逻辑关系，共分如下为4层。

- **Operator** ：算子是最基本的数据处理单元。
- **Task**： Flink Runtime 中真正去进行调度的最小单位 由一系列算子链式组合而成（chained operators）（Note：如果两个 Operator 属于同一个 Task，那么不会出现一个 Operator 已经开始运行另一个 Operator 还没被调度的情况。）
- **Job** ：对应一个 Job Graph
- **Flink Cluster**： 1 Flink Master + N Task Managers

**可以从下面图看出逻辑层次**

![1637126206136](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/131647-199138.png)



刚介绍的与资源调度相关的组件中

- **JobManager、Secheduler和SlotPool**对应于Job级别，
- **ResourceManager、SlotManager和TaskManager**对应于FlinkCluster级别。

在 Operator 和 Task 中间的 Chaining 是指如何用 Operator 组成 Task 。在 Task 和 Job 之间的 Slot Sharing 是指多个 Task 如何共享一个 Slot 资源，这种情况不会发生在跨作业的情况中。在 Flink Cluster 和 Job 之间的 Slot Allocation 是指 Flink Cluster 中的 Slot 是怎样分配给不同的 Job 。

#### 两层资源调度

Flink的资源调度是一个经典的**两层模型**

- 其中从**Cluster到Job**的分配过程是由**SlotManager**来完成，
- **Job内部分配给Task**资源过程则是由**Scheduler**来完成，

> 一个JobManager中的Scheduler所管理这一个Job的左右资源。
>
> SlotManager管理者Flink集群的所有资源。

Scheduler向SlotPool发出SlotRequest(资源请求)，Slot Pool 如果不能满足该资源需求则会进一步请求 Resource Manager，具体来满足该请求的组件是 Slot Manager。

![1637126374302](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/131935-765696.png)

> 如何理解呢？
>
> - 就是说针对每一个JobManager，其实就是对每一个Job作业，Resource Manager是通过Slot Manager分配资源，因为一个Flink集群中可以启动多个Job作业。
> - 而针对一个具体的Job作业，因为Job中最小的调度单位是Task，而一个Job中有多个Task，所以每一个JobManager所拥有的资源，是通过Schedual进行分配。

#### 机制与策略(Flink1.10)

##### TaskManager有哪些资源？

![1637127191524](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/133311-998142.png)

> 资源类型：

- 内存
- CPU
- 其他拓展资源GPU

> TaskManager资源由配置决定

- Standalone 部署模式下，TaskManager资源可能不同
- 其他部署模式下，所有 TaskManager资源均相同

##### Slot有哪些资源？

![1637127247028](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/133408-243120.png)

- TaskManager有固定数量的 Slot 资源
- Slot 数量由配置决定
- Slot 资源由 TaskManager资源及 Slot 数量决定
- 同一 TaskManager上的 Slot 之间无差别

**Slot计算管理**

![1637127343879](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/133544-812028.png)

##### TaskManager管理

> Standalone部署模式

在 Standalone 部署模式下，Task Manager 的数量是固定的，如果是 start-cluster.sh 脚本来启动集群，可以通过修改以下文件中的配置来决定 TM 的数量；也可以通过手动执行 taskmanager.sh 脚本来启动一个 TM 。

```plain
<FLINK_DIR>/conf/slaves
```

> ActiveResourceManager部署模式

- Kubernetes，Yarn，Mesos
- Slot 数量按需分配，根据 Slot Request 请求数量启动 TaskManager
- TaskManager 空闲一段时间后，超时释放
- On-Yarn 部署模式不再支持固定数量的 TaskManager, 不再支持指定固定数量的 TM ，即以下命令参数已经失效。

```plain
yarn-session.sh -n <num>
flink run -yn <num>
```

##### Cluster -> Job 资源调度的过程

![1637127566943](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/133934-818801.png)

> 红色箭头

Scheduler 向 Slot Pool 发送请求(Task向Job请求资源)，如果 Slot 资源足够则直接分配，如果 Slot 资源不够，则由 Slot Pool 再向 Slot Manager发送请求（此时即为 Job 向 Cluster 请求资源），如果 Slot Manager 判断集群当中有足够的资源可以满足需求，那么就会向 Task Manager 发送 Assign 指令，Task Manager 就会提供 Slot 给 Slot Pool，Slot Pool 再去满足 Scheduler 的资源请求。

> 蓝色箭头

在 Active Resource Manager 资源部署模式下，当 Resource Manager 判定 Flink Cluster 中没有足够的资源去满足需求时，它会进一步去底层的资源调度系统请求资源，由调度系统把新的 Task Manager 启动起来，并且 TaskManager 向 Resource Manager 注册，则完成了新 Slot 的补充。

##### Job -> Task 资源调度的过程

> Scheduler

- 根据 Execution Graph 和 Task 的执行状态，决定接下来要调度的 Task
- 发起 SlotRequest
  - Task + Slot -> Allocate TaskSlot
- 决定 Task / Slot 之间的分配

> Slot Sharing

- Slot Sharing Group 中的任务可共用Slot
- 默认所有节点在一个 Slot Sharing Group 中
- 一个 Slot 中相同任务只能有一个

> 优点

- 运行一个作业所需的 Slot 数量为最大并发数
- 相对负载均衡

**Job 到 Task 资源调度过程**

![1637127642442](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/134043-607770.png)

Slot Sharing 过程（每一行分别是一个 task 的多个并发，自下而上分别是 A、B、C），A、B、C 的并行度分别是4、4、3，这些 Task 属于同一个 Slot Sharing Group 中，所以不同的 Task 可以放在相同的 Slot 中运行，如图7右侧所示，有3个 Slot 放入了 ABC，而第四个 Slot 放入了 AB 。通过以上过程我们可以很容易推算出这个 Job 需要的 Slot 数是4，也是最大并发数。

##### 资源调优

通过以上介绍的机制，我们容易发现，Flink 所采用的是自顶向下的资源管理，我们所配置的是 Job 整体的资源，而 Flink 通过 Slot Sharing 机制控制 Slot 的数量和负载均衡，通过调整 Task Manager / Slot 的资源，以适应一个 Slot Sharing Group 的资源需求。Flink 的资源管理配置简单，易用性强，适合拓扑结构简单或规模较小的作业。

### Flink on Yarn 原理及实践

#### Yarn 架构原理–总览

Yarn 模式在国内使用比较广泛，基本上大多数公司在生产环境中都使用过 Yarn 模式。首先介绍一下 Yarn 的架构原理，因为只有足够了解 Yarn 的架构原理，才能更好的知道 Flink 是如何在 Yarn 上运行的。

![1637123434199](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/123034-402600.png)

Yarn 的架构原理如上图所示，最重要的角色是 **ResourceManager**，主要用来负责整个资源的管理，Client 端是负责向 ResourceManager 提交任务。

用户在 Client 端提交任务后会先给到 Resource Manager。Resource Manager 会启动 Container，接着进一步启动 **Application Master**(代表整个作业的老大），即对 Master 节点的启动。当 Master 节点启动之后，会向 Resource Manager 再重新申请资源，当 Resource Manager 将资源分配给 Application Master 之后，Application Master 再将具体的 Task 调度起来去执行。

#### Yarn 架构原理–组件

**Yarn 集群中的组件包括：**

- **ResourceManager (RM)**：ResourceManager (RM)负责处理客户端请求、启动/监控 ApplicationMaster、监控 NodeManager、资源的分配与调度，包含 Scheduler 和 Applications Manager。
- **ApplicationMaster (AM)**：ApplicationMaster (AM)运行在 Slave 上，负责数据切分、申请资源和分配、任务监控和容错，代表一个作业的老大。
- **NodeManager (NM)**：NodeManager (NM)运行在 Slave 上，用于单节点资源管理、AM/RM通信以及汇报状态。
- **Container**：Container 负责对资源进行抽象，包括内存、CPU、磁盘，网络等资源。

#### Yarn 架构原理–交互

![1637123722657](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/123522-175953.png)

以在 Yarn 上运行 MapReduce 任务为例来讲解下 Yarn 架构的交互原理：

- 首先，用户编写 MapReduce 代码后，通过 Client 端进行任务提交
- ResourceManager 在接收到客户端的请求后，会分配一个 Container 用来启动 ApplicationMaster，并通知 NodeManager 在这个 Container 下启动 ApplicationMaster。
- ApplicationMaster 启动后，向 ResourceManager 发起注册请求。接着 ApplicationMaster 向 ResourceManager 申请资源。根据获取到的资源，和相关的 NodeManager 通信，要求其启动程序。
- 一个或者多个 NodeManager 启动 Map/Reduce Task。
- NodeManager 不断汇报 Map/Reduce Task 状态和进展给 ApplicationMaster。
- 当所有 Map/Reduce Task 都完成时，ApplicationMaster 向 ResourceManager 汇报任务完成，并注销自己。

### Flink on Yarn–Per Job

![1637123893091](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/123814-804998.png)

Flink on Yarn 中的 Per Job 模式是指每次提交一个任务，然后任务运行完成之后资源就会被释放。在了解了 Yarn 的原理之后，Per Job 的流程也就比较容易理解了，具体如下：

- 首先 Client 提交 Yarn App，比如 JobGraph 或者 JARs。
- 接下来 Yarn 的 ResourceManager 会申请第一个 Container。这个 Container 通过 Application Master 启动进程，Application Master 里面运行的是 Flink 程序，也就是FlinkMaster，即 Flink-Yarn ResourceManager 和 JobManager，还有Dispatcher。
- 最后 Flink-Yarn ResourceManager 向 Yarn ResourceManager 申请资源。当分配到资源后，启动 TaskManager。TaskManager 启动后向 Flink-Yarn ResourceManager 进行注册，注册成功后 JobManager 就会分配具体的任务给 TaskManager 开始执行。

> per模式中Dispatcher是运行在客户端。

### Flink on Yarn–Session

![1637123940310](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/17/123900-222621.png)

在 Per Job 模式中，执行完任务后整个资源就会释放，包括 JobManager、TaskManager 都全部退出。而 Session 模式则不一样，它的 Dispatcher 和 ResourceManager 是可以复用的。Session 模式下，当 Dispatcher 在收到请求之后，会启动 JobManager(A)，让 JobManager(A) 来完成启动 TaskManager，接着会启动 JobManager(B) 和对应的 TaskManager 的运行。当 A、B 任务运行完成后，资源并不会释放。Session 模式也称为多线程模式，其特点是资源会一直存在不会释放，多个 JobManager 共享一个 Dispatcher，而且还共享 Flink-YARN ResourceManager。

Session 模式和 Per Job 模式的应用场景不一样。Per Job 模式比较适合那种对启动时间不敏感，运行时间较长的任务。Seesion 模式适合短时间运行的任务，一般是批处理任务。若用 Per Job 模式去运行短时间的任务，那就需要频繁的申请资源，运行结束后，还需要资源释放，下次还需再重新申请资源才能运行。显然，这种任务会频繁启停的情况不适用于 Per Job 模式，更适合用 Session 模式。

### Yarn 模式特点

**Yarn 模式的优点有：**

- 资源的统一管理和调度。Yarn 集群中所有节点的资源（内存、CPU、磁盘、网络等）被抽象为 Container。计算框架需要资源进行运算任务时需要向 Resource Manager 申请 Container，YARN 按照特定的策略对资源进行调度和进行 Container 的分配。Yarn 模式能通过多种任务调度策略来利用提高集群资源利用率。例如 FIFO Scheduler、Capacity Scheduler、Fair Scheduler，并能设置任务优先级。
- 资源隔离：Yarn 使用了轻量级资源隔离机制 Cgroups 进行资源隔离以避免相互干扰，一旦 Container 使用的资源量超过事先定义的上限值，就将其杀死。
- 自动 failover 处理。例如 Yarn NodeManager 监控、Yarn ApplicationManager 异常恢复。

Yarn 模式虽然有不少优点，但是也有诸多缺点，**例如运维部署成本较高，灵活性不够。**