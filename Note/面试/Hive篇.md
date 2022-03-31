## Hive篇

### 什么是Hive，为什么要使用Hive，你如何理解Hive

Hive是一个基于Hadoop的数据仓库工具，可以将一些结构化的文件映射为结构化的表，并且提供了类似sql的查询功能。**Hive的本质也是将hql转化为mapreduce任务进行计算。**

平时我们都使用hadoop框架进行数据的计算，但是上手使用hadoop编写mr程序需要一定的编程基础，所以为了更好的开发出我们的mr程序，就有了hive工具，hlq学习起来的成本很低，我们可以直接写hql查询来查看数据，hive底层会将hql语句自动转换为mr程序。

> 所以说HIve是一个数据仓库工具，提供了类sql的语句hql可以转化为mr程序操作hdfs上面存储的数据。

### Hive架构

![1633138397495](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/093319-612218.png)

**用户接口**

- cui是自带的命令行接口
- JDBC/ODBC是数据库操作
- web ui使用http协议通过浏览器操作

> 用户接口主要有三个：
>
> - CLI，
>
> - Client 
>
> - WUI
>
> 其中最常用的是CLI，Cli启动的时候，会同时启动一个Hive副本。
>
> Client是Hive的客户端，用户连接至Hive Server。在启动 Client模式的时候，需要指出Hive Server所在节点，并且在该节点启动Hive Server。 
>
> WUI是通过浏览器访问Hive。

**元数据**

- 默认使用derby数据库，但是只可以建立一个链接
- 生产中一般使用mysql数据库，可以建立多个链接

**驱动器**

- 解释器：将类sql语句转换为hql语句
- 编译器：将hql语句转换为job作业
- 优化器：优化job作业
- 执行器：yarn调用job在hadoop执行

### Hive在生产环境中的架构

![1633139076675](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/094652-261004.png)

Hive是一个数据仓库的工具，方便程序员们编写mapreduce程序，hql转换mapreduce程序是hive框架底层自动实现的

- mysql中产生的业务数据使用sqoop导入到Hive中

- 用户行为数据通过flume采集到hive中，
- 在hive底层，数去全部存储在hdfs上面，然后通过hive工具去分析hdfs上面的数据。

### 画出hive的架构图，解释每一部分的作用

![1633139402469](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/095004-113468.png)

**用户接口**

- cui是自带的命令行接口
- JDBC/ODBC是数据库操作
- web ui使用http协议通过浏览器操作

> 用户接口主要有三个：
>
> - CLI，
>
> - Client 
>
> - WUI
>
> 其中最常用的是CLI，Cli启动的时候，会同时启动一个Hive副本。
>
> Client是Hive的客户端，用户连接至Hive Server。在启动 Client模式的时候，需要指出Hive Server所在节点，并且在该节点启动Hive Server。 
>
> WUI是通过浏览器访问Hive。

**元数据**

- 默认使用derby数据库，但是只可以建立一个链接
- 生产中一般使用mysql数据库，可以建立多个链接

**驱动器**

- 解释器：将类sql语句转换为hql语句
- 编译器：将hql语句转换为job作业
- 优化器：优化job作业
- 执行器：yarn调用job在hadoop执行

### Hive优缺点

**优点**

- 简单，非常容易上手，会写sql就能开发出mr程序，**实现分布式计算**。因为其底层使用的是mapreduce计算模型，而这个计算模型是分布式计算的。
- 是一个数据仓库的工具，可以处理大量的数据，但是对于小数据没有优势，离线分析处理，延迟很高。
- 可以支持用户自定义的函数UDF/UDTF。

**缺点**

- hql表达能力有限，迭代式计算无法表达。
- 数据挖掘方面不擅长，实现不了挖掘算法效率高的算法。
- HIVE效率比较低，延迟比较高，因为底层使用的是hadoop。
- hive调优比较困难，**粒度比较粗**
- 不能修改数据，适合读多写少的情景。

### Hql转化为mapreduce程序的过程

- 编译器首先完成hql的词法和语法的解析，然后将hql转化为抽象语法树AST。
- 遍历抽象抽象语法树，抽象出查询的基本组成单位，查询块，生成基本的查询块。
- 接着遍历生成的基本查询块，然后将遍历出来的结果翻译为执行操作树。
- 生成完操作树后会在逻辑层去优化，会对生成的操作树进行优化变换，去合并不必要的reduceSink操作，目的就是为了减少shuffle的数据量，
- 优化之后会再次遍历优化之后的操作树，翻译为mr任务，这一次翻译出来的任务就是物理的mr任务，物理层的优化器会再次进行mr任务的一个转换，再次从物理层进行优化，最后生成一个执行计划。
- 最后提交执行计划执行。

> Hql-解释器-->抽象语法树-编译器-->查询块-编译器--逻辑优化器>操作树-物理优化器-->mr任务-执行器-->执行计划

### Hive的两张表关联，使用MapReduce怎么实现？

hive中的join其实就是mapreduce中的join操作，分为两种，一种是map join，一种是reduce join。

- Common Join(Reduce阶段完成join)
- Map Join(Map阶段完成join)

#### Common Join

如果没开启hive.auto.convert.join=true或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join,在Reduce阶段完成join。并且整个过程包含Map、Shuffle、Reduce阶段。

**Map阶段**

1. 读取表的数据，Map输出时候以 Join on 条件中的列为key，如果Join有多个关联键，则以这些关联键的组合作为key;
2. Map输出的 value 为 join 之后需要输出或者作为条件的列；同时在value中还会包含表的 Tag 信息，用于标明此value对应的表；按照key进行排序

**Shuffle阶段**

1. 根据key取哈希值,并将key/value按照哈希值分发到不同的reduce中

**Reduce阶段**

1. 根据key的值完成join操作，并且通过Tag来识别不同表中的数据。在合并过程中，把表编号扔掉

**下面看一个案例**

![1639220618669](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/190339-242036.png)

#### Hive map join

1. MapJoin通常用于一个很小的表和一个大表进行join的场景，具体小表有多小，由参数**hive.mapjoin.smalltable.filesize**来决定，该参数表示小表的总大小，默认值为25000000字节，即25M。
2. Hive0.7之前，需要使用hint提示 + mapjoin(table) 才会执行MapJoin,否则执行Common Join，但在0.7版本之后，默认自动会转换Map Join，由参数**hive.auto.convert.join**来控制，默认为true.
3. 仍然以9.1中的HQL来说吧，假设a表为一张大表，b为小表，并且hive.auto.convert.join=true,那么Hive在执行时候会自动转化为MapJoin。

![1639220718733](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/190520-412132.png)

如图中的流程，首先是Task A，它是一个Local  Task（在客户端本地执行的Task），负责扫描小表b的数据，将其转换成一个HashTable的数据结构，并写入本地的文件中，之后将该文件加载到DistributeCache中，该HashTable的数据结构可以抽象为：

![1639220767516](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/190641-30926.png)

- 接下来是Task B，该任务是一个没有Reduce的MR，启动MapTasks扫描大表a,在Map阶段，根据a的每一条记录去和DistributeCache中b表对应的HashTable关联，并直接输出结果。
- 由于MapJoin没有Reduce，所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件。

### 请谈一下Hive的特点，Hive和RDBMS有什么异同？

hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供完整的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析，**但是Hive不支持实时查询，Hive的延迟比较高**。

**Hive与关系型数据库的区别**

![1632707644904](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/27/095406-581410.png)

![1633139502521](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/095144-999240.png)

### 请说明hive中 Sort By，Order By，Cluster By，Distrbute By各代表什么意思？

- Order by：会对输入做**全局排序**，因此只有一个reducer（多个reducer无法保证全局有序）。只有一个reducer，会导致当输入规模较大时，需要较长的计算时间。
- Sort by：不是全局排序，其在数据进入reducer 前完成排序.因此，如果用 sort by 进行排序，并且设置 mapred.reduce.tasks>1，也就是map任务大于1个， 则 **sort by 只保证每个 reducer 的输出有序，不保证全局有序，保证每一个reduce内局部有序**。
- Distribute by：**按照指定的字段对数据进行划分输出到不同的reduce中，可能是按照字段的hash值进行划分。**
- Cluster by：**除了具有 distribute by 的功能外还兼具 sort by 的功能。**但是分区字段和排序字段需要相同。

**小结**

![1633139753219](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/095554-608131.png)

### Hql中常用的窗口函数和排序函数有哪些

![1633139907243](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/095828-317055.png)

![1633140160037](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/100241-101850.png)

#### 详解窗口函数

![1633140465134](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/100745-431581.png)

上面代表分完区的数据：

current row代表的是当前行，从当前行，可以选择前n行和后n行，也可以选择到分区的上边界或者分区的后边界。

### 写出Hive中split、coalesce及collect_list函数的用法（可举例）？

- split将字符串转化为数组，即：split('a,b,c,d' , ',') ==> ["a","b","c","d"]。
- coalesce(T v1, T v2, …) 返回参数中的第一个非空值；如果所有值都为 NULL，那么返回NULL。
- collect_list列出该字段所有的值，**不去重** => select collect_list(id) from table。

### Hive有哪些方式保存元数据，各有哪些特点？

Hive支持三种不同的元存储服务器，分别为：内嵌式元存储服务器（derby数据库）、本地元存储服务器（mysql数据库）、远程元存储服务器，每种存储方式使用不同的配置参数。

1. 内嵌式元存储主要用于单元测试，在该模式下每次只有一个进程可以连接到元存储，Derby是内嵌式元存储的默认数据库。自带的存储数据库，每一次只可以建立一个会话链接。
2. 在本地模式下，每个Hive客户端都会打开到数据存储的连接并在该连接上请求SQL查询。通常我们使用的是Mysql数据库存储元数据信息，而使用HDFS存储数据信息。
3. 在远程模式下，所有的Hive客户端都将打开一个到元数据服务器的连接，该服务器依次查询元数据，元数据服务器和客户端之间使用Thrift协议通信。

> 在实际的开发中，我们一般将数据存储在mysql数据库中。

### Hive内部表和外部表的区别？

未被 external 修饰的是内部表，被 external 修饰的为外部表。

- 创建表时：创建内部表时，会将数据移动到数据仓库指向的路径，位置是`hive.metastore.warehouse.dir`（默认：`/user/hive/warehouse`），若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。
- 删除表时：在删除表的时候，内部表的元数据和数据会被一起删除， 而外部表只删除元数据，不删除数据。这样外部表相对来说更加安全些，数据组织也更加灵活，方便共享源数据，内部表的数据属于hive管理，而外部表的数据属于hdfs管理。

### Hive 有索引吗

Hive 支持索引（3.0 版本之前），但是 Hive 的索引与关系型数据库中的索引并不相同，比如，Hive 不支持**主键或者外键**。并且 Hive 索引提供的功能很有限，效率也并不高，因此 Hive 索引很少使用。

- **索引适用的场景：**

适用于不更新的**静态字段**。以免总是重建索引数据。每次建立、更新数据后，都要重建索引以构建索引表。

- **Hive 索引的机制如下：**

hive 在指定列上建立索引，会产生一张**索引表**（Hive 的一张物理表），里面的字段包括：**索引列的值**、该值对应的 HDFS 文件路径、该值在文件中的偏移量。

Hive 0.8 版本后引入 bitmap 索引处理器，这个处理器适用于去重后，值较少的列（例如，某字段的取值只可能是几个枚举值）因为索引是用**空间换时间**，索引列的取值过多会导致建立 bitmap 索引表过大。

**注意**：Hive 中每次有数据时需要及时更新索引，相当于重建一个新表，否则会影响数据查询的效率和准确性，**Hive 官方文档已经明确表示 Hive 的索引不推荐被使用，在新版本的 Hive 中已经被废弃了**。

> **扩展**：Hive 是在 0.7 版本之后支持索引的，在 0.8 版本后引入 bitmap 索引处理器，在 3.0 版本开始移除索引的功能，取而代之的是 2.3 版本开始的物化视图，自动重写的物化视图替代了索引的功能。

### 数据建模用的哪些模型？

> 介绍一下事实表和维度表：
>
> 事实表，即为事实数据表的简称。主要特点是含有大量的实时性数据，并且这些数据是可以汇总，并被记录的，通常一条具体的购买或者行为就是一条事实数据，事实数据数据量很大，每一条都有新增数据。
>
> 记录既定事实数据的数据集，比如某时某地某物发生某事，记录为一条数据，一般是原始数据；例如2020年1月1日购物平台袜子销售金额为1000元，这条数据也可以继续细分，如2020年1月1日1时1分1秒20元卖出一双A牌B型号袜子。
>
> 维度表可以看作是用户来分析数据的窗口，维度表中包含事实数据表中事实记录的特性，有些特性提供描述性信息，有些特性指定如何汇总事实数据表数据，以便为分析者提供有用的信息，维度表包含帮助汇总数据的特性的层次结构。
>
> 维度表由主键和属性组成，一个维度表只能有一个主键，且主键的值不能重复，一个维度表可以包含多个属性，且这些属性可以进行扩展。一般我们用DIM来表示维度表。
>
> 维度表是用来描述我们的实体信息的表，通常很大。

**星型模型**

![1632835904327](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/28/213147-392488.png)

**星形模式**

星形模式(Star Schema)是最常用的维度建模方式。星型模式是以**事实表**为中心，所有的维度表直接连接在事实表上，像星星一样。星形模式的维度建模由一个事实表和一组维表成，且具有以下特点：

- 维度表只和事实表关联，维表之间没有关联；
- 每个维表主键为单列，且该主键放置在事实表中，作为两边连接的外键；
- 以事实表为核心，维表围绕核心呈星形分布。

**雪花模型**

![1632836011529](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/28/213333-529476.png)

雪花模式(Snowflake Schema)是对星形模式的扩展。**雪花模式的维度表可以拥有其他维度表的**，虽然这种模型相比星型更规范一些，但是由于这种模型不太容易理解，维护成本比较高，而且性能方面需要关联多层维表，性能比星型模型要低。

**星座模型**

![1632836060943](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202109/28/213423-5483.png)

星座模式是星型模式延伸而来，星型模式是**基于一张事实表**的，而**星座模式是基于多张事实表的，而且共享维度信息**。前面介绍的两种维度建模方法都是多维表对应单事实表，但在很多时候维度空间内的事实表不止一个，而一个维表也可能被多个事实表用到。在业务发展后期，绝大部分维度建模都采用的是星座模式。

#### 模型对比

**星型模型**

星形模型中有一张事实表，以及零个或多个维度表，事实表与维度表通过主键外键相关联，维度表之间没有关联，当所有维表都直接连接到“ 事实表”上时，整个图解就像星星一样，故将该模型称为星型模型。

星形模型是最简单，也是最常用的模型。由于星形模型只有一张大表，因此它相比于其他模型更适合于大数据处理。其他模型可以通过一定的转换，变为星形模型。

星型架构是一种非正规化的结构，多维数据集的每一个维度都直接与事实表相连接，不存在渐变维度，所以数据有一定的冗余，如在地域维度表中，存在国家 A 省B 的城市 C 以及国家 A 省 B 的城市 D 两条记录，那么国家 A 和省 B 的信息分别存储了两次，即存在冗余。

> 星型模型的缺点是存在一定程度的数据冗余。因为其维表只有一个层级，有些信息被存储了多次。比如一张包含国家、省份、地市三列的维表，国家列会有很多重复的信息。

比如：销售数据仓库中的星型模型

![1639221924440](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/192525-242384.png)

**雪花模型**

当有一个或多个维表没有直接连接到事实表上，而是通过其他维表连接到事实表上时，其图解就像多个雪花连接在一起，故称雪花模型。

雪花模型是对星型模型的扩展。它对星型模型的维表进一步层次化，原有的各维表可能被扩展为小的维度表，形成一些局部的 " 层次 " 区域，这些被分解的表都连接到主维度表而不是事实表。如图，将地域维表又分解为国家，省份，城市等维表。

它的优点是 : 通过最大限度地减少数据存储量以及联合较小的维表来改善查询性能。雪花型结构去除了数据冗余。

> 其优点是通过最大限度地减少数据存储量以及联合较小的维表来改善查询性能，避免了数据冗余。其缺点是增加了主键-外键关联的几率，导致查询效率低于星型模型，并且不利于开发。

![1639221955942](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/192556-297391.png)

**星座模型**

星座模型是由**星型模型**延伸而来，星型模型是基于一张事实表而星座模式是基于多张事实表，并且共享维度表信息，这种模型往往应用于数据关系比星型模型和雪花模型更复杂的场合。星座模型需要多个事实表共享维度表，因而可以视为星形模型的集合，故亦被称为星系模型

![1639221787072](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/192308-224896.png)

**星型模型和雪花模型对比**

1. 星型模型因为**数据的冗余所以很多统计查询不需要做外部的连接**，因此一般情况下效率比雪花模型要高。
2. 星型模型不用考虑很多正规化的因素，设计和实现都比较简单。
3. 雪花模型由于去除了冗余，有些统计就需要通过表的连接才能产生，所以效率不一定有星型模型高。
4. 正规化也是一种比较复杂的过程，相应的数据库结构设计、数据的ETL、以及后期的维护都要复杂一些。因此在冗余可以接受的前提下，实际运用中星型模型使用更多，也更有效率。

![1639222077208](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/192758-982154.png)

![1639222410547](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/193331-405712.png)

> 从上面的对比中，我们也发现，有时候规范化和效率是一组矛盾。一般我们会采取牺牲空间（规范化）来换取好的性能，把尽可能多的维度信息存在一张“大表”里面是最快的。通常会视情况而定，采取折中的策略。
>
> 数据仓库更适合使用星型模型来构建底层数据 hive 表，通过数据冗余来减少查询次数以提高查询效率。雪花模型在关系型数据库中（MySQL/Oracle）更加常见

### 为什么要对数据仓库进行分层？

- 用空间换时间，通过大量的预处理来提升应用系统的用户体验（效率），因此数据仓库会存在大量冗余的数据。
- 如果不分层的话，如果源业务系统的业务规则发生变化将会影响整个数据清洗过程，工作量巨大。
- 通过数据分层管理可以简化数据清洗的过程，因为把原来一步的工作分到了多个步骤去完成，相当于把一个复杂的工作拆成了多个简单的工作，把一个大的黑盒变成了一个白盒，每一层的处理逻辑都相对简单和容易理解，这样我们比较容易保证每一个步骤的正确性，当数据发生错误的时候，往往我们只需要局部调整某个步骤即可。

### Hive中常见的函数

**聚合函数**（多进一出）

- count()
- sum()
- avg()

**数学函数**

- around（）四舍五入函数

**集合函数**

- size()查询集合长度
- mapkeys()查询map中所有的key

**类型转换函数**

- cast()

**日期函数**

- unix_timestamp()
- time_add()

**条件函数**

- isNull()

**字符串函数**

- length()
- subString()

**表生成函数（列转行）**

- explode()

**开窗函数**

- rank()
- rownumber()

### Hive的函数：UDF、UDAF、UDTF的区别？

- UDF：用户自定义函数，单行进入，单行输出（一进一出）
- UDAF：多行进入，单行输出（多进一出）
- UDTF：单行输入，多行输出（一进多出）

**讲一下自定义用户函数的步骤**

![1633140847597](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/101411-224626.png)

**自定义UDF函数**

1. 继承`org.apache.hadoop.hive.ql.exec.UDF`
2. 需要实现`evaluate`函数；`evaluate`函数支持重载，不可以修改函数名；
3. 在`hive`的命令行窗口创建函数(可以创建临时函数和永久的函数)
4. 如果自定义函数要输入多个参数怎么办，就重载`EVALUTE()`方法，每个方法中的参数不同即可，也可以用可变形参的方法，但是局限性在于可变形参的类型必须一样。

**自定义UDTF函数**

1. 自定义类继承`genericUDTF`类。
2. 实现其三个方法：`initialize`,`process`,`close`等三个方法。
   1. `initialize()`：进行初始化操作，定义输出数据类型
   2. `process`:处理业务逻辑
   3. `close`：关闭资源
3. `initialize`函数里面声明输出参数的名称和类型，`process`里面写具体处理的业务逻辑，`close`关闭流操作。

### 使用过 Hive 解析 JSON 串吗

**Hive 处理 json 数据总体来说有两个方向的路走**：

1. 将 json 以字符串的方式整个入 Hive 表，然后通过使用 UDF 函数解析已经导入到 hive 中的数据，比如使用LATERAL VIEW json_tuple的方法，获取所需要的列名，本离线数据仓库中使用的是这种方法。
2. 在导入之前将 json 拆成各个字段，导入 Hive 表的数据是已经解析过的。这将需要使用第三方的 SerDe，或者阿里巴巴的fastjson工具库。

### 所有的Hive任务都会有MapReduce的执行吗？

hive 0.10.0为了执行效率考虑，简单的查询，就是只是select，不带count,sum,group by这样的，都不走map/reduce，直接读取hdfs文件进行filter过滤。这样做的好处就是不新开mr任务，执行效率要提高不少，但是不好的地方就是用户界面不友好，有时候数据量大还是要等很长时间，但是又没有任何返回。

### 说说对Hive桶表的理解？

桶表是对数据`某个字段`进行哈希取值，然后放到不同文件中存储。

数据加载到桶表时，会对字段取hash值，然后与桶的数量取模。把数据放到对应的文件中。物理上，每个桶就是表(或分区）目录里的一个文件，一个作业产生的桶(输出文件)和reduce任务个数相同。桶表专门用于抽样查询，是很专业性的，不是日常用来存储数据的表，需要抽样查询时，才创建和使用桶表。

分桶则是指定分桶表的某一列，让该列数据按照哈希取模的方式随机、均匀的分发到各个桶文件中。因为分桶操作需要根据某一列具体数据来进行哈希取模操作，故指定的分桶列必须基于表中的某一列（字段）。分桶改变了数据的存储方式，它会把哈希取模相同或者在某一个区间的数据行放在同一个桶文件中。如此一来便可以提高查询效率。如果我们需要对两张在同一个列上进行了分桶操作的表进行JOIN操作的时候，只需要对保存相同列值的通进行JOIN操作即可。

> 这里要和分区分开，分区产生的是文件夹目录，而分桶产生的是文件。

### Hive底层与数据库交互原理？

Hive 的查询功能是由 HDFS 和 MapReduce结合起来实现的，对于大规模数据查询还是不建议在 hive 中，因为过大数据量会造成查询十分缓慢。

Hive 与 MySQL的关系：只是借用 MySQL来存储 hive 中的表的元数据信息，称为 metastore（元数据信息），而真正的数据存储在Hdfs上面。

### Hive本地模式

大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时Hive的输入数据量是非常小的。在这种情况下，为查询触发执行任务时消耗可能会比实际job的执行时间要多的多，因为一旦触发执行JOB,必定会涉及集群资源的管理与调度，很费性能和耗时。对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。

用户可以通过设置hive.exec.mode.local.auto的值为true，来让Hive在适当的时候自动启动这个优化。

> hive的本地模式也是建立在hadoop基础之上的。

### Hive 中的压缩格式TextFile、SequenceFile，RCfile 、ORCfile各有什么区别？

**TextFile**

默认格式，**存储方式为行存储，数据不做压缩，磁盘开销大，数据解析开销大**。可结合Gzip、Bzip2使用(系统自动检查，执行查询时自动解压)，但使用这种方式，压缩后的文件不支持split，Hive不会对数据进行切分，从而无法对数据进行并行操作。并且在反序列化过程中，必须逐个字符判断是不是分隔符和行结束符，因此反序列化开销会比SequenceFile高几十倍。

**SequenceFile**

SequenceFile是Hadoop API提供的一种二进制文件支持，**存储方式为行存储，其具有使用方便、可分割、可压缩的特点**。

SequenceFile支持三种压缩选择：`NONE`，`RECORD`，`BLOCK`。Record压缩率低，**一般建议使用BLOCK压缩**。

优势是文件和hadoop api中的MapFile是相互兼容的

**RCFile**

存储方式：**数据按行分块，每块按列存储**。结合了行存储和列存储的优点：

首先，RCFile 保证同一行的数据位于同一节点，因此元组重构的开销很低；

其次，像列存储一样，RCFile 能够利用列维度的数据压缩，并且能跳过不必要的列读取；

**ORCFile**

存储方式：数据按行分块 每块按照列存储。

压缩快、快速列存取。

效率比rcfile高，是rcfile的改良版本。

> 小结：
>
> **相比TEXTFILE和SEQUENCEFILE，RCFILE由于列式存储方式，数据加载时性能消耗较大，但是具有较好的压缩比和查询响应**。
>
> **数据仓库的特点是一次写入、多次读取，因此，整体来看，RCFILE相比其余两种格式具有较明显的优势**。

### Hive数据倾斜

在hadoop中数据倾斜的体现，就是有的任务卡在了99%之后执行不成功。

数据倾斜分为map端倾斜和reduce端倾斜。

#### 什么操作会导致发生数据倾斜

![1639223112413](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/11/194513-326850.png)

- map输出数据按key Hash的分配到reduce中
- 由于key分布不均匀
- 业务数据本身的特性
- 建表时考虑不周

> 分组 注：group by 优于distinct group
> 情形：group by 维度过小，某值的数量过多
> 后果：处理某值的reduce非常耗时
>
> 去重 distinct count(distinct xx)
> 情形：某特殊值过多
> 后果：处理此特殊值的reduce耗时
> 连接 join
> 情形1：其中一个表较小，但是key集中
> 后果1：分发到某一个或几个Reduce上的数据远高于[平均值](https://www.zhihu.com/search?q=%E5%B9%B3%E5%9D%87%E5%80%BC&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A248861446%7D)

如何避免：对于key为空产生的数据倾斜，可以对其赋予一个随机值。

#### 数据倾斜解决方案

##### 参数调节：

```java
hive.map.aggr = true //首先在map端进行聚合
hive.groupby.skewindata=true--开启负载均衡
```

有数据倾斜的时候进行负载均衡，当选项设定位true,生成的查询计划会有两个MR Job。

- 第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；
- 第二个MR Job再根据预处理的数据结果按照Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个Reduce中），最后完成最终的聚合操作。

##### SQL 语句调节

**如何Join：**

- 关于驱动表的选取，选用join key分布最均匀的表作为驱动表。
- 做好列裁剪和filter操作，以达到两表做join的时候，数据量相对变小的效果。
- 当出现小文件过多，需要合并小文件。可以通过set hive.merge.mapfiles=true来解决。

**大小表join**

大小表Join：使用map join让小的维度表（1000 条以下的记录条数）先进内存。在map端完成reduce。

小表在join左侧，大表在右侧，或使用mapjoin 将小表加载到内存中。然后再对比较大的表进行map操作。也就是小表驱动大表。

**大表join大表**

遇到需要进行join的但是关联字段有数据为null，如表一的id需要和表二的id进行关联，null值的reduce就会落到一个节点上

- 解决方法1：子查询中过滤掉null值，id为空的不参与关联。

- 解决方法2：用case when给空值分配随机的key值（字符串+rand()）

> 大表Join大表：把空值的key变成一个字符串加上随机数，把倾斜的数据分到不同的reduce上，由于null 值关联不上，处理后并不影响最终结果。也就是执行两个mr任务。
>
> 在加个combiner函数，加上combiner相当于提前进行reduce,就会把一个mapper中的相同key进行了聚合，减少shuffle过程中数据量，以及reduce端的计算量。这种方法可以有效的缓解数据倾斜问题，但是如果导致数据倾斜的key 大量分布在不同的mapper的时候，这种方法就不是很有效了。

**count （distinct）大量相同特殊值**

count distinct大量相同特殊值:count distinct 时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在最后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union。

### 大表Join大表

**如果空key可以过滤掉**

空KEY过滤，有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。例如key对应的字段为空。

**空key不能够过滤掉**

1. 空key转换，有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。这种方式是局部聚合+全局聚合，使用两个mr任务来聚合结果。
2. 在sql结尾使用distribute by rand() ,将数据均匀的分发到多个reducer上面。

#### 数据倾斜总结

- 原因一：关联字段中有空值或者脏数据，

- 原因二：关联字段中key都为有效值是，某些key量大，造成reduce计算量大

- 原因三：由于group by 中某个key值较大引起的

**解决办法**：

原因一：

如果不能过滤，则将这部分给key赋予随机值，交给不同的reduce处理

例如 cast(ceiling(rand() * -65535) as string) end )

原因二：

 1、如果大小表join，则使用mapjoin，将小表放入缓存中，广播的方式分发到不同的map中，与大表在map端进行join。

2、参数设置，提高reduce个数

### Group By

默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

**开启Map端聚合参数设置**     

1. 是否在Map端进行聚合，默认为True，`hive.map.aggr = true`   
2. 在Map端进行聚合操作的条目数目`hive.groupby.mapaggr.checkinterval = 100000`   
3. 有数据倾斜的时候进行负载均衡（默认是false）`hive.groupby.skewindata = true`，**当选项设定为 true，生成的查询计划会有两个MR Job**。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是**相同的Group By Key有可能被分发到不同的Reduce中**，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

### Hive 小文件过多怎么解决

![1633154997892](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/140959-627050.png)

第二个表示：在map结束的时候合并小文件，默认是true

第三个参数表示在mr结束的时候合并小文件，默认的参数是false

最后一个参数表示设置文件的最小大小，当文件小于这个值的时候，就会启动一个线程对文件进行合并操作

1. 使用 hive 自带的 concatenate 命令，自动合并小文件

```sql
-- 用法
--对于非分区表
alter table A concatenate;

--对于分区表
alter table B partition(day=20201224) concatenate;
```

> 注意：
>
> 1、concatenate 命令只支持 RCFILE 和 ORC 文件类型。
>
> 2、使用 concatenate 命令合并小文件时不能指定合并后的文件数量，但可以多次执行该命令。
>
> 3、当多次使用 concatenate 后文件数量不在变化，这个跟参数 mapreduce.input.fileinputformat.split.minsize=256mb 的设置有关，可设定每个文件的最小 size。

2. 调整参数减少 Map 数量

设置 map 输入合并小文件的相关参数（执行 Map 前进行小文件合并）：在 mapper 中将多个文件合成一个 split 作为输入（CombineHiveInputFormat底层是 Hadoop 的CombineFileInputFormat方法）：

```sql
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; -- 默认

-- 每个 Map 最大输入大小（这个值决定了合并后文件的数量）：
set mapred.max.split.size=256000000;   -- 256M

-- 一个节点上 split 的至少大小（这个值决定了多个 DataNode 上的文件是否需要合并）：
set mapred.min.split.size.per.node=100000000;  -- 100M

-- 一个交换机下 split 的至少大小(这个值决定了多个交换机上的文件是否需要合并)：
set mapred.min.split.size.per.rack=100000000;  -- 100M

```

3. 减少 Reduce 的数量

reduce 的个数决定了输出的文件的个数，所以可以调整 reduce 的个数控制 hive 表的文件数量。

hive 中的分区函数 distribute by 正好是控制 MR 中 partition 分区的，可以通过设置 reduce 的数量，结合分区函数让数据均衡的进入每个 reduce 即可：

```sql
#设置reduce的数量有两种方式，第一种是直接设置reduce个数
set mapreduce.job.reduces=10;

#第二种是设置每个reduce的大小，Hive会根据数据总大小猜测确定一个reduce个数
set hive.exec.reducers.bytes.per.reducer=5120000000; -- 默认是1G，设置为5G

#执行以下语句，将数据均衡的分配到reduce中
set mapreduce.job.reduces=10;
insert overwrite table A partition(dt)
select * from B
distribute by rand();
```

对于上述语句解释：如设置 reduce 数量为 10，使用 rand()， 随机生成一个数 x % 10 ，这样数据就会随机进入 reduce 中，防止出现有的文件过大或过小。

4. 使用 hadoop 的 archive 将小文件归档

Hadoop Archive 简称 HAR，是一个高效地将小文件放入 HDFS 块中的文件存档工具，它能够将多个小文件打包成一个 HAR 文件，这样在减少 namenode 内存使用的同时，仍然允许对文件进行透明的访问。

```sql
#用来控制归档是否可用
set hive.archive.enabled=true;
#通知Hive在创建归档时是否可以设置父目录
set hive.archive.har.parentdir.settable=true;
#控制需要归档文件的大小
set har.partfile.size=1099511627776;

使用以下命令进行归档：
ALTER TABLE A ARCHIVE PARTITION(dt='2021-05-07', hr='12');

对已归档的分区恢复为原文件：
ALTER TABLE A UNARCHIVE PARTITION(dt='2021-05-07', hr='12');

```

> 注意:**归档的分区可以查看不能 insert overwrite，必须先 unarchive**

### 源码角度理解hive执行原理

![1633142184004](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/103625-164480.png)

1. 提交一个执行计划。
2. 在driver端生成一个执行计划，
3. 编译器去元数据库查询执行计划
4. 元数据库返元数据信息
5. 编译器返回执行计划给驱动器
6. 编译器发送执行计划命令给执行引擎开始执行执行计划
7. 执行引擎提交计划到集群中去执行
8. 执行引擎返回执行的结果到驱动器
9. 驱动器返回结果给用户

### Hive的存储格式和压缩格式

![1633142813335](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/104654-665346.png)

### Hive中的分析函数

![1633146131360](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/114212-897226.png)

![1633146654696](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/115056-678547.png)

### 说说你对分区表和分桶表的理解

![1633147155726](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/115917-71055.png)

### Hive调优

![1633141298756](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/102140-147107.png)

![1633141656384](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/102738-108120.png)

![1633141854445](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/02/103056-117747.png)

#### Fetch抓取（Hive可以避免进行MapReduce）

 Hive中对某些情况的查询`可以不必使用MapReduce计算`。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。  

在`hive-default.xml.template`文件中`hive.fetch.task.conversion`默认是more，老版本hive默认是`minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce`。

```xml
<property>
    <name>hive.fetch.task.conversion</name>
    <value>more</value>
    <description>
      Expects one of [none, minimal, more].
      Some select queries can be converted to single FETCH task minimizing latency.
      Currently the query should be single sourced not having any subquery and should not have
      any aggregations or distincts (which incurs RS), lateral views and joins.
      0. none : disable hive.fetch.task.conversion
    </description>
  </property>
```

**案例**

把`hive.fetch.task.conversion`设置成`none`，然后执行查询语句，都会`执行mapreduce`程序。

```sql
hive (default)> set hive.fetch.task.conversion=none;
hive (default)> select * from score;
hive (default)> select s_score from score;
hive (default)> select s_score from score limit 3;
```

把`hive.fetch.task.conversion`设置成`more`，然后执行查询语句，如下查询方式都`不会执行mapreduce程序`。

#### 本地模式

大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时`Hive的输入数据量是非常小`的。在这种情况下，为查询触发执行任务时消耗可能会比实际job的执行时间要多的多。对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。`对于小数据集，使用本地模式执行时间可以明显被缩短`。   

用户可以通过设置`hive.exec.mode.local.auto`的值为`true`，来让Hive在适当的时候`自动启动这个优化`。

```java
set hive.exec.mode.local.auto=true;  --开启本地MapReduce
-- 设置local mr的最大输入数据量，当输入数据量小于这个值时采用local  mr的方式，默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=51234560;
-- 设置local mr的最大输入文件个数，当输入文件个数小于这个值时采用local mr的方式，默认为4
set hive.exec.mode.local.auto.input.files.max=10
```

开启本地模式查询：

```sql
set hive.exec.mode.local.auto=true; 
select * from score cluster by s_id;
```

#### group By

 默认情况下，Map阶段同一Key数据分发给一个reduce，`当一个key数据过大时就倾斜了`。         

并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以`先在Map端进行部分聚合，最后在Reduce端得出最终结果`。

**开启Map端聚合参数设置**

```java
--  是否在Map端进行聚合，默认为True
set hive.map.aggr = true;
--在Map端进行聚合操作的条目数目
set hive.groupby.mapaggr.checkinterval = 100000;
-- 有数据倾斜的时候进行负载均衡（默认是false）
set hive.groupby.skewindata = true;
```

当负载均衡选项设定为 true，生成的查询计划会有两个MR Job。

- 第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；
- 第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

#### Count(distinct)；

数据量小的时候无所谓，数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换，也就是在使用Count进行统计的时候，先对数据进行分组然后在统计。

```sql
-- 直接对数据进行统计
SELECT count(DISTINCT id) FROM bigtable;
-- 先分组然后在查询
SELECT count(id) FROM (SELECT id FROM bigtable GROUP BY id) a;
```

但是这样做的话，会多执行一个分组的任务，但是对于数据量特别大的情况，是非常值得的。

#### 笛卡尔积

尽量避免笛卡尔积，即避免join的时候不加on条件，或者无效的on条件，Hive只能使用1个reducer来完成笛卡尔积。不论在什么情况下都应该绝对的避免笛卡尔积。

#### 使用分区剪裁、列剪裁

在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。         

在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤（要尽量先进行where过滤，然后在进行关联操作），比如：

**案例**

```sql
create table ori(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';
create table bigtable(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

-- 先关联再where
select a.id FROM bigtable a left join ori o on a.id=o.id where o.id<=10;

-- 先过滤然后在进行关联
select a.id FROM bigtable a left join ori o  on (o.id<=10 and a.id=o.id);

-- 子查询
select a.id from bigtable a join (select  id from ori o where o.id<=10) o on o.id=a.id;
```

#### 动态分区调整

关系型数据库中，对分区表Insert数据时候，数据库自动会根据分区字段的值，将数据插入到相应的分区中，Hive中也提供了类似的机制，即动态分区(Dynamic Partition)，只不过，使用Hive的动态分区，需要进行相应的配置。  

以第一个表的分区规则，来对应第二个表的分区规则，将第一个表的所有分区，全部拷贝到第二个表中来，第二个表在加载数据的时候，不需要指定分区了，直接用第一个表的分区即可。

**开启动态分区需要设置一些参数**

1. 开启动态分区参数设置

```java
set hive.exec.dynamic.partition=true;
```

2. 设置为非严格模式（动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。）

```java
set hive.exec.dynamic.partition.mode=nonstrict;
```

3. 在所有执行MR的节点上，最大一共可以创建多少个动态分区。

```java
set  hive.exec.max.dynamic.partitions=1000;
```

4. 在每个执行MR的节点上，最大可以创建多少个动态分区`。该参数需要根据实际的数据来设定。比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。

```java
set hive.exec.max.dynamic.partitions.pernode=100
```

5. 整个MR Job中，最大可以创建多少个HDFS文件。

在linux系统当中，每个linux用户最多可以开启1024个进程，每一个进程最多可以打开2048个文件，即持有2048个文件句柄，下面这个值越大，就可以打开文件句柄越大。

```java
set hive.exec.max.created.files=100000
```

6. 当有空分区生成时，是否抛出异常。一般不需要设置。

```java
set hive.error.on.empty.partition=false
```

#### 数据倾斜

**Map数量**

通常情况下，作业会通过input的目录产生一个或者多个map任务。

主要的决定因素有：`input的文件总个数`，`input的文件大小`，集群设置的文件块大小(目前为128M，可在hive中通过set dfs.block.size;命令查看到，该参数不能自定义修改）。

*举例：*

- **一个大文件**：假设input目录下有1个文件a，`大小为780M`，那么hadoop会将该文件a`分隔成7个块`（**6个128m的块和1个12m的块**），从而产生7个map数。
- **多个小文件** ：假设input目录下有`3个文件a，b，c`大小分别为`10m，20m，150m`，那么hadoop会`分隔成4个块`（**10m，20m，128m，22m**），从而产生4个map数。即，如果文件大于块大小(128m)，那么会拆分，如果小于块大小，则把该文件当成一个块，Hadoop是按照文件大小进行切分。
- **是不是map数越多越好？**：`答案是否定的`。如果一个任务有很多小文件（远远小于块大小128m），则每个小文件也会被当做一个块，用一个map任务来完成，而一个map任务启动和初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费。而且，同时可执行的map数是受限的。
- **是不是保证每个map处理接近128m的文件块，就高枕无忧了？**：`答案也是不一定`。比如有一个127m的文件，正常会用一个map去完成，但这个文件只有一个或者两个字段，却有几千万的记录，如果map处理的逻辑比较复杂，用一个map任务去做，肯定也比较耗时
- **小结**：针对上面的问题3和4，我们需要采取两种方式来解决：即减少map数和增加map数；这个就看看真实的场景去确认map的数量。

如何适当增加map数

当input的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。

**案例**

针对：**是不是保证每个map处理接近128m的文件块，就高枕无忧了？**

```sql
Select data_desc,
count(1),
count(distinct id),
sum(case when …),
sum(case when …),
sum(…)
from a group by data_desc
```

如果表a只有一个文件，大小为120M，但包含几千万的记录，如果用1个map去完成这个任务，肯定是比较耗时的，这种情况下，我们要考虑将这一个文件合理的拆分成多个，这样就可以用多个map任务去完成。

```sql
set mapreduce.job.reduces =10;
create table a_1 as
select * from a
distribute by rand();
```

 这样会将a表的记录，随机的分散到包含10个文件的a_1表中，再用a_1代替上面sql中的a表，则会用10个map任务去完成。        

 每个map任务处理大于12M（几百万记录）的数据，效率肯定会好很多。         

> 看上去，貌似这两种有些矛盾，一个是要合并小文件，一个是要把大文件拆成小文件，这点正是重点需要关注的地方，根据实际情况，控制map数量需要遵循两个原则：`使大数据量利用合适的map数；使单个map任务处理合适的数据量`；

**Reduce数量**

调整reduce个数方法一：

- 每个Reduce处理的数据量默认是256MB

```sql
-- 参数一
hive.exec.reducers.bytes.per.reducer=256123456
```

- 设置Reduce个数

```sql
-- 参数二
hive.exec.reducers.max=1009

计算reducer数的公式很简单：

N=min(参数2，总输入数据量/参数1)
```

调整reduce个数方法二

- 设置每个job的Reduce个数

```sql
set mapreduce.job.reduces = 15;
```

**reduce个数并不是越多越好**

- 过多的启动和初始化reduce也会消耗时间和资源；
- 另外，有多少个reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则也会出现小文件过多的问题.
- 在设置reduce个数的时候也需要考虑这两个原则：`处理大数据量利用合适的reduce数；使单个reduce任务处理数据量大小要合适`

#### 并行执行

Hive会将一个查询转化成`一个或者多个阶段`。这样的阶段可以使MapReduce阶段、抽样阶段、合并阶段、limit阶段。或者Hive执行过程中可能需要的其他阶段。默认情况下，Hive一次只会执行一个阶段。不过，某个特定的job可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，这样可能使得整个job的执行时间缩短。不过，如果有更多的阶段可以并行执行，那么job可能就越快完成。      

通过设置参数hive.exec.parallel值为true，就可以开启并发执行。不过，在共享集群中，需要注意下，如果job中并行阶段增多，那么集群利用率就会增加。

```sql
set hive.exec.parallel=true;     -- 打开任务并行执行
set hive.exec.parallel.thread.number=16; -- 同一个sql允许最大并行度，默认为8。
```

#### 严格模式

Hive提供了一个严格模式，可以`防止用户执行“高危”的查询`。        

通过设置属性`hive.mapred.mode`值为默认是非严格模式`nonstrict` 。开启严格模式需要修改`hive.mapred.mode`值为`strict`，开启严格模式可以禁止3种类型的查询。

**开启严格模式**

```xml
<property>
    <name>hive.mapred.mode</name>
    <value>strict</value>
    <description>
      The mode in which the Hive operations are being performed. 
      In strict mode, some risky queries are not allowed to run. They include:
        Cartesian Product.
        No partition being picked up for a query.
        Comparing bigints and strings.
        Comparing bigints and doubles.
        Orderby without limit.
    </description>
  </property>
```

1. 对于分区表，`用户不允许扫描所有分区`，除非where语句中含有分区字段过滤条件来限制范围，否则不允许执行。进行这个限制的原因是，通常分区表都拥有非常大的数据集，而且数据增加迅速。**没有进行分区限制的查询可能会消耗令人不可接受的巨大资源来处理这个表。**
2. 对于`使用了order by语句的查询，要求必须使用limit语句`。因为order by为了执行排序过程会将所有的结果数据分发到同一个Reducer中进行处理，**强制要求用户增加这个LIMIT语句可以防止Reducer额外执行很长一段时间**。
3. `限制笛卡尔积的查询`。对关系型数据库非常了解的用户可能期望在执行JOIN查询的时候`不使用ON语句而是使用where语句`，**这样关系数据库的执行优化器就可以高效地将WHERE语句转化成那个ON语句。\不幸的是，Hive并不会执行这种优化，因此，如果表足够大，那么这个查询就会出现不可控的情况。**

#### JVM重用

JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是对于很难避免小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。         

Hadoop的默认配置通常是使用派生JVM来执行map和Reduce任务的。这时JVM的启动过程可能会造成相当大的开销，尤其是执行的job包含有成百上千task任务的情况。`JVM重用可以使得JVM实例在同一个job中重新使用N次`。N的值可以在`Hadoop的mapred-site.xml`文件中进行配置。通常在10-20之间，具体多少需要根据具体业务场景测试得出。

```xml
<property>
  <name>mapreduce.job.jvm.numtasks</name>
  <value>10</value>
  <description>How many tasks to run per jvm. If set to -1, there is
  no limit. 
  </description>
</property>
```

也可以在hive中开启jvm重用

```sql
set  mapred.job.reuse.jvm.num.tasks=10;
```

这个设置来设置我们的jvm重用，这个功能的缺点是，开启JVM重用将一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡的”job中有某几个reduce task执行的时间要比其他Reduce task消耗的时间多的多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。

#### 推测执行

 在分布式集群环境下，因为程序Bug（包括Hadoop本身的bug），负载不均衡或者资源分布不均等原因，会造成同一个作业的多个任务之间运行速度不一致，有些任务的运行速度可能明显慢于其他任务（比如一个作业的某个任务进度只有50%，而其他所有任务已经运行完毕），则这些任务会拖慢作业的整体执行进度。为了避免这种情况发生，Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。 `Hive 同样可以开启推测执行`设置开启推测执行参数：Hadoop的`mapred-site.xml文`件中进行配置

```xml
<property>
  <name>mapreduce.map.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some map tasks  may be executed in parallel.</description>
</property>

<property>
  <name>mapreduce.reduce.speculative</name>
  <value>true</value>
  <description>If true, then multiple instances of some reduce tasks   may be executed in parallel.</description>
</property>
```

不过hive本身也提供了配置项来控制`reduce-side`的推测执行：

```sql
<property>
    <name>hive.mapred.reduce.tasks.speculative.execution</name>
    <value>true</value>
    <description>Whether speculative execution for reducers should be turned on. </description>
  </property>
```

#### 表的优化

**Join**

**Join 原则**

1. 小表Join大表：将key相对分散，并且`数据量小的表放在join的左边`，这样可以有效减少内存溢出错误发生的几率；再进一步，可以使用Group让小的维度表（1000条以下的记录条数）先进内存。在map端完成reduce。

```sql
select  count(distinct s_id)  from score;
select count(s_id) from score group by s_id; -- 在map端进行聚合，效率更高x`
```

2. 多个表关联时，最好分拆成小段，避免大sql（无法控制中间Job）
3. 大表Join大表空KEY过滤

有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。例如key对应的字段为空，操作如下：

```sql
create table ori(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

create table nullidtable(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

create table jointable(id bigint, time bigint, uid string, keyword string, url_rank int, click_num int, click_url string) row format delimited fields terminated by '\t';

```

不过滤进行连接

```sql
INSERT OVERWRITE TABLE jointable SELECT a.* FROM nullidtable a JOIN ori b ON a.id = b.id;
```

过滤空值后连接

```sql
INSERT OVERWRITE TABLE jointable SELECT a.* FROM (SELECT * FROM nullidtable WHERE id IS NOT NULL ) a JOIN ori b ON a.id = b.id;
```

**空Key转换**

有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。例如:

```sql
set hive.exec.reducers.bytes.per.reducer=32123456;
set mapreduce.job.reduces=7;
INSERT OVERWRITE TABLE jointable SELECT a.* FROM nullidtable a LEFT JOIN ori b ON  CASE WHEN  a.id IS NULL  THEN 'hive'  ELSE a.id  END;
```

结果：这样的后果就是所有为null值的id全部都变成了相同的字符串“hive”，及其容易造成数据的倾斜（所有的key相同，相同key的数据会到同一个reduce当中去）

为了解决这种情况，我们可以通过hive的rand函数，随记的给每一个为空的id赋上一个随机值，这样就不会造成数据倾斜**随机分布：**

```sql
set hive.exec.reducers.bytes.per.reducer=32123456;
set mapreduce.job.reduces=7;
INSERT OVERWRITE TABLE jointable SELECT a.* FROM nullidtable a LEFT JOIN ori b ON  CASE WHEN  a.id IS NULL  THEN concat('hive',rand())  ELSE a.id  END;
```

> 大表join小表或者小表join大表，就算是关闭map端join的情况下，在新的版本当中基本上没有区别了（hive为了解决数据倾斜的问题，会自动进行过滤）

**MapJoin**

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join（在Reduce阶段完成join）。容易发生数据倾斜。`可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

开启MapJoin参数设置：

1. 设置自动选择Mapjoin

```sql
set hive.auto.convert.join = true; --默认为true
```

2. 大表小表的阈值设置（默认25M以下认为是小表）：

```sql
set hive.mapjoin.smalltable.filesize=25123456;
```

![1643853762591](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/03/100244-654537.png)

首先是Task A，它是一个Local Task（在客户端本地执行的Task），负责扫描小表b的数据，将其转换成一个HashTable的数据结构，并写入本地的文件中，之后将该文件加载到DistributeCache中。         

接下来是Task B，该任务是一个没有Reduce的MR，启动MapTasks扫描大表a,在Map阶段，根据a的每一条记录去和DistributeCache中b表对应的HashTable关联，并直接输出结果。

由于MapJoin没有Reduce，所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件。

**案例实操：**

1. 开启Mapjoin功能

```sql
set hive.auto.convert.join = true; -- 默认为true
```

2. 执行小表JOIN大表语句

```sql
INSERT OVERWRITE TABLE jointable2
SELECT b.id, b.time, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
FROM smalltable s
JOIN bigtable  b
ON s.id = b.id;
```

3. 执行大表JOIN小表语句

```sql
NSERT OVERWRITE TABLE jointable2
SELECT b.id, b.time, b.uid, b.keyword, b.url_rank, b.click_num, b.click_url
FROM smalltable s
JOIN bigtable  b
ON s.id = b.id;
```

