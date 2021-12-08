## Kafka篇

### 请说明什么是Apache Kafka？

Kafka 是由 `Linkedin` 公司开发的，它是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台，它同时也是一款开源的**基于发布订阅模式的消息引擎系统**。

### Kafka的基本术语

消息：Kafka 中的数据单元被称为`消息`，也被称为记录，**可以把它看作数据库表中某一行的记录**。

批次：为了提高效率， 消息会`分批次`写入 Kafka，批次就代指的是一组消息。

主题：消息的种类称为 `主题`（Topic）,可以说一个主题代表了一类消息。相当于是对消息进行分类。**主题就像是数据库中的表。**

分区：主题可以被分为若干个分区（partition），同一个主题中的分区可以不在一个机器上，有可能会部署在多个机器上，由此来实现 kafka 的`伸缩性`，单一主题中的分区有序，但是无法保证主题中所有的分区有序，把分区理解为一张表，二分区表就是主题的分区。

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

### Kafka消息队列

Kafka 的消息队列一般分为两种模式：**点对点模式和发布订阅模式**

Kafka 是支持消费者群组的，也就是说 Kafka 中会有一个或者多个消费者，如果一个生产者生产的消息由一个消费者进行消费的话，那么这种模式就是点对点模式

点对点模型通常是一个**基于拉取或者轮询的消息传送模型**，这种模型从队列中请求信息，而不是将消息推送到客户端。**这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。**

![1634970879317](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/143442-369189.png)

- 消费者去队列中拉去消息，即使有多个消费者，那生产者也会把消息发送到队列中，多个消费者轮询到队列中拉去消息。

发布订阅模式有两种形式，**一种是消费者主动拉取数据的形式，另一种是生产者推送消息的形式**，kafka是基于发布订阅模式中**消费者拉取**的方式。消费者的消费速度可以由消费者自己决定。但是这种方式也有缺点，当没有消息的时候，kafka的消费者还需要不停的访问kafka生产者拉取消息，浪费资源。

![1634971018569](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/143718-760603.png)

- 这种方式中，消费者和生产者都可以有多个。

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

### 请说明Kafka相对于传统的消息传递方法有什么优势？

1. 高性能：单一的Kafka代理可以处理成千上万的客户端，每秒处理数兆字节的读写操作，Kafka性能远超过传统的ActiveMQ、RabbitMQ等，而且Kafka支持Batch操作
2. 可扩展：Kafka集群可以透明的扩展，增加新的服务器进集群
3. 容错性： Kafka每个Partition数据会复制到几台服务器，当某个Broker失效时，Zookeeper将通知生产
   者和消费者从而使用其他的Broker。
4. 高吞吐、低延迟：kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒。
5. 高伸缩性： 每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。
6. 持久性、可靠性： Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失，Kafka 底层的数据存储是基于 Zookeeper 存储的，Zookeeper 我们知道它的数据能够持久存储。

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
### Kafka的工作机制

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/182809-557369.png)

上图表示`kafka`集群有一个`topic A`,并且有三个分区，分布在三个节点上面，注意点：每个分区有两个副本，两个副本分别是`leader,follower`,并且每一个副本一定不和自己的`leader`分布在一个节点上面。`Kafka`中消息是以  **`topic`**进行分类的，生产者生产消息，消费者消费消息，都是面向  `topic`的。`topic`是逻辑上的概念，而`partition`是物理上的概念，每个` partition`对应于一个` log`文件，该` log`文件中存储的就是  `producer`生产的数据。`Producer`生产的数据会被不断追加到该`log`文件末端，且每条数据都有自己的  `offset`。消费者组中的每个消费者，都会实时记录自己消费到了哪个 `offset`，以便出错恢复时，从上次的位置继续消费。每一个分区内部的数据是有序的，但是全局不是有序的。

**kafka文件存储结构**

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/145145-384586.png)

由于生产者生产的消息会不断追加到` log`文件末尾，为防止`  log`文件过大导致数据定位效率低下，`Kafka`采取了**分片和索引机制**，将每个  `partition`分为多个 ` segment`。每个  `segment`对应两个文件`“.index”`文件和`“.log”`文件。这些文件位于一个文件夹下，该文件夹的命名规则为：`topic名称+分区序号`。例如，`first`这个  ` topic`有三个分区，则其对应的文件夹为  :

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

上面`kafka`再查找偏移量的时候是以**二分查找法进行查找的**。也就是查询index的时候使用的是二分查找法，查找原理是：文件头的偏移量和文件大小快速定位。`“.index”`文件存储大量的索引信息，在查找`index`的时候使用的是二分查找法，`“.log”`文件存储大量的数据，索引文件中的元数据指向对应数据文件中 `message`的物理偏移地址。

### Kafka工作流程

1. 消息分类按不同类别,分成不同的Topic,Topic⼜又拆分成多个partition,每个partition均衡分散到不同的服务器器(提高并发访问的能力)
2. 消费者按顺序从partition中读取,不支持随机读取数据,但可通过改变保存到zookeeper中的offset位置实现从任意位置开始读取。
3. 服务器消息定时清除(不管有没有消费)
4. 每个partition还可以设置备份到其他服务器上的个数以保证数据的可⽤性。通过Leader,Follower方式。
5. zookeeper保存kafka服务器和客户端的所有状态信息.(确保实际的客户端和服务器轻量级)
6. 在kafka中,一个partition中的消息只会被group中的一个consumer消费;每个group中consumer消息消费互相独立;我们可以认为一个group是一个"订阅"者,⼀个Topic中的每个partions,只会被一个"订阅者"中的一个consumer消费,不过⼀个consumer可以消费多个partitions中的消息。
7. 如果所有的consumer都具有相同的group（也就是所有的消费者再同一个消费者组）,这种情况和queue模式很像;消息将会在consumers之间负载均衡.
8. 如果所有的consumer都具有不同的group,那这就是"发布-订阅";消息将会广播给所有的消费者.
9. 持久性,当收到的消息时先buffer起来,等到了一定的阀值再写入磁盘文件,减少磁盘IO.在一定程度上依赖OS的文件系统(对文件系统本身优化几乎不可能)。
10. 除了磁盘IO,还应考虑网络IO，批量对消息发送和接收,并对消息进行压缩。
11. 在JMS实现中,Topic模型基于push方式,即broker将消息推送给consumer端.不过在kafka中,采用了pull方式,即consumer在和broker建立连接之后,主动去pull(或者说fetch)消息;这种模式有些优点,首先consumer端可以根据⾃己的消费能力适时的去fetch消息并处理,且可以控制消息消费的进度(offset);此外,消费者可以良好的控制消息消费的数量,batch fetch.
12. kafka无需记录消息是否接收成功,是否要重新发送等,所以kafka的producer是非常轻量级的,consumer端也只需要将fetch后的offset位置注册到zookeeper,所以也是非常轻量级的.

**kafka应用场景**

对于一些常规的消息系统,kafka是个不错的选择;partitons/replication和容错,可以使kafka具有良好的扩展性和性能优势，

kafka只能使用作为"常规"的消息系统,在一定程度上,尚未确保消息的发送与接收绝对可靠(⽐如,消息重发,消息发送丢失等)

kafka的特性决定它非常适合作为"日志收集中心";application可以将操作日志"批量""异步"的发送到kafka集群中,而不是保存在本地或者DB中;kafka可以批量提交消息/压缩消息等,这对producer端而言,几乎感觉不到性能的开支.

### kafka数据分区和消费者的关系，kafka的数据offset读取流程，kafka内部如何保证顺序，结合外部组件如何保证消费者的顺序？

1. kafka数据分区和消费者的关系：1个partition只能被同组的⼀个consumer消费，同组的consumer则起到均衡效果
2. kafka的数据offset读取流程
   1. 连接ZK集群，从ZK中拿到对应topic的partition信息和partition
      的Leader的相关信息



### kafka生产者写入数据

**副本**

同一个`partition`可能会有多个`replication`（对应` server.properties `配置中default.replication.factor=N）没有`replication`的情况下，一旦`broker` 宕机，其上所有` patition` 的数据都不可被消费，同时producer也不能再将数据存于其上的`patition`。引入`replication`之后，同一个`partition`可能会有多个`replication`，而这时需要在这些`replication`之间选出一个`leader`，`producer`和`consumer`只与这个`leader`交互，其它`replication`的`follower`从leader 中复制数据,保证数据的一致性。

**写入方式**

`producer`采用推`（push）`模式将消息发布到`broker`，每条消息都被追加`（append）`到分区`（patition）`中，属于**顺序写磁盘**（顺序写磁盘效率比随机写内存要高，保障`kafka`吞吐率）。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090838-364327.png)

1. producer先从`zookeeper`的 `"/brokers/.../state"`节点找到该`partition`的`leader`
2. `producer`将消息发送给该`leader`
3. `leader`将消息写入本地`log`
4. `followers`从`leader pull`消息，写入本地`log`后向`leader`发送`ACK`
5. `leader`收到所有`ISR`中的`replication`的`ACK`后，增加`HW（high watermark，最后commit 的offset）`并向`producer`发送`ACK`

**broker保存消息**

存储方式：物理上把`topic`分成一个或多个`patition`，每个`patition`物理上对应一个文件夹（该文件夹存储该`patition`的所有消息和索引文件）

**存储策略**

无论消息是否被消费，`kafka`都会保留所有消息。有两种策略可以删除旧数据：

- 基于时间：`log.retention.hours=168`
- 基于大小：`log.retention.bytes=1073741824`

需要注意的是，因为`Kafka`读取特定消息的时间复杂度为`O(1)`，即与文件大小无关，所以这里删除过期文件与提高` Kafka `性能无关

**分区**

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

**数据写入的可靠性保证。**

为保证 `producer`发送的数据，能可靠的发送到指定的  `topic`，`topic`的每个`  partition`收到

`producer`发送的数据后，都需要向`   producer`发送 `  ack（acknowledgement确认收到）`，如果

`producer`收到 ` ack`，就会进行下一轮的发送，否则重新发送数据。

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

- `Leader`维护了一个动态的 ` in-sync replica set (ISR)`，意为和 `leader`保持同步的 `follower`集合，是一个队列。当 `ISR`中的 ` follower`完成数据的同步之后，`leader`就会给 `follower`发送  `ack`。如果  `follower`长时间未向`leader`同步数据，则该`follower`将被踢出`ISR`，该时间阈值由**`replica.lag.time.max.ms`**参数设定。`Leader`发生故障之后，就会从`ISR`中选举新的`leader`。在这个时间内，就添加到isr中，否则就提出isr集合中，不在isr中的follower也不可能被选举为leader。

**`ack`**应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等` ISR`中的`follower`全部接收成功。所以` Kafka`为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。

`ACKS`参数配置：

- 0：`producer`不等待  `broker`的`ack`,这一操作提供了一个最低的延迟，`broker`一接收到还没有写入磁盘就已经返回，**当 `broker`故障时有可能丢失数据；**
- 1：`producer`等待  `broker`的`ack`，`partition`的  `leader`落盘成功后返回`ack`，如果在`follower`同步成功之前 `leader`故障，那么将会丢失数据；

**ack=1也可能丢失数据**

![1618623903699](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/094505-503238.png)

- -1（all）：`producer`等待  `broker`的`ack`，`partition`的`leader`和`follower`（这里指的是isr中的follower）全部落盘成功后才返回 `ack`。但是如果在 `follower`同步完成后，`broker`发送`ack`之前，`leader`发生故障，那么会造成数据重复。

  =-1也可能会发生数据的丢失，发生重复数据的情况是leader接收到数据并且在follower之间已经同步完成后，但是此时leader挂掉，没有返回ack确认，此时又重新选举产生了leader,那么producer会重新发送一次数据，所以会导致数据重复。

**ack=-1数据重复案例**

![1618625608045](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/101329-814647.png)

> ack是保证生产者生产的数据不丢失，hw是保证消费者消费数据的一致性问题。hw实际就是最短木桶原则，根据这个原则消费者进行消费数据。不能解决数据重复和丢失问题。ack解决丢失和重复问题。

**故障处理细节**

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/090845-425507.png)

​	**`LEO`**：指的是每个副本最大的 **`offset`**，也就是灭一个副本的最后offset值。

​	**`HW`**：指的是消费者能见到的最大的 **`offset`**，**`ISR`**队列中最小的  **`LEO`**。

1. **`follower`**故障

   `follower`发生故障后会被临时踢出 ` ISR`，待该` follower`恢复后，`follower`会读取本地磁盘记录的上次的 `HW`，并将 `log`文件高于 ` HW`的部分截取掉，从  `HW`开始向`  leader`进行同步。等该 **`follower`**的  **`LEO`**大于等于该  **`Partition`**的  **`HW`**，即` follower`追上 ` leader`之后，就可以重新加入` ISR`了。

2. **`leader`**故障

   `leader`发生故障之后，会从 `ISR`中选出一个新的  `leader`，之后，为保证多个副本之间的数据一致性，其余的 `follower`会先将各自的`  log`文件高于  `HW`的部分截掉，然后从新的 `leader`同步数据。

   **注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复,`ack`确认机制可以保证数据的不丢失和不重复，`LEO`和`hw`可以保证数据的一致性问题**

   #### **Exactly Once**语义

   将服务器的 `ACK`级别设置为`-1`，可以保证`Producer`到`Server`之间不会丢失数据，即`At Least Once`语义。相对的，将服务器`ACK`级别设置为`0`,可以保证生产者每条消息只会被发送一次，即 `At Most Once`语义。`At Least Once`可以保证数据不丢失，但是不能保证数据不重复；相对的，`At Most Once`可以保证数据不重复，但是不能保证数据不丢失。

   但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即` Exactly  Once`语义。在  `0.11`版本以前的` Kafka`，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。`0.11`版本的  `Kafka`，引入了一项重大特性：**幂等性。所谓的幂等性就是指 `Producer`不论向` Server`发送多少次重复数据，`Server`端都只会持久化一条。**

   

   幂等性结合`At  Least Once`语义，就构成了 `Kafka`的 ` Exactly Once`语义。即：`At Least Once +幂等性= Exactly Once`要启用幂等性，只需要将 `Producer`的参数中  `enable.idompotence`设置为`true`即可。`Kafka`的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的` Producer`在初始化的时候会被分配一个  `PID`，发往同一  `Partition`的消息会附带`Sequence Number`。而`Broker`端会对`<PID,  Partition, SeqNumber>`做缓存，当具有相同主键的消息提交时，`Broker`只会持久化一条。但是 `PID`重启就会变化，同时不同的 ` Partition`也具有不同主键，所以幂等性无法保证跨分区跨会话（也就是重新建立producer链接的情况）的 `Exactly Once`。即只能保证单次会话不重复问题。幂等性只能解决但回话单分区的问题。

### kafka消费者

**消费方式**

- `consumer`采用 ` pull`（拉）模式从 `broker`中读取数据。
- `push`（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 `consumer`来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而` pull`模式则可以根据  `consumer`的消费能力以适当的速率消费消息。对于`Kafka`而言，`pull`模式更合适，它可简化`broker`的设计，`consumer`可自主控制消费消息的速率，同时`consumer`可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义
- `pull`模式不足之处是，如果 ` kafka`没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，`Kafka`的消费者在消费数据时会传入一个时长参数 ` timeout`，如果当前没有数据可供消费，`consumer`会等待一段时间之后再返回，这段时长即为  `timeout`。

**分区分配策略**

一个` consumer group`中有多个`consumer`，一个  `topic`有多个  `partition`，所以必然会涉及

到 `partition`的分配问题，即确定那个`partition`由哪个`consumer`来消费。`Kafka`有两种分配策略，一是  `RoundRobin`（按照组分配），一是 `Range`（按照主题分配）。同一个消费者组中的不同消费者，不能同时消费同一个分区，但是可以同时消费一个主题，因为一个主题中有多个分区。

按照消费者组中的消费者进行轮询：

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

### zookeeper在kafka中的作用

`Kafka`集群中有一个 ` broker`会被选举为 `  Controller`，负责管理集群` broker`的上下线，所有` topic`的分区副本分配和`  leader`选举等工作。

`Controller`的管理工作都是依赖于 ` Zookeeper`的。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/24/125143-749577.png)



### Kafka服务器能接收到的最大信息是多少？

Kafka服务器可以接收到的消息的最大大小是1000000字节

### Kafka中的ZooKeeper是什么？Kafka是否可以脱离ZooKeeper独立运行？

本篇针对的是2.8版本之前的Kafka,2.8版本及之后Kafka已经移除了对Zookeeper的依赖，通过KRaft进行自己的集群管理,不过目前只是测试阶段。

#### 一、概述

Apache Kafka是一个使用Zookeeper构建的分布式系统。Zookeeper的主要作用是在集群中的不同节点之间建立协调；如果任何节点失败，我们还使用Zookeeper从先前提交的偏移量中恢复，因为它做周期性提交偏移量工作。

**partition的leader选举过程：**

![1635053790695](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/24/133631-899094.png)

**说明**

- 从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合
- 调用配置的分区选择算法选择分区的leader

#### 二、作用

##### 2.1 Broker注册

Broker是分布式部署并且相互之间相互独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来，此时就使用到了Zookeeper。在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点。

##### 2.2 Topic注册

在Kafka中，同一个Topic的消息会被分成多个分区并将其分布在多个Broker上，这些分区信息及与Broker的对应关系也都是由Zookeeper在维护，由专门的节点来记录。

##### 2.3 生产者负载均衡

由于同一个Topic消息会被分区并将其分布在多个Broker上，因此，生产者需要将消息合理地发送到这些分布式的Broker上，那么如何实现生产者的负载均衡，Kafka支持传统的四层负载均衡，也支持Zookeeper方式实现负载均衡。

##### 2.4 消费者负载均衡

与生产者类似，Kafka中的消费者同样需要进行负载均衡来实现多个消费者合理地从对应的Broker服务器上接收消息，每个消费者分组包含若干消费者，每条消息都只会发送给分组中的一个消费者，不同的消费者分组消费自己特定的Topic下面的消息，互不干扰。

##### 2.5 分区与消费者的关系

消费组（Consumer Group）：consumer group下有多个Consumer（消费者），对于每个消费者组 （Consumer Group），Kafka都会为其分配一个全局唯一的Group ID，Group 内部的所有消费者共享该 ID。 订阅的topic下的每个分区只能分配给某个 group 下的一个consumer（当然该分区还可以被分配给其他group）。 同时，Kafka为每个消费者分配一个Consumer ID，通常采用”Hostname:UUID”形式表示。

在Kafka中，规定了每个消息分区只能被同组的一个消费者进行消费，因此，需要在Zookeeper上记录 消息分区 与 Consumer之间的关系，每个消费者一旦确定了对一个消息分区的消费权力，需要将其Consumer ID 写入到 Zookeeper 对应消息分区的临时节点上。

##### 2.6 消费进度Offset记录

在消费者对指定消息分区进行消息消费的过程中，需要定时地将分区消息的消费进度Offset记录到Zookeeper上，以便在该消费者进行重启或者其他消费者重新接管该消息分区的消息消费后，能够从之前的进度开始继续进行消息消费。Offset在Zookeeper中由一个专门节点进行记录。节点内容是Offset的值。

##### 2.7 消费者注册

每个消费者服务器启动时，都会到Zookeeper的指定节点下创建一个属于自己的消费者节点。

早期版本的Kafka用zk做meta信息存储，consumer的消费状态，group的管理以及offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题，新版本中确实逐渐弱化了zookeeper的作用。新的consumer使用了kafka内部的group coordination协议，也减少了对zookeeper的依赖。

Zookeeper是一个开放源码的、高性能的协调服务，它用于Kafka的分布式应用

不可以，不可能越过Zookeeper直接联系Kafka broker，一旦Zookeeper停止工作，它就不能服务客户
端请求

Zookeeper主要用于在集群中不同节点之间进行通信，在Kafka中，它被用于提交偏移量，因此如果节
点在任何情况下都失败了，它都可以从之前提交的偏移量中获取，除此之外，它还执行其他活动,如:
leader检测、分布式同步、配置管理、识别新节点何时离开或连接、集群、节点实时状态等等

### 解释一下，在数据制作过程中，你如何能从Kafka得到准确的信息？

在数据中，为了精确地获得Kafka的消息，你必须遵循两件事：**在数据消耗期间避免重复，在数据生产**
**过程中避免重复。**

这里有两种方法，可以在数据生成时准确地获得一个语义:

1. 每个分区使用一个单独的写入器，每当你发现一个网络错误，检查该分区中的最后一条消息，以查看您的最后一次写入是否成功
2. 在消息中包含一个主键(UUID或其他)，并在用户中进行反复制

### 解释如何减少ISR中的扰动？broker什么时候离开ISR？

ISR是一组与leaders完全同步的消息副本，也就是说ISR中包含了所有提交的消息。ISR应该总是包含所
有的副本，直到出现真正的故障。如果一个副本从leader中脱离出来，将会从ISR中删除

### Kafka为什么需要复制？

Kafka的信息复制确保了任何已发布的消息不会丢失，并且可以在机器错误、程序错误或更常见些的软
件升级中使用。

### 请说明Kafka 的消息投递保证（delivery guarantee）机制以及如何实现？

Kafka支持三种消息投递语义：

1. At most once：消息可能会丢，但绝不会重复传递
2. At least one：消息绝不会丢，但可能会重复传递
3. Exactly once：每条消息肯定会被传输一次且仅传输一次，很多时候这是用户想要的

consumer在从broker读取消息后，可以选择commit，该操作会在Zookeeper中存下该consumer在该
partition下读取的消息的offset，该consumer下一次再读该partition时会从下一条开始读取。如未
commit，下一次读取的开始位置会跟上一次commit之后的开始位置相同

可以将consumer设置为autocommit，即consumer一旦读到数据立即自动commit。如果只讨论这一
读取消息的过程，那Kafka是确保了Exactly once。但实际上实际使用中consumer并非读取完数据就结
束了，而是要进行进一步处理，而数据处理与commit的顺序在很大程度上决定了消息从broker和
consumer的delivery guarantee semantic

读完消息先commit再处理消息。这种模式下，如果consumer在commit后还没来得及处理消息就crash
了，下次重新开始工作后就无法读到刚刚已提交而未处理的消息，这就对应于At most once

读完消息先处理再commit消费状态(保存offset)。这种模式下，如果在处理完消息之后commit之前
Consumer crash了，下次重新开始工作时还会处理刚刚未commit的消息，实际上该消息已经被处理过
了，这就对应于At least once

如果一定要做到Exactly once，就需要协调offset和实际操作的输出。经典的做法是引入两阶段提交，
但由于许多**输出系统不支持两阶段提交**，更为通用的方式是将offset和操作输入存在同一个地方。比
如，consumer拿到数据后可能把数据放到HDFS，如果把最新的offset和数据本身一起写到HDFS，那
就可以保证数据的输出和offset的更新要么都完成，要么都不完成，间接实现Exactly once。（目前就
high level API而言，offset是存于Zookeeper中的，无法存于HDFS，而low level API的offset是由自己
去维护的，可以将之存于HDFS中）

总之，Kafka默认保证At least once，并且允许通过设置producer异步提交来实现At most once，而
Exactly once要求与目标存储系统协作，Kafka提供的offset可以较为容易地实现这种方式

### 如何保证Kafka的消息有序

Kafka对于消息的重复、丢失、错误以及顺序没有严格的要求

Kafka只能保证一个partition中的消息被某个consumer消费时是顺序的，事实上，从Topic角度来说，
当有多个partition时，消息仍然不是全局有序的

### kafka数据丢失问题,及如何保证？

**数据丢失：**

- acks=1的时候(只保证写入leader成功)，如果刚好leader挂了。数据会丢失
- acks=0的时候，使用异步模式的时候，该模式下kafka无法保证消息，有可能会丢

**brocker如何保证不丢失：**

- acks=all（-1） : 所有副本都写入成功并确认
- retries = 一个合理值
- min.insync.replicas=2 消息至少要被写入到这么多副本才算成功
- unclean.leader.election.enable=false 关闭unclean leader选举，即不允许非ISR中的副本被选举为
  leader，以避免数据丢失

**Consumer如何保证不丢失：**

- 如果在消息处理完成前就提交了offset，那么就有可能造成数据的丢失
- enabel.auto.commit=false关闭自动提交offset
- 处理完数据之后手动提交

### kafka的balance是怎么做的？

生产者将数据发布到他们选择的主题。生产者可以选择在主题中分配哪个分区的消息。这可以通过循环
的方式来完成，只是为了平衡负载，或者可以根据一些语义分区功能（比如消息中的一些键）来完成。
更多关于分区在一秒钟内的使用

### kafka的消费者方式？

consumer采用pull（拉）模式从broker中读取数据

push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是
尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服
务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息

对于Kafka而言，pull模式更合适，它可简化broker的设计，consumer可自主控制消费消息的速率，同
时consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从
而实现不同的传输语义

pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直等待数据到达。为了避免这
种情况，我们在我们的拉请求中有参数，允许消费者请求在等待数据到达的“长轮询”中进行阻塞

### Kafka 都有哪些特点？

**高吞吐量、低延迟**：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多
个partition, consumer group 对partition进行consume操作

**可扩展性**：kafka集群支持热扩展

**持久性、可靠性**：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失

**容错性**：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败） 高并发：支持数千个客户端同
时读写

### 请简述下你在哪些场景下会选择 Kafka？

日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各
种consumer，例如hadoop、HBase、Solr等

消息系统：解耦和生产者和消费者、缓存消息等

用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等
活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监
控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘

运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集
中反馈，比如报警和报告

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
4. Consumer Group：这是 kafka 用来实现一个 topic 消息的广播（发给所有的 consumer）和单播（发
   给任意一个 consumer）的手段。一个 topic 可以有多个 Consumer Group
5. Broker：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多
   个 topic
6. Partition：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker上，每个 partition 是一个有
   序的队列。partition 中的每条消息都会被分配一个有序的id（offset）。将消息发给 consumer，kafka
7. 只保证按一个 partition 中的消息的顺序，不保证一个 topic 的整体（多个 partition 间）的顺序
8. Offset：kafka 的存储文件都是按照 offset.kafka 来命名，用 offset 做名字的好处是方便查找。例如你
   想找位于 2049 的位置，只要找到 2048.kafka 的文件即可。当然 the first offset 就是
   00000000000.kafka。

### Kafka 分区的目的？

分区对于 Kafka 集群的好处是：实现负载均衡。分区对于消费者来说，可以提高并发度，提高效率

### Kafka 是如何做到消息的有序性？

kafka 中的每个 partition 中的消息在写入时都是有序的，而且单独一个 partition 只能由一个消费者去
消费，可以在里面保证消息的顺序性。但是分区之间的消息是不保证有序的

### Kafka 的高可靠性是怎么实现的？

注意：也可回答“Kafka在什么情况下会出现消息丢失？”数据可靠性（可回答“怎么尽可能保证Kafka的可靠性？”）

Kafka 作为一个商业级消息中间件，消息可靠性的重要性可想而知。本文从 Producter 往 Broker 发送
消息、Topic 分区副本以及 Leader 选举几个角度介绍数据的可靠性。

**Topic分区副本**

在 Kafka 0.8.0 之前，Kafka 是没有副本的概念的，那时候人们只会用 Kafka 存储一些不重要的数据，
因为没有副本，数据很可能会丢失。但是随着业务的发展，支持副本的功能越来越强烈，所以为了保证
数据的可靠性，Kafka 从 0.8.0 版本开始引入了分区副本（详情请参见 KAFKA-50）。也就是说每个分
区可以人为的配置几个副本（比如创建主题的时候指定 replication-factor，也可以在 Broker 级别进行
配置 default.replication.factor），一般会设置为3

Kafka 可以保证单个分区里的事件是有序的，分区可以在线（可用），也可以离线（不可用）。在众多
的分区副本里面有一个副本是 Leader，其余的副本是 follower，所有的读写操作都是经过 Leader 进行
的，同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候，其中一个 follower 会重
新成为新的 Leader。通过分区副本，引入了数据冗余，同时也提供了 Kafka 的数据可靠性。

Kafka 的分区多副本架构是 Kafka 可靠性保证的核心，把消息写入多个副本可以使 Kafka 在发生崩溃
时仍能保证消息的持久性

**Producer 往 Broker 发送消息**

如果我们要往 Kafka 对应的主题发送消息，我们需要通过 Producer 完成。前面我们讲过 Kafka 主题对
应了多个分区，每个分区下面又对应了多个副本；为了让用户设置数据可靠性， Kafka 在 Producer 里
面提供了消息确认机制。也就是说我们可以通过配置来决定消息发送到对应分区的几个副本才算消息发
送成功。可以在定义 Producer 时通过 acks 参数指定（在 0.8.2.X 版本之前是通过request.required.acks 参数设置的）

这个参数支持以下三种值：

acks = 0：意味着如果生产者能够通过网络把消息发送出去，那么就认为消息已成功写入Kafka。在这
种情况下还是有可能发生错误，比如发送的对象无能被序列化或者网卡发生故障，但如果是分区离线或
整个集群长时间不可用，那就不会收到任何错误。在 acks=0+ 模式下的运行速度是非常快的（这就是为
什么很多基准测试都是基于这个模式），你可以得到惊人的吞吐量和带宽利用率，不过如果选择了这种
模式， 一定会丢失一些消息

acks = 1：意味若 Leader 在收到消息并把它写入到分区数据文件（不一定同步到磁盘上）时会返回确
认或错误响应。在这个模式下，如果发生正常的 Leader 选举，生产者会在选举时收到一个
LeaderNotAvailableException 异常，如果生产者能恰当地处理这个错误，它会重试发送悄息，最终消
息会安全到达新的 Leader 那里。不过在这个模式下仍然有可能丢失数据，比如消息已经成功写入
Leader，但在消息被复制到 follower 副本之前 Leader发生崩溃

acks = all（这个和 request.required.acks = -1 含义一样）：意味着 Leader 在返回确认或错误响应之
前，会等待所有同步副本都收到悄息。如果和 min.insync.replicas 参数结合起来，就可以决定在返回
确认前至少有多少个副本能够收到悄息，生产者会一直重试直到消息被成功提交。不过这也是最慢的做
法，因为生产者在继续发送其他消息之前需要等待所有副本都收到当前的消息
根据实际的应用场景，我们设置不同的 acks，以此保证数据的可靠性

**Leader 选举**
在介绍 Leader 选举之前，让我们先来了解一下 ISR（in-sync replicas）列表。每个分区的 leader 会维
护一个 ISR 列表，ISR 列表里面就是 follower 副本的 Borker 编号，只有跟得上 Leader 的 follower 副
本才能加入到 ISR 里面，这个是通过 replica.lag.time.max.ms 参数配置的。只有 ISR 里的成员才有被
选为 leader 的可能。

**数据一致性（可回答“Kafka数据一致性原理？”）**

这里介绍的数据一致性主要是说不论是老的 Leader 还是新选举的 Leader，Consumer 都能读到一样
的数据。那么 Kafka 是如何实现的呢？

![1634968962472](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/23/140242-153085.png)

假设分区的副本为3，其中副本0是 Leader，副本1和副本2是 follower，并且在 ISR 列表里面。虽然副
本0已经写入了 Message4，但是 Consumer 只能读取到 Message2。因为所有的 ISR 都同步了
Message2，只有 High Water Mark 以上的消息才支持 Consumer 读取，而 High Water Mark 取决于
ISR 列表里面偏移量最小的分区，对应于上图的副本2，这个很类似于木桶原理

这样做的原因是还没有被足够多副本复制的消息被认为是“不安全”的，如果 Leader 发生崩溃，另一个
副本成为新 Leader，那么这些消息很可能丢失了。如果我们允许消费者读取这些消息，可能就会破坏
一致性。试想，一个消费者从当前 Leader（副本0） 读取并处理了 Message4，这个时候 Leader 挂掉
了，选举了副本1为新的 Leader，这时候另一个消费者再去从新的 Leader 读取消息，发现这个消息其
实并不存在，这就导致了数据不一致性问题

当然，引入了 High Water Mark 机制，会导致 Broker 间的消息复制因为某些原因变慢，那么消息到达
消费者的时间也会随之变长（因为我们会先等待消息复制完毕）。延迟时间可以通过参数
replica.lag.time.max.ms 参数配置，它指定了副本在复制消息时可被允许的最大延迟时间

### ISR、OSR、AR 是什么？

- ISR：In-Sync Replicas 副本同步队列
- OSR：Out-of-Sync Replicas
- AR：Assigned Replicas 所有副本

ISR是由leader维护，follower从leader同步数据有一些延迟（具体可以参见 图文了解 Kafka 的副本复
制机制），超过相应的阈值会把 follower 剔除出 ISR, 存入OSR（Out-of-Sync Replicas ）列表，新加入
的follower也会先存放在OSR中。AR=ISR+OSR

### LEO、HW、LSO、LW等分别代表什么？

LEO：是 LogEndOffset 的简称，代表当前日志文件中下一条

HW：水位或水印（watermark）一词，也可称为高水位(high watermark)，通常被用在流式处理领域
（比如Apache Flink、Apache Spark等），以表征元素或事件在基于时间层面上的进度。在Kafka中，
水位的概念反而与时间无关，而是与位置信息相关。严格来说，它表示的就是位置信息，即位移
（offset）。取 partition 对应的 ISR中 最小的 LEO 作为 HW，consumer 最多只能消费到 HW 所在的
位置上一条信息

LSO：是 LastStableOffset 的简称，对未完成的事务而言，LSO 的值等于事务中第一条消息的位置
(firstUnstableOffset)，对已完成的事务而言，它的值同 HW 相同

LW：Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值

### 数据传输的事务有几种？

数据传输的事务定义通常有以下三种级别：

- 最多一次（At most once）：消息不会被重复发送，最多被传输一次，但也有可能一次不传输
- 最少一次（At least one）：消息不会被漏发送，最少被传输一次，但也有可能被重复传输

- 精确的一次（Exactly once）：不会漏传输也不会重复传输，每个消息都传输被接收

### Kafka 消费者是否可以消费指定分区消息？

Kafka consumer消费消息时，向broker发出fetch请求去消费特定分区的消息，consumer指定消息在
日志中的偏移量（offset），就可以消费从这个位置开始的消息，customer拥有了offset的控制权，可
以向后回滚去重新消费之前的消息，这是很有意义的

### Kafka消息是采用Pull模式，还是Push模式？

Kafka最初考虑的问题是，customer应该从brokes拉取消息还是brokers将消息推送到consumer，也
就是pull还push。在这方面，Kafka遵循了一种大部分消息系统共同的传统的设计：producer将消息推
送到broker，consumer从broker拉取消息

一些消息系统比如Scribe和Apache Flume采用了push模式，将消息推送到下游的consumer。这样做
有好处也有坏处：由broker决定消息推送的速率，对于不同消费速率的consumer就不太好处理了。消
息系统都致力于让consumer以最大的速率最快速的消费消息，但不幸的是，push模式下，当broker推
送的速率远大于consumer消费的速率时，consumer恐怕就要崩溃了。最终Kafka还是选取了传统的
pull模式

Pull模式的另外一个好处是consumer可以自主决定是否批量的从broker拉取数据。Push模式必须在不
知道下游consumer消费能力和消费策略的情况下决定是立即推送每条消息还是缓存之后批量推送。如
果为了避免consumer崩溃而采用较低的推送速率，将可能导致一次只推送较少的消息而造成浪费。
Pull模式下，consumer就可以根据自己的消费能力去决定这些策略。Pull有个缺点是，如果broker没有
可供消费的消息，将导致consumer不断在循环中轮询，直到新消息到t达。为了避免这点，Kafka有个
参数可以让consumer阻塞知道新消息到达(当然也可以阻塞知道消息的数量达到某个特定的量这样就可
以批量发送

### Kafka 高效文件存储设计特点？

1. Kafka把topic中一个parition大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除
   已经消费完文件，减少磁盘占用
2. 通过索引信息可以快速定位message和确定response的最大大小
3. 通过index元数据全部映射到memory，可以避免segment file的IO磁盘操作
4. 通过索引文件稀疏存储，可以大幅降低index文件元数据占用空间大小

### Kafka创建Topic时如何将分区放置到不同的Broker中？

1. 副本因子不能大于 Broker 的个数
2. 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的\
3. 其他分区的第一个副本放置位置相对于第0个分区依次往后移。也就是如果我们有5个 Broker，5个
   分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推
4. 剩余的副本相对于第一个副本放置位置其实是由 nextReplicaShift 决定的，而这个数也是随机产生
   的

### Kafka新建的分区会在哪个目录下创建？

我们知道，在启动 Kafka 集群之前，我们需要配置好 log.dirs 参数，其值是 Kafka 数据的存放目录，这
个参数可以配置多个目录，目录之间使用逗号分隔，通常这些目录是分布在不同的磁盘上用于提高读写
性能。当然我们也可以配置 log.dir 参数，含义一样。只需要设置其中一个即可

如果 log.dirs 参数只配置了一个目录，那么分配到各个 Broker 上的分区肯定只能在这个目录下创建文
件夹用于存放数据

但是如果 log.dirs 参数配置了多个目录，那么 Kafka 会在哪个文件夹中创建分区目录呢？答案是：
Kafka 会在含有分区目录最少的文件夹中创建新的分区目录，分区目录名为 Topic名+分区ID。注意，是
分区文件夹总数最少的目录，而不是磁盘使用量最少的目录！也就是说，如果你给 log.dirs 参数新增了
一个新的磁盘，新的分区目录肯定是先在这个新的磁盘上创建直到这个新的磁盘目录拥有的分区目录不
是最少为止

### 谈一谈 Kafka 的再均衡

在Kafka中，当有新消费者加入或者订阅的topic数发生变化时，会触发Rebalance(再均衡：在同一个消
费者组当中，分区的所有权从一个消费者转移到另外一个消费者)机制，Rebalance顾名思义就是重新均
衡消费者消费。Rebalance的过程如下：

- 第一步：所有成员都向coordinator发送请求，请求入组。一旦所有成员都发送了请求，coordinator会
  从中选择一个consumer担任leader的角色，并把组成员信息以及订阅信息发给leader
- 第二步：leader开始分配消费方案，指明具体哪个consumer负责消费哪些topic的哪些partition。一旦
  完成分配，leader会将这个方案发给coordinator。coordinator接收到分配方案之后会把方案发给各个
  consumer，这样组内的所有成员就都知道自己应该消费哪些分区了
- 所以对于Rebalance来说，Coordinator起着至关重要的作用

### Kafka 是如何实现高吞吐率的？

Kafka是分布式消息系统，需要处理海量的消息，Kafka的设计是把所有的消息都写入速度低容量大的硬
盘，以此来换取更强的存储能力，但实际上，使用硬盘并没有带来过多的性能损失。kafka主要使用了
以下几个方式实现了超高的吞吐率：

1. 顺序读写
2. 零拷贝
3. 文件分段
4. 批量发送
5. 数据压缩

### Kafka 缺点？

1. 由于是批量发送，数据并非真正的实时
2. 对于mqtt协议不支持
3. 不支持物联网传感数据直接接入
4. 仅支持统一分区内消息有序，无法实现全局消息有序
5. 监控不完善，需要安装插件
6. 依赖zookeeper进行元数据管理。

### Kafka 分区数可以增加或减少吗？为什么？

我们可以使用 bin/kafka-topics.sh 命令对 Kafka 增加 Kafka 的分区数据，但是 Kafka 不支持减少分区
数。 Kafka 分区数据不支持减少是由很多原因的，比如减少的分区其数据放到哪里去？是删除，还是保
留？删除的话，那么这些没消费的消息不就丢了。如果保留这些消息如何放到其他分区里面？追加到其
他分区后面的话那么就破坏了 Kafka 单个分区的有序性。如果要保证删除分区数据插入到其他分区保证
有序性，那么实现起来逻辑就会非常复杂