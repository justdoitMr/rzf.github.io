## Spark中的join操作

### JOIN的条件

JOIN的条件会涉及字段之间的逻辑比较。根据JOIN的条件，JOIN可分为两大类：**等值连接**和**非等值连接**。等值连接会涉及一个或多个需要同时满足的相等条件。在两个输入数据集的属性之间应用每个等值条件。当使用其他运算符(运算连接符不为**=**)时，称之为非等值连接。

### JOIN的类型

在输入数据集的记录之间应用连接条件之后，JOIN类型会影响JOIN操作的结果。主要有以下几种JOIN类型：

- 内连接(*Inner Join*)：仅从输入数据集中输出匹配连接条件的记录。
- 外连接(*Outer Join*)：又分为左外连接、右外链接和全外连接。
- 半连接(*Semi Join*)：右表只用于过滤左表的数据而不出现在结果集中。
- 交叉连接(*Cross Join*)：交叉联接返回左表中的所有行，左表中的每一行与右表中的所有行组合。交叉联接也称作笛卡尔积。

### Spark中JOIN执行的5种策略

Spark提供了5种JOIN机制来执行具体的JOIN操作。该5种JOIN机制如下所示：

- Shuffle Hash Join
- Broadcast Hash Join
- Sort Merge Join
- Cartesian Join
- Broadcast Nested Loop Join

> Shuffle Hash Join : 适合一张小表和一张大表进行join，或者是两张小表之间的join
>
> Broadcast Hash Join ： 适合一张较小的表和一张大表进行join
>
> Sort Merge Join ： 适合两张较大的表之间进行join

前两者都基于的是Hash Join，只不过在hash join之前需要先shuffle还是先broadcast。下面将详细的解释一下这三种不同的join的具体原理

### Shuffle Hash Join

#### 传统join操作

先来看看这样一条SQL语句：`select * from order,item where item.id = order.i_id`，很简单一个Join节点，参与join的两张表是item和order，join key分别是item.id以及order.i_id。现在假设这个Join采用的是hash join算法，整个过程会经历三步：

1. 确定Build Table以及Probe Table：这个概念比较重要，Build Table使用join key构建Hash Table，而Probe Table使用join key进行探测，探测成功就可以join在一起。通常情况下，**小表会作为Build Table，大表作为Probe Table**。此事例中item为Build Table，order为Probe Table；
2. 构建Hash Table：依次读取Build Table（item）的数据，对于每一行数据根据join key（`item.id`）进行hash，hash到对应的Bucket，生成hash table中的一条记录。数据缓存在内存中，如果内存放不下需要dump到外存；
3. 探测：再依次扫描Probe Table（order）的数据，使用相同的hash函数映射Hash Table中的记录，映射成功之后再检查join条件（`item.id = order.i_id`），如果匹配成功就可以将两者join在一起。

![1639633211549](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/134012-316526.png)

为什么Build Table选择小表？

道理很简单，因为构建的Hash Table最好能全部加载在内存，效率最高；**这也决定了hash join算法只适合至少一个小表的join场景**，对于两个大表的join场景并不适用。

上文说过，hash join是传统数据库中的单机join算法，在分布式环境下需要经过一定的分布式改造，说到底就是尽可能利用分布式计算资源进行并行化计算，提高总体效率。hash join分布式改造一般有两种经典方案：

1. broadcast hash join：将其中一张小表广播分发到另一张大表所在的分区节点上，分别并发地与其上的分区记录进行hash join。broadcast适用于小表很小，可以直接广播的场景；
2. shuffler hash join：一旦小表数据量较大，此时就不再适合进行广播分发。这种情况下，可以根据join key相同必然分区相同的原理，将两张表分别按照join key进行重新组织分区，这样就可以将join分而治之，划分为很多小join，充分利用集群资源并行化。x`

#### shuffler hash join简介

当要JOIN的表数据量比较大时，可以选择Shuffle Hash Join。这样可以将大表进行**按照JOIN的key进行重分区**，保证每个相同的JOIN key都发送到同一个分区中。

> 因为broadcast策略首先是收集数据到driver节点，然后分发到每个executor节点，当表太大时，broadcastchelve将会给driver和executor造成很大压力。
>
> Shuffle Hash Join会减少对driver和exeuctor的压力，操作步骤如下：
>
> 1. 两张表分别按照连接列进行重组，目的是相同连接列的记录分配到同一个分区
> 2. 两张表的小表分区构造成hash表，大表根据相应的记录进行映射

![1639629569895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/123930-547177.png)

Shuffle Hash Join的基本步骤主要有以下两点：

- 首先，对于两张参与JOIN的表，分别按照join key进行重分区，该过程会涉及Shuffle，其目的是将相同join key的数据发送到同一个分区，方便分区内进行join。
- 其次，对于每个Shuffle之后的分区，会将小表的分区数据构建成一个Hash table，然后根据join key与大表的分区数据记录进行匹配。

#### 条件与特点

- 仅支持等值连接，join key不需要排序
- 支持除了全外连接(full outer joins)之外的所有join类型
- 需要对小表构建Hash map，属于内存密集型的操作，如果构建Hash表的一侧数据比较大，可能会造成OOM
- 将参数*spark.sql.join.prefersortmergeJoin (default true)*置为false

### Broadcast Hash Join

#### 为什么需要Broadcast Hash Join

大家知道，在数据库的常见模型中（比如星型模型或者雪花模型），表一般分为两种：事实表和维度表。维度表一般指固定的、变动较少的表，例如联系人、物品种类等，一般数据有限。而事实表一般记录流水，比如销售清单等，通常随着时间的增长不断膨胀。

因为Join操作是对两个表中key值相同的记录进行连接，在SparkSQL中，对两个表做Join最直接的方式是先根据key分区，再在每个分区中把key值相同的记录拿出来做连接操作。但这样就不可避免地涉及到shuffle，而shuffle在Spark中是比较耗时的操作，我们应该尽可能的设计Spark应用使其避免大量的shuffle。

当维度表和事实表进行Join操作时，为了避免shuffle，我们可以将大小有限的维度表的全部数据分发到每个节点上，供事实表使用。executor存储维度表的全部数据，一定程度上牺牲了空间，换取shuffle操作大量的耗时，这在SparkSQL中称作Broadcast Join，如下图所示：

![1639633593692](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/134635-311294.png)

Table B是较小的表，黑色表示将其广播到每个executor节点上，Table A的每个partition会通过block manager取到Table A的数据。根据每条记录的Join Key取到Table B中相对应的记录，根据Join Type进行操作。这个过程比较简单，不做赘述。

Broadcast Join的条件有以下几个：

1. 被广播的表需要小于spark.sql.autoBroadcastJoinThreshold所配置的值，默认是10M （或者加了broadcast join的hint）
2. 基表不能被广播，比如left outer join时，只能广播右表

看起来广播是一个比较理想的方案，但它有没有缺点呢？也很明显。这个方案只能用于广播较小的表，否则数据的冗余传输就远大于shuffle的开销；另外，广播时需要将被广播的表现collect到driver端，当频繁有广播出现时，对driver的内存也是一个考验。

如下图所示，broadcast hash join可以分为两步：

1. broadcast阶段：将小表广播分发到大表所在的所有主机。广播算法可以有很多，最简单的是先发给driver，driver再统一分发给所有executor；要不就是基于bittorrete的p2p思路；
2. hash join阶段：在每个executor上执行单机版hash join，小表映射，大表试探；

![1639633677946](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/134915-975446.png)

SparkSQL规定broadcast hash join执行的基本条件为被广播小表必须小于参数spark.sql.autoBroadcastJoinThreshold，默认为10M。

Shuffle Hash Join
当一侧的表比较小时，我们选择将其广播出去以避免shuffle，提高性能。但因为被广播的表首先被collect到driver段，然后被冗余分发到每个executor上，所以当表比较大时，采用broadcast join会对driver端和executor端造成较大的压力。

但由于Spark是一个分布式的计算引擎，可以通过分区的形式将大批量的数据划分成n份较小的数据集进行并行计算。这种思想应用到Join上便是Shuffle Hash Join了。利用key相同必然分区相同的这个原理，SparkSQL将较大表的join分而治之，先将表划分成n个分区，再对两个表中相对应分区的数据分别进行Hash Join，这样即在一定程度上减少了driver广播一侧表的压力，也减少了executor端取整张被广播表的内存消耗。其原理如下图：

![1639629569895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/123930-547177.png)

Shuffle Hash Join分为两步：

1. 对两张表分别按照join keys进行重分区，即shuffle，目的是为了让有相同join keys值的记录分到对应的分区中
2. 对对应分区中的数据进行join，此处先将小表分区构造为一张hash表，然后根据大表分区中记录的join keys值拿出来进行匹配

Shuffle Hash Join的条件有以下几个：

1. 分区的平均大小不超过spark.sql.autoBroadcastJoinThreshold所配置的值，默认是10M
2. 基表不能被广播，比如left outer join时，只能广播右表
3. 一侧的表要明显小于另外一侧，小的一侧将被广播（明显小于的定义为3倍小，此处为经验值）

我们可以看到，在一定大小的表中，SparkSQL从时空结合的角度来看，将两个表进行重新分区，并且对小表中的分区进行hash化，从而完成join。在保持一定复杂度的基础上，尽量减少driver和executor的内存压力，提升了计算时的稳定性。

在大数据条件下如果一张表很小，执行join操作最优的选择无疑是broadcast hash join，效率最高。但是一旦小表数据量增大，广播所需内存、带宽等资源必然就会太大，broadcast hash join就不再是最优方案。此时可以按照join key进行分区，根据key相同必然分区相同的原理，就可以将大表join分而治之，划分为很多小表的join，充分利用集群资源并行化。如下图所示，shuffle hash join也可以分为两步：

1. shuffle阶段：分别将两个表按照join key进行分区，将相同join key的记录重分布到同一节点，两张表的数据会被重分布到集群中所有节点。这个过程称为shuffle
2. hash join阶段：每个分区节点上的数据单独执行单机hash join算法。

![1639633776280](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/134956-629293.png)

看到这里，可以初步总结出来如果两张小表join可以直接使用单机版hash join；如果一张大表join一张极小表，可以选择broadcast hash join算法；而如果是一张大表join一张小表，则可以选择shuffle hash join算法；那如果是两张大表进行join呢？

#### 简介

也称之为**Map端JOIN**。当有一张表较小时，我们通常选择Broadcast Hash Join，这样可以避免Shuffle带来的开销，从而提高性能。比如事实表与维表进行JOIN时，由于维表的数据通常会很小，所以可以使用Broadcast Hash Join将维表进行Broadcast。这样可以避免数据的Shuffle(在Spark中Shuffle操作是很耗时的)，从而提高JOIN的效率。在进行 Broadcast Join 之前，Spark 需要把处于 Executor 端的数据先发送到 Driver 端，然后 Driver 端再把数据广播到 Executor 端。如果我们需要广播的数据比较多，会造成 Driver 端出现 OOM。

![1639629721344](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/124202-690557.png)

Broadcast Hash Join主要包括两个阶段：

- **Broadcast**阶段 ：小表被缓存在executor中
- **Hash Join**阶段：在每个 executor中执行Hash Join

#### 条件与特点

- Broadcast Hash Join相比其他的JOIN机制而言，效率更高。但是，Broadcast Hash Join属于网络密集型的操作(数据冗余传输)，除此之外，需要在Driver端缓存数据，所以当小表的数据量较大时，会出现OOM的情况
- 被广播的小表的数据量要小于**spark.sql.autoBroadcastJoinThreshold**值，默认是10MB(10485760)
- 被广播表的大小阈值不能超过8GB，spark2.4源码如下：**BroadcastExchangeExec.scala**

~~~ java
longMetric("dataSize") += dataSize
if (dataSize >= (8L << 30)) {
throw new SparkException(
s"Cannot broadcast the table that is larger than 8GB: ${dataSize >> 30} GB")
}
~~~

> 1. 表需要broadcast，那么必须小于spark.sql.autoBroadcastJoinThreshold配置的值（默认是10M），或者明确添加broadcast join hint。
> 2. base table不能broadcast，例如在left outer join中，仅仅right表可以broadcast
> 3. 这种算法仅仅用来broadcast小表，否则数据的传输可能比shuffle操作成本高
> 4. broadcast也需要到driver，如果有太多的broadcast，对driver内存也是压力
>
> 条件和特点
>
> - 仅支持等值连接，join key不需要排序
> - 支持除了全外连接(full outer joins)之外的所有join类型
> - Broadcast Hash Join相比其他的JOIN机制而言，效率更高。但是，Broadcast Hash Join属于网络密集型的操作(数据冗余传输)，除此之外，需要在Driver端缓存数据，所以当小表的数据量较大时，会出现OOM的情况
> - 被广播的小表的数据量要小于**spark.sql.autoBroadcastJoinThreshold**值，默认是10MB(10485760)
> - 被广播表的大小阈值不能超过8GB，spark2.4源码如下：**BroadcastExchangeExec.scala**

### Sort Merge Join

#### 详细过程

上面介绍的两种实现对于一定大小的表比较适用，但当两个表都非常大时，显然无论适用哪种都会对计算内存造成很大压力。这是因为join时两者采取的都是hash join，是将一侧的数据完全加载到内存中，使用hash code取join keys值相等的记录进行连接。

当两个表都非常大时，SparkSQL采用了一种全新的方案来对表进行Join，即Sort Merge Join。这种实现方式不用将一侧数据全部加载后再进星hash join，但需要在join前将数据排序，如下图所示：

可以看到，首先将两张表按照join keys进行了重新shuffle，保证join keys值相同的记录会被分在相应的分区。分区后对每个分区内的数据进行排序，排序后再对相应的分区内的记录进行连接，如下图示：

看着很眼熟吧？也很简单，因为两个序列都是有序的，从头遍历，碰到key相同的就输出；如果不同，左边小就继续取左边，反之取右边。

可以看出，无论分区有多大，Sort Merge Join都不用把某一侧的数据全部加载到内存中，而是即用即取即丢，从而大大提升了大数据量下sql join的稳定性。

SparkSQL对两张大表join采用了全新的算法－sort-merge join，如下图所示，整个过程分为三个步骤：

![1639634115230](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/135523-424695.png)

![1646303291505](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/03/182812-647977.png)

1. shuffle阶段：将两张大表根据join key进行重新分区，两张表数据会分布到整个集群，以便分布式并行处理；
2. sort阶段：对单个分区节点的两表数据，分别进行排序；
3. merge阶段：对排好序的两张分区表数据执行join操作。join操作很简单，分别遍历两个有序序列，碰到相同join key就merge输出，否则取更小一边，见下图示意：

![1639634134482](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/135544-617294.png)

经过上文的分析，可以明确每种Join算法都有自己的适用场景，数据仓库设计时最好避免大表与大表的join查询，SparkSQL也可以根据内存资源、带宽资源适量将参数spark.sql.autoBroadcastJoinThreshold调大，让更多join实际执行为broadcast hash join。

#### 简介

该JOIN机制是Spark默认的，可以通过参数**spark.sql.join.preferSortMergeJoin**进行配置，默认是true，即优先使用Sort Merge Join。一般在**两张大表**进行JOIN时，使用该方式。Sort Merge Join可以减少集群中的数据传输，该方式不会先加载所有数据的到内存，然后进行hashjoin，但是在JOIN之前需要对join key进行排序。具体图示：

> 上面两种发现适应一定大小的表，但是当两张表足够大时，上面方法对内存造成很大压力，这是因为当两张表做Hash Join时，其中一张表必须完成加载到内存中。
>
> 当两张表很大时，Spark SQL使用一种新的算法来做Join操作，叫做Sort Merge Join，这种算法并不会加载所有的数据，然后开始Hash Join，而是在Join之前进行数据排序。
>
> 两张表需要进行数据重组，保证相同连接列值到一个分区中，当分区好数据后，排序分区中的数据，然后相应的记录进行关联。
>
> 不管表分区多大，Sort Merge Join并不会加载一张表的所有数据到内存。

![1639629872191](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/16/124433-51400.png)	Sort Merge Join主要包括三个阶段：

- **Shuffle Phase** : 两张大表根据Join key进行Shuffle重分区
- **Sort Phase**: 每个分区内的数据进行排序
- **Merge Phase**: 对来自不同表的排序好的分区数据进行JOIN，通过遍历元素，连接具有相同Join key值的行来合并数据集

#### 条件与特点

- 仅支持等值连接
- 支持所有join类型
- Join Keys是排序的
- 参数**spark.sql.join.prefersortmergeJoin (默认true)**设定为true

### Cartesian Join

#### 简介

如果 Spark 中两张参与 Join 的表没指定join key（ON 条件）那么会产生 Cartesian product join，这个 Join 得到的结果其实就是两张行数的乘积。

#### 条件

- 仅支持内连接
- 支持等值和不等值连接
- 开启参数spark.sql.crossJoin.enabled=true

#### 简介

该方式是在没有合适的JOIN机制可供选择时，最终会选择该种join策略。优先级为：*Broadcast Hash Join > Sort Merge Join > Shuffle Hash Join > cartesian Join > Broadcast Nested Loop Join*.

在Cartesian 与Broadcast Nested Loop Join之间，如果是内连接，或者非等值连接，则优先选择Broadcast Nested Loop策略，当时非等值连接并且一张表可以被广播时，会选择Cartesian Join。

#### 条件与特点

- 支持等值和非等值连接

- 支持所有的JOIN类型，主要优化点如下：

- - 当右外连接时要广播左表
  - 当左外连接时要广播右表
  - 当内连接时，要广播左右两张表

### Spark是如何选择JOIN策略的

#### 等值连接的情况

##### 有join提示(hints)的情况，按照下面的顺序

1. Broadcast Hint：如果join类型支持，则选择broadcast hash join
2. Sort merge hint：如果join key是排序的，则选择 sort-merge join
3. shuffle hash hint：如果join类型支持， 选择 shuffle hash join
4. shuffle replicate NL hint： 如果是内连接，选择笛卡尔积方式

##### 没有join提示(hints)的情况，则逐个对照下面的规则

- 1.如果join类型支持，并且其中一张表能够被广播(值，默认是10MB)，则选择 broadcast hash join
- 2.如果参数**spark.sql.join.preferSortMergeJoin设定为false**，且一张表足够小(可以构建一个hash map) ，则选择shuffle hash join
- 3.如果join keys 是排序的，则选择sort-merge join
- 4.如果是内连接，选择 cartesian join
- 5.如果可能会发生OOM或者没有可以选择的执行策略，则最终选择broadcast nested loop join

#### 非等值连接情况

##### 有join提示(hints)，按照下面的顺序

- 1.broadcast hint：选择broadcast nested loop join.
- 2.shuffle replicate NL hint: 如果是内连接，则选择cartesian product join

##### 没有join提示(hints)，则逐个对照下面的规则

- 1.如果一张表足够小(可以被广播)，则选择 broadcast nested loop join
- 2.如果是内连接，则选择cartesian product join
- 3.如果可能会发生OOM或者没有可以选择的执行策略，则最终选择broadcast nested loop join




​    