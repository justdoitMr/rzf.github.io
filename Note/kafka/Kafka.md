
<!-- TOC -->

- [批量发送的基本单位, 默认是16384Bytes, 即16kB](#批量发送的基本单位-默认是16384bytes-即16kb)
- [延迟时间](#延迟时间)
- [两者满足其一便发送](#两者满足其一便发送)
      - [数据压缩](#数据压缩)
    - [解释如何减少ISR中的扰动？broker什么时候离开ISR？](#解释如何减少isr中的扰动broker什么时候离开isr)
    - [ISR、OSR、AR 是什么？](#isrosrar-是什么)
    - [LEO、HW、LSO、LW等分别代表什么？](#leohwlsolw等分别代表什么)
    - [Kafka为什么需要复制？](#kafka为什么需要复制)
    - [如何保证Kafka的消息有序](#如何保证kafka的消息有序)
    - [Kafka 的高可靠性是怎么实现的？](#kafka-的高可靠性是怎么实现的)
      - [Topic分区副本](#topic分区副本)
      - [Producer往Broker 发送消息](#producer往broker-发送消息)
      - [Leader 选举](#leader-选举)
      - [数据一致性（可回答“Kafka数据一致性原理？”）](#数据一致性可回答kafka数据一致性原理)
    - [Kafka 缺点？](#kafka-缺点)
    - [Kafka 分区数可以增加或减少吗？为什么？](#kafka-分区数可以增加或减少吗为什么)
    - [Kafka消息可靠性的保证](#kafka消息可靠性的保证)
      - [Broker](#broker)
      - [Producer](#producer)
      - [Consumer消费消息有下面几个步骤：](#consumer消费消息有下面几个步骤)
    - [为什么kafka中1个partition只能被同组的一个consumer消费?](#为什么kafka中1个partition只能被同组的一个consumer消费)
    - [zookeeper在kafka中的作用](#zookeeper在kafka中的作用)
      - [作用](#作用)
        - [Broker注册](#broker注册)
        - [Topic注册](#topic注册)
        - [生产者负载均衡](#生产者负载均衡)
        - [消费者负载均衡](#消费者负载均衡)
        - [分区与消费者的关系](#分区与消费者的关系)
        - [消费进度Offset记录](#消费进度offset记录)
        - [消费者注册](#消费者注册)
    - [Kafka服务器能接收到的最大信息是多少？](#kafka服务器能接收到的最大信息是多少)
    - [Kafka中的ZooKeeper是什么？Kafka是否可以脱离ZooKeeper独立运行？](#kafka中的zookeeper是什么kafka是否可以脱离zookeeper独立运行)

<!-- /TOC -->


## Kafka篇

### 请说明什么是Apache Kafka？

Kafka 是由 `Linkedin` 公司开发的，**它是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台**，它同时也是一款开源的**基于发布订阅模式的消息引擎系统**。

### Kafka的基本术语

消息：Kafka 中的数据单元被称为`消息`，也被称为记录，**可以把它看作数据库表中某一行的记录**。

批次：为了提高效率， 消息会`分批次`写入 Kafka，批次就代指的是一组消息。

主题：消息的种类称为 `主题`（Topic）,可以说一个主题代表了一类消息。相当于是对消息进行分类。**主题就像是数据库中的表。**

分区：主题可以被分为若干个分区（partition），同一个主题中的分区可以不在一个机器上，有可能会部署在多个机器上，由此来实现 kafka 的`伸缩性`，单一主题中的分区有序，但是无法保证主题中所有的分区有序，把分区理解为一张表，二分区表就是主题的分区。

**术语解释**

| 术语          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| Topic         | 主题，可以理解为一个队列,可以有多个Topic                     |
| Partition     | 分区，为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序 |
| offset        | 偏移量，kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafkaOffset |
| Broker        | 一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic |
| Producer      | 消息生产者，向kafka broker发消息的客户端                     |
| Consumer      | 消息消费者，向kafka broker取消息的客户端                     |
| Group消费者组 | 这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partion只会把消息发给该CG中的一个consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic |

![1634970459399](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/142740-513794.png)

**生产者**： 向主题发布消息的客户端应用程序称为`生产者`（Producer），生产者用于持续不断的向某个主题发送消息。

**消费者**：订阅主题消息的客户端程序称为`消费者`（Consumer），消费者用于处理生产者产生的消息。

**消费者群组**：生产者与消费者的关系就如同餐厅中的厨师和顾客之间的关系一样，一个厨师对应多个顾客，也就是一个生产者对应多个消费者，`消费者群组`（Consumer Group）指的就是由一个或多个消费者组成的群体。

![1634970503069](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/142824-167740.png)

**偏移量**：`偏移量`（Consumer Offset）是一种元数据，它是一个不断**递增的整数值**，用来记录消费者发生重平衡时的位置，以便用来恢复数据。

**broker**: 一个独立的 Kafka 服务器就被称为 `broker`，broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。

**broker 集群**：broker 是`集群` 的组成部分，broker 集群由一个或多个 broker 组成，每个集群都有一个 broker 同时充当了`集群控制器`的角色（自动从集群的活跃成员中选举出来）。

**副本**：Kafka 中消息的备份又叫做 `副本`（Replica），副本的数量是可以配置的，Kafka 定义了两类副本：领导者副本（Leader Replica） 和 追随者副本（Follower Replica），前者对外提供服务，后者只是被动跟随。

**重平衡**：Rebalance。消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。或者说某一个消费者加入一个消费者组，那么就需要重新平衡消息的发送，Rebalance 是 Kafka 消费者端实现高可用的重要手段。

### 请说明什么是传统的消息传递方法？

传统的消息传递方法包括两种：

- 队列：在队列中，一组用户可以从服务器中读取消息，每条消息都发送给其中一个人，某个消费者消费数据之后，消息会随机删除。
- 发布-订阅：在这个模型中，消息被广播给所有的用户，可以有多个生产者和多个消费者。

### 数据传输的事务有几种？

数据传输的事务定义通常有以下三种级别：

- 最多一次（At most once）：消息不会被重复发送，最多被传输一次，但也有可能一次不传输

- 最少一次（At least one）：消息不会被漏发送，最少被传输一次，但也有可能被重复传输

- 精确的一次（Exactly once）：不会漏传输也不会重复传输，每个消息都传输被接收

### Kafka消息队列

Kafka 的消息队列一般分为两种模式：**点对点模式和发布订阅模式**

Kafka 是支持消费者群组的，也就是说 Kafka 中会有一个或者多个消费者，**如果一个生产者生产的消息由一个消费者进行消费的话，那么这种模式就是点对点模式**.

点对点模型通常是一个**基于拉取或者轮询的消息传送模型**，这种模型从队列中请求信息，而不是将消息推送到客户端。**这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。**

![1634970879317](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/143442-369189.png)

- 消费者去队列中拉去消息，即使有多个消费者，那生产者也会把消息发送到队列中，多个消费者轮询到队列中拉去消息。

发布订阅模式有两种形式，**一种是消费者主动拉取数据的形式，另一种是生产者推送消息的形式**，kafka是基于发布订阅模式中**消费者拉取**的方式。消费者的消费速度可以由消费者自己决定。但是这种方式也有缺点，当没有消息的时候，kafka的消费者还需要不停的访问kafka生产者拉取消息，浪费资源。

![1634971018569](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/143718-760603.png)

- 这种方式中，消费者和生产者都可以有多个。

### 请简述下你在哪些场景下会选择 Kafka？

日志收集：一个公司可以用Kafka收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、HBase、Solr等。

消息系统：解耦和生产者和消费者、缓存消息等

用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等

活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。

运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告

流式处理：比如spark streaming和 Flink

### Kafka 的设计架构？

**简单的设计架构**

![1634968102765](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/134824-348938.png)

**详细架构**

![1634968150612](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/134912-58211.png)

Kafka 架构分为以下几个部分：

1. Producer：消息生产者，就是向 kafka broker 发消息的客户端

2. Consumer：消息消费者，向 kafka broker 取消息的客户端

3. Topic：可以理解为一个队列，一个 Topic 又分为一个或多个分区，主题是逻辑上的划分。

4. Consumer Group：这是 kafka 用来实现一个 topic 消息的广播（发给所有的 consumer）和单播（发给任意一个consumer）的手段。一个 topic 可以有多个 Consumer Group

5. Broker：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic。

6. Partition：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker上，每个 partition 是一个有序的队列。partition 中的每条消息都会被分配一个有序的id（offset）。将消息发给 consumer，kafka

7. 只保证按一个 partition 中的消息的顺序，不保证一个 topic 的整体（多个 partition 间）的顺序

8. Offset：kafka 的存储文件都是按照 offset.kafka 来命名，用 offset 做名字的好处是方便查找。例如你想找位于 2049 的位置，只要找到 2048.kafka 的文件即可。当然 the first offset 就是00000000000.kafka。

### Kafka 分区的目的？

分区对于Kafka 集群的好处是：实现负载均衡。

分区对于消费者来说，可以提高并发度，提高效率，当然，对于是生产者也可以提高并行度的目的，将数据写入多个分区，提高写入速度，各个分区都是独立的，不会产生并发安全问题。

### Kafka消息是采用Pull模式，还是Push模式？

Kafka最初考虑的问题是，customer应该从brokes拉取消息还是brokers将消息推送到consumer，也就是pull还push。在这方面，Kafka遵循了一种大部分消息系统共同的传统的设计：**producer将消息推送到broker，consumer从broker拉取消息**。

一些消息系统比如Scribe和Apache Flume采用了push模式，将消息推送到下游的consumer。这样做有好处也有坏处：由broker决定消息推送的速率，对于不同消费速率的consumer就不太好处理了。消息系统都致力于让consumer以最大的速率最快速的消费消息，但不幸的是，push模式下，当broker推送的速率远大于consumer消费的速率时，consumer恐怕就要崩溃了。最终Kafka还是选取了传统的pull模式

Pull模式的另外一个好处是consumer可以自主决定是否批量的从broker拉取数据。Push模式必须在不知道下游consumer消费能力和消费策略的情况下决定是立即推送每条消息还是缓存之后批量推送。如果为了避免consumer崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。Pull模式下，consumer就可以根据自己的消费能力去决定这些策略

 Pull有个缺点是，如果broker没有可供消费的消息，将导致consumer不断在循环中轮询，直到新消息到t达。为了避免这点，Kafka有个参数可以让consumer阻塞知道新消息到达(当然也可以阻塞知道消息的数量达到某个特定的量这样就可以批量发。

### kafka消息队列优点或者对比

#### 请说明Kafka相对于传统的消息传递方法有什么优势？

1. 高性能：单一的Kafka代理可以处理成千上万的客户端，每秒处理数兆字节的读写操作，Kafka性能远超过传统的ActiveMQ、RabbitMQ等，而且Kafka支持Batch操作
2. 可扩展：Kafka集群可以透明的扩展，增加新的服务器进集群
3. 容错性： Kafka每个Partition数据会复制到几台服务器，当某个Broker失效时，Zookeeper将通知生产
   者和消费者从而使用其他的Broker。
4. 高吞吐、低延迟：kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒。
5. 高伸缩性： 每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。
6. 持久性、可靠性： Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失，Kafka 底层的数据存储是基于 Zookeeper 存储的，Zookeeper 我们知道它的数据能够持久存储。

#### Kafka与传统消息队列的区别？

在说区别的时候，我们先来看看kafka的应用场景：

kafka是个**日志处理缓冲组件**，在大数据信息处理中使用。和传统的消息队列相比较简化了队列结构和功能，**以流形式处理存储（持久化）消息（主要是日志）**。日志数据量巨大，处理组件一般会处理不过来，所以作为缓冲曾的kafka，支持巨大吞吐量。为了防止信息丢失，其消息被消费后不直接丢弃，要多存储一段时间，等过期时间过了才丢弃。这是mq和redis不能具备的。

主要特点入下:

巨型存储量: 

- 支持TB甚至PB级别数据。

高吞吐，高IO：
- 一般配置的服务器能实现单机每秒100K条以上消息的传输。

消息分区，分布式消费：
- 首先kafka会将接收到的消息分区（partition），每个主题（topic）的消息有不同的分区，这样一方面消息的存储就不会受到单一服务器存储空间大小的限制，另一方面消息的处理也可以在多个服务器上并行。也做到了负载均衡的目的，将数据均衡到堕胎服务器的多个分区中。
- 能保消息顺序传输。 支持离线数据处理和实时数据处理。

Scale out：
- 支持在线水平扩展，以支持更大数据处理量。

高可用机制：

- 其次为了保证高可用，每个分区都会有一定数量的副本(replica)。这样如果有部分服务器不可用，副本所在的服务器就会接替上来，保证应用的持续性。
- 然后保证分区内部消息的消费有序性。

消费者组：
- Kafka还具有consumer group的概念，每个分区只能被同一个group的一个consumer消费，但可以被多个group消费。

而传统的消息队列，比如Rides

redis只是提供一个高性能的、原子操作内存键值队，具有高速访问能力，可用做消息队列的存储，但是不具备消息队列的任何功能和逻辑，要作做为消息队列来实现的话，功能和逻辑要通过上层应用自己实现。

redis 消息推送（基于分布式 pub/sub）多用于实时性较高的消息推送，并不保证可靠。

作为消息队列来说，企业中选择mq的还是多数，因为像Rabbit，Rocket等mq中间件都属于很成熟的产品，性能一般但可靠性较强，而kafka原本设计的初衷是日志统计分析，现在基于大数据的背景下也可以做运营数据的分析统计，而redis的主要场景是内存数据库，作为消息队列来说可靠性太差，而且速度太依赖网络IO，在服务器本机上的速度较快，且容易出现数据堆积的问题，在比较轻量的场合下能够适用

还由MQ

我们以是RabbitMQ为例介绍。它是用Erlang语言开发的开源的消息队列，支持多种协议，包括AMQP，XMPP, SMTP, STOMP。适合于企业级的开发。

MQ支持Broker构架，消息发送给客户端时需要在中心队列排队。对路由，负载均衡或者数据持久化都有很好的支持。

### 使用消息队列的好处

- 解耦
  - 允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
- 可恢复性
  - 系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
- 缓冲
  - 有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。
- 灵活性  &峰值处理能力
  - 在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
- 异步通信
  - 很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。


### Kafka的设计是什么样的呢？

- Kafka将消息以topic为单位进行归纳

- 将向Kafka topic发布消息的程序成为producers.
- producers通过网络将消息发送到Kafka集群，集群向消费者提供消息

- 将预订topics并消费消息的程序成为consumer.

- Kafka以集群的方式运行，可以由一个或多个服务组成，每个服务叫做一个broker.



###  数据传输的事物定义有哪三种？

数据传输的事务定义通常有以下三种级别：

1. 最多一次: 消息不会被重复发送，最多被传输一次，但也有可能一次不传输，这种情况会发生数据的丢失。
2. 最少一次: 消息不会被漏发送，最少被传输一次，但也有可能被重复传输.
3. 精确的一次（Exactly once）: 不会漏传输也不会重复传输,每个消息都传输被一次而且仅仅被传输一次，这是大家所期望的，端到端的精确一次性最难保证。

### Kafka判断一个节点是否还活着有那两个条件？

1. 节点必须可以维护和ZooKeeper的连接，Zookeeper通过心跳机制检查每个节点的连接

2. 如果节点是个follower,他必须能及时的同步leader的写操作，延时不能太久，注意，这里的同步，同步的是isr中的follower节点。

### Kafa consumer是否可以消费指定分区消息？

Kafa consumer消费消息时，向broker发出"fetch"请求去消费**特定分区的消息**，consumer指定消息在日志中的偏移量（offset），就可以消费从这个位置开始的消息，**customer拥有了offset的控制权，可以向后回滚去重新消费之前的消息**，这是很有意义的。

### producer是否直接将数据发送到broker的leader(主节点)？

producer直接将数据发送到topic的leader(主节点)，不需要在多个节点进行分发，为了帮助producer做到这点，所有的Kafka节点都可以及时的告知:哪些节点是活动的，目标topic目标分区的leader在哪。这样producer就可以直接将消息发送到目的地了。

###  Kafka存储在硬盘上的消息格式是什么？

消息由一个**固定长度的头部和可变长度的字节数组**组成。头部包含了一个版本号和CRC32校验码。

- 消息长度: 4 bytes (value: 1+4+n)
- 版本号: 1 byte
- CRC校验码: 4 bytes
- 具体的消息: n bytes

### Kafka高效文件存储设计特点：

1. Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用,健儿来说就是通过一种分而治之的方法，逐步清除没有用的小文件。
2. 通过索引信息可以快速定位message和确定response的最大大小。
3. 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作。
4. 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小。

### Kafka工作流程

1. 消息分类按不同类别,分成不同的Topic,Topic⼜又拆分成多个partition,每个partition均衡分散到不同的服务器器(**提高并发访问的能力**)
2. 消费者按顺序从partition中读取,不支持随机读取数据,但可通过改变保存到zookeeper中的offset位置实现从任意位置开始读取。
3. 服务器消息定时清除(不管有没有消费)
4. 每个partition还可以设置备份到其他服务器上的个数以保证数据的可⽤性。**通过Leader,Follower方式**。
5. zookeeper保存kafka服务器和客户端的所有状态信息.(确保实际的客户端和服务器轻量级)
6. 在kafka中,一个partition中的消息只会被group中的一个consumer消费;每个group中consumer消息消费互相独立;我们可以认为一个group是一个"订阅"者,⼀个Topic中的每个partions,只会被一个"订阅者"中的一个consumer消费,不过⼀个consumer可以消费多个partitions中的消息。
7. 如果所有的consumer都具有相同的group（也就是所有的消费者再同一个消费者组）,这种情况和queue模式很像;消息将会在consumers之间负载均衡.
8. 如果所有的consumer都具有不同的group,那这就是"发布-订阅";消息将会广播给所有的消费者.
9. 持久性,当收到的消息时先buffer起来,等到了一定的阀值再写入磁盘文件,减少磁盘IO.在一定程度上依赖OS的文件系统(对文件系统本身优化几乎不可能)，所以在这个过程中，如果断电的话，存在消息的丢失。
10. 除了磁盘IO,还应考虑网络IO，批量对消息发送和接收,并对消息进行压缩。
11. 在JMS实现中,Topic模型基于push方式,即broker将消息推送给consumer端.不过在kafka中,采用了pull方式,即consumer在和broker建立连接之后,主动去pull(或者说fetch)消息;这种模式有些优点,首先consumer端可以根据⾃己的消费能力适时的去fetch消息并处理,且可以控制消息消费的进度(offset);此外,消费者可以良好的控制消息消费的数量,batch fetch.
12. kafka无需记录消息是否接收成功,是否要重新发送等,所以kafka的producer是非常轻量级的,consumer端也只需要将fetch后的offset位置注册到zookeeper,所以也是非常轻量级的.

### Kafka创建Topic时如何将分区放置到不同的Broker中

1. 副本因子不能大于 Broker 的个数；
2. 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的；
3. 其他分区的第一个副本放置位置相对于第0个分区依次往后移。也就是如果我们有5个 Broker，5个分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推；
4. 剩余的副本相对于第一个副本放置位置其实是由 nextReplicaShift 决定的，而这个数也是随机产生的。

### Kafka新建的分区会在哪个目录下创建

在启动 Kafka 集群之前，我们需要配置好 log.dirs 参数，其值是 Kafka 数据的存放目录，这个参数可以配置多个目录，目录之间使用逗号分隔，通常这些目录是分布在不同的磁盘上用于提高读写性能。

当然我们也可以配置 log.dir 参数，含义一样。只需要设置其中一个即可。

如果 log.dirs 参数只配置了一个目录，那么分配到各个 Broker 上的分区肯定只能在这个目录下创建文件夹用于存放数据。

但是如果 log.dirs 参数配置了多个目录，那么 Kafka 会在哪个文件夹中创建分区目录呢？

答案是：Kafka 会在含有分区目录最少的文件夹中创建新的分区目录，分区目录名为 Topic名+分区ID。注意，是分区文件夹总数最少的目录，而不是磁盘使用量最少的目录！也就是说，如果你给 log.dirs 参数新增了一个新的磁盘，新的分区目录肯定是先在这个新的磁盘上创建直到这个新的磁盘目录拥有的分区目录不是最少为止。

### partition的数据如何保存到硬盘

topic中的多个partition以文件夹的形式保存到broker（也就是每一个kafka的服务器上），每个分区序号从0递增，且消息有序，Partition文件下有多个segment（xxx.index，xxx.log），segment 文件里的 大小和配置文件大小一致可以根据要求修改 默认为1g，如果大小大于1g时，会滚动一个新的segment并且以上一个segment最后一条消息的偏移量命名。

### Kafka的消费者如何消费数据

消费者每次消费数据的时候，消费者都会记录消费的物理偏移量（offset）的位置，等到下次消费时，他会接着上次位置继续消费。

这个offset的位置是由zookeeper来维护的。

###  数据有序

一个消费者组里它的内部是有序的，消费者组与消费者组之间是无序的。

那么这里可以思考一些为什么是区间内有序，而各个分区之间无序呢？

分区内部有序，提高了kafka查询消息的效率，使用的是二分法查找，但是如果各个区间之间保证有序，那肯定就需要做一个全局的排序，这样做是否增加了系统的复杂程度。

### kafaka生产数据时数据的分组策略

生产者决定数据产生到集群的哪个partition中

每一条消息都是以（key，value）格式

Key是由生产者发送数据传入

所以生产者（key）决定了数据产生到集群的哪个partition

### kafka集群架构

![1639045527540](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/182528-493077.png)

### Kafka的工作机制

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/182809-557369.png)

上图表示`kafka`集群有一个`topic A`,并且有三个分区，分布在三个节点上面。

注意点：每个分区有两个副本，两个副本分别是`leader,follower`,并且每一个副本一定不和自己的`leader`分布在一个节点上面。`Kafka`中消息是以  **`topic`**进行分类的，生产者生产消息，消费者消费消息，都是面向  `topic`的。

`topic`是逻辑上的概念，而`partition`是物理上的概念，每个` partition`对应于一个` log`文件，该` log`文件中存储的就是  `producer`生产的数据。`Producer`生产的数据会被不断追加到该`log`文件末端，且每条数据都有自己的  `offset`。消费者组中的每个消费者，都会实时记录自己消费到了哪个 `offset`，以便出错恢复时，从上次的位置继续消费。每一个分区内部的数据是有序的，但是全局不是有序的。

### kafka文件存储结构

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/145145-384586.png)

由于生产者生产的消息会不断追加到`log`文件末尾，为防止`  log`文件过大导致数据定位效率低下，`Kafka`采取了**分片和索引机制**，将每个  `partition`分为多个 ` segment`。每个  `segment`对应两个文件`“.index”`文件和`“.log”`文件。这些文件位于一个文件夹下，该文件夹的命名规则为：`topic名称+分区序号`。例如，`first`这个  ` topic`有三个分区，则其对应的文件夹为  :

~~~ java
first-0,first-1,first-2
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
//index文件中存储的是每一个log文件的起始数据的偏移量
~~~

index和 ` log`文件以当前   `segment`的**第一条消息的  `offset`命名**。下图为   `index`文件和   `log`文件的结构示意图

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/145247-76106.png)

上面`kafka`再查找偏移量的时候是以**二分查找法进行查找的**。也就是查询index的时候使用的是二分查找法。

查找原理是：文件头的偏移量和文件大小快速定位。`“.index”`文件存储大量的索引信息，在查找`index`的时候使用的是二分查找法，`“.log”`文件存储大量的数据，索引文件中的元数据指向对应数据文件中 `message`的物理偏移地址。

### kafka应用场景

对于一些常规的消息系统,kafka是个不错的选择;**partitons/replication和容错**,可以使kafka具有良好的扩展性和性能优势，

kafka只能使用作为"常规"的消息系统,在一定程度上,尚未确保消息的发送与接收绝对可靠(⽐如,消息重发,消息发送丢失等)

kafka的特性决定它非常适合作为"日志收集中心";

application可以将操作日志"批量""异步"的发送到kafka集群中,而不是保存在本地或者DB中;

kafka可以批量提交消息/压缩消息等,这对producer端而言,几乎感觉不到性能的开支.

### kafka生产者写入数据

#### 副本

同一个`partition`可能会有多个`replication`（对应` server.properties `配置中default.replication.factor=N）没有`replication`的情况下，一旦`broker` 宕机，其上所有` patition` 的数据都不可被消费，同时producer也不能再将数据存于其上的`patition`。引入`replication`之后，同一个`partition`可能会有多个`replication`，而这时需要在这些`replication`之间选出一个`leader`，`producer`和`consumer`只与这个`leader`交互，其它`replication`的`follower`从leader 中复制数据,保证数据的一致性。

#### 写入方式

`producer`采用推`（push）`模式将消息发布到`broker`，每条消息都被追加`（append）`到分区`（patition）`中，属于**顺序写磁盘**（顺序写磁盘效率比随机写内存要高，保障`kafka`吞吐率）。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090838-364327.png)

1. producer先从`zookeeper`的 `"/brokers/.../state"`节点找到该`partition`的`leader`
2. `producer`将消息发送给该`leader`
3. `leader`将消息写入本地`log`
4. `followers`从`leader pull`消息，写入本地`log`后向`leader`发送`ACK`
5. `leader`收到所有`ISR`中的`replication`的`ACK`后，增加`HW（high watermark，最后commit 的offset）`并向`producer`发送`ACK`

#### broker保存消息

存储方式：物理上把`topic`分成一个或多个`patition`，每个`patition`物理上对应一个文件夹（该文件夹存储该`patition`的所有消息和索引文件）

#### 存储策略

无论消息是否被消费，`kafka`都会保留所有消息。有两种策略可以删除旧数据：

- 基于时间：`log.retention.hours=168`
- 基于大小：`log.retention.bytes=1073741824`

需要注意的是，因为`Kafka`读取特定消息的时间复杂度为`O(1)`，即与文件大小无关，所以这里删除过期文件与提高` Kafka `性能无关

#### 分区

消息发送时都被发送到一个`topic`，其本质就是一个目录，而`topic`是由一些`Partition Logs`(分区日志)组成，其组织结构如下图所示：

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090858-30434.png)

我们可以看到，每个`Partition`中的消息都是有序的，生产的消息被不断追加到`Partition log`上，其中的每一个消息都被赋予了一个唯一的`offset`值。

- 分区的原因
  - 方便在集群中扩展，每个`Partition`可以通过调整以适应它所在的机器，而一个`topic`又可以有多个`Partition`组成，因此整个集群就可以适应任意大小的数据了；
  - 可以提高并发，因为可以以`Partition`为单位读写了。
- 分区的原则
  - 指定了`patition`，则直接使用；
  - 未指定`patition`但指定`key`，通过对`key`的`value`进行`hash`出一个`patition`；
  - 既没有`  partition`值又没有    `key`值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 ` topic`可用的   ` partition`总数取余得到    `partition`值，也就是常说的 ` round-robin`算法（轮询算法）。.

### kafka写入数据可靠性保障

produce写入消息的可靠性保证

**数据写入的可靠性保证。**

为保证 `producer`发送的数据，能可靠的发送到指定的  `topic`，`topic`的每个`  partition`收到`producer`发送的数据后，都需要向`   producer`发送 `  ack（acknowledgement确认收到）`，如果`producer`收到 ` ack`，就会进行下一轮的发送，否则重新发送数据。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090843-235649.png)

- 副本数据同步策略

| 方案                              | 优点                                                   | 缺点                                                |
| --------------------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| 半数以上同步完成，发送`ack`确认   | 延迟低                                                 | 选举新的节点时，容忍`n`个节点故障，需要`2n+1`个副本 |
| 全部同步完成以后，才发送`ack`确认 | 选举新的`leader`时，容忍`n`台节点故障，需要`n+1`个副本 | 延迟低                                              |

`Kafka`选择了第二种方案，原因如下：

- 同样为了容忍 `n`台节点的故障，第一种方案需要`2n+1`个副本，而第二种方案只需要`n+1`个副本，而 `Kafka`的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。
- 虽然第二种方案的网络延迟会比较高，但网络延迟对` Kafka`的影响较小。

**ISR**:

采用第二种方案之后，设想以下情景：`leader`收到数据，所有`follower`都开始同步数据，但有一个 `follower`，因为某种故障，迟迟不能与` leader`进行同步，那  `leader`就要一直等下去，直到它完成同步，才能发送 `ack`。这个问题怎么解决呢？

- `Leader`维护了一个动态的 ` in-sync replica set (ISR)`，意为和 `leader`保持同步的 `follower`集合，是一个队列。当 `ISR`中的 ` follower`完成数据的同步之后，`leader`就会给 `follower`发送  `ack`。如果  `follower`长时间未向`leader`同步数据，则该`follower`将被踢出`ISR`，该时间阈值由`replica.lag.time.max.ms`参数设定。`Leader`发生故障之后，就会从`ISR`中选举新的`leader`。在这个时间内，就添加到isr中，否则就提出isr集合中，不在isr中的follower也不可能被选举为leader。

~~~java
rerplica.lag.time.max.ms=10000
//如果leader发现flower超过10秒没有向它发起fech请求, 那么leader考虑这个flower是不是程序出了点问题，或者资源紧张调度不过来, 它太慢了, 不希望它拖慢后面的进度, 就把它从ISR中移除.
rerplica.lag.max.messages=4000
//相差4000条就移除，flower慢的时候, 保证高可用性, 同时满足这两个条件后又加入ISR中,在可用性与一致性做了动态平衡
min.insync.replicas=1
//需要保证ISR中至少有多少个replica
~~~

**ack**应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等` ISR`中的`follower`全部接收成功。所以` Kafka`为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。

`ACKS`参数配置：

- 0：`producer`不等待  `broker`的`ack`,这一操作提供了一个最低的延迟，`broker`一接收到还没有写入磁盘就已经返回，**当 `broker`故障时有可能丢失数据；**
- 1：`producer`等待  `broker`的`ack`，`partition`的  `leader`落盘成功后返回`ack`，如果在`follower`同步成功之前 `leader`故障，那么将会丢失数据；

**ack=1也可能丢失数据**

![1618623903699](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/094505-503238.png)

- -1（all）：`producer`等待  `broker`的`ack`，`partition`的`leader`和`follower`（这里指的是isr中的follower）全部落盘成功后才返回 `ack`。但是如果在 `follower`同步完成后，`broker`发送`ack`之前，`leader`发生故障，那么会造成数据重复。

  =-1也可能会发生数据的丢失，发生重复数据的情况是leader接收到数据并且在follower之间已经同步完成后，但是此时leader挂掉，没有返回ack确认，此时又重新选举产生了leader,那么producer会重新发送一次数据，所以会导致数据重复。

**小结**

~~~java
request.required.asks=0
//0:相当于异步的, 不需要leader给予回复, producer立即返回, 发送就是成功,那么发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息2.Leader与Follower数据不同步), 既有可能丢失也可能会重发
//1：当leader接收到消息之后发送ack, 丢会重发, 丢的概率很小
//-1：当所有的follower都同步消息成功后发送ack. 不会丢失消息
~~~

**ack=-1数据重复案例**

![1618625608045](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/101329-814647.png)

> **ack是保证生产者生产的数据不丢失，hw是保证消费者消费数据的一致性问题**。hw实际就是最短木桶原则，根据这个原则消费者进行消费数据。不能解决数据重复和丢失问题。ack解决丢失和重复问题。

**故障处理细节**

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090845-425507.png)

​	**`LEO`**：指的是每个副本最大的 **`offset`**，也就是灭一个副本的最后offset值。

​	**`HW`**：指的是消费者能见到的最大的 **offset**，**ISR**队列中最小的  **LEO**。

**follower**故障

`follower`发生故障后会被临时踢出 ` ISR`，待该` follower`恢复后，`follower`会读取本地磁盘记录的上次的 `HW`，并将 `log`文件高于 ` HW`的部分截取掉，从  `HW`开始向`  leader`进行同步。等该 **`follower`**的  **`LEO`**大于等于该  **`Partition`**的  **`HW`**，即` follower`追上 ` leader`之后，就可以重新加入` ISR`了。

**leader**故障

`leader`发生故障之后，会从 `ISR`中选出一个新的  `leader`，之后，为保证多个副本之间的数据一致性，其余的 `follower`会先将各自的`  log`文件高于  `HW`的部分截掉，然后从新的 `leader`同步数据。

**注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复,`ack`确认机制可以保证数据的不丢失和不重复，`LEO`和`hw`可以保证数据的一致性问题**


![1639045936250](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/183219-639145.png)

![1639046087062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/183448-152461.png)

### kafka的ack机制

request.required.acks有三个值 0 1 -1

- 0:生产者不会等待broker的ack，这个延迟最低但是存储的保证最弱当server挂掉的时候就会丢数据
- 1：服务端会等待ack值 leader副本确认接收到消息后发送ack但是如果leader挂掉后他不确保是否复制完成新leader也会导致数据丢失
- -1：同样在1的基础上 服务端会等所有的follower的副本受到数据后才会受到leader发出的ack，这样数据不会丢失

#### **Exactly Once**语义

将服务器的 `ACK`级别设置为`1`，可以保证`Producer`到`Server`之间不会丢失数据，即`At Least Once`语义。相对的，将服务器`ACK`级别设置为`0`,可以保证生产者每条消息只会被发送一次，即 `At Most Once`语义。`At Least Once`可以保证数据不丢失，但是不能保证数据不重复；相对的，`At Most Once`可以保证数据不重复，但是不能保证数据不丢失。

但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即` Exactly  Once`语义。在  `0.11`版本以前的` Kafka`，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。`0.11`版本的  `Kafka`，引入了一项重大特性：**幂等性。所谓的幂等性就是指 `Producer`不论向` Server`发送多少次重复数据，`Server`端都只会持久化一条。**

幂等性结合`At  Least Once`语义，就构成了 `Kafka`的 ` Exactly Once`语义。即：`At Least Once +幂等性= Exactly Once`要启用幂等性，只需要将 `Producer`的参数中  `enable.idompotence`设置为`true`即可。`Kafka`的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的` Producer`在初始化的时候会被分配一个  `PID`，发往同一  `Partition`的消息会附带`Sequence Number`。而`Broker`端会对`<PID,  Partition, SeqNumber>`做缓存，当具有相同主键的消息提交时，`Broker`只会持久化一条。但是 `PID`重启就会变化，同时不同的 ` Partition`也具有不同主键，所以幂等性无法保证跨分区跨会话（也就是重新建立producer链接的情况）的 `Exactly Once`。**即只能保证单次会话不重复问题。幂等性只能解决但回话单分区的问题**。

### kafka消费者

#### 消费方式

- `consumer`采用 ` pull`（拉）模式从 `broker`中读取数据。
- `push`（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 `consumer`来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而` pull`模式则可以根据  `consumer`的消费能力以适当的速率消费消息。对于`Kafka`而言，`pull`模式更合适，它可简化`broker`的设计，`consumer`可自主控制消费消息的速率，同时`consumer`可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义
- `pull`模式不足之处是，如果 ` kafka`没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，`Kafka`的消费者在消费数据时会传入一个时长参数 ` timeout`，如果当前没有数据可供消费，`consumer`会等待一段时间之后再返回，这段时长即为  `timeout`。

#### Consumer Group

在 Kafka 中, 一个 Topic 是可以被一个消费组消费, 一个Topic 分发给 Consumer Group 中的Consumer 进行消费, 保证同一条 Message 不会被不同的 Consumer 消费。

注意: 当Consumer Group的 Consumer 数量大于 Partition 的数量时, 超过 Partition 的数量将会拿不到消息

#### 分区分配策略

一个` consumer group`中有多个`consumer`，一个  `topic`有多个`partition`，所以必然会涉及到`partition`的分配问题，即确定那个`partition`由哪个`consumer`来消费。`Kafka`有两种分配策略：
- 一是  `RoundRobin`（按照组分配）
- 一是 `Range`（按照主题分配）。

同一个消费者组中的不同消费者，不能同时消费同一个分区，但是可以同时消费一个主题，因为一个主题中有多个分区。

Kafka分配Replica的算法有两种: RangeAssignor 和 RoundRobinAssignor

默认为RangeAssignor:

1. 将所有Broker(假设共n个Broker)和待分配的Partition排序
2. 将第i个Partition分配到第(i mod n)个Broker上
3. 将第i个Partition的第j个Replica分配到第((i + j) mod n)个Broker上

**按照消费者组中的消费者进行轮询：**

![1618629372089](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/115925-934544.png)

如下图：

![1618627826708](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/105027-10616.png)

图中有一个消费者组和两个topic，kafka会把两个topic当做一个整体来考虑，根据两个topic中的所有分区，使用分区的哈希值对其进行排序，然后在轮训分发到消费者组中的每一个消费者。好处是多个消费者之间消费分区的个数最多差一个。我们说到一个消费者组中的所有消费者可以消费同一个分区中的不同分区。

但是使用轮训的方式需要有一个条件，也就是保证当前消费者组中所有消费者订阅的主题topic是一样的。比如下面这种情况可能吧t1的数据发送给b,把t3的数据发送给a，

![1618628205143](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/24/123619-581189.png)

（按照主题分配），kafka默认使用的分区策略

是按照单个主题进行划分的，比如当前主题中有7个分区，有三个消费者，那么就一个消费者消费3个分区的数据，另外两个消费者消费2个分区的数据。

![1618628385070](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/105946-426265.png)

下面有两个主题，每一个主题中三个分区，消费者组中有两个消费者，三个分区的数据，a消费者消费2个，b消费者消费1个，对于topic2，三个分区数据，a消费者消费2个，b消费者消费1个，所以导致a,b消费者消费的分区数据差距越来越大，这是一个缺点。也就是消费者消费数据不对等的问题。

![1618628574628](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/24/121718-243035.png)

当消费者组中的消费者个数发生变化时，增加或者减少都会触发重新分配策略。轮询方式避免了多个消费者消费数据不均衡问题。但是`range`方式可能会带来不均衡问题，轮询方式中，每次都是把消息轮换的发送给每一个消费者，所以没有分配不均衡问题的存在，但是按照主题分配就不一定能避免分配不均衡问题，应为是按照主题进行分配的。

**案例**

![1618730431904](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/18/152147-342731.png)

可以看到，消费者组1中a订阅主题T1，消费者b订阅主题T1,T2，消费者组2中消费者c订阅主题T1，如果此时采用轮训的方法进行分配数据，那么是以组为单位进行分发数据，t2主题给消费者组1发送数据，可能吧数据分发给消费者A，所以此时不能采用轮询的方式发送数据，如果采用range方式发送数据，是以主题为单位发送数据，那么主题t1会把数据01发送给消费者A，把数据2发送给消费者b，而主题T2则会把数据发送给消费者b。按照主题划分一定是看谁订阅主题，那个消费者订阅主题，就发送给哪一个消费者。按照主题划分，会先找到哪一个消费者订阅了此主题，然后在考虑组。消费者优先，然后在考虑组。

### Rebalance (重平衡)

Rebalance 本质上是一种协议, 规定了一个 Consumer Group 下的所有 consumer 如何达成一致,来分配订阅 Topic 的每个分区。

Rebalance 发生时, 所有的 Consumer Group 都停止工作, 知道 Rebalance完成。

#### Coordinator

kafka0.9之后：

Group Coordinator 是一个服务, 每个 Broker 在启动的时候都会启动一个该服务， Group Coordinator 的作用是用来存储 Group 的相关 Meta 信息, 并将对应 Partition 的 Offset 信息记录到 Kafka 内置 Topi(__consumer_offsets)中。

Kafka 在0.9之前是基于 Zookeeper 来存储Partition的 offset信息(consumers/{group}/offsets/{topic}/{partition}), 因为 Zookeeper 并不适用于频繁的写操作, 所以在0.9之后通过内置 Topic 的方式来记录对应 Partition 的 offset。

#### 触发条件

1. 组成员个数发生变化
   1. 新的消费者加入到消费组
   2. 消费者主动退出消费组
   3. 消费者被动下线. 比如消费者长时间的GC, 网络延迟导致消费者长时间未向Group
       Coordinator发送心跳请求, 均会认为该消费者已经下线并踢出
2. 订阅的 Topic 的 Consumer Group 个数发生变化
3. Topic 的分区数发生变化

#### Rebalace 流程

Rebalance 过程分为两步：Join 和 Sync

1. Join: 顾名思义就是加入组. 这一步中, 所有成员都向 Coordinator 发送 JoinGroup 请求, 请求
    加入消费组. 一旦所有成员都发送了 JoinGroup 请求, Coordinator 会从中选择一个
    Consumer 担任 Leader 的角色, 并把组成员信息以及订阅信息发给 Consumer Leader 注意
    Consumer Leader 和 Coordinator不是一个概念. Consumer Leader负责消费分配方案的制
    定
2. Sync: Consumer Leader 开始分配消费方案, 即哪个 Consumer 负责消费哪些 Topic 的哪些
    Partition. 一旦完成分配, Leader 会将这个方案封装进 SyncGroup 请求中发给 Coordinator,
    非 Leader 也会发 SyncGroup 请求, 只是内容为空. Coordinator 接收到分配方案之后会把方
    案塞进SyncGroup的Response中发给各个Consumer. 这样组内的所有成员就都知道自己应
    该消费哪些分区了

![1639046633091](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/143629-409917.png)

#### 如何避免 Rebalance

对于触发条件的 2 和 3, 我们可以人为避免. 1 中的 1 和 3 人为也可以尽量避免, 主要核心为 3

~~~java
心跳相关
session.timeout.ms = 6s
heartbeat.interval.ms = 2s

消费时间
max.poll.interval.ms
~~~

### 日志索引

Kafka 能支撑 TB 级别数据, 在日志级别有两个原因: 

- 顺序写
- 日志索引. 

顺序写后续会讲。

Kafka 在一个日志文件达到一定数据量 (1G) 之后, 会生成新的日志文件, 大数据情况下会有多个日
志文件, 通过偏移量来确定到某行纪录时, 如果遍历所有的日志文件, 那效率自然是很差的. Kafka
在日志级别上抽出来一层日志索引, 来方便根据 offset 快速定位到是某个日志文件
每一个 partition 对应多个个 log 文件(最大 1G), 每一个 log 文件又对应一个 index 文件
通过 offset 查找 Message 流程:

1. 先根据 offset (例: 368773), 二分定位到最大 小于等于该 offset 的 index 文件
    (368769.index)
2. . 通过二分(368773 - 368769 = 4)定位到 index 文件 (368769.index) 中最大 小于等于该
    offset 的 对于的 log 文件偏移量(3, 497)
3. 通过定位到该文件的消息行(3, 497), 然后在往后一行一行匹配揭露(368773 830)

![1639046789714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/09/184632-876670.png)

### Kafka 是如何实现高吞吐率的？

Kafka是分布式消息系统，需要处理海量的消息，Kafka的设计是把所有的消息都写入速度低容量大的硬盘，以此来换取更强的存储能力，但实际上，使用硬盘并没有带来过多的性能损失。kafka主要使用了以下几个方式实现了超高的吞吐率：

1. 顺序读写

2. 零拷贝

![20211211152243](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211211152243.png)

3. 文件分段

4. 批量发送

5. 数据压缩

### 高性能高吞吐

#### 分区的原因

如果我们假设像标准 MQ 的 Queue, 为了保证一个消息只会被一个消费者消费, 那么我们第一想到的就是**加锁**. 对于发送者, 在多线程并且非顺序写环境下, 保证数据一致性, 我们同样也要加锁. 一旦考虑到加锁, 就会极大的影响性能. 

我们再来看Kafka 的 Partition, Kafka 的消费模式和发送模式都是以 Partition 为分界. 也就是说对于一个 Topic 的并发量限制在于有多少个 Partition, 就能支撑多少的并发. 可以参考 Java 1.7 的 ConcurrentHashMap 的桶设计, 原理一样, 有多少桶, 支持多少的并发

在这里简单的说一些零拷贝技术：

简单的来说，比如用户读取一个文件，那么首先文件会读取到内存的缓冲区，然后在从缓冲区通过网络将数据发送给用户。可以抽象为下面两部：

```java
read(file, tmp_buf, len); 
write(socket, tmp_buf, len);
```

但是实际上，中间经历了四个过程，因为读取文件需要在用户态和和心态之间相互转换：
1. 程序调用 **read 产生一次用户态到内核态的上下文切换**。DMA 模块从磁盘读取文件内容，将其拷贝到内核空间的缓冲区，完成第 1 次拷贝。
2. 数据从内核缓冲区拷贝到用户空间缓冲区，之后系统调用 read 返回，这回导致从内核空间到用户空间的上下文切换。这个时候数据存储在用户空间的 tmp_buf 缓冲区内，可以后续的操作了。
3. 程序调用 write 产生一次用户态到内核态的上下文切换。数据从用户空间缓冲区被拷贝到内核空间缓冲区，完成第 3 次拷贝。但是这次数据存储在一个和 socket 相关的缓冲区中，而不是第一步的缓冲区。
4. write 调用返回，产生第 4 个上下文切换。第 4 次拷贝在 DMA 模块将数据从内核空间缓冲区传递至协议引擎的时候发生，这与我们的代码的执行是独立且异步发生的。你可能会疑惑：“为何要说是独立、异步？难道不是在 write 系统调用返回前数据已经被传送了？write 系统调用的返回，并不意味着传输成功——它甚至无法保证传输的开始。调用的返回，只是表明以太网驱动程序在其传输队列中有空位，并已经接受我们的数据用于传输。可能有众多的数据排在我们的数据之前。除非驱动程序或硬件采用优先级队列的方法，各组数据是依照FIFO的次序被传输的(上图中叉状的 DMA copy 表明这最后一次拷贝可以被延后)。

**Mmap**

上面的数据拷贝非常多，我们可以减少一些重复拷贝来减少开销，提升性能。某些硬件支持完全绕开内存，将数据直接传送给其他设备。这个特性消除了系统内存中的数据副本，因此是一种很好的选择，但并不是所有的硬件都支持。此外，来自于硬盘的数据必须重新打包(**地址连续**)才能用于网络传输，这也引入了某些复杂性。为了减少开销，我们可以从消除内核缓冲区与用户缓冲区之间的拷贝开始。

减少数据拷贝的一种方法是将 read 调用改为 mmap。例如：
```java
tmp_buf = mmap(file, len); 
write(socket, tmp_buf, len);
```

mmap 调用导致文件内容通过 DMA 模块拷贝到内核缓冲区。然后与用户进程共享缓冲区，这样不会在内核缓冲区和用户空间之间产生任何拷贝。

write 调用导致内核将数据从原始内核缓冲区拷贝到与 socket 关联的内核缓冲区中。

第 3 次数据拷贝发生在 DMA 模块将数据从 socket 缓冲区传递给协议引擎时。


#### 顺序写

磁盘的顺序写的性能要比内存随机写的还要强. 磁盘顺序写和随机写的差距也是天壤之别

#### 批发送

批处理是一种常用的用于**提高I/O性能**的方式. 对Kafka而言, 批处理既减少了网络传输的Overhead, 又提高了写磁盘的效率. Kafka 0.82 之后是将多个消息合并之后再发送, 而并不是send一条就立马发送(之前支持)

~~~ java
# 批量发送的基本单位, 默认是16384Bytes, 即16kB
batch.size
# 延迟时间
linger.ms
# 两者满足其一便发送
~~~

#### 数据压缩

数据压缩的一个基本原理是, 重复数据越多压缩效果越好. 因此将整个Batch的数据一起压缩能更大幅度减小数据量, 从而更大程度提高网络传输效率。

Broker接收消息后，并不直接解压缩，而是直接将消息以压缩后的形式持久化到磁盘， Consumer接受到压缩后的数据再解压缩，

整体来讲: Producer 到 Broker, 副本复制, Broker 到 Consumer 的数据都是压缩后的数据, 保证高效率的传输

### 解释如何减少ISR中的扰动？broker什么时候离开ISR？


ISR是一组与leaders完全同步的消息副本，也就是说ISR中包含了所有提交的消息。ISR应该总是包含所有的副本，直到出现真正的故障。

如果一个副本从leader中脱离出来，将会从ISR中删除。那么leader挂掉之后，就会从isr中选取新的leader.

isr就像nameNode和SecondnAMEnODE一样，保存这和Leader完全同步的数据。

### ISR、OSR、AR 是什么？

- ISR：In-Sync Replicas 副本同步队列

- OSR：Out-of-Sync Replicas

- AR：Assigned Replicas 所有副本

ISR是由leader维护，follower从leader同步数据有一些延迟（具体可以参见 图文了解 Kafka 的副本复制机制），超过相应的阈值会把 follower 剔除出 ISR, 存入OSR（Out-of-Sync Replicas ）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR

### LEO、HW、LSO、LW等分别代表什么？

LEO：是 LogEndOffset 的简称，代表当前日志文件中下一条。

HW：水位或水印（watermark）一词，也可称为高水位(high watermark)，通常被用在流式处理领域（比如Apache Flink、Apache Spark等），以表征元素或事件在基于时间层面上的进度。在Kafka中，水位的概念反而与时间无关，而是与位置信息相关。严格来说，它表示的就是位置信息，即位移（offset）。取 partition 对应的 ISR中 最小的 LEO 作为 HW，consumer 最多只能消费到 HW 所在的位置上一条信息

LSO：是 LastStableOffset 的简称，对未完成的事务而言，LSO 的值等于事务中第一条消息的位置(firstUnstableOffset)，对已完成的事务而言，它的值同 HW 相同。

LW：Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值

### Kafka为什么需要复制？

Kafka的信息复制确保了任何已发布的消息不会丢失，并且可以在机器错误、程序错误或更常见些的软件升级中使用。


请说明Kafka 的消息投递保证（delivery guarantee）机制以及如何实现？

Kafka支持三种消息投递语义：

1. At most once：消息可能会丢，但绝不会重复传递

2. At least one：消息绝不会丢，但可能会重复传递

3. Exactly once：每条消息肯定会被传输一次且仅传输一次，很多时候这是用户想要的

consumer在从broker读取消息后，可以选择commit，该操作会在Zookeeper中存下该consumer在该partition下读取的消息的offset，该consumer下一次再读该partition时会从下一条开始读取。如未commit，下一次读取的开始位置会跟上一次commit之后的开始位置相同，可以将consumer设置为autocommit，即consumer一旦读到数据立即自动commit。

如果只讨论这个读取消息的过程，那Kafka是确保了Exactly once。但实际上实际使用中consumer并非读取完数据就结束了，而是要进行进一步处理，而**数据处理与commit的顺序**在很大程度上决定了消息从broker和consumer的delivery guarantee semantic。

读完消息先commit再处理消息。这种模式下，如果consumer在commit后还没来得及处理消息就crash了，下次重新开始工作后就无法读到刚刚已提交而未处理的消息，这就对应于At most once。

读完消息先处理再commit消费状态(保存offset)。这种模式下，如果在处理完消息之后commit之前Consumer crash了，下次重新开始工作时还会处理刚刚未commit的消息，实际上该消息已经被处理过了，这就对应于At least once

如果一定要做到Exactly once，就需要协调offset和实际操作的输出。经典的做法是引入两阶段提交，

但由于许多**输出系统不支持两阶段提交**，更为通用的方式是将offset和操作输入存在同一个地方。比如，consumer拿到数据后可能把数据放到HDFS，如果把最新的offset和数据本身一起写到HDFS，就可以保证数据的输出和offset的更新要么都完成，要么都不完成，间接实现Exactly once。（目前就high level API而言，offset是存于Zookeeper中的，无法存于HDFS，而low level API的offset是由自己去维护的，可以将之存于HDFS中）

总之，Kafka默认保证At least once，并且允许通过设置producer异步提交来实现At most once，而Exactly once要求与目标存储系统协作，Kafka提供的offset可以较为容易地实现这种方式。

### 如何保证Kafka的消息有序

Kafka对于消息的重复、丢失、错误以及顺序没有严格的要求

Kafka只能保证一个partition中的消息被某个consumer消费时是顺序的，事实上，从Topic角度来说，

当有多个partition时，消息仍然不是全局有序的。

### Kafka 的高可靠性是怎么实现的？

注意：也可回答“Kafka在什么情况下会出现消息丢失？”数据可靠性（可回答“怎么尽可能保证Kafka的可靠性？”）

Kafka 作为一个商业级消息中间件，消息可靠性的重要性可想而知。本文从Producter向Broker发送消息、Topic 分区副本以及 Leader选举几个角度介绍数据的可靠性。

#### Topic分区副本

在 Kafka 0.8.0 之前，Kafka 是没有副本的概念的，那时候人们只会用 Kafka 存储一些不重要的数据，

因为没有副本，数据很可能会丢失。但是随着业务的发展，支持副本的功能越来越强烈，所以为了保证

数据的可靠性，Kafka 从 0.8.0 版本开始引入了分区副本（详情请参见 KAFKA-50）。也就是说每个分区可以人为的配置几个副本（比如创建主题的时候指定 replication-factor，也可以在 Broker 级别进行配置 default.replication.factor），一般会设置为3

Kafka 可以保证单个分区里的事件是有序的，分区可以在线（可用），也可以离线（不可用）。在众多的分区副本里面有一个副本是 Leader，其余的副本是 follower，所有的读写操作都是经过 Leader 进行的，同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候，其中一个 follower 会重新成为新的 Leader。通过分区副本，引入了数据冗余，同时也提供了 Kafka 的数据可靠性。

Kafka 的**分区多副本架构**是 Kafka 可靠性保证的核心，把消息写入多个副本可以使 Kafka 在发生崩溃时仍能保证消息的持久性。

#### Producer往Broker 发送消息

如果我们要往 Kafka 对应的主题发送消息，我们需要通过 Producer 完成。前面我们讲过 Kafka 主题对应了多个分区，每个分区下面又对应了多个副本；为了让用户设置数据可靠性， Kafka 在 Producer 里面提供了消息确认机制。也就是说我们可以通过配置来决定消息发送到对应分区的几个副本才算消息发送成功。可以在定义 Producer 时通过 acks 参数指定（在 0.8.2.X 版本之前是通过request.required.acks 参数设置的）。

这个参数支持以下三种值：

acks = 0：意味着如果生产者能够通过网络把消息发送出去，那么就认为消息已成功写入Kafka。在这种情况下还是有可能发生错误，比如发送的对象无能被序列化或者网卡发生故障，但如果是分区离线或整个集群长时间不可用，那就不会收到任何错误。在 acks=0模式下的运行速度是非常快的（这就是为什么很多基准测试都是基于这个模式），你可以得到惊人的吞吐量和带宽利用率，不过如果选择了这种模式，一定会丢失一些消息。

acks = 1：意味若 Leader 在收到消息并把它写入到分区数据文件（不一定同步到磁盘上）时会返回确认或错误响应。在这个模式下，如果发生正常的 Leader 选举，生产者会在选举时收到一个
LeaderNotAvailableException 异常，如果生产者能恰当地处理这个错误，它会重试发送悄息，最终消息会安全到达新的 Leader 那里。不过在这个模式下仍然有可能丢失数据，比如消息已经成功写入Leader，但在消息被复制到 follower 副本之前 Leader发生崩溃

acks = all（这个和 request.required.acks = -1 含义一样）：意味着 Leader 在返回确认或错误响应之前，会等待所有同步副本都收到悄息。如果和 min.insync.replicas 参数结合起来，就可以决定在返回确认前至少有多少个副本能够收到悄息，生产者会一直重试直到消息被成功提交。不过这也是最慢的做法，因为生产者在继续发送其他消息之前需要等待所有副本都收到当前的消息

根据实际的应用场景，我们设置不同的 acks，以此保证数据的可靠性

#### Leader 选举

在介绍 Leader 选举之前，让我们先来了解一下 ISR（in-sync replicas）列表。每个分区的 leader 会维护一个 ISR 列表，ISR 列表里面就是 follower 副本的 Borker 编号，只有跟得上 Leader 的 follower 副本才能加入到 ISR 里面，这个是通过 replica.lag.time.max.ms 参数配置的。只有 ISR 里的成员才有被选为 leader 的可能。

#### 数据一致性（可回答“Kafka数据一致性原理？”）

这里介绍的数据一致性主要是说不论是老的 Leader 还是新选举的 Leader，Consumer 都能读到一样的数据。那么 Kafka 是如何实现的呢？

![1634968962472](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/140242-153085.png)

假设分区的副本为3，其中副本0是 Leader，副本1和副本2是 follower，并且在 ISR 列表里面。虽然副本0已经写入了 Message4，但是 Consumer 只能读取到 Message2。因为所有的 ISR 都同步了Message2，只有 High Water Mark 以上的消息才支持 Consumer 读取，而 High Water Mark 取决于

ISR 列表里面偏移量最小的分区，对应于上图的副本2，这个很类似于木桶原理这样做的原因是还没有被足够多副本复制的消息被认为是“不安全”的，如果 Leader 发生崩溃，另一个副本成为新 Leader，那么这些消息很可能丢失了。如果我们允许消费者读取这些消息，可能就会破坏一致性。试想，一个消费者从当前 Leader（副本0） 读取并处理了 Message4，这个时候 Leader 挂掉了，选举了副本1为新的 Leader，这时候另一个消费者再去从新的 Leader 读取消息，发现这个消息其实并不存在，这就导致了数据不一致性问题

当然，引入了 High Water Mark 机制，会导致 Broker 间的消息复制因为某些原因变慢，那么消息到达消费者的时间也会随之变长（因为我们会先等待消息复制完毕）。延迟时间可以通过参数
replica.lag.time.max.ms 参数配置，它指定了副本在复制消息时可被允许的最大延迟时间。

### Kafka 缺点？

1. 由于是批量发送，数据并非真正的实时

2. 对于mqtt协议不支持

3. 不支持物联网传感数据直接接入

4. 仅支持统一分区内消息有序，无法实现全局消息有序

5. 监控不完善，需要安装插件

6. 依赖zookeeper进行元数据管理。

### Kafka 分区数可以增加或减少吗？为什么？

我们可以使用 bin/kafka-topics.sh 命令对 Kafka 增加 Kafka 的分区数据，但是 Kafka 不支持减少分区数。 Kafka 分区数据不支持减少是由很多原因的，比如减少的分区其数据放到哪里去？是删除，还是保留？删除的话，那么这些没消费的消息不就丢了。如果保留这些消息如何放到其他分区里面？追加到其分区后面的话那么就破坏了 Kafka 单个分区的有序性。如果要保证删除分区数据插入到其他分区保证有序性，那么实现起来逻辑就会非常复杂.

### Kafka消息可靠性的保证

![1639191012755](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/105014-372899.png)

Kafka存在丢消息的问题，消息丢失会发生在**Broker，Producer和Consumer**三种。

#### Broker

**Broker丢失消息是由于Kafka本身的原因造成的，kafka为了得到更高的性能和吞吐量，将数据异步批量的存储在磁盘中**。消息的刷盘过程，为了提高性能，减少刷盘次数，kafka采用了批量刷盘的做法。即，按照一定的消息量，和时间间隔进行刷盘。

那么kafka首先会将数据写入内存页中，系统通过刷盘的方式将数据持久化到内存当中，但是在存储在内存那一会，如果发生断电行为，内存中的数据是有可能发生丢失的，也就是说kafka中的数据可能丢失。

Broker配置刷盘机制，是通过调用fsync函数接管了刷盘动作。从单个Broker来看，pageCache的数据会丢失。

也就是说，理论上，要完全让kafka保证单个broker不丢失消息是做不到的，只能通过调整刷盘机制的参数缓解该情况。

- 比如，减少刷盘间隔，减少刷盘数据量大小，也就是频繁的刷盘操作。时间越短，性能越差，可靠性越好(尽可能可靠)。这是一个选择题。

- 而减少刷盘频率，可靠性不高，但是性能好。

为了解决该问题，kafka通过producer和broker协同处理单个broker丢失参数的情况。一旦producer发现broker消息丢失，即可自动进行retry。除非retry次数超过阀值(可配置)，消息才会丢失。此时需要生产者客户端手动处理该情况。那么producer是如何检测到数据丢失的呢？是通过ack机制，类似于http的三次握手的方式。

- acks=0，producer不等待broker的响应，效率最高，但是消息很可能会丢。这种情况也就是说还没有等待leader同步完数据，所以肯定发生数据的丢失。

- acks=1，leader broker收到消息后，不等待其他follower的响应，即返回ack。也可以理解为ack数为1。此时，如果follower还没有收到leader同步的消息leader就挂了，那么消息会丢失，也就是如果leader收到消息，成功写入PageCache后，会返回ack，此时producer认为消息发送成功。但此时，数据还没有被同步到follower。如果此时leader断电，数据会丢失。

- acks=-1，leader broker收到消息后，挂起，等待所有ISR列表中的follower返回结果后，再返回ack。-1等效与all。这种配置下，只有leader写入数据到pagecache是不会返回ack的，还需要所有的ISR返回“成功”才会触发ack。如果此时断电，producer可以知道消息没有被发送成功，将会重新发送。如果在follower收到数据以后，成功返回ack，leader断电，数据将存在于原来的follower中。在重新选举以后，新的leader会持有该部分数据。数据从leader同步到follower，需要2步：

  - 数据从pageCache被刷盘到disk。因为只有disk中的数据才能被同步到replica。

  - 数据同步到replica，并且replica成功将数据写入PageCache。在producer得到ack后，哪怕是所有机器都停电，数据也至少会存在于leader的磁盘内。

那么在这上面提到一个isr的概念，可以想象以下，如果leader接收到消息之后，一直等待所有的followe返回ack确认，但是有一个发生网络问题，始终无法返回ack怎么版？

显然这种情况不是我们希望的，所以就产生了isr，这个isr就是和leader同步数据的最小子集和，只要在isr中的follower，那么leader必须等待同步完消息并且返回ack才可以，否则就不反悔ack。isr的个数通常通过``min.insync.replicas`参数配置。

借用网上的一张图，感觉说的很明白：

![1639191919444](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/145111-262605.png)

0,1,-1性能一次递减，但是可靠性一直在提高。

#### Producer

Producer丢失消息，发生在**生产者客户端**。

为了提升效率，减少IO，producer在发送数据时可以将多个请求进行合并后发送。被合并的请求先缓存在本地buffer中。缓存的方式和前文提到的刷盘类似，**producer可以将请求打包成“块”或者按照时间间隔**，将buffer中的数据发出。**通过buffer我们可以将生产者改造为异步的方式，而这可以提升我们的发送效率。**

但是，buffer中的数据就是危险的。在正常情况下，客户端的异步调用可以通过callback来处理消息发送失败或者超时的情况，但是，一旦producer被非法的停止了，那么buffer中的数据将丢失，broker将无法收到该部分数据。又或者，当Producer客户端内存不够时，如果采取的策略是丢弃消息(另一种策略是block阻塞)，消息也会被丢失。抑或，消息产生(异步产生)过快，导致挂起线程过多，内存不足，导致程序崩溃，消息丢失。

![1639192224822](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/153515-998847.png)

![1639192247686](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/111054-717824.png)

根据上图，可以想到几个解决的思路：

- 异步发送消息改为同步发送。或者service产生消息时，使用阻塞的线程池，并且线程数有一定上限。整体思路是控制消息产生速度。

- 扩大Buffer的容量配置。这种方式可以缓解该情况的出现，但不能杜绝。

- service不直接将消息发送到buffer(内存)，而是将消息写到本地的磁盘中(数据库或者文件)，由另一个(或少量)生产线程进行消息发送。相当于是在buffer和service之间又加了一层空间更加富裕的缓冲层

#### Consumer消费消息有下面几个步骤：

- 接收消息

- 处理消息

- 反馈“处理完毕”(commited)

Consumer的消费方式主要分为两种：

- 自动提交offset，Automatic Offset Committing

- 手动提交offset，Manual Offset Control

Consumer自动提交的机制是根据一定的**时间间隔**，将收到的消息进行commit。**commit过程和消费消息的过程是异步的**。也就是说，可能存在消费过程未成功(比如抛出异常)，commit消息已经提交了。此时消息就丢失了。

~~~ java

Properties props = new **Properties**();

props.**put**("bootstrap.servers", "localhost:9092");

props.**put**("group.id", "test");*// 自动提交开关props.put("enable.auto.commit", "true");// 自动提交的时间间隔，此处是1sprops.put("auto.commit.interval.ms", "1000");props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");KafkaConsumer consumer = new KafkaConsumer<>(props);consumer.subscribe(Arrays.asList("foo", "bar"));while (true) {// 调用poll后，1000ms后，消息状态会被改为 committed ConsumerRecords records = consumer.poll(100);for (ConsumerRecord record : records)  insertIntoDB(record); // 将消息入库，时间可能会超过1000ms}*

~~~

上面的示例是自动提交的例子。如果此时，`insertIntoDB(record)`发生异常，消息将会出现丢失。接下来是手动提交的例子：

~~~ java

Properties props = new **Properties**();

props.**put**("bootstrap.servers", "localhost:9092");

props.**put**("group.id", "test");*// 关闭自动提交，改为手动提交props.put("enable.auto.commit", "false");props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");KafkaConsumer consumer = new KafkaConsumer<>(props);consumer.subscribe(Arrays.asList("foo", "bar"));final int minBatchSize = 200;List> buffer = new ArrayList<>();while (true) {// 调用poll后，不会进行auto commit ConsumerRecords records = consumer.poll(100);for (ConsumerRecord record : records) {  buffer.add(record); }if (buffer.size() >= minBatchSize) {  insertIntoDb(buffer);// 所有消息消费完毕以后，才进行commit操作  consumer.commitSync();  buffer.clear(); }}*

~~~

将提交类型改为手动以后，可以保证消息“至少被消费一次”(at least once)。但此时可能出现重复消费的情况。也就是数据处理完成之后，手动进行提交的方式。

### 为什么kafka中1个partition只能被同组的一个consumer消费?

Kafka通过消费者组机制同时实现了发布/订阅模型和点对点模型。**多个组的消费者消费同一个分区属于多订阅者的模式**，自然没有什么问题；

**而在单个组内某分区只交由一个消费者处理的做法则属于点对点模式**。其实这就是设计上的一种取舍，如果Kafka真的允许组内多个消费者消费同一个分区，也不是什么灾难性的事情，只是没什么意义，而且还会重复消费消息。

通常情况下，我们还是希望一个组内所有消费者能够分担负载，让彼此做的事情没有交集，做一些重复性的劳动纯属浪费资源。就如同电话客服系统，每个客户来电只由一位客服人员响应。那么请问我就是想让多个人同时接可不可以？当然也可以了，我不觉得技术上有什么困难，只是这么做没有任何意义罢了，既拉低了整体的处理能力，也造成了人力成本的浪费。

还由另外一点，如果让一个消费者组中的多个消费者消费同一个分区数据，那么我们保证多个消费者之间顺序的去消费数据的话，这里就产生了线程安全的问题，导致系统的设计更加的复杂。

总之，我的看法是这种设计不是出于技术上的考量而更多还是看效率等非技术方面。

### zookeeper在kafka中的作用

`Kafka`集群中有一个 ` broker`会被选举为 `  Controller`，负责管理集群` broker`的上下线，所有` topic`的分区副本分配和`  leader`选举等工作。

`Controller`的管理工作都是依赖于 ` Zookeeper`的。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/24/125143-749577.png)

Apache Kafka是一个使用Zookeeper构建的分布式系统。Zookeeper的主要作用是在集群中的不同节点之间建立协调；如果任何节点失败，我们还使用Zookeeper从先前提交的偏移量中恢复，因为它做周期性提交偏移量工作。

**说明**

- 从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合
- 调用配置的分区选择算法选择分区的leader

#### 作用

##### Broker注册

Broker是分布式部署并且相互之间相互独立，**但是需要有一个注册系统能够将整个集群中的Broker管理起来**，此时就使用到了Zookeeper。在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点。

##### Topic注册

在Kafka中，同一个Topic的消息会被分成多个分区并将其分布在多个Broker上，这些分区信息及与Broker的对应关系也都是由Zookeeper在维护，由专门的节点来记录。

##### 生产者负载均衡

由于同一个Topic消息会被分区并将其分布在多个Broker上，因此，生产者需要将消息合理地发送到这些分布式的Broker上，那么如何实现生产者的负载均衡，Kafka支持传统的四层负载均衡，也支持Zookeeper方式实现负载均衡。

##### 消费者负载均衡

与生产者类似，Kafka中的消费者同样需要进行负载均衡来实现多个消费者合理地从对应的Broker服务器上接收消息，每个消费者分组包含若干消费者，每条消息都只会发送给分组中的一个消费者，不同的消费者分组消费自己特定的Topic下面的消息，互不干扰。

##### 分区与消费者的关系

消费组（Consumer Group）：consumer group下有多个Consumer（消费者），对于每个消费者组 （Consumer Group），Kafka都会为其分配一个全局唯一的Group ID，Group 内部的所有消费者共享该 ID。 订阅的topic下的每个分区只能分配给某个 group 下的一个consumer（当然该分区还可以被分配给其他group）。 同时，Kafka为每个消费者分配一个Consumer ID，通常采用”Hostname:UUID”形式表示。

在Kafka中，规定了每个消息分区只能被同组的一个消费者进行消费，因此，需要在Zookeeper上记录 消息分区 与 Consumer之间的关系，每个消费者一旦确定了对一个消息分区的消费权力，需要将其Consumer ID 写入到 Zookeeper 对应消息分区的临时节点上。

##### 消费进度Offset记录

在消费者对指定消息分区进行消息消费的过程中，需要定时地将分区消息的消费进度Offset记录到Zookeeper上，以便在该消费者进行重启或者其他消费者重新接管该消息分区的消息消费后，能够从之前的进度开始继续进行消息消费。Offset在Zookeeper中由一个专门节点进行记录。节点内容是Offset的值。

##### 消费者注册

每个消费者服务器启动时，都会到Zookeeper的指定节点下创建一个属于自己的消费者节点。

早期版本的Kafka用zk做meta信息存储，consumer的消费状态，group的管理以及offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题，新版本中确实逐渐弱化了zookeeper的作用。新的consumer使用了kafka内部的group coordination协议，也减少了对zookeeper的依赖。

Zookeeper是一个开放源码的、高性能的协调服务，它用于Kafka的分布式应用，kafka不可能越过Zookeeper直接联系Kafka broker，一旦Zookeeper停止工作，它就不能服务客户
端请求。

Zookeeper主要用于在集群中不同节点之间进行通信，在Kafka中，它被用于提交偏移量，因此如果节点在任何情况下都失败了，它都可以从之前提交的偏移量中获取，除此之外，它还执行其他活动,如:

leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等

### Kafka服务器能接收到的最大信息是多少？

Kafka服务器可以接收到的消息的最大大小是1000000字节

### Kafka中的ZooKeeper是什么？Kafka是否可以脱离ZooKeeper独立运行？

本篇针对的是2.8版本之前的Kafka,2.8版本及之后Kafka已经移除了对Zookeeper的依赖，通过KRaft进行自己的集群管理,不过目前只是测试阶段。



















一个消费者组中只有一个消费者可以消费分区数据，这样所还可以保证线程的安全性，如果由多个写哦飞着可以消费一个分区中的数据，那么如和保证多个线程之间顺序的消费这一个分区中的数据，可能还需要添加锁机制，所以提高了系统的复杂度。