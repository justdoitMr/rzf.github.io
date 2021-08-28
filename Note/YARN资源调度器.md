# YARN资源调度器

[TOC]

## Yarn的重要概念

1. Yarn并不清楚用户提交的程序的运行机制
2. Yarn只提供运算资源的调度（用户程序向Yarn申请资源，Yarn就负责分配资源）
3. Yarn中的主管角色叫ResourceManager
4. Yarn中具体提供运算资源的角色叫NodeManager
5. 这样一来，Yarn其实就与运行的用户程序完全解耦，就意味着Yarn上可以运行各种类型的分布式运算程序（mapreduce只是其中的一种），比如mapreduce、storm程序，spark程序……
6. 所以spark、storm等运算框架都可以整合在Yarn上运行，只要他们各自的框架中有符合Yarn规范的资源请求机制即可
7. Yarn就成为一个通用的资源调度平台，可以把他想象为一个操作系统，从此，企业中以前存在的各种运算集群都可以整合在一个物理集群上，提高资源利用率，方便数据共享

## Yarn资源调度器

- 思考：
  - 如何管理集群资源？
  - 如何给任务合理分配资源？

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而mapreduce等运算程序则相当于运行于操作系统之上的应用程序

## Yarn基础架构

YARN主要由ResourceManager（负责管理整个集群的资源）、NodeManager（负责管理单个节点的资源）、ApplicationMaster（负责管理单个作业的资源）和Container（虚拟计算机资源 ）等组件构成

**基础组件介绍**

![1621253241898](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/200722-919743.png)

- ResourceManager是整个集群管理资源的老大

- NodeManager是管理单个节点的老大。

- ApplicationMaster是管理作业或者任务的老大。

- Container是计算机资源的抽象，是一个容器。

## Yarn的工作机制

![1621253713426](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/17/201513-549758.png)

**提交的文件**

job.split可以控制切片的数量，控制开启多少个mapTask。

Job.xml可以控制集群的各个参数信息。

wc.jar是程序的代码。

1. MR 程序提交到客户端所在的节点，提交后会创建一个YarnRunner进程。
2. YarnRunner 向 ResourceManager 申请一个 Application，也就是申请执行一个应用。
3. ResourceManager  将该应用程序的资源路径返回给 YarnRunner。
4. YarnRunner将该程序将运行所需资源提交到 HDFS 上，也就是上面写出的三个文件。
5. YarnRunner进程资源提交完毕所需要的资源后，申请运行 mrApplicationMaster，也就是这一个作业的老大。
6. ResourceManager  将用户的请求初始化成一个 Task，然后存放到任务队列当中，如果有多个任务，那么就全部存放到任务队列中。
7. 其中一个 NodeManager 领取到 Task 任务。
8. 该 NodeManager 创建容器 Container，因为任何一个任务的执行都是在容器中执行，并产生 mrApplicationMaster。
9. Container 从 HDFS 上拷贝资源到本地。
10. mrApplicationMaster向 ResourceManager 申请运行 MapTask 资源，会根据刚才读取的文件信息，也就是切片的数量，来申请map任务的个数。
11. ResourceManager将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager 分别领取任务并创建容器。
12. ResourceManager向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager
    分别启动 MapTask，MapTask 对数据分区排序，执行完后的数据持久化到磁盘当中。
13. mrApplicationMaster等待所有 MapTask 运行完毕后，向 ResourceManager申请容器，申请容器的个数是根据分区的个数来划分的，有多少个分区，就开启多少个ReduceTask进程，运行 ReduceTask。
14. ReduceTask 向 MapTask 获取相应分区的数据。
15. 程序运行完毕后，mrApplicationMaster 会向 ResourceManager 申请注销自己。

> 注意：在集群模式下面创建的是YarnRunner，在mian()函数最后执行waitForCompleation()方法的时候，会创建YarnRunner，如果是在本地模式下面，那么会创建LocalRunner。
>
> mrApplicationMaster 是单个作业的调度中心，负责整个作业的执行。

## 作业提交全流程

**Hdfs，Yarn,MapReduce三者之间的关系**

![1621420847219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/184047-440924.png)

**Yarn任务调度**

![1611805156296](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/113917-936052.png)

客户端要想运行一个作业，首先联系资源管理器（resourcemanager),资源管理器找到一个能够在容器中启动application master的节点管理器（图中的2a和2b步骤），application master运行起来可以做什么依赖于应用本身，可能是简单的运行一个计算任务，也可能是向resources manager请求更多的容器资源（步骤三）来运行一个分布式计算任务（步骤4a和4b）。

作业提交全过程详解

1. 作业提交

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

2. 作业初始化

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

3. 任务分配

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

4. 任务运行

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

5. 进度和状态更新

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

6. 作业完成

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

**mapReducer作业的提交**

![1611805793862](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/114957-457459.png)

**作业提交**

![1611805836562](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/115038-294763.png)



**读取数据过程**

![1611805865232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/115108-80525.png)

**写数据过程**

![1611805890703](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/115133-262134.png)

## Yarn资源调度器

### 先进先出调度器（FIFO）

![1611802720420](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/185711-67928.png)

作业按照先到先运行的方法进行调度，也就是先进先出的调度顺序。

### 容量调度器（Capacity Scheduler）

容量调度器是一个多用户的调度器。

![1611802838738](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/110041-117502.png)

![1621421589683](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/185312-776055.png)

- 容量调度器中，每一个队列占总资源的一个百分比，资源的分配优先满足先进来的任务， 
- 每一个队列占据的资源有下限和上限
- 资源可以共享。
- 每一个队列中可以有多个用户的任务的一部分。

每一个队列的资源可以任意分配，容量调度器是多个fifo调度器的整合，可以提高并发程度。有多少个队列，并发度就是几。

**容量调度器资源分配算法**

![1621421997210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/185958-657310.png)

- 队列的选择：优先选择占用资源比较少的对列
- 作业的执行：按照优先级或者作业到达的时间。
- 容器资源的分配：按照容器的优先级或者数据本地行的原则。

### 公平调度器（Fair Scheduler）

公平调度器也是一种多用户的调度器。

![1611803190582](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/28/110636-942398.png)

![1621422416909](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/190657-771547.png)

什么是核心调度策略不同：

- 容量调度器：优先选择资源利用率低的队列。
- 公平调度器：优先选择对资源缺额比例较大的队列。

最大的不同是每一个队列可以单独设置资源的分配方式。

这种缺额，也就是把缺额资源看做是一种优先级。同一个队列中，也可以有多个作业同时运行，比容量调度器并发度更高。

**公平调度器---缺额**

![1621422878199](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621422878199.png)

也就是保证获取资源最少的任务也可以得到执行，不会产生饥饿现象。

**公平调度器的队列资源分配方式**

![1621423892244](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/193133-806207.png)

**Fire策略**

具体的资源分配顺序和容量调度器是一致的。

- 选择队列
- 选择作业
- 选择容器

但是上面每一个步骤都是按照公平的算法进行选择的，公平的体现在每一次分配资源的时候都是进行平均分配。

- 队列资源的分配方式

![1621423847499](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/193048-966126.png)

100个资源首先平均给每一个队列分33.33的资源（这里就是体现出公平的资源分配方式），由于第二个队列还少资源，所以就把其他两个队列剩下的资源分配给第二个队列，这样第二个队列的资源也满足。

**作业资源的分配方式**

有加权和不加劝两种分配方式

![1621424272749](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/19/193756-730468.png)

不加劝方式分配：每一次对剩余资源的分配都是平均分配，一共12个资源，第一次平均分配每一个作业分配3个，其中第一第二个作业有剩余的资源，对剩余的资源再一次进行平均分配。只要有剩余的资源，就一直进行分配，直到没有资源分配为止。

加权方式分配资源：就是每一次按照每一个作业权值占总的权重的比值进行资源的分配。

**DRF策略**

![1621510948191](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/194231-183070.png)

## Yarn常用命令

