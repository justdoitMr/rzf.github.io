# MapReduce原理

## MapReduce工作流程

MapReduce体系结构主要由四个部分组成，分别是：Client、JobTracker（ResourceManager）、TaskTracker（TaskManager）以及Task。

![1633674955663](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/143556-411934.png)

**作业提交过程**

1. 在客户端启动一个作业。
2. 向ResourceManager请求一个Job ID。
3. 将运行作业所需要的资源文件复制到HDFS上，包括MapReduce程序打包的jar文件、配置文件和客户端计算所得的计算划分信息。这些文件都存放在ResourceManager专门为该作业创建的文件夹中。文件夹名为该作业的Job ID。jar文件默认会有10个副本（mapred.submit.replication属性控制）；输入划分片信息告诉了ResourceManager应该为这个作业启动多少个map任务等信息，数据是逻辑上的划分，在HDFS上面的块才是物理上的划分。
4. ResourceManager接收到作业后，将其放在一个作业队列里，等待作业调度器对其进行调度（这里是不是很像微机中的进程调度呢），当作业调度器根据自己的调度算法调度到该作业时，会根据输入划分信息为每个划分创建一个map任务，并将map任务分配给TaskManager执行。对于map和reduce任务，TaskManager根据主机核的数量和内存的大小有固定数量的map槽和reduce槽。这里需强调的是：map任务不是随随便便地分配给某个TaskManager的，这里有个概念叫：数据本地化（Data-Local）。意思是：将map任务分配给含有该map处理的数据块的TaskManager上，同事将程序jar包复制到该TaskManager上来运行，这叫“运算移动，数据不移动”。而分配reduce任务时并不考虑数据本地化。
5. TaskManager每隔一段时间会给ResourceManager发送一个心跳，告诉ResourceManager它依然在运行，同时心跳中还携带者很多信息，比如当前map任务完成的进度等信息。当ResourceManager收到作业的最后一个任务完成信息时，便把该作业设置成“成功”。当ResourceManager查询状态时，它将得知任务已完成，便显示一条消息给用户。

## MapReduce任务的Shuffle和排序过程

![1633671588732](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/133949-142955.png)

### Map端流程分析

1. 每个输入分片会让一个map任务来处理，默认情况下，以HDFS的一个块的大小（默认128M）为一个分片，当然我们也可以设置块的大小。map输出的结果会暂且放在一个环形内存缓冲区中（该缓冲区的大小默认为100M，由io.sort.mb属性控制），当该缓冲区快要溢出时（默认为缓冲区大小的80%，由io.sort.spill.percent属性控制），会在本地文件系统中创建一个溢出文件，将该缓冲区中的数据写入这个文件。（map任务中可能会溢出多个本地文件）
2. 在写入磁盘之前，线程首先根据reduce任务的数目将数据划分为相同数目的分区，这里对数据是打上逻辑上的分区，也就是一个reduce任务对应一个分区的数据。这样做是为了避免有些reduce任务分配到大量数据，而有些reduce任务却分到很少数据（数据倾斜），甚至没有分到数据的尴尬局面。其实分区就是对数据进行hash的过程。然后对每个分区中的数据进行排序，如果此时设置了Combiner，将排序后的结果进行Combianer操作，这样做的目的是让尽可能少的数据写入到磁盘。

> 在这里有一点需要注意：从环形缓冲区溢出到本地文件之前，已经对所有的数据进行打分区标记了，然后对待溢出的数据进行排序，这里的排序也是一个局部的排序，相当于在一个文件中有多个分区的数据，对每一个分区的数据进行排序，这样可以保证一个溢出文件中，每一个分区内部的数据有序，在这里也可以设置Combiner操作，这样可以减少网络传输的数据量，对每一个分区中的数据加你单做一次Combiner操作，比如累加操作，这样就大大减少文件中每一个分区中数据的数量。

3. 当map任务输出最后一个记录时（也就是说map任务完成时候），可能会有很多的溢出文件，这时需要将这些文件合并（因为最终在map端输出的只有一个本地文件）。合并的过程中会不断地进行排序（排序是对多个溢出文件中，相同分区合并后的排序操作，这样保证在最终的输出文件中每一个分区中的数据都有序）和combiner操作（多次迭代做combiner操作），目的有两个：
   1. 尽量减少每次写入磁盘的数据量；
   2. 2、尽量减少下一复制阶段网络传输的数据量。最后合并成了一个已分区且已排序的文件。为了减少网络传输的数据量，这里可以将数据压缩，只要将mapred.compress.map.out设置为true就可以。**数据压缩：Gzip、Lzo、snappy。**

4. 将分区中的数据拷贝给相对应的reduce任务。有人可能会问：分区中的数据怎么知道它对应的reduce是哪个呢？其实map任务一直和其父TaskTracker保持联系，而TaskTracker又一直和obTracker保持心跳。所以JobTracker中保存了整个集群中的宏观信息。只要reduce任务向JobTracker获取对应的map输出位置就OK了。

### Shuffle分析

Shuffle的中文意思是“洗牌”，如果我们这样看：一个map产生的数据，结果通过hash过程分区缺分配给了不同的reduce任务，是不是一个对数据洗牌的过程呢？

![1633671588732](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/133949-142955.png)

MapReduce里的Shuffle：描述着数据从map task输出到reduce task输入的这段过程。

![1633673777833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/141618-358709.png)

> 上图中：可以看到在溢写文件这一步，包括三个步骤
>
> 1. 给数据打上逻辑的分区标记，这样相同key的数据就可以分到一个reduce端。
> 2. 然后对文件中的数据做一个排序操作，这一步使用的是快排操作，排序后，文件中各个分区中的数据是有序的。
> 3. 因为在map执行完成后，会有多个本地溢出文件，所以要对多个溢出文件进行葛冰操作，这一步会进行多个排序和合并操作，最后形成一个文件，文件中有多个合并后的分区，分区中的数据都有序。

1. 每个map task都有一个内存缓冲区，存储着map的输出结果，当缓冲区快满的时候需要将缓冲区的数据以一个临时文件的方式存放到磁盘，当整个map task结束后在对磁盘中这个map task产生的所有临时文件做一个合并，生成最终的正式输出文件，然后等待reduce task来拉数据。

2. 在map task执行时，它的输入数据来源于HDFS的block，当然在MapReduce概念中，map task只读取split。split与block对应关系可能是多对一，默认是一对一。在wordcount例子里，假设map的输入数据都是是像“aaa”这样的字符串。

3. 在经过mapper的运行后，我们得知mapper的输出是这样一个key/value对：key是“aaa”，value是数值1。因为当前map端只做加1的操作，在reduce task里采取合并结果集。前面我们知道这个job有3个reduce task。那到底当前的“aaa”究竟该丢给哪个reduce去处理呢？是需要现在做决定的。

4. MapReduce提供Partitioner接口，作用就是根据key或value及reduce的数量来决定当前的输出数据最终应该交由哪个reduce task处理。默认对key hash后再以reduce task数据取模。默认的取模方式只是为了平均reduce的处理能力，如果用户自己对Partitioner有需求，可以定制并设置到job上。

5. 在例子中，“aaa”经过Partition后返回0，也就是这对值应当交由第一个reduce来处理。接下来，需要将数据写入内存缓冲区中，缓冲区的作用是批量收集map结果，减少磁盘IO的影响。我们的key/value对以及Partition的结果都会被写入缓冲区。当然，写入之前，key与value值都会被序列化成字节数组。

6. 内存缓冲区是有大小限制的，默认是100MB。当map task的输出结果很多时，就可能会撑爆内存，所以需要在一定条件下将缓冲区中的数据临时写入磁盘，然后重新利用这块缓冲区。这个从内存往磁盘写数据的过程被称为spill，中文可理解为溢写。溢写是由单独线程来完成，不影响往缓冲区写map结果的线程。溢写线程启动时不应该阻止map的结果输出，所以整个缓冲区有个溢写的比例spill.percent。比例默认是0.8，也就是当缓冲区的数据值已经达到阈值（buffer size * spill percent = 100MB * 0.8 = 80MB），溢写线程启动，锁定这80MB的内存，执行溢写过程。map task的输出结果还可以往剩下的20MB内存中写，互不影响。

7. 当溢写线程启动后，需要对这80MB空间内的key做排序（sort）。排序是MapReduce模型默认的行为，这里的排序也是对序列化的字节做的排序。

8. 因为map task的输出是需要发送到不同的reduce端去，而内存缓冲区没有对将发送到相同reduce端的数据做合并，那么这种合并应该是体现在磁盘文件中的。从官方图上也可以看到写到磁盘中的一些文件是对不同的reduce端的数值做过合并。所以溢写过程一个很重要的细节在于，如果有很多个key/value对需要发送到某个reduce端去，那么需要将这些key/value值拼接到一块，减少与partition相关的索引记录。

9. 在针对每个reduce端而合并数据时，有些数据可能像这样：“aaa”/1，“aaa”/1。对于wordcount例子，只是简单地统计单词出现的次数，如果在同一个map task的结果中有很多像“aaa”一样出现多次的key，我们就应该把它们的值合并到一块，这个过程叫reduce也叫combine。但MapReduce的术语中，reduce是指reduce端执行从多个map task取数据做计算的过程。除reduce外，非正式地合并数据只能算作combine了。其实大家知道的，MapReduce中将Combiner等同于Reducer。

   如果client设置过Combiner，那么现在就是使用Combiner的时候了。将有相同key的key/value对的value加起来，减少溢写到磁盘的数据量。Combiner会优化MapReduce的中间结果，所以它在整个模型中会多次使用。那哪些场景才能使用Combiner呢？从这里分析，Combiner的输出是Reducer的输入，Combiner绝不能改变最终的计算结果。所以从我的想法来看，**Combiner只应该用于那种Reduce的输入key/value与输出key/value类型完全一致，且不影响最终结果的场景。**比如累加，最大值等。Combiner的使用一定得慎重，如果用好，它对job执行效率有帮助，反之会影响reduce的最终结果。

10. 每次溢写会在磁盘上生成一个溢写文件，如果map的输出结果真的很大，有多次这样的溢写发生，磁盘上相应的就会有多个溢写文件存在。当map task真正完成时，内存缓冲区中的数据也全部溢写到磁盘中形成一个溢写文件。最终磁盘中会至少有一个这样的溢写文件存在（如果map的输出结果很少，当map执行完成时，只会产生一个溢写文件），因为最终的文件只有一个，所以需要将这些溢写文件归并到一起，这个过程就叫Merge。Merge是怎样的？如前面的例子，“aaa”从某个map task读取过来时值是5，从另外一个map读取时值是8，因为他们有相同的key，所以要merge成group。

    什么是group：对于“aaa”就是像真阳的：{“aaa”，[5,8,2,…]}，数组中的值就是从不同的溢写文件中读取出来的，然后再把这些值加起来。请注意，因为merge是将多个溢写文件合并到一个文件，所以可能也有相同的key存在，在这个过程中，如果client设置过Combiner，也会使用Combiner来合并相同的key。

    至此，map端的所有工作都已经结束，最终生成的这个文件也存放在TaskManager够得到的某个本地目录中。每个reduce task不断地通过RPC从ResourceManager那获取map task是否完成的信息，如果reduce task得到通知，获知某台TaskManager上的map task执行完成，Shuffle的后半段过程开始启动。

**小结**

- 每个Map任务分配一个缓存.
- MapReduce默认100MB缓存.

- 设置溢写比例0.8。
- 分区默认采用哈希函数。
- 排序是默认的操作。（map端的排序是块排，reduce端的排序是归并排序）
- 排序后可以合并（Combine）。
- 合并不能改变最终结果。

- 在Map任务全部结束之前进行归并。
- 归并得到一个大的文件，放在本地磁盘。
- 文件归并时，如果溢写文件数量大于预定值（默认是3）则可以再次启动Combiner，少于3不需要。
- ResourceManager会一直监测Map任务的执行，并通知Reduce任务来领取数据。

> **合并（Combine）和归并（Merge）的区别：**
>
> 两个键值对<“a”,1>和<“a”,1>，如果合并，会得到<“a”,2>，如果归并，会得到<“a”,<1,1>>

### Reduce端的shuffle过程：

![1633674054015](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/142057-341633.png)

1. reduce会接收到不同map任务传来的数据，并且每个map传来的数据都是有序的。如果reduce端接收的数据量相当小，则直接存储在内存中（缓冲区大小由mapred.job.shuffle.input.buffer.percent属性控制，表示用作此用途的堆空间百分比），如果数据量超过了该缓冲区大小的一定比例（由mapred.job.shuffle.merg.percent决定），则对数据合并后溢写到磁盘中。
2. 随着溢写文件的增多，后台线程会将它们合并成一个更大的有序的文件，这样做是为了给后面的合并节省空间。其实不管在map端还是在reduce端，MapReduce都是反复地执行排序，合并操作，现在终于明白了有些人为什么会说：排序是hadoop的灵魂。（reduce端使用的是归并排序）
3. 合并的过程中会产生许多的中间文件（写入磁盘了），但MapReduce会让写入磁盘的数据尽可能地少，并且最后一次合并的结果并没有写入磁盘，而是直接输入到reduce函数。
4. Reducer的输入文件。不断地merge后，最后会生成一个“最终文件”。为什么加引号？因为这个文件可能存在于磁盘上，也可能存在于内存中。对我们来说，希望它存放于内存中，直接作为Reducer的输入，但默认情况下，这个文件是存放于磁盘中的。当Reducer的输入文件已定，整个Shuffle才最终结束。然后就是Reducer执行，把结果放到HDSF上。

注意：对MapReduce的调优在很大程度上就是对MapReduce Shuffle的性能的调优。

Reduce领取数据先放入缓存，来自不同Map机器，先归并，再合并，写入磁盘。

### MapReduce全局过程

![1633673665761](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/141428-775087.png)

## MapReduce应用程序执行过程

![1633674232750](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/142353-427928.png)

## 小结

![1633674305551](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/142506-424345.png)

![1633674320497](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/08/142521-470385.png)

## Mr程序总结

**过程**

0. 输入文件可以进行压缩，但是需要支持分片操作

1. InputFormat（输入阶段又多种输入）
   1. 默认的是TextInputformat  kv  key偏移量，v :一行内容
   2. 处理小文件CombineTextInputFormat 把多个文件合并到一起统一切片
   3. 按照行进行输入：NLineInputFormat
   4. 自定义输入方式
2. Mapper 
   1. setup()初始化；  
   2. map()用户的业务逻辑； 
   3. clearup() 关闭资源；
3. 分区
   1. 默认分区HashPartitioner ，默认按照key的hash值%numreducetask个数
   2. 自定义分区
4. 排序
   1. 部分排序  每个输出的文件内部有序。
   2. 全排序：  一个reduce ,对所有数据大排序。
   3. 二次排序：  自定义排序范畴， 实现 writableCompare接口， 重写compareTo方法
   4. 流量倒序  按照上行流量 正序

5. Combiner 
   1. 前提：不影响最终的业务逻辑（求和 没问题   求平均值有问题）
   2. 提前聚合map  => 解决数据倾斜的一个方法
6. map阶段的输出可以进行压缩（这里需要解压缩速率高的压缩方式）

7. Reducer
   1. 用户的业务逻辑；
   2. setup()初始化；reduce()用户的业务逻辑； clearup() 关闭资源；
8. OutputFormat（有多种输出方式）
   1. 默认TextOutputFormat  按行输出到文件
   2. 自定义
9. 输出结果压缩：需要压缩比高的方式压缩，因为需要输出到磁盘。

