## Spark版WordCount

### 增加Scala插件

Spark 由 Scala语言开发的，所以本课件接下来的开发所使用的语言也为 Scala，咱们当前使用的Spark 版本为 3.0.0，默认采用的 Scala编译版本为 2.12，所以后续开发时。我们依然采用这个版本。开发前请保证 IDEA 开发工具中含有 Scala 开发插件

### 增加依赖关系

修改 Maven 项目中的POM文件，增加 Spark 框架的依赖关系。本课件基于 Spark3.0 版本，使用时请注意对应版本。

```java
</executions>
</plugin>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-assembly-plugin</artifactId>
<version>3.1.0</version>
<configuration>
<descriptorRefs>
<descriptorRef>jar-with-dependencies</descriptorRef>
</descriptorRefs>
</configuration>
<executions>
<execution>
<id>make-assembly</id>
<phase>package</phase>
<goals>
<goal>single</goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>
</build>

```

### WordCount代码

为了能直观地感受 Spark 框架的效果，接下来我们实现一个大数据学科中最常见的教学案例WordCount

```java
// 创建 Spark 运行配置对象
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

// 创建 Spark 上下文环境对象（连接对象）
val sc : SparkContext = new SparkContext(sparkConf)

// 读取文件数据
val fileRDD: RDD[String] = sc.textFile("input/word.txt")

// 将文件中的数据进行分词
val wordRDD: RDD[String] = fileRDD.flatMap( _.split(" ") )

// 转换数据结构 word => (word, 1)
val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_,1))

// 将转换结构后的数据按照相同的单词进行分组聚合
val word2CountRDD: RDD[(String, Int)] = word2OneRDD.reduceByKey(_+_)

// 将数据聚合结果采集到内存中
val word2Count: Array[(String, Int)] = word2CountRDD.collect()

// 打印结果
word2Count.foreach(println)

//关闭 Spark 连接
sc.stop()

```

执行过程中，会产生大量的执行日志，如果为了能够更好的查看程序的执行结果，可以在项目的 resources 目录中创建log4j.properties 文件，并添加日志配置信息：

```java
log4j.rootCategory=ERROR, console	
log4j.appender.console=org.apache.log4j.ConsoleAppender log4j.appender.console.target=System.err log4j.appender.console.layout=org.apache.log4j.PatternLayout log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Set the default spark-shell log level to ERROR. When running the spark-shell, the
# log level for this class is used to overwrite the root logger's log level, so that
# the user can have different defaults for the shell and regular Spark apps. log4j.logger.org.apache.spark.repl.Main=ERROR

# Settings to quiet third party logs that are too verbose log4j.logger.org.spark_project.jetty=ERROR log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR

# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
```

### 异常处理

如果本机操作系统是 Windows，在程序中使用了 Hadoop 相关的东西，比如写入文件到

 HDFS，则会遇到如下异常：

![1614162613380](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183014-908834.png)

出现这个问题的原因，并不是程序的错误，而是windows 系统用到了 hadoop相关的服务，解决办法是通过配置关联到 windows的系统依赖就可以了

![1614162651445](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183051-376431.png)

在 IDEA 中配置Run Configuration，添加HADOOP_HOME 变量

![1614162677667](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/10/185321-572066.png)

![1614162699458](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183139-709519.png)

### 程序执行过程

![1621503287579](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/20/173453-756684.png)

![1621666875776](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/150119-520237.png)

### main函数详细执行过程

 Spark Application应用程序运行时，无论client还是cluster部署模式DeployMode，当Driver Program和Executors启动完成以后，就要开始执行应用程序中MAIN函数的代码，以词频统计WordCount程序为例剖析讲解。

![1621666968036](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165611-908767.png)

- 构建SparkContex对象和关闭SparkContext资源，都是在Driver Program中执行，上图中①和③都是，如下图所示：

![1621667013112](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621667013112.png)

- 上图中②的**加载数据【A】、处理数据【B】和输出数据【C】**代码，都在Executors上执行，从WEB UI监控页面可以看到此Job（RDD#action触发一个Job）对应DAG图，如下所示：

![1621667064260](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202110/28/165612-140919.png)

将结果数据resultRDD调用saveAsTextFile方法，保存数据到外部存储系统中，代码在Executor中执行的。但是如果resultRDD调用take、collect或count方法时，获取到最终结果数据返回给Driver，代码如下：

![1621667091226](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621667091226.png)

运行应用程序时，将数组resultArray数据打印到标准输出，Driver Program端日志打印结果：

![1621667108160](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/150511-983217.png)

综上所述Spark Application中Job执行有两个主要点：

- RDD输出函数分类两类
  - 返回值给Driver Progam，比如count、first、take、collect等，也就是最后由行动算子触发操作。
  - 没有返回值，比如直接打印结果、保存至外部存储系统（HDFS文件）等，不调用行动算子。
- 在Job中从读取数据封装为RDD和一切RDD调用方法都是在Executor中执行，其他代码都是在Driver Program中执行
- SparkContext创建与关闭、其他变量创建等在Driver Program中执行
- RDD调用函数都是在Executors中执行