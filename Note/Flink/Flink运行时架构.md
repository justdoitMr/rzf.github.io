## Flink运行时架构

Flink 是一个用于**状态化并行流**处理的分布式系统。它的搭建涉及多个进程，这些进程通常会分布在多台机器上。分布式系统需要应对的常见挑战包括分配和管理集群计算资源，进程协调，持久且高可用的数据存储及故障恢复等。

### flink原理初探

![1621655348577](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/114910-990149.png)

flink客户端不仅会提交应用程序的代码，还会将提交的程序代码转换为数据流图，然后进行一些优化，形成一个数据流图，然后提交给资源管理的节点job manager节点，也就是master，负责整个作业的管理，任务的分配和资源的申请。job manager会将任务分给task mansger节点。task manager是工作节点，工作节点里面是task slot，可以把taskslot看做是线程，taskmanager看做的一个jvm进程，执行的是任务。如果和spark类比,job manager就相当于spark中的driver进程，taskmanager就相当于worker。

- 简单来说在clint上面会生成最初的程序流图，也就是streamGraph，
- 然后把oneToOne的算子进行合并，做一些优化生成jobGraph图，
- 接下来在jobManager上面根据程序中设置的并行度和资源的申请状况生成executorGraph图。
- 将executorGraph落实到具体的taskmanager上面，并将具体的subtask落实到具体的slot中进行运行，这一步是物理的执行图。

> 生成前两部分的图形是在clint端进行的。
>
> 生成后两步的图是在jobmanager上面进行的。

### Flink运行时的组件

![1614333429684](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222550-73500.png)

Flink 运行时架构主要包括四个不同的组件，它们会在运行流处理应用程序时协同工作：作业管理器（JobManager）、资源管理器（ResourceManager）、任务管理器（TaskManager），以及分发器（Dispatcher）。因为 Flink 是用 Java 和 Scala 实现的，所以所有组件都会运行在Java 虚拟机上。每个组件的职责如下：

**作业管理器（JobManager）**可以类比spark中的driver

- 控制一个应用程序执行的**主进程**，也就是说，每个应用程序(作业)都会被一个不同的JobManager 所控制执行。也就是说提交一个flink程序，那么就由jobmanager来控制执行，提交作业就是提交给jobmanager。它扮演的是集群管理者的角色，负责调度任务、协调checkpoints、协调故障恢复、收集 Job 的状态信息，并管理 Flink 集群中的从节点 TaskManager。
- JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其它资源的 JAR 包。
- JobManager 会把 JobGraph 转换成一个物理层面的数据流图dataflow图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。
- JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调。

**任务管理器（TaskManager）**类比spark中的executor

- Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager都包含了一定数量的插槽（slots）。**插槽的数量限制了 TaskManager 能够执行的任务数量。**
- 启动之后，TaskManager 会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。
- JobManager 就可以向插槽分配任务（tasks）来执行了。在执行过程中，一个 TaskManager 可以跟其它运行同一应用程序的 TaskManager 交换数据。所以交换数据有时候是夸槽，有时候是夸taskmanager。
- 实际负责执行计算的 Worker，在其上执行 Flink Job 的一组 Task；TaskManager
  还是所在节点的管理员，它负责把该节点上的服务器信息比如内存、磁盘、任务运行情况等向 JobManager 汇报。

**客户端**

- 用户在提交编写好的 Flink 工程时，会先创建一个客户端再进行提交，这个客户端就是 Client

**分发器（Dispatcher）**

- Dispatcher可以跨作业运行，它为应用提交提供了REST接口。来让我们提交需要执行的应用。
- 当一个应用被提交执行时，Dispatcher分发器就会启动并将应用移交给一个JobManager。
- Dispatcher也会启动一个Web UI，用来方便地展示和监控作业执行的信息。
- Dispatcher在架构中可能并不是必需的，这取决于应用提交运行的方式。

**资源管理器（ResourceManager）**

- 主要负责管理任务管理器（TaskManager）的插槽（slot,其实就是工作线程），TaskManger 插槽是Flink中定义的处理计算资源单元。
- Flink为不同的环境和资源管理工具提供了不同资源管理器，比如YARN、Mesos、K8s，以及standalone部署。
- 当JobManager申请插槽资源时，ResourceManager会将有空闲插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽来满足JobManager的请求，它还可以向资源提供平台发起会话，以提供启动TaskManager进程的容器。
- 针对不同的环境和资源提供者(resource provider) (如YARN 、Mesos 、Kubernetes 或独立部署) , Flink 提供了不同的ResourceManager 。ResourceManager 负责管理Flink 的处理资源单元一-TaskManager 处理槽。当JobManager 申请TaskManager 处理槽时， ResourceManager 会指示一个拥有空闲处理槽的TaskManager 将其处理槽提供给JobManager 。如果ResourceManager 的处理槽数无怯满足JobManager 的请求，则ResourceManager 可以和资源提供者通信，让它们提供额外容器来启动更多TaskManager 进程。同时， ResourceManager 还负责终止空闲的TaskManager 以释放计算资源。

![1621654816302](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/114018-741117.png)

这个JobManager很像Spark中的Driver进程，负责整个作业的运行，把整个作业分为多个任务，调度任务在TaskManager上面运行。

**各个组件之间的交互**

![1614337366271](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/190616-19485.png)

> JobManager<=>driver
>
> TaskManager<=>Executor

### 名词说明

 Dataflow、Operator、Partition、SubTask、Parallelism

1.Dataflow:Flink程序在执行的时候会被映射成一个数据流模型

2.Operator:数据流模型中的每一个操作被称作Operator,Operator分为:Source/Transform/Sink

3.Partition:数据流模型是分布式的和并行的,执行中会形成1~n个分区

4.Subtask:多个分区任务可以并行,每一个都是独立运行在一个线程中的,也就是一个Subtask子任务

5.Parallelism:并行度,就是可以同时真正执行的子任务数/分区数

**说明** ![1621656307856](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/120549-525043.png)

### 任务提交流程

![1614337366271](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/190616-19485.png)

注意，我们在flink配置文件中配置的jobmanager是对节点的配置，而在这里的jobmanager实际上是对每一个作业（job)的管理者jobmanager，需要注意区分，提交作业后，jobmanager知道所有的作业信息。

上图是从一个较为高层级的视角，来看应用中各组件的交互协作。如果部署的集群环境不同（例如 YARN，Mesos，Kubernetes，standalone 等），其中一些步骤可以被省略，或是有些组件会运行在同一个 JVM 进程中。

#### on Yarn

具体地，如果我们将 Flink 集群部署到 YARN 上，那么就会有如下的提交流程：

![1614339707142](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/101138-891397.png)

![1621655962978](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/05/144123-807412.png)

- Flink 任务提交后，yarn的Client （类似于spark中的driver）向 HDFS 上传 Flink 的 Jar 包和配置，之后向 YarnResourceManager 提交任务，申请运行applicationmaster进程，由applicationmaster来负责这个作业的资源和任务的管理，注册并且申请资源。
- ResourceManager 分配 Container 资源并通知对应的NodeManager 启动 ApplicationMaster，然后该applicationmaster就是该yarn集群上该作业的老大，负责管理作业的资源，ApplicationMaster 启动后加载 Flink 的 Jar 包和配置构建环境，然后启动 JobManager，ApplicationMaster和JobManager一般在同一个节点上面，具体的任务细节由jobmanager来负责。
- 之后 ApplicationMaster 向 ResourceManager申请资源启动 TaskManager ， ResourceManager 分 配 Container 资 源 后 ， 由ApplicationMaster 通 知 资 源 所 在 节 点 的 NodeManager 启 动 TaskManager ，
- NodeManager 加载 Flink 的 Jar 包和配置构建环境并启动 TaskManager，TaskManager启动后向 JobManager 发送心跳包，并等待 JobManager 向其分配任务。

#### Standalone版

![1621662107867](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/134157-303510.png)

### 任务调度原理

![1614340253325](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222509-288284.png)



客户端不是运行时和程序执行 的一部分，但它用于准备并发送dataflow(JobGraph)给Master(JobManager)，然后，客户端断开连接或者维持连接以等待接收计算结果。

当 Flink 集 群 启 动 后 ， 首 先 会 启 动 一 个 JobManger 和一个或多个的TaskManager。由 Client 提交任务给 JobManager，JobManager 再调度任务到各个TaskManager 去执行，然后 TaskManager 将心跳和统计信息汇报给 JobManager。TaskManager 之间以流的形式进行数据的传输。上述三者均为独立的 JVM 进程。

Client 为提交 Job 的客户端，可以是运行在任何机器上（与 JobManager 环境连通即可）。提交 Job 后，Client 可以结束进程（Streaming 的任务），也可以不结束并等待结果返回。

JobManager 主 要 负 责 调 度 Job 并 协 调 Task 做 checkpoint， 职 责 上 很 像Storm 的 Nimbus。从 Client 处接收到 Job 和 JAR 包等资源后，会生成优化后的执行计划，并以 Task 的单元调度到各个 TaskManager 去执行。

TaskManager 在启动的时候就设置好了槽位数（Slot），每个 slot 能启动一个Task，Task 为线程。从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立 Netty 连接，接收数据并处理。

1. 怎样实现并行计算？

高吞吐量就是由我们的集群保证的，对于我们的每一个任务，都可以设置一个并行度，然后拆分成多个任务由task去执行，相当于多个线程同时去执行我们的任务，就相当于并行计算。不同的slot就是执行不同的任务，slot其实就是一个线程

2. 并行的任务，需要占用多少slot？

slot的个数个每一步的操作设置的slot个数有关

3. 一个流处理程序，到底包含多少个任务？

### Operator传递模式

数据在两个operator(算子)之间传递的时候有两种模式：

1. One to One模式：

两个operator用此模式传递的时候，会保持数据的分区数和数据的排序；如上图中的Source1到Map1，它就保留的Source的分区特性，以及分区元素处理的有序性。--类似于Spark中的窄依赖

2. Redistributing 模式：

这种模式会改变数据的分区数；每个一个operator subtask会根据选择transformation把数据发送到不同的目标subtasks,比如keyBy()会通过hashcode重新分区,broadcast()和rebalance()方法会随机重新分区。--类似于Spark中的宽依赖

### 并行度（Parallelism）

- Flink 程序的执行具有并行、分布式的特性。
- 在执行过程中，一个流（ stream） 包含一个或多个分区（ stream partition） ，而每一个算子（ operator）可以包含一个或多个子任务（ operator subtask） ，这些子任务在不同的线程、不同的物理机或不同的容器中彼此互不依赖地执行。 
- 一个特定算子的子任务（ subtask） 的个数被称之为其并行度（ parallelism） 。一般情况下， 一个流程序的并行度，可以认为就是其所有算子中最大的并行度。一个程序中，不同的算子可能具有不同的并行度。 

![1621656307856](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/120549-525043.png)

- 一个特定算子的 子任务（subtask）的个数被称之为其并行度（parallelism）。一般情况下，一个 stream 的并行度，可以认为就是其所有算子中最大的并行度。

Stream 在算子之间传输数据的形式可以是 one-to-one(forwarding)的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类 

分区上的一系列算子操作叫做子任务。

One-to-one： stream(比如在 source 和 map operator 之间)维护着分区以及元素的顺序。那意味着 map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务生产的元素的个数、顺序相同， map、 fliter、 flatMap 等算子都是 one-to-one 的对应关系。 多个一对一的算子可以合并操作形成一个操作链条，任务链之间可以并行执行，每一个操作连或者单个算子叫做任务，task下面可以有多个子任务，**类似于 spark 中的窄依赖 **

Redistributing： stream(map()跟 keyBy/window 之间或者 keyBy/window 跟 sink之间)的分区会发生改变。每一个算子的子任务依据所选择的 transformation 发送数据到不同的目标任务。例如， keyBy() 基于 hashCode 重分区、 broadcast 和 rebalance会随机重新分区，这些算子都会引起 redistribute 过程，而 redistribute 过程就类似于Spark 中的 shuffle 过程，**类似于 spark 中的宽依赖**

![1621656757723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/121346-275584.png) 

### 任务链（Operator Chains） 

相同并行度的 one to one 操作， Flink 这样相连的算子链接在一起形成一个 task，原来的算子成为里面的一部分。将算子链接成 task 是非常有效的优化：它能减少线程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。链接的行为可以在编程 API 中进行指定。 

![1614392790996](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/131819-860063.png)

![1621656757723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/121346-275584.png)

客户端在提交任务的时候会对Operator进行优化操作，能进行合并的Operator会被合并为一个Operator，

合并后的Operator称为Operator chain，实际上就是一个执行链，每个执行链会在TaskManager上一个独立的线程中执行--就是SubTask。

### TaskManager 和 Slots

Flink 中每一个 worker(TaskManager)都是一个 JVM 进程，它可能会在独立的线程上执行一个或多个 subtask。为了控制一个 worker 能接收多少个 task， worker 通过 task slot 来进行控制（一个 worker 至少有一个 task slot）。 

每个 task slot 表示 TaskManager 拥有资源的一个固定大小的子集。假如一个TaskManager 有三个 slot，那么它会将其管理的内存分成三份给各个 slot。资源 slot化意味着一个 subtask 将不需要跟来自其他 job 的 subtask 竞争被管理的内存，取而代之的是它将拥有一定数量的内存储备。需要注意的是，这里不会涉及到 CPU 的隔离， slot 目前仅仅用来隔离 task 的受管理的内存。 

通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离。如果一个TaskManager 一个 slot，那将意味着每个 task group 运行在独立的 JVM 中（该 JVM可能是通过一个特定的容器启动的），而一个 TaskManager 多个 slot （线程）意味着更多的subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将共享 TCP 连接（基于多路复用）和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个task 的负载。 

![1621657225993](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/122027-990485.png)

- Flink 中每一个 TaskManager 都是一个JVM进程，它可能会在独立的线程上执行一个或多个子任务，其中slot的个数就代表并发线程执行的个数。
- 为了控制一个 TaskManager 能接收多少个 task， TaskManager 通过 taskslot 来进行控制（一个 TaskManager 至少有一个 slot）
- 也就是说每一个线程都要单独分配slot资源去执行

![1614386765807](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/084627-278044.png)

### Sharing Slot

![1621657450129](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/122412-10545.png)

草共享可以避免线程的创建和销毁带来的开销，可以看做是线程池。

- 默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务。 这样的结果是，一个 slot 可以保存作业的整个管道。相同的任务的子任务不能放在同一个slot中执行，必须是不同的任务的子任务可以共享slot,相同组的线程可以占用同一个slot,不同组的线程占据不同的slot.
- 推荐根据cpu的核心数设置slot的个数
- Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力
- 不同的slot共享组占据不同的slot。

Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过参数 taskmanager.numberOfTaskSlots 进行配置； 而并行度 parallelism 是动态概念，即 TaskManager 运行程序时实际使用的并发能力，可以通过参数 parallelism.default进行配置。

也就是说，假设一共有 3 个 TaskManager，每一个 TaskManager 中的分配 3 个TaskSlot，也就是每个 TaskManager 可以接收 3 个 task，一共 9 个 TaskSlot，如果我们设置 parallelism.default=1，即运行程序默认的并行度为 1， 9 个 TaskSlot 只用了 1个，有 8 个空闲，因此，设置合适的并行度才能提高效率。

**允许插槽共享有两个主要好处：**

- 资源分配更加公平，如果有比较空闲的slot可以将更多的任务分配给它。
- 有了任务槽共享，可以提高资源的利用率。

**注意:**

- slot是静态的概念，是指taskmanager具有的并发执行能力
- parallelism是动态的概念，是指程序运行时实际使用的并发能  

### 并行子任务的分配

一个 TaskManager 允许同时执行多个任务。这些任务可以属于同一个算子(数据并行) ，也可以是不同算子(任务并行) ，甚至还可以来自不同的应用(作业并行) 0 TaskManager 通过提供固定数量的处理槽来控制可以并行执行的任务数。 每个处理槽可以执行应用的一部分，即算子的一个并行任务。图 3-2展示了TaskManager、处理槽、任务以及算子之间的关系。 

![1614694340976](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/221358-388017.png)

左侧的 JobGraph (应用的非并行化表示)包含了 5 个算子，其中算子 A 和 C是数据橱，算子 E 是数据汇。算子 C 和 E 的并行度为 2 ，其余算子的并行度为 4 。由于算子最大并行度是 4 ，因此应用若要执行则至少需要 4 个处理槽。如果每个 TaskManager 内有两个处理槽，则运行两个 TaskManager 即可满足该需求。
JobManager 将 JobGraph "展开成" ExecutionGraph 并把任务分配到 4 个空闲处理槽。对于井行度为 4 的算子，其任务会每个处理槽分配一个。其余两个算子 C 和 E 的任务会分别放到处理槽1.1、 2. 1 和处理槽 1.2 、 2 .2 中。将任务以切片的形式调度至处理槽中有一个好处: TaskManager 中的多个任务可以在同一进程 内高效地执行数据交换而无须访问网络。然而，任务过于集中也会使TaskManager 负载变高，继而可能导致性能下降。 

### 程序与数据流（DataFlow）

![1614390712367](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/095152-707756.png)

- 所有的 Flink 程序都是由三部分组成的： Source 、 Transformation 和 Sink。 
- Source 负责读取数据源， Transformation 利用各种算子进行处理加工， Sink 负责输出。 
- 在运行时， Flink 上运行的程序会被映射成“逻辑数据流”（ dataflows） ，它包含了这三部分。 每一个 dataflow 以一个或多个 sources 开始以一个或多个 sinks 结束。 dataflow 类似于任意的有向无环图（ DAG）。在大部分情况下，程序中的转换运算（ transformations） 跟 dataflow 中的算子（ operator） 是一一对应的关系，但有时候，一个 transformation 可能对应多个 operator。

![1614392321655](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/101842-896076.png)

### 执行图（ExecutionGraph） 

由 Flink 程序直接映射成的数据流图是 StreamGraph，也被称为逻辑流图，因为它们表示的是计算逻辑的高级视图。为了执行一个流处理程序， Flink 需要将逻辑流图转换为物理数据流图（也叫执行图） ，详细说明程序的执行方式。 

Flink 中的执行图可以分成四层： StreamGraph -> JobGraph -> ExecutionGraph ->物理执行图。 

**执行图生成过程**

![1621657844935](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/123047-672135.png)

![1621662703581](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/135542-374085.png)

**原理介绍**

- Flink执行executor会自动根据程序代码生成DAG数据流图
- Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图。
- **StreamGraph**：是根据用户通过 Stream API 编写的代码生成的最初的图。表示程序的拓扑结构。
- **JobGraph**：StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点，这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
- **ExecutionGraph**：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
- **物理执行图**：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。

**简单理解：**

- StreamGraph：最初的程序执行逻辑流程，也就是算子之间的前后顺序--在Client上生成
- JobGraph：将OneToOne的Operator合并为OperatorChain--在Client上生成
- ExecutionGraph：将JobGraph根据代码中设置的并行度和请求的资源进行并行化规划!--在JobManager上生成
- 物理执行图：将ExecutionGraph的并行计划,落实到具体的TaskManager上，将具体的SubTask落实到具体的TaskSlot内进行运行。