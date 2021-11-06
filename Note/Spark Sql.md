# 深入理解SparkSql编程

<!-- TOC -->

- [深入理解SparkSql编程](#深入理解sparksql编程)
    - [概述](#概述)
        - [数据分析方式](#数据分析方式)
            - [命令式](#命令式)
            - [SQL](#sql)
            - [小结](#小结)
        - [Spark Sql历史](#spark-sql历史)
        - [Spark sql的应用场景](#spark-sql的应用场景)
            - [结构化数据](#结构化数据)
            - [半结构化数据](#半结构化数据)
            - [非结构化数据](#非结构化数据)
            - [Spark Sql](#spark-sql)
        - [DataFrame&Dataset](#dataframedataset)
            - [命令式API使用](#命令式api使用)
            - [声明式api的使用](#声明式api的使用)
        - [Catasyst优化器](#catasyst优化器)
            - [什么是catasyst优化器](#什么是catasyst优化器)
            - [catasyst的优化步骤](#catasyst的优化步骤)
    - [DataSet](#dataset)
        - [为什么需要Dataset](#为什么需要dataset)
        - [Dataset是什么](#dataset是什么)
        - [Dataset底层是什么](#dataset底层是什么)
    - [DataFrame](#dataframe)
        - [DataFrame是什么](#dataframe是什么)
        - [创建DataFrame](#创建dataframe)
            - [通过隐式转换](#通过隐式转换)
            - [通过集合创建](#通过集合创建)
            - [通过读取外部数据集](#通过读取外部数据集)
        - [操作DataFrame](#操作dataframe)
        - [DataSet和DataFrame的区别](#dataset和dataframe的区别)
        - [Row对象](#row对象)
        - [RDD、DataFrame和Dataset](#rdddataframe和dataset)
        - [小结](#小结-1)
    - [SparkSql实战](#sparksql实战)
        - [SparkSql初体验](#sparksql初体验)
        - [获取DataFrame/DataSet](#获取dataframedataset)
            - [使用样例类](#使用样例类)
            - [指定类型+列名](#指定类型列名)
            - [自定义Schema](#自定义schema)
        - [RDD、DF、DS相互转换](#rdddfds相互转换)
            - [转换API](#转换api)
            - [代码演示](#代码演示)
        - [SparkSQL花式查询](#sparksql花式查询)
            - [基于SQL查询](#基于sql查询)
            - [基于DSL分析](#基于dsl分析)
            - [代码演示](#代码演示-1)
    - [SparkSql扩展](#sparksql扩展)
        - [数据源与格式](#数据源与格式)
            - [DataFrameReader](#dataframereader)
            - [text数据读取](#text数据读取)
            - [Json数据读取](#json数据读取)
            - [CSV格式](#csv格式)
            - [Parquet数据](#parquet数据)
            - [jdbc数据](#jdbc数据)
            - [数据读取框架案例](#数据读取框架案例)
        - [DataFrameWriter](#dataframewriter)
        - [Parquet文件格式](#parquet文件格式)
            - [Parquet文件](#parquet文件)
            - [写入文件指定分区](#写入文件指定分区)
        - [JSON格式文件](#json格式文件)
    - [SpaqkSql整合Hive](#spaqksql整合hive)
        - [概述](#概述-1)
        - [spark-sql中集成Hive](#spark-sql中集成hive)
        - [SparkSql访问Hive表](#sparksql访问hive表)
            - [使用Hive创建表](#使用hive创建表)
            - [访问表](#访问表)
            - [使用SparkSql创建表](#使用sparksql创建表)
            - [SparkSql访问Hive](#sparksql访问hive)
        - [Hive的SQL交互方式](#hive的sql交互方式)
        - [SparkSQL交互方式](#sparksql交互方式)
                - [使用beeline 客户端连接](#使用beeline-客户端连接)
    - [SparkSql读写jdbc](#sparksql读写jdbc)
        - [创建库和表](#创建库和表)
        - [向mysql中写入数据](#向mysql中写入数据)
    - [数据操作](#数据操作)
        - [有类型的转换算子](#有类型的转换算子)
            - [map&flatMap&mapPartitions](#mapflatmapmappartitions)
            - [transform](#transform)
            - [as](#as)
            - [Filter](#filter)
            - [groupByKey](#groupbykey)
            - [randomSplit](#randomsplit)
            - [OrderBy&sort](#orderbysort)
            - [distinct&dropDuplicates](#distinctdropduplicates)
        - [集合操作](#集合操作)
        - [无类型的转换算子](#无类型的转换算子)
            - [选择](#选择)
            - [新建列](#新建列)
            - [剪除&聚合](#剪除聚合)
        - [Column对象](#column对象)
            - [Column对象的创建](#column对象的创建)
            - [操作](#操作)
            - [方法](#方法)
        - [缺失值处理](#缺失值处理)
            - [缺失值的产生](#缺失值的产生)
            - [缺失值的处理](#缺失值的处理)
                - [NaN&NULL缺失值处理](#nannull缺失值处理)
                - [字符串类型缺失值处理](#字符串类型缺失值处理)
        - [聚合操作](#聚合操作)
            - [groupBy](#groupby)
            - [多维聚合](#多维聚合)
            - [RelationalGroupedDataset](#relationalgroupeddataset)
        - [连接操作](#连接操作)
            - [交叉连接](#交叉连接)
            - [内连接](#内连接)
            - [全外连接](#全外连接)
            - [左外连接](#左外连接)
            - [右外连接](#右外连接)
            - [LeftAnti](#leftanti)
            - [LeftSemi](#leftsemi)
        - [函数](#函数)
        - [窗口函数](#窗口函数)
    - [小项目](#小项目)
        - [业务](#业务)
        - [流程分析](#流程分析)
        - [读取数据](#读取数据)
        - [数据转换](#数据转换)
        - [异常的处理](#异常的处理)
        - [减掉反常数据](#减掉反常数据)
        - [行政区信息](#行政区信息)
            - [需求分析](#需求分析)
            - [工具介绍](#工具介绍)
        - [最终代码](#最终代码)

<!-- /TOC -->

## 概述

### 数据分析方式

#### 命令式

~~~ java
sc.textFile("...")
  .flatMap(_.split(" "))
  .map((_, 1))
  .reduceByKey(_ + _)
  .collect()
~~~

**命令式的优点**

- 操作粒度更细，能够控制数据的每一个处理环节

- 操作更明确，步骤更清晰，容易维护

- 支持半/非结构化数据的操作

**命令式的缺点**

- 需要一定的代码功底

- 写起来比较麻烦

简单来说命令式就是借助了函数式编程的思想。

#### SQL

对于一些数据科学家/数据库管理员/DBA, 要求他们为了做一个非常简单的查询, 写一大堆代码, 明显是一件非常残忍的事情, 所以SQL on Hadoop 是一个非常重要的方向.

~~~ java
SELECT
   name,
   age,
   school
FROM students
WHERE age > 10
~~~

**SQL的优点**

- 表达非常清晰, 比如说这段 SQL 明显就是为了查询三个字段，条件是查询年龄大于 10 岁的

**SQL的缺点**

- 试想一下3层嵌套的 SQL维护起来应该挺力不从心的吧

- 试想一下如果使用SQL来实现机器学习算法也挺为难的吧

#### 小结

- SQL 擅长数据分析和通过简单的语法表示查询，命令式操作适合过程式处理和算法性的处理.

- 在 Spark 出现之前，对于结构化数据的查询和处理， 一个工具一向只能支持命令式如MR或者只能使用SQL如Hive，开发者被迫要使用多个工具来适应两种场景，并且多个工具配合起来比较费劲.

- 而 Spark 出现了以后，提供了两种数据处理范式：RDD的命令式和SparkSQL的SQL式，是一种革新性的进步！

### Spark Sql历史

![1621771965125](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/201245-712190.png)

**Hive**

![1621772005373](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/201611-979073.png)

hive延迟太高，因为其底层式基于MapReduce计算框架。

Hive是基于进程并行的，因为MapReduce是基于进程并行执行，spark是基于线程并行执行。

**Shark**

![1621772066975](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/201428-702440.png)

**Spark sql**

![1621772322369](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/201842-808650.png)

**DataSet**

![1621772387490](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/201948-394429.png)

现在spark sql不仅支持命令式api，还支持声明式api操作。

**Spark sql的重要性**

![1621772482838](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621772482838.png)

可以看到，spark streaming,Graphs和MLIBS都是基于spark sql的api进行实现的。spark sql底层是RDD。

### Spark sql的应用场景

**数据集的分类**

![1621772642159](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/23/202403-665086.png)

#### 结构化数据

![1621772662871](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621772662871.png)

- 字段有约束
- 字段类型也有约束

> 特指关系数据库中的表

#### 半结构化数据

![1621772730106](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621772730106.png)

- 有列
- 列有类型
- 但是没有严格的约束，可以任意的修改

![1621772820874](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621772820874.png)

#### 非结构化数据

音频视频数据都是非结构化的数据，没有固定的格式和约束。

#### Spark Sql

spark sql用于处理什么类型的数据？

![1621772940514](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621772940514.png)

虽然Spark sql是基于RDD的，但是SparkSql的速度比RDD快很多。spark sql在编写的时候可以通过更方便的结构化的api来进行更好的操作。

> SparkSql主要用于处理结构化数据

![1621860930981](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/205532-354757.png)

sparksession

- 因为老的sparkcontext已经不适用于sparksql
- sparksql需要读取更多的数据源，更多的数据写入

### DataFrame&Dataset

就易用性而言，对比传统的MapReduce API，Spark的RDD API有了数量级的飞跃并不为过。然而，对于没有MapReduce和函数式编程经验的新手来说，RDD API仍然存在着一定的门槛。

另一方面，数据科学家们所熟悉的R、Pandas等传统数据框架虽然提供了直观的API，却局限于单机处理，无法胜任大数据场景。

为了解决这一矛盾，Spark SQL 1.3在Spark1.0原有SchemaRDD的基础上提供了与R和Pandas风格类似的DataFrame API。

新的DataFrame AP不仅可以大幅度降低普通开发者的学习门槛，同时还支持Scala、Java与Python三种语言。更重要的是，由于脱胎自SchemaRDD，DataFrame天然适用于分布式大数据场景。

**注意:**

DataFrame它不是Spark SQL提出来的，而是早在R、Pandas语言中就已经有了的。

![1621860146739](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/204228-802067.png)

rdd中存储的对象，只能把person当做一个整体来看待，无法操作对象的属性信息，但是对于dataset和dataframe来说，可以直接操作Person中的某一个属性信息。

#### 命令式API使用

~~~ java
object Test01 {

	def main(args: Array[String]): Unit = {

		//构建者模式
		//Create SparkConf() And Set AppName
		val spark=  SparkSession.builder()
			.appName("Spark Sql basic example")
			.config("spark.some.config.option", "some-value")
			.master("local[6]")
			.getOrCreate()
		val ss=  SparkSession.builder()
			.appName("Spark Sql basic example")
			.config("spark.some.config.option", "some-value")
			.master("local[6]")
			.getOrCreate()

		//导入隐式转换
		import spark.implicits._
		//import ss.implicits._
		//	通过SparkSession对象使用sparkcontext对象
		val sourceRDD: RDD[Person] = spark.sparkContext.parallelize(Seq(Person("zhangsan", 12), Person("lisi", 24)))

		//toDS()是一个隐式转换，其实就是一个dataSet对象
		val personDs: Dataset[Person] = sourceRDD.toDS()

		val value: Dataset[String] = personDs.where('age > 10)
			.where('age < 20)
			.select('name)//可以拿到某一列
			.as[String]

		value.show()

	}
	case class Person(name:String,age: Int)
}
~~~

#### 声明式api的使用

也就是使用sql语句查询

~~~ java
object Test02 {

	def main(args: Array[String]): Unit = {

		//构建者模式
		//Create SparkConf() And Set AppName
		val spark=  SparkSession.builder()
			.appName("Spark Sql basic example")
			.config("spark.some.config.option", "some-value")
			.master("local[6]")
			.getOrCreate()


		//导入隐式转换
		import spark.implicits._
		//	通过SparkSession对象使用sparkcontext对象
		val sourceRDD: RDD[Person] = spark.sparkContext.parallelize(Seq(Person("zhangsan", 12), Person("lisi", 24)))

		//toDS()是一个隐式转换，其实就是一个dataSet对象
		val dsData: Dataset[Person] = sourceRDD.toDS()
		val dfData: DataFrame = sourceRDD.toDF()
		//首先创建一个临时表
		dfData.createOrReplaceTempView("personTemp")

		val res: DataFrame = spark.sql("select * from personTemp where age >10 and age < 20")
		res.show()

	}
	case class Person(name:String,age: Int)
}
~~~

![1621861934640](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/24/211215-516949.png)

下面图说明RDD，Dataset和DataFrame三者的结构

![1622027975957](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/191938-278834.png)

上图中左侧的RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解Person类的内部结构。而中间的DataFrame却提供了详细的结构信息，使得Spark SQL可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。了解了这些信息之后，Spark SQL的查询优化器就可以进行针对性的优化。后者由于在编译期有详尽的类型信息，编译期就可以编译出更加有针对性、更加优化的可执行代码。

**Schema 信息**

查看DataFrame中Schema是什么，执行如下命令：

~~~ java
df.schema
//我们可以看到，schema是StructType类型，schema定义在Row中
def schema: StructType = null
~~~

Schema信息封装在StructType中，包含很多StructField对象

> spark提供了sql和命令式api两种不同的方式访问结构化数据，并且他们之间可以无缝衔接

### Catasyst优化器

#### 什么是catasyst优化器

![1621901599674](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/081320-746534.png)

![1621901641194](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/081401-724308.png)

#### catasyst的优化步骤

![1621901779564](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/081621-958335.png)

执行计划的生成经过四个阶段：解析，优化，生成物理计划，运行，

**第一步**

![1621901919210](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621901919210.png)

不管什么语言，编译器在进行编译的时候，首先是进行解析工作，解析为语法树。

**第二步**

![1621901991722](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621901991722.png)

生成元数据信息，也就是给相应的数据添加类型信息。

**第三步**

优化，按照一定的规则进行优化

![1621902146243](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/082226-545599.png)

![1621902240837](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/082401-529264.png)

**第四步**

![1621902349552](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/082550-478169.png)

整个过程中，会生成多个物理计划，最终通过成本模型选择一个最优的物理计划执行。

**查看逻辑执行计划**

![1621902590011](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/082950-627561.png)

**查看物理执行计划**

![1621902657106](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/083057-114308.png)

## DataSet

sparksql支持：命令式，SQL

spark用于支持这两种方式的api叫做，dataset和dataframe

### 为什么需要Dataset

Spark在Spark 1.3版本中引入了Dataframe，DataFrame是组织到命名列中的分布式数据集合，但是有如下几点限制：

- 编译时类型不安全：
  - Dataframe API不支持编译时安全性，这限制了在结构不知道时操纵数据。
  - 以下示例在编译期间有效。但是，执行此代码时将出现运行时异常。

![1622028516170](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/192921-405749.png)

也就是说在编译时检查不出异常，但是在运行的时候会出现异常。

- 无法对域对象（丢失域对象）进行操作：
  - 将域对象转换为DataFrame后，无法从中重新生成它；
  - 下面的示例中，一旦我们从personRDD创建personDF，将不会恢复Person类的原始RDD（RDD [Person]）；

![1622028624393](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/193026-201016.png)

基于上述的两点，从Spark1.6开始出现Dataset，至Spark 2.0中将DataFrame与Dataset合并，其中DataFrame为Dataset特殊类型，类型为Row。

![1622028664021](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/193227-511782.png)

针对RDD、DataFrame与Dataset三者编程比较来说，Dataset API无论语法错误和分析错误在编译时都能发现。

![1622028715158](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/193228-117460.png)

此外RDD与Dataset相比较而言，由于Dataset数据使用特殊编码，所以在存储数据时更加节省内存。

### Dataset是什么

![1622028799877](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622028799877.png)

Spark 框架从最初的数据结构RDD、到SparkSQL中针对结构化数据封装的数据结构DataFrame，最终使用Dataset数据集进行封装，发展流程如下。

![1622028832085](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622028832085.png)

Dataset是Spark 1.6推出的最新的数据抽象，可以理解为是DataFrames的扩展，它提供了一种类型安全的，面向对象的编程接口。

从Spark 2.0开始，DataFrame与Dataset合并，每个Dataset也有一个被称为一个DataFrame的类型化视图，这种DataFrame是Row类型的Dataset，即Dataset[Row]。DataFrame = DataSet[Row]

Dataset结合了RDD和DataFrame的优点：

- 与RDD相比：Dataset保存了更多的描述信息，概念上等同于关系型数据库中的二维表；

- 与DataFrame相比：Dataset保存了类型信息，是强类型的，提供了编译时类型检查，调用Dataset的方法先会生成逻辑计划，然后被Spark的优化器进行优化，最终生成物理计划，然后提交到集群中运行；

所以在实际项目中建议使用Dataset进行数据封装，数据分析性能和数据存储更加好。

~~~ java
object Test03 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark = new sql.SparkSession.Builder()
		.master("local[6]")
		.appName("dataset")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._

		val datasource: RDD[Person] = spark.sparkContext.parallelize(Seq(Person("zhangsan", 15), Person("lisi", 25)))
	//	3 使用dataset
	//	把生成的数据转换为dataset
	val daData: Dataset[Person] = datasource.toDS()
		//dataset支持rdd类型的api
		daData.filter(item => item.age>10).show()
	//	dataste支持弱类型的api
		daData.filter('age> 10).show()
		daData.filter($"age">10).show()
	//	可以直接编写sql表达式
		daData.filter("age > 10").show()
	}
	case class Person(name:String,age: Int)
}
~~~

![1621904017335](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/085338-134659.png)

在dataset中，不仅有类型信息，而且还有结构信息。

### Dataset底层是什么

**查看执行计划**

~~~ java
object Test03 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark = new sql.SparkSession.Builder()
		.master("local[6]")
		.appName("dataset")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._

		val datasource: RDD[Person] = spark.sparkContext.parallelize(Seq(Person("zhangsan", 15), Person("lisi", 25)))
	//	3 使用dataset
	//	把生成的数据转换为dataset
	val daData: Dataset[Person] = datasource.toDS()
		//查看逻辑执行计划
		daData.queryExecution

		//查看物理执行计划和逻辑执行计划
		daData.explain(true)
	}
	case class Person(name:String,age: Int)
}

//执行计划
== Parsed Logical Plan ==//解析AST语法树
SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).name, true, false) AS name#3, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).age AS age#4]
+- ExternalRDD [obj#2]

== Analyzed Logical Plan ==//分析树。添加元数据信息
name: string, age: int
SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).name, true, false) AS name#3, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).age AS age#4]
+- ExternalRDD [obj#2]

== Optimized Logical Plan ==//优化语法树
SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).name, true, false) AS name#3, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).age AS age#4]
+- ExternalRDD [obj#2]

== Physical Plan ==//生成物理执行计划
*(1) SerializeFromObject [staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).name, true, false) AS name#3, knownnotnull(assertnotnull(input[0, hm.hmsql.Test03$Person, true])).age AS age#4]
+- Scan[obj#2]
~~~

![1621904658412](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/090419-79076.png)

也就是说dataset不管是否执行sql语句，都会被优化器进行优化。

**dataset底层数据结构**

![1621904992790](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/090953-126901.png)

**底层代码**

![1621905089396](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621905089396.png)

从底层代码来看，他对所有的数据类型都进行了一个包装，对外表现为不同的数据类型，但是对内使用统一的数据类型表示。

使用dataset.rdd可以直接把dataset转换为rdd，spark是一个非常弹性的工具，在一个程序中，既可以使用rdd，又可以使用rdd，还可以使用sql。

![1621905444940](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/091726-314564.png)

~~~  java
object Test04 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark = new sql.SparkSession.Builder()
		.master("local[6]")
		.appName("dataset")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._

		val datasource: RDD[Person] = spark.sparkContext.parallelize(Seq(Person("zhangsan", 15), Person("lisi", 25)))
	//	3 使用dataset
	//	第一种创建dataset的方式：把生成的数据转换为dataset
	val daData: Dataset[Person] = datasource.toDS()

	//	第二种方式创建dataset
	val dataDs: Dataset[Person] = spark.createDataset(Seq(Person("zhangsan", 15), Person("lisi", 25)))

		//直接获取已经分析和结果过的dataset的执行计划，从中拿到rdd
		val rdd1: RDD[InternalRow] = dataDs.queryExecution.toRdd
		//将dataset底层的RDD[InternalRow]，通过底层的DECODER转换成和dataset类型一致的RDD
		val rdd: RDD[Person] = dataDs.rdd

		//查看rdd的执行步骤
		println(rdd1.toDebugString)
		println()
		println(rdd.toDebugString)
	}
	case class Person(name:String,age: Int)
}
(2) SQLExecutionRDD[3] at toRdd at Test04.scala:29 []
 |  MapPartitionsRDD[2] at toRdd at Test04.scala:29 []
 |  ParallelCollectionRDD[1] at toRdd at Test04.scala:29 []

(2) MapPartitionsRDD[8] at rdd at Test04.scala:30 []
 |  SQLExecutionRDD[7] at rdd at Test04.scala:30 []
 |  MapPartitionsRDD[6] at rdd at Test04.scala:30 []
 |  MapPartitionsRDD[5] at rdd at Test04.scala:30 []
 |  ParallelCollectionRDD[4] at rdd at Test04.scala:30 []
~~~

- dataDs.queryExecution.toRdd:直接获取已经分析和解析过的dataset的执行计划，从中获取到rdd和其类型，必然是InternalRow类型的。
- dataDs.rdd：将dataset底层的RDD[InternalRow]，通过底层的DECODER转换成和dataset类型一致的RDD[person]，也就是转换为具体的类型。

## DataFrame

### DataFrame是什么

![1622027869159](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/191749-972388.png)

![1621906546993](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/093548-398551.png)

- 在Spark中，DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库中的二维表格。

- DataFrame与RDD的主要区别在于，前者带有schema元信息，即DataFrame所表示的二维表数据集的每一列都带有名称和类型。 

- 使得Spark SQL得以洞察更多的结构信息，从而对藏于DataFrame背后的数据源以及作用于DataFrame之上的变换进行针对性的优化，最终达到大幅提升运行时效率。反观RDD，由于无从得知所存数据元素的具体内部结构，Spark Core只能在stage层面进行简单、通用的流水线优化。

**案例**

~~~ java
Dataobject Test05 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._

	//	3 创建dataframe
	val df: DataFrame = Seq(Person("zhangsan", 15), Person("lisi", 25)).toDF()

	//	操作dataframe
	//	select name from table where age > 16
	//	sql语句支持的语法，dataframe也都支持
		df
			.where('age>16)
			.select("name")
			.show()


	}
	case class Person(name:String,age: Int)
}
~~~

DataFrame类似于关系型数据库中的一张表，在DataFrame上面的操作，非常类似于sql语句的操作，因为dataframe被划分为行和列，并且列中包含Schema信息，表示表的结构信息。

**数据清洗的步骤**

ETL：

- E：抽取
- T：转换和处理
- L：装载和落地

spark代码编写规则：

- 创建DataFrame，DataSet,RDD，读取数据
- 通过Df，DataSet,RDD，的API进行数据的处理
- 通过DF，DataSet,RDD，进行数据的落地

### 创建DataFrame

#### 通过隐式转换

~~~ java
object Test06 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
		val personList = Seq(Person("zhangsan", 15), ("lisi", 26))

	//	3 创建dataframe
	// 		1 通过隐式转换
		val df1: DataFrame = personList.toDF()
		val df2: DataFrame = spark.sparkContext.parallelize(personList).toDF()

	}
	case class Person(name:String,age: Int)
}
~~~

#### 通过集合创建

~~~ java
object Test06 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
		val personList = Seq(Person("zhangsan", 15), ("lisi", 26))

	//	3 创建dataframe
	//		2 通过createDataFrame()创建
	val df3: DataFrame = spark.createDataFrame(personList)
	}
	case class Person(name:String,age: Int)
}
~~~

#### 通过读取外部数据集

~~~ JAVA
object Test06 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
		val personList = Seq(Person("zhangsan", 15), ("lisi", 26))

	//	3 创建dataframe

				//3 通过read()方式
				val df4: DataFrame = spark.read.csv("")
	}
	case class Person(name:String,age: Int)
}
~~~

### 操作DataFrame

![1621909770100](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/102930-996365.png)

~~~ java
object Test07 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
	//	读取数据
	val df: DataFrame = spark.read
			.option("header",value = true)//告诉spark在读取文件的时候，把第一行当做头处理
			.csv("")

	//	处理
	//		1 选择列
	//		2 过滤掉NA的值
	//	 	3 分组处理
	//		4 做聚合处理

		df.select('year,'month,'PM_Dongsi)
			.where('PM_Dongsi =!= "NA")
			.groupBy('year,'month)
			.count()
			.show()

		//直接使用sql语句查询
		//将dataframe注册为临时表
		df.createOrReplaceTempView("pm")
		val res: DataFrame = spark.sql("select year,month,count(PM_Dongsi) from pm where PM_Dongsi != 'NA' group by year,month ")
		res.show()

		spark.stop()
	}
	case class Person(name:String,age: Int)
}
~~~

### DataSet和DataFrame的区别

![1621910227467](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/103708-493972.png)

从源码中可以看到DataFrame就是Dataset

~~~ java
package object sql {
  @DeveloperApi
  @Unstable
  type Strategy = SparkStrategy

  type DataFrame = Dataset[Row]

  private[sql] val SPARK_VERSION_METADATA_KEY = "org.apache.spark.version"
    
  private[sql] val SPARK_LEGACY_DATETIME = "org.apache.spark.legacyDateTime"
}
~~~

![1621911860684](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/100000-205409.png)

~~~ java
object Test08 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
		val personList = Seq(Person("zhangsan", 15), Person("lisi", 26))

		//无论DataFrame里面存储的是什么类型的数据。其底层都是Dataset[Row]类型，DataFrame是弱类型
		val df: DataFrame = personList.toDF()
		//但是对于Dataset来说。里面可以存储具体的类型对象，Dataset是强类型
		val ds: Dataset[Person] = personList.toDS()

	}
	case class Person(name:String,age: Int)
}
~~~

![1621911889359](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/100208-907963.png)

不管DataFrame里面存储的是什么，类型永远是Row对象，但是对于Dataset来说，可以具体存储类型。

![1621994912645](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/100833-689224.png)

DataFrame代表的弱类型在编译的时候是不安全的，Dataset是编译时类型安全的。

![1621995099988](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/101141-948512.png)

### Row对象

DataFrame中每条数据封装在Row中，Row表示每行数据

![1621995737001](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/102304-11076.png)

**row与DataFrame**

![1621995794558](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621995794558.png)

可以看到，DataFrame就是一个二维的表格，row就是一个行，Row有一个schema对象表示表结构，DataFrame就是由放置了Row的Dataset组成组成的二维表格。其实DataFrame就是DataSet。

~~~ java
object Test09 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//	2 导入隐式转换,这里的spark是创建出来的spark对象，而不是包
		import spark.implicits._
	//Row 是什么，如何创建
	val p = new Person("zhangsan", 25)
		//row对象必须配合schema对象才会有列名
		val row: Any = Row("lisi", 24)
	//	从row中获取数据是根据索引获取的
		row.getString(0) //根据索引修改值，就像数组一样
		row.getInt(1)
	//	row也是样例类
		row match{
			case Row(name,age)=>println(name,age)
		}

	}
	case class Person(name:String,age: Int)
}
~~~

> 如何理解DataFrame就是Dataset，我们可以从DataFrame和Dataset的结构看一下。

对于RDD中的数据：

![1621996339397](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621996339397.png)

对应到Dataset中就是这样存放的：

![1621996399553](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/103320-531066.png)

如果存放到DataFrame中，那么就是下面这样的。

![1621996372128](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/103329-417977.png)

很明显，我们可以看出，对于Dataset中，数据是按照对象的格式存储在每一行中，每一行都表示一个对象，表头的schema信息也是一个对象。我们说DataFrame是存储了类型为Row的Dataset，可以把Row想象为一个一位数组，那么对象的每一个属性信息就存储在数组的每一个单元中，就是相当于把数据更加细粒度的存储。

### RDD、DataFrame和Dataset

![1622028925177](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/193605-202091.png)

SparkSQL中常见面试题：如何理解Spark中三种数据结构RDD、DataFrame和Dataset关系？

**RDD**

- RDD（Resilient Distributed Datasets）叫做弹性分布式数据集，是Spark中最基本的数据抽象，源码中是一个抽象类，代表一个不可变、可分区、里面的元素可并行计算的集合。
- 编译时类型安全，但是无论是集群间的通信，还是IO操作都需要对对象的结构和数据进行序列化和反序列化，还存在较大的GC的性能开销，会频繁的创建和销毁对象。

**DataFrame**

- 与RDD类似，DataFrame是一个分布式数据容器，不过它更像数据库中的二维表格，除了数据之外，还记录这数据的结构信息（即schema）。

- DataFrame也是懒执行的，性能上要比RDD高（主要因为执行计划得到了优化）。

- 由于DataFrame每一行的数据结构一样，且存在schema中，Spark通过schema就能读懂数据，因此在通信和IO时只需要序列化和反序列化数据，而结构部分不用。

- Spark能够以二进制的形式序列化数据到JVM堆以外（off-heap：非堆）的内存，这些内存直接受操作系统管理，也就不再受JVM的限制和GC的困扰了。但是DataFrame不是类型安全的。

**Dataset**

- Dataset是Spark1.6中对DataFrame API的一个扩展，是Spark最新的数据抽象，结合了RDD和DataFrame的优点。

- DataFrame=Dataset[Row]（Row表示表结构信息的类型），DataFrame只知道字段，但是不知道字段类型，而Dataset是强类型的，不仅仅知道字段，而且知道字段类型。

- 样例类CaseClass被用来在Dataset中定义数据的结构信息，样例类中的每个属性名称直接对应到Dataset中的字段名称。

- Dataset具有类型安全检查，也具有DataFrame的查询优化特性，还支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。

### 小结

![1621910134132](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/25/103534-993976.png)

## SparkSql实战

### SparkSql初体验

SparkSession对象实例通过建造者模式构建，代码如下：

**其中**

①表示导入SparkSession所在的包，

②表示建造者模式构建对象和设置属性，

③表示导入SparkSession类中implicits对象object中隐式转换函数。

![1622029318667](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622029318667.png)

### 获取DataFrame/DataSet

 实际项目开发中，往往需要将RDD数据集转换为DataFrame，本质上就是给RDD加上Schema信息

#### 使用样例类

当RDD中数据类型CaseClass样例类时，底层可以通过反射Reflecttion获取属性名称和类型，构建Schema，应用到RDD数据集，将其转换为DataFrame。

~~~ java
object Test15 {

	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		//注意:在新版的Spark中,使用SparkSession来进行SparkSQL开发!
		//因为SparkSession封装了SqlContext、HiveContext、SparkContext
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		//2.获取RDD
		val fileRDD: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		val personRDD: RDD[Person] = fileRDD.map(line => {
			val arr: Array[String] = line.split(" ")
			Person(arr(0).toInt, arr(1), arr(2).toInt)
		})
		//3.RDD->DataFrame/DataSet
		import spark.implicits._ //隐式转换
		//将RDD转换为DataFrame，是弱类型的
		val df: DataFrame = personRDD.toDF()
		//将RDD转换为Dataset，是强类型的
		val ds: Dataset[Person] = personRDD.toDS()

		//4.输出约束和类型
		df.printSchema()
		df.show()

		ds.printSchema()
		ds.show()

		//5.关闭资源
		sc.stop()
		spark.stop()
	}
	case class Person(id:Int,name:String,age:Int)

}
~~~

此种方式要求RDD数据类型必须为CaseClass，转换的DataFrame中字段名称就是CaseClass中属性名称。

#### 指定类型+列名

SparkSQL中提供一个函数：toDF，通过指定列名称，将数据类型为元组的RDD或Seq转换为DataFrame，实际开发中也常常使用。

~~~ java
object Test16 {
	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		//注意:在新版的Spark中,使用SparkSession来进行SparkSQL开发!
		//因为SparkSession封装了SqlContext、HiveContext、SparkContext
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		//2.获取RDD
		val fileRDD: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		//tupleRDD: RDD[(Int, String, Int)]--指定类型:(Int, String, Int)
		//将读取的数据转换为三元组格式
		val tupleRDD: RDD[(Int, String, Int)] = fileRDD.map(line => {
			val arr: Array[String] = line.split(" ")
			(arr(0).toInt, arr(1), arr(2).toInt)
		})

		//3.RDD->DataFrame/DataSet
		import spark.implicits._ //隐式转换
		//指定列名
		val df: DataFrame = tupleRDD.toDF("id","name","age")
		//dataset没有这样的用法

		//4.输出约束和类型
		df.printSchema()
		df.show()

		//5.关闭资源
		sc.stop()
		spark.stop()
	}
}
~~~

#### 自定义Schema

依据RDD中数据自定义Schema，类型为StructType，每个字段的约束使用StructField定义，具体步骤如下：

1. RDD中数据类型为Row：**RDD[Row]**；

2. 针对Row中数据定义Schema：**StructType**；

3. 使用SparkSession中方法将定义的Schema应用到RDD[Row]上；

~~~ java
object Test17 {
	def main(args: Array[String]): Unit = {
			//1.准备SparkSQL开发环境
			//注意:在新版的Spark中,使用SparkSession来进行SparkSQL开发!
			//因为SparkSession封装了SqlContext、HiveContext、SparkContext
			val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
			val sc: SparkContext = spark.sparkContext
			sc.setLogLevel("WARN")

			//2.获取RDD
			val fileRDD: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
			//准备rowRDD:RDD[Row]
			val rowRDD: RDD[Row] = fileRDD.map(line => {
				val arr: Array[String] = line.split(" ")
				Row(arr(0).toInt, arr(1), arr(2).toInt)
			})

			//准备Schema
			/*val schema: StructType = StructType(
						StructField("id", IntegerType, true) ::
						StructField("name", StringType, true) ::
						StructField("age", IntegerType, true) :: Nil)*/
			val schema: StructType = StructType(
				List(
					StructField("id", IntegerType, true),
					StructField("name", StringType, true),
					StructField("age", IntegerType, true)
				)
			)

			//3.RDD->DataFrame/DataSet
			import spark.implicits._ //隐式转换
			val df: DataFrame = spark.createDataFrame(rowRDD, schema)


			//4.输出约束和类型
			df.printSchema()
			df.show()

			//5.关闭资源
			sc.stop()
			spark.stop()
		}
}
~~~

此种方式可以更加体会到DataFrame = RDD[Row] + Schema组成，在实际项目开发中灵活的选择方式将RDD转换为DataFrame。

### RDD、DF、DS相互转换

#### 转换API

实际项目开发中，常常需要对RDD、DataFrame及Dataset之间相互转换，其中要点就是Schema约束结构信息。

![1622030207520](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/195707-106963.png)

- RDD转换DataFrame或者Dataset
  - 转换DataFrame时，定义Schema信息，两种方式
  - 转换为Dataset时，不仅需要Schema信息，还需要RDD数据类型为CaseClass类型

- Dataset或DataFrame转换RDD
  - 由于Dataset或DataFrame底层就是RDD，所以直接调用rdd函数即可转换
  - dataframe.rdd 或者dataset.rdd

- DataFrame与Dataset之间转换
  - 由于DataFrame为Dataset特例，所以Dataset直接调用**toDF**函数转换为DataFrame
  - 当将DataFrame转换为Dataset时，使用函数**as[Type]**，指定CaseClass类型即可。

**RDD、DataFrame和DataSet之间的转换如下:**

~~~ java
RDD转换到DataFrame：rdd.toDF(“name”)
  
RDD转换到Dataset：rdd.map(x => Emp(x)).toDS

DataFrame转换到Dataset：df.as[Emp]

DataFrame转换到RDD：df.rdd

Dataset转换到DataFrame：ds.toDF

Dataset转换到RDD：ds.rdd
~~~

> 注意：
>
> RDD与DataFrame或者DataSet进行操作，都需要引入隐式转换import spark.implicits._，其中的spark是SparkSession对象的名称！

#### 代码演示

~~~ java
object Test18 {
	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		//2.获取RDD
		val fileRDD: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		val personRDD: RDD[Person] = fileRDD.map(line => {
			val arr: Array[String] = line.split(" ")
			Person(arr(0).toInt, arr(1), arr(2).toInt)
		})

		import spark.implicits._ //隐式转换

		//3.相互转换
		// RDD->DF
		val df: DataFrame = personRDD.toDF()
		// RDD->DS
		val ds: Dataset[Person] = personRDD.toDS()

		// DF->RDD
		val rdd: RDD[Row] = df.rdd //注意:rdd->df的时候泛型丢了,所以df->rdd的时候就不知道原来的泛型了,给了个默认的
		// DF->DS
		val ds2: Dataset[Person] = df.as[Person] //给df添加上泛型

		// DS->RDD
		val rdd2: RDD[Person] = ds.rdd
		// DS->DF
		val df2: DataFrame = ds.toDF()

		//4.输出约束和类型
		df.printSchema()
		df.show()

		ds.printSchema()
		ds.show()

		//5.关闭资源
		sc.stop()
		spark.stop()
	}
	case class Person(id:Int,name:String,age:Int)
}
~~~

### SparkSQL花式查询

在SparkSQL模块中，将结构化数据封装到DataFrame或Dataset集合中后，提供了两种方式分析处理数据：

1. SQL 编程，将DataFrame/Dataset注册为临时视图或表，编写SQL语句，类似HiveQL；
2. DSL（domain-specific language）编程，调用DataFrame/Dataset API（函数），类似RDD中函数；

#### 基于SQL查询

将Dataset/DataFrame注册为临时视图，编写SQL执行分析，分为两个步骤：

1. 注册为临时视图

![1622030824685](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622030824685.png)

2. 编写sql执行分析

![1622030855659](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622030855659.png)

其中SQL语句类似Hive中SQL语句，查看Hive官方文档，SQL查询分析语句语法，官方文档文档：

![1622030890353](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/200844-322655.png)

#### 基于DSL分析

调用DataFrame/Dataset中API（函数）分析数据，其中函数包含**RDD中转换函数**和类似**SQL语句函数**，部分截图如下：

![1622030944541](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622030944541.png)

类似SQL语法函数：调用Dataset中API进行数据分析，Dataset中涵盖很多函数，大致分类如下：

1. 选择函数**select**：选取某些列的值

![1622030990520](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622030990520.png)

2. 过滤函数**filter/where**：设置过滤条件，类似SQL中WHERE语句

![1622031022522](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031022522.png)

3. 分组函数**groupBy/rollup/cube**：对某些字段分组，在进行聚合统计

![1622031052837](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031052837.png)

4. 聚合函数**agg**：通常与分组函数连用，使用一些count、max、sum等聚合函数操作

![1622031086409](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/201128-332232.png)

5. 排序函数**sort/orderBy**：按照某写列的值进行排序（升序ASC或者降序DESC）

![1622031122930](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031122930.png)

6. 限制函数**limit**：获取前几条数据，类似RDD中take函数

![1622031161950](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031161950.png)

7. 重命名函数**withColumnRenamed**：将某列的名称重新命名

![1622031194110](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031194110.png)

8. 删除函数**drop**：删除某些列

![1622031221439](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031221439.png)

9. 增加列函数**withColumn**：当某列存在时替换值，不存在时添加此列

![1622031253979](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031253979.png)

上述函数在实际项目中经常使用，尤其数据分析处理的时候，其中要注意，调用函数时，通常指定某个列名称，传递Column对象，通过**隐式转换转换字符串String类型为Column对象**。

![1622031297792](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622031297792.png)

Dataset/DataFrame中转换函数，类似RDD中Transformation函数，使用差不多：

![1622031322685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/201716-305086.png)

#### 代码演示

~~~ java
object Test19 {

	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		//2.获取RDD
		val fileRDD: RDD[String] = sc.textFile("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		val personRDD: RDD[Person] = fileRDD.map(line => {
			val arr: Array[String] = line.split(" ")
			Person(arr(0).toInt, arr(1), arr(2).toInt)
		})
		//3.RDD->DataFrame
		import spark.implicits._ //隐式转换
		val df: DataFrame = personRDD.toDF()

		//4.输出约束和类型
		df.printSchema()
		df.show()

		//TODO =============花式查询============
		println("===========SQL风格========")
		//-1.注册表
		//df.registerTempTable("t_person")
		//df.createOrReplaceGlobalTempView("t_person")//创建一个全局的视图/表,所有SparkSession可用--生命周期太长
		df.createOrReplaceTempView("t_person") //创建一个临时视图/表,该SparkSession可用
		//-2.各种查询
		//=1.查看name字段的数据
		spark.sql("select name from t_person").show(false)
		//=2.查看 name 和age字段数据
		spark.sql("select name,age from t_person").show(false)
		//=3.查询所有的name和age，并将age+1
		spark.sql("select name,age,age+1 from t_person").show(false)
		//=4.过滤age大于等于25的
		spark.sql("select id,name,age from t_person where age >= 25").show(false)
		//=5.统计年龄大于30的人数
		spark.sql("select count(*) from t_person where age > 30").show(false)
		//=6.按年龄进行分组并统计相同年龄的人数
		spark.sql("select age,count(*) from t_person group by age").show(false)
		//=7.查询姓名=张三的
		val name = "zhangsan"
		spark.sql("select id,name,age from t_person where name='zhangsan'").show(false)
		spark.sql(s"select id,name,age from t_person where name='${name}'").show(false)

		println("===========DSL风格========")
		//=1.查看name字段的数据
		df.select(df.col("name")).show(false)
		import org.apache.spark.sql.functions._
		df.select(col("name")).show(false)
		df.select("name").show(false)

		//=2.查看 name 和age字段数据
		df.select("name", "age").show(false)

		//=3.查询所有的name和age，并将age+1
		//df.select("name","age","age+1").show(false)//报错:没有"age+1"这个列名
		//df.select("name","age","age"+1).show(false)//报错:没有"age+1"这个列名
		df.select($"name", $"age", $"age" + 1).show(false) //$"age"表示获取该列的值/$"列名"表示将该列名字符串转为列对象
		df.select('name, 'age, 'age + 1).show(false) //'列名表示将该列名字符串转为列对象

		//=4.过滤age大于等于25的
		df.filter("age >= 25").show(false)
		df.where("age >= 25").show(false)

		//=5.统计年龄大于30的人数
		val count: Long = df.filter("age > 30").count()
		println("年龄大于30的人数"+count)

		//=6.按年龄进行分组并统计相同年龄的人数
		df.groupBy("age").count().show(false)

		//=7.查询姓名=张三的
		df.filter("name ='zhangsan'").show(false)
		df.where("name ='zhangsan'").show(false)
		df.filter($"name" === "zhangsan").show(false)
		df.filter('name === "zhangsan").show(false)
		//=8.查询姓名!=张三的
		df.filter($"name" =!= name).show(false)
		df.filter('name =!= "zhangsan").show(false)


		//TODO =============花式查询============

		//5.关闭资源
		sc.stop()
		spark.stop()
	}

	case class Person(id: Int, name: String, age: Int)
}
~~~

## SparkSql扩展

在SparkSQL模块，提供一套完成API接口，用于方便读写外部数据源的的数据（从Spark 1.4版本提供），框架本身内置外部数据源：

![1622032473209](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/203435-344568.png)

### 数据源与格式

数据分析处理中，数据可以分为结构化数据、非结构化数据及半结构化数据。

![1622032548537](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622032548537.png)

**结构化数据（Structured）**

- 结构化数据源可提供有效的存储和性能。例如，Parquet和ORC等柱状格式使从列的子集中提取值变得更加容易。

- 基于行的存储格式（如Avro）可有效地序列化和存储提供存储优势的数据。然而，这些优点通常以灵活性为代价。如因结构的固定性，格式转变可能相对困难。

**非结构化数据（UnStructured）**

- 相比之下，非结构化数据源通常是自由格式文本或二进制对象，其不包含标记或元数据以定义数据的结构。

- 报纸文章，医疗记录，图像，应用程序日志通常被视为非结构化数据。这些类型的源通常要求数据周围的上下文是可解析的。

**半结构化数据（Semi-Structured）**

- 半结构化数据源是按记录构建的，但不一定具有跨越所有记录的明确定义的全局模式。每个数据记录都使用其结构信息进行扩充。 

- 半结构化数据格式的好处是，它们在表达数据时提供了最大的灵活性，因为每条记录都是自我描述的。但这些格式的主要缺点是它们会产生额外的解析开销，并且不是特别为ad-hoc(特定)查询而构建的。

#### DataFrameReader

![1622005563689](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/130712-436516.png)

读取文件，至少设置以下内容：

- 文件地址
- 文件类型(format)
- 读取数据的参数(option)
- 结构信息（schema）

![1622005680164](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/130800-761526.png)

#### text数据读取

SparkSession加载文本文件数据，提供两种方法，返回值分别为DataFrame和Dataset

~~~ java
object Test20 {

	def main(args: Array[String]): Unit = {
		//1.准备SparkSQL开发环境
		val spark: SparkSession = SparkSession.builder().appName("hello").master("local[*]").getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")

		val frame: DataFrame = spark.read.text("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
		spark.read
				.format("text")
			.load("D:\\soft\\idea\\work\\work04\\src\\main\\resources\\person")
				.show(3)

		//frame.show(3)

		//5.关闭资源
		sc.stop()
		spark.stop()
	}
}
~~~

可以查看一下底层源码

~~~ java
//text方法返回的是DataFrame
def text(path: String): DataFrame = {
    // This method ensures that calls that explicit need single argument works, see SPARK-16009
    text(Seq(path): _*)
  }

~~~

![1622033516232](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622033516232.png)

可以看出textFile方法底层还是调用text方法，先加载数据封装到DataFrame中，再使用as[String]方法将DataFrame转换为Dataset，实际中推荐使用textFile方法，从Spark 2.0开始提供。

无论是text方法还是textFile方法读取文本数据时，一行一行的加载数据，每行数据使用UTF-8编码的字符串，列名称为【value】。

#### Json数据读取

实际项目中，有时处理数据以JSON格式存储的，尤其后续结构化流式模块：StructuredStreaming，从Kafka Topic消费数据很多时间是JSON个数据，封装到DataFrame中，需要解析提取字段的值。以读取github操作日志JSON数据为例，数据结构如下：

![1622032925471](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/204715-700777.png)

使用**textFile**加载数据，对每条JSON格式字符串数据，使用SparkSQL函数库functions中自带**get_json_obejct**函数提取字段：id、type、public和created_at的值。

**函数：get_json_obejct使用说明**

![1622033604262](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622033604262.png)

~~~ java
object Test21 {

	def main(args: Array[String]): Unit = {
		val spark = SparkSession.builder()
			.appName(this.getClass.getSimpleName.stripSuffix("$"))
			.master("local[*]")
			// 通过装饰模式获取实例对象，此种方式为线程安全的
			.getOrCreate()
		val sc: SparkContext = spark.sparkContext
		sc.setLogLevel("WARN")
		import spark.implicits._

		// TODO: 从LocalFS上读取json格式数据(压缩）
		val jsonDF: DataFrame = spark.read.json("data/input/2015-03-01-11.json.gz")
		//jsonDF.printSchema()
		jsonDF.show(5, truncate = true)

		println("===================================================")
		val githubDS: Dataset[String] = spark.read.textFile("data/input/2015-03-01-11.json.gz")
		//githubDS.printSchema() // value 字段名称，类型就是String
		githubDS.show(5,truncate = true)

		// TODO：使用SparkSQL自带函数，针对JSON格式数据解析的函数
		import org.apache.spark.sql.functions._
		// 获取如下四个字段的值：id、type、public和created_at
		val gitDF: DataFrame = githubDS.select(
			get_json_object($"value", "$.id").as("id"),
			get_json_object($"value", "$.type").as("type"),
			get_json_object($"value", "$.public").as("public"),
			get_json_object($"value", "$.created_at").as("created_at")
		)
		gitDF.printSchema()
		gitDF.show(10, truncate = false)

		// 应用结束，关闭资源
		spark.stop()
	}
}
~~~

运行结果展示

![1622033732195](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622033732195.png)

#### CSV格式

在机器学习中，常常使用的数据存储在csv/tsv文件格式中，所以SparkSQL中也支持直接读取格式数据，从2.0版本开始内置数据源。关于CSV/TSV格式数据说明：

![1622033775839](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622033775839.png)

SparkSQL中读取CSV格式数据，可以设置一些选项，重点选项：

**分隔符：sep** 

默认值为逗号，必须单个字符

**数据文件首行是否是列名称：header** 

默认值为false，如果数据文件首行是列名称，设置为true

**是否自动推断每个列的数据类型：inferSchema** 

默认值为false，可以设置为true

当读取CSV/TSV格式数据文件首行是否是列名称，读取数据方式（参数设置）不一样的

- 行是列的名称，如下方式读取数据文件

~~~ java
  // TODO: 读取TSV格式数据
        val ratingsDF: DataFrame = spark.read
            // 设置每行数据各个字段之间的分隔符， 默认值为 逗号
            .option("sep", "\t")
            // 设置数据文件首行为列名称，默认值为 false
            .option("header", "true")
            // 自动推荐数据类型，默认值为false
            .option("inferSchema", "true")
            // 指定文件的路径
            .csv("datas/ml-100k/u.dat")
        
        ratingsDF.printSchema()
        ratingsDF.show(10, truncate = false)
~~~

- 首行不是列的名称，如下方式读取数据（设置Schema信息）

~~~ java
  // 定义Schema信息
        val schema = StructType(
            StructField("user_id", IntegerType, nullable = true) ::
                StructField("movie_id", IntegerType, nullable = true) ::
                StructField("rating", DoubleType, nullable = true) ::
                StructField("timestamp", StringType, nullable = true) :: Nil
        )
        
        // TODO: 读取TSV格式数据
        val mlRatingsDF: DataFrame = spark.read
            // 设置每行数据各个字段之间的分隔符， 默认值为 逗号
            .option("sep", "\t")
            // 指定Schema信息
            .schema(schema)
            // 指定文件的路径
            .csv("datas/ml-100k/u.data")
        
        mlRatingsDF.printSchema()
        mlRatingsDF.show(5, truncate = false)
~~~

- 将DataFrame数据保存至CSV格式文件，演示代码如下：

~~~ java
 						mlRatingsDF
            // 降低分区数，此处设置为1，将所有数据保存到一个文件中
            .coalesce(1)
            .write
            // 设置保存模式，依据实际业务场景选择，此处为覆写
            .mode(SaveMode.Overwrite)
            .option("sep", ",")
            // TODO: 建议设置首行为列名
            .option("header", "true")
            .csv("datas/ml-csv-" + System.nanoTime())
~~~

**案例**

~~~ java
object Test22 {
	
		def main(args: Array[String]): Unit = {
			val spark = SparkSession.builder()
				.appName(this.getClass.getSimpleName.stripSuffix("$"))
				.master("local[*]")
				// 通过装饰模式获取实例对象，此种方式为线程安全的
				.getOrCreate()
			val sc: SparkContext = spark.sparkContext
			sc.setLogLevel("WARN")
			import spark.implicits._

			/**
			 * 实际企业数据分析中
			 * csv\tsv格式数据，每个文件的第一行（head, 首行），字段的名称（列名）
			 */
			// TODO: 读取CSV格式数据
			val ratingsDF: DataFrame = spark.read
				// 设置每行数据各个字段之间的分隔符， 默认值为 逗号
				.option("sep", "\t")
				// 设置数据文件首行为列名称，默认值为 false
				.option("header", "true")
				// 自动推荐数据类型，默认值为false
				.option("inferSchema", "true")
				// 指定文件的路径
				.csv("data/input/rating_100k_with_head.data")

			ratingsDF.printSchema()
			ratingsDF.show(10, truncate = false)

			println("=======================================================")
			// 定义Schema信息
			val schema = StructType(
				StructField("user_id", IntegerType, nullable = true) ::
					StructField("movie_id", IntegerType, nullable = true) ::
					StructField("rating", DoubleType, nullable = true) ::
					StructField("timestamp", StringType, nullable = true) :: Nil
			)

			// TODO: 读取CSV格式数据
			val mlRatingsDF: DataFrame = spark.read
				// 设置每行数据各个字段之间的分隔符， 默认值为 逗号
				.option("sep", "\t")
				// 指定Schema信息
				.schema(schema)
				// 指定文件的路径
				.csv("data/input/rating_100k.data")

			mlRatingsDF.printSchema()
			mlRatingsDF.show(10, truncate = false)

			println("=======================================================")
			/**
			 * 将电影评分数据保存为CSV格式数据
			 */
			mlRatingsDF
				// 降低分区数，此处设置为1，将所有数据保存到一个文件中
				.coalesce(1)
				.write
				// 设置保存模式，依据实际业务场景选择，此处为覆写
				.mode(SaveMode.Overwrite)
				.option("sep", ",")
				// TODO: 建议设置首行为列名
				.option("header", "true")
				.csv("data/output/ml-csv-" + System.currentTimeMillis())

			// 关闭资源
			spark.stop()
		}
}
~~~

#### Parquet数据

SparkSQL模块中默认读取数据文件格式就是parquet列式存储数据，通过参数【spark.sql.sources.default】设置，默认值为【parquet】。

~~~ java
object SparkSQLParquet {
    def main(args: Array[String]): Unit = {
        val spark = SparkSession.builder()
          .appName(this.getClass.getSimpleName.stripSuffix("$"))
          .master("local[*]")
          // 通过装饰模式获取实例对象，此种方式为线程安全的
          .getOrCreate()
        val sc: SparkContext = spark.sparkContext
        sc.setLogLevel("WARN")
        import spark.implicits._

        // TODO: 从LocalFS上读取parquet格式数据
        val usersDF: DataFrame = spark.read.parquet("data/input/users.parquet")
        usersDF.printSchema()
        usersDF.show(10, truncate = false)

        println("==================================================")

        // SparkSQL默认读取文件格式为parquet
        val df = spark.read.load("data/input/users.parquet")
        df.printSchema()
        df.show(10, truncate = false)

        // 应用结束，关闭资源
        spark.stop()
    }
}
~~~

**结果**

![1622034171994](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622034171994.png)

#### jdbc数据

回顾在SparkCore中读取MySQL表的数据通过JdbcRDD来读取的，在SparkSQL模块中提供对应接口，提供三种方式读取数据：

- 单分区模式

![1622034270194](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622034270194.png)

- **多分区模式**，可以设置列的名称，作为分区字段及列的值范围和分区数目

![1622034306063](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622034306063.png)

- **高度自由分区模式**，通过设置条件语句设置分区数据及各个分区数据范围

![1622034329911](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622034329911.png)

当加载读取RDBMS表的数据量不大时，可以直接使用单分区模式加载；当数据量很多时，考虑使用多分区及自由分区方式加载。

从RDBMS表中读取数据，需要设置连接数据库相关信息，基本属性选项如下：

![1622034358412](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622034358412.png)

**代码演示**

~~~ java
// 连接数据库三要素信息
        val url: String = "jdbc:mysql://node1.itcast.cn:3306/?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true"
        val table: String = "db_shop.so"
        // 存储用户和密码等属性
        val props: Properties = new Properties()
        props.put("driver", "com.mysql.cj.jdbc.Driver")
        props.put("user", "root")
        props.put("password", "123456")
        
        // TODO: 从MySQL数据库表：销售订单表 so
        // def jdbc(url: String, table: String, properties: Properties): DataFrame
        val sosDF: DataFrame = spark.read.jdbc(url, table, props)
        println(s"Count = ${sosDF.count()}")
        sosDF.printSchema()
        sosDF.show(10, truncate = false)

~~~

可以使用option方法设置连接数据库信息，而不使用Properties传递，代码如下：

~~~ java
// TODO： 使用option设置参数
        val dataframe: DataFrame = spark.read
            .format("jdbc")
            .option("driver", "com.mysql.cj.jdbc.Driver")
            .option("url", "jdbc:mysql://node1.itcast.cn:3306/?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true")
            .option("user", "root")
            .option("password", "123456")
            .option("dbtable", "db_shop.so")
            .load()
        dataframe.show(5, truncate = false)

~~~



#### 数据读取框架案例

~~~ java
object Test10 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//第一种读取形式
		spark.read
				.format("csv")//设置文件的格式
			.option("header",true)//设置表头信息
			.option("inferScheme",value = true)//设置自动推断数据类型，也就是说schema是来自csv文件当中的
			.load("")//读取数据的路径
			.show(10)//查看10条数据


	//	第二种读取形式
		spark.read
			.option("header",true)//设置表头信息
			.option("inferScheme",value = true)//设置自动推断数据类型，也就是说schema是来自csv文件当中的
			.csv("")
			.show(10)
	}
}
~~~

但是这两种方法本质上是一致的，因为类似csv这样的方式只是load()方法的封装。可以从下面的源码看出来：

~~~ java
def csv(paths: String*): DataFrame = format("csv").load(paths : _*)
~~~

> 如果使用load()方式加载数据，但是没有指定format的话，默认是按照Parquet的格式读取文件。
>
> 也就是说sparksql默认读取文件的格式是Parquet格式。

![1622013939274](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152539-357155.png)

### DataFrameWriter

![1622013978207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152627-708490.png)

![1622014095287](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152815-241737.png)

**写入模式**

![1622014153714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/152914-689673.png)

![1622014215443](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/153016-958555.png)

**案例**

~~~ java
object Test11 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		//读取一个文件
		val df: DataFrame = spark.read
			.format("csv") //设置文件的格式
			.option("header", true) //设置表头信息
			.load("")//读取数据的路径


	//	写入文件
	val json: Unit = df.write
		.json("")

	//	第二种写入方式
		df.write
			.format("json")
			.save("")
	}
}
~~~

### Parquet文件格式

#### Parquet文件

![1622017248166](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/162050-259946.png)

**文件的读取**

![1622020881136](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/172122-190055.png)

~~~ java
object Test12 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")

	//	数据存储为parquet格式
		df.write//spark默认写入文件的格式是parquet格式
				.mode(SaveMode.Overwrite)//指定写入模式为重写
			.format("parquet").save("")


	//	读取parquet格式的文件
	//	默认的读取格式也是parquet格式文件
	//	可以读取文件夹
		spark.read
			.format("json")
			.load("")

	}
}
~~~

spark默认读取和写入的都是parquet格式的文件。

在写入的时候，会写入文件夹内，因为spark是支持分区操作单位，每一个分区会写入一个文件夹内。

读取数据的时候也是按照分区进行读取的，逐个文件夹进行遍历读取操作。

####  写入文件指定分区

为什么要进行分区？

![1622020819885](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/172021-707124.png)

![1622018506792](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622018506792.png)

表分区不仅在parquet格式的文件上面有，在其他格式的文件也有分区。

~~~ java
object Test13 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")


	//	写文件，表分区
	//	写分区表的时候，分区列不会包含在生成的文件中，仅仅显示在文件夹上面
	//	读取的时候，自然也读取不到分区信息
		df.write
			//这里是按照列进行分区操作
			.partitionBy("year","month")//按照年和月进行分区
			.save("")

	//	读取文件，自行发现分区
	//	如果读取的时候没有指定具体的分区，而是指定分区的文件夹，那么会自动的发现分区，这个时候
	//	分区列也会输出在文件中
		spark.read
			.parquet("")//这里需要指定具体分区的路径
			.printSchema()//打印表头信息
	}
}
~~~

### JSON格式文件

![1622021108833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/173403-591287.png)

![1622021698139](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622021698139.png)

![1622021725453](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/173527-130371.png)

~~~ java
object Test14 {

	def main(args: Array[String]): Unit = {

	//	1 创建sparksession
	val spark: SparkSession = SparkSession.builder()
		.appName("dataframe")
		.master("local[6]")
		.getOrCreate()//创建sparksession的对象

		val df: DataFrame = spark.read
			.option("header", true)
			.csv("")

	//	写入外部的时候，以json方式写入
	//但是写入的不是标准格式的json文件
		df.write
			.json("")
	}
}
~~~

**处理json文件格式的小技巧**

- toJSON()：可以将一个DataFrame（里面存储的是对象）转换为JSON格式的DataFrame（也就是json字符串）
  - toJSON()应用场景：处理完数据之后，DataFrame中如果是一个对象，如果其他的系统只支持json格式的数据，sparksql和这种系统进行整合的时候，那么就需要这种形式的转换。

~~~ java
df.toJSON.show()
~~~

- 可以直接从RDD读取JSON格式的DataFrame

~~~ java
//将json格式的数据转换为一个RDD
		val rdd: RDD[String] = df.toJSON.rdd
		//从rdd中读取出一个dataframe
		val frame: DataFrame = spark.read.json(rdd)
~~~

![1622022486043](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/26/174806-270779.png)

## SpaqkSql整合Hive

### 概述

Spark SQL模块从发展来说，从Apache Hive框架而来，发展历程：Hive（MapReduce）-> Shark (Hive on Spark) -> Spark SQL（SchemaRDD -> DataFrame -> Dataset)，

SparkSQL天然无缝集成Hive，可以加载Hive表数据进行分析。

**HiveOnSpark和SparkOnHive**

- HiveOnSpark：SparkSql诞生之前的Shark项目使用的，是把Hive的执行引擎换成Spark,剩下的使用Hive的，严重依赖Hive，早就淘汰了没有人用了
- SparkOnHive：SparkSQL诞生之后，Spark提出的，是仅仅使用Hive的元数据(库/表/字段/位置等信息...)，剩下的用SparkSQL的，如:执行引擎,语法解析,物理执行计划,SQL优化

![1622802574038](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622802574038.png)



### spark-sql中集成Hive

和文件的格式不同，Hive是一个外部的数据存储和查询引擎，所以说如果Spark需要访问Hive的话，就需要先整合HIve。

SparkSQL集成Hive本质就是：SparkSQL读取Hive的元数据MetaStore

![1622771034553](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/094356-602144.png)

![1622771197591](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/094639-706048.png)

![1622771312228](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/094834-882647.png)

**配置SparksQL到HIve**

拷贝hive-site.xml 到spark的conf目录下面。

**拷贝hdfs的配置文件到spark。**

~~~ java
[rzf@hadoop100 hadoop]$ cp core-site.xml /opt/module/spark/conf/
[rzf@hadoop100 hadoop]$ cp hdfs-site.xml /opt/module/spark/conf/
~~~

1. 启动Hive的元数据库服务

hive所在机器hadoop100上启动

~~~ java
nohup /export/server/hive/bin/hive --service metastore &
~~~

注意:Spark3.0需要Hive2.3.7版本

2. 告诉SparkSQL:Hive的元数据库在哪里

哪一台机器需要使用spark-sql命令行整合，hive就把下面的配置放在哪一台

也可以将hive-site.xml分发到集群中所有Spark的conf目录，此时任意机器启动应用都可以访问Hive表数据。

~~~ java
<configuration>
        <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://hadoop100:3306/metastore?createDatabaseIfNotExist=true</value>
          <description>JDBC connect string for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>com.mysql.jdbc.Driver</value>
          <description>Driver class name for a JDBC metastore</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
          <description>username to use against metastore database</description>
        </property>

        <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>root</value>
          <description>password to use against metastore database</description>
        </property>

<property>
<name>hive.metastore.warehouse.dir</name>
<value>/user/hive/warehouse</value>
<description>location of default database for the warehouse</description>
</property>


<property>
<name>hive.cli.print.header</name>
        <value>true</value>
</property>

<property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
</property>

<property>
  <name>hive.metastore.schema.verification</name>
  <value>false</value>
  <description>
      Enforce metastore schema version consistency.
      True: Verify that version information stored in is compatible with one from Hive jars.  Also disable automatic
            schema migration attempt. Users are required to manually migrate schema after Hive upgrade which ensures
            proper metastore schema migration. (Default)
      False: Warn if the version information stored in metastore doesn't match with one from in Hive jars.
  </description>



<property>
    <name>datanucleus.readOnlyDatastore</name>
    <value>false</value>
</property>
<property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
</property>
<property>
    <name>datanucleus.autoCreateSchema</name>
    <value>true</value>
</property>
<property>
    <name>datanucleus.autoCreateTables</name>
    <value>true</value>
</property>
<property>
    <name>datanucleus.autoCreateColumns</name>
    <value>true</value>
</property>

</property>

<property>
      <name>hive.metastore.local</name>
      <value>false</value>
</property>


"hive-site.xml" 94L, 2379C 已写入                                                                                 
Enter password: 

  
    <name>datanucleus.readOnlyDatastore</name>
    <value>false</value>
</property>
<property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
</property>
<property>
    <name>datanucleus.autoCreateSchema</name>
    <value>true</value>
</property>
<property>
    <name>datanucleus.autoCreateTables</name>
    <value>true</value>
</property>
<property>
    <name>datanucleus.autoCreateColumns</name>
    <value>true</value>
</property>

</property>


<property>
      <name>hive.metastore.local</name>
      <value>false</value>
</property>
//配置matastore占用哪一个ip地址
<property>
      <name>hive.metastore.uris</name>
      <value>thrift://hadoop100:9083</value>
 </property>
 
//开启metaStore进程
[rzf@hadoop100 hive]$ nohup /opt/module/hive/bin/hive --service metastore 2>&1 >> /opt/module/hive/logs/log.log &
~~~

### SparkSql访问Hive表

#### 使用Hive创建表

![1622773691938](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/102814-889761.png)

**创建表**

~~~ java
create external table stu 
(name STRING,
age INT,
sex STRING)
row format delimited fields terminated by '\t' 
lines terminated by '\n' 
STORED as TEXTFILE;

//加载数据
LOAD DATA INPATH '/dataset/stud' overwrite into table stu;
~~~

#### 访问表

![1622775303290](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/113822-397332.png) 

**案例**

~~~ java
spark.sql("use table")
val res=spark.sql("select * from student limit 10")
res.show()
~~~

#### 使用SparkSql创建表

![1622783645369](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/131418-87466.png)

#### SparkSql访问Hive

编写程序打成jar包放到集群中运行程序。

**添加依赖**

![1622783876771](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/131758-476827.png)

**配置SparkSession**

![1622784294441](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/132455-927890.png)

**独立的spark程序**

~~~ java
object HiveAccess {

	def main(args: Array[String]): Unit = {

		val spark: SparkSession = SparkSession.builder()
			.appName("hive exam")
			.config("spqrk.sql.warehouse.dir", "hdfs://hadoop100:9000/dataset/hive") //指定warehouse位置
			.config("hive.metastore.uirs", "thrift://hadoop100:9083") //指定metastore的位置
			.enableHiveSupport() //默认使用的是sparksql，需要但是这里使用的是hive，所以要支持hiveContext
			.getOrCreate()
	//	因为是放在集群中执行，所以不需要指定master

	//	编些逻辑代码
	//	1 读取数据，因为是要在集群中执行，没有办法保证程序在哪一台机器上执行，所以需要把程序上传到所有的机器上面，才可以读取本地文件
	//	把文件上传到hdfs上面，这样所有的机器都可以读取到数据，他是一个外部系统
	//	2 使用df读取数据

		import spark.implicits._

		//定义schema类型，类型是StructType
		val schema: StructType = StructType(
			//列表里面定义的是列的类型信息
			List(
				StructField("name",StringType),
				StructField("age",IntegerType),
				StructField("sex",StringType)
			)
		)

		val df: DataFrame = spark.read
			.option("delimiter", "\t") //数据的分隔符
			.schema(schema)
			.csv("hdfs///dataset/stud")


	//	对df进行查询
	val resdf: Dataset[Row] = df.where('age > 23)

	//	写入数据，
		resdf.write.mode(SaveMode.Overwrite).saveAsTable("spark03")
	}
}
~~~

### Hive的SQL交互方式

- 方式1：交互式命令行（CLI）
  - bin/hive，编写SQL语句及DDL语句

- 方式2：启动服务HiveServer2（Hive ThriftServer2)
  - 将Hive当做一个服务启动(类似MySQL数据库，启动一个服务)，端口为10000
  - 交互式命令行，bin/beeline，CDH 版本HIVE建议使用此种方式，CLI方式过时
  - JDBC/ODBC方式，类似MySQL中JDBC/ODBC方式

### SparkSQL交互方式

SparkSQL模块从Hive框架衍生发展而来，所以Hive提供的所有功能（数据分析交互式方式）都支持

- 方式1: 上一章节已经学习了
  - SparkSQL命令行或SparkSQL代码中访问
- 启动sparkSQL的thriftserver使用beeline或使用JDBC协议访问

**补充 ：sparksql的thriftserver**

Spark Thrift Server将Spark Applicaiton当做一个服务运行，提供Beeline客户端和JDBC方式访问，与Hive中HiveServer2服务一样的。

在实际大数据分析项目中，使用SparkSQL时，往往启动一个ThriftServer服务，分配较多资源（Executor数目和内存、CPU），不同的用户启动beeline客户端连接，编写SQL语句分析数据。

![1622804021490](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/185344-153486.png)

在$SPARK_HOME目录下的**sbin**目录，有相关的服务启动命令:

##### 使用beeline 客户端连接

1. 启动SaprkSQL的thriftserver--类似与Hive的HiveServer2

~~~ java
//在hadoop100节点上启动下面的服务
/export/server/spark/sbin/start-thriftserver.sh \
--hiveconf hive.server2.thrift.port=10000 \
--hiveconf hive.server2.thrift.bind.host=hadoop100 \
--master local[2]
~~~

2. 停止命令

~~~ java
/export/server/spark/sbin/stop-thriftserver.sh
~~~

3. 使用SparkSQL的beeline客户端命令行连接ThriftServer

~~~ java
/export/server/spark/bin/beeline
!connect jdbc:hive2://hadoop100:10000
root
123456
~~~

3. 查看web ui界面

`http://hadoop100:4040/jobs/`

## SparkSql读写jdbc

###  创建库和表

在mysql数据库上创建如下数据库和表。

![1622786743600](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/140545-670247.png)

### 向mysql中写入数据

![1622787128726](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622787128726.png)

![1622787145589](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/141227-710951.png)

**添加mysql的驱动程序**

~~~ java
//要正确读取数据，还需要添加mysql数据库的驱动程序 
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql.connector-java</artifactId>
   <version>5.1.47</version>
</dependency>
~~~

**写入数据**

~~~ java
object MysqlWrite {

	/**
	 * mysql的访问方式有两种，使用本地运行，提交到集群运行
	 * 在写入mysql数据的时候，使用本地运行的形式，读取的时候，使用集群的形式
	 */
	def main(args: Array[String]): Unit = {

		val spark: SparkSession = SparkSession.builder()
			.appName("mysql exam")
			.master("local[6]")
			.getOrCreate()



		import spark.implicits._

		//定义schema类型，类型是StructType
		val schema: StructType = StructType(
			//列表里面定义的是列的类型信息
			List(
				StructField("name",StringType),
				StructField("age",IntegerType),
				StructField("sex",StringType)
			)
		)

		//读取数据，创建df
		val df: DataFrame = spark.read
			.option("delimiter", "\t") //数据的分隔符
			.schema(schema)
			.csv("")

	//	处理数据
	val resdf: Dataset[Row] = df.where('age < 25)

	//	落地数据
		resdf.write
			.format("jdbc")
			.option("url","jdbc:mysql://hadoop100:3306/spark02")
			.option("dbtable","stu")//指定数据库
			.option("user","spark03")//指定用户名
			.option("password","root")//指定密码
			.mode(SaveMode.Overwrite)
			.save()

	}
}
~~~

## 数据操作

### 有类型的转换算子

#### map&flatMap&mapPartitions

![1622790753632](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/151235-928834.png)

**案例**

~~~ java
object Test23 {

		def main(args: Array[String]): Unit = {
			val spark = SparkSession.builder()
				.master("local[6]")
				.appName("typed")
				.getOrCreate()

			import spark.implicits._

			val ds: Dataset[String] = Seq("hello spark", "hello flink").toDS()

			//1 flatMap算子操作
			ds.flatMap(item => item.split(" ")).show()

			//2 map算子操作
			val ds1: Dataset[Person] = Seq(Person(1, "xiaomign", 24), Person(2, "xiaohong", 25)).toDS()
			// 使用map算子做一个转换操作
			ds1.map(person => Person(person.id,person.name,person.age*2)).show()

			//3 mapPartitions
			//mapPartitions算子作用于每一个分区，而map算子是作用于每一条数据，可以提高执行的效率
			//如果一个分区的数据可以放进内存中，才可以进行mapPartitions操作，否则是不能进行mapPartitions操作的

			ds1.mapPartitions(
				//接受的是一个集合，但是这个集合不能超过内存的大小，否则就会发生内存溢出
				//对每一个集合中的元素进行转换，然后生成一个新的集合
				iter=>{
					//这个map算子是scala中的算子
					val res: Iterator[Person] = iter.map(person => Person(person.id, person.name, person.age * 2))
					res
				}
			).show()

			// 关闭资源
			spark.stop()
		}
	case class Person(id:Int,name:String,age:Int)
}
~~~

#### transform

![1622791810174](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/153530-702024.png)

**案例**

~~~ java
object Test23 {

		def main(args: Array[String]): Unit = {
			val spark = SparkSession.builder()
				.master("local[6]")
				.appName("typed")
				.getOrCreate()

			import spark.implicits._

			val data: Dataset[lang.Long] = spark.range(10)

			//doubled是定义新的列的列明
			data.transform(dataset => dataset.withColumn("doubled",'id*2)).show()

			// 关闭资源
			spark.stop()
		}
}
~~~

#### as

![1622792282528](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/153805-12164.png)

**案例**

~~~ java
object Test24 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		//定义schema类型，类型是StructType
		val schema: StructType = StructType(
			//列表里面定义的是列的类型信息
			List(
				StructField("name",StringType),
				StructField("age",IntegerType),
				StructField("sex",StringType)
			)
		)

		//把类型给为Dataset，里面存放的是Row类型
		//val df: Dataset[Row] = spark.read
		val df: DataFrame= spark.read
			.option("delimiter", "\t") //数据的分隔符
			.schema(schema)
			.csv("")

		/**
		 * 下面的转换，本质上进行的是：DataSet[Row].as[Student]==>DataSet[Student]的转换
		 */

		//转换,把df通过as 转换为另一种类型
		val ds: Dataset[Student] = df.as[Student]
		
		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### Filter

![1622793187628](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/155308-810346.png)

对数据进行过滤操作

~~~ java
object Test25 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15)).toDS()
		
		ds.filter(person => person.age>20).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### groupByKey

![1622793222112](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/04/155345-470658.png)

**案例**

~~~ java
object Test26 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		//算子里面的函数返回key信息，
		val grouped: KeyValueGroupedDataset[String, Student] = ds.groupByKey(person => person.name)

		//通过count()方法，返回的是Dataset类型,这里通过grouped的聚合方法，将其转换为DataSet类型
		val res: Dataset[(String, Long)] = grouped.count()

		res.show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### randomSplit

如何将一个数据集切分为多个数据集，如何从一个数据集中抽出一个比较小的数据集

**案例**

~~~ java
object Test27 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[lang.Long] = spark.range(15)

		//randomSplit 切多少分，权重是多少
		//切分为3分，因为数组中传入了3个权重信息，数组中的权重并不代表数据的数目，依然采用随机分配数据，并不是按照严格的比例分配
		val array: Array[Dataset[lang.Long]] = ds.randomSplit(Array(5, 2, 3))

		array.foreach(_.show())

		//采样数据,withReplacement表示有放回的抽样，采样比例是0.4
		ds.sample(withReplacement=false,fraction = 0.4)

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### OrderBy&sort

**案例**

~~~ java
object Test28 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		//ds.orderBy("age")
		//默认是按照升序排列
		ds.orderBy('age.desc).show()//使用orderBy就是为了配合sql语句
		//使用sort算子
		ds.sort('age).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### distinct&dropDuplicates

**案例**

~~~ java
object Test29 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("zhangsan", 35),Student("lisi", 35),Student("lisi", 35)).toDS()

		ds.distinct().show()
		//针对某一个列进行去重操作
		ds.dropDuplicates("age").show()
		
		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

### 集合操作

~~~ java
object Test30 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		//定义两个集合
		val coll1: Dataset[lang.Long] = spark.range(1, 10)
		val coll2: Dataset[lang.Long] = spark.range(5, 15)

		//计算差集
		coll1.except(coll2).show()

		//计算交集
		coll1.intersect(coll2).show()

		//计算并集
		coll1.union(coll2).show()

		//limit()的使用
		coll1.limit(3).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

### 无类型的转换算子

#### 选择

~~~ java
object Test31 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		//选中某一列进行输出,在dataset当中，select可以在任何的位置调用
		ds.select('name).show()

		//表达式的写法，计算年龄的累加和
		ds.selectExpr("count(age)").show()

		import org.apache.spark.sql.functions._
		//使用函数的方式调用
		ds.select(expr("sum(age)")).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### 新建列

~~~ java
object Test32 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		import org.apache.spark.sql.functions._

		/**
		 * 如果想使用函数功能
		 * 1 使用functions.xxx的形式
		 * 2 使用表达式，可以使用expr("xxx")形式，随时就可以编写
		 */

		//新添加一列,值是后面的表达式，产生随机值
		ds.withColumn("random",expr("rand()")).show()

		//对某一列进行重新命名,但是是重新创建一列，列的内容使用的是name列的内容，
		ds.withColumn("name_new",'name).show()
		//创建列的时候，也可以对列进行操作
		ds.withColumn("name_new1",'name+" ").show()

		//给列重新命名
		ds.withColumnRenamed("name","joy_name").show()
		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### 剪除&聚合

![1622857114476](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/093837-475991.png)

**案例**

~~~ java
object Test33 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		import org.apache.spark.sql.functions._

		/**
		 * 为什么groupByKey()是有类型的
		 * groupByKey()对数据分组后，必须调用聚合函数，、也就是说groupByKey()生成的对象，里面的算子是有类型的
		 */
		ds.groupByKey(item => item.name)

		/**
		 * groupBy()是没有类型的，可以按照某一列进行分组
		 * 主要是groupBy生成的对象的算子是无类型的
		 */
		//按照名字分组，然后取年龄的平均值
		ds.groupBy('name).agg(mean("age")).show()
		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

### Column对象

#### Column对象的创建

![1622857993227](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622857993227.png)

![1622858025402](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/095348-431289.png)

![1622858053936](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/095416-989989.png)

**案例**

~~~ java
object Test34 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()
		val ds1: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()
		val df: Dataset[Row] = Seq(("ziaorui", 25), ("xiaoming",15),("lisi", 35)).toDF("name","age")

		import org.apache.spark.sql.functions._
		//创建Column对象

		/**
		 * 使用 ' 创建对象，必须导入spark的隐式转换
		 * Symbol类似于java的internal,internal类型对象会放入常量池中
		 * 隐式转换：implicit def symbolToColumn(s: Symbol): ColumnName = new ColumnName(s.name)
		 */
		val column: Symbol = 'name

		/**
		 * $"name" 创建column，必须导入隐式转换
		 * $也是一个隐式转换，在底层会转换为ColumnName类型
		 */
		val column1:ColumnName = $"name"

		/**
		 * col 方式创建column,但是必须先导入functions
		 */
		val column2: Column = col("name")


		/*
		column方式创建column，但是必须先导入functions
		 */
		val column3: Column = column("name")

		//上面的四种创建方式，都没关联任何的dataset,但是可以直接使用
		//column没有关联ds，但是这里可以直接使用
		ds.select(column).show()

		df.select(column).show()
		//select算子可以使用column对象来选中某一个列，其他的算子也可以进行这样的操作

		df.where(column ==="xiaoming").show()

		/*
		dataset.col()
		下面的创建方式有什么区别，都是通过dataset进行创建，只不过两个column对象绑定了不同的dataset
		使用dataset获取某一个column对象，会和某一个dataset进行绑定，在逻辑执行计划中，会有不同的表现
		为什么要和dataset进行绑定呢？因为在进行连接的时候，要具体指明是哪一个dataset的列进行连接
		 */
		val column4: Column = ds.col("name")
		val column5: Column = ds1.col("name")
		//但是不能在一个dataset中去使用另外一个dataset的column对象
		//ds.select(column5).show()  这样使用会报错

		/*
		dataset.apply()创建column对象,通过ds对象的apply()方法进行创建
		下面的这两种写法等价
		 */
		val column6: Column = ds.apply("name")
		val column7: Column = ds("name")
		
		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

- column对象有四种创建方式
- column对象可以作用于Dataset和DataFrame当中
- column可以和命令式的弱类型的api配合使用，比如select,where
- column对象的创建分为两大类：
  - 有绑定的，有绑定方式，都一样，
  - 无绑定的，四种创建方式都一样

#### 操作

![1622861939141](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/105901-301043.png)

**案例**

~~~ java
object Test35 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("ziaorui", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		//给某一列添加别名
		ds.select('name.as("new_name")).show()

		//列的数据类型转换
		ds.select('age.as[Long]).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### 方法

![1622862290459](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/110452-329745.png)

**案例**

~~~ java
object Test36 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val ds: Dataset[Student] = Seq(Student("zhangsan", 25), Student("xiaoming",15),Student("lisi", 35)).toDS()

		//新添加一列
		//age*2本质上是将一个表达式附着到column对象上面，表达式在执行的时候，对应于每一条数据进行执行操作
		ds.withColumn("doubledAge",'age*2).show()

		//模糊查询：like
		ds.where('name like "li%").show()

		//对数据进行排序
		ds.sort('age desc).show()

		//枚举判断
		ds.where('name isin("lisi","zhangsan")).show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

### 缺失值处理

#### 缺失值的产生

![1622863075443](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/111757-224069.png)

![1622863655956](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/112736-188689.png)

![1622863732544](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/112853-370542.png)

#### 缺失值的处理

##### NaN&NULL缺失值处理

![1622864133292](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/113535-651636.png)

**数据的读取**

~~~ java
object Test37 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		/**
		 * 1 读取数据，通过spark.csv()自动推断类型进行读取
		 * 下面这种读取方式存在问题，推断数字的时候，会将nan这种值推断为字符串，导致整个列都变为字符串
		 */
		//spark.read
		//		.option("header",true)
		//		.option("inferSchema",true)//自动推断schema进行数据的读取
		//		.csv()

		//2 直接读取为字符串，然后使用map算子转换类型
		spark.read.csv("").map(row=>{row.anyNull})

		//3 指定schema,不要进行自动类型推断，
		//创建schema
		val schema=StructType(
			List(
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)
		//通过指定列的类型，如果在读取数据的时候，某一列有nan数值，那么就会转换为当前列数据类型下的nan类型
		//Double.NaN
		spark.read
				.option("header",true)
				.schema(schema)
				.csv("")

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

**第一种处理策略---丢弃**

![1622865243702](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/05/115405-252943.png)

案例

~~~ java
object Test37 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		/**
		 * 1 读取数据，通过spark.csv()自动推断类型进行读取
		 * 下面这种读取方式存在问题，推断数字的时候，会将nan这种值推断为字符串，导致整个列都变为字符串
		 */
		//spark.read
		//		.option("header",true)
		//		.option("inferSchema",true)//自动推断schema进行数据的读取
		//		.csv()

		//2 直接读取为字符串，然后使用map算子转换类型
		spark.read.csv("").map(row=>{row.anyNull})

		//3 指定schema,不要进行自动类型推断，
		//创建schema
		val schema=StructType(
			List(
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)
		//通过指定列的类型，如果在读取数据的时候，某一列有nan数值，那么就会转换为当前列数据类型下的nan类型
		//Double.NaN
		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv("")


		/*
		丢弃的规则：
		2019,12,12,60.0
			1 any:数据中，只要某一列为nan就丢弃数据记录
			2 all：所有都是nan的行才丢弃
			3 针对某些列丢弃
		 */
		//使用any()处理缺失值,
		//source.na返回的是一个DataFrameNaFunctions对象，不能直接调用show()算子，必须先进行缺失值处理，才可以show
		//val na: DataFrameNaFunctions = source.na

		source.na.drop("any").show()//默认也是使用any

		//使用all
		source.na.drop("all").show()

		//针对某一些列的规则,
		//也就是说只要list集合中的某一列为null，就执行丢弃操作,此时的any只是作用于list集合中的列
		source.na.drop("any",List("year","month","day")).show()


		/*
		缺失值的填充：
			1 针对所有列数据的默认值填充
			2 针对特定的列进行填充
		 */
		//针对所有列数据进行填充,把所有的缺失值填充为0
		source.na.fill(0).show()

		//针对特定的列进行填充
		source.na.fill(0,List("month","year"))

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

##### 字符串类型缺失值处理

![1622866307041](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622866307041.png)

**案例**

~~~ java
object Test39 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		/**
		 * 1 读取数据，通过spark.csv()自动推断类型进行读取
		 * 下面这种读取方式存在问题，推断数字的时候，会将nan这种值推断为字符串，导致整个列都变为字符串
		 */
		val source: DataFrame = spark.read
			.option("header", true)
			.option("inferSchema", true) //自动推断schema进行数据的读取
			.csv()

		//1 丢弃策略
		source.where('PM_Dongsi =!= "NA").show()

		//2 替换策略
		/*
		在select 语句中可以使用when ..... then 语句
		select name,age,grade from table
		when....then
		when....then
		else...
		 */
		import org.apache.spark.sql.functions._

		source.select('No as("id"),'year,'month,'day,when('PM_Dongsi === "NA",Double.NaN)
			.otherwise('PM_Dongsi cast(DoubleType))
		.as("pm")).show()

		//使用replace()方法替换
		//但是原来的类型和转换过后的类型必须一致
		source.na.replace("PM_Dongsi",Map("NA"-> "NAN","NULL"->"null")).show()



		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

### 聚合操作

数据操作的一般步骤

- 读取数据
- 数据操作
  - 类型转换，数据清洗
- 落地数据

#### groupBy

![1622874023559](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/101051-705820.png)

**案例**

~~~ java
object Test40 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._
		import org.apache.spark.sql.functions._

		//创建schema
		val schema=StructType(
			List(
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)

		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv()

		//首先对数据进行清洗
		source.where('pm =!=Double.NaN).show()

		//对数据进行分组操作,先按照年进行分组，在按照月进行分组
		val dataset: RelationalGroupedDataset = source.groupBy('year, $"month")

		//使用functions函数完成聚合操作
		//avg本质上定义了一个操作，把操作设置给pm列中的所有数据
		//select avg(pm) from table group by
		dataset.agg(avg('pm).as("pm_avg"))
			.orderBy('pm_avg.desc)//按照降序排列
			.show()

		//直接使用RelationalGroupedDataset的avg算子进行聚合操作
		dataset.avg("pm")
				.orderBy("avg(pm)")
				.show()

		//在这里还可以求出其他的统计量：mean(),max(),min(),sum(),stddev()方差

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

#### 多维聚合

需求一：

- 不同来源的pm统计
- 在同一个月，不同来源的pm值是多少
- 在同一年，不同来源的pm值平均是多少
- 整体来看，不同来源的pm值是多少

什么是多维聚合，多维聚合就是在一个统计结果中，包含总计，小计，

**案例**

~~~ java
object Test41 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		//创建schema
		val schema=StructType(
			List(
				//这里新增加一列，表示统计pm值的来源
				StructField("source",StringType),
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)

		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv()

		//不同年，不同来源，pm值的平均数
		//select source,year,avg(pm) as pmfrom table group by source,year
		val frame: DataFrame = source.groupBy('source, 'year)
			.agg(avg('pm).as("pm"))


		//在整个数据集中，按照不同来源统计pm值的平均数
		val frame1: DataFrame = source.groupBy('source)
			.agg(avg('pm).as("pm"))
				.select('source,lit(null) as "year",'pm)//lit的含义是新添加一列，内容是null


		//把上面的两个统计结果合并在一个数据集中
		frame.union(frame1)
				.sort('source,'year.asc_nulls_last,'pm)
				.show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

![1622945501757](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/101144-256127.png)

**rollup**

![1622945819842](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622945819842.png)

简单来说就是进行多次的分组操作，第一次先按照所有指定的字段进行分组，每次减少一个字段进行分组，直到剩下一个字段为止，对全局进行一个分组。

**案例**

~~~ java
object Test42 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		val source=Seq(
			("beijing",2016,100),
			("beijing",2017,200),
			("shanghai",2015,50),
			("shanghai",2016,150),
			("guangzhou",2015,50),
		).toDF("city","year","amount")

		//统计每一个城市每一年的消费额

		//统计每一个城市总共的消费额

		//总体的销售额


		/**
		 * 滚动分组 A，B==>最终会生成AB , A, NULL的分组结果，分完组后，还要对最终的结果进行聚合操作
		 */
		source.rollup('city,'year)
				.agg(sum('amount).as("amount"))
				.sort('city.asc_nulls_last,'year.asc_nulls_last)//表示这一列的空值排列在最后
				.show()

		/*
		+---------+----+------+
|     city|year|amount|
+---------+----+------+
|  beijing|2016|   100|  按照city,year统计的结果
|  beijing|2017|   200|
|  beijing|null|   300|  按照city进行统计，北京全部的销售额
|guangzhou|2015|    50|
|guangzhou|null|    50|
| shanghai|2015|    50|
| shanghai|2016|   150|
| shanghai|null|   200|
|     null|null|   550|  按照全部把整个数据加起来，整个公司所有年份的销售额
+---------+----+------+

		 */


		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

**案例二**

~~~ java
object Test43 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		//创建schema
		val schema=StructType(
			List(
				//这里新增加一列，表示统计pm值的来源
				StructField("source",StringType),
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)

		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv()

		/**
		 * 需求一：每一个pm统计者，一年pm值统计的平均值，按照source year进行分组
		 * 需求二：每一个pm统计者，整体上pm的平均值，按照source进行分组
		 * 需求三：全局所有的pm统计者，和日期的pm平均值，按照null进行分组
		 */

		source.rollup('source,'year)
				.agg(avg('pm.as("pm")))
				.sort('source.asc_nulls_last,'year.asc_nulls_last)
				.show()


		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

![1622948069981](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/105433-686867.png)

产生的问题

![1622948297777](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622948297777.png)

如果把source和year相互交换位置，产生的结果会一样么？明显不一样，因为在第二次分组时候，产生了分歧，rollup分组会按照指定的第一列为支点进行分组操作。所以可以使用cube进行弥补。

**cube**

![1622948483511](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/110127-494119.png)

**案例**

~~~ java
object Test44 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		//创建schema
		val schema=StructType(
			List(
				//这里新增加一列，表示统计pm值的来源
				StructField("source",StringType),
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)

		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv()

		source.cube('source,'year)
				.agg(avg('pm.as("pm")))
				.sort('source.asc_nulls_last,'year.asc_nulls_last)
				.show()

		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

![1622948717536](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622948717536.png)

最明显的结果是最后把每一年的平均值也统计出来了。

**使用sql语句完成**

~~~ java
object Test45 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		//创建schema
		val schema=StructType(
			List(
				//这里新增加一列，表示统计pm值的来源
				StructField("source",StringType),
				StructField("id",LongType),
				StructField("year",IntegerType),
				StructField("month",IntegerType),
				StructField("day",IntegerType),
				StructField("season",IntegerType),
				StructField("pm",DoubleType),
			)
		)

		val source: DataFrame = spark.read
			.option("header", true)
			.schema(schema)
			.csv()

		source.cube('source,'year)
				.agg(avg('pm.as("pm")))
				.sort('source.asc_nulls_last,'year.asc_nulls_last)
				.show()

		//使用sql语句完成
		//创建一个临时的表
		source.createOrReplaceTempView("pm_final")

		val res: DataFrame = spark.sql("select source,year,avg(pm) as pm from pm_final group by source,year grouping sets((source,year),(source),(year),())"
			+ "order by source asc nulls last,year asc nulls last")

		res.show()


		spark.stop()
	}
	case class Student(name:String,age:Int)
}
~~~

groupBy,cube,rollup返回的结果都是RelationalGroupedDataset类型。

#### RelationalGroupedDataset

![1622949559206](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622949559206.png)

![1622949677863](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/112120-734566.png)

RelationalGroupedDataset是一个分过组的数据集。

### 连接操作

![1622950090417](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/112813-874755.png)

![1622951925287](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/115847-97007.png)

![1622951942666](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/115906-964223.png)

**案例**

~~~ java
object Test46 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 0)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")


		val person=df.join(city,df.col("cityId") === city.col("id"))
				.select(df.col("id"),df.col("name"),city.col("name").as("city"))


		person.createOrReplaceTempView("per_city")

		spark.sql("select id,name,city from per_city where city='beijing'").show()

		spark.stop()
	}
}
+---+------+-------+
| id|  name|   city|
+---+------+-------+
|  0|  Lucy|beijing|
|  1|  lily|beijing|
|  3|Danial|beijing|
+---+------+-------+
~~~

#### 交叉连接

![1622952090803](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/120134-340878.png)

![1622952194580](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/120317-241979.png)

交叉连接，可以认为是笛卡尔积操作。

~~~ java
object Test47 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 0)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")


		//交叉连接,添加限定条件准确的说是内连接
		df.crossJoin(city)
				.where(df.col("cityId")===city.col("id"))
				.show()

		df.createOrReplaceTempView("df")
		city.createOrReplaceTempView("city")

		//使用sql语句写
		spark.sql("select u.id,u.name,c.name from df u cross join city c where u.cityId = c.id").show()
		
		spark.stop()
	}
}
~~~

#### 内连接

![1622953428003](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/122350-30297.png)

**案例**

~~~ java
object Test48 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"inner").show()

		//使用sql表达
		spark.sql("select p.id,p.name,c.name from p inner join c on p.cityId = c.id").show()
		spark.stop()
	}
}
~~~

#### 全外连接

![1622954224923](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/123711-316454.png)

**案例**

~~~ java
object Test49 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"full").show()

		//使用sql表达
		spark.sql("select p.id,p.name,c.name from p full outer join c on p.cityId = c.id").show()
		spark.stop()
	}
}
~~~

#### 左外连接

![1622954645973](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/124408-614422.png)

**案例**

~~~ java
object Test50 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"left").show()

		//使用sql表达
		spark.sql("select p.id,p.name,c.name from p left join c on p.cityId = c.id").show()
		spark.stop()
	}
}
~~~

#### 右外连接

![1622954757256](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/124559-879564.png)

**案例**

~~~ java
object Test50 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"right").show()

		//使用sql表达
		spark.sql("select p.id,p.name,c.name from p right join c on p.cityId = c.id").show()
		spark.stop()
	}
}
~~~

#### LeftAnti

显示两个表没有连接上的部分，右侧表不显示

![1622956575669](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622956575669.png)

![1622956593767](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/131635-582249.png)

**案例**

~~~ java
object Test51 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"leftanti").show()

		//使用sql表达,因为不显示右边的数据，所以查询不到c.name
		spark.sql("select p.id,p.name from p left anti join c on p.cityId = c.id").show()
		/*
		+---+------+
| id|  name|
+---+------+
|  3|Danial|
+---+------+
		 */
		spark.stop()
	}
}
~~~

#### LeftSemi

显示连接上的部分，右侧表不显示

![1622956632836](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1622956632836.png)

![1622956723389](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/06/131917-72256.png)

**案例**

~~~ java
object Test52 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._

		val df: DataFrame = Seq((0, "Lucy", 0), (1, "lily", 0), (2, "Tim", 2), (3, "Danial", 3)).toDF("id", "name", "cityId")

		val city: DataFrame = Seq((0, "beijing"), (1, "shanghai"), (2, "guangzhou")).toDF("id", "name")

		df.createOrReplaceTempView("p")
		city.createOrReplaceTempView("c")


		//内连接,最后一个参数指定的是连接的方式
		df.join(city,df.col("cityId")===city.col("id"),"leftsemi").show()

		//使用sql表达,因为不显示右边的数据，所以查询不到c.name
		spark.sql("select p.id,p.name from p left semi join c on p.cityId = c.id").show()
		/*
		+---+----+
| id|name|
+---+----+
|  0|Lucy|
|  1|lily|
|  2| Tim|
+---+----+
		 */
	
		spark.stop()
	}
}
~~~

### 函数

**案例**

~~~ java
object Test53 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._
		import org.apache.spark.sql.functions._


		val source = Seq(
			("Thin", "Cell phone", 6000),
			("normal", "Tablet", 1500),
			("Mini", "Tablet", 5500),
			("Ultra thin", "Cell phone", 5500),
			("Very thin", "Cell phone", 6000),
			("Big", "Tablet", 2500),
			("Bendable", "Cell phone", 3000),
			("Foldable", "Cell phone", 3000),
			("Pro", "Tablet", 4500),
			("Pro2", "Tablet", 6500)
		).toDF("product","category","revenue")


		//聚合每一个类别的总价，先分组，然后对每一组的数据进行聚合操作
		source.groupBy('category)
				.agg(sum('revenue))
				.show()

		//把名称变为小写
		//注意：functions下面的函数，会作用于一列数据，然后在返回新的一列
		source.select(lower('product)).show()

		//把价格变为字符串的形式
		//注册自定义函数
		val toStrUDF: UserDefinedFunction = udf(toStr(_))

		source.select('product,'category,toStrUDF('revenue)).show()




		spark.stop()
	}
	//自定义函数
	def toStr(reveine:Long):String={
		(reveine/1000)+"k"
	}
}
~~~

### 窗口函数

![1623025265601](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623025265601.png)

![1623025281264](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623025281264.png)

**案例**

~~~ java
object Test54 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import org.apache.spark.sql.functions._
		import spark.implicits._


		val source = Seq(
			("Thin", "Cell phone", 6000),
			("normal", "Tablet", 1500),
			("Mini", "Tablet", 5500),
			("Ultra thin", "Cell phone", 5500),
			("Very thin", "Cell phone", 6000),
			("Big", "Tablet", 2500),
			("Bendable", "Cell phone", 3000),
			("Foldable", "Cell phone", 3000),
			("Pro", "Tablet", 4500),
			("Pro2", "Tablet", 6500)
		).toDF("product","category","revenue")

		//1 定义窗口

		val win: WindowSpec = Window.partitionBy('category)
			.orderBy('revenue.desc)

		//dense_rank会生成编号,根据生成的win进行排序
		source.select('product,'category,dense_rank().over(win).as("rank"))
				.where('rank <= 2)
				.show()

		//使用sql完成
		source.createOrReplaceTempView("tab")
		//spark.sql("select product,category,revenue from (select *,dense_rank() " +
		//	"over (partitionBy category order by revenue desc as rank) from tab) where rank <=2")

		//统计每一个商品，和此品类商品最贵商品之间的差值

		spark.stop()
	}
}
~~~

**从逻辑上讲，窗口函数的执行大致分为以下步骤**

![1623026710355](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/084514-214857.png)

开辟窗口函数，使用over函数，over函数里面的内容表示如何开辟窗口函数。

![1623026818092](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/084701-549952.png)

![1623026860783](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/084902-636171.png)

![1623026985084](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/084947-775099.png)

函数部分为作用于窗口的函数。

**窗口的定义部分**

![1623027074280](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623027074280.png)

窗口的定义使用partitionBy()函数，就是对数据进行分区操作。

![1623027130792](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623027130792.png)

定义每一个分区或者窗口内部数据的顺序。

![1623027306210](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623027306210.png)

![1623027344921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/085548-937304.png)

**函数部分**

![1623027399668](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/085642-791192.png)

![1623027460846](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623027460846.png)

**统计每一个商品，和此品类商品最贵商品之间的差值**

~~~ java
object Test55 {

	def main(args: Array[String]): Unit = {

		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._
		import org.apache.spark.sql.functions._


		val source = Seq(
			("Thin", "Cell phone", 6000),
			("normal", "Tablet", 1500),
			("Mini", "Tablet", 5500),
			("Ultra thin", "Cell phone", 5500),
			("Very thin", "Cell phone", 6000),
			("Big", "Tablet", 2500),
			("Bendable", "Cell phone", 3000),
			("Foldable", "Cell phone", 3000),
			("Pro", "Tablet", 4500),
			("Pro2", "Tablet", 6500)
		).toDF("product","category","revenue")

		//统计每一个商品，和此品类商品最贵商品之间的差值

		//1 定义窗口，按照分类进行倒序排列
		val win: WindowSpec = Window.partitionBy('category)
			.orderBy('revenue.desc)

		//2 找到最贵商品价格,根据窗口，找到最大值
		//max是作用于窗口的函数，win是开辟好的窗口
		val maxPrice:Column=max('revenue) over(win)

		//3 获取结果
		source.select('product,'category,'revenue,(maxPrice-'revenue).as("reve_diff"))
				.show()

		spark.stop()
	}
}
~~~

## 小项目

### 业务

**数据结构**

![1623028757244](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093838-363777.png)

**业务场景**

![1623029918131](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093856-684959.png)

**技术分析**

![1623029936833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/093858-855850.png)

### 流程分析

![1623030565008](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623030565008.png)

![1623030863366](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/095426-64231.png)

### 读取数据

数据的schema信息。

![1623034020906](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/104757-694256.png)

### 数据转换

![1623034427231](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623034427231.png)

### 异常的处理

![1623060467625](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1623060467625.png)

![1623060485758](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/180824-529125.png)

![1623060537857](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/180902-364382.png)

![1623062104561](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/07/183507-564039.png)

**案例**

~~~ java
object EitherTest {
	/**
	 * Either的使用
	 * @param args
	 */
	def main(args: Array[String]): Unit = {
		val res: Either[Double, (Double, Exception)] = safe(process, 2)

		res match {
			case Left(r)=>println(res.left.get)
			case Right((b,e))=>println(b,e)
		}

	}

	/**
	 *Either就相当于left or right,left和right并不代表具体的某一种情况，只是代表左右
	 * Option===>some  None
	 * @param f 代表要传进来的函数
	 * @param b 代表传进来的函数的参数
	 * @return
	 */
	def safe(f:Double => Double,b:Double):Either[Double,(Double,Exception)]={
		try{
			val res: Double = f(b)
			Left(res)
		}catch {
			//		出错误的话，返回right,第一个参数是函数的参数，第二个参数是出错误的e
			case e:Exception=>Right(b,e)
		}
	}

	def process(b:Double):Double={
		val a=10.0
		a/b
	}
}
~~~

### 减掉反常数据







### 行政区信息

#### 需求分析

![1623112785578](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/083946-268376.png)

![1623112824122](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084024-403266.png)

![1623112894330](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084136-594658.png)

![1623112935319](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084215-356957.png)

![1623112957795](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084238-900562.png)

![1623113118227](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084518-619249.png)

![1623113139182](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084539-629311.png)

![1623113170599](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084612-745342.png)

#### 工具介绍

![1623113388683](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/084951-501297.png)

![1623113412493](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/085013-401388.png)

**导入Maven依赖**

~~~ java
<!--json解析库-->
        <dependency>
            <groupId>org.json4s</groupId>
            <artifactId>json4s-native_2.11</artifactId>
            <version>${json4s.version}</version>
        </dependency>
        <!--json4s的jackson继承库-->
        <dependency>
            <groupId>org.json4s</groupId>
            <artifactId>json4s-scalap_2.12</artifactId>
            <version>3.6.6</version>
        </dependency>
~~~

**案例**

~~~ java
package rzf.qq.com

import org.json4s.jackson.Serialization
import org.json4s.jackson.Serialization.read
object JsonTest {

	def main(args: Array[String]): Unit = {

		//import org.json4s._
		//import org.json4s.jackson.JsonMethods._
		//
		//val product=
		//	"""
		//		|{"name":"Toy","price":35.5}
		//	""".stripMargin
		//
		//implicit val formats=Serialization.formats(NoTypeHints)
		//
		////具体解析为某一个对象
		//val pro: Any = println(parse(product)).extract[Product]
		//
		////可以直接通过一个方法，将字符串转换为对象
		//val value: Any = read[Product](product)
		//println(pro)
		//
		//val pro1: Product = Product("电视", 20)
		//compact(render(parse(pro1)))
		//val value1: Any = write(pro1)

	}


}
case class Product(name:String,price:Double)
~~~

![1623115079075](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/092128-666625.png)

**小结**

![1623115301150](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202106/08/093615-290331.png)

### 最终代码

~~~ java
package rzf.qq.com

import java.text.SimpleDateFormat
import java.util.Locale
import java.util.concurrent.TimeUnit

import com.esri.core.geometry.{Geometry, GeometryEngine, MapGeometry, Point, SpatialReference}
import org.apache.ivy.osgi.updatesite.xml.FeatureParser
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.expressions.UserDefinedFunction
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
import org.json4s.{JObject, NoTypeHints}
import org.json4s.jackson.Serialization
import org.json4s.jackson.Serialization.read

import scala.io.{BufferedSource, Source}

object ProcessTaxi {

	def main(args: Array[String]): Unit = {
		val spark = SparkSession.builder()
			.master("local[6]")
			.appName("typed")
			.getOrCreate()

		import spark.implicits._
		import org.apache.spark.sql.functions._
		//1 读取数据
		val source: DataFrame = spark.read
			.option("header", value = true)
			.csv("")
	//	2 数据转换操作,将数据转换为具体的数据类型
	val taxiParse: RDD[Either[Trip, (Row, Exception)]] = source.rdd.map(safe((parse)))
		//可以通过如下方式过滤出所有异常的Row
		//taxiParse.filter(e => e.isRight)
		//	.map(e => e.right.get._1)

		val taxiGood: Dataset[Trip] = taxiParse.map(either => either.left.get).toDS()

	//	绘制时长直方图
	//	编写udf函数，完成时长的计算，将毫秒转换为小时
		val hours=(pickUpTime:Long,dropOffTime:Long)=>{
		val duration=dropOffTime-pickUpTime
		val hours=TimeUnit.HOURS.convert(duration,TimeUnit.MICROSECONDS)
		hours
	}
		val hoursUdf=udf(hours)
	//	具体的统计
		taxiGood.groupBy(hoursUdf($"pickUpTime",$"dropOffTime").as("duration"))
			.count()
			.sort("duration")
			.show()

	//	根据直方图的显示，查看数据分布之后，减掉异常的数据
		spark.udf.register("hours",hours)

		val taxiClean: Dataset[Trip] = taxiGood.where("hours(pickUpTime,dropOffTime) BETWEEN 0 AND 3")

	//	增加行政区信息
		/**
		 * 读取数据
		 * 排序
		 * 广播
		 * 创建udf，完成功能
		 * 统计信息
		 */

		val string: String = Source.fromFile("").mkString
		val collection: FeatureCollection = jsonParse(string)
	//	后期要找到每一个出粗车所在的行政区，拿到经纬度，遍历feature搜索其所在的行政区
	//	在搜索的过程中，面积越大，命中的概率越大，把大的行政区放在前面，减少遍历次数
	val sortedFeat: List[Feature] = collection.features.sortBy(feat => {
		(feat.properties("brooughCode"), feat.getGeometry().calculateArea2D())
	})
		val featBC: Broadcast[List[Feature]] = spark.sparkContext.broadcast(sortedFeat)

		val boroughLookUp=(x:Double,y:Double)=>{
		//	搜索经纬度所在的区域
		val featHit: Option[Feature] = featBC.value.find(feat => {
			GeometryEngine.contains(feat.getGeometry(), new Point(x, y), SpatialReference.create(4326))
		})
			val borough: String = featHit.map(feat => feat.properties("borought")).getOrElse("NA")
			borough
		}

		val brough: UserDefinedFunction = udf(boroughLookUp)
		taxiClean.groupBy(brough('dropOffx,'dropOffy))
			.count()
			.show()

	}


	//解析json对象
	def jsonParse(json:String):FeatureCollection={
		//导入隐式转换
		implicit val formats=Serialization.formats(NoTypeHints)

	//	json--->obj
		val collection: FeatureCollection = read[FeatureCollection](json)
		collection
	}


	/**
	 *
	 * @tparam P 参数的类型
	 * @tparam R 返回值的类型
	 * @return
	 */
	def safe[P,R](f:P=>R):P=>Either[R,(P,Exception)]={
		new Function[P,Either[R,(P,Exception)]] with Serializable{
			override def apply(param: P): Either[R, (P, Exception)] = {
				try{
					Left(f(param))
				}catch {
					case exception: Exception=>Right((param,exception))
				}
			}
		}
	}

	def parse(row:Row):Trip={
		val richRow = new RichRow(row)
		val license=richRow.getAs[String]("hack_license").orNull
		val pickUpTime=parseTime(richRow,"pickup_datetime")
		val dropOffTime=parseTime(richRow,"dropoff_datetime")
		val pickUpX=parseLocation(richRow,"pickup_longitude")
		val pickUpY=parseLocation(richRow,"pickup_latitude")
		val dropOffX=parseLocation(richRow,"dropoff_longitude")
		val dropOffY=parseLocation(richRow,"dropoff_latitude")
		Trip(license,pickUpTime,dropOffTime,pickUpX,pickUpY,dropOffX,dropOffY)
	}

	/**
	 * 时间的转换
	 * @param richRow
	 * @param field
	 * @return
	 */
	def parseTime(richRow:RichRow,field:String):Long={
	//	表示出时间的格式
		var pattern="yyyy-MM-dd HH:mm:ss"
		val formatter = new SimpleDateFormat(pattern, Locale.ENGLISH)
	//	执行转换，获取Data对象，getTime()获取时间戳
		val time: Option[String] = richRow.getAs[String](field)
		//将String类型的时间，转换为Long类型
		val timeoption: Option[Long] = time.map(time => formatter.parse(time).getTime)
		timeoption.getOrElse(0l)
	}

	/*
	地理位置的转换
	 */
	def parseLocation(richRow: RichRow,field:String):Double={
	//	获取数据，获取的数据类型是option类型
		val location: Option[String] = richRow.getAs[String](field)
	//	将String类型转换为Double类型
		val loca: Option[Double] = location.map(loc => loc.toDouble)
		loca.getOrElse(0.0d)
	}

}

/**
 * DataFrame中Row的包装类型，主要是为了包装getAs()方法
 * @param row
 */
class RichRow(row:Row){
	def getAs[T](field:String):Option[T]={
	//	1 判断row.getAs()是否是空，row中对应的field是否是空的
	//	row.fieldIndex(field):根据
		if(row.isNullAt(row.fieldIndex(field))){
		//NULL---->返回None
			None
		}else{
			Some(row.getAs[T](field))
		}

	}
}

case class Trip(
	license:String,
	pickUpTime:Long,
	dropOffTime:Long,
	pickUpX:Double,
	pickUpY:Double,
	dropOffX:Double,
	dropOffY:Double
)

case class FeatureCollection(features:List[Feature])

case class Feature(properties:Map[String,String],geometry:JObject){
	def getGeometry():Geometry={

		import org.json4s._
		import org.json4s.jackson.JsonMethods._
		val geometry1: MapGeometry = GeometryEngine.geoJsonToGeometry(compact(render(geometry)), 0, Geometry.Type.Unknown)
		geometry1.getGeometry
	}
}
~~~

---

over!