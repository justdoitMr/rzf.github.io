# 深入理解Spark原理

## Spark 核心编程
<!-- TOC -->

- [深入理解Spark原理](#深入理解spark原理)
	- [Spark 核心编程](#spark-核心编程)
		- [RDD](#rdd)
			- [为什么需要RDD](#为什么需要rdd)
			- [什么是RDD](#什么是rdd)
			- [RDD的设计核心](#rdd的设计核心)
			- [SparkContext](#sparkcontext)
			- [小案例](#小案例)
			- [IO类比](#io类比)
			- [RDD之间的依赖关系](#rdd之间的依赖关系)
			- [RDD原理](#rdd原理)
			- [核心属性](#核心属性)
				- [分区列表](#分区列表)
				- [分区计算函数](#分区计算函数)
				- [RDD之间的依赖关系](#rdd之间的依赖关系-1)
				- [分区器（可选）](#分区器可选)
				- [首选位置（可选）](#首选位置可选)
			- [执行原理](#执行原理)
			- [基础编程](#基础编程)
				- [RDD对象的创建](#rdd对象的创建)
					- [从集合（内存）中创建 RDD](#从集合内存中创建-rdd)
					- [从外部存储（文件）创建RDD](#从外部存储文件创建rdd)
					- [从其他 RDD 创建](#从其他-rdd-创建)
					- [直接创建 RDD（new）](#直接创建-rddnew)
				- [RDD 并行度与分区](#rdd-并行度与分区)
			- [RDD方法分类](#rdd方法分类)
				- [对key-value数据类型的支持](#对key-value数据类型的支持)
				- [对数字型数据的支持](#对数字型数据的支持)
				- [**RDD特点**](#rdd特点)
				- [什么是弹性分布式数据集](#什么是弹性分布式数据集)
				- [RDD属性小结](#rdd属性小结)
			- [RDD算子概述](#rdd算子概述)
				- [转换算子](#转换算子)
				- [行动算子](#行动算子)
				- [常用算子](#常用算子)
			- [RDD 转换算子](#rdd-转换算子)
				- [**Value** **类型**](#value-类型)
					- [map](#map)
					- [foreach](#foreach)
					- [**saveAsTextFile**](#saveastextfile)
					- [mapValues](#mapvalues)
					- [mapPartitions](#mappartitions)
					- [mapPartitionsWithIndex](#mappartitionswithindex)
					- [flatmap](#flatmap)
					- [glom](#glom)
					- [groupBy](#groupby)
					- [Filter](#filter)
					- [sample](#sample)
					- [distinct](#distinct)
					- [coalesce](#coalesce)
					- [repartition](#repartition)
					- [sortBy](#sortby)
				- [双 Value 类型](#双-value-类型)
					- [intersection](#intersection)
					- [union](#union)
					- [subtract](#subtract)
					- [zip](#zip)
				- [Key - Value 类型](#key---value-类型)
					- [聚合函数详解](#聚合函数详解)
					- [partitionBy](#partitionby)
					- [reduceByKey](#reducebykey)
					- [groupByKey](#groupbykey)
					- [aggregateByKey](#aggregatebykey)
					- [foldByKey](#foldbykey)
					- [combineByKey](#combinebykey)
					- [sortByKey](#sortbykey)
					- [小结](#小结)
					- [join](#join)
					- [leftOuterJoin](#leftouterjoin)
					- [cogroup](#cogroup)
					- [获取键值集合](#获取键值集合)
					- [面试题](#面试题)
				- [项目练手](#项目练手)
					- [数据准备](#数据准备)
					- [需求描述](#需求描述)
					- [代码实现](#代码实现)
			- [行动算子](#行动算子-1)
				- [reduce](#reduce)
				- [collect](#collect)
				- [count](#count)
				- [first](#first)
				- [take](#take)
				- [takeOrdered](#takeordered)
				- [案例](#案例)
				- [aggregate](#aggregate)
				- [fold](#fold)
				- [countByKey](#countbykey)
				- [wordcount](#wordcount)
				- [save 相关算子](#save-相关算子)
				- [foreach](#foreach-1)
			- [阶段性项目](#阶段性项目)
			- [RDD 序列化](#rdd-序列化)
			- [RDD 依赖关系](#rdd-依赖关系)
				- [RDD血缘关系](#rdd血缘关系)
				- [RDD 窄依赖](#rdd-窄依赖)
				- [RDD 宽依赖](#rdd-宽依赖)
				- [RDD阶段划分](#rdd阶段划分)
				- [RDD 任务划分](#rdd-任务划分)
				- [Stage的切割规则](#stage的切割规则)
				- [shuffle 是划分 DAG 中 stage 的标识,同时影响 Spark 执行速度的关键步骤](#shuffle-是划分-dag-中-stage-的标识同时影响-spark-执行速度的关键步骤)
		- [RDD 缓存](#rdd-缓存)
			- [RDD的三个特性](#rdd的三个特性)
			- [使用缓存的意义](#使用缓存的意义)
			- [问题引出](#问题引出)
			- [RDD Cache 缓存](#rdd-cache-缓存)
			- [缓存级别](#缓存级别)
		- [RDD检查点](#rdd检查点)
			- [**CheckPoint的作用**](#checkpoint的作用)
			- [RDD CheckPoint API](#rdd-checkpoint-api)
			- [缓存和检查点区别](#缓存和检查点区别)
		- [RDD 分区器](#rdd-分区器)
				- [Hash 分区：对于给定的 key，计算其 hashCode,并除以分区个数取余](#hash-分区对于给定的-key计算其-hashcode并除以分区个数取余)
				- [Range分区](#range分区)
				- [自定义分区](#自定义分区)
				- [面试题目](#面试题目)
				- [RDD 文件读取与保存](#rdd-文件读取与保存)
		- [RDD的分区和Shuffle](#rdd的分区和shuffle)
			- [查看分区的方式](#查看分区的方式)
			- [指定RDD分区个数的方式](#指定rdd分区个数的方式)
			- [重定义分区数](#重定义分区数)
			- [通过其他算子指定分区个数](#通过其他算子指定分区个数)
			- [RDD中的shuffle过程](#rdd中的shuffle过程)
		- [Spark原理](#spark原理)
			- [总体概述](#总体概述)
				- [再看WordCount案例](#再看wordcount案例)
				- [再看Spark集群](#再看spark集群)
				- [逻辑执行图](#逻辑执行图)
				- [物理执行图](#物理执行图)
			- [逻辑执行计划](#逻辑执行计划)
					- [明确边界](#明确边界)
					- [RDD的生成](#rdd的生成)
					- [RDD之间的依赖关系](#rdd之间的依赖关系-2)
			- [物理执行图](#物理执行图-1)
				- [物理执行图的作用](#物理执行图的作用)
				- [RDD的计算-Task](#rdd的计算-task)
				- [如何划分阶段](#如何划分阶段)
				- [数据如何流动](#数据如何流动)
			- [运行过程](#运行过程)
				- [首先生成逻辑图](#首先生成逻辑图)
				- [物理图](#物理图)
				- [Job和Stage的关系](#job和stage的关系)
				- [Stage和Task的关系](#stage和task的关系)
				- [整体执行流程](#整体执行流程)
		- [累加器](#累加器)
			- [为什么要累加器](#为什么要累加器)
			- [实现原理](#实现原理)
			- [系统累加器](#系统累加器)
			- [可能出现的问题](#可能出现的问题)
			- [自定义累加器](#自定义累加器)
		- [广播变量](#广播变量)
			- [为什么要广播变量](#为什么要广播变量)
			- [实现原理](#实现原理-1)
			- [案例使用](#案例使用)
	- [三层架构模式](#三层架构模式)
		- [三层架构模式代码实现](#三层架构模式代码实现)
			- [模式包名](#模式包名)
			- [application层](#application层)
			- [common层](#common层)
			- [util层](#util层)
			- [Controller层](#controller层)
			- [Dao层](#dao层)
			- [service层](#service层)

<!-- /TOC -->
Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集(可以看作是一种数据结构)
- 累加器：分布式共享**只写**变量
- 广播变量：分布式共享**只读**变量

接下来我们一起看看这三大数据结构是如何在数据处理中使用的，上面的三大数据结构是Spark中的地基api操作。

一共有两种低级的API,一种是用于处理分布式数据(RDD)，另一种是用于分发和处理分布式共享变量（广播变量和累加器），sparkContext是底层API的入口，

### RDD

#### 为什么需要RDD

没有RDD/DataSet之前,做WordCount(大数据计算)可以使用:

1. 原生集合:Java/Scala中的List,但是只支持单机版! 不支持分布式!如果要做分布式的计算,需要做很多额外工作,**线程/进程通信,容错,自动均衡**.....麻烦,所以就诞生了框架!

2. MR:效率低(运行效率低,开发效率低)--早就淘汰，所以需要有一个分布式的数据抽象,也就是用该抽象,可以表示分布式的数据集合,那么基于这个分布式集合进行操作,就可以很方便的完成分布式的WordCount!(**该分布式集合底层应该将实现的细节封装好,提供简单易用的API**!)

![1621747526106](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165810-78279.png)

![1621747665897](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/132807-799841.png)

![1621747684369](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/132816-462746.png)

> 上面直观的展示了spark可以基于内存的迭代计算，而hadoop不仅不可以做迭代计算，中间结果还需要落盘，做迭代计算智能是多个作业顺序执行，效率非常的低效。

#### 什么是RDD

RDD（Resilient Distributed Dataset）叫做**弹性分布式数据集（区别于我们普通的集合式数据结构，只能够在单机上面运行，RDD可以在集群中运行，也就是分布式，但是我们编程的时候，可以像使用单机版的集合一样去使用，简化了编程方式。）**，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个**弹性的、不可变、可分区、里面的元素可并行计算**的集合。类似于我们程序中的Task类。subTask是真正意义上的计算任务，Task算是一种数据结构，可以认为类似于spark中的RDD数据结构。

RDD提供了一个抽象的数据模型，不必担心底层数据的分布式特性（**可以认为将底层的分布式特性进行封装，对我们只提供接口编程**），只需将具体的应用逻辑表达为一系列转换操作（函数），不同RDD之间的转换操作之间还可以形成依赖关系，进而实现管道化，从而避免了中间结果的存储，大大降低了数据复制、磁盘IO和序列化开销,并且还提供了更多的API(map/reduec/filter/groupBy...)。Spark中所有的运算以及操作都建立在 RDD 数据结构的基础之上。

#### RDD的设计核心

![1621687801583](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/205007-358874.png)

**通俗理解RDD**

可以认为RDD是分布式的列表List或数组Array，是一种抽象的数据结构，可以处理分布式的计算任务：

RDD是一个抽象类Abstract Class和泛型Generic Type：

**源码**

~~~ java
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging 
//transinent表示币可以进行序列化操作
~~~

**图示**

![1614214634651](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/085824-578280.png)

我们把每一种计算都封装为一个个的RDD计算单元，然后发送到不同的Executor计算节点上面进行计算，这样可以实现迭代式的计算。可以把RDD看做是Task。其实就是一种来准备计算逻辑的一种数据结构，一个大的计算，可以分解为多个RDD计算组合。

![1614219619509](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091635-340950.png)

RDD中还包括一个分区的概念，分区的目的就是为了把数据划分为小的数据集，可以分到多个task当中，提高task的并行度。

**什么是不可变，分区，并行计算**

![1621763638087](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/175400-422833.png)

一个RDD有多个分区，每一个分区都被一个Task任务进行处理分析。

![1621763670184](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165821-417273.png)

#### SparkContext

SparkContext是程序的入口，相当于程序的执行环境，提交我们的作业之后，Driver会创建我们的SparkContext环境，想要使用RDD数据结构，我们必须使用SparkContext创建。

**获取SparkContext上下文环境**

~~~ java
object Test01 {

	def main(args: Array[String]): Unit = {
	//	创建SparkContext
	//	1 创建SparkConf配置文件
	val conf: SparkConf = new SparkConf().setMaster("local[6]").setAppName("sparkContext")

	//	2 创建SparkContext,是程序的入口，可以设置参数，创建rdd
	val sc = new SparkContext(conf)


	//	3 关闭SparkContext，释放集群的资源

	//	通过SparkContext可以创建集群的上写文，可以创建RDD

	}
}
~~~

#### 小案例

```java
object Test04 {

		def main(args: Array[String]): Unit = {
			//	创建SparkContext
			//	1 创建SparkConf配置文件
			val conf: SparkConf = new SparkConf().setMaster("local[6]").setAppName("sparkContext")

			//	2 创建SparkContext,是程序的入口，可以设置参数，创建rdd
			val sc = new SparkContext(conf)

			//读取文件生成数据集
			val sourceRDD: RDD[String] = sc.textFile("")

			//取出ip，转换数据结构
			val ipRDD: RDD[(String, Int)] = sourceRDD.map((item: String) => {
				//对字符串进行切割，然后取出ip地址，最后对其进行包装
				(item.split(" ")(0), 1)
			})

			//对数据进行清洗操作，去掉空数据，非法的数据
			val clearRDD: RDD[(String, Int)] = ipRDD.filter(item => {
				//判断ip地址是否是空，如果是空，就过滤掉,返回true保留，返回false就过滤掉
				StringUtils.isNotEmpty(item._1)
			})

			//对ip出现的次数进行聚合操作,初始值：curr:1,agg:0
			val IpAggRDD: RDD[(String, Int)] = clearRDD.reduceByKey((curr, agg) => {
				curr + agg
			})
			//根据ip出现的次数进行排序,item是reduceByKey的结果,按照降序进行排序

			val sordetRDD: RDD[(String, Int)] = IpAggRDD.sortBy((item: (String, Int)) => item._2, ascending = false)

			//取出结果进行打印
			val res: Array[(String, Int)] = sordetRDD.collect()

			//取出前10条ip地址
			res.take(10).foreach(println)

			sc.stop()

		}

}
```

**提出问题**

![1621745561514](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/125244-130276.png)

- 当文件特别大的时候，如何处理？---->分区
- 如何放在集群上面执行？
- 任务如何分解？
- 如何移动计算？
- 如何容错？
- 如果多个RDD之间依赖过长如何解决？

**针对上面的问题，给出解决方案**

- 对于大文件，可以做并行计算，并行计算的前提是对数据集可以分区操作。

![1621746053474](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/130059-252502.png)

- 把一个大作业分成多个小作业，由任务调度器调度执行

![1621746301334](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/175627-134758.png)

- 可以把文件划分为多个分区，一个分区形成一个Task。

![1621746493012](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/175908-120721.png)

在对数据进行分区的时候，每一个hdfs上面的数据块对应为spark中的一个分区。

![1621746603214](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/175935-425200.png)

这是一个很基础的优化，简单来说就是让存储的单元尽量靠近计算的单元，不要进行数据的传输。尽量在同一台服务器上面或者机架上面。

![1621747102311](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/175939-308466.png)

简单来说两种容错方式，**数据备份副本，记录依赖关系**
- 对计算过的数据做备份，存储副本，耗费内存。
- 记录此数据的计算关系，出错的话，重新计算一遍，可能费时间。

![1621747142938](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/131908-963209.png)

#### IO类比

io操作体现装饰着模式

**使用FileInputStream**

![1614215428911](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091638-485155.png)

如果只是用简单的FileInputStream输入流，那么是一个一个字节读取，每读取到一个字符，就会输出到控制台，这样效率很低，但是如果使用缓冲流，先把读取到的数据缓存到一个缓缓区中，等待到缓冲区满后，全部输出，类似spark中的批处理，效率高很多。批处理比一个一个执行性能高很多。

**使用BufferedInputStream**

![1614215562161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/091244-732780.png)

**InputStreamReader**

![1614216292488](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091642-298330.png)

![1614216305685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/123209-531147.png)

从文件中读取的时候是以字节流形式读取，然后使用InputStreamReader流转换为字节流，最后使用BufferedReader流缓冲到缓冲区后，最后全部输出，上述的一系列流操作，真正读取的是FileInputStream流在底层读取，而InputStreamReader和BufferedReader仅仅是做装饰工作，是对底层的封装，InputStreamReader负责把字节流转换为字符流，比如3个字节组成一个汉字，那么就等到三个字节后，InputStreamReader就把这三个字节转换为一个汉字，然后交给BufferedReader存放在缓冲区，等到缓冲区快满的时候打印，上述过程中，最重要的方法是readLine()函数，应为此函数会触发读取操作，在建立流的过程中只是建立一套传输系统，不会真正的读取数据，只有真正readLine()读取数据的时候，才会触发读取操作，可以认为是一个懒加载操作。

RDD也是我们的一个最小计算单元，以后的操作都是在RDD上封装叠加操作。

#### RDD之间的依赖关系

以词频统计WordCount程序为例，查看整个Job中各个RDD类型及依赖关系

运行程序结束后，查看WEB UI监控页面，此Job（RDD调用**foreach**触发）执行DAG图：

![1621765886934](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165849-860274.png)

RDD 的一个重要优势是能够记录 RDD 间的依赖关系，即所谓血统（lineage）。通过丰富的转移操作（Transformation），可以构建一个复杂的有向无环图，并通过这个图来一步步进行计算。

**上图中相关说明如下：**

黑色圆圈表示一个RDD，上图中有5个黑色圆圈，说明整个Job中有个5个RDD

- 【1号】RDD类型：HadoopRDD，从HDFS或LocalFS读取文件数据；

- 【2号、3号和4号】RDD类型：MapPartitionsRDD，从一个RDD转换而来，没有经过shuffle操作；

- 【5号】RDD类型：ShuffledRDD，从一个RDD转换而来，经过Shuffle重分区操作，Spark Shuffle类似MapReduce流程中Map Phase和Reduce Phase中的Shuffle；

浅蓝色矩形框表示调用RDD函数

上图中【5号】RDD所在蓝色矩形框上的函数【reduceByKey】，表明【5号】RDD是【4号】RDD调用reduceByKey函数得到；

查看ShuffleRDD源码，实现了RDD的5个特性

![1621766118667](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165854-158400.png)

- 依赖关系
- 分区列表
- 计算函数
- 分区函数
- 最佳位置

每一种类型的RDD基本都需要实现RDD的5个特点，RDD是所有其他RDD类型的父类，所有子类可以重新实现这5个方法或者是继承。

#### RDD原理

![1614217505781](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/094507-970505.png)

![1614217563861](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091647-495924.png)

**小结**

RDD数据的处理方式类似于IO流，也有装饰者设计模式，但是也有不同，Io中真正new对象时候不会触发文件数据的读取，RDD数据只有在调用collect的时候才会真正执行逻辑操作，之前的封装都是对功能的扩展，RDD他是不保存数据，也就是没有缓冲区，但是IO流有缓冲区，会临时保存数据。RDD其实就是通过功能的组合，最终完成一个复杂的功能计算。

> 共同点都是通过懒惰加载，触发执行后才加载数据然后执行。
> 
>不同之处是IO有缓冲区，但是RDD没有缓冲区

我们可以看到RDD有很多的子类，也就是我们可以有多种数据源，其中在执行转换的时候，RDD还可以转换为其他类型的RDD。

![1621764853796](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/181418-940811.png)

**源码角度理解**

![1621764624142](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/181027-557303.png)

~~~ java
//读取文件
val fileRDD: RDD[String] = sc.textFile("input/word.txt")
  
 def textFile(
      path: String,
      minPartitions: Int = defaultMinPartitions): RDD[String] = withScope {
    assertNotStopped()
    hadoopFile(path, classOf[TextInputFormat], classOf[LongWritable], classOf[Text],
      minPartitions).map(pair => pair._2.toString).setName(path)
  }

def hadoopFile[K, V](
      path: String,
      inputFormatClass: Class[_ <: InputFormat[K, V]],
      keyClass: Class[K],
      valueClass: Class[V],
      minPartitions: Int = defaultMinPartitions): RDD[(K, V)] = withScope {
    assertNotStopped()

    // This is a hack to enforce loading hdfs-site.xml.
    // See SPARK-11227 for details.
    FileSystem.getLocal(hadoopConfiguration)

    // A Hadoop configuration can be about 10 KiB, which is pretty big, so broadcast it.
    val confBroadcast = broadcast(new SerializableConfiguration(hadoopConfiguration))
    val setInputPathsFunc = (jobConf: JobConf) => FileInputFormat.setInputPaths(jobConf, path)
      //这里创建出HadoopRDD数据结构
    new HadoopRDD(
      this,
      confBroadcast,
      Some(setInputPathsFunc),
      inputFormatClass,
      keyClass,
      valueClass,
      minPartitions).setName(path)
  }

//对读取的数据做扁平化处理
val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))
  
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] = withScope {
    val cleanF = sc.clean(f)
      //这里创建MapPartitionsRDD类型的RDD
    new MapPartitionsRDD[U, T](this, (_, _, iter) => iter.flatMap(cleanF))
  }

	// 转换数据结构 word => (word, 1)
val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_, 1))
      
def map[U: ClassTag](f: T => U): RDD[U] = withScope {
    val cleanF = sc.clean(f)
      //这里创建MapPartitionsRDD数据结构
    new MapPartitionsRDD[U, T](this, (_, _, iter) => iter.map(cleanF))
  }

// 将转换结构后的数据按照相同的单词进行分组聚合
		val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_ + _)
def reduceByKey(func: (V, V) => V): RDD[(K, V)] = self.withScope {
    reduceByKey(defaultPartitioner(self), func)
  }

 def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)] = self.withScope {
    combineByKeyWithClassTag[V]((v: V) => v, func, func, partitioner)
  }

def combineByKeyWithClassTag[C](
      createCombiner: V => C,
      mergeValue: (C, V) => C,
      mergeCombiners: (C, C) => C,
      partitioner: Partitioner,
      mapSideCombine: Boolean = true,
      serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)] = self.withScope {
    require(mergeCombiners != null, "mergeCombiners must be defined") // required as of Spark 0.9.0
    if (keyClass.isArray) {
      if (mapSideCombine) {
        throw new SparkException("Cannot use map-side combining with array keys.")
      }
      if (partitioner.isInstanceOf[HashPartitioner]) {
        throw new SparkException("HashPartitioner cannot partition array keys.")
      }
    }
    val aggregator = new Aggregator[K, V, C](
      self.context.clean(createCombiner),
      self.context.clean(mergeValue),
      self.context.clean(mergeCombiners))
    if (self.partitioner == Some(partitioner)) {
      self.mapPartitions(iter => {
        val context = TaskContext.get()
        new InterruptibleIterator(context, aggregator.combineValuesByKey(iter, context))
      }, preservesPartitioning = true)
    } else {
      //创建ShuffledRDD数据结构
      new ShuffledRDD[K, V, C](self, partitioner)
        .setSerializer(serializer)
        .setAggregator(aggregator)
        .setMapSideCombine(mapSideCombine)
    }
  }
//做最终的聚合操作
def collect(): Array[T] = withScope {
    val results = sc.runJob(this, (iter: Iterator[T]) => iter.toArray)
    Array.concat(results: _*)
  }
~~~

**名词说明**

- **弹性**（可变）
  - 存储的弹性：内存与磁盘的自动切换；比MR计算模型效率高，数据首选存储在内存当中，如果内存存储不下，会进行落盘。
  - 容错的弹性：数据丢失可以自动恢复，有多种方式可以实现容错(依赖链，数据副本)；
  - 计算的弹性：计算出错重试机制；计算出错后，可以重头重新计算。
  - 分片的弹性：可根据需要重新分片。分区操作，
- 分布式：数据可以进行分片，存储在大数据集群不同节点上。
- 数据集：RDD 封装了计算逻辑，并不保存数据，也就是封装的是对数据的计算操作步骤，不会像Io一样会存储数据。
- 数据抽象：RDD 是一个抽象类，需要子类具体实现。
- 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD 里面封装计算逻辑，也就是一个计算封装为RDD后，以后不可以对这个封装好的RDD在进行扩展，只能重新创建一个RDD对前面的RDD逻辑进行封装和装饰。
- 可分区、并行计算，可以分区是进行并行计算的前提。

#### 核心属性

RDD 是一个分布式数据集的抽象表示，不仅表示了数据集，还表示了这个数据集从哪来、如何计算，主要属性包括五个方面（必须牢记，通过编码加深理解，面试常问），如下为RDD 源码：

~~~ java
* Internally, each RDD is characterized by five main properties:
 *
 *  - A list of partitions
 *  - A function for computing each split
 *  - A list of dependencies on other RDDs
 *  - Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
 *  - Optionally, a list of preferred locations to compute each split on (e.g. block locations for
 *    an HDFS file)
 *
~~~

前三个特征每个RDD都具备的，后两个特征可选的。

RDD将Spark的底层的细节都隐藏起来（**自动容错、位置感知、任务调度执行，失败重试等**），让开发者可以像操作本地集合一样**以函数式编程的方式操作RDD**这个分布式数据集，进行各种并行计算，RDD中很多数据处理函数/API与Scala集合相同/类似。

![1621765124844](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/181849-32430.png)

> 简单说明一下：
>
> 对于大数据集，需要进行并行计算加快计算速度，所以要对数据进行分区操作，以分区为单位对数据进行并行计算，而分区函数就是针对Source端数据进行分区操作，当然spark还需要考虑容错机制，具体来说有两种容错机制，spark使用的是根据前一步的计算逻辑去计算下一步数据集，所以需要依赖，也就是如果知道其父亲的操作，那么就可以推出下一步数据集，那么在其父亲和儿子之间的计算方法就是计算函数。

**下面详细介绍五大属性**
##### 分区列表

- 一组分片(Partition)/一个分区(Partition)列表，即数据集的基本组成单位；数据集的最小单位就是分区。
- RDD 数据结构中存在分区列表，用于执行任务时并行计算，**是实现分布式计算的重要属性**,分区之间的数据没有相互关系，把数据分为多个分区，然后封装为Task在不同的节点上面执行。也就是说多个分区之间是相互独立的，不会相互一影响。
- 对于RDD来说，每个分片都会被一个计算任务处理，分片数决定并行度；
- 用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值；
- **分区的目的是提高并行度**。

~~~ java
 /**
   * Implemented by subclasses to return the set of partitions in this RDD. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   *
   * The partitions in this array must satisfy the following property:
   *   `rdd.partitions.zipWithIndex.forall { case (partition, index) => partition.index == index }`
   */
  protected def getPartitions: Array[Partition]  //返回的是一个分区列表
~~~

##### 分区计算函数

- 一个函数会被作用在每一个分区；

- Spark中RDD的计算是以分片为单位的，compute函数会被作用到每个分区上；

- Spark在计算时，是使用分区函数对每一个分区进行计算，即计算每一个分区,计算分区的逻辑是完全一样的。

~~~ java
/**
   * :: DeveloperApi ::
   * Implemented by subclasses to compute a given partition.
   */
  @DeveloperApi
  def compute(split: Partition, context: TaskContext): Iterator[T]
~~~

##### RDD之间的依赖关系

- 一个RDD会依赖于其他多个RDD；

- **RDD的每次转换都会生成一个新的RDD**，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算（**Spark的容错机制**）；

- RDD 是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD 建立依赖关系。
- 这种依赖关系就像迭代计算一样，上一步的计算结果是存储在内存中，下一步计算可以接着进行完成计算，效率非常高。

~~~ java
 /**
   * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   */
  protected def getDependencies: Seq[Dependency[_]] = deps
~~~

##### 分区器（可选）

- 可选项,对于KeyValue类型的RDD会有一个Partitioner，即RDD的分区函数，会根据key对数据进行分区操作，也可以自定义分区操作。

- **当前Spark中实现了两种类型的分区函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。**

- 只有对于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。

- Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。

- 当数据为 KV 类型数据时，可以通过设定分区器自定义数据的分区,读取数据的时候需要分区，分区器就是定义如何分区数据的规则。

~~~ java
/** Optionally overridden by subclasses to specify how they are partitioned. */
  @transient val partitioner: Option[Partitioner] = None

~~~

##### 首选位置（可选）

- 可选项，一个列表，存储存取每个Partition的优先位置(preferred location)；

- 对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。

- **按照"移动数据不如移动计算"的理念，Spark在进行任务调度的时候，会尽可能选择那些存有数据的worker节点来进行任务计算。（数据本地性）**

- **计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算，也就是决定每一个Task分发给哪一个节点进行执行,效率最优，（移动数据不如计算）**

> 在这里我们可以想想一下，首选位置记录的是我们每一个分片数据存储在hdfs上的哪一个块上，也就是保存这两者之间的映射，而task正是我们封装好的计算任务，是没有数据的，这些计算任务需要发送到具体的节点上然后拉取数据进行计算，所以有时候，某些task会被发送到距离某些数据最近的节点上进行计算，最好的情况是发送到数据存储的节点上面，这样就不需要网络传输数据，效率非常高。

> 从这个角度，理解RDD是封装了我们的计算，也许会更好理解一点，RDD本身是不存储数据，只是封装了计算的步骤，也就是我们处理数据的方法步骤。

~~~ java
/**
   * Optionally overridden by subclasses to specify placement preferences.
   */
  protected def getPreferredLocations(split: Partition): Seq[String] = Nil
~~~

**图示**

![1614221512980](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/184038-830467.png)

移动数据需要消耗网络传输资源。

![1621765744583](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/182907-853981.png)

#### 执行原理

从计算的角度来讲，数据处理过程中需要计算资源（内存 & CPU）和计算模型（逻辑）。执行时，需要将计算资源和计算模型进行协调和整合。

Spark 框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的计算任务。然后将任务发到已经分配资源的计算节点上,  按照指定的计算模型进行数据计算。最后得到计算结果。

**WordCount中的RDD**

1. 分区列表:每个RDD都有会分区的概念,类似与HDFS的分块, 分区的目的:提高并行度!
2. 用于计算每个分区的函数:用函数来操作各个分区中的数据
3. 对其他RDD的依赖列表:后面的RDD需要依赖前面的RDD
4. 可选地，键值RDDs的分区器（例如，reduceByKey中的默认的Hash分区器）
5. 可选地，计算每个分区的首选位置列表/最佳位置（例如HDFS文件）--移动计算比移动数据更划算!

我们看看每一个属性所在的位置：
![20211106141332](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106141332.png)

RDD 是 Spark 框架中用于数据处理的核心模型，接下来我们看看，在 Yarn 环境中，RDD 的工作原理:

1. 启动 Yarn 集群环境

![1614221773058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/184107-901473.png)

2. Spark 通过申请资源创建作业调度节点和任务计算节点（Driver和Executor），client模式。

   Driver，Executor都是运行在某一个NodeManager节点上面的。Resourcemanager是负责管理整个所有的Nodemanager节点的，真正工作的节点是Nodemanager节点

   ![1614221855307](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/105736-882674.png)

3. Spark 框架根据需求将计算逻辑根据分区划分成不同的任务

   可以看到，**任务的数量取决与分区的个数**，Driver负责调度RDD，多个RDD计算逻辑进行关联，然后被分解为多个task任务，然后把任务放进任务池中。

![1614222023617](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/110025-291725.png)

4. 调度节点将任务根据计算节点状态发送到对应的计算节点进行计算。

![1614222280933](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/110442-417648.png)

从以上流程可以看出 RDD在整个流程中主要用于将**计算逻辑**进行封装，并生成Task 发送给Executor节点执行计算，接下来我们就一起看看Spark 框架中RDD 是具体是如何进行数据处理的。

#### 基础编程

##### RDD对象的创建

RDD中的数据可以来源于2个地方：本地集合或外部数据源

创建RDD有四种方式

- 通过本地集合创建RDD
- 通过外部文件创建RDD
- 通过RDD衍生出新的RDD
- 直接NEW出一个RDD

![1616723294144](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/094826-620596.png)

在 Spark 中创建RDD 的创建方式可以分为四种：

###### 从集合（内存）中创建 RDD

从集合中创建RDD，`[Spark ](https://www.iteblog.com/archives/tag/spark/)`主要提供了两个方法：parallelize 和 makeRDD，数据的分区个数可以指定。

~~~ java
object RddMemory {

	def main(args: Array[String]): Unit = {


	//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
	val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")
	//	创建上写文
	val context = new SparkContext(conf)

	//	创建RDD，从内存中创建RDD，将内存中集合中的数据作为数据的来源
		val seq=Seq[Int](1,2,3,4,5)
		//此时我们的结合seq中的数据就已经和rdd关联
		//parallelize:并行的意思
		//val rdd:RDD[Int] = context.parallelize(seq)
		//在makeRDD方法的底层其实还是调用的是parallelize方法创建RDD
		val rdd:RDD[Int]=context.makeRDD(seq)

		rdd.collect().foreach(println)
	//	关闭环境
		context.stop()
	}
}
//makeRDD()方法的底层代码，使用makRDD方法创建需要给出分区的个数
def makeRDD[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope {
    parallelize(seq, numSlices)//调用parallelize方法创建RDD对象
  }

//parallelize()方法
def parallelize[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope {
    assertNotStopped()
    new ParallelCollectionRDD[T](this, seq, numSlices, Map[Int, Seq[String]]())
  }
~~~

###### 从外部存储（文件）创建RDD

由外部存储系统的数据集创建RDD 包括：本地的文件系统，所有Hadoop 支持的数据集，比如HDFS、HBase 等，数据的分区数量取决于外部数据源的分区个数。

**本地文件创建RDD**

~~~ java
object RDDFileCreate {

	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")
		//	创建上写文
		val context = new SparkContext(conf)

		//	创建RDD，从文件中创建RDD，将文件中的数据作为数据的来源
		//path路径一当前环境的跟路径为基准，可以写相对路径，也可以写绝对路径,
		//如果向一次读取多个文件，可以指定一个目录名字
		//文件中的数据是按照行进行处理的
		//val rdd:RDD[String] = context.textFile(path = "datas/data")
		//读取目录下所有文件统计，文件名称还可以使用通配符，读取指定的多个文件
		val rdd:RDD[String] = context.textFile(path = "datas/")
		//path还可以是分布式系统文件的路径，比如hdfs文件系统路径，如果是读取hdfs上面的文件，那么会按照hdfs系统的上分块来算
		rdd.foreach(println)
      
		//	关闭环境
		context.stop()
	}
}
~~~

**判断结果来自哪一个文件&&小文件读取**

在实际项目中，有时往往处理的数据文件属于小文件（每个文件数据数据量很小，比如KB，几十MB等），文件数量又很大，如果一个个文件读取为RDD的一个个分区，计算数据时很耗时性能低下，使用SparkContext中提供：wholeTextFiles类，专门读取小文件数据。

~~~ java
object RDDFileCre {

	def main(args: Array[String]): Unit = {


		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")
		//	创建上写文
		val context = new SparkContext(conf)

		//	创建RDD，从文件中创建RDD，将文件中的数据作为数据的来源
		//textFile:以行为单位读取数据
		//wholeTextFiles:以文件为单位读取数据
		//读取结果以元组形式表示，第一个结果表示路径，第二个内容表示文件内容
		val rdd = context.wholeTextFiles(path = "datas")
		rdd.collect().foreach(println)
      
		//	关闭环境
		context.stop()
	}
}
~~~

###### 从其他 RDD 创建

主要是通过一个RDD 运算完后，再产生新的RDD，但是RDD是不可以变的，也就是会返回一个新的RDD。详情请参考后续章节

###### 直接创建 RDD（new）

使用 new 的方式直接构造RDD，一般由Spark 框架自身使用。

#####  RDD 并行度与分区

在讲解 RDD 属性时，多次提到了分区（partition）的概念。分区是一个偏物理层的概念，也是 RDD 并行计算的单位。数据在 RDD 内部被切分为多个子集合，每个子集合可以被认为是一个分区，运算逻辑最小会被应用在每一个分区上，每个分区是由一个单独的任务（task）来运行的，所以分区数越多，整个应用的并行度也会越高。

默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建RDD 时指定。记住，**这里的并行执行的任务数量，并不是指的切分任务的数量**，不要混淆了。不要和读取数据的分片数记错

**获取RDD分区个数的方法**

![1621766520984](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165929-481463.png)

**RDD分区原则**

是使得分区的个数尽量等于集群中的CPU核心(core)数目，这样可以充分利用CPU的计算资源；

在实际中为了更加充分的压榨CPU的计算资源，会把并行度设置为cpu核数的2~3倍；

**RDD分区数的影响因数--了解**

RDD分区数和启动时指定的核数、调用方法时指定的分区数、如文件本身分区数有关系，具体如下说明：

1. 启动的时候指定的CPU核数确定了一个参数值：

~~~ java
spark.default.parallelism=指定的CPU核数(集群模式最小2)
~~~

2. 对于parallelize/makeRDD(集合,分区数)方法

~~~ java
如果没有指定分区数，就使用spark.default.parallelism

如果指定了就使用指定的分区数(不要指定大于spark.default.parallelism)
~~~

3. 对于textFile(文件, 分区数)方法

~~~ java
defaultMinPartitions

如果没有指定分区数sc.defaultMinPartitions=min(defaultParallelism,2) 

如果指定了就使用指定的分区数sc.defaultMinPartitions=指定的分区数rdd的分区数
~~~

4. rdd的分区数

~~~~ java
对于本地文件

rdd的分区数 = max(本地file的分片数，sc.defaultMinPartitions)

对于HDFS文件

rdd的分区数 = max(hdfs文件的block数目，sc.defaultMinPartitions)

所以如果分配的核数为多个，且从文件中读取数据创建RDD，即使hdfs文件只有1个切片，最后的Spark的RDD的partition数也有可能是2
~~~~

**案例**

~~~ JAVA
object RDD_PAR {

	def main(args: Array[String]): Unit = {


		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")

		//配置spark.default.parallelism属性。默认分区个数
		conf.set("spark.default.parallelism","5")
		//	创建上写文
		val context = new SparkContext(conf)

		//numSlices表示切片数，也可以认为是分区的数量
		//如果第二个参数不传的话，会使用默认值defaultParallelism，默认是8个分区
		//scheduler.conf.getInt("spark.default.parallelism", totalCores)
		//默认情况下,spark会从配置对象中获取配置参数：spark.default.parallelism
		//如果获取不到，就使用totalCores，这个取值为当前环境的最大核心数量
		val rdd=context.makeRDD(
			List(1,2,3,4),2
		)

		//将处理的数据保存为分区文件,处理的数据以分区为单位进行保存
		rdd.saveAsTextFile("output")

		//	关闭环境
		context.stop()
	}
}

~~~

读取内存数据时，数据可以按照并行度的设定进行数据的分区操作，数据分区规则的源码如下

~~~ java
override def getPartitions: Array[Partition] = {
    val slices = ParallelCollectionRDD.slice(data, numSlices).toArray
    slices.indices.map(i => new ParallelCollectionPartition(id, i, slices(i))).toArray
  }
 /**
   * Slice a collection into numSlices sub-collections. One extra thing we do here is to treat Range
   * collections specially, encoding the slices as other Ranges to minimize memory cost. This makes
   * it efficient to run Spark over RDDs representing large sets of numbers. And if the collection
   * is an inclusive Range, we use inclusive range for the last slice.
   */
  def slice[T: ClassTag](seq: Seq[T], numSlices: Int): Seq[Seq[T]] = {
    if (numSlices < 1) {
      throw new IllegalArgumentException("Positive number of partitions required")
    }
    // Sequences need to be sliced at the same set of index positions for operations
    // like RDD.zip() to behave as expected
    def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
      (0 until numSlices).iterator.map { i =>
        val start = ((i * length) / numSlices).toInt
        val end = (((i + 1) * length) / numSlices).toInt
        (start, end)
      }
    }
    seq match {
      case r: Range =>
        positions(r.length, numSlices).zipWithIndex.map { case ((start, end), index) =>
          // If the range is inclusive, use inclusive range for the last slice
          if (r.isInclusive && index == numSlices - 1) {
            new Range.Inclusive(r.start + start * r.step, r.end, r.step)
          }
          else {
            new Range(r.start + start * r.step, r.start + end * r.step, r.step)
          }
        }.toSeq.asInstanceOf[Seq[Seq[T]]]
      case nr: NumericRange[_] =>
        // For ranges of Long, Double, BigInteger, etc
        val slices = new ArrayBuffer[Seq[T]](numSlices)
        var r = nr
        for ((start, end) <- positions(nr.length, numSlices)) {
          val sliceSize = end - start
          slices += r.take(sliceSize).asInstanceOf[Seq[T]]
          r = r.drop(sliceSize)
        }
        slices
      case _ =>
        val array = seq.toArray // To prevent O(n^2) operations for List etc
        positions(array.length, numSlices).map { case (start, end) =>
            array.slice(start, end).toSeq
        }.toSeq
    }
  }
//下面的方法是主要计算分区的方法
//第一个参数表示数组的长度，第二个参数表示切片的数量
 def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
      (0 until numSlices).iterator.map { i =>
        val start = ((i * length) / numSlices).toInt
        val end = (((i + 1) * length) / numSlices).toInt
        (start, end)
      }
    }
~~~

**划分数据切片如下**

![1614252535631](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/192857-210853.png)

**文件读取**

读取文件数据时，数据是按照Hadoop 文件读取的规则进行切片分区，而切片规则和数据读取的规则有些差异，具体 Spark 核心源码如下

~~~ java
throw new IOException("Not a file: "+ file.getPath());
}
totalSize += file.getLen();
}

long goalSize = totalSize / (numSplits == 0 ? 1 : numSplits);
long minSize = Math.max(job.getLong(org.apache.hadoop.mapreduce.lib.input. FileInputFormat.SPLIT_MINSIZE, 1), minSplitSize);

...

for (FileStatus file: files) {

...

if (isSplitable(fs, path)) {
long blockSize = file.getBlockSize();
long splitSize = computeSplitSize(goalSize, minSize, blockSize);

...

}
protected long computeSplitSize(long goalSize, long minSize,
long blockSize) {
return Math.max(minSize, Math.min(goalSize, blockSize));
}

~~~

**如何划分文件切片**

~~~ java
object RDDFile_ {

	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")
		//	创建上写文
		val context = new SparkContext(conf)

		//textFile默认从文件读取数据，可以设定分区的个数
		//minPartitions：最小分区数量是2
		//def defaultMinPartitions: Int = math.min(defaultParallelism, 2)
		//val rdd = context.textFile("datas/data")
		//通过第二个参数指定分区数量
		//spark读取文件，其实底层就是使用hadoop读取文件的方式
		//totalSize：表示文件所占据的字节数从，7字节
		//7/2=3字节...1字节，剩余的字节数占据分区数量的多少，如果大于1.1就产生新的分区，否则不产生新的分区，所以产生新的分区
		//val goalSize: Long = totalSize / (if (numSplits == 0) 1 else numSplits)

		//分区确定后，数据是如何划分的

		val rdd = context.textFile("datas/data",3)

		rdd.saveAsTextFile("output")

		//	关闭环境
		context.stop()
	}

}
~~~

**如何读取数据到切片**

~~~ java
object RDDFile__ {
	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("RDD")
		//	创建上写文
		val context = new SparkContext(conf)

		//数据分区的分配
		//分区确定后，数据是如何划分的
		// 1 数据是以行为单位进行读取，和字节数没有关系，
		// 2 数据读取以偏移量为单位
		//3 偏移量不会被重复读取
		/**				偏移地址
		 * 1@@   0,1,2
		 * 2@@   3，4,5
		 * 3     6
		 */
		//	一共两个分区，每一个分区3字节：
		//	0 [0 0+3]=>1,2
		//	闭区间，并且按照行读取
		//	1 [3,3+3=6]=》3
		//	2 [6,7] =》空
		//

		val rdd = context.textFile("datas/data",3)

		rdd.saveAsTextFile("output")

		//	关闭环境
		context.stop()
	}

}

~~~

**读取数据案例**

![1614317020653](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/132342-932759.png)

14字节的数据量，数据分两片，每一片7字节的数据。

如果数据源有多个文件，那么计算分区时候是以文件为单位的。

#### RDD方法分类

转换：功能的补充和封装，将旧的RDD包装为新的RDD，比如map,flatmap，这一步会生成一个图

行动：触发任务的调度和作业的执行，比如collect，触发生成的图的执行。

RDD方法=>算子

**从功能的角度分类**

- 转换算子
- 行动算子

**RDD存放数据类型分类**

- String,对象类型
- key-value类型
- 数值类型

##### 对key-value数据类型的支持

**对键值对类型的数据支持的方法全部在下面的类中**

~~~ java
class PairRDDFunctions[K, V](self: RDD[(K, V)])
    (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null)
  extends Logging with Serializable {}
~~~

那么为什么RDD数据可以使用这些方法呢？

~~~ java
implicit def rddToPairRDDFunctions[K, V](rdd: RDD[(K, V)])
    (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null): PairRDDFunctions[K, V] = {
    new PairRDDFunctions(rdd)
  }
//在RDD的半生对象中有一个隐式转换，当我们传进来的数据是键值对类型的时候，会自动给我们new出来一个PairRDDFunctions对象
~~~

![1621923288333](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/141751-75409.png)

##### 对数字型数据的支持

下面是对RDD里面数据类型是数值型的数据的支持操作。

![1621923530126](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/142150-785867.png)

这些对数据类型操作的支持都是action操作。

##### **RDD特点**

- rdd不仅是数据集，而且还是编程模型

![1616723901576](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/080724-389059.png)

最初的所有转换仅仅是形成一条数据流，不会真正的做计算操作，而是在真正的行动算子触发之后，才会去做计算操作。

- RDD可以进行分区，因为spark是一个分布式的并行计算框架，只有进行分区操作后，才可以进行并行计算

![1616723956725](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/101605-430662.png) 

- RDD是只读的

![1616723714723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/095552-537657.png)

RDD一旦生成，就无法改变。如果支持次改，那么很明显是需要保存数据的，这明显违背RDD设计的初衷。

- RDD是可以容错的

![1621748272365](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/133756-221332.png)

##### 什么是弹性分布式数据集

![1616725004784](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/102723-7165.png)

##### RDD属性小结

![1616725674853](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/125122-945542.png)

- 为什么RDD要有分区？因为RDD要进行并行计算
- RDD要记录依赖关系？因为要进行容错，有了算子之间的依赖关系，如何从一个RDD转换到另一个RDD，这就需要使用到计算函数，所以要记录计算函数。
- Partitioner是为了进行shuffed，只有在键值对类型的数据级中才有Partitioiner。
- 尽量把RDD的分区调度到数据存放的位置，这是一个简单的优化，移动数据不如移动计算。

#### RDD算子概述

![1621827782938](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/114304-893306.png)

**Transformation 和 Action**

RDD 的操作主要可以分为 Transformation 和 Action 

![1621766949865](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165959-547361.png)

- Transformation转换操作：
  - 返回一个新的RDD,which create a new dataset from an existing one
  - 所有Transformation函数都是Lazy，不会立即执行，需要Action函数触发
- Action动作操作：返回值不是RDD(无返回值或返回其他的)
  - which return a value to the driver program after running a computation on the datase
  - 所有Action函数立即执行（Eager），比如count、first、collect、take等

![1621767065273](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170003-975656.png)

> Notice
>
> 1. RDD不实际存储真正要计算的数据，而是记录了数据的位置在哪里，数据的转换关系(调用了什么方法，传入什么函数)；
>
> 2. RDD中的所有转换都是惰性求值/延迟执行的，也就是说并不会直接计算。只有当发生Action动作时，这些转换才会真正运行。之所以这样，是因为这样可以在Action时对RDD操作形成DAG有向无环图进行Stage的划分和并行优化，这种设计可以让Spark更加高效地运行。

##### 转换算子

- 在Spark中Transformation操作表示将一个RDD通过一系列操作变为另一个RDD的过程，这个操作可能是简单的加减操作，也可能是某个函数或某一系列函数。

- 值得注意的是Transformation操作并不会触发真正的计算，只会建立RDD间的关系图。

**常用Transformation转换函数：**

![1621767227097](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170011-695081.png)

**算子说明**

| 转换                                                   | 含义                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| map(func)                                              | 返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成 |
| filter(func)                                           | 返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成 |
| flatMap(func)                                          | 类似于map，但是每一个输入元素可以被映射为0或多个输出元素(所以func应该返回一个序列，而不是单一元素) |
| mapPartitions(func)                                    | 类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator[T] => Iterator[U] |
| mapPartitionsWithIndex(func)                           | 类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是   (Int,   Interator[T]) => Iterator[U] |
| sample(withReplacement,   fraction, seed)              | 根据fraction指定的比例对数据进行采样，可以选择是否使用随机数进行替换，seed用于指定随机数生成器种子 |
| union(otherDataset)                                    | 对源RDD和参数RDD求并集后返回一个新的RDD                      |
| intersection(otherDataset)                             | 对源RDD和参数RDD求交集后返回一个新的RDD                      |
| distinct([numTasks]))                                  | 对源RDD进行去重后返回一个新的RDD                             |
| groupByKey([numTasks])                                 | 在一个(K,V)的RDD上调用，返回一个(K, Iterator[V])的RDD        |
| reduceByKey(func,   [numTasks])                        | 在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，与groupByKey类似，reduce任务的个数可以通过第二个可选的参数来设置 |
| aggregateByKey(zeroValue)(seqOp,   combOp, [numTasks]) |                                                              |
| sortByKey([ascending],   [numTasks])                   | 在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD |
| sortBy(func,[ascending],   [numTasks])                 | 与sortByKey类似，但是更灵活                                  |
| join(otherDataset,   [numTasks])                       | 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD |
| cogroup(otherDataset,   [numTasks])                    | 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD |
| cartesian(otherDataset)                                | 笛卡尔积                                                     |
| pipe(command,   [envVars])                             | 对rdd进行管道操作                                            |
| coalesce(numPartitions)                                | 减少 RDD 的分区数到指定值。在过滤大量数据之后，可以执行此操作 |
| repartition(numPartitions)                             | 重新给 RDD 分区                                              |

##### 行动算子

- 不同于Transformation操作，Action操作代表一次计算的结束，不再产生新的 RDD，将结果返回到Driver程序或者输出到外部。

- 所以Transformation操作只是建立计算关系，而Action 操作才是实际的执行者。

- 每个Action操作都会形成一个DAG并调用SparkContext的runJob 方法向集群正式提交请求，所以每个Action操作对应一个DAG/Job。

**常用Action执行函数**

![1621767347320](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170019-441768.png)

**说明**

| 动作                                    | 含义                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| reduce(func)                            | 通过func函数聚集RDD中的所有元素，这个功能必须是可交换且可并联的 |
| collect()                               | 在驱动程序中，以数组的形式返回数据集的所有元素               |
| count()                                 | 返回RDD的元素个数                                            |
| first()                                 | 返回RDD的第一个元素(类似于take(1))                           |
| take(n)                                 | 返回一个由数据集的前n个元素组成的数组                        |
| takeSample(withReplacement,num, [seed]) | 返回一个数组，该数组由从数据集中随机采样的num个元素组成，可以选择是否用随机数替换不足的部分，seed用于指定随机数生成器种子 |
| takeOrdered(n, [ordering])              | 返回自然顺序或者自定义顺序的前 n 个元素                      |
| saveAsTextFile(path)                    | 将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本 |
| saveAsSequenceFile(path)                | 将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。 |
| saveAsObjectFile(path)                  | 将数据集的元素，以 Java 序列化的方式保存到指定的目录下       |
| countByKey()                            | 针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。 |
| foreach(func)                           | 在数据集的每一个元素上，运行函数func进行更新。               |
| foreachPartition(func)                  | 在数据集的每一个分区上，运行函数func                         |

##### 常用算子

![1621854244279](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/190406-805960.png)

#### RDD 转换算子

RDD 根据数据处理方式的不同将算子整体上分为Value 类型、双 Value 类型和Key-Value类型

RDD中map、filter、flatMap及foreach等函数为最基本函数，都是对RDD中每个元素进行操作，将元素传递到函数中进行转换。

##### **Value** **类型**                                       
###### map

map的转换为一对一的转换。map(f:T=>U) :RDD[T]=>RDD[U]，表示将 RDD 经由某一函数 f 后，转变为另一个RDD。

![1621854536366](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/190858-218627.png)

函数签名

~~~ java
def map[U: ClassTag](f: T => U): RDD[U]
~~~

函数说明：将处理的数据**逐条**进行映射转换，这里的转换可以是**类型**的转换，也可以是**值**的转换。要和mapValue进行区分。输入数据类型可以和输出数据类型不一样。

~~~ java
object Spark_RDD_transform {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5))
		//转换函数
		def mapFun(num:Int):Int={
			num*2
		}
		//对rdd中的数据*2
		//这样每一次传进来一个函数很不方便，所以我们传进来匿名函数
		//val mapRdd:RDD[Int] = rdd.map(mapFun)
		//使用匿名函数
		val mapRdd:RDD[Int] = rdd.map((num:Int)=>{
			num*2
		})
      
      	//对返回的数据在做一次过滤操作，过滤出偶数
		val res = mapRdd.filter((num: Int) => {
			num % 2 == 0
		})
      //调用collect()方法后返回的是一个数组，数组是scala中的数据类型
		res.collect().foreach(println)
		mapRdd.collect().foreach(println)
		context.stop()
	}
}

~~~

- 小功能：从服务器日志数据 apache.log 中获取用户请求URL 资源路径

~~~ java
object Spark_RDD_transform_log {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	val rdd = context.textFile(path = "datas/apache.log")

		//长的字符串转换为短的字符串
		val mapRdd:RDD[String] = rdd.map((line: String) => {
			//	切分数据
			val words = line.split(" ")
			//	返回第六条数据
			words(6)
		})
		mapRdd.collect().foreach(println)
		context.stop()
	}
}
~~~

**并行度**

~~~ java
object Spark_RDD_transform_par {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		//	TODO 算子

		//rdd的计算一个分区内的数据是一个一个执行逻辑
		//只有当前面的一个数据的逻辑全部执行完毕之后，才会执行下一个逻辑
		//分区内数据的执行是有序的
		//不同分区之间数据的执行是无顺序的，相同分区内数据执行有序
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)

		//val mapRdd:RDD[Int]=rdd.map((num:Int)=>
		//{
		//	println(">>>>>>>>>"+num)
		//	num
		//})
		//
		//val mapRdd1:RDD[Int]=rdd.map((num:Int)=>
		//{
		//	println("^^^^^^^^^"+num)
		//	num
		//})
		mapRdd1.collect()
		context.stop()
	}
}
~~~

###### foreach

foreach(func)，将函数func 应用在数据集的每一个元素上。关于 foreach，在后续章节中还会使用，到时会详细介绍它的使用方法及注意事项。

###### **saveAsTextFile** 

saveAsTextFile(path:String)，数据集内部的元素会调用其 toString 方法，转换为字符串形式，然后根据传入的路径保存成文本文件，既可以是本地文件系统，也可以是HDFS 等。

###### mapValues

mapValue只对键值类型的值进行转换，不会改变键,和map()算子使用方法一样。

**函数签名**

~~~ java
 def mapValues[U](f: V => U): RDD[(K, U)] 
 //泛型表示键值对value的类型
~~~

**代码演示**

~~~ java
def testMapValue():Unit={
		val rdd: RDD[(Char, Int)] = context.parallelize(Seq(('a', 1), ('b', 2), ('c', 3)))

		val mapValueRdd: RDD[(Char, Int)] = rdd.mapValues {
			data => {//data表示键值对的值，
				data * 10
			}
		}
		mapValueRdd.collect().foreach(print)
	}
~~~

> map和mapValue都之作用与单条数据，但是mapValue只对键值对的值进行操作，不会改变键值。

###### mapPartitions 

每个RDD由多分区组成的，实际开发中如果涉及到资源相关操作建议对每个分区进行操作，即使用mapPartitions代替map函数使用foreachPartition代替foreache函数

- mapPartitions 操作是在内存中进行操作，所以性能比较高，并且mapPartitions 方法会把一个分区中的数据全部加载完成之后全部进行处理，所以效率比map函数高很多，而map是对每一条数据进行操作。比如访问数据库操作，如果使用map的话，每次访问都需要建立链接，很浪费资源，如果使用mapPartitions （）算子，就可以以分区为单位进行建立链接访问数据。
- mapPartitions 可以以分区为单位进行数据的转换操作，会将整个分区中的数据加载到内存中引用，但是使用完的数据不会被释放掉，当数据量比较多，内存较小的情况下，容易出现内存溢出，这种情况使用map函数比较好。
- 有时候可以以分区为单位对数据进行访问操作，减少内存的开销，比如访问数据库中的数据，如果对单条数据进行访问，那么每一条数据都要建立一个链接，但是如果对数据进行分区操作，那么每一个分区建立一个链接即可。
- map和mapPartitions 之间最大的区别是，map操作数据最小的粒度是每一条数据，而mapPartitions 操作数据的最小粒度是分区。

**函数签名**

~~~ java
def mapPartitions[U: ClassTag](
  //其中函数中的参数是一个迭代器
f: Iterator[T] => Iterator[U],preservesPartitioning: Boolean = false): RDD[U]
~~~

- 说明
  - 将待处理的数据**以分区为单位**发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。 

**案例**

~~~ java
object Spark_RDD_transform_partition {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	设置分片的个数是2，也就是分区数是2个
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)
		//mapPartitions方法会把一个分区的数据全部拿到之后再做操作，而不是一个一个数据操作
		val mapRdd = rdd.mapPartitions(
      //里面的参数是一个迭代器，里面有很多元素，迭代器就相当于一个分区中的数据
			iter => {
				//对迭代器中的元素全部*2
				println("**************")
          //iter.map()是scala中的方法。map返回是一个迭代器
				iter.map(_ * 2)
			}
		)
		mapRdd.collect().foreach(println)
		context.stop()
	}
}

//下面打印每一个分区的最大值
object Spark_RDD_transform_mappartition {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	设置分片的个数是2
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)
		//mapPartitions方法会把一个分区的数据全部拿到之后再做操作，而不是一个一个数据操作
		val mapRdd = rdd.mapPartitions(
      //一个迭代器就相当于一个分区中的数据
			iter => {
				//返回每一个分区中的最大值
				//需要返回的是一个迭代器
				List(iter.max).iterator
			}
      }

		mapRdd.collect().foreach(println)
		context.stop()
	}
}
      
//案例二
object Test05 {

		def main(args: Array[String]): Unit = {
			//	创建SparkContext
			//	1 创建SparkConf配置文件
			val conf: SparkConf = new SparkConf().setMaster("local[6]").setAppName("sparkContext")

			//	2 创建SparkContext,是程序的入口，可以设置参数，创建rdd
			val sc = new SparkContext(conf)

			sc.parallelize(Seq(1,2,3,4,5,6),2)
				//mapPartitions()方法接受一个函数，函数的参数是一个集合,iter是一个集合
				.mapPartitions(iter =>{
					//iter.foreach(println)//对集合进行遍历
					//遍历每一个分区中的数据，对分区中的每一个数据做转换，转换完毕之后，返回iter集合
				//	iter是scala集合类型，下面的map()是scala中的函数
					val res: Iterator[Int] = iter.map(item => item * 10)
					res
				})
					.collect()
					.foreach(println)


			sc.parallelize(Seq(1,2,3,4,5,6),2)
					.mapPartitionsWithIndex((index,iter)=>{
						println("index"+index)
						iter.foreach(println)
						iter
					})
					.collect()


			sc.stop()
		}

}
      
 //案例三
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

/**
 * Author itcast
 * Desc 演示RDD的分区操作函数
 *
 */
object RDDDemo02_Partition{
  def main(args: Array[String]): Unit = {
    //1.准备环境(Env)sc-->SparkContext
    val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
    val sc: SparkContext = new SparkContext(sparkConf)
    sc.setLogLevel("WARN")

    //2.加载文件(Source)
    val fileRDD: RDD[String] = sc.textFile("data/input/words.txt")

    //3.处理数据(Transformation)
    val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))//_下划线表示每一行
    //map(函数),该函数会作用在每个分区中的每一条数据上,value是每一条数据
    /*val wordAndOneRDD: RDD[(String, Int)] = wordRDD.map(value =>{
      //开启连接-有几条数据就几次
      (value, 1)
      //关闭连接-有几条数据就几次
    })*/

    //mapPartitions(函数):该函数会作用在每个分区上,values是每个分区上的数据
    val wordAndOneRDD: RDD[(String, Int)] = wordRDD.mapPartitions(values => {
      //开启连接-有几个分区就几次
      values.map(value => (value, 1)) //value是该分区中的每一条数据
      //关闭连接-有几个分区就几次
    })

    val resultRDD: RDD[(String, Int)] = wordAndOneRDD.reduceByKey(_+_)

    //4.输出结果(Sink)
    //foreach(函数),该函数会作用在每个分区中的每一条数据上,value是每一条数据
    //Applies a function f to all elements of this RDD.
    /*resultRDD.foreach(value=>{
      //开启连接-有几条数据就几次
      println(value)
      //关闭连接-有几条数据就几次
    })*/

    //foreachPartition(函数),该函数会作用在每个分区上,values是每个分区上的数据
    //Applies a function f to each partition of this RDD.
    resultRDD.foreachPartition(values=>{
      //开启连接-有几个分区就几次
      values.foreach(value=>println(value))//value是该分区中的每一条数据
      //关闭连接-有几个分区就几次
    })

    //5.关闭资源
    sc.stop()
  }
}
~~~

小功能：获取每个数据分区的最大值 

这个题不可以使用map算子，因为map算子只是拿到一个一个的数据，具体的不知道数据的来源，但是mapPartitions 可以知道数据具体是来自于哪一个分区，可以在分区内对数据进行操作。

~~~ java
object Spark_RDD_transform_mappartition {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	设置分片的个数是2
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)
		//mapPartitions方法会把一个分区的数据全部拿到之后再做操作，而不是一个一个数据操作
		val mapRdd = rdd.mapPartitions(
			iter => {
				//返回每一个分区中的最大值
				//需要返回的是一个迭代器
				List(iter.max).iterator
			}
		)
		mapRdd.collect().foreach(println)
		context.stop()
	}
}

~~~

**map 和 mapPartitions 的区别 **

- 数据处理角度
  - Map 算子是分区内一个数据一个数据的执行，类似于串行操作。而 mapPartitions 算子是以分区为单位进行批处理操作。 map中参数是单条数据，mapPartitions参数中的是一个分区的数据，每一次针对一个分区的数据进行操作，效率比较高
  - map的fun返回的数据是单条，只是对每一条数据做一个转换操作，mapPartitions返回结果是一个集合，针对集合进行操作然后在返回，参数是一个迭代器，存储整个分区的数据。
- 功能的角度
  - Map 算子主要目的将数据源中的数据进行转换和改变。但是不会**减少或增多数据**。MapPartitions 算子需要传递一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变，所以可以增加或减少数据 
- 性能的角度
  - Map 算子因为类似于串行操作，所以性能比较低，而是 mapPartitions 算子类似于批处理，所以性能较高。但是 mapPartitions 算子会长时间占用内存，那么这样会导致内存可能不够用，出现内存溢出的错误。所以在内存有限的情况下，不推荐使用。使用 map 操作 

###### mapPartitionsWithIndex 

**函数签名**

~~~ java
def mapPartitionsWithIndex[U: ClassTag](
  //可以看到传进来函数的参数int表示分区索引号，第二个参数表示分区的值
f: (Int, Iterator[T]) => Iterator[U],
preservesPartitioning: Boolean = false): RDD[U]
~~~

**函数说明**

将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。 

~~~ java
object Spark_RDD_transform_mapPartitionsWithIndex {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	设置分片的个数是2
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)
		//mapPartitions方法会把一个分区的数据全部拿到之后再做操作，而不是一个一个数据操作
		val mapRdd = rdd.mapPartitionsWithIndex(
			(index,iter) => {
				if(index ==1){
					iter
				}else{
					Nil.iterator
				}
			}
		)
		mapRdd.collect().foreach(println)
		context.stop()
	}
}
//如果在这里吧两个分区的数据全部打印，分区中的数据会顺序输出，这是因为两个分区的算子是并行执行的，
~~~

查看数据的分区

~~~ java
object Spark_RDD_transform_mapPartitionsWithIndex_ {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	设置分片的个数是2
		val rdd = context.makeRDD(List(1, 2, 3, 4, 5),2)
		//mapPartitions方法会把一个分区的数据全部拿到之后再做操作，而不是一个一个数据操作
		val mapRdd = rdd.mapPartitionsWithIndex(
			(index,iter) => {
			iter.map(
				//查看每一个数字所在的分区号
				num=>{
					(index,num)
				}
			)
			}
		)
		mapRdd.collect().foreach(println)
		context.stop()
	}
}
//案例二
object Test05 {

		def main(args: Array[String]): Unit = {
			//	创建SparkContext
			//	1 创建SparkConf配置文件
			val conf: SparkConf = new SparkConf().setMaster("local[6]").setAppName("sparkContext")

			//	2 创建SparkContext,是程序的入口，可以设置参数，创建rdd
			val sc = new SparkContext(conf)

			sc.parallelize(Seq(1,2,3,4,5,6),2)
				//mapPartitions()方法接受一个函数，函数的参数是一个集合,iter是一个集合
				.mapPartitions(iter =>{
					//iter.foreach(println)//对集合进行遍历
					//遍历每一个分区中的数据，对分区中的每一个数据做转换，转换完毕之后，返回iter集合
				//	iter是scala集合类型，下面的map()是scala中的函数
					val res: Iterator[Int] = iter.map(item => item * 10)
					res
				})
					.collect()
					.foreach(println)


			sc.parallelize(Seq(1,2,3,4,5,6),2)
					.mapPartitionsWithIndex((index,iter)=>{
						println("index"+index)
						iter.foreach(println)
						iter
					})
					.collect()


			sc.stop()
		}
}
~~~

> mapPartitions和mapPartitionsWithIndex最大的区别是mapPartitionsWithIndex的参数中多出一个分区号，可以针对指定的分区进行操作。

###### flatmap

flatmap是一对多的转换算子。flatMap(f:T=>Seq[U]): RDD[T]=>RDD[U])，表示将 RDD 经由某一函数 f 后，转变为一个新的 RDD，但是与map 不同，RDD 中的每一个元素会被映射成新的 0 到多个元素（f 函数返回的是一个序列 Seq）。

- 首先把RDD中的数据转换为数组或者集合的形式
- 把集合或者数组展开处理
- 生成多条数据

![1621854660110](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/191108-119152.png)

**函数签名**

~~~ java
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]
~~~

**函数说明**

将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射 

~~~ java
object Spark_RDD_transform_flatmap {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd:RDD[List[Int]] = context.makeRDD(List(List(1,2),List(3,4)))

		val flatAdd = rdd.flatMap(
			//返回一个集合，集合中的元素是int类型
			List => {
				List
			}
		)

		val str:RDD[String]=context.makeRDD(List("hello wprd","hello scala"))
		val word = str.flatMap(
			s => {
				//返回的结果是一个可以迭代的集合
				s.split(" ")
			}
		)
		word.collect().foreach(println)

		//flatAdd.collect().foreach(println)//返回1,2,3,4
		context.stop()
	}
}

object Test03 {

		def main(args: Array[String]): Unit = {
	//	创建SparkContext
	//	1 创建SparkConf配置文件
	val conf: SparkConf = new SparkConf().setMaster("local[6]").setAppName("sparkContext")

	//	2 创建SparkContext,是程序的入口，可以设置参数，创建rdd
	val sc = new SparkContext(conf)

	//		创建RDD，通过本地集合创建RDD
	val value: RDD[String] = sc.parallelize(Seq("hello spark", "hello java", "hello word"))

			val value1: RDD[String] = value.flatMap((str: String) => {
				str.split(" ")
			})
			val strings: Array[String] = value1.collect()
			value1.foreach(println)
			//strings.foreach(println)
			sc.stop()
	}
~~~

将 List(List(1,2),3,List(4,5))进行扁平化操作 

~~~ java
val data=context.makeRDD(List(List(1,2),3,List(4,5)))
		val dataFlat = data.flatMap(data => {
      //使用的是模式匹配
			data match {
				case list: List[_] => list
				case dat => List(dat)
			}
		})


dataFlat.collect().foreach(println)
~~~

###### glom 

**函数签名**

~~~ java
def glom(): RDD[Array[T]]
~~~

**函数说明**

将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变 

也就是某一个分区的数据转换后还是在自己的分区，不会发送到其他的分区，转换操作不会影响分区。

![1614478270369](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/28/101112-283315.png)

**案例**

~~~ java
object Spark_RDD_transform_glom {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd:RDD[Int] = context.makeRDD(List(1,2,3,4,5),2)
		//list => int
		//int => array

		val glomRdd:RDD[Array[Int]] = rdd.glom()

		glomRdd.collect().foreach(data=>{
			println(data.mkString(","))})
		//返回两个分区的结果
		//1,2
		//3,4,5
		context.stop()
	}
}

~~~

计算所有分区最大值求和（分区内取最大值，分区间最大值求和） 

~~~ java
object Spark_RDD_transform_glom_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd:RDD[Int] = context.makeRDD(List(1,2,3,4,5),2)

		//每一个分区当做一个数组
		val rddglom:RDD[Array[Int]] = rdd.glom()
		//获取每一个数组中的最大值
		val maxRdd:RDD[Int] = rddglom.map(array => {
			array.max
		})

		//求和
		println(maxRdd.collect().sum)
		context.stop()
	}
}
~~~

###### groupBy 

**函数签名**

~~~ java
def groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
~~~

**函数说明**

将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为 shuffle。极限情况下，数据可能被分在同一个分区中一个组的数据在一个分区中，但是并不是说一个分区中只有一个组 

**案例**

~~~ java
object Spark_RDD_transform_groupBy {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd:RDD[Int] = context.makeRDD(List(1,2,3,4,5),2)


		//	groupBy会将数据源中的数据分组逐个判断，根据返回的分组key进行分组操作
		//相同的key分到一个组中

		def groupFunction(num:Int):Int={
		//		按照num%2进行分组
		num%2
		}
		val groupRdd:RDD[(Int,Iterable[Int])] = rdd.groupBy(groupFunction)

		groupRdd.collect().foreach(println)
		context.stop()
	}
}

~~~

小功能： 将 List("Hello", "hive", "hbase", "Hadoop")根据单词首写字母进行分组。 

~~~ java
object Spark_RDD_transform_groupBy_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd = context.makeRDD(List("hello","scala","hadoop","spark"))

		//按照单词的开头字母进行分组表，使用匿名函数
		val groupRdd = rdd.groupBy(_.charAt(0))
		groupRdd.collect().foreach(println)
		context.stop()
	}
}

~~~

> 分组和分区没有必然的关系，也就是groupBy操作不会改变分区的个数，但是所有分区的数据会重新打乱重分配。

![1614480400951](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170048-581398.png)

从服务器日志数据 apache.log 中获取每个时间段访问量 

~~~ java
object Spark_RDD_transform_groupBy__ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd = context.textFile("datas/apache.log")

		//val timeRdd:RDD[(String,Iterable[Int])] = rdd.map(
		//	line => {
		//		val datas = line.split(" ")
		//		val time = datas(3)
		//		val format = new SimpleDateFormat("dd/MM/dd:HH:mm:ss")
		//		val date: Date = format.parse(time)
		//		val stdf = new SimpleDateFormat("HH")
		//		val hour = stdf.format(time)
		//		(hour, 1)
		//	}
		//).groupBy(_._1)

		//timeRdd.map {
		//	case (hour, iter) => {
		//		(hour, iter.size)
		//	}
		//}.collect().foreach(println)


		context.stop()
	}
}
~~~

###### Filter

filter(f:T=>Bool) : RDD[T]=>RDD[T]，表示将 RDD 经由某一函数 f 后，只保留f 返回为 true 的数据，组成新的 RDD。

![1621854730793](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/191213-651610.png)

**函数签名**

~~~ java
def filter(f: T => Boolean): RDD[T]
~~~

**函数说明**

- 将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。
- 当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。 
- 当返回true的时候，会保留数据，当返回false的时候，会过滤掉数据

~~~ java
object Spark_RDD_transform_filter {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	现在需要把RDD中的集合list进行打散操作
		val rdd:RDD[Int] = context.makeRDD(List(1,2,3,4,5),2)

		val filterRdd:RDD[Int] = rdd.filter(num => {
			//把奇数过滤出来
			num % 2 != 0
		})

		filterRdd.collect().foreach(println)
		context.stop()
	}
}

~~~

从服务器日志数据 apache.log 中获取 2015 年 5 月 17 日的请求路径 

~~~ java
object Spark_RDD_transform_filter_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd:RDD[String] = context.textFile("datas/apache.log")

		rdd.filter(
			line=>{
				val words = line.split(" ")
				val time=words(3)
				time.startsWith("17/05/2015")
			}
		).collect().foreach(println)
		context.stop()
	}
}
~~~

###### sample 

- 把大数据集变为小的数据集
- 同时尽量减少对数据级特征的损失
- 和filter的区别是filter会按照一定的规律，过滤掉一部分数据集，但是sample过滤数据没有规律，通过随机抽样算法获取数据。

**函数签名**

~~~ java
def sample(
withReplacement: Boolean,//是否采取有放回的抽样，true是有放回的抽样
fraction: Double,//采样的比例
seed: Long = Utils.random.nextLong): RDD[T]//随机数种子
~~~

**函数说明**

根据指定的规则从数据集中抽取数据 ，把大数据集变为小数据集，尽量减少数据特征信息的损失。抽样过程是随机的。

~~~ java
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
// 抽取数据不放回（伯努利算法）
// 伯努利算法：又叫 0、 1 分布。例如扔硬币，要么正面，要么反面。
// 具体实现：根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不要
// 第一个参数：抽取的数据是否放回， false：不放回
// 第二个参数：抽取的几率，范围在[0,1]之间,0：全不取； 1：全取；
// 第三个参数：随机数种子
val dataRDD1 = dataRDD.sample(false, 0.5)
// 抽取数据放回（泊松算法）
// 第一个参数：抽取的数据是否放回， true：放回； false：不放回
// 第二个参数：重复数据的几率，范围大于等于 0.表示每一个元素被期望抽取到的次数
// 第三个参数：随机数种子
val dataRDD2 = dataRDD.sample(true, 2)
~~~

**案例**

~~~ java
object Spark_RDD_transform_sample {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd = context.makeRDD(List(1,2,3,4,5,6,7,8,9,0))
		//从rdd数据及中抽取数据
		/**
		 * 第一个参数表示抽取数据后，是否放回，true表示放回，false表示不放回
		 * 		如果抽取放回的场合，表示数据源中每一条数据可能被抽取的次数
		 * 		如果抽取不放回，那么表示数据源中每一条数据抽取到的概率
		 * 第二个参数表示数据源中每一条数据被抽取的概率
		 * 第三个参数表示抽取数据时候随机算法的种子
		 * 随机数种子确定之后，每一次抽取的数据是一样的，如果不传种子的值，那么每一次就是随机产生，种子使用的是当前的系统时间确定
		 */
		//val value = rdd.sample(false, 0.4, 1)

		val value = rdd.sample(true, 2, 1)

		println(value.collect().mkString(","))

		//rdd.collect().foreach(println)
		context.stop()
	}
}
~~~

###### distinct 

**函数签名**

~~~ java
def distinct()(implicit ord: Ordering[T] = null): RDD[T]
def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
~~~

**函数说明**

将数据集中重复的数据去重 

~~~ java
object Spark_RDD_transform_distinct {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd = context.makeRDD(List(1,2,3,4,5,5,4,6,1,2))

		//去重源码
		//case _ => map(x => (x, null)).reduceByKey((x, _) => x, numPartitions).map(_._1)
		//、第一步：(1, null)(2, null)(3, null)(4, null)(5, null)(5, null)(1, null)(3, null)
		//第二部：做聚合操作，(1, null)(1, null)相同的key使用v来做聚合(null,null)->null最终得到(1,null)
		//第三部：map操作只拿第一个位置的元素
		val res:RDD[Int] = rdd.distinct()

		res.collect().foreach(println)
		context.stop()
	}
}
~~~

###### coalesce 

注意: coalesce的shuffle参数默认为false,不会产生Shuffle，默认只能减少分区

![1621919650802](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/131413-327924.png)

**函数签名**

~~~ java
def coalesce(numPartitions: Int, shuffle: Boolean = false,
partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
(implicit ord: Ordering[T] = null)
: RDD[T]
shuffle：是否shuffle，如果新的分区数量比旧的分区数量多，必须进行shuffle的，否则分区无效
~~~

**函数说明**

- 根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率
- 当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少分区的个数，减小任务调度成本 
- 默认情况下只能把分区变小，不可以增大分区数，repartition可以把分区改大或者改小

**案例**

~~~ java
object Spark_RDD_transform_coalesce {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd = context.makeRDD(List(1,2,3,4,5,6),3)
		//上面读取数据划分4个分区，下面缩减到2个分区

		/**
		 * coalesce默认情况下不会把数据打乱重新分组的
		 * 这种情况下缩减分区可能导致数据发生不均衡，出现数据倾斜
		 * 如果想要数据均衡，可以进行shuffer处理，打乱重新组合
		 * coalesce第二个参数就表示是否进行shuffer操作，默认情况下不进行shuffer处理
		 */
//把shuffle参数设置为true，就可以增大分区数，因为可以进行shuffle操作
		val res:RDD[Int] = rdd.coalesce(2,true)
      val res:RDD[Int] = rdd.coalesce(5,true)

		//以分区文件的形式进行保存
		res.saveAsTextFile("out")
		context.stop()
	}
}
~~~

默认情况下不会进行shuffer处理，也就是不会把一个分区中的数据打乱重新分配，但是这种情况可能发生数据倾斜

![1614748100169](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/130822-396981.png)

如果我们想要数据分布均匀，尽量不发生数据倾斜，就可以设置第二个参数是true，就会进行shuffer操作，打乱重拍

![1614748191594](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/130953-723569.png)

**扩大分区**

如果想扩大分区，必须把shuffer设置为true

~~~ java
object Spark_RDD_transform_coalesce {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
		val rdd = context.makeRDD(List(1,2,3,4,5,6),3)
		//上面读取数据划分4个分区，下面缩减到2个分区

		/**
		 * coalesce默认情况下不会把数据打乱重新分组的
		 * 这种情况下缩减分区可能导致数据发生不均衡，出现数据倾斜
		 * 如果想要数据均衡，可以进行shuffer处理，打乱重新组合
		 * coalesce第二个参数就表示是否进行shuffer操作，默认情况下不进行shuffer处理
		 * coalesce算子可以扩大分区，但是如果不设置shuffer操作是没有 意义的，只有shuffer设置为true才会
		 * 把数据全部打乱重新分配
		 * spark提供了一个简化的操作
		 * 	如果缩减分区，使用coalesce算子
		 * 	如果想使得数据均衡，可以使用shuffer
		 *如果想要扩大分区，就使用repartition，其底层就是把shuffer设置为truecoalesce(numPartitions, shuffle = true)
		 * 
		 */

		//val res:RDD[Int] = rdd.coalesce(4,true)

		val value = rdd.repartition(4)
		//以分区文件的形式进行保存
		value.saveAsTextFile("out")
		context.stop()
	}
}

~~~

###### repartition 

注意: repartition底层调用coalesce(numPartitions, shuffle=true)会产生Shuffle

**函数签名**

~~~ java
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
//参数表示分区的个数
~~~

**函数说明**

该操作内部其实执行的是 coalesce 操作，参数 shuffle 的默认值为 true。无论是将分区数多的RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD， repartition操作都可以完成，因为无论如何都会经 shuffle 过程。 

~~~~~~ java
object Spark_RDD_transform_coalesce {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子，在这里分区数量是3个
		val rdd = context.makeRDD(List(1,2,3,4,5,6),3)
		//上面读取数据划分4个分区，下面缩减到2个分区

		/**
		 * coalesce默认情况下不会把数据打乱重新分组的
		 * 这种情况下缩减分区可能导致数据发生不均衡，出现数据倾斜
		 * 如果想要数据均衡，可以进行shuffer处理，打乱重新组合
		 * coalesce第二个参数就表示是否进行shuffer操作，默认情况下不进行shuffer处理
		 * coalesce算子可以扩大分区，但是如果不设置shuffer操作是没有 意义的，只有shuffer设置为true才会
		 * 把数据全部打乱重新分配
		 * spark提供了一个简化的操作
		 * 	如果缩减分区，使用coalesce算子
		 * 	如果想使得数据均衡，可以使用shuffer
		 *如果想要扩大分区，就使用repartition，其底层就是把shuffer设置为truecoalesce(numPartitions, shuffle = true)
		 * 
		 */

		//val res:RDD[Int] = rdd.coalesce(4,true)
//重新设置分区为4个，因为增大分区个数，所以有shuffle操作 
		val value = rdd.repartition(4)
		//以分区文件的形式进行保存
		value.saveAsTextFile("out")
		context.stop()
	}
}

~~~~~~

思考一个问题： coalesce 和 repartition 区别？ 

**案例二**

~~~ java
object Test07 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		val rdd1: RDD[String] = sc.parallelize(List("hadoop", "spark", "flink"), 4)
		println(rdd1.getNumPartitions) //4

		//注意：重分区:改变的是新生成的RDD的分区数,原来的RDD的分区数不变
		//-1.repartition,可以增加和减少分区
		val rdd2: RDD[String] = rdd1.repartition(5)
		println(rdd1.getNumPartitions) //4,原来rdd的分区个数
		println(rdd2.getNumPartitions) //5 新生成的RDD的分区的个数

		val rdd3: RDD[String] = rdd1.repartition(3)
		println(rdd1.getNumPartitions) //4
		println(rdd3.getNumPartitions) //3 ，只是改变新生成的RDD的分区的个数

		//-2.coalesce,默认只能减少分区
		val rdd4: RDD[String] = rdd1.coalesce(5)
		println(rdd1.getNumPartitions) //4
		println(rdd4.getNumPartitions) //4
		val rdd5: RDD[String] = rdd1.coalesce(3)
		println(rdd1.getNumPartitions) //4
		println(rdd5.getNumPartitions) //3


		//-3.默认按照key的hash进行分区
		val resultRDD: RDD[(String, Int)] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\word")
			.flatMap(_.split(" "))
			.map((_, 1)) //_表示每一个单词
			.reduceByKey(_ + _)

		resultRDD.foreachPartition(iter => {
			iter.foreach(t => {
				val key: String = t._1
				val num: Int = TaskContext.getPartitionId()
				println(s"默认的hash分区器进行的分区:分区编号:${num},key:${key}")
			})
		})

		//-4.自定义分区
		val resultRDD2: RDD[(String, Int)] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\word")
			.flatMap(_.split(" "))
			.map((_, 1)) //_表示每一个单词
			.reduceByKey(new MyPartitioner(1), _ + _)
		resultRDD2.foreachPartition(iter => {
			iter.foreach(t => {
				val key: String = t._1
				val num: Int = TaskContext.getPartitionId()
				println(s"自定义分区器进行的分区:分区编号:${num},key:${key}")
			})
		})

		sc.stop()
	}

	//自定义的分区器
	class MyPartitioner(partitions: Int) extends Partitioner {
		override def numPartitions: Int = partitions

		override def getPartition(key: Any): Int = {
			0
		}
	}
}
~~~



###### sortBy 

可以作用于任何数据类型的RDD，可以按照任何部分进行排序，需要和sortByKey区分开，sortByKey只能作用于k-v类型数据。

![1621919152931](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/131159-812466.png)

**函数签名**

~~~ java
def sortBy[K](
f: (T) => K,
  ascending: Boolean = true,
numPartitions: Int = this.partitions.length)
(implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]

f:通过这个函数返回需要排序的字段
ascending：是否升序排序
numPartitions:分区的个数
~~~

**函数说明**

该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理的结果进行排序，默认为升序排列。排序后新产生的 RDD 的分区数与原 RDD 的分区数一致。 中间存在 shuffle 的过程 

~~~ java
object Spark_RDD_transform_sortedBy {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

	//	TODO 算子
	//	val rdd = context.makeRDD(List(4,6,2,3,4,8,7),2)

		//对数据进行排序
		//使用匿名函数指定排序字段
		//val value = rdd.sortBy(num => num)

		//value.saveAsTextFile("sortFile")

		//默认是升序排列，第二个参数可以设置排序的方式
		//默认情况下，不会改变数据的分区，但是会存在shuffer,打乱数据，重新排列

		val rdd = context.makeRDD(List(("1",1),("0",0),("5",8),("3",9),("2",7),("6",6)),2)
		//按照字典来排排序
		val value = rdd.sortBy(num => num._1)//指定按照第一个位置的元素进行排序

		value.collect().foreach(println)

		context.stop()
	}
}

object Test14 {
	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val sc = new SparkContext(conf)
		val data: RDD[Int] = sc.parallelize(Seq(2, 54, 32, 76, 23, 1, 45, 78))

		//第一个参数表示指出排序的字段
		val sortData: RDD[Int] = data.sortBy(item => item,false)

		sortData.collect().foreach(println)

		sc.stop()
	}
}
~~~

##### 双 Value 类型 

双value类型其实就是我们两个数据源之间的关联操作

###### intersection 

**函数签名**

~~~ java
def intersection(other: RDD[T]): RDD[T]
~~~

**函数说明**

对源 RDD 和参数 RDD 求交集后返回一个新的 RDD 

~~~ java
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.intersection(dataRDD2)
~~~

并集要求两个数据源的数据类型要保持一致

###### union 

**函数签名**

def union(other: RDD[T]): RDD[T] 

**函数说明**

对源 RDD 和参数 RDD 求并集后返回一个新的 RDD 

~~~ java
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.union(dataRDD2)
~~~

###### subtract 

**函数签名**

~~~ java
def subtract(other: RDD[T]): RDD[T]
~~~

**函数说明**

以一个 RDD 元素为主， 去除两个 RDD 中重复元素，将其他元素保留下来。求差集 

~~~ java
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.subtract(dataRDD2)
~~~

###### zip

**函数签名**

~~~ java
def zip[U: ClassTag](other: RDD[U]): RDD[(T, U)]
~~~

**函数说明**

将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD中的元素， Value 为第 2 个 RDD 中的相同位置的元素。 

~~~ java
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.zip(dataRDD2)
~~~

- 如果两个 RDD 数据类型不一致怎么办？ 
  - 拉链操作两个数据源的数据类型可以不一样
- 如果两个 RDD 数据分区不一致怎么办？ 
  - 拉链操作必须保证两个数据源的分区一样
- 如果两个 RDD 分区数据数量不一致怎么办？ 
  - 拉链操作必须保证两个数据源的分区个数和每一个分区中的数据量一样

**案例**

~~~ java
object Spark_RDD_interSection {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		/**
		 * 交集，并集，差集要求两个数据源的数据类型必须保持一致
		 * 拉链操作两个数据源类型可以不一致
		 */
	//	TODO 算子
		val rdd1 = context.makeRDD(List(1, 2, 3, 4))
		val rdd2 = context.makeRDD(List(4,5,6,7))
		//计算交集4
		val res1:RDD[Int] = rdd1.intersection(rdd2)
		println(res1.collect().mkString(","))

		//计算并集1,2,3,4,4,5,6,7
		val res2:RDD[Int] = rdd1.union(rdd2)
		println(res2.collect().mkString(","))

		//计算差集1,2,3
		val res3:RDD[Int] = rdd1.subtract(rdd2)
		println(res3.collect().mkString(","))

		//计算拉链[(1-4),(2,5),(3,6),(4,7)]
		val res4:RDD[(Int,Int)] = rdd1.zip(rdd2)
		println(res4.collect().mkString(","))
		context.stop()
	}
}

~~~

**zip案例**

~~~ java
object Spark_RDD_interSection_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		/**
		 * 交集，并集，差集要求两个数据源的数据类型必须保持一致
		 * 拉链操作两个数据源类型可以不一致
		 */
	//	TODO 算子
		val rdd1 = context.makeRDD(List(1, 2, 3, 4),2)
		val rdd2 = context.makeRDD(List(4,5,6,7),4)

		/**
		 * 要求每一个分区中数据的数量要保持一致
		 * //两个数据源的分区数量需要保持一致
		 */
		val res = rdd1.zip(rdd2)
		println(res.collect().mkString(","))
		context.stop()
	}
}
~~~

##### Key - Value 类型 

也叫做聚合函数

没有key的聚合函数API如下

~~~ java
sum
reduce
fold
aggregate
底层都可以用aggregate实现
~~~

###### 聚合函数详解

在RDD中提供类似列表List中聚合函数reduce和fold，查看如下：

**reduce**

- 函数签名

~~~ java
def reduce(f: (T, T) => T): T
~~~

- 求列表List中元素之和，RDD中分区数目为2，核心业务代码如下：

~~~ java
object Test08 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		val data: RDD[Int] = sc.parallelize(1 to 10, 2)

		val sum: Int = data.reduce((curr, agg) => {
			println("curr=" + curr + "  agg=" + agg)
			curr + agg
		})
		println(sum)
	}
}
~~~

- 原理分析

![1621857288158](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/195506-757079.png)

**fold**

- 函数签名

~~~ java
def fold(zeroValue: T)(op: (T, T) => T): T
//比起reduce可以设置初始值
~~~

- 案例

~~~ java
object Test09 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		val data: RDD[Int] = sc.parallelize(1 to 10, 2)

		//设定初始值是10，只是参与一次运算
		val sum: Int = data.fold(10)((curr, agg) => {
			println("curr=" + curr + "  agg=" + agg)
			curr + agg
		})
		println(sum)
	}
}
~~~

**aggregate**

- 函数签名

~~~ java
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U 
~~~

![1621857652566](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/200057-492379.png)

- seqOp函数的第一个参数是累加器，第一次执行时，会把zeroValue赋给累加器。第一次之后会把返回值赋给累加器，作为下一次运算的第一个参数。

- seqOP函数每个分区下的每个key有个累加器，combOp函数全部分区有几个key就有几个累加器。如果某个key只存在于一个分区下，不会对他执行combOp函数

**案例**

业务需求：使用aggregate函数实现RDD中最大的两个数据，分析如下：

![1621858437745](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/201401-292938.png)

~~~ java
object Test10 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")


		val data: RDD[Int] = sc.parallelize(1 to 10, 2)

		//查看每一个分区的数据
		data.foreachPartition(iter =>{
			//println(TaskContext.getPartitionId()+" "+iter.foreach(println))
			println(s"p-${TaskContext.getPartitionId()}: ${iter.mkString(", ")}")
		})

		val top2: mutable.Seq[Int] = data.aggregate(new ListBuffer[Int]())(
			// 分区内聚合函数，每个分区内数据如何聚合  seqOp: (U, T) => U,
			(u, t) => {
				println(s"p-${TaskContext.getPartitionId()}: u = $u, t = $t")
				// 将元素加入到列表中
				u += t //
				// 降序排序
				val top = u.sorted.takeRight(2)
				// 返回
				top
			},
			// 分区间聚合函数，每个分区聚合的结果如何聚合 combOp: (U, U) => U
			(u1, u2) => {
				println(s"p-${TaskContext.getPartitionId()}: u1 = $u1, u2 = $u2")
				u1 ++= u2 // 将列表的数据合并，到u1中
				//
				u1.sorted.takeRight(2)
			}
		)
		println(top2)
	}
}
~~~

- 原理分析

![1621858409726](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/201334-356444.png)

![1621858508732](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/201714-42551.png)

**综合测试**

~~~ java
object Test11 {

	def main(args: Array[String]): Unit = {
		//1.准备环境(Env)sc-->SparkContext
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		//2.加载数据
		val nums: RDD[Int] = sc.parallelize( 1 to 10) //和55

		val result1: Double = nums.sum() //55//底层:fold(0.0)(_ + _)
		val result2: Int = nums.reduce(_+_)//55 注意:reduce是action ,reduceByKey是transformation
		val result3: Int = nums.fold(0)(_+_)//55
		//(zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U)
		//(初始值)(各个分区的聚合函数,多个分区结果的聚合函数)
		val result4: Int = nums.aggregate(0)(_+_,_+_)

		println(result1)
		println(result2)
		println(result3)
		println(result4)

		//5.关闭资源
		sc.stop()
	}
}
~~~

###### partitionBy

**函数签名**

~~~ java
def partitionBy(partitioner: Partitioner): RDD[(K, V)]
~~~

**函数说明**

将数据按照指定 Partitioner 重新进行分区。 Spark 默认的分区器是 HashPartitioner 

~~~ java
object Spark_RDD_partitionBy {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd:RDD[Int] = context.makeRDD(List(1, 2, 3, 4, 5, 6),2)

		val res:RDD[(Int,Int)] =rdd.map((_,1))
		//RDD=>PairRDDFunctions
		//隐士转换


		/**
		 * partitionBy不是RDD中的方法
		 * 把RDD数据转换为PairRDDFunctions是在RDD中有一个隐士转换，源码如下
		 * implicit def rddToPairRDDFunctions[K, V](rdd: RDD[(K, V)])
		 * (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null): PairRDDFunctions[K, V] = {
		 * new PairRDDFunctions(rdd)
		 * }
		 * //partitionBy根据指定的分区规则重新进行分区操作
		 * spark默认提供HashPartitioner进行分区
		 *
		 * 分区源码
		 * def getPartition(key: Any): Int = key match {
		 * case null => 0  如果key是null，就永远放在0号分区
		 * case _ => Utils.nonNegativeMod(key.hashCode, numPartitions)
		 * }
		 * def nonNegativeMod(x: Int, mod: Int): Int = {
		 * val rawMod = x % mod 取模运算分区
		 * rawMod + (if (rawMod < 0) mod else 0)
		 * }
		 *
		 */
		res.partitionBy(new HashPartitioner(2)).saveAsTextFile("outp")


		context.stop()
	}
}
~~~

- 如果重分区的分区器和当前 RDD 的分区器一样怎么办？ 
  - 这里的一样指的是分区器的类型和数量一样，那么此时会返回一个相同的重分区操作。
- Spark 还有其他分区器吗？ 

![1614760002534](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170208-620856.png)

可以看到还有另外一个Range分区器，这个分区器是在排序的时候使用。

- 如果想按照自己的方法进行数据分区怎么办？ 
  - 可以自定义分区器，见后文。

###### reduceByKey 

此算子还是一个分组聚合操作，不能做分组操作

reduceByKey的底层是combineByKey

- 首先对二元组类型进行byKey操作，也就是分组操作，在spark中二元元祖就是代表key-value类型
- 然后对分好的每一个组进行reduce操作

**函数签名**

~~~ java
def reduceByKey(func: (V, V) => V): RDD[(K, V)]
def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]
~~~

**函数说明**

可以将数据按照**相同**的 Key 对 Value 进行聚合 

此操作不会发生shuffle过程，因为reduce发生在每一个分组中，·并且不需要将所有的内容放在内存中，在执行最后的reduce之前所有的任务都是在每一个工作节点单独执行，大大提高执行的速度和稳定性。

~~~ java
object Spark_RDD_ResucerByKey {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("a", 3), ("b", 4)
		))
		//	对有相同键值类型的元祖做聚合操作

		/**
		 * reduceByKey:相同数据的key的值进行聚合操作
		 * scala语言中一般的聚合都是进行两两聚合，spark也是两两聚合
		 * [1,2,3]
		 * [3,3]
		 * [6] 两两聚合
		 * reduceByKey中如果数据只有一个，是不会参与运算的，直接返回结果
		 */
      //num2是局部的聚合结果，num1是当前键值对的值
		val value = rdd.reduceByKey((num1: Int, num2: Int) => {

			println(s"x=${num1},y=${num2}")
			num1 + num2
		})

		value.collect().foreach(println)
		context.stop()
	}
}

~~~

**原理说明**

![1621743019425](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170223-975374.png)

首先对每一个键值对进行分组操作，然后对每一个组分别进行reduce操作，curr代表当前组中每一个键值对的value，而agg代表局部的聚合结果，第一次,curr为1，局部聚合结果agg=0,第二部，curr=1,局部聚合结果agg=1，一次类推进行聚合操作。

###### groupByKey 

这个算子本质上是一个shuffer操作，reduceByKey本质上是一个reduce操作，

**原理**

![1635411816440](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170340-226995.png)

![1621831753634](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/125026-10073.png)

**函数签名**

~~~ java
def groupByKey(): RDD[(K, Iterable[V])]
def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]
def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]
~~~

**函数说明**

将数据源的数据根据 key 对 value 进行**分组** 

~~~ java
object Spark_RDD_groupRdd {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("a", 3), ("b", 4)
		))

		//groupBy相同key的数据，会分到一个组中,最终会形成一个元祖
		/**
		 * 元祖中的第一个元素就是key,第二个元素就是相同key的value集合
		 */
		val groupRdd:RDD[(String,Iterable[Int])] = rdd.groupByKey()

		groupRdd.collect().foreach(println)
		//(a,CompactBuffer(1, 2, 3))
		//(b,CompactBuffer(4))

		val groupRdd1:RDD[(String,Iterable[(String,Int)])] = rdd.groupBy(_._1)

		groupRdd1.collect().foreach(println)
		//(a,CompactBuffer((a,1), (a,2), (a,3)))
		//(b,CompactBuffer((b,4)))

		/**
		 * 上面两个分组函数的区别
		 * groupByKey
		 * 	1 会按照key进行分组，会把相同key的数据的value聚合在一起，、
		 结果形式是(key,(value1,value2....))
		 *
		 * groupBy
		 * 1 这个方法没有by key 进行分组，所以会把元祖整体进行分组操作
		结果形式:(key,(key,value),(key,value))
		 *
		 */
		context.stop()
	}
}
~~~

reduceByKey 和 groupByKey 的区别？ 

- reduceByKey 有分组的概念，相同的key分到一个组中，然后对value值做聚合操作。
- groupByKey 没有聚合的概念，仅仅是根据key对数据进行分组

reduceByKey 在map端做combiner是有意义的，因为在map做了分组聚合，就可以减少传到reduce端的数据量，减少了io操作

groupByKey 在map端做combiner是没有意义的，即使做分组，但是没有减少数据量，这些数据最终还是要发送到reducer端进行聚合操作。

reduceByKey 在map端有combiner，但是groupByKey 在map端没有combiner

**groupByKey**

![1614764721685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/174525-412785.png)

![1614764970507](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/175418-632654.png)

groupByKey会按照key把数据分组，但是每一个分区的数据量不一样，所以可能存在有的分组等待没有处理完数据的分区，这样在内存会积累大量的数据，可能存在内存溢出，所以中间要经过落盘操作，然后在加载到内存处理，所以有io操作，有损性能。只有分组，没有聚合操作，最终的聚合是通过map进行操作。

**reduceByKey **

![1614765385356](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614765385356.png)

最后又聚合操作，

![1614765569116](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170400-281859.png)

![1614765577376](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/180051-323968.png)

reduceByKey性能优于GroupByKeyDE的原因就在于reduceByKey在自己的分区内做了预聚合操作，使得io到磁盘上的数据量大大减小，这个功能叫做conbine操作。

- 从功能角度讲，reduceByKey是对数据两两进行聚合操作，如果不需要对数据进行聚合操作，只需要做分组，那么就使用groupByKey操作，两个的功能有所区别。

**小结**

- 从 shuffle 的角度： reduceByKey 和 groupByKey 都存在 shuffle 的操作，但是 reduceByKey可以在 shuffle 前对分区内相同 key 的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而 groupByKey 只是进行分组，不存在数据量减少的问题， reduceByKey 性能比较高。
- 从功能的角度： reduceByKey 其实包含分组和聚合的功能。 GroupByKey 只能分组，不能聚合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那么还是只能使用 groupByKey 
- 预聚合是发生在一个分区内，也就是在同一个分区内先做一次聚合处理，而落盘是在多个分区间进行的聚合操作，数据来自不同的分区。reduceByKey 在分区内和分区之间的聚合规则是一样的。

###### aggregateByKey 

![1616840367035](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/181929-532424.png)

![1621834637655](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/133722-712411.png)

**函数签名**

~~~ java
//可以指定一个初始值，后面两个参数是函数
def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U,combOp: (U, U) => U): RDD[(K, U)]
zeroValue：代表初始值
seqOp：是转换每一个值的函数，作用于每一个数据，根据初始值进行计算
combOp：将转换过的值进行聚合的函数，按照key进行聚合

//适合先对每一条数据进行处理，然后在分组聚合的应用
~~~

**函数说明**

所有按照key分组后的函数，在传进参数的时候传入的都是后面的value值，不需要传key

初始值会作用于每一个数据

将数据根据不同的规则进行**分区内计算和分区间计算** 

起始值在分区内和分区间的聚合都使用

~~~ java
// TODO : 取出每个分区内相同 key 的最大值然后分区间相加
// aggregateByKey 算子是函数柯里化，存在两个参数列表
// 1. 第一个参数列表中的参数表示初始值
// 2. 第二个参数列表中含有两个参数
// 2.1 第一个参数表示分区内的计算规则
// 2.2 第二个参数表示分区间的计算规则
val rdd =
sc.makeRDD(List(
("a",1),("a",2),("c",3),
("b",4),("c",5),("c",6)
),2)
// 0:("a",1),("a",2),("c",3) => (a,10)(c,10)
// => (a,10)(b,10)(c,20)
// 1:("b",4),("c",5),("c",6) => (b,10)(c,10)
val resultRDD =
rdd.aggregateByKey(10)(
(x, y) => math.max(x,y),
(x, y) => x + y
)
resultRDD.collect().foreach(println)
  
  object Spark_RDD_aggregateKey {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("a", 3), ("a", 4)
		))

		/**
		 * (a [1,2]) (a[3,4])
		 * (a [2]) (a[4])
		 * (a[6])
		 */

		/**
		 *aggregateByKey存在函数颗粒化，有两个参数列表
		 * 第一个参数列表：需要传递一个参数，初始值
		 * 		主要用于遇到第一个key的时候和value进行分区内计算
		 * 第二个参数列表需要传递两个参数
		 * 		第一个参数表示分区内分区规则
		 * 		第二个参数表示分区间分区规则
		 *
		 */

		val value = rdd.aggregateByKey(0)(
			(x, y) => {
				math.max(x, y)
			},
			(x, y) => {
				x + y
			}
		)

		value.collect().foreach(println)

		context.stop()
	}
}

//案例
object Test12 {

	def main(args: Array[String]): Unit = {
		//1.准备环境(Env)sc-->SparkContext
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		//2.加载数据
		val data = Seq(("华为", 10), ("小米", 6), ("电脑", 15))
		val rdd: RDD[(String, Int)] = sc.parallelize(data)
		//(zeroVal, item) => zeroVal * item:zeroVal代表的是初始值，item代表每一个值
		//(curr, agg) => curr + agg：curr是当前的值，因为是按照key进行聚合操作，所以不用管键，直接使用值，agg是局部的聚合结果
		val res: RDD[(String, Double)] = rdd.aggregateByKey(0.8)((zeroVal, item) => zeroVal * item, (curr, agg) => curr + agg)

		res.collect().foreach(println)

		//5.关闭资源
		sc.stop()
	}
}
~~~

**案例**

~~~ java
object Spark_RDD_aggregateKey_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("b", 3), ("b", 4),("b", 5),("a", 6)
		))

		/**
		 * (a [1,2]) (a[3,4])
		 * (a [2]) (a[4])
		 * (a[6])
		 */

		/**
		 *aggregateByKey存在函数颗粒化，有两个参数列表
		 * 第一个参数列表：需要传递一个参数，初始值
		 * 		主要用于遇到第一个key的时候和value进行分区内计算
		 * 第二个参数列表需要传递两个参数
		 * 		第一个参数表示分区内分区规则
		 * 		第二个参数表示分区间分区规则
		 *
		 */

		val value = rdd.aggregateByKey(0)(
			(x, y) => {
				math.max(x, y)
			},
			(x, y) => {
				x + y
			}
		)

		value.collect().foreach(println)

		context.stop()
	}
}
~~~

**图解**

初始值

![1614818625333](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170416-419794.png)

分区内计算

![1614818672755](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/084435-924387.png)

分区间计算

![1614818639782](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/084417-599334.png)

分区内计算规则和分区间计算规则相同怎么办？ 

~~~ java
object Spark_RDD_aggregateKey__ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("b", 3), ("b", 4),("b", 5),("a", 6)
		))


		val value = rdd.aggregateByKey(0)(
			(x, y) => {
				x+y
			},
			(x, y) => {
				x + y
			}
		)
		//	如果分区内和分区间计算规则相同，可以使用下面函数
		rdd.foldByKey(0)(_+_).collect().foreach(println)

		//value.collect().foreach(println)

		context.stop()
	}
}
~~~

**类型补充**

71

~~~ java
object Spark_RDD_aggregateKey___ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("b", 3), ("b", 4),("b", 5),("a", 6)
		))

		//aggregateByKey最终返回类型要和初始的类型保持一致

		//获取相同key的平均值(a,3)(b,4)
		/**
		 * 第一个0表示和第一个数据value比较的0，第二个0表示出现的次数的初始值
		 */
		val newRdd:RDD[(String,(Int,Int))]=rdd.aggregateByKey((0,0))(
			(t,v)=>{
				//第一个表示值相加，第二个表示次数相加
				(t._1+v,t._2+1)

			},
			(t1,t2)=>{
				(t1._1+t2._1,t1._2+t2._2)
			}
		)

		/**
		 * 求平均值，使用mapValues，这个方法可以保持key不变，只对value做计算
		 */

		val resultRdd:RDD[(String,Int)] = newRdd.mapValues {
			case (num, cnt) => {
				num / cnt
			}
		}

		resultRdd.collect().foreach(println)

		context.stop()
	}
}

~~~

**图解**

![1614821215557](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170432-81493.png)

![1614821226136](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170435-547797.png)

![1614821232927](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/101148-586705.png)

###### foldByKey 

foldByKey和reduceByKey的区别是前者可以指定一个初始值，reduceByKey的初始值是0.

foldByKey和scala中的foldLeft或者foldRight的区别是，前者会作用于每一条数据，但是后者只会作用一次

foldByKey的底层是aggregateByKey算子

**函数签名**

~~~ java
def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]
~~~

**函数说明**

当分区内计算规则和分区间计算规则相同时， aggregateByKey 就可以简化为 foldByKey 

~~~ java
//	如果分区内和分区间计算规则相同，可以使用下面函数
		rdd.foldByKey(0)(_+_).collect().foreach(println)
      
      
      
object test {
	def main(args: Array[String]): Unit = {


		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")
		//	创建上写文
		val sc = new SparkContext(conf)

		val RddData: RDD[(String, Int)] = sc.parallelize(Seq(
			("zhangsan", 1),
			("zhangsan", 1),
			("lisi", 2),
			("lisi", 2),
			("zhangsan", 1)
		))
		
//curr是当前值,agg是聚合后累计的结果值
		val fbk: RDD[(String, Int)] = RddData.foldByKey(10)((curr, agg) => curr + agg)
		fbk.collect().foreach(println)
		sc.stop()

  }
}
//结果,可以看到，初始值作用于每一个数据，而不是在整体上作用一次
(lisi,24)
(zhangsan,33)
~~~

###### combineByKey 

![1621832371339](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/125934-562744.png)

**函数签名**

~~~ java
def combineByKey[C](
createCombiner: V => C,
mergeValue: (C, V) => C,
mergeCombiners: (C, C) => C): RDD[(K, C)]

createCombiner：对value进行初步的转换
mergeValue：在每一个分区上面吧上一步转换的结果进行聚合操作
mergeCombiners：把所有分区上每一个分区的聚合结果进行聚合
partitioner:可选，分区函数
mapSideCombiner:可选，是否在map端进行combiner
serializer:序列化器

~~~

**函数说明**

最通用的对 key-value 型 rdd 进行聚集操作的聚集函数（aggregation function）。类似于aggregate()， combineByKey()允许用户返回值的类型与输入不一致。 

将数据 List(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98))求每个 key 的平
均值 

~~~ java
object Spark_RDD_aggregateKey___ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("b", 3), ("b", 4),("b", 5),("a", 6)
		))

		//aggregateByKey最终返回类型要和初始的类型保持一致

		//获取相同key的平均值(a,3)(b,4)
		/**
		 * 第一个0表示和第一个数据value比较的0，第二个0表示出现的次数的初始值
		 * 元祖里面的值，一个代表相加的结果，一个代表数据的个数
		 */
		val newRdd:RDD[(String,(Int,Int))]=rdd.aggregateByKey((0,0))(
			(t,v)=>{
				//第一个表示值相加，第二个表示次数相加
				(t._1+v,t._2+1)

			},
      //分区之间相加
			(t1,t2)=>{
				(t1._1+t2._1,t1._2+t2._2)
			}
		)

		/**
		 * 求平均值，使用mapValues，这个方法可以保持key不变，只对value做计算
		 */

		val resultRdd:RDD[(String,Int)] = newRdd.mapValues {
			case (num, cnt) => {
				num / cnt
			}
		}

		resultRdd.collect().foreach(println)

		context.stop()
	}
}

~~~

**求平均分**

![1616834063840](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/125944-847098.png)

**代码案例**

~~~ java
object Test06 {

	def main(args: Array[String]): Unit = {


		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")
		//	创建上写文
		val sc = new SparkContext(conf)

		val RddData: RDD[(String, Int)] = sc.parallelize(Seq(
			("zhangsan", 99),
			("zhangsan", 96),
			("lisi", 97),
			("lisi", 98),
			("zhangsan", 89.0)
		))
		/*
		combineByKey算子接受三个参数
				createCombiner函数是转换函数，但是仅仅作用于第一条数据，做一个初始化操作，用于开启计算
				mergeValue作用在分区上面，对分区做聚合操作
				mergeCombiners对最终所有的分区做聚合操作
		 */

		//接受三个参数，三个参数是三个函数
		val value1: Any = RddData.combineByKey(
			createCombiner = (curr: Double) => (curr, 1), //double类型是输出的分区，对于keuy,combineByKey已经帮助我们分好了
			//集合中的数据，只有第一条记录经过createCombiner方法，以后每一条数据都通过mergeValue方法做聚合操作
			mergeValue = (curr: (Double, Int), nextValue: Double) =>
				//curr对应的就是createCombiner处理完毕后生成的元祖类型，nextValue是我们的下一个键值对的值
				(curr._1 + nextValue, curr._2 + 1), //在分区上做初步的聚合操作，curr._1 + nextValue做分数的聚合，curr._2 + 1做人数的聚合
			//下面是对所有分区的结果进行聚合操作，类型都是元祖类型
			mergeCombiners = (curr: (Double, Int), agg: (Double, Int)) =>
				(curr._1 + agg._1, curr._2 + agg._2)
		)

		val value: Any = RddData.combineByKey(
			createCombiner = (curr: Double) => (curr, 1),
			mergeValue =
				//直接和下一条数据的值进行聚合操作
				//		其中curr:(Double,Int)是createCombiner聚合后的结果
				//		nextValue是RddData中的下一条数据
				(curr: (Double, Int), nextValue: Double) =>
					(curr._1 + nextValue, curr._2 + 1)
			,
			mergeCombiners =
				(curr: (Double, Int), agg: (Double, Int)) =>
					(curr._1 + agg._1, curr._2 + agg._2)

		)
		//求最后的结果
		//上面的结果形式是("zhangsan",(888,3))
		

		/*
		createCombiner:转换数据
		mergeValue:分区上的聚合
		mergeCombiners:把所有分区上的结果进行聚合生成最终结果
		 */

		sc.stop()
	}
}
~~~

reduceByKey、 foldByKey、 aggregateByKey、 combineByKey 的区别？ 

- reduceByKey: 相同 key 的第一个数据不进行任何计算，分区内和分区间计算规则相同 
- FoldByKey: 相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则相同 
- AggregateByKey：相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则可以不相同 
- CombineByKey:当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区
  内和分区间计算规则不相同。 

###### sortByKey 

只能作用于键值对类型的数据。

**函数签名**

~~~ java
def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
: RDD[(K, V)]
~~~

**函数说明**

在一个(K,V)的 RDD 上调用， K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序的 

~~~ java
object Spark_RDD_sortedByKey {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd1 = context.makeRDD(List(
			("b", 1), ("c", 2), ("a", 3),("n", 55),("f", 5),("h", 44),("g", 9)
		),2)
      //默认是按照key进行排序操作
		val value = rdd1.sortByKey(true)


		value.collect().foreach(println)


		context.stop()
	}
}
~~~

###### 小结

~~~~ java
object Spark_RDD_xiaojie {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("a", 1), ("a", 2), ("b", 3), ("b", 4),("b", 5),("a", 6)
		),2)


		/*
			reduceByKey:
		*		combineByKeyWithClassTag[V]((v: V) => v,第一个值不会参加运算
																					 func,分区内计算规则
																					 func,分区间计算规则，区内和区间计算规则一样
																					 partitioner)
		*
		aggregateByKey:
				combineByKeyWithClassTag[U]((v: V) =>
																					createZero初始值，v相同key的第一个值
																					cleanedSeqOp(createZero(), v),初始值和第一条数据的value分区内进行的操作
      																		cleanedSeqOp,分区内计算规则
      																		combOp,分区间计算规则，发现分区内和分区间规则不同
      																		partitioner)
		*
		foldByKey:
				combineByKeyWithClassTag[V]((v: V) =>
																					createZero初始值，v相同key的第一个值
																						cleanedFunc(createZero(), v),初始值和第一条数据的value分区内进行的操作
      																		cleanedFunc,分区内计算规则
      																		cleanedFunc,分区间计算规则 但是发现分区间和分区内使用的是同一个方法
      																		partitioner)
     combineByKey:
																					combineByKeyWithClassTag(
																					createCombiner,相同key第一条数据进行的处理
																					mergeValue,分区内的数据处理函数
																					mergeCombiners,分区间的数据处理函数
																					defaultPartitioner(self))
		*/
		rdd.reduceByKey(_+_)
		rdd.aggregateByKey(0)(_+_,_+_)
		rdd.foldByKey(0)(_+_)
		rdd.combineByKey(v=>v,(x:Int,y)=>x+y,(x:Int,y:Int)=>x+y)

		//resRdd.collect().foreach(println)

		context.stop()
	}
}

~~~~

###### join 

![1616840514998](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/182419-866963.png)

![1621918969027](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/130250-549417.png)

![1621944495706](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/200818-356644.png)

**Venn图**

![1621944540616](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/200903-310256.png)

RDD中关联JOIN函数都在PairRDDFunctions中，具体截图如下：

![1621944591207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170525-171188.png)

**函数签名**

~~~ java
def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]
~~~

**函数说明**

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的(K,(V,W))的 RDD 

~~~ java
object Spark_RDD_join {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd1 = context.makeRDD(List(
			("a", 1), ("b", 2), ("c", 3)
		))

		val rdd2 = context.makeRDD(List(
			("a", 4), ("a", 5), ("c", 6),("a",6)
		))
		val rdd3 = context.makeRDD(List(
			(1, 4), (2, 5), (3, 6)
		))


		/**
		 * 参加链接的数据源的key类型必须一样，value类型可以不同
		 * 两个不同数据源的数据，相同key的value会链接在一起，形成元祖
		 * 如果两个数据源中的key没有匹配上，那么数据就不会出现在结果中
		 * 如果两个数据源中key有多个相同，那么就会一次匹配，可能出现笛卡尔积的情况，出具量会出现几何式增长
		 */
		val value = rdd1.join(rdd2)

		//key类型不同会报错
		//rdd1.join(rdd3)


		value.collect().foreach(println)


		context.stop()
	}
}
~~~

###### leftOuterJoin 

**函数签名**

~~~ java
def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))] 
~~~

**函数说明**

类似于 SQL 语句的左外连接 

~~~ java
object Spark_RDD_leftjoin {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd1 = context.makeRDD(List(
			("a", 1), ("b", 2), ("c", 3),("d",6)
		))

		val rdd2 = context.makeRDD(List(
			("a", 4), ("a", 5), ("c", 6)
		))

		val value = rdd1.leftOuterJoin(rdd2)
		val value1 = rdd1.rightOuterJoin(rdd2)

		value.collect().foreach(println)


		context.stop()
	}
}
~~~

###### cogroup 

**函数签名**

~~~ java
def cogroup[W](other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]
~~~

**函数说明**

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD 

~~~ java
object Spark_RDD_cogroup {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd1 = context.makeRDD(List(
			("a", 1), ("b", 2), ("c", 3)
		))

		val rdd2 = context.makeRDD(List(
			("a", 4), ("a", 5), ("c", 6),("a",6)
		))
		val rdd3 = context.makeRDD(List(
			(1, 4), (2, 5), (3, 6)
		))

		/**
		 * cogroup:connect+group
		 */

		val value:RDD[(String,(Iterable[Int],Iterable[Int]))] = rdd1.cogroup(rdd2)


		value.collect().foreach(println)
		context.stop()
	}
}
~~~

###### 获取键值集合

如果RDD中的数据类型是键值对的类型，还可以单独获取键或者值的集合

~~~ java
object Spark_RDD_substract {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[(Int,String)] = context.makeRDD(List((1,"xiaohong"),(2,"xiaohua"),(3,"xiaobai"),(4,"xiaomign")))


		makRdd.keys.collect().foreach(println)

		makRdd.values.collect().foreach(println)
		//makRdd.collect().foreach(println)
		context.stop()
	}
}
~~~

###### 面试题

groupByKey和reduceByKey

**groupByKey+sum：分组+聚合**

简单来说就是对键值对类型的数据进行分组操作。

![1621944201615](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/200324-249805.png)

**reduceByKey函数：分组+聚合，但有预聚合**

先分组，在按照组进行聚合操作，预聚合就是先在每一个分区中做一次聚合操作。

![1621944380034](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170539-17376.png)

##### 项目练手

###### 数据准备

agent.log：时间戳，省份，城市，用户，广告，中间字段使用空格分隔。 

###### 需求描述

统计出每一个省份每个广告被点击数量排行的 Top3 

**思路分析**

![1614833391174](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/124959-656754.png)

![1614833397360](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/125001-142922.png)

###### 代码实现

~~~ java
object ExerTest {

	def main(args: Array[String]): Unit = {


		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		val sc = new SparkContext(conf)

		/**
		 * 获取原始数据：时间戳，省份，城市，用户，广告，
		 */

		val  dataRdd = sc.textFile("datas/agent.txt")

		/**
		 * 		将原始数据进行结构的转换
		 * 时间戳，省份，城市，用户，广告，=>((省份，广告)1)
		 * 转换结构一般使用map方法：时间戳，省份，城市，用户，广告
		 * 1516609240717 1 4 4 5
 		 */
		val mapRdd:RDD[((String,String),Int)] = dataRdd.map(line => {
			//先使用空格进行切分
			val fields = line.split(" ")
			//对返回的结果进行包装
			((fields(1), fields(4)), 1)
		})


		/**
		 * 将转换后的数据进行分组聚合操作,相同的key做value的聚合操作
		 * ((省份，广告),1)=>((省份，广告),sum)
		 */
		val reduceRdd:RDD[((String,String),Int)] = mapRdd.reduceByKey(_ + _)


		/**
		 * 将聚合的结果进行结构转换
		 * ((省份，广告)sum)=>(省份(广告,sum))
		 * 再次做数据的结构转换
		 */
		val mapRddT:RDD[(String,(String,Int))]=reduceRdd.map {
			case ((pre, adv), sum) => {
				(pre, (adv, sum))
			}
		}


		/**
		 * 将转换后的数据根据省份进行分组操作
		 * (省份,[(广告A,sumA),(广告b,sumb)])
		 */

		val groupRdd:RDD[(String,Iterable[(String,Int)])] = mapRddT.groupByKey()

		/**
		 * 将分组后的数据进行组内排序，降序排列，
		 * mapValues:保持key不变，只对value做改变时使用很方便
		 */

		val resRdd=groupRdd.mapValues(
			iter=>{
				//按照第二个值排序，也就是int指数值Iterable[(String,Int)]
				iter.toList.sortBy(_._2)(Ordering.Int.reverse).take(3)
			}
		)

		resRdd.collect().foreach(println)

		/**
		 * (4,List((12,25), (2,22), (16,22)))
		 * (8,List((2,27), (20,23), (11,22)))
		 * (6,List((16,23), (24,21), (22,20)))
		 */

	}
}
~~~

#### 行动算子

上面的转换算子的执行难都是惰性的，在执行的时候，并不会去调度执行，而是只生成对应的RDD，只有在触发行动算子之后，才会真正的求得最终的结果。

##### reduce

![1621920681549](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/133222-173238.png)

![1621921234313](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170559-128346.png)

**shuffle过程**

![1621921261449](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170603-338605.png)

**reduce过程**

![1621921289297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/134133-464192.png)

**函数签名**

~~~ java
def reduce(f: (T, T) => T): T
~~~

**函数说明**

聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据 ，最终输出的是一个结果，这就是所谓的聚合操作。指定一个函数将RDD中的任何类型的值规约为一个值。

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 聚合数据，求和操作，第一个_是当前的值,第二个_是累加的局部值
//对于单个的值，curr代表的是值，对于键值对类型,curr代表的是当前的键值对，要和reducebYkEY分开
  //结果是什么类型，那么agg就是什么类型,agg是什么类型，最终生成的结果就是什么类型
val reduceResult: Int = rdd.reduce(_+_)  （curr,agg）
~~~

**案例二**

~~~ java
object Test08 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		val data: RDD[Int] = sc.parallelize(1 to 10, 2)

		val sum: Int = data.reduce((curr, agg) => {
			println("curr=" + curr + "  agg=" + agg)
			curr + agg
		})
		println(sum)
	}
}
~~~

##### collect

以数组的形式返回结果集中的所有元素

**函数签名**

~~~ java
def collect(): Array[T]
~~~

**函数说明**

在驱动程序中，以数组 Array 的形式返回数据集的所有元素 ，汇总最终的结果

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 收集数据到 Driver
rdd.collect().foreach(println)
~~~

##### count

**函数签名**

~~~ java
def count(): Long
~~~

**函数说明**

返回 RDD 中元素的个数 

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val countResult: Long = rdd.count()
  
object Test15 {
	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val sc = new SparkContext(conf)
		val data: RDD[Int] = sc.parallelize(Seq(2, 54, 32, 76, 23, 1, 45, 78))

		val l: Long = data.count()

		println(l)
		sc.stop()
	}
}
~~~

##### first

**函数签名**

~~~ java
def first(): T
~~~

函数说明

返回 RDD 中的第一个元素 

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val firstResult: Int = rdd.first()
println(firstResult)
~~~

##### take

**函数签名**

~~~ java
def take(num: Int): Array[T]
~~~

**函数说明**

返回一个由 RDD 的前 n 个元素组成的数组 

~~~ java
vval rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val takeResult: Array[Int] = rdd.take(2)
println(takeResult.mkString(","))
~~~

##### takeOrdered 

**函数签名**

~~~ java
def takeOrdered(num: Int)(implicit ord: Ordering[T]): Array[T]
~~~

**函数说明**

返回该 RDD 排序后的前 n 个元素组成的数组 

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,3,2,4))
// 返回 RDD 中元素的个数
val result: Array[Int] = rdd.takeOrdered(2)
~~~

##### 案例

~~~ java
object Spark_RDD_action {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4,5,6,7,8,9,9
		))


		/**
		 * 所谓的行动算子，其实就是触发作业(job)执行方法
		 * 底层代码调用的是环境对象的runjob()方法
		 * 底层代码会创建activeJob,并提交执行
		 */
		//	reduce()行动算子返回的直接是一个int类型的值，而不是rdd数据结构
		val i = rdd.reduce(_ + _)
		//println(i)

		/**
		 * collect()：采集，将不同分区的数据按照顺序进行采集到driver端的内存中
		 * 返回的结果是一个数组类型
		 */

		//rdd.collect().foreach(println)

		/**
		 * count():统计数据源中数据的个数
		 */
		val l = rdd.count()
		println(i)

		/*
		first():获取数据源中的第一个数据
		 */
		val fir = rdd.first()
		println(fir)

		/*
		take():获取多少个元素
		 */
		val array:Array[Int] = rdd.take(5)
		array.foreach(println)

		/*
		takeOrdered:数据首先进行排序，然后在取前几个
		 */
		val arr:Array[Int] = rdd.takeOrdered(3)
		arr.foreach(println)

		context.stop()
	}
}

~~~

##### aggregate 

**函数签名**

~~~ java
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U
~~~

**函数说明**

分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合 

~~~ java
object Spark_RDD_agg {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4
		),2)

		/**
		 *aggregateByKey:仅仅会参与分区内的计算
		 * aggregate：不仅参与分区内的计算，还会参与分区间的计算
		 */

		//	输出40:13+17+10=40，而不是30，因为10也参与运算
		val res = rdd.aggregate(10)(_ + _, _ + _)

		println(res)

		context.stop()
	}
}
~~~

**案例二**

~~~ java
object Test11 {

	def main(args: Array[String]): Unit = {
		//1.准备环境(Env)sc-->SparkContext
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		//2.加载数据
		val nums: RDD[Int] = sc.parallelize( 1 to 10) //和55

		val result1: Double = nums.sum() //55//底层:fold(0.0)(_ + _)
		val result2: Int = nums.reduce(_+_)//55 注意:reduce是action ,reduceByKey是transformation
		val result3: Int = nums.fold(0)(_+_)//55
		//(zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U)
		//(初始值)(各个分区的聚合函数,多个分区结果的聚合函数)
		val result4: Int = nums.aggregate(0)(_+_,_+_)

		println(result1)
		println(result2)
		println(result3)
		println(result4)

		//5.关闭资源
		sc.stop()
	}
}
~~~

**案例三**

~~~ java
object Test10 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")


		val data: RDD[Int] = sc.parallelize(1 to 10, 2)

		//查看每一个分区的数据
		data.foreachPartition(iter =>{
			//println(TaskContext.getPartitionId()+" "+iter.foreach(println))
			println(s"p-${TaskContext.getPartitionId()}: ${iter.mkString(", ")}")
		})

		val top2: mutable.Seq[Int] = data.aggregate(new ListBuffer[Int]())(
			// 分区内聚合函数，每个分区内数据如何聚合  seqOp: (U, T) => U,
			(u, t) => {
				println(s"p-${TaskContext.getPartitionId()}: u = $u, t = $t")
				// 将元素加入到列表中
				u += t //
				// 降序排序
				val top = u.sorted.takeRight(2)
				// 返回
				top
			},
			// 分区间聚合函数，每个分区聚合的结果如何聚合 combOp: (U, U) => U
			(u1, u2) => {
				println(s"p-${TaskContext.getPartitionId()}: u1 = $u1, u2 = $u2")
				u1 ++= u2 // 将列表的数据合并，到u1中
				//
				u1.sorted.takeRight(2)
			}
		)
		println(top2)
	}
}
~~~

##### fold 

**函数签名**

~~~ java
def fold(zeroValue: T)(op: (T, T) => T): T
~~~

**函数说明**

折叠操作， aggregate 的简化版操作 

~~~ java
object Spark_RDD_agg {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4
		),2)

		/**
		 *aggregateByKey:仅仅会参与分区内的计算
		 * aggregate：不仅参与分区内的计算，还会参与分区间的计算
		 */

		//	输出40:13+17+10=40，而不是30，因为10也参与运算
		//val res = rdd.aggregate(10)(_ + _, _ + _)
		/*
		分区内和分区之间的运算规则一样，使用同一个函数
		 */
		val res = rdd.fold(10)(_ + _)


		println(res)

		context.stop()
	}
}
~~~

**案例二**

~~~ java
object Test09 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")


		val data: RDD[Int] = sc.parallelize(1 to 10, 2)
		//设定初始值是10，只是参与一次运算
		val sum: Int = data.fold(10)((curr, agg) => {
			println("curr=" + curr + "  agg=" + agg)
			curr + agg
		})
		println(sum)
	}
}
~~~

##### countByKey 

先按照key进行分组，然后求每一个组中元素的个数。

**函数签名**

~~~ java
def countByKey(): Map[K, Long]
~~~

**函数说明**

统计每种 key 的个数 ,但是数据类型必须是键值对形式，countByValue不要求数据是键值对类型

~~~ java
object Spark_RDD_countByValue {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4
		),2)

		val intToLong:collection.Map[Int,Long] = rdd.countByValue()
		//Map(4 -> 1, 2 -> 1, 1 -> 1, 3 -> 1),返回的是每一个值出现的次数
		//println(intToLong)

		val rdd1 = context.makeRDD(List(
			("a",1),("a",2),("a",3),("b",2),("b",5)
		),2)

		val stringToLong:collection.Map[String,Long] = rdd1.countByKey()
		//Map(b -> 2, a -> 3):统计的结果代表每一个键值出现的次数，而不是把value进行累加
		println(stringToLong)
		context.stop()
	}
}
~~~

##### wordcount

~~~ java
object Spark_RDD_wordCount {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		context.stop()
	}

	/**
	 * 使用group实现wordcount功能
	 * @param sc 上下文
	 */
	private def wordcount1(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val groupRddL:RDD[(String,Iterable[Int])] = flatmapRdd.groupBy(word => word)
		val wordCount:RDD[(String,Int)] = groupRddL.mapValues(iter => iter.size)
	}


	/**
	 * groupByKey:要求我们的数据必须有key-value类型，但是效率不是很高，因为中间有shuffer过程
	 * @param sc
	 */
	private def wordcount2(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val groupRddL:RDD[(String,Iterable[Int])] = mapRdd.groupByKey()
		val wordCount:RDD[(String,Int)] = groupRddL.mapValues(iter => iter.size)
	}

	/**
	 * 没有shuffer过程，效率比较高
	 * @param sc
	 */
	private def wordcount3(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val wordCount:RDD[(String,Int)]= mapRdd.reduceByKey(_+_)

	}

	/**
	 * aggregateByKey
	 * @param sc
	 */
	private def wordcount4(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val wordCount:RDD[(String,Int)]= mapRdd.aggregateByKey(0)(_+_,_+_)
	}

	/**
	 * foldByKey
	 * @param sc
	 */
	private def wordcount5(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val wordCount:RDD[(String,Int)]= mapRdd.foldByKey(0)(_+_)
	}

	/**
	 *combineByKey
	 * @param sc
	 */
	private def wordcount6(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val wordCount:RDD[(String,Int)]= mapRdd.combineByKey(
			v=>v,
			(x:Int,y)=>{x+y},
			(x:Int,y:Int)=>{x+y}
		)
	}

	/**
	 * countByKey
	 * @param sc
	 */
	private def wordcount7(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val mapRdd:RDD[(String,Int)] = flatmapRdd.map((_, 1))
		val wordCount:collection.Map[String,Long]= mapRdd.countByKey()
	}

	/**
	 * countByValue
	 * @param sc
	 */
	private def wordcount8(sc:SparkContext):Unit={
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd:RDD[String] = word.flatMap(line => line.split(""))
		val stringToLong:collection.Map[String,Long]= flatmapRdd.countByValue()
	}

	/**
	 * countByValue
	 * @param sc
	 */
	private def wordcount9(sc:SparkContext):Unit= {
		val word: RDD[String] = sc.makeRDD(List("hello word", "hello spark", "hello scala"))
		val flatmapRdd: RDD[String] = word.flatMap(line => line.split(""))
		/**
		 * word=>map[(word,1)]
		 */
		val mapWord = flatmapRdd.map(
			word => {
				mutable.Map[String, Long]((word, 1))
			}
		)

		val wordcount = mapWord.reduce(
			(map1, map2) => {
				map2.foreach({
					case (word,count)=>{
					val newCount=map1.getOrElse(word,0l)+count
						map1.update(word,newCount)
					}
				})
				map1
			}
		)
		println(wordcount)
	}
}

~~~

##### save 相关算子 

**函数签名**

~~~ java
def saveAsTextFile(path: String): Unit
def saveAsObjectFile(path: String): Unit
def saveAsSequenceFile(
path: String,
codec: Option[Class[_ <: CompressionCodec]] = None): Unit
~~~

**函数说明**

将数据保存到不同格式的文件中 

~~~ java
// 保存成 Text 文件
rdd.saveAsTextFile("output")
// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")
// 保存成 Sequencefile 文件,数据类型必须是k-v类型
rdd.map((_,1)).saveAsSequenceFile("output2")
~~~

##### foreach 

**函数签名**

~~~ java
def foreach(f: T => Unit): Unit = withScope {
val cleanF = sc.clean(f)
sc.runJob(this, (iter: Iterator[T]) => iter.foreach(cleanF))
}
~~~

**函数说明**

分布式遍历 RDD 中的每一个元素，调用指定函数 

~~~ java

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4,5,6,7,8,9,
		))

		/**
		 * foreach:driver端内存集合的循环遍历方法
		 */
		rdd.collect().foreach(println)//在driver端完成
		println("******************")

		/**
		 * 是executor端内存的数据打印
		 */

		/**
		 * 什么叫做算子=》算子就是操作(operator)
		 * 		RDD的方法和scala结合方法不一样
		 * 		集合对象的方法都是在同一个节点的聂村中完成的，依托同一块内存
		 * 	但是RDD方法可以将我们的计算逻辑发布到分布式的节点执行
		 * 	为了区分RDD的方法和SCALA集合的方法，我们把RDD集合的方法叫做算子
		 RDD方法的外部操作都是在driver端进行的，而内部的逻辑代码是在executor端执行的
		 */
		rdd.foreach(println)//在节点的内存中完成
		context.stop()
	}
}
~~~

**图示**

![1614851041159](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170705-466363.png)

collect会先把各个节点上的数据全部收集回来，然后在driver端的内存中进行打印操作

![1614851105867](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170715-422562.png)

如果不收集直接打印，那么各个节点的打印顺序不能保证，所以输出的结果也无法预测。

**案例**

~~~ java
object Spark_RDD_foreach_ {

	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			1, 2, 3, 4,5,6
		))

		val user:User=new User()

		rdd.foreach(
			num =>{
				println("user age:"+(user.age+num))
			}
		)

		context.stop()
	}

	/**
	 * 需要实现序列化接口，因为在driver端和executor传递需要进行网络传输，所以要序列化
	 * 样例类在编译的时候回自动实现序列化接口
	 */
	//class User extends Serializable {
	case class User(){
		var age:Int=30
	}
}
~~~

每次调用行动算子，都会触发一个job。

#### 阶段性项目

1. 读取文件
2. 抽取需要的列
3. 按照年和月进行聚合操作
4. 排序，获取最终结果

**代码实现**

~~~ java
object Test16 {
	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[6]").setAppName("exertise")

		//	创建上写文
		val sc = new SparkContext(conf)
		//读取数据集,读取的时候，是把每一行数据作为字符串读取进来,列以,分隔
		val sourceRDD: RDD[String] = sc.textFile("")

		//抽取固定列的数据
		val mapRDD: RDD[((String, String), String)] = sourceRDD.map(item => {
			val words: Array[String] = item.split(",")
			//	抽取出:年，月，Pm,但是这种形式不是键值对类型，所以我们把年，月当做组合的键值
			((words(1), words(2)), words(6)) //((key,value),value)
		})
		//数据的清洗,过滤空的字符串，过滤na值
		val filterRDD: RDD[((String, String), String)] = mapRDD.filter((item => {
			StringUtils.isNotEmpty(item._2) && !item._2.equalsIgnoreCase("NA")
		}))
		//聚合操作,按照(年，月)key进行聚合操作
		//但是首先要把第二项转换为int类型的数据值
		val rdd: RDD[((String, String), Int)] = filterRDD.map(item => {
			(item._1, item._2.toInt)
		})

		//对数据做聚合操作
		//curr是当前对象的值，agg是局部聚合的结果
		val reduceRes: RDD[((String, String), Int)] = rdd.reduceByKey((curr, agg) => {
			curr + agg
		})

		//对聚合后的结果进行排序操作
		//按照元祖的第二项进行排序
		val res: RDD[((String, String), Int)] = reduceRes.sortBy(item => {
			item._2
		})

		//获取前10的数据
		val finalRes: Array[((String, String), Int)] = res.take(10)

		//打印输出最后的结果
		finalRes.foreach(println)

		//关闭sc
		sc.stop()
	}
}
~~~

通过上面程序的练习，要掌握RDD程序的编写步骤：

1. 创建SparkContext()运行环境
2. 创建原始的RDD数据级
3. 处理RDD，进行数据处理
4. 触发行动算子，获取结果

![1622103180737](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170733-303983.png)

![1622103206865](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/27/161336-470579.png)

#### RDD 序列化 

**闭包检查**

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行。 那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列化，这个操作我们称之为闭包检测。 Scala2.12 版本后闭包编译方式发生了改变 

**序列化方法和属性**

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行 。

~~~ java
object Spark_RDD_serial {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		//3.创建一个 RDD
		val rdd: RDD[String] = context.makeRDD(Array("hello world", "hello spark",
			"hive", "rzf"))

		val search=new Search("h")
		//search.getMatch1(rdd).collect().foreach(println)
		search.getMatch2(rdd).collect().foreach(println)

		context.stop()
	}

	/**
	 * 类的构造函数其实就是类的属性，构造参数需要进行闭包检测
	 * @param query
	 * 在这里也可以实现序列化的接口
	 */
	case class Search(query:String){

		def isMatch(s: String): Boolean = {
			s.contains(query)
		}

		// 函数序列化案例
		def getMatch1 (rdd: RDD[String]): RDD[String] = {
			//rdd.filter(this.isMatch)
			rdd.filter(isMatch)
		}

		// 属性序列化案例
		def getMatch2(rdd: RDD[String]): RDD[String] = {
			//rdd.filter(x => x.contains(this.query))
			rdd.filter(x => x.contains(query))
			//val q = query，这样做因为q是一个字符串类型。而字符串类型可以序列化
			//rdd.filter(x => x.contains(q))
		}
	}
}
~~~

**Kryo 序列化框架**

Java 的序列化能够序列化任何的类。但是比较重（字节多） ，序列化后，对象的提交也比较大。 Spark 出于性能的考虑， Spark2.0 开始支持另外一种 Kryo 序列化机制。 Kryo 速度是 Serializable 的 10 倍。当 RDD 在 Shuffle 数据的时候，简单数据类型、数组和字符串类型已经在 Spark 内部使用 Kryo 来序列化。 

注意：即使使用 Kryo 序列化，也要继承 Serializable 接口。 

~~~ java
object Spark_RDD_kryo {


	def main(args: Array[String]): Unit = {
		val conf: SparkConf = new SparkConf()
			.setAppName("SerDemo")
			.setMaster("local[*]")
			// 替换默认的序列化机制
			.set("spark.serializer",
				"org.apache.spark.serializer.KryoSerializer")
			// 注册需要使用 kryo 序列化的自定义类
			.registerKryoClasses(Array(classOf[Searcher]))

		//创建上下文执行环境
		val sc = new SparkContext(conf)
		//创建RDD数据结构
		val rdd: RDD[String] = sc.makeRDD(Array("hello world", "hello rzf",
			"rzf", "hahah"), 2)

		val searcher = new Searcher("hello")
		val result: RDD[String] = searcher.getMatchedRDD1(rdd)
		result.collect.foreach(println)
	}
}

 //case
//需要继承Serializable接口
 class Searcher(val query: String) extends Serializable {
	def isMatch(s: String) = {
		s.contains(query)
	}

	def getMatchedRDD1(rdd: RDD[String]) = {
		rdd.filter(isMatch)
	}

	def getMatchedRDD2(rdd: RDD[String]) = {
		val q = query
		rdd.filter(_.contains(q))
	}
}
~~~

> 使用kryo序列化需要在创建SparkContext的时候设置序列化类型，然后自定义类型也需要继承Serializable类。

#### RDD 依赖关系 

##### RDD血缘关系

RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage（血统）记录下来，以便恢复丢失的分区。 RDD 的 Lineage 会记录 RDD 的**元数据信息和转换行为**，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。 

![1614933707058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/164149-246145.png)

![1614934144895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/164908-814529.png)

![1614934303590](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170802-120012.png)

黄色部分就表示保存的血缘关系，依赖关系包括转换行为和元数据信息。

**血缘关系**

~~~ java
object Spark_RDD_depdence {

	def main(args: Array[String]): Unit = {

		// 创建 Spark 运行配置对象
		val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

		// 创建 Spark 上下文环境对象（连接对象）
		val sc: SparkContext = new SparkContext(sparkConf)

		// 读取文件数据
		val fileRDD: RDD[String] = sc.textFile("datas/word.txt")

		//打印血缘关系
		println(fileRDD.toDebugString)
		println("*****************************")

		// 将文件中的数据进行分词
		val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))

		println(wordRDD.toDebugString)
		println("*****************************")


		// 转换数据结构 word => (word, 1)
		val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_, 1))

		println(word2OneRDD.toDebugString)
		println("*****************************")


		// 将转换结构后的数据按照相同的单词进行分组聚合
		val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_ + _)

		println(word2CountRDD.toDebugString)
		println("*****************************")


		// 将数据聚合结果采集到内存中
		val word2Count: Array[(String, Int)] = word2CountRDD.collect()

		// 打印结果
		word2Count.foreach(println)

		//关闭 Spark 连接
		sc.stop()

	}
}

//依赖关系
(2) datas/word.txt MapPartitionsRDD[1] at textFile at Spark_RDD_depdence.scala:17 []
 |  datas/word.txt HadoopRDD[0] at textFile at Spark_RDD_depdence.scala:17 []
*****************************
(2) MapPartitionsRDD[2] at flatMap at Spark_RDD_depdence.scala:24 []
 |  datas/word.txt MapPartitionsRDD[1] at textFile at Spark_RDD_depdence.scala:17 []
 |  datas/word.txt HadoopRDD[0] at textFile at Spark_RDD_depdence.scala:17 []
*****************************
(2) MapPartitionsRDD[3] at map at Spark_RDD_depdence.scala:31 []
 |  MapPartitionsRDD[2] at flatMap at Spark_RDD_depdence.scala:24 []
 |  datas/word.txt MapPartitionsRDD[1] at textFile at Spark_RDD_depdence.scala:17 []
 |  datas/word.txt HadoopRDD[0] at textFile at Spark_RDD_depdence.scala:17 []
*****************************
(2) ShuffledRDD[4] at reduceByKey at Spark_RDD_depdence.scala:38 []
 +-(2) MapPartitionsRDD[3] at map at Spark_RDD_depdence.scala:31 []
    |  MapPartitionsRDD[2] at flatMap at Spark_RDD_depdence.scala:24 []
    |  datas/word.txt MapPartitionsRDD[1] at textFile at Spark_RDD_depdence.scala:17 []
    |  datas/word.txt HadoopRDD[0] at textFile at Spark_RDD_depdence.scala:17 []
*****************************
~~~

**相邻两个RDD之间的依赖关系**

~~~ java
object Spark_RDD_depdence_ {

	def main(args: Array[String]): Unit = {

		// 创建 Spark 运行配置对象
		val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

		// 创建 Spark 上下文环境对象（连接对象）
		val sc: SparkContext = new SparkContext(sparkConf)

		// 读取文件数据
		val fileRDD: RDD[String] = sc.textFile("datas/word.txt")

		//打印相邻两个RDD之间的依赖关系
		println(fileRDD.dependencies)
		println("*****************************")

		// 将文件中的数据进行分词
		val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))

		println(wordRDD.dependencies)
		println("*****************************")


		// 转换数据结构 word => (word, 1)
		val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_, 1))

		println(word2OneRDD.dependencies)
		println("*****************************")


		// 将转换结构后的数据按照相同的单词进行分组聚合
		val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_ + _)

		println(word2CountRDD.dependencies)
		println("*****************************")


		// 将数据聚合结果采集到内存中
		val word2Count: Array[(String, Int)] = word2CountRDD.collect()

		// 打印结果
		word2Count.foreach(println)

		//关闭 Spark 连接
		sc.stop()

	}
}

//依赖关系
List(org.apache.spark.OneToOneDependency@4da6d664)一对一的依赖
*****************************
List(org.apache.spark.OneToOneDependency@7fe82967)
*****************************
List(org.apache.spark.OneToOneDependency@5af8bb51)
*****************************
21/03/05 17:04:18 INFO FileInputFormat: Total input paths to process : 1
List(org.apache.spark.ShuffleDependency@1e236278)shuffer依赖关系
*****************************
~~~

##### RDD 窄依赖 

窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游） RDD 的一个 Partition 使用，窄依赖我们形象的比喻为独生子女。 

**一对一的依赖**也叫做窄依赖

![1614935447609](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/171324-789475.png)

这种情况下，新的RDD中分区中的数据来自对应的老的RDD中分区中的数据，不会把老RDD中的数据打乱重新分布，存在一对一的关系。

**源码角度**

~~~ java
//窄依赖
@DeveloperApi
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd) {
  override def getParents(partitionId: Int): List[Int] = List(partitionId)
}
~~~

##### RDD 宽依赖 

宽依赖表示同一个父（上游） RDD 的 一个Partition 被多个子（下游） RDD 的 Partition 依赖，会引起 Shuffle，总结：宽依赖我们形象的比喻为多生。 

**shuffer 依赖**也叫做宽依赖

![1614935603198](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/171401-695375.png)

这种依赖的话会把老的RDD中的数据全部打乱重新分配给新的RDD，所以叫做shuffle依赖。

**源码角度**

~~~ java

//宽依赖
class ShuffleDependency[K: ClassTag, V: ClassTag, C: ClassTag](
    @transient private val _rdd: RDD[_ <: Product2[K, V]],
    val partitioner: Partitioner,
    val serializer: Serializer = SparkEnv.get.serializer,
    val keyOrdering: Option[Ordering[K]] = None,
    val aggregator: Option[Aggregator[K, V, C]] = None,
    val mapSideCombine: Boolean = false,
    val shuffleWriterProcessor: ShuffleWriteProcessor = new ShuffleWriteProcessor)
  extends Dependency[Product2[K, V]] {
~~~

**类继承结构**

![1614935943959](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/172134-528166.png)


> 其实区分宽窄依赖主要就是看父RDD的一个Partition的流向，要是流向一个的话就是窄依赖，流向多个的话就是宽依赖,如下图所示，很容易理解：

![20211106144726](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106144726.png)

如果使用宽依赖，那么任务的数量就会增加，因为数据全部被打散，如下图所示，新的RDD中的数据可能来自多个父类RDD中的数据。下图中一共有两个任务。

![1614936854674](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173416-455965.png)

**宽依赖使任务量增加，所以就要划分阶段执行任务，不同分区的数据到达的时间不一样，所以就会存在等待的关系，所以需要划分阶段执行，保证不同阶段里面的task全部执行完毕，才可以执行下一个阶段。**

> 这里有很重要的一代女，为什么需要在宽依赖处划分阶段，就是因为子RDD中的数据来自多个父RDD中的数据，数据到来的时间不一样，如果不划分阶段，那么所有的任务都在这里等待，耗时浪费资源，划分阶段可以保证尽可能的让少的任务等待，其他的任务继续向下执行。

下图中，可以发现，一共有两个任务，从shuffle位置划分为两个阶段。

![1614937061280](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173836-818567.png)

窄依赖不会使任务的数量增加，所以不需要划分阶段执行任务

![1614936891785](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173453-172770.png) 

当RDD分区丢失时（某个节点故障），spark会对数据进行重算。

**窄依赖**

![20211106151713](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151713.png)

对于窄依赖，由于父RDD的一个分区只对应一个子RDD分区，这样只需要重算和子RDD分区对应的父RDD分区即可，所以这个重算对数据的利用率是100%的。

**宽依赖**

![20211106151752](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151752.png)

对于宽依赖，重算的父RDD分区对应多个字RDD分区，这样实际上父RDD中只有一部分的数据是被用于恢复这个丢失的子RDD分区的，另一部分对应子RDD的其他未丢失分区，这就造成了多余的计算，宽依赖中子RDD分区通常来自于多个父RDD分区，极端情况下，所有的父RDD分区都要重新计算。

如下图所示，b1分区丢失，则需要重新计算a1，a2和a3，这样就产生了冗余计算（a1,a2,a3中对应着b2的数据）。

![20211106151947](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151947.png)

区分这两种依赖很有用，首先，窄依赖允许在一个集群节点上以流水线的方式（pipeline）计算所有父分区。例如，逐个元素地执行map，然后filter操作；而宽依赖则需要首先计算好所有父分区数据，然后在节点间进行shuffle，这和MapReduce类似。第二，窄依赖能够更有效地进行失效节点的恢复，即只需要重新计算丢失RDD分区的父分区，而且不同节点间可以并行计算；而对于一个宽依赖关系的Lineage图，单个节点失效可能导致这个RDD的所有祖先丢失部分分区，因而需要整体重新计算。

在深入分区级别来看待这个问题，重算的效用并不在于算了多少，而是在于有多少是冗余的计算。窄依赖中需要重算的都是必须的，所以重算并不会产生冗余计算。

##### RDD阶段划分

> Task：被送到某个executor上的工作单元
> 
> Job：包含很多任务（Task）的并行计算，可以看做和action对应
> 
> Stage：一个Job会被拆分成很多组任务，每组任务被称为Stage
> 
> 按照资源层面划分：Master->Worker->Executor->ThreadPool
> 
> 按照任务层面划分：Application->job->stage->tasks





Spark中的Stage其实就是一组并行的任务，任务是一个个的task 。

Spark任务会根据RDD之间的依赖关系，形成一个DAG有向无环图，DAG会提交给DAGScheduler，DAGScheduler会把DAG划分相互依赖的多个stage，划分stage的依据就是RDD之间的宽窄依赖。遇到宽依赖就划分stage,每个stage包含一个或多个task任务。然后将这些task以taskSet的形式提交给TaskScheduler运行。 stage是由一组并行的task组成。

> DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如， DAG 记录了 RDD 的转换过程和任务的阶段 

##### RDD 任务划分 

RDD 任务切分中间分为： Application、 Job、 Stage 和 Task 

- Application：初始化一个 SparkContext 即生成一个 Application； 

~~~ java
//setAppName：就是设置应用的名字
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
~~~

- Job：一个 Action 算子就会生成一个 Job； 如果一个应用程序中有多个行动算子，那么就是说有多个job,应用程序就是我们在客户端提交的代码。

~~~ java
 //行动算子底层执行的是runJob
def collect(): Array[T] = withScope {
    val results = sc.runJob(this, (iter: Iterator[T]) => iter.toArray)
    Array.concat(results: _*)
  }
~~~

- Stage： Stage 等于宽依赖(ShuffleDependency)的个数加 1； 也就是shuffle依赖的数量+resultStage，最终阶段。
- Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。也就是说任务的数量等于最后一个阶段中分区的数量。 

> 注意： Application->Job->Stage->Task 每一层都是 1 对 n 的关系。 
>
> - 一个应用程序中如果有多个行动算子，那么就是说有多个job。
> - 一个job中如果有shuffle依赖，那么一个job中就右多个阶段。
> - 而一个阶段中可能会有多个分区（或者说有多少个并行计算的任务），所以就会产生多个TASK,极限情况下有一个分区，产生一个task。

##### Stage的切割规则

切割规则：从后往前，遇到宽依赖就切割stage。
![20211106145333](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106145333.png)

**Stage的计算模式**

pipeline管道计算模式,pipeline只是一种计算思想，模式。

![20211106145438](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106145438.png)

**说明**

Spark的pipeLine的计算模式，相当于执行了一个高阶函数f3(f2(f1(textFile))) !+!+!=3 也就是来一条数据然后计算一条数据，把所有的逻辑走完，然后落地，准确的说一个task处理遗传分区的数据 因为跨过了不同的逻辑的分区。而MapReduce是 1+1=2,2+1=3的模式，也就是计算完落地，然后在计算，然后再落地到磁盘或内存，最后数据是落在计算节点上，按reduce的hash分区落地。所以这也是比Mapreduce快的原因，完全基于内存计算。

管道中的数据何时落地：shuffle write的时候，对RDD进行持久化的时候。

Stage的task并行度是由stage的最后一个RDD的分区数来决定的 。一般来说，一个partiotion对应一个task,但最后reduce的时候可以手动改变reduce的个数，也就是分区数，即改变了并行度。例如reduceByKey(XXX,3),GroupByKey(4)，union由的分区数由前面的相加。

如何提高stage的并行度：reduceBykey(xxx,numpartiotion),join(xxx,numpartiotion)，也就是手动设置我们每一个stage里面并行执行的任务个数。

##### shuffle 是划分 DAG 中 stage 的标识,同时影响 Spark 执行速度的关键步骤

RDD 的 Transformation 函数中,又分为窄依赖(narrow dependency)和宽依赖(wide dependency)的操作.窄依赖跟宽依赖的区别是是否发生 shuffle(洗牌) 操作.宽依赖会发生 shuffle 操作. 窄依赖是子 RDD的各个分片(partition)不依赖于其他分片,能够独立计算得到结果,宽依赖指子 RDD 的各个分片会依赖于父RDD 的多个分片,所以会造成父 RDD 的各个分片在集群中重新分片, 看如下两个示例:

![20211106150248](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106150248.png)

第一个 Map 操作将 RDD 里的各个元素进行映射, RDD 的各个数据元素之间不存在依赖,可以在集群的各个内存中独立计算,也就是并行化

第二个 groupby 之后的 Map 操作,为了计算相同 key 下的元素个数,需要把相同 key 的元素聚集到同一个 partition 下,所以造成了数据在内存中的重新分布,即 shuffle 操作,shuffle 操作是 spark 中最耗时的操作,应尽量避免不必要的 shuffle。

宽依赖主要有两个过程: shuffle write 和 shuffle fetch. 类似 Hadoop 的 Map 和 Reduce 阶段,shuffle write 将 ShuffleMapTask 任务产生的中间结果缓存到内存中, shuffle fetch 获得 ShuffleMapTask 缓存的中间结果进行 ShuffleReduceTask 计算,这个过程容易造成OutOfMemory。因为会从各个节点拉去大量的数据存放到内存，当内存放不下的时候，会进行落地操作。

shuffle 过程内存分配使用 ShuffleMemoryManager 类管理,会针对每个 Task 分配内存,Task 任务完成后通过 Executor 释放空间，这里可以把 Task 理解成不同 key 的数据对应一个 Task. 早期的内存分配机制使用公平分配,即不同 Task 分配的内存是一样的,但是这样容易造成内存需求过多的 Task 的 OutOfMemory, 从而造成多余的磁盘 IO 过程,影响整体的效率.(例:某一个 key 下的数据明显偏多,但因为大家内存都一样,这一个 key 的数据就容易 OutOfMemory)，1.5版以后 Task 共用一个内存池,内存池的大小默认为 JVM 最大运行时内存容量的16%,分配机制如下:假如有 N 个 Task,ShuffleMemoryManager 保证每个 Task 溢出之前至少可以申请到1/2N 内存,且至多申请到1/N,N 为当前活动的 shuffle Task 数,因为N 是一直变化的,所以 manager 会一直追踪 Task 数的变化,重新计算队列中的1/N 和1/2N.但是这样仍然容易造成内存需要多的 Task 任务溢出,所以最近有很多相关的研究是针对 shuffle 过程内存优化的.

![20211106151010](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151010.png)

如下 DAG 流程图中,分别读取数据,经过处理后 join 2个 RDD 得到结果:

![20211106151134](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151134.png)

在这个图中,根据是否发生 shuffle 操作能够将其分成如下的 stage 类型:

![20211106151204](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211106151204.png)

(join 需要针对同一个 key 合并,所以需要 shuffle)

运行到每个 stage 的边界时，数据在父 stage 中按照 Task 写到磁盘上，而在子 stage 中通过网络按照 Task 去读取数据。这些操作会导致很重的网络以及磁盘的I/O，所以 stage 的边界是非常占资源的，在编写 Spark 程序的时候需要尽量避免的 。父 stage 中 partition 个数与子 stage 的 partition 个数可能不同，所以那些产生 stage 边界的 Transformation 常常需要接受一个 numPartition 的参数来决定子 stage 中的数据将被切分为多少个 partition。

>PS:shuffle 操作的时候可以用 combiner 压缩数据,减少 IO 的消耗

**Stage:**

一个Job会被拆分为多组Task，每组任务被称为一个Stage就像Map Stage， Reduce Stage。Stage的划分，简单的说是以shuffle和result这两种类型来划分。在Spark中有两类task，一类是shuffleMapTask，一类是resultTask，第一类task的输出是shuffle所需数据，第二类task的输出是result，stage的划分也以此为依据，shuffle之前的所有变换是一个stage，shuffle之后的操作是另一个stage。

比如 rdd.parallize(1 to 10).foreach(println) 这个操作没有shuffle，直接就输出了，那么只有它的task是resultTask，stage也只有一个；

如果是rdd.map(x => (x, 1)).reduceByKey(_ + _).foreach(println), 这个job因为有reduce，所以有一个shuffle过程，那么reduceByKey之前的是一个stage，执行shuffleMapTask，输出shuffle所需的数据，reduceByKey到最后是一个stage，直接就输出结果了。如果job中有多次shuffle，那么每个shuffle之前都是一个stage.

会根据RDD之间的依赖关系将DAG图划分为不同的阶段，对于窄依赖，由于partition依赖关系的确定性，partition的转换处理就可以在同一个线程里完成，窄依赖就被spark划分到同一个stage中，而对于宽依赖，只能等父RDD shuffle处理完成后，下一个stage才能开始接下来的计算。之所以称之为ShuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中 

**Stage划分思路**

因此spark划分stage的整体思路是：从后往前推，遇到宽依赖就断开，划分为一个stage；遇到窄依赖就将这个RDD加入该stage中。

在spark中，Task的类型分为2种：ShuffleMapTask和ResultTask；简单来说，DAG的最后一个阶段会为每个结果的partition生成一个ResultTask，即每个Stage里面的Task的数量是由该Stage中最后一个RDD的Partition的数量所决定的！

而其余所有阶段都会生成ShuffleMapTask；之所以称之为ShuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中。

### RDD 缓存

#### RDD的三个特性

- RDD的分区和suffle过程

- RDD的缓存

- RDD的Checkpoint

#### 使用缓存的意义

1. **可以减少shuffle操作次数**

![1616988151858](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/145628-224411.png)

因为在最后两步获取结果的时候，调用了两次Action算子，相当于触发了两个作业(一个行动算子触发一个job)，但是这两个作业前半部分的数据处理基本一致，这样就导致了重复计算，浪费资源，并且每一个作业都执行了两次shuffle操作，我们说程序中尽量减少shuffle操作。但是如果可以把前面的相同计算的结果进行缓存，那么这样就可以减少shuffle操作。

2. **容错**

![1616988200887](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/150207-401488.png)

可以对中间的计算结果进行缓存操作，后面如果出现失败时候可以直接从缓存点获取计算数据，重新计算。

- 减少shuffle，减少其他算子执行缓存算子生成的结果。
- 容错

#### 问题引出

~~~ java
object Spark_RDD_persist {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val list=List("heppo word","hello spark","hello scala")
		val dataRdd:RDD[String] = context.makeRDD(list)

		val flatRdd = dataRdd.flatMap(line => {
			line.split(" ")
		})

		val mapRdd = flatRdd.map(word => {
			(word, 1)
		})

		val resRdd:RDD[(String,Int)] = mapRdd.reduceByKey(_ + _)

		resRdd.collect().foreach(println)
      //可以看到，相同的操作重新执行一遍

		println("**************************************")


		val flatRdd1 = dataRdd.flatMap(line => {
			line.split(" ")
		})

		val mapRdd1 = flatRdd1.map(word => {
			(word, 1)
		})

		val resRdd1= mapRdd1.groupByKey()

		resRdd1.collect().foreach(println)


		context.stop()
	}
}
~~~

**改进，使用cache()缓存**

~~~ java
object Spark_RDD_persist_ {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val list=List("heppo word","hello spark","hello scala")
		val dataRdd:RDD[String] = context.makeRDD(list)

		val flatRdd = dataRdd.flatMap(line => {
			line.split(" ")
		})

		val mapRdd = flatRdd.map(word => {
			(word, 1)
		})
//在这里调用了缓存的操作，可以把计算结果进行缓存
//cache()也是一个算子，会返回一个新的RDD
		val resRdd:RDD[(String,Int)] = mapRdd.reduceByKey(_ + _).cache()
      
      //两次行动算子的调用，会执行两次数据流的操作，每一个行动算子都会生成一个job
      //转换算子的作用是生成rdd以及RDD之间的依赖关系
      //action算子的作用是生成jb然后执行 job
      //下面两次调用collect会两次执行前面所有的转换算子，也就是说代码执行了两次，因为生成了两个job

		resRdd.collect().foreach(println)

		println("**************************************")
		//重用上面代码
      //真正的情况是对象的重用，而前面的数据流会重新执行一遍
		val resRdd1= mapRdd.groupByKey()
		resRdd1.collect().foreach(println)


		context.stop()
	}
}
~~~

**cache()源码**

~~~ java
 def cache(): this.type = persist()
//默认的存储级别
def persist(): this.type = persist(StorageLevel.MEMORY_ONLY)
~~~

> 可以发现,cache的底层使用的是persist()方法，默认是是缓存在内存中的。

**图解**

也就是说把map算子处的数据使用缓存进行缓存处理，然后后面的算子可以直接使用数据，不需要在进行前面算子的转换操作，可以有效减少shufffle操作次数，提高效率。多个作业之间可以共用数据。

![1614945775874](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/110430-490727.png)

**持久化改进**

添加持久化操作，数据可以存放到内存当中，也可以存放到磁盘当中，作为数据的备份。

![1614945944979](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/170924-976462.png)

RDD持久化操作不一定是为了重用操作，在数据执行时间较长或者比较重要的场合也用持久化操作。当转换过程中出现故障，可以直接从磁盘中加载持久化的数据。

#### RDD Cache 缓存 

RDD 通过 Cache （也是一个算子操作）或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。 

~~~ java
// cache 操作会增加血缘关系，不改变原有的血缘关系
println(wordToOneRdd.toDebugString)
// 数据缓存。
wordToOneRdd.cache()
// 可以更改存储级别
//mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
~~~

#### 缓存级别

![1616989064097](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/113906-955896.png)

![1616989352669](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/114238-64154.png)

是否以反序列化的形式进行存储，如果是，那么存储的就是一个对象，如果不是，那么存储的就是一个序列化过的值。必需要序列化之后对象才可以存储在磁盘中，如果deserialized是true的话存储的就是一个对象，如果是false的话存储的就是二进制数据。

![1616989874428](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/153603-809804.png)

上图中带2后缀的表示存储的副本数是2

~~~ java
object StorageLevel {
val NONE = new StorageLevel(false, false, false, false)
val DISK_ONLY = new StorageLevel(true, false, false, false)
val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)//数据表示副本的个数 
val MEMORY_ONLY = new StorageLevel(false, true, false, true)
val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
~~~

**如何选择存储的级别**

![1616989958204](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/115334-975778.png)

![1614946766976](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/112750-313978.png)

缓存有可能丢失，因为缓存默认是存储在内存当中，断电数据会丢失，或者存储于内存的数据由于内存不足而被删除， RDD 的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于 RDD 的一系列转换，丢失的数据会被重算，由于 RDD 的各个 Partition 是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部 Partition。 

Spark 会自动对一些 Shuffle 操作的中间数据做持久化操作(比如： reduceByKey)。这样做的目的是为了当一个节点 Shuffle 失败了避免重新计算整个输入，加快计算。但是，在实际使用的时候，如果想重用数据，仍然建议调用 persist 或 cache。 

### RDD检查点

####  **CheckPoint的作用**

hdfs的edits机制

![1622188275718](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/155119-274350.png)

**检查点作用**

将数据checkpoint的情况非常少，一般都是缓存在HDFS上面保存。

![1616990352556](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/154326-179761.png)

将checkpoint数据存储在本地，和缓存的差别特别小。

什么是依赖连，多个RDD之间会形成依赖的关系，如果这个依赖太长的话，那么假如后边某一个RDD计算出现错误，那么就会依赖前面的RDD，需要从最开始的RDD重新计算一遍数据，所以，需要使用checkPoint机制去斩断这个依赖过长的链子。

![1622188716121](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/171001-209134.png)

解决办法就是把RDD4的数据存储在HDFS上面，这样就可以保证数据出错误后，依赖链虽然断了，但是已经checkpoint数据到磁盘上面，可以直接加载直接使用，不用再次重新计算一遍。

#### RDD CheckPoint API 

所谓的检查点其实就是通过将 RDD 中间结果写入磁盘,由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题， 可以从检查点开始重做血缘，减少了开销。对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发。 

![1622189132343](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/160543-335853.png)

使用缓存后会返回一个新的RDD，而rdd.heckpoint()后，不会返回新的RDD，rdd本身就变为一个checkpoint的rdd。不准确的说,checkpoint()是一个action()操作，如果调用checkpoint()算子，会重新计算一下rdd，然后把结果存储在hdfs上面。所以在checkpoint()之前应该进行一次缓存操作，这样每一次checkpoint()的时候，使用的都是缓存的数据，自然也就不会触发多个作业的执行。

**代码演示**

~~~ java
object Spark_RDD_checkpoint {


	def main(args: Array[String]): Unit = {
		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		//在这路设置检查点的保存路径
		context.setCheckpointDir("./")

		val list=List("heppo word","hello spark","hello scala")
		val dataRdd:RDD[String] = context.makeRDD(list)

		val flatRdd = dataRdd.flatMap(line => {
			line.split(" ")
		})

		val mapRdd = flatRdd.map(word => {
			println("&&&&&&&&&&&&&&&&&&&&&&&&&&&&")
			(word, 1)
		})

		/**
		 * checkpoint会持久化到磁盘，所以需要存储路径
		 * persist持久化到磁盘是一个临时文件，当作业执行完毕之后会删除
		 * 但是checkpoint当作业执行完毕之后也不会删除
		 * 一般都存盘在分布式文件系统当中
		 */

			mapRdd.checkpoint()

		val resRdd:RDD[(String,Int)] = mapRdd.reduceByKey(_ + _)
		resRdd.collect().foreach(println)

		println("**************************************")
		//重用上面代码
		val resRdd1= mapRdd.groupByKey()
		resRdd1.collect().foreach(println)


		context.stop()
	}
}
~~~

#### 缓存和检查点区别 

- cache:将数据临时存储在内存中重用，会在血缘关系中添加新的依赖，一旦出现问题，可以从头读取数据。
- persist将数据临时存放在磁盘中进行数据重用，但是涉及到磁盘Io操作，性能低，但是数据安全，如果作业执行完毕，临时保存的数据的数据文件就会消失。
- checkpoint可以将长久的保存在磁盘中进行重用，涉及磁盘的Io操作，效率比较低，但是数据安全，为了保证数据的安全，所以一般情况下，会独立执行作业(里面有触发作业执行的操作)，也就是单独在执行一遍作业，为了提高效率，所以一般和cache联合使用，先进行缓存操作，然后checkpoint()方法里面就不会触发作业的执行。

~~~ java
mapRdd.cache()
mapRdd.checkpoint()
~~~

- checkpoint在执行过程中会切断血缘关系，重新建立新的血缘关系，checkpoint等同于改变我们的数据源

![1622188892411](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/160137-266981.png)

> Cache 缓存只是将数据保存起来内存或者磁盘中，不可靠存储，不切断血缘依赖。 Checkpoint 检查点切断血缘依赖。 
>
> Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。 Checkpoint 的数据通常存储在 HDFS 等容错、高可用的文件系统，可靠性高。 
>
> 建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存中读取数据即可，否则需要再从头计算一次 RDD。 

**cache()与persist()：**

会被重复使用的(但是)不能太大的RDD需要cache。cache 只使用 memory，写磁盘的话那就叫 checkpoint 了。

哪些 RDD 需要 checkpoint？运算时间很长或运算量太大才能得到的 RDD，computing chain 过长或依赖其他 RDD 很多的RDD。 实际上，将 ShuffleMapTask 的输出结果存放到本地磁盘也算是 checkpoint，只不过这个 checkpoint 的主要目的是去 partition 输出数据。

cache 机制是每计算出一个要 cache 的 partition 就直接将其 cache 到内存了。但 checkpoint 没有使用这种第一次计算得到就存储的方法，而是等到 job 结束后另外启动专门的 job 去完成 checkpoint 。 也就是说需要 checkpoint 的 RDD 会被计算两次。因此，在使用 rdd.checkpoint() 的时候，建议加上 rdd.cache()， 这样第二次运行的 job 就不用再去计算该 rdd 了，直接读取 cache 写磁盘。

> cache 与 checkpoint 的区别:
> 
> 关于这个问题，Tathagata Das 有一段回答: There is a significant difference between cache and checkpoint. Cache materializes the RDD and keeps it in memory and/or disk（其实只有 memory）. But the lineage（也就是 computing chain） of RDD (that is, seq of operations that generated the RDD) will be remembered, so that if there are node failures and parts of the cached RDDs are lost, they can be regenerated. However, checkpoint saves the RDD to an HDFS file and actually forgets the lineage completely. This is allows long lineages to be truncated and the data to be saved reliably in HDFS (which is naturally fault tolerant by replication).

**persist()与checkpoint():**

深入一点讨论，rdd.persist(StorageLevel.DISK_ONLY) 与 checkpoint 也有区别。前者虽然可以将 RDD 的 partition 持久化到磁盘，但该 partition 由 blockManager 管理。一旦 driver program 执行结束，也就是 executor 所在进程 CoarseGrainedExecutorBackend stop，blockManager 也会 stop，被 cache 到磁盘上的 RDD 也会被清空（整个 blockManager 使用的 local 文件夹被删除）。而 checkpoint 将 RDD 持久化到 HDFS 或本地文件夹，如果不被手动 remove 掉（ 话说怎么 remove checkpoint 过的 RDD？ ），是一直存在的，也就是说可以被下一个 driver program 使用，而 cached RDD 不能被其他 dirver program 使用。

### RDD 分区器 

Spark 目前支持 **Hash** 分区和 **Range** 分区，和**用户自定义分区**。 Hash 分区为当前的默认分区。分区器直接决定了 RDD 中分区的个数、 RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了 Reduce 的个数。 

- 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None 
- 每个 RDD 的分区 ID 范围： 0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。 

##### Hash 分区：对于给定的 key，计算其 hashCode,并除以分区个数取余 

~~~ java
class HashPartitioner(partitions: Int) extends Partitioner {
	require(partitions >= 0, s"Number of partitions ($partitions) cannot be negative.")
def numPartitions: Int = partitions
def getPartition (key: Any): Int = key match {
	case null => 0
	case _ => Utils.nonNegativeMod (key.hashCode, numPartitions)
	}
override def equals (other: Any): Boolean = other match {
		case h: HashPartitioner =>
		h.numPartitions == numPartitions
		case _ =>
		false
	}
	override def hashCode: Int = numPartitions
}
~~~

##### Range分区

Range 分区：将一定范围内的数据映射到一个分区中，**尽量保证每个分区数据均匀**，而且分区间有序 。

##### 自定义分区

1. 需要继承Partitioner接口
2. 重写getPartition方法，根据key返回对应的分区号

~~~ java
object Spark_RDD_partition {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)
		val rdd = context.makeRDD(List(
			("nba", "csxgsx"), ("wcba", "abucd"), ("cba", "cxscdxs"), ("nba", "csxdwqxs")
		))
		//如果直接在这里分片，那么spark会按照自己的规则进行分区，不会按照我们的意思进行分区、
		//而我们要自定义某些数据分到固定的分区，所以需要自定义
		//现在对我们的数据进行分区
		val parRdd: RDD[(String, String)] = rdd.partitionBy(new MyPartition())

		//保存到分区文件
		parRdd.saveAsTextFile("par")


		context.stop()
	}

	/**
	 * 自定义分区器
	 * 1 继承Partitioner类
	 * 2 重写方法
	 */

	class MyPartition extends Partitioner {

		//分区的数量
		override def numPartitions: Int = 3

		//根据key值返回数据所在的分区索引，从0开始分区
		override def getPartition(key: Any): Int = {

			//使用模式匹配
			key match {
				case "nba" => {
					0
				}
				case "wcba" => {
					1
				}
				case "cba" => {
					2
				}
				//	其他情况也放进分区2中
				case _ => {
					2
				}

			}

			//	if(key == "nba"){
			//		0
			//	}else if(key == "wcba"){
			//		1
			//	}else if(key == "cba"){
			//		2
			//	}else
			//		{
			//			2
			//		}
			//}
		}
	}
}
~~~

**案例二**

~~~ java
object Test07 {

	def main(args: Array[String]): Unit = {
		val sparkConf: SparkConf = new SparkConf().setAppName("wc").setMaster("local[*]")
		val sc: SparkContext = new SparkContext(sparkConf)
		sc.setLogLevel("WARN")

		val rdd1: RDD[String] = sc.parallelize(List("hadoop", "spark", "flink"), 4)
		println(rdd1.getNumPartitions) //4

		//注意：重分区:改变的是新生成的RDD的分区数,原来的RDD的分区数不变
		//-1.repartition,可以增加和减少分区
		val rdd2: RDD[String] = rdd1.repartition(5)
		println(rdd1.getNumPartitions) //4,原来rdd的分区个数
		println(rdd2.getNumPartitions) //5 新生成的RDD的分区的个数

		val rdd3: RDD[String] = rdd1.repartition(3)
		println(rdd1.getNumPartitions) //4
		println(rdd3.getNumPartitions) //3 ，只是改变新生成的RDD的分区的个数

		//-2.coalesce,默认只能减少分区
		val rdd4: RDD[String] = rdd1.coalesce(5)
		println(rdd1.getNumPartitions) //4
		println(rdd4.getNumPartitions) //4
		val rdd5: RDD[String] = rdd1.coalesce(3)
		println(rdd1.getNumPartitions) //4
		println(rdd5.getNumPartitions) //3


		//-3.默认按照key的hash进行分区
		val resultRDD: RDD[(String, Int)] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\word")
			.flatMap(_.split(" "))
			.map((_, 1)) //_表示每一个单词
			.reduceByKey(_ + _)

		resultRDD.foreachPartition(iter => {
			iter.foreach(t => {
				val key: String = t._1
				val num: Int = TaskContext.getPartitionId()
				println(s"默认的hash分区器进行的分区:分区编号:${num},key:${key}")
			})
		})

		//-4.自定义分区
		val resultRDD2: RDD[(String, Int)] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\word")
			.flatMap(_.split(" "))
			.map((_, 1)) //_表示每一个单词
			.reduceByKey(new MyPartitioner(1), _ + _)
		resultRDD2.foreachPartition(iter => {
			iter.foreach(t => {
				val key: String = t._1
				val num: Int = TaskContext.getPartitionId()
				println(s"自定义分区器进行的分区:分区编号:${num},key:${key}")
			})
		})

		sc.stop()
	}

	//自定义的分区器
	class MyPartitioner(partitions: Int) extends Partitioner {
		override def numPartitions: Int = partitions

		override def getPartition(key: Any): Int = {
			0
		}
	}
}
~~~

##### 面试题目

在实际开发中，什么时候需要调整RDD的分区数目呢？

- 当处理的数据很多的时候，可以考虑增加RDD的分区数以提高并行度

![1621856389409](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/193957-381716.png)

- **当对RDD数据进行过滤操作（filter函数）后，考虑减少分区数**

![1621856439352](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/171040-938741.png)

- **当对结果RDD存储到外部系统，考虑减少分区数**

![1621856487838](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/194134-228933.png)

##### RDD 文件读取与保存 

Spark 的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。

- 文件格式分为： text 文件、 csv 文件、 sequence 文件以及 Object 文件；
- 文件系统分为：本地文件系统、 HDFS、 HBASE 以及数据库。 

**读取文本文件**

~~~ java
object Spark_RDD_FileSave {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		//读取text文件,设置1个分区
		val readFile = context.textFile("datas/file",1)

		//保存读取的文件
		readFile.saveAsTextFile("file")


		context.stop()
	}
}
~~~

**Sequence文件**

SequenceFile 文件是 Hadoop 用来存储二进制形式的 key-value 对而设计的一种平面文件(Flat File)。 在 SparkContext 中，可以调用` sequenceFile [keyClass, valueClass](path)`。 

~~~ java
object Spark_RDD_FileSaveSeq {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val rdd = context.makeRDD(List(
			("nba", "csxgsx"), ("wcba", "abucd"), ("cba", "cxscdxs"), ("nba", "csxdwqxs")
		))
		//然后保存为seq文件
		rdd.saveAsSequenceFile("seqFile")


		//读取seq文件
		val value:RDD[(String,String)] = context.sequenceFile[String, String]("seqFile")

		value.collect().foreach(println)
		context.stop()
	}
}
~~~

**Object对象文件**

对象文件是将对象序列化后保存的文件，采用 Java 的序列化机制。 可以通过` objectFile[T:ClassTag](path)`函数接收一个路径， 读取对象文件， 返回对应的 RDD， 也可以通过调用saveAsObjectFile()实现对对象文件的输出。因为是序列化所以要指定类型。 

~~~ java
object Spark_RDD_FileSaveObj {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val rdd = context.makeRDD(List(
			("nba", "csxgsx"), ("wcba", "abucd"), ("cba", "cxscdxs"), ("nba", "csxdwqxs")
		))
		//然后保存为对象类型文件
		rdd.saveAsObjectFile("objFile")


		//读取obj文件
		val value = context.objectFile[Tuple2[String,String]]("objFile")
		value.collect().foreach(println)
		context.stop()
	}
}
~~~

### RDD的分区和Shuffle

RDD分区的作用

- RDD通常需要读取外部系统的数据文件进行创建RDD，外部存储系统往往**支持分片操作**，分片侧重于存储，分区侧重于计算，所以RDD需要支持分区来和外部系统的分片进行一一对应，
- RDD是一个并行计算的实现手段

![1616978719222](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/132943-57112.png)

#### 查看分区的方式

1.  通过webUi方式进行查看
2. 通过rdd.partitions.size方法进行查看

#### 指定RDD分区个数的方式

1. 通过本地集合创建的时候指定分区

~~~ java
val rdd = context.makeRDD(List(1, 2, 3, 4, 5),3)//指定三个分区
  rdd.partitions.size//查看分区数
~~~

2. 通过文件创建RDD时候指定分区数

~~~ java
val value1: RDD[String] = context.textFile("path", 5)
		value1.partitions.size
//但是这里指定的分区数是最小的分区数，一实际分区数比这个值大
~~~

#### 重定义分区数

coalesce()这个方法默认是不进行shuffle操作的，所以也就限制了在分区的时候只能把分区的个数变小，不能增大分区的个数，如果想要增大分区的个数，必须把shuffle操作设置为true,每一次调用这个方法都会生成新的Rdd，改变分区也是在新的RDD上面改变分区的个数，不会再旧的分区上面改变RDD分区的个数。

![1622180675786](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/134436-379378.png)

repartition():这个方法不仅可以增加分区个数，还可以减少分区的个数，只有一个参数可以设置。在repartition的底层，仍然使用的是coalease()方法，并且shuffle永远指定为true.

~~~ java
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
  //可以看到底层调用的是coalesce
    coalesce(numPartitions, shuffle = true)
  }
~~~

#### 通过其他算子指定分区个数

reduceByKey():此方法有一个参数可以指定分区的个数，含义是指定新生成的RDD生成的分区的个数。

~~~ java
//重载的方法，可以指定分区的个数
def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)] = self.withScope {
    reduceByKey(new HashPartitioner(numPartitions), func)
  }
~~~

groupByKey:也可以指定分区的个数

joink（）：此方法也可以指定分区的个数

很多方法都可以指定分区的个数，都有重载的方法，可以重新指定分区的个数。**一般涉及shuffle操作的方法都可以重新指定分区的个数（因为shuffle操作会导致子RDD中的数据有多个数据源，所以一般可以重新设置分区个数）**。如果没有指定分区的个数，那么就会从父级的RDD中继承分区个数。

**分区函数**

![1616981407434](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/133747-903556.png)

默认使用的是Hash算法进行分区操作，首先获取key的哈希码，然后哈希值对分区的个数取余，就可以存储到对应的分区当中，我们也可以重写partitoner函数自定义分区操作。

**分区接口**

~~~ java
//分区是一个抽象类
abstract class Partitioner extends Serializable {
  def numPartitions: Int
  def getPartition(key: Any): Int
}
//默认的hash分区也是集成子这个抽象类
class HashPartitioner(partitions: Int) extends Partitioner{}
~~~

#### RDD中的shuffle过程

**什么是shuffle**

![1622182893936](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/142155-875814.png)

如何确定map端的数据分发到reduce端的哪一个分区---->通过分区函数确定，默认使用HashPartitions

![1622183172936](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/142718-591744.png)

![1622183258139](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/142742-817591.png)

partitoner用于计算数据发送到那一台机器上面。

Hash base和Sort Base用于描述中间过程如何存储文件。

**Hash base shuffle**

![1616985508729](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/141922-12632.png)

如果使用Hash base ,右端有2个reduce，那么每一个map端应该输出两个文件，（发往reduce1的数据存储到第一个文件当中，发往reduce2的数据存储到第二个文件当中），然后reduce去各自的文件中拉去数据，每一个map都是如此操作。

HashBase shuffle会在nap端把准备发往reducer端的数据进行份文件存储，然后reducer端可以根据自己分区的数据区各个map端输出额文件中拉取数据，但是这种方式的效率非常的底下，比如有1000个map和1000个reducer，那么中间会生成1000*1000g个临时的文件，所以非常占用资源

mapreduce没有使用hash base shuffle，但是spark RDD使用的是hash base shuffle

**Sort base shuffle**

![1616986258409](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/28/141832-409739.png)

对于每一个map需要输出的数据，不在根据reducer的个数进行划分然后输出到磁盘文件存储，而是把map中的所有输出数据按照追加的方式输出到一个文件当中，然后对这个大文件，首先按照partition id进行排序操作，然后按照key的哈希码进行排序操作，但是所有的数据全部存储在一个集合中，可以想象为mr中的环形缓冲区一样，然后把数据分发到各个reducer端，这个大文件中间有分界符，按照分界符把每一个数据分发到reduce端。

这种方法可以明显解决临时文件过多的问题

### Spark原理

#### 总体概述

![1617162057870](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/31/114100-22630.png)

##### 再看WordCount案例

~~~ java
object Test18 {

	def main(args: Array[String]): Unit = {

		val conf = new SparkConf().setMaster("local[6]").setAppName("wordcount")
		val sc = new SparkContext(conf)

		val sourceData: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\word")

		val flatmapData: RDD[String] = sourceData.flatMap(item => {
			val words: Array[String] = item.split(" ")
			words
		})

		val mapData: RDD[(String, Int)] = flatmapData.map(item => {
			(item, 1)
		})

	//	做最后的数据统计
	val reduceData: RDD[(String, Int)] = mapData.reduceByKey((curr, agg) => {
		curr + agg
	})

		//最终结果转换为字符串
		val strRDD: RDD[String] = reduceData.map(item => {
			s"${item._1},${item._2}"
		})

		strRDD.collect().foreach(println(_))

		sc.stop()
	}
}
~~~

##### 再看Spark集群

![1622545211777](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/190016-688571.png)

守护进程，可以理解为一般不做什么工作，只是负责管理一台机器的资源。

为什么把一个节点叫做master或者是worker，因为在master节点上运行了一个master daeman进程负责接收用户提交的作业，而在worker节点上面，运行着worker daeman进程，负责执行master分发的任务，所以运行master daemon进程的节点我们叫做master节点，而运行worker daemon进程的节点叫做worker节点，他们都负责自己所在的节点的事务处理。

另外，workerdaemo进程还负责启动executor进程去执行作业，executor进程运行在一个容器当中。这个容器叫做Executor Backend，worker daemon就是通过Executor Backend来管理我们的Executor进程的。每一个Executor Backend只负责管理一个Executor进程,也就是说Executor BackendJVM实例持有一个Executor对象。

Driver是整个应用程序的驱动节点，负责整个作业的具体执行。当所有节点把任务执行完毕之后，所有的结果最终会汇总到Driver节点然后进行输出。其实Action操作获取结果是把结果发送给Dreiver进程。

##### 逻辑执行图

逻辑执行图就是描述数据如何进行流动，如何计算。

![1622546628507](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/192356-531101.png)

`println(strRDD.toDebugString)`可以打印程序的逻辑执行图。

~~~ java
(2) MapPartitionsRDD[5] at map at Test18.scala:31 []
 |  ShuffledRDD[4] at reduceByKey at Test18.scala:26 []
 +-(2) MapPartitionsRDD[3] at map at Test18.scala:21 []
    |  MapPartitionsRDD[2] at flatMap at Test18.scala:16 []
    |  D:\soft\idea\work\work04\src\main\resources\word MapPartitionsRDD[1] at textFile at Test18.scala:14 []
    |  D:\soft\idea\work\work04\src\main\resources\word HadoopRDD[0] at textFile at Test18.scala:14 []
~~~

可以看到，strRDD是一个MapPartitionsRDD,他依赖于其父RDD，父RDD是一个ShuffleRDD,也就是ReduceByKey产生的RDD，具体的过程通过下面的图说明：

![1622546939803](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/192907-142590.png)

> 逻辑执行图就是RDD之间的依赖关系，也就是多个RDD之间形成的链条，描述的是数据处理步骤。但是这个逻辑计划并不能放到集群中去执行，需要转换为物理计划放到集群中去执行。

##### 物理执行图

物理执行图就是描述RDD如何放到集群中运行，

![1622547745733](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/194230-668707.png)

阶段的划分是按照是否有shuffle操作来划分的，比如上图中，在shuffleRDD之前的操作，分区的个数没有改变，所以叫做一个阶段stage,而一个阶段中，每一个分区上的操作叫做任务，上面的阶段中有三个分区，每一个分区上的算子都是一个任务。

#### 逻辑执行计划

###### 明确边界

在逻辑图中研究的就是数据的流转方向。描述的是数据的处理和存储过程的表达。

![1622548395511](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/195334-657438.png)

~~~ java
override def compute(theSplit: Partition, context: TaskContext): InterruptibleIterator[(K, V)]
//这个方法本来是RDD的方法，在HadoopRDD中重新复写了这个方法，也就是说改变了RDD的计算方式。
//HadoopRDD继承自RDD类，所以需要重写里面的一些方法
~~~

![1622613397614](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/135642-733667.png)

###### RDD的生成

**textFile算子原理**

![1622548508367](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/195512-340497.png)

![1622548555942](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/195605-459688.png)

> 继承RDD的类，都需要进行重写五大属性，根据这五大属性来实现不同的功能。例如HadoopRDD就描述了每一个分区对应hdfs中的一个块数据，compute计算函数就是从块中读取数据到每一个分区当中。

**下面我们也可以查看map算子背后生成的RDD**

![1622548963630](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/200735-527506.png)

![1622549275990](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202111/06/160325-681790.png)

**flatMap算子**

![1622549407179](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/01/201235-214972.png)

![1622613367214](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/135610-22420.png)

######  RDD之间的依赖关系

RDD之间的关系，其实就是分区之间的关系。

对于窄依赖，其对应的RDD的分区可以放在一个流水线上面执行，也就是可以放在一个task中去执行。而宽依赖中间有shuffle过程，必须等待所有rdd执行完毕后才可以执行，所以不能放在一个task中去执行。

**一对一关系**

![1622607330697](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/121532-673360.png)

**多对一关系**

![1622608740390](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/123903-632422.png)

**窄依赖**

- 求笛卡尔积操作

~~~ java
object WordCountTest {

	def main(args: Array[String]): Unit = {

		// 创建 Spark 运行配置对象
		val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

		// 创建 Spark 上下文环境对象（连接对象）
		val sc: SparkContext = new SparkContext(sparkConf)

		val rdd1: RDD[Int] = sc.parallelize(Seq(1, 2, 3, 4, 5, 6))
		val rdd2: RDD[Char] = sc.parallelize(Seq('a', 'b', 'c'))

		//求两个rdd的笛卡尔积
		val rdd3: RDD[(Int, Char)] = rdd1.cartesian(rdd2)

		rdd3.collect().foreach(print)
		//关闭 Spark 连接
		sc.stop()

	}

}
~~~

![1622609372039](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/125534-308512.png)

上面的rddc的p1分区只依赖了rdda中的p1分区，并且还依赖了rddb中的p1分区，并没有依赖一个rdd中的多个分区，所以可以认为rddc中的p1分区只依赖了一个分区，所以上面的数据流向是一个窄依赖，不是一个宽依赖。shuffle操作一般有分区操作。

![1622610307898](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/130511-159092.png)

从源码角度查看是宽窄依赖

~~~ java
class CartesianRDD[T: ClassTag, U: ClassTag](
    sc: SparkContext,
    var rdd1 : RDD[T],
    var rdd2 : RDD[U])
  extends RDD[(T, U)](sc, Nil)
  with Serializable {
  //
   override def getDependencies: Seq[Dependency[_]] = List(
    new NarrowDependency(rdd1) {
      def getParents(id: Int): Seq[Int] = List(id / numPartitionsInRdd2)
    },
    new NarrowDependency(rdd2) {
      def getParents(id: Int): Seq[Int] = List(id % numPartitionsInRdd2)
    }
  )
}
//CartesianRDD重写了父类的getDependencies方法，我们可以看到，返回的是一个NarrowDependency
~~~

宽窄依赖的源码

~~~ java
@DeveloperApi
abstract class Dependency[T] extends Serializable {
  def rdd: RDD[T]
}


/**
 * :: DeveloperApi ::
 * Base class for dependencies where each partition of the child RDD depends on a small number
 * of partitions of the parent RDD. Narrow dependencies allow for pipelined execution.
 */
//窄依赖
@DeveloperApi
abstract class NarrowDependency[T](_rdd: RDD[T]) extends Dependency[T] {
  /**
   * Get the parent partitions for a child partition.
   * @param partitionId a partition of the child RDD
   * @return the partitions of the parent RDD that the child partition depends upon
   */
  def getParents(partitionId: Int): Seq[Int]

  override def rdd: RDD[T] = _rdd
}


/**
 * :: DeveloperApi ::
 * Represents a dependency on the output of a shuffle stage. Note that in the case of shuffle,
 * the RDD is transient since we don't need it on the executor side.
 *
 * @param _rdd the parent RDD
 * @param partitioner partitioner used to partition the shuffle output
 * @param serializer [[org.apache.spark.serializer.Serializer Serializer]] to use. If not set
 *                   explicitly then the default serializer, as specified by `spark.serializer`
 *                   config option, will be used.
 * @param keyOrdering key ordering for RDD's shuffles
 * @param aggregator map/reduce-side aggregator for RDD's shuffle
 * @param mapSideCombine whether to perform partial aggregation (also known as map-side combine)
 * @param shuffleWriterProcessor the processor to control the write behavior in ShuffleMapTask
 */
//宽依赖
@DeveloperApi
class ShuffleDependency[K: ClassTag, V: ClassTag, C: ClassTag](
    @transient private val _rdd: RDD[_ <: Product2[K, V]],
    val partitioner: Partitioner,
    val serializer: Serializer = SparkEnv.get.serializer,
    val keyOrdering: Option[Ordering[K]] = None,
    val aggregator: Option[Aggregator[K, V, C]] = None,
    val mapSideCombine: Boolean = false,
    val shuffleWriterProcessor: ShuffleWriteProcessor = new ShuffleWriteProcessor)
  extends Dependency[Product2[K, V]] {

  if (mapSideCombine) {
    require(aggregator.isDefined, "Map-side combine without Aggregator specified!")
  }
  override def rdd: RDD[Product2[K, V]] = _rdd.asInstanceOf[RDD[Product2[K, V]]]

  private[spark] val keyClassName: String = reflect.classTag[K].runtimeClass.getName
  private[spark] val valueClassName: String = reflect.classTag[V].runtimeClass.getName
  // Note: It's possible that the combiner class tag is null, if the combineByKey
  // methods in PairRDDFunctions are used instead of combineByKeyWithClassTag.
  private[spark] val combinerClassName: Option[String] =
    Option(reflect.classTag[C]).map(_.runtimeClass.getName)

  val shuffleId: Int = _rdd.context.newShuffleId()

  val shuffleHandle: ShuffleHandle = _rdd.context.env.shuffleManager.registerShuffle(
    shuffleId, this)

  _rdd.sparkContext.cleaner.foreach(_.registerShuffleForCleanup(this))
  _rdd.sparkContext.shuffleDriverComponents.registerShuffle(shuffleId)
}


/**
 * :: DeveloperApi ::
 * Represents a one-to-one dependency between partitions of the parent and child RDDs.
 */
@DeveloperApi
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd) {
  override def getParents(partitionId: Int): List[Int] = List(partitionId)
}


/**
 * :: DeveloperApi ::
 * Represents a one-to-one dependency between ranges of partitions in the parent and child RDDs.
 * @param rdd the parent RDD
 * @param inStart the start of the range in the parent RDD
 * @param outStart the start of the range in the child RDD
 * @param length the length of the range
 */
@DeveloperApi
class RangeDependency[T](rdd: RDD[T], inStart: Int, outStart: Int, length: Int)
  extends NarrowDependency[T](rdd) {

  override def getParents(partitionId: Int): List[Int] = {
    if (partitionId >= outStart && partitionId < outStart + length) {
      List(partitionId - outStart + inStart)
    } else {
      Nil
    }
  }
}
~~~

**宽依赖**

什么是宽依赖

注：宽窄依赖的分辨就是看是否有shuffle操作

![1622610742744](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/131239-438484.png)

上图中我们可以看到，RDDB中的每一个分区都依赖于RDDA中的三个分区，所以是shuffle操作。判断是不是宽依赖就看有没有shuffle操作，判断有没有shuffle操作就看子RDD和父RDD之间是一对一的关系还是多对一的关系，如果是多对一的关系，那么就是shuffle操作。简单来说shuffle就是一种广播的操作。

>  上面我们可以看到,rddb中的p1分区依赖了rdda中的三个分区，是一种zshuffle操作

如何分辨宽窄依赖

![1617344256206](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/132255-450350.png)

![1617343496571](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/02/141432-340187.png)

下面这种多对一关系不是宽依赖，因为前两个分区只是把数据合并了一下，然后复制到下一个分区，并没有做数据的分发工作。

![1617344097042](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/02/141538-508253.png)

下面这种才是shuffle操作，因为每一个父分区都在分发自己分区内的数据给子分区。

![1617344182939](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/132430-502778.png)

通过rdd中的dependence()函数可以查看依赖关系，默认的rdd是窄依赖。如果看到一个rdd没有重写getDependence()方法，那么默认使用的的就是父类的。依赖关系，而父类默认使用的是窄依赖关系。

依赖的继承关系

![1622612446436](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/134051-300436.png)

常见的窄依赖

![1622612974187](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/134941-367502.png)

![1622612946642](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/134945-924220.png)

![1622613097877](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/10/085743-164106.png)

比如有3个RDD是窄依赖关系，那么这三个RDD的分区是可以放到一个task中去执行。

![1622613338810](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/135541-925463.png)

**小结**

![1622606829462](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/121430-17428.png)

划分宽窄依赖入党依据是是否有shuffle操作。

#### 物理执行图

##### 物理执行图的作用

![1622614085476](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622614085476.png)

![1622614314085](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/141158-206279.png)

![1622614451993](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622614451993.png)

![1622614612547](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622614612547.png)

##### RDD的计算-Task

![1622616060811](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/144656-674997.png)

这样设计task的话，每一个RDD中有多个分区，每一个分区设计一个task的话，多个算子之间进行数据传输很消耗资源，基本上和hadoop的mr一样，

![1622616732425](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/145215-673416.png)

如果是这样的话，中间有shuffle操作的话，这就要等待其他所有分区的数据全部执行完毕后才可以继续向下执行

![1622616821048](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/145344-237906.png)

##### 如何划分阶段

![1622617073537](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/145758-31966.png)

stage的划分，根是否有shuffle操作，把处理数据的流划分为多个阶段，有shuffle，就断开形成一个stage。

![1622617171995](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/145935-429264.png)

##### 数据如何流动

![1622617610914](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/150654-28529.png)

触发数据的计算发生在需要数据的地方，也就是最后由Action算子触发。

#### 运行过程

##### 首先生成逻辑图

![1622618028000](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/151354-487968.png)

![1622618056836](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/151421-195877.png)

##### 物理图

![1622618108237](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/151514-770304.png)

Action会触发job的运行，调用Action操作，首先会调用runJob(）方法，然后再会调用dagScheduler方法，dagScheduler的作用就是把生成的逻辑执行计划转换为一个一个的Stage和Task任务，然后放到物理机器上面执行，最后调用taskScheduler把任务分发到集群中运行。

![1622618380841](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/10/092454-532883.png)

##### Job和Stage的关系

![1622618806873](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622618806873.png)

一个job中有多个Stage，多个Stage之间是串行的关系。

![1622619333586](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/153542-703365.png)

##### Stage和Task的关系

![1622619467380](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622619467380.png)

一个Stage中可能有多个Task，一个Stage就是一组Task在运行。

![1622619542646](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622619542646.png)

![1622619586695](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/02/153951-697348.png)

一个Stage对应一个TaskSet，可以把TaskSet想象为线程池。

![1622619697307](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622619697307.png)

##### 整体执行流程

![1622619827049](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622619827049.png)



### 累加器 

#### 为什么要累加器

~~~ java
object Spark_RDD_Acc01 {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[Int] = context.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9))

		//val num:Int= makRdd.reduce((num1, num2) => {
		//	num1 + num2
		//})
      //输出45

		var sum:Int =0
		makRdd.foreach(
			num => {
				sum += num
			}
		)
      //输出0


		println(sum)
		context.stop()
	}
}
~~~

上面的程序中，我们省略了使用reduce进行聚合，因为其中有shuffle操作，很耗时间，所以采用下面逐个遍历元素进行累加的操作，但是发现最后结果是0，这是因为在driver端定义的sum传递给每一个executor端之后，每一个executor端会进行累加操作，但是累加之后的sum并不会再次传给driver端进行汇总操作，所以最终输出结果是0，可以用下面这张图片表示：

![1615078034340](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1615078034340.png)

- 要解决上面的问题需要使用spark中的累加器操作，累加器会从每一个executor端返回结果到driver端做最后的汇总操作

![1615078087761](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/07/085553-704602.png)

**使用累加器实现**

~~~ java
object Spark_RDD_Acc022 {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[Int] = context.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9))

		//spark默认提供简单的数据累加器

		val sumAcc = context.longAccumulator("sum")

		context.doubleAccumulator("double")
		context.collectionAccumulator("coll")
		//		使用累加器进行累加
		makRdd.foreach(
			num=>{
				sumAcc.add(num)
			}
		)

		//获取累加器的值,结果是45
		println(sumAcc.value)
		context.stop()
	}
}
~~~

#### 实现原理

累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行 merge。 

#### 系统累加器

~~~ java
object Spark_RDD_Acc022 {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[Int] = context.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9))

		//spark默认提供简单的数据累加器

		val sumAcc = context.longAccumulator("sum")

		context.doubleAccumulator("double")
		context.collectionAccumulator("coll")
		//		使用累加器进行累加
		makRdd.foreach(
			num=>{
				sumAcc.add(num)
			}
		)

		//获取累加器的值,结果是45
		println(sumAcc.value)
		context.stop()
	}
}
~~~

#### 可能出现的问题

~~~ java
object Spark_RDD_Acc03 {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[Int] = context.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9))

		//spark默认提供简单的数据累加器

		val sumAcc = context.longAccumulator("sum")

		context.doubleAccumulator("double")
		context.collectionAccumulator("coll")
		//		使用累加器进行累加
		//累加器数据少加，转换算子中调用累加器，如果没有行动算子的话，不会执行，所以数据少加
		// 0
		val value = makRdd.map(
			num => {
				sumAcc.add(num)
				num
			}
		)
		//累加器值多加的情况，行动算子每调用一次，累加器就会重新加一遍
		//一般情况下，累加器放在行动算子中操作
		value.collect().foreach(println)//90
		value.collect().foreach(println)//90
		//获取累加器的值,结果是45


		println(sumAcc.value)
		context.stop()
	}
}
~~~

- 转换算子中调用累加器，如果没有行动算子的话，那么就不会执行操作
- 多次调用行动算子，会多次执行累加操作。

> 分布式共享只写变量：executor端可能有多个累加变量，但是这多个累加变量之间不可以相互访问，只有全部返回到driver端，由driver进行汇总操作。

#### 自定义累加器

- 继承抽象类AccumulatorV2，定义泛型 in,out
- 重写继承的方法

~~~ java
object Spark_RDD_Acc_wordcount {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd:RDD[String] = context.makeRDD(List("hello","spark","scala"))

		//创建累加器对象
		val wcAcc=new MyAccumulator()

		//向spark进行注册
		context.register(wcAcc,"wordcount")
		//context.longAccumulator()


		makRdd.foreach(
			word=>{
		wcAcc.add(word)
			}
		)

		//获取累加器执行的结果
		println(wcAcc.value)
		context.stop()
	}

//	自定义累加器实现
	/**
	 * 1, 继承抽象类AccumulatorV2，定义泛型 in,out
	 * 		IN:累加器输入类型String
	 * 		OUT:累加器返回类型Map[String,Long]
	 * 2,重写方法
	 */
	class MyAccumulator extends AccumulatorV2[String,mutable.Map[String,Long]]{

		//判断是否初始状态
		override def isZero: Boolean ={
			//如果是空，那么就是初始状态
			wcMap.isEmpty
		}

		private  var wcMap= mutable.Map[String,Long]()

		/**
		 * 复制一个新的累加器
		 * @return
		 */
		override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] ={
			new MyAccumulator()
		}

		/**
		 * 重置累加器
		 */
		override def reset(): Unit ={
			wcMap.clear()
		}

		/**
		 * 获取累加器需要累加的值
		 * @param v 输入值
		 */
		override def add(v: String): Unit ={
			//判断map中是否有当前出现的单词，+1就是加当前出现的单词
			var cnt=wcMap.getOrElse(v,0l)+1
		//	更新map集合
			wcMap.update(v,cnt)
		}

		/**
		 * 在zdriver端进行合并操作
		 * 两个map类型放入合并
		 * @param other
		 */
		override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit ={
			val map1=this.wcMap
			val map2=other.value

		//	合并两个集合，对map2进行遍历操作
			map2.foreach {
				//		更新的结果全部放在map1中
				case (word, num) => {}
					val newCnt = map1.getOrElse(word, 0l) + num
					map1.update(word, newCnt)
			}
		}

		/**
		 * 获取累加器的结果
		 * @return
		 */
		override def value: mutable.Map[String, Long] ={
			//返回map集合
			wcMap
		}
	}

}
~~~

### 广播变量

#### 为什么要广播变量

~~~ java
object Spark_RDD_Acc_broad {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd1:RDD[(String,Int)] = context.makeRDD(List(("a",1),("b",2),("c",3)))
		val makRdd2:RDD[(String,Int)] = context.makeRDD(List(("a",4),("b",5),("c",6)))

		val map=mutable.Map(("a",4),("b",5),("c",6))

		/**
		 * 使用join的过程中，中间操作可能出现shuffle或者笛卡尔积操作，影响性能，不推荐使用
		 */
		val value = makRdd1.join(makRdd2)
		value.collect().foreach(println)

		/**
		 * (a,(1,4))(b,(2,5))(c,(3,6))
		 * ("a",1),("b",2),("c",3)
		 */
		makRdd1.map {
			case (w, c) => {
				val l = map.getOrElse(w, 0l)
				(w,(c,l))
			}

		}.collect().foreach(println)




		context.stop()
	}

}
~~~

**图解**

![1615084035971](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1615084035971.png)

这样是以任务为单位进行分配map的数据，所以如果数据量很大的情况下，很可能发生内存溢出，因为存在大量的冗余数据。

**广播变量**

![1615084113558](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1615084113558.png)

以一个jvm进程为单位进行分配数据，多个任务之间共享一个进程中的数据，减少数据的冗余

![1615084163375](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/07/102926-950794.png)

#### 实现原理

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送 

#### 案例使用

~~~ java
object Spark_RDD_Acc_broad_ {


	def main(args: Array[String]): Unit = {

		//准备环境,local[*]:local表示本地环境，[*]表示使用本机默认的核数
		val conf = new SparkConf().setMaster("local[*]").setAppName("operator")

		//	创建上写文
		val context = new SparkContext(conf)

		val makRdd1:RDD[(String,Int)] = context.makeRDD(List(("a",1),("b",2),("c",3)))
		val map=mutable.Map(("a",4),("b",5),("c",6))
		//封装广播变量
		val bc:Broadcast[mutable.Map[String,Int]] = context.broadcast(map)

		makRdd1.map {
			case (w, c) => {
				//在这里使用广播变量
				val l = bc.value.getOrElse(w, 0l)
				(w,(c,l))
			}

		}.collect().foreach(println)

		context.stop()
	}

}
~~~

## 三层架构模式

![1615379041545](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/11/082851-555349.png)

ThreadLocal可以对线程的内存进行控制，存储数据，共享数据，相当于内存中的一块存储区域，用来使多个线程之间可以共享数据，但是不能保证线程安全问题。

### 三层架构模式代码实现

#### 模式包名

![1615422693537](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1615422693537.png)

#### application层

程序执行的入口

~~~ java
package qq.com.Spark_core.framework.application

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD
import qq.com.Spark_core.framework.common.TApplication
import qq.com.Spark_core.framework.controller.WordCountController

object WordCountApplication extends App with TApplication{


		//启动应用程序
		start(){
			val controller=new WordCountController()
			//执行调度功能
			controller.dispatch()
		}
}
~~~

#### common层

这一层的类全部是特质，相当于java中的抽象类，是应用程序逻辑功能的高层抽象。

![1615422807548](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1615422807548.png)

**TApplication**

~~~ java
package qq.com.Spark_core.framework.common

import jdk.internal.util.EnvUtils
import org.apache.spark.{SparkConf, SparkContext}
import qq.com.Spark_core.framework.controller.WordCountController
import qq.com.Spark_core.framework.util.EnvUtil

trait TApplication {

	def start(master:String="local[*]",app:String="WordCount")(op : =>Unit):Unit={

		// 创建 Spark 运行配置对象
		val sparkConf = new SparkConf().setMaster(master).setAppName(app)

		// 创建 Spark 上下文环境对象（连接对象）
		val sc: SparkContext = new SparkContext(sparkConf)

		EnvUtil.put(sc)

		//在这里执行传进来的处理逻辑
		try{
		op
		}catch {
			case ex=>println(ex.getMessage)
		}

		//关闭 Spark 连接
		sc.stop()

		EnvUtil.clear()
	}
}
~~~

**Tcontroller**

~~~ java
trait TController {
	def dispatch():Unit
}
~~~

**TDao**

~~~ java
package qq.com.Spark_core.framework.common

import org.apache.spark.rdd.RDD
import qq.com.Spark_core.framework.util.EnvUtil

trait TDao {

	def readFile(path:String):Any

}
~~~

**TService**

~~~ java
package qq.com.Spark_core.framework.common

trait Tservice {
	def dataAnalysis():Any
}
~~~

#### util层

封装的是通用的功能，可以共享

**EnvUtil**

~~~ java
package qq.com.Spark_core.framework.util

import org.apache.spark.SparkContext

object EnvUtil {

	private val scLocal=new ThreadLocal[SparkContext]()

	def put(scL:SparkContext):Unit={
		scLocal.set(scL)
	}

	def take():SparkContext={
		scLocal.get()
	}

	def clear():Unit={
		scLocal.remove()
	}

}
~~~

#### Controller层

**wordcountcontroller**

~~~ java
package qq.com.Spark_core.framework.controller

import org.apache.spark.rdd.RDD
import qq.com.Spark_core.framework.common.TController
import qq.com.Spark_core.framework.service.WordCountService


/*
控制层
 */
class WordCountController extends TController{

	private val wordCountService=new WordCountService()

	//调度
	def dispatch():Unit={

		val word2Count = wordCountService.dataAnalysis()

		// 打印结果
		word2Count.foreach(println)

	}

}
~~~

#### Dao层

**wordcountDao**

~~~ java
package qq.com.Spark_core.framework.Dao

import org.apache.spark.rdd.RDD
import qq.com.Spark_core.framework.common.TDao
import qq.com.Spark_core.framework.util.EnvUtil

class WordCountDao extends TDao{

	def readFile(path:String):RDD[String]={
		// 读取文件数据
		EnvUtil.take().textFile(path)
	}
}
~~~

#### service层

**wordcountService**

~~~ java
package qq.com.Spark_core.framework.service

import org.apache.spark.rdd.RDD
import qq.com.Spark_core.framework.Dao.WordCountDao

import qq.com.Spark_core.framework.common.Tservice


/*
服务层
 */
class WordCountService extends Tservice{

	private val wordCountDao=new WordCountDao()

	//数据分析
	def dataAnalysis():Array[(String, Int)]={

		val fileRDD = wordCountDao.readFile("datas/word.txt")


		// 将文件中的数据进行分词
		val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))

		// 转换数据结构 word => (word, 1)
		val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_, 1))

		// 将转换结构后的数据按照相同的单词进行分组聚合
		val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_ + _)

		// 将数据聚合结果采集到内存中
		val word2Count: Array[(String, Int)] = word2CountRDD.collect()

		word2Count
	}

}
~~~

