## Shuffle简介

Shuffle描述着数据从map task输出到reduce  task输入的这段过程。shuffle是连接Map和Reduce之间的桥梁，Map的输出要用到Reduce中必须经过shuffle这个环节，shuffle的性能高低直接影响了整个程序的性能和吞吐量。因为在分布式情况下，reduce  task需要跨节点去拉取其它节点上的map  task结果。这一过程将会产生网络资源消耗和内存，磁盘IO的消耗。通常shuffle分为两部分：Map阶段的数据准备和Reduce阶段的数据拷贝处理。一般将在map端的Shuffle称之为Shuffle  Write，在Reduce端的Shuffle称之为Shuffle Read.

## Hadoop MapReduce Shuffle

Apache Spark 的 Shuffle 过程与 Apache  Hadoop 的 Shuffle 过程有着诸多类似，一些概念可直接套用，例如，Shuffle 过程中，提供数据的一端，被称作 Map 端，Map  端每个生成数据的任务称为 Mapper，对应的，接收数据的一端，被称作 Reduce 端，Reduce 端每个拉取数据的任务称为  Reducer，Shuffle 过程本质上都是将 Map 端获得的数据使用分区器进行划分，并将数据发送给对应的 Reducer 的过程。

![1637842781543](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/201942-821414.png)

### map端的Shuffle简述:

1)input, 根据split输入数据，运行map任务;

2)patition, 每个map task都有一个内存缓冲区，存储着map的输出结果;

3)spill, 当缓冲区快满的时候需要将缓冲区的数据以临时文件的方式存放到磁盘;

4)merge, 当整个map task结束后再对磁盘中这个map task产生的所有临时文件做合并，生成最终的正式输出文件，然后等待reduce task来拉数据。

### reduce 端的Shuffle简述:

reduce task在执行之前的工作就是不断地拉取当前job里每个map task的最终结果，然后对从不同地方拉取过来的数据不断地做merge，也最终形成一个文件作为reduce task的输入文件。

1) Copy过程，拉取数据。

2)Merge阶段，合并拉取来的小文件

3)Reducer计算

4)Output输出计算结果

我们可以将Shuffle的过程以数据流的方式呈现：

![1637842832122](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202033-696588.png)

图形象的描述了MR数据流动的整个过程：

图解释：

map端，有4个map;Reduce端，有3个reduce。4个map  也就是4个JVM，每个JVM处理一个数据分片(split1~split4)，每个map产生一个map输出文件，但是每个map都为后面的reduce产生了3部分数据(分别用红1、绿2、蓝3标识)，也就是说每个输出的map文件都包含了3部分数据。正如前面第二节所述：

mapper运行后，通过Partitioner接口，根据key或value及reduce的数量来决定当前map的输出数据最终应该交由哪个reduce task处理.Reduce端一共有3个reduce，去前面的4个map的输出结果中抓取属于自己的数据。

关于Hadoop MR的Shuffle的详细请查看博客：“戏”说hadoop--hadoop MapReduce Shuffle过程详解

## Spark Shuffle

![1637842851485](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202052-237171.png)

在“戏”说Spark-Spark核心-Stage划分及Pipline的计算模式一文中，我们知道stage中是高效快速的pipline的计算模式，宽依赖之间会划分stage，而Stage之间就是Shuffle，如图中的stage0，stage1和stage3之间就会产生Shuffle。

在Spark的中，负责shuffle过程的执行、计算和处理的[组件](https://www.2cto.com/kf/all/zujian/)主要就是ShuffleManager，也即shuffle管理器。ShuffleManager随着Spark的发展有两种实现的方式，分别为HashShuffleManager和SortShuffleManager，因此spark的Shuffle有Hash  Shuffle和Sort Shuffle两种

### Spark Shuffle发展史

在Spark的版本的发展，ShuffleManager在不断迭代，变得越来越先进。

在Spark  1.2以前，默认的shuffle计算引擎是HashShuffleManager。该ShuffleManager而HashShuffleManager有着一个非常严重的弊端，就是会产生大量的中间磁盘文件，进而由大量的磁盘IO操作影响了性能。因此在Spark   1.2以后的版本中，默认的ShuffleManager改成了SortShuffleManager。SortShuffleManager相较于HashShuffleManager来说，有了一定的改进。主要就在于，每个Task在进行shuffle操作时，虽然也会产生较多的临时磁盘文件，但是最后会将所有的临时文件合并(merge)成一个磁盘文件，因此每个Task就只有一个磁盘文件。在下一个stage的shuffle  read task拉取自己的数据时，只要根据索引读取每个磁盘文件中的部分数据即可。

### Hash shuffle

HashShuffleManager的运行机制主要分成两种，一种是普通运行机制，另一种是合并的运行机制。

合并机制主要是通过复用buffer来优化Shuffle过程中产生的小文件的数量。Hash shuffle是不具有排序的Shuffle。

普通机制的Hash shuffle

![1637842872818](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202113-78964.png)

图解：

这里我们先明确一个假设前提：每个Executor只有1个CPU core，也就是说，无论这个Executor上分配多少个task线程，同一时间都只能执行一个task线程。

图中有3个 Reducer，从Task 开始那边各自把自己进行 Hash  计算(分区器：hash/numreduce取模)，分类出3个不同的类别，每个 Task  都分成3种类别的数据，想把不同的数据汇聚然后计算出最终的结果，所以Reducer 会在每个 Task  中把属于自己类别的数据收集过来，汇聚成一个同类别的大集合，每1个 Task 输出3份本地文件，这里有4个 Mapper  Tasks，所以总共输出了4个 Tasks x 3个分类文件 = 12个本地小文件。

1：shuffle write阶段

主要就是在一个stage结束计算之后，为了下一个stage可以执行shuffle类的算子(比如reduceByKey，groupByKey)，而将每个task处理的数据按key进行“分区”。所谓“分区”，就是对相同的key执行hash算法，从而将相同key都写入同一个磁盘文件中，而每一个磁盘文件都只属于reduce端的stage的一个task。在将数据写入磁盘之前，会先将数据写入内存缓冲中，当内存缓冲填满之后，才会溢写到磁盘文件中去。

那么每个执行shuffle  write的task，要为下一个stage创建多少个磁盘文件呢?很简单，下一个stage的task有多少个，当前stage的每个task就要创建多少份磁盘文件。比如下一个stage总共有100个task，那么当前stage的每个task都要创建100份磁盘文件。如果当前stage有50个task，总共有10个Executor，每个Executor执行5个Task，那么每个Executor上总共就要创建500个磁盘文件，所有Executor上会创建5000个磁盘文件。由此可见，未经优化的shuffle  write操作所产生的磁盘文件的数量是极其惊人的。

2：shuffle read阶段

shuffle  read，通常就是一个stage刚开始时要做的事情。此时该stage的每一个task就需要将上一个stage的计算结果中的所有相同key，从各个节点上通过网络都拉取到自己所在的节点上，然后进行key的聚合或连接等操作。由于shuffle  write的过程中，task给Reduce端的stage的每个task都创建了一个磁盘文件，因此shuffle  read的过程中，每个task只要从上游stage的所有task所在节点上，拉取属于自己的那一个磁盘文件即可。

shuffle read的拉取过程是一边拉取一边进行聚合的。每个shuffle  read  task都会有一个自己的buffer缓冲，每次都只能拉取与buffer缓冲相同大小的数据，然后通过内存中的一个Map进行聚合等操作。聚合完一批数据后，再拉取下一批数据，并放到buffer缓冲中进行聚合操作。以此类推，直到最后将所有数据到拉取完，并得到最终的结果。

注意：

1).buffer起到的是缓存作用，缓存能够加速写磁盘，提高计算的效率,buffer的默认大小32k。

分区器：根据hash/numRedcue取模决定数据由几个Reduce处理，也决定了写入几个buffer中

block file：磁盘小文件，从图中我们可以知道磁盘小文件的个数计算公式：

block file=M*R

2).M为map task的数量，R为Reduce的数量，一般Reduce的数量等于buffer的数量，都是由分区器决定的

Hash shuffle普通机制的问题

1).Shuffle前在磁盘上会产生海量的小文件，建立通信和拉取数据的次数变多,此时会产生大量耗时低效的 IO 操作 (因為产生过多的小文件)

2).可能导致OOM，大量耗时低效的 IO 操作  ，导致写磁盘时的对象过多，读磁盘时候的对象也过多，这些对象存储在堆内存中，会导致堆内存不足，相应会导致频繁的GC，GC会导致OOM。由于内存中需要保存海量文件操作句柄和临时信息，如果数据处理的规模比较庞大的话，内存不可承受，会出现  OOM 等问题。

合并机制的Hash shuffle

合并机制就是复用buffer，开启合并机制的配置是spark.shuffle.consolidateFiles。该参数默认值为false，将其设置为true即可开启优化机制。通常来说，如果我们使用HashShuffleManager，那么都建议开启这个选项。

![1637842896185](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202137-422668.png)

这里还是有4个Tasks，数据类别还是分成3种类型，因为Hash算法会根据你的  Key  进行分类，在同一个进程中，无论是有多少过Task，都会把同样的Key放在同一个Buffer里，然后把Buffer中的数据写入以Core数量为单位的本地文件中，(一个Core只有一种类型的Key的数据)，每1个Task所在的进程中，分别写入共同进程中的3份本地文件，这里有4个Mapper  Tasks，所以总共输出是 2个Cores x 3个分类文件 = 6个本地小文件。

图解：

开启consolidate机制之后，在shuffle  write过程中，task就不是为下游stage的每个task创建一个磁盘文件了。此时会出现shuffleFileGroup的概念，每个shuffleFileGroup会对应一批磁盘文件，磁盘文件的数量与下游stage的task数量是相同的。一个Executor上有多少个CPU   core，就可以并行执行多少个task。而第一批并行执行的每个task都会创建一个shuffleFileGroup，并将数据写入对应的磁盘文件内。

Executor的CPU  core执行完一批task，接着执行下一批task时，下一批task就会复用之前已有的shuffleFileGroup，包括其中的磁盘文件。也就是说，此时task会将数据写入已有的磁盘文件中，而不会写入新的磁盘文件中。因此，consolidate机制允许不同的task复用同一批磁盘文件，这样就可以有效将多个task的磁盘文件进行一定程度上的合并，从而大幅度减少磁盘文件的数量，进而提升shuffle  write的性能。

假设第二个stage有100个task，第一个stage有50个task，总共还是有10个Executor，每个Executor执行5个task。那么原本使用未经优化的HashShuffleManager时，每个Executor会产生500个磁盘文件，所有Executor会产生5000个磁盘文件的。但是此时经过优化之后，每个Executor创建的磁盘文件的数量的计算公式为：CPU  core的数量 *  下一个stage的task数量。也就是说，每个Executor此时只会创建100个磁盘文件，所有Executor只会创建1000个磁盘文件。

注意：

1).启动HashShuffle的合并机制ConsolidatedShuffle的配置:

spark.shuffle.consolidateFiles=true

2).block file=Core*R

Core为CPU的核数，R为Reduce的数量

Hash shuffle合并机制的问题

如果 Reducer 端的并行任务或者是数据分片过多的话则 Core * Reducer Task 依旧过大，也会产生很多小文件。

### Sort shuffle

SortShuffleManager的运行机制主要分成两种，一种是普通运行机制，另一种是bypass运行机制。当shuffle  read  task的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值时(默认为200)，就会启用bypass机制。

Sort shuffle的普通机制

图解：

![1637842926658](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202207-364488.png)

写入内存数据结构

该图说明了普通的SortShuffleManager的原理。在该模式下，数据会先写入一个内存数据结构中(默认5M)，此时根据不同的shuffle算子，可能选用不同的数据结构。如果是reduceByKey这种聚合类的shuffle算子，那么会选用Map数据结构，一边通过Map进行聚合，一边写入内存;如果是join这种普通的shuffle算子，那么会选用Array数据结构，直接写入内存。接着，每写一条数据进入内存数据结构之后，就会判断一下，是否达到了某个临界阈值。如果达到临界阈值的话，那么就会尝试将内存数据结构中的数据溢写到磁盘，然后清空内存数据结构。

注意：

shuffle中的定时器：定时器会检查内存数据结构的大小，如果内存数据结构空间不够，那么会申请额外的内存，申请的大小满足如下公式：

applyMemory=nowMenory*2-oldMemory

申请的内存=当前的内存情况*2-上一次的内嵌情况

意思就是说内存数据结构的大小的动态变化，如果存储的数据超出内存数据结构的大小，将申请内存数据结构存储的数据*2-内存数据结构的设定值的内存大小空间。申请到了，内存数据结构的大小变大，内存不够，申请不到，则发生溢写

排序

在溢写到磁盘文件之前，会先根据key对内存数据结构中已有的数据进行排序。

溢写

排序过后，会分批将数据写入磁盘文件。默认的batch数量是10000条，也就是说，排序好的数据，会以每批1万条数据的形式分批写入磁盘文件。写入磁盘文件是通过[Java](https://www.2cto.com/kf/ware/Java/)的BufferedOutputStream实现的。BufferedOutputStream是Java的缓冲输出流，首先会将数据缓冲在内存中，当内存缓冲满溢之后再一次写入磁盘文件中，这样可以减少磁盘IO次数，提升性能。

merge

一个task将所有数据写入内存数据结构的过程中，会发生多次磁盘溢写操作，也就会产生多个临时文件。最后会将之前所有的临时磁盘文件都进行合并，这就是merge过程，此时会将之前所有临时磁盘文件中的数据读取出来，然后依次写入最终的磁盘文件之中。此外，由于一个task就只对应一个磁盘文件，也就意味着该task为Reduce端的stage的task准备的数据都在这一个文件中，因此还会单独写一份索引文件，其中标识了下游各个task的数据在文件中的start  offset与end offset。

SortShuffleManager由于有一个磁盘文件merge的过程，因此大大减少了文件数量。比如第一个stage有50个task，总共有10个Executor，每个Executor执行5个task，而第二个stage有100个task。由于每个task最终只有一个磁盘文件，因此此时每个Executor上只有5个磁盘文件，所有Executor只有50个磁盘文件。

注意：

1)block file= 2M

一个map task会产生一个索引文件和一个数据大文件

2) m*r>2m(r>2)：SortShuffle会使得磁盘小文件的个数再次的减少

### Sort shuffle的bypass机制

![1637842949611](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/25/202230-416350.png)

bypass运行机制的触发条件如下：

1)shuffle map task数量小于spark.shuffle.sort.bypassMergeThreshold参数的值。

2)不是聚合类的shuffle算子(比如reduceByKey)。

此时task会为每个reduce端的task都创建一个临时磁盘文件，并将数据按key进行hash然后根据key的hash值，将key写入对应的磁盘文件之中。当然，写入磁盘文件时也是先写入内存缓冲，缓冲写满之后再溢写到磁盘文件的。最后，同样会将所有临时磁盘文件都合并成一个磁盘文件，并创建一个单独的索引文件。

该过程的磁盘写机制其实跟未经优化的HashShuffleManager是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。因此少量的最终磁盘文件，也让该机制相对未经优化的HashShuffleManager来说，shuffle  read的性能会更好。

而该机制与普通SortShuffleManager运行机制的不同在于：

第一，磁盘写机制不同;

第二，不会进行排序。也就是说，启用该机制的最大好处在于，shuffle write过程中，不需要进行数据的排序操作，也就节省掉了这部分的性能开销。

总结：

Shuffle 过程本质上都是将 Map 端获得的数据使用分区器进行划分，并将数据发送给对应的 Reducer 的过程。

shuffle作为处理连接map端和reduce端的枢纽，其shuffle的性能高低直接影响了整个程序的性能和吞吐量。map端的shuffle一般为shuffle的Write阶段，reduce端的shuffle一般为shuffle的read阶段。Hadoop和spark的shuffle在实现上面存在很大的不同，spark的shuffle分为两种实现，分别为HashShuffle和SortShuffle，

HashShuffle又分为普通机制和合并机制，普通机制因为其会产生M*R个数的巨量磁盘小文件而产生大量性能低下的Io操作，从而性能较低，因为其巨量的磁盘小文件还可能导致OOM，HashShuffle的合并机制通过重复利用buffer从而将磁盘小文件的数量降低到Core*R个，但是当Reducer  端的并行任务或者是数据分片过多的时候，依然会产生大量的磁盘小文件。

SortShuffle也分为普通机制和bypass机制，普通机制在内存数据结构(默认为5M)完成排序，会产生2M个磁盘小文件。而当shuffle  map  task数量小于spark.shuffle.sort.bypassMergeThreshold参数的值。或者算子不是聚合类的shuffle算子(比如reduceByKey)的时候会触发SortShuffle的bypass机制，SortShuffle的bypass机制不会进行排序，极大的提高了其性能

在Spark  1.2以前，默认的shuffle计算引擎是HashShuffleManager，因为HashShuffleManager会产生大量的磁盘小文件而性能低下，在Spark   1.2以后的版本中，默认的ShuffleManager改成了SortShuffleManager。SortShuffleManager相较于HashShuffleManager来说，有了一定的改进。主要就在于，每个Task在进行shuffle操作时，虽然也会产生较多的临时磁盘文件，但是最后会将所有的临时文件合并(merge)成一个磁盘文件，因此每个Task就只有一个磁盘文件。在下一个stage的shuffle  read task拉取自己的数据时，只要根据索引读取每个磁盘文件中的部分数据即可。