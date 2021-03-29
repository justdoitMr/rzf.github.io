# Spark笔记

RDD的三个特性

- 分区和shuffle
- 缓存
- 检查点

## Spark概述

Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

### Spark and Hadoop

首先从时间节点上来看:

**Hadoop**

- 2006 年 1 月，Doug Cutting 加入Yahoo，领导Hadoop 的开发

- 2008 年 1 月，Hadoop 成为 Apache 顶级项目

- 2011 年 1.0 正式发布

- 2012 年 3 月稳定版发布

- 2013 年 10 月发布 2.X (Yarn)版本

**Spark**

- 2009 年，Spark 诞生于伯克利大学的AMPLab 实验室

- 2010 年，伯克利大学正式开源了 Spark 项目

- 2013 年 6 月，Spark 成为了 Apache 基金会下的项目

- 2014 年 2 月，Spark 以飞快的速度成为了 Apache 的顶级项目

- 2015 年至今，Spark 变得愈发火爆，大量的国内公司开始重点部署或者使用 Spark

从功能上面看

**Hadoop**

- Hadoop 是由 java 语言编写的，在分布式服务器集群上存储海量数据并运行分布式分析应用的开源框架

- 作为 Hadoop 分布式文件系统，HDFS 处于 Hadoop 生态圈的最下层，存储着所有的数据， 支持着 Hadoop 的所有服务。 它的理论基础源于 Google 的TheGoogleFileSystem 这篇论文，它是GFS 的开源实现。

- **MapReduce 是一种编程模型**，Hadoop 根据 Google 的 MapReduce 论文将其实现， 作为 Hadoop 的分布式计算模型，是 Hadoop 的核心。基于这个框架，分布式并行程序的编写变得异常简单。综合了 HDFS 的分布式存储和 MapReduce 的分布式计算，Hadoop 在处理海量数据时，性能横向扩展变得非常容易。

- HBase 是对 Google 的 Bigtable 的开源实现，但又和 Bigtable 存在许多不同之处。HBase 是一个基于HDFS 的分布式数据库，**擅长实时地随机读/写超大规模数据集**。它也是 Hadoop 非常重要的组件。

**Spark**

- Spark 是一种由 Scala 语言开发的快速、通用、可扩展的大数据分析引擎
- Spark Core 中提供了 Spark 最基础与最核心的功能
- Spark SQL 是Spark 用来操作结构化数据的组件。通过 Spark SQL，用户可以使用SQL 或者 Apache Hive 版本的 SQL 方言（HQL）来查询数据。
- Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API。

### Spark和Hadoop的对比

**MapReducer计算架构**

![1611807626524](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/080027-550187.png)

可以看到，mapreducer多次的迭代计算，中间的数据存储使用的是磁盘，所以会有很多次的io操作。多个作业之间的交互是基于磁盘进行的。

**Spark计算架构**

![1611807662937](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/081140-335630.png)

基于Spark的迭代计算，中间的数据计算过程存储在内存中，减少io所花费的时间。

### Spark or Hadoop

Hadoop 的 MR 框架和Spark 框架都是数据处理框架，那么我们在使用时如何选择呢？

- Hadoop MapReduce 由于其设计初衷并不是为了满足循环迭代式数据流处理，因此在多并行运行的数据可复用场景（如：机器学习、图挖掘算法、交互式数据挖掘算法）中存在诸多计算效率等问题。所以 Spark 应运而生，Spark 就是在传统的MapReduce 计算框架的基础上，利用其计算过程的优化，从而大大加快了数据分析、挖掘的运行和读写速度，并将计算单元缩小到更适合并行计算和重复使用的RDD 计算模型
- 机器学习中 ALS、凸优化梯度下降等。这些都需要基于数据集或者数据集的衍生数据反复查询反复操作。MR 这种模式不太合适，即使多 MR 串行处理，性能和时间也是一个问题。数据的共享依赖于磁盘。另外一种是交互式数据挖掘，MR 显然不擅长。而Spark 所基于的 scala 语言恰恰擅长函数的处理。
- Spark 是一个分布式数据快速分析项目。它的核心技术是**弹性分布式数据集**（Resilient Distributed Datasets），提供了比MapReduce 丰富的模型，可以快速在内存中对数据集进行多次迭代，来支持复杂的数据挖掘算法和图形计算算法。

- Spark 和Hadoop 的**根本差异是多个作业之间的数据通信问题** : Spark 多个作业之间数据通信是基于内存，而 Hadoop 是基于磁盘。

- Spark  Task 的启动时间快。Spark 采用 fork 线程的方式，而 Hadoop 采用创建新的进程的方式。

- Spark 只有在 shuffle 的时候将数据写入磁盘，而 Hadoop 中多个 MR 作业之间的数据交互都要依赖于磁盘交互

- Spark 的缓存机制比HDFS 的缓存机制高效。

经过上面的比较，我们可以看出在绝大多数的数据计算场景中，Spark 确实会比 MapReduce 更有优势。但是Spark 是基于内存的，所以在实际的生产环境中，由于内存的限制，可能会由于内存资源不够导致 Job 执行失败，此时，MapReduce 其实是一个更好的选择，所以 Spark 并不能完全替代 MR。

### Spark核心模块



![1614162089530](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/182130-763014.png)

#### Spark Core

Spark Core 中提供了 Spark 最基础与最核心的功能，Spark 其他的功能如：Spark SQL， Spark Streaming，GraphX, MLlib 都是在 Spark Core 的基础上进行扩展的

#### Spark SQL

Spark SQL 是Spark 用来操作**结构化数据**的组件。通过 Spark SQL，用户可以使用 SQL或者Apache Hive 版本的 SQL 方言（HQL）来查询数据。

#### Spark Streaming

Spark Streaming 是 Spark 平台上针对实时数据进行流式计算的组件，提供了丰富的处理数据流的API。

#### Spark MLlib

MLlib 是 Spark 提供的一个机器学习算法库。MLlib 不仅提供了模型评估、数据导入等额外的功能，还提供了一些更底层的机器学习原语

#### Spark GraphX

GraphX 是 Spark 面向图计算提供的框架与算法库。

## Spark版WordCount

### 增加Scala插件

Spark 由 Scala语言开发的，所以本课件接下来的开发所使用的语言也为 Scala，咱们当前使用的Spark 版本为 3.0.0，默认采用的 Scala编译版本为 2.12，所以后续开发时。我们依然采用这个版本。开发前请保证 IDEA 开发工具中含有 Scala 开发插件

### 增加依赖关系

修改 Maven 项目中的POM文件，增加 Spark 框架的依赖关系。本课件基于 Spark3.0 版本，使用时请注意对应版本。

~~~ java
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

~~~

### WordCount代码

为了能直观地感受 Spark 框架的效果，接下来我们实现一个大数据学科中最常见的教学案例WordCount

~~~ java
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

~~~

执行过程中，会产生大量的执行日志，如果为了能够更好的查看程序的执行结果，可以在项目的 resources 目录中创建log4j.properties 文件，并添加日志配置信息：

~~~ java
log4j.rootCategory=ERROR, console	
log4j.appender.console=org.apache.log4j.ConsoleAppender log4j.appender.console.target=System.err log4j.appender.console.layout=org.apache.log4j.PatternLayout log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Set the default spark-shell log level to ERROR. When running the spark-shell, the
# log level for this class is used to overwrite the root logger's log level, so that
# the user can have different defaults for the shell and regular Spark apps. log4j.logger.org.apache.spark.repl.Main=ERROR

# Settings to quiet third party logs that are too verbose log4j.logger.org.spark_project.jetty=ERROR log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR

# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
~~~

### 异常处理

如果本机操作系统是 Windows，在程序中使用了 Hadoop 相关的东西，比如写入文件到

 HDFS，则会遇到如下异常：

![1614162613380](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183014-908834.png)

出现这个问题的原因，并不是程序的错误，而是windows 系统用到了 hadoop相关的服务，解决办法是通过配置关联到 windows的系统依赖就可以了

![1614162651445](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183051-376431.png)

在 IDEA 中配置Run Configuration，添加HADOOP_HOME 变量

![1614162677667](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/10/185321-572066.png)

![1614162699458](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183139-709519.png)

## Spark运行环境

Spark 作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行, 在国内工作中主流的环境为Yarn，不过逐渐容器式环境也慢慢流行起来。接下来，我们就分别看看不同环境下Spark 的运行。

![1614129731403](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/092212-410526.png)

### Local模式

所谓的Local 模式，就是不需要其他任何节点资源就可以在本地执行 Spark 代码的环境，一般用于教学，调试，演示等， 之前在 IDEA 中运行代码的环境我们称之为开发环境，和本地模式不是一个概念。

#### 解压缩文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到Linux 并解压缩，放置在指定位置，路径中不要包含中文或空格。

~~~ java
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module cd /opt/module
mv spark-3.0.0-bin-hadoop3.2 spark-local
~~~

#### 启动 Local 环境

1. 进入解压缩后的路径，执行如下指令

~~~ java
bin/spark-shell
~~~

**启动界面**

![1614129938413](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/092539-625717.png)

2. 启动成功后，可以输入网址进行 Web UI 监控页面访问，默认的端口号是4040

![1614129986153](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/092627-5531.png)

#### 退出本地模式

按键Ctrl+C 或输入 Scala 指令：quit

#### 提交应用

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \  这里表示是本地模式
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10
~~~

- --class 表示要执行程序的主类，此处可以更换为咱们自己写的应用程序，需要写全类名

- --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟CPU 核数量

-  spark-examples_2.12-3.0.0.jar 运行的应用类所在的 jar 包，实际使用时，可以设定为咱们自己打的 jar 包

- 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

**执行程序结果**

![1614130382503](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/093303-148263.png)

### Standalone 模式

local 本地模式毕竟只是用来进行练习演示的，真实工作中还是要将应用提交到对应的集群中去执行，这里我们来看看只使用 Spark 自身节点运行的集群模式，也就是我们所谓的独立部署（Standalone）模式。Spark 的 Standalone模式体现了经典的master-slave 模式（主从模式）。

master和worder是和资源有关的节点，driver和executor是和计算有关的节点。

**集群规划**

|       | Linux1        | Linux2 | Linux3 |
| ----- | ------------- | ------ | ------ |
| Spark | Worker Master | Worker | Worker |

#### 解压缩文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到Linux 并解压缩在指定位置

~~~ java
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module cd /opt/module
mv spark-3.0.0-bin-hadoop3.2 spark-standalone
~~~

#### 修改配置文件

1. 进入解压缩后路径的 conf 目录，修改 slaves.template 文件名为 slaves

~~~ java
//修改文件名
mv slaves.template slaves 
//修改slaves 文件，添加work 节点
hadoop100
hadoop101
hadoop102
~~~

2. 修改 spark-env.sh.template 文件名为 spark-env.sh

~~~ JAVA
//修改文件名
mv spark-env.sh.template spark-env.sh
//修改 spark-env.sh 文件，添加 JAVA_HOME 环境变量和集群对应的 master 节点
export JAVA_HOME=/opt/module/jdk1.8.0_144 
SPARK_MASTER_HOST=linux1 主机名称   master主机地址
SPARK_MASTER_PORT=7077 默认端口号，表示通过哪一个端口号链接集群
~~~

- 注意：7077 端口，相当于 hadoop内部通信的 8020 端口，此处的端口需要确认自己的 Hadoop

3. 分发 spark-standalone 目录

~~~ java
xsync spark
~~~

#### 启动集群

1. 执行脚本命令：

~~~ java
//启动全部节点
sbin/start-all.sh 
~~~

**查看进程**

hadoop100主机进程，其他节点都是从机，worker节点

![1614131301562](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/183847-88831.png)

2. 查看 Master 资源监控Web UI 界面: http://linux1:8080，默认是通过8080端口访问

![1614131626694](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/095415-520270.png)

#### 提交应用

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop100:7077 \     这里是和本地模式的区别，通过master节点提交
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10
~~~

- --class 表示要执行程序的主类
-  --master spark://hadoop100:7077 独立部署模式，连接到Spark 集群
- spark-examples_2.12-3.0.0.jar 运行类所在的 jar 包
- 数字 10 表示程序的入口参数，用于设定当前应用的任务数量

执行任务时，默认采用服务器集群节点的总核数，每个节点内存 1024M。

#### 提交参数说明

在提交应用中，一般会同时一些提交参数

~~~ java
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \ [application-arguments]
~~~

| 参数                     | 解释                                                         | 可选值举例                                   |
| ------------------------ | ------------------------------------------------------------ | -------------------------------------------- |
| --class                  | Spark 程序中包含主函数的类                                   |                                              |
| --master                 | Spark 程序运行的模式(环境)                                   | 模式：local[*]、spark://linux1:7077、   Yarn |
| --executor-memory 1G     | 指定每个 executor 可用内存为 1G                              | 符合集群内存配置即可，具体情况具体分析。     |
| --total-executor-cores 2 | 指定所有executor 使用的cpu 核数   为 2 个                    |                                              |
| --executor-cores         | 指定每个executor 使用的cpu 核数                              |                                              |
| application-jar          | 打包好的应用 jar，包含依赖。这 个 URL 在集群中全局可见。   比如 hdfs://    共享存储系统，如果是file:// path， 那么所有的节点的   path 都包含同样的 jar |                                              |
| application-arguments    | 传给 main()方法的参数                                        |                                              |

#### 配置历史服务

由于 spark-shell 停止掉后，集群监控 linux1:4040 页面就看不到历史任务的运行情况，所以开发时都配置历史服务器记录任务运行情况。

1. 修改spark-defaults.conf.template 文件名为 spark-defaults.conf

~~~ java
mv spark-defaults.conf.template spark-defaults.conf
~~~

2. 修改 spark-default.conf 文件，配置日志存储路径

~~~ java
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://hadoop100:8020/directory
~~~

注意：需要启动 hadoop 集群，HDFS 上的directory 目录需要提前存在

~~~ java
sbin/start-dfs.sh
hadoop fs -mkdir /directory
~~~

3. 修改 spark-env.sh 文件, 添加日志配置

~~~ java
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://hadoop100:8020/directory
-Dspark.history.retainedApplications=30"
~~~

- 参数 1 含义：WEB UI 访问的端口号为 18080
- 参数 2 含义：指定历史服务器日志存储路径
- 参数 3 含义：指定保存Application 历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4. 分发配置文件

~~~ java
xsync conf/
~~~

5. 重新启动集群和历史服务

~~~ java
sbin/start-all.sh
sbin/start-history-server.sh
~~~

6. 重新执行任务

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop100:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10
~~~

7. 查看历史服务器

~~~ java
http://hadoop100:18080
~~~

#### 配置高可用（ HA）

所谓的高可用是因为当前集群中的 Master 节点只有一个，所以会存在单点故障问题。所以为了解决单点故障问题，需要在集群中配置多个 Master节点，一旦处于活动状态的Master 发生故障时，由备用 Master 提供服务，保证作业可以继续执行。这里的高可用一般采用Zookeeper设置

**集群规划**

|       | Linux1                    | Linux2                    | Linux3             |
| ----- | ------------------------- | ------------------------- | ------------------ |
| Spark | Master Zookeeper   Worker | Master Zookeeper   Worker | Zookeeper   Worker |

1. 停止集群

~~~ java
sbin/stop-all.sh
~~~

2. 启动Zookeeper

~~~ java
 bin/zkServer.sh start
~~~

3. 修改spark-env.sh 文件添加如下配置

~~~ java
//注 释 如 下 内 容 ： 
#SPARK_MASTER_HOST=linux1 
#SPARK_MASTER_PORT=7077

//添加如下内容:
//#Master 监控页面默认访问端口为 8080，但是可能会和 Zookeeper 冲突，所以改成 8989，也可以自定义，访问 UI 监控页面时请注意
SPARK_MASTER_WEBUI_PORT=8989
export SPARK_DAEMON_JAVA_OPTS="

-Dspark.deploy.recoveryMode=ZOOKEEPER
-Dspark.deploy.zookeeper.url=hadoop100,hadoop101,hadoop102
-Dspark.deploy.zookeeper.dir=/spark"

~~~

4. 分发配置文件

~~~ java
xsync conf/
~~~

5. 在hadoop100节点上启动集群

~~~ java
[rzf@hadoop100 spark]$ sbin/start-all.sh 
~~~

hadoop100节点

![1614140696986](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/122458-868093.png)

6. 起 动 hadoop101 的单独 Master 节点，此时 hadoop101 节点 Master 状态处于备用状态

~~~ java
[rzf@hadoop101 spark]$ sbin/start-master.sh 
~~~

此时hadoop101节点出现备用的master

![1614140664274](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/122425-6732.png)

**备用节点ui界面**

![1614140794509](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/122743-766225.png)

7. 提交应用到集群

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop100:7077,hadoop101:7077 \
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10
~~~

8. 停止 hadoop100 的 Master 资源监控进程

~~~ java
kill -9 6790
~~~

![1614143661623](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/131422-850234.png)

9. 查看 hadoop101的 Master 资源监控Web UI，稍等一段时间后，hadoop101节点的 Master 状态提升为活动状态,可以看到，主节点出现故障后，从节点会代替主节点。

![1614143693307](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/131514-722346.png)

### Yarn 模式

独立部署（Standalone）模式由 Spark自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark 主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的Yarn 环境下 Spark是如何工作的（其实是因为在国内工作中，Yarn 使用的非常多）。

在集群模式中，用户将编译过的jar包提交给集群管理器（可能是yarn),除了执行器进程之外，集群管理器还会在集群内部的某一个工作节点上面启动驱动器进程，这意味着集群管理器负责维护所有的与spark应用相关的进程。

#### 解压文件

将 spark-3.0.0-bin-hadoop3.2.tgz 文件上传到linux 并解压缩，放置在指定位置。

~~~ java
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module cd /opt/module
mv spark-3.0.0-bin-hadoop3.2 spark-yarn
~~~

#### 修改配置文件

1. 修改 hadoop 配置文件/opt/module/hadoop/etc/hadoop/yarn-site.xml, 并且分发

~~~ java
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是 true -->
<property>
<name>yarn.nodemanager.pmem-check-enabled</name>
<value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是 true -->
<property>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>

~~~

2. 修改 conf/spark-env.sh，添加 JAVA_HOME 和YARN_CONF_DIR配置

~~~ java
export JAVA_HOME=/opt/module/jdk1.8.0_144
YARN_CONF_DIR=/opt/module/hadoop-2.7.2/etc/hadoop
~~~

3. 分发spark

~~~ java
//这里不再需要分发spark，因为yarn只是部署在一台节点上面
xsync spark-yarn
~~~

#### 启动 HDFS 以及 YARN 集群

~~~ java
//启动hdfs
sbin/start-dfs.sh
//启动yarn
sbin/start-yarn.sh 
//启动spark集群
sbin/start-all.sh 
~~~

#### 提交应用

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \   表示运行在yarn上
--deploy-mode cluster \   集群方式启动运行
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10

~~~

查看http://hadoop101:8088页面，点击History，查看历史页面

#### 配置历史服务器

1. 修改 spark-defaults.conf.template 文件名为 spark-defaults.conf

~~~ java
mv spark-defaults.conf.template  spark-defaults.conf
~~~

2. 修改 spark-default.conf 文件，配置日志存储路径

~~~ java
spark.eventLog.enabled true
spark.eventLog.dir	hdfs://hadoop100:8020/directory
~~~

注意：需要启动 hadoop 集群，HDFS 上的目录需要提前存在

3. 修改 spark-env.sh 文件, 添加日志配置

~~~ JAVA
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://hadoop100:8020/directory
-Dspark.history.retainedApplications=30"
~~~

- 参数 1 含义：WEB UI 访问的端口号为 18080，
- 参数 2 含义：指定历史服务器日志存储路径
- 参数 3 含义：指定保存Application 历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

4. 修改 spark-defaults.conf

~~~ java
spark.yarn.historyServer.address=hadoop100:18080 
spark.history.ui.port=18080
~~~

5. 启动历史服务

~~~ java
sbin/start-history-server.sh
~~~

6. 提交应用

~~~ java
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode client \
./examples/jars/spark-examples_2.12-3.0.0.jar \ 10
~~~

### K8S & Mesos 模式

Mesos 是Apache 下的开源分布式资源管理框架，它被称为是分布式系统的内核,在Twitter 得到广泛使用,管理着Twitter 超过 30,0000 台服务器上的应用部署，但是在国内，依然使用着传统的Hadoop大数据框架，所以国内使用Mesos 框架的并不多，但是原理其实都差不多，这里我们就不做过多讲解了。

![1614145560122](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/134601-250279.png)

容器化部署是目前业界很流行的一项技术，基于Docker 镜像运行能够让用户更加方便地对应用进行管理和运维。容器管理工具中最为流行的就是Kubernetes（k8s），而 Spark 也在最近的版本中支持了k8s 部署模式。

**参考资料**

~~~ java
https://spark.apache.org/docs/latest/running-on-kubernetes.html
~~~

### Windows 模式

在同学们自己学习时，每次都需要启动虚拟机，启动集群，这是一个比较繁琐的过程， 并且会占大量的系统资源，导致系统执行变慢，不仅仅影响学习效果，也影响学习进度， Spark 非常暖心地提供了可以在windows 系统下启动本地集群的方式，这样，在不使用虚拟机的情况下，也能学习 Spark 的基本使用

####  解压缩文件

将文件 spark-3.0.0-bin-hadoop3.2.tgz解压缩到无中文无空格的路径中

#### 启动本地环境

执行解压缩文件路径下 bin 目录中的 spark-shell.cmd 文件，启动 Spark 本地环境,以管理员身份运行

![1614146318288](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/135838-413691.png)

#### 命令行提交应用

在 DOS 命令行窗口中执行提交指令

~~~ java
spark-submit --class org.apache.spark.examples.SparkPi --master local[2] ../examples/jars/spark-examples_2.12-3.0.0.jar 10
~~~

![1614146450886](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/140051-508780.png)

### 部署模式对比

| 模式       | Spark 安装机器数 | 需启动的进程     | 所属者 | 应用场景 |
| ---------- | ---------------- | ---------------- | ------ | -------- |
| Local      | 1                | 无               | Spark  | 测试     |
| Standalone | 3                | Master 及 Worker | Spark  | 单独部署 |
| Yarn       | 1                | Yarn   及 HDFS   | Hadoop | 混合部署 |

**端口号**

- Spark 查看当前 Spark-shell 运行任务情况端口号：4040（计算）

- Spark Master 内部通信服务端口号：7077

- Standalone 模式下，Spark Master Web 端口号：8080（资源）

- Spark 历史服务器端口号：18080

- Hadoop YARN 任务运行情况查看端口号：8088

## Spark 运行架构

Driver 和Executor是和计算相关的组件，Master和Worker是和资源调度相关的组件，如果让**资源和计算**之间直接交互，耦合性太强，所以就添加ApplicationMaster组件，如果Driver需要申请资源，那么就找ApplicationMaster申请资源，而ApplicationMaster在向Master申请资源，这样可以解耦。

### 运行架构

Spark 框架的核心是一个计算引擎，整体来说，它采用了标准 master-slave 的结构。如下图所示，它展示了一个 Spark 执行时的基本结构。图形中的Driver 表示 master，负责管理整个集群中的作业任务调度。图形中的Executor 则是 slave，负责实际执行任务。

![1614146784770](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/24/140626-84669.png)

### 核心组件

由上图可以看出，对于 Spark 框架有两个核心组件：

#### Driver：spark驱动器

Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作。Spark驱动器是控制你应用程序的进程，他负责控制整个spark应用程序的执行并且维护着spark集群的状态，也就是维护执行器的任务和状态，他必须和集群管理器进行交互才可以获得物理资源并启动执行器，总的来说，spark驱动器只是物理机上的一个进程，负责维护集群运行的应用程序的状态。

 Driver 在Spark 作业执行时主要负责：

- 将用户程序转化为作业（job）

- 在 Executor 之间调度任务(task)

- 跟踪Executor 的执行情况

- 过UI 展示查询运行情况

实际上，我们无法准确地描述Driver 的定义，因为在整个的编程过程中没有看到任何有关Driver 的字眼。所以简单理解，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为Driver 类。

#### Executor：spark执行器

Spark Executor 是集群中工作节点（Worker）中的一个 JVM 进程，负责在 Spark 作业中运行具体任务（Task），任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有Executor 节点发生了故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点上继续运行。

spark执行器也是一个进程，他负责执行由spark驱动器分配的任务，执行器的核心功能是:完成驱动器分配的任务，运行他们，并报告结果的状态和执行的结果，每一个spark应用程序都有自己的执行器进程。

Executor 有两个核心功能：

- 负责运行组成 Spark 应用的任务，并将结果返回给驱动器进程,也就是返回给driver
- 它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算。

#### Master & Worker：集群管理器

Spark 集群的**独立部署环境**中，不需要依赖其他的资源调度框架，自身就实现了资源调度的功能，所以环境中还有其他两个核心组件：Master 和 Worker，这里的 Master 是一个进程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM(resourcemanager), 而Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对数据进行并行的处理和计算，类似于 Yarn 环境中 NM(namenode)。

#### ApplicationMaster

Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含ApplicationMaster，用于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

说的简单点就是，ResourceManager（资源）和Driver（计算）之间的解耦合靠的就是ApplicationMaster。

#### 集群管理器

spark驱动器和执行器并不是孤立存在的，集群管理器会将他们联系起来，集群管理器负责维护一组运行spark应用程序的机器，集群管理器也拥有自己的“driver驱动器“，（也就是master和worker)的抽象，核心区别是集群管理器管理的是物理机器，而不是进程，spark目前支持三种集群管理器，一个是简单的内置的独立的计算管理器，Apache Mesos和hadoop yarn集群管理器。

### 核心概念

#### Executor 与 Core

Spark Executor 是集群中运行在工作节点（Worker）中的一个 **JVM 进程**，是整个集群中的专门用于计算的节点。在提交应用中，可以提供参数指定计算节点的个数，以及对应的资源。这里的资源一般指的是工作节点 Executor 的**内存大小和使用的虚拟 CPU 核（Core）数量**。

应用程序相关启动参数如下：

| 名称              | 说明                                       |
| ----------------- | ------------------------------------------ |
| --num-executors   | 配置 Executor 的数量                       |
| --executor-memory | 配置每个 Executor 的内存大小               |
| --executor-cores  | 配置每个 Executor 的虚拟   CPU   core 数量 |

#### 并行度（Parallelism）

在分布式计算框架中一般都是多个任务同时执行，由于任务分布在不同的计算节点进行计算，所以能够真正地实现多任务并行执行，记住，这里是并行，而不是并发。这里我们将整个集群并行执行任务的数量称之为并行度。那么一个作业到底并行度是多少呢？这个取决于框架的默认配置。应用程序也可以在运行过程中动态修改。

#### 有向无环图（DAG）

![1614152317309](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/183105-780325.png)

- 大数据计算引擎框架我们根据使用方式的不同一般会分为四类：
  - 其中第一类就是Hadoop 所承载的 MapReduce,它将计算分为两个阶段，分别为 Map 阶段和 Reduce 阶段。对于上层应用来说，就不得不想方设法去拆分算法，甚至于不得不在上层应用实现多个 Job 的串联，以完成一个完整的算法，例如迭代计算。
  - 由于这样的弊端，催生了支持 DAG 框架的产生。因此，支持 DAG 的框架被划分为第二代计算引擎。如Tez 以及更上层的Oozie。这里我们不去细究各种 DAG 实现之间的区别，不过对于当时的Tez 和Oozie 来说，大多还是**批处理**的任务。
  - 接下来就是以 Spark 为代表的第三代的计算引擎。**第三代计算引擎的特点主要是 Job 内部的 DAG 支持（不跨越 Job），以及实时计算。**
- 这里所谓的有向无环图，并不是真正意义的图形，而是由Spark 程序直接映射成的数据流的高级抽象模型。简单理解就是将整个程序计算的执行过程用图形表示出来,这样更直观， 更便于理解，可以用于表示程序的拓扑结构。
- DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。
- 在spark中的作用就是规划多个作业的执行顺序，防止出现环发生死锁。

### 提交流程

所谓的提交流程，其实就是我们开发人员根据需求写的应用程序通过Spark 客户端提交给 Spark 运行环境执行计算的流程。在不同的部署环境中，这个提交过程基本相同，但是又有细微的区别，我们这里不进行详细的比较，但是因为国内工作中，将 Spark 引用部署到Yarn 环境中会更多一些，所以本课程中的提交流程是基于 Yarn 环境的。

**提交过程**

![1614152639267](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/183313-192534.png)

可以看到提交过程分成两条线，集群管理器那条线是负责申请资源，因为运行作业需要资源，执行main这一条线就是我们编写的逻辑程序，最后把我们的程序提交到executor节点上面进行执行

Spark 应用程序提交到 Yarn 环境中执行的时候，一般会有两种部署执行的方式：Client（运行在集群之外） 和 Cluster（运行在集群里面）。两种模式主要区别在于：**Driver程序的运行节点位置。**

#### Yarn Client 模式

Client 模式将用于**监控和调度的Driver 模块在客户端执行**，而不是在 Yarn 中，所以一般用于测试。

- Driver 在任务提交的本地机器上运行

- Driver 启动后会和ResourceManager 通讯申请启动ApplicationMaster

- ResourceManager 分配 container，在合适的NodeManager 上启动ApplicationMaster，负责向ResourceManager 申请 Executor 内存

- ResourceManager 接到 ApplicationMaster 的资源申请后会分配 container，然后ApplicationMaster 在资源分配指定的NodeManager 上启动 Executor 进程

- Executor 进程启动后会向Driver 反向注册，Executor 全部注册完成后Driver 开始执行main 函数
- 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个stage 生成对应的TaskSet，之后将 task 分发到各个Executor 上执行。

#### Yarn Cluster 模式

Cluster 模式将用于监控和调度的 Driver 模块启动在Yarn 集群资源中执行。一般应用于实际生产环境。

- 在 YARN Cluster 模式下，任务提交后会和ResourceManager 通讯申请启动ApplicationMaster，
- 随后ResourceManager 分配 container，在合适的 NodeManager 上启动 ApplicationMaster，此时的 ApplicationMaster 就是Driver。
- Driver 启动后向 ResourceManager 申请Executor 内存，ResourceManager 接到ApplicationMaster 的资源申请后会分配container，然后在合适的NodeManager 上启动Executor 进程
- Executor 进程启动后会向Driver 反向注册，Executor 全部注册完成后Driver 开始执行main 函数，
- 之后执行到 Action 算子时，触发一个 Job，并根据宽依赖开始划分 stage，每个stage 生成对应的TaskSet，之后将 task 分发到各个Executor 上执行。

## Spark 核心编程

Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集

- 累加器：分布式共享**只写**变量
- 广播变量：分布式共享**只读**变量

接下来我们一起看看这三大数据结构是如何在数据处理中使用的，上面的三大数据结构是Spark中的地基api操作。

一共有两种低级的API,一种是用于处理分布式数据(RDD)，另一种是用于分发和处理分布式共享变量（广播变量和累加器），sparkContext是地基API的入口，

### RDD

rdd特点：数据集，编程模型，相互之间有依赖，可以分区

#### 什么是RDD

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可**并行计算**的集合。类似于我们程序中的Task类。subTask是真正意义上的计算任务，Task算是一种数据结构，可以认为类似于spark中的RDD数据结构。

**源码**

~~~ java
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging 
~~~

**图示**

![1614214634651](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/085824-578280.png)

我们把每一种计算都封装为一个个的RDD计算单元，然后发送到不同的Executor计算节点上面进行计算，这样可以实现迭代式的计算。可以把RDD看做是Task。其实就是一种来准备计算逻辑的一种数据结构，一个大的计算，可以分解为多个RDD计算组合。

![1614219619509](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091635-340950.png)

RDD中还包括一个分区的概念，分区的目的就是为了把数据划分为小的数据集，可以分到多个task当中，提高task的并行度。

#### 为什么需要RDD

![1616723127821](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/094529-208892.png)

#### IO类比

io操作体现装饰着模式

**使用FileInputStream**

![1614215428911](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091638-485155.png)

如果只是用简单的TileInputStream输入流，那么是一个一个字节读取，每读取到一个字符，就会输出到控制台，这样效率很低，但是如果使用缓冲流，先把读取到的数据缓存到一个缓缓区中，等待到缓冲区满后，全部输出，类似spark中的批处理，效率高很多。批处理比一个一个执行性能高很多。

**使用BufferedInputStream**

![1614215562161](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/091244-732780.png)

**InputStreamReader**

![1614216292488](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091642-298330.png)

![1614216305685](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/26/123209-531147.png)

从文件中读取吧的时候是以字节流形式读取，然后使用InputStreamReader流转换为字节流，最后使用BufferedReader流缓冲到缓冲区后，最后全部输出，上述的一系列流操作，真正读取的是FileInputStream流在底层读取，而InputStreamReader和BufferedReader仅仅是做装饰工作，是对底层的封装，InputStreamReader负责吧字节流转换为字符流，比如3个字节组成一个汉字，那么就等到三个字节后，InputStreamReader就把这三个字节转换为一个汉字，然后交给BufferedReader存放在缓冲区，等到缓冲区快满的时候打印，上述过程中，最重要的方法是readLine()函数，应为此函数会触发读取操作，在建立流的过程中只是建立一套传输系统，不会真正的读取数据，只有真正readLine()读取数据的时候，才会触发读取操作，可以认为是一个懒加载操作。RDD也是我们的一个最小计算单元，以后的操作都是在RDD上封装叠加操作。

#### RDD原理

![1614217505781](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/094507-970505.png)

![1614217563861](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/091647-495924.png)

**小结**

RDD数据的处理方式类似于IO流，也有装饰者设计模式，但是也有不同，io中真正new对象时候不会触发文件数据的读取，RDD数据只有在调用collect的时候才会真正执行逻辑操作，之前的封装都是对功能的扩展，RDD他是不保存数据，也就是没有缓冲区，但是IO流有缓冲区，会临时保存数据。RDD其实就是通过功能的组合，最终完成一个复杂的功能计算。

**源码角度理解**

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

- **弹性**（可变）
  - 存储的弹性：内存与磁盘的自动切换；比mr计算模型效率高
  - 容错的弹性：数据丢失可以自动恢复；
  - 计算的弹性：计算出错重试机制；计算出错后，可以重头重新计算
  - 分片的弹性：可根据需要重新分片。分区操作，
- 分布式：数据存储在大数据集群不同节点上
- 数据集：RDD 封装了计算逻辑，并不保存数据，也就是封装的是对数据的计算操作步骤，不会像io一样会存储数据。
- 数据抽象：RDD 是一个抽象类，需要子类具体实现
- 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD 里面封装计算逻辑，也就是一个计算封装为RDD后，以后不可以对这个封装好的RDD在进行扩展，只能重新创建一个RDD对前面的RDD逻辑进行封装和装饰。
- 可分区、并行计算

#### 核心属性

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

##### 分区列表

RDD 数据结构中存在分区列表，用于执行任务时并行计算，**是实现分布式计算的重要属性**,分区之间的数据没有相互关系，把数据分为多个分区，然后封装为Task在不同的节点上面执行。

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

Spark在计算时，是使用分区函数对每一个分区进行计算，即计算每一个分区,计算分区的逻辑是完全一样的。

~~~ java
/**
   * :: DeveloperApi ::
   * Implemented by subclasses to compute a given partition.
   */
  @DeveloperApi
  def compute(split: Partition, context: TaskContext): Iterator[T]
~~~

##### RDD之间的依赖关系

RDD 是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD 建立依赖关系。

~~~ java
 /**
   * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   */
  protected def getDependencies: Seq[Dependency[_]] = deps
~~~

##### 分区器（可选）

当数据为 KV 类型数据时，可以通过设定分区器自定义数据的分区,读取数据的时候需要分区，分区器就是定义如何分区数据的规则。

~~~ java
/** Optionally overridden by subclasses to specify how they are partitioned. */
  @transient val partitioner: Option[Partitioner] = None

~~~

##### 首选位置（可选）

计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算，也就是决定每一个Task分发给哪一个节点进行执行,效率最优，（移动数据不如计算）

~~~ java
/**
   * Optionally overridden by subclasses to specify placement preferences.
   */
  protected def getPreferredLocations(split: Partition): Seq[String] = Nil
~~~

**图示**

![1614221512980](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/184038-830467.png)

移动数据需要消耗网络传输资源。

#### 执行原理

从计算的角度来讲，数据处理过程中需要计算资源（内存 & CPU）和计算模型（逻辑）。执行时，需要将计算资源和计算模型进行协调和整合。

Spark 框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的计算任务。然后将任务发到已经分配资源的计算节点上,  按照指定的计算模型进行数据计算。最后得到计算结果。

RDD 是 Spark 框架中用于数据处理的核心模型，接下来我们看看，在 Yarn 环境中，RDD 的工作原理:

1. 启动 Yarn 集群环境

![1614221773058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/27/184107-901473.png)

2. Spark 通过申请资源创建调度节点和计算节点（driver和Executor）

   driver，Executor都是运行在某一个NodeManager节点上面的。resourcemanager是负责管理整个所有的nodemanager节点的，真正工作的节点是nodemanager节点

   ![1614221855307](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/105736-882674.png)

3. Spark 框架根据需求将计算逻辑根据分区划分成不同的任务

   可以看到，任务的数量取决与分区的个数，driver负责调度RDD，多个RDD计算逻辑进行关联，然后被分解为多个task任务，然后把任务放进任务池中。

![1614222023617](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/110025-291725.png)

4. 调度节点将任务根据计算节点状态发送到对应的计算节点进行计算

![1614222280933](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/25/110442-417648.png)

从以上流程可以看出 RDD在整个流程中主要用于将计算逻辑进行封装，并生成Task 发送给Executor节点执行计算，接下来我们就一起看看Spark 框架中RDD 是具体是如何进行数据处理的。

#### 基础编程

##### RDD对象的创建

RDD中的数据可以来源于2个地方：本地集合或外部数据源

![1616723294144](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/094826-620596.png)

在 Spark 中创建RDD 的创建方式可以分为四种：

###### 从集合（内存）中创建 RDD

从集合中创建RDD，`[Spark ](https://www.iteblog.com/archives/tag/spark/)`主要提供了两个方法：parallelize 和 makeRDD

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

由外部存储系统的数据集创建RDD 包括：本地的文件系统，所有Hadoop 支持的数据集，比如HDFS、HBase 等

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
		//path还可以是分布式系统文件的路径，比如hdfs文件系统路径
		rdd.foreach(println)
      
		//	关闭环境
		context.stop()
	}
}
~~~

**判断结果来自哪一个文件**

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

主要是通过一个RDD 运算完后，再产生新的RDD。详情请参考后续章节

###### 直接创建 RDD（new）

使用 new 的方式直接构造RDD，一般由Spark 框架自身使用。

#####  RDD 并行度与分区

默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建RDD 时指定。记住，**这里的并行执行的任务数量，并不是指的切分任务的数量**，不要混淆了。不要和读取数据的分片数记错

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

##### **rdd特点**

- rdd不仅是数据集，而且还是编程模型

![1616723901576](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/080724-389059.png)

- RDD可以进行分区，因为spark是一个分布式的并行计算框架，只有进行分区操作后，才可以进行并行计算

![1616723956725](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/101605-430662.png) 

- RDD是只读的

![1616723714723](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/095552-537657.png)

- RDD是可以容错的

![1616723782278](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/095920-102806.png)

##### 什么是弹性分布式数据集

![1616725004784](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/102723-7165.png)

##### RDD属性小结

![1616725674853](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616725674853.png)

#### RDD 转换算子

RDD 根据数据处理方式的不同将算子整体上分为Value 类型、双 Value 类型和Key-Value类型

##### **Value** **类型**                                       
###### map

map的转换为一对一的转换。

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

###### mapValue

mapValue只对键值类型的值进行转换，不会改变键

**函数签名**

~~~ java
 def mapValues[U](f: V => U): RDD[(K, U)] 
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

- mapPartitions 操作是在内存中进行操作，所以性能比较高，并且mapPartitions 方法会把一个分区中的数据全部加载完成之后全部进行处理，所以效率比map函数高很多，而map是对每一条数据进行操作。
- mapPartitions 可以以分区为单位进行数据的转换操作，会将整个分区中的数据加载到内存中引用，但是使用完的数据不会被释放掉，当数据量比较多，内存较小的情况下，容易出现内存溢出，这种情况使用map函数比较好。
- 有时候可以以分区为单位对数据进行访问操作，减少内存的开销，比如访问数据库中的数据，如果对单条数据进行访问，那么每一条数据都要建立一个链接，但是如果对数据进行分区操作，那么每一个分区建立一个链接即可。

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

~~~

> mapPartitions和mapPartitionsWithIndex最大的区别是mapPartitionsWithIndex的参数中多出一个分区号，可以针对指定的分区进行操作。

###### flatmap

flatmap是一对多的转换算子。

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

![1614480400951](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614480400951.png)

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

**函数签名**

~~~ java
def filter(f: T => Boolean): RDD[T]
~~~

**函数说明**

- 将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。
- 当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。 

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

**函数签名**

~~~ java
def sample(
withReplacement: Boolean,
fraction: Double,
seed: Long = Utils.random.nextLong): RDD[T]
~~~

**函数说明**

根据指定的规则从数据集中抽取数据 ，把大数据集变为小数据集，尽量减少数据特征信息的损失。抽样过程是随机的。

~~~ java
val dataRDD = sparkContext.makeRDD(List(
1,2,3,4
),1)
// 抽取数据不放回（伯努利算法）
// 伯努利算法：又叫 0、 1 分布。例如扔硬币，要么正面，要么反面。
// 具体实现：根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不
要
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

###### sortBy 

可以作用于任何数据类型的RDD，可以按照任何部分进行排序，需要和sortByKey区分开，sortByKey只能作用于k-v类型数据。

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

![1614760002534](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614760002534.png)

可以看到还有另外一个Range分区器，这个分区器是在排序的时候使用。

- 如果想按照自己的方法进行数据分区怎么办？ 
  - 可以自定义分区器，见后文。

###### reduceByKey 

此算子还是一个分组聚合操作，不能做分组操作

reduceByKey的底层是combineByKey

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
		val value = rdd.reduceByKey((num1: Int, num2: Int) => {

			println(s"x=${num1},y=${num2}")
			num1 + num2
		})

		value.collect().foreach(println)
		context.stop()
	}
}

~~~

###### groupByKey 

这个算子本质上是一个shuffer操作

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

![1614765569116](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614765569116.png)

![1614765577376](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/03/180051-323968.png)

reduceByKey性能优于GroupByKeyDE的原因就在于reduceByKey在自己的分区内做了预聚合操作，使得io到磁盘上的数据量大大减小，这个功能叫做conbine操作。

- 从功能角度讲，reduceByKey是对数据两两进行聚合操作，如果不需要对数据进行聚合操作，只需要做分组，那么就使用groupByKey操作，两个的功能有所区别。

**小结**

- 从 shuffle 的角度： reduceByKey 和 groupByKey 都存在 shuffle 的操作，但是 reduceByKey可以在 shuffle 前对分区内相同 key 的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而 groupByKey 只是进行分组，不存在数据量减少的问题， reduceByKey 性能比较高。
- 从功能的角度： reduceByKey 其实包含分组和聚合的功能。 GroupByKey 只能分组，不能聚合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那么还是只能使用 groupByKey 
- 预聚合是发生在一个分区内，也就是在同一个分区内先做一次聚合处理，而落盘是在多个分区间进行的聚合操作，数据来自不同的分区。reduceByKey 在分区内和分区之间的聚合规则是一样的。

###### aggregateByKey 

![1616840367035](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/181929-532424.png)

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

![1614818625333](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614818625333.png)

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

![1614821215557](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614821215557.png)

![1614821226136](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614821226136.png)

![1614821232927](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/04/101148-586705.png)

###### foldByKey 

foldByKey和reduceByKey的区别是前者可以指定一个初始值

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

![1616834063840](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616834063840.png)

**代码案例**

~~~ java
object test {
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
			createCombiner函数是转换函数，但是仅仅作用于第一条数据，做一个初始化操作
			mergeValue作用在分区上面，对分区做聚合操作
			mergeCombiners对最终所有的分区做聚合操作
		 */

		val combineRes: Any = RddData.combineByKey(
			createCombiner = (curr: Double) => (curr, 1),
			mergeValue =
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


		context.stop()

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

##### 获取键值集合

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

##### reduce

**函数签名**

~~~ java
def reduce(f: (T, T) => T): T
~~~

**函数说明**

聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据 ，最终输出的是一个结果，这就是所谓的聚合操作。指定一个函数将RDD中的任何类型的值规约为一个值。

~~~ java
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 聚合数据，求和操作
val reduceResult: Int = rdd.reduce(_+_)
~~~

##### collect

**函数签名**

~~~ java
def collect(): Array[T]
~~~

**函数说明**

在驱动程序中，以数组 Array 的形式返回数据集的所有元素 

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

##### countByKey 

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

![1614851041159](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614851041159.png)

collect会先把各个节点上的数据全部收集回来，然后在driver端的内存中进行打印操作

![1614851105867](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614851105867.png)

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

#### RDD 序列化 

**闭包检查**

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行。 那么在 scala 的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给 Executor端执行，就会发生错误，所以需要在执行任务计算前，检测闭包内的对象是否可以进行序列化，这个操作我们称之为闭包检测。 Scala2.12 版本后闭包编译方式发生了改变 

**序列化方法和属性 **

从计算的角度, 算子以外的代码都是在 Driver 端执行, 算子里面的代码都是在 Executor端执行 

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

**Kryo 序列化框架 **

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

#### RDD 依赖关系 

##### RDD血缘关系

RDD 只支持粗粒度转换，即在大量记录上执行的单个操作。将创建 RDD 的一系列 Lineage（血统）记录下来，以便恢复丢失的分区。 RDD 的 Lineage 会记录 RDD 的元数据信息和转换行为，当该 RDD 的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。 

![1614933707058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/164149-246145.png)

![1614934144895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/164908-814529.png)

![1614934303590](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614934303590.png)

黄色部分就表示保存的血缘关系。

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

这种情况下，新的RDD中分区中的数据来自对应的老的RDD中分区中的数据，存在一对一的关系。

**源码角度**

~~~ java
//窄依赖
@DeveloperApi
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd) {
  override def getParents(partitionId: Int): List[Int] = List(partitionId)
}
~~~

##### RDD 宽依赖 

宽依赖表示同一个父（上游） RDD 的 Partition 被多个子（下游） RDD 的 Partition 依赖，会
引起 Shuffle，总结：宽依赖我们形象的比喻为多生。 

**shuffer 依赖**也叫做宽依赖

![1614935603198](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/171401-695375.png)

这种依赖的话会把老的RDD中的数据全部打乱重新分配给新的RDD，所以叫做shuffle依赖

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

如果使用宽依赖，那么任务的数量就会增加，因为数据全部被打散，

![1614936854674](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173416-455965.png)

宽依赖使任务量增加，所以就要划分阶段执行任务，不同分区的数据到达的时间不一样，所以就会存在等待的关系，所以需要划分阶段执行，保证不同阶段里面的task全部执行完毕，才可以执行下一个阶段。

![1614937061280](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173836-818567.png)

窄依赖不会使任务的数量增加，所以不需要划分阶段执行任务

![1614936891785](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/05/173453-172770.png) 

##### RDD阶段划分

96

DAG（Directed Acyclic Graph）有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如， DAG 记录了 RDD 的转换过程和任务的阶段 



##### RDD 任务划分 

98

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

- Stage： Stage 等于宽依赖(ShuffleDependency)的个数加 1； 也就是shuffle依赖的数量+resultStage，最终阶段
- Task：一个 Stage 阶段中，最后一个 RDD 的分区个数就是 Task 的个数。也就是说任务的数量等于最后一个阶段中分区的数量。 

> 注意： Application->Job->Stage->Task 每一层都是 1 对 n 的关系。 
>
> - 一个应用程序中如果有多个行动算子，那么就是说有多个job，
> - 一个job中如果有shuffle依赖，那么一个job中就右多个阶段
> - 而一个阶段中可能会有多个分区，所以就会产生多个TASK,极限情况下有一个分区，产生一个task

#### RDD 持久化 

##### 使用缓存的意义

**可以减少shuffle操作次数**

![1616988151858](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616988151858.png)

**容错**

![1616988200887](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616988200887.png)

可以对中间的计算结果进行缓存操作，后面如果出现失败时候可以直接从缓存点获取计算数据，重新计算。

##### 问题引出

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

**改进**

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

**图解**

也就是说把map算子处的数据使用缓存进行缓存处理，然后后面的算子可以直接使用数据，不需要在进行前面算子的转换操作，可以有效减少shufffle操作次数，提高效率。多个作业之间可以共用数据。

![1614945775874](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/110430-490727.png)

**改进**

添加持久化操作，数据可以存放到内存当中，也可以存放到磁盘当中

![1614945944979](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1614945944979.png)

RDD持久化操作不一定是为了重用操作，在数据执行时间较长或者比较重要的场合也用持久化操作。

##### RDD Cache 缓存 

RDD 通过 Cache （也是一个算子操作）或者 Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存在 JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该 RDD 将会被缓存在计算节点的内存中，并供后面重用。 

~~~ java
// cache 操作会增加血缘关系，不改变原有的血缘关系
println(wordToOneRdd.toDebugString)
// 数据缓存。
wordToOneRdd.cache()
// 可以更改存储级别
//mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
~~~

**存储级别 **

![1616989064097](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/113906-955896.png)

![1616989352669](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/114238-64154.png)

是否以反序列化的形式进行存储，如果是，那么存储的就是一个对象，如果不是，那么存储的就是一个序列化过的值。必粗要序列化之后对象才可以存储在磁盘中，如果deserialized是true的话存储的就是一个对象，如果是false的话存储的就是二进制数据。

![1616989874428](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616989874428.png)

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

缓存有可能丢失，或者存储于内存的数据由于内存不足而被删除， RDD 的缓存容错机制保证了即使缓存丢失也能保证计算的正确执行。通过基于 RDD 的一系列转换，丢失的数据会被重算，由于 RDD 的各个 Partition 是相对独立的，因此只需要计算丢失的部分即可，并不需要重算全部 Partition。 

Spark 会自动对一些 Shuffle 操作的中间数据做持久化操作(比如： reduceByKey)。这样做的目的是为了当一个节点 Shuffle 失败了避免重新计算整个输入。但是，在实际使用的时候，如果想重用数据，仍然建议调用 persist 或 cache。 

##### RDD CheckPoint 检查点 

所谓的检查点其实就是通过将 RDD 中间结果写入磁盘,由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题， 可以从检查点开始重做血缘，减少了开销。对 RDD 进行 checkpoint 操作并不会马上被执行，必须执行 Action 操作才能触发。 

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

##### 缓存和检查点区别 

**检查点作用**

将数据checkpoint的情况非常少，一般都是缓存在hdfs上面保存。

![1616990352556](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616990352556.png)

- cache:将数据临时存储在内存中重用，会在血缘关系中添加新的依赖，一旦出现问题，可以从头读取数据
- persist将数据临时存放在磁盘中进行数据重用，但是涉及到磁盘io操作，性能低，但是数据安全，如果作业执行完毕，临时保存的数据的数据文件就会消失
- checkpoint可以将长久的保存在磁盘中进行重用，设计磁盘的io操作，效率比较低，但是数据安全，为了保证数据的安全，所以一般情况下，会独立执行作业(里面有触发作业执行的操作)，也就是单独在执行一遍作业，为了提高效率，所以一般和cache联合使用，先进行缓存操作，然后checkpoint()方法里面就不会触发作业的执行。

~~~ java
mapRdd.cache()
mapRdd.checkpoint()
~~~

- checkpoint在执行过程中会切断血缘关系，重新建立新的血缘关系，checkpoint等同于改变我们的数据源

> Cache 缓存只是将数据保存起来，不切断血缘依赖。 Checkpoint 检查点切断血缘依赖。 
>
> Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。 Checkpoint 的数据通常存储在 HDFS 等容错、高可用的文件系统，可靠性高。 
>
> 建议对 checkpoint()的 RDD 使用 Cache 缓存，这样 checkpoint 的 job 只需从 Cache 缓存中读取数据即可，否则需要再从头计算一次 RDD。 

#### RDD 分区器 

Spark 目前支持 Hash 分区和 Range 分区，和用户自定义分区。 Hash 分区为当前的默认分区。分区器直接决定了 RDD 中分区的个数、 RDD 中每条数据经过 Shuffle 后进入哪个分区，进而决定了 Reduce 的个数。 

- 只有 Key-Value 类型的 RDD 才有分区器，非 Key-Value 类型的 RDD 分区的值是 None 
- 每个 RDD 的分区 ID 范围： 0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。 

##### Hash 分区：对于给定的 key，计算其 hashCode,并除以分区个数取余 

~~~ java
class HashPartitioner(partitions: Int) extends Partitioner {
			require(partitions >= 0, s"Number of partitions ($partitions) cannot be
				negative.")
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

Range 分区：将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而且分区间有序 

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

**sequence 文件 **

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

**object 对象文件 **

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

#### RDD的分区和Shuffle

RDD分区的作用

- RDD通常需要读取外部系统的数据文件进行创建RDD，外部存储系统往往支持分片操作，分片侧重于存储，分区侧重于计算，所以RDD需要支持分区来和外部系统的分片进行一一对应，
- RDD是一个并行计算的实现手段

![1616978719222](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616978719222.png)

##### 查看分区的方式

1.  通过webUi方式进行查看
2. 通过partitions.size方法进行查看

##### 指定RDD分区个数的方式

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

##### 重定义分区数

coalesce()这个方法默认是不进行shuffle操作的，所以也就限制了在分区的时候只能把分区的个数变小，不能增大分区的个数，如果想要增大分区的个数，必须把shuffle操作设置为true,每一次调用这个方法都会生成新的rdd，改变分区也是在新的RDD上面改变分区的个数，不会再旧的分区上面改变RDD分区的个数。

repartition():这个方法不仅可以增加分区个数，还可以减少分区的个数，只有一个参数可以设置。在repartition的底层，仍然使用的是coalease()方法，并且shuffle永远指定为true.

##### 通过其他算子指定分区个数

reduceByKey():此方法有一个参数可以指定分区的个数，含义是指定新生成的RDD生成的分区的个数。

groupByKey:也可以指定分区的个数

joink（）：此方法也可以指定分区的个数，很多方法都可以指定分区的个数。

hen多方法都有重载的方法，可以重新指定分区的个数。一般涉及shuffle操作的方法都可以重新指定分区的个数。如果没有指定分区的个数，那么就会从父级的RDD中继承分区个数。

**分区函数**

![1616981407434](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616981407434.png)

默认使用的是Hash算法进行分区操作，我们也可以重写partitoner函数自定义分区操作

**分区接口**

~~~ java
abstract class Partitioner extends Serializable {
  def numPartitions: Int
  def getPartition(key: Any): Int
}
~~~

##### RDD中的shuffle过程

![1616985508729](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616985508729.png)

HashBase nshuffle会在nap端把准备发往reducer端的数据进行份文件存储，然后reducer端可以根据自己分区的数据区各个map端输出额文件中拉取数据，但是这种方式的效率非常的底下，比如有1000个map和1000个reducer，那么中间会生成1000*1000g个临时的文件，所以非常占用资源

mapreduce没有使用hash base shuffle，但是spark RDD使用的是hash base shuffle

**第二种**

![1616986258409](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616986258409.png)

对于每一个map需要输出的数据，不在根据reducer进行划分然后输出到磁盘文件存储，而是把map中的所有数据首先按照partition id进行排序操作，然后按照key的哈希码进行排序操作，但是所有的数据全部存储在一个集合中，可以想象为mr中的环形缓冲区一样，然后把数据分发到各个reducer端

这种方法可以明显解决临时文件过多的问题

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

## 小项目

### 数据准备

![1615089633115](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/07/120035-673350.png)

上面的数据图是从数据文件中截取的一部分内容，表示为电商网站的用户行为数据，主要包含用户的 4 种行为： 搜索，点击，下单，支付。 数据规则如下： 

- 数据文件中每行数据采用下划线分隔数据 
- 每一行数据表示用户的一次行为，这个行为只能是 4 种行为的一种 
- 如果搜索关键字为 null,表示数据不是搜索数据 
- 如果点击的品类 ID 和产品 ID 为-1，表示数据不是点击数据 
- 针对于下单行为，一次可以下单多个商品，所以品类 ID 和产品 ID 可以是多个， id 之间采用逗号分隔，如果本次不是下单行为，则数据采用 null 表示 
- 支付行为和下单行为类似 

### 字段说明

![1615089768383](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/07/120250-642379.png)

![1615089787199](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/07/121346-398272.png)

**样例类**

~~~ java
//用户访问动作表
	case class UserVisitAction(
															date: String,//用户点击行为的日期
															user_id: Long,// 用 户 的 ID
															session_id: String,//Session 的 ID
															page_id: Long,// 某 个 页 面 的 ID
															action_time: String,//动作的时间点
															search_keyword: String,//用户搜索的关键词
															click_category_id: Long,// 某 一 个 商 品 品 类 的 ID
															click_product_id: Long,// 某 一 个 商 品 的 ID
															order_category_ids: String,//一次订单中所有品类的 ID 集合
															order_product_ids: String,//一次订单中所有商品的 ID 集合
															pay_category_ids: String,//一次支付中所有品类的 ID 集合
															pay_product_ids: String,//一次支付中所有商品的 ID 集合
															city_id: Long
														)//城市 id
~~~

### 需求一：

Top10 热门品类 

#### 需求说明

品类是指产品的分类，大型电商网站品类分多级，咱们的项目中品类只有一级，不同的公司可能对热门的定义不一样。我们按照每个品类的点击、下单、支付的量来统计热门品类。 

~~~ java
鞋 点击数 下单数 支付数
衣服 点击数 下单数 支付数
电脑 点击数 下单数 支付数
例如，综合排名 = 点击数*20%+下单数*30%+支付数*50%
本项目需求优化为： 先按照点击数排名，靠前的就排名高；如果点击数相同，再比较下单数；下单数再相同，就比较支付数。
~~~

#### 实现一

分别统计每个品类点击的次数，下单的次数和支付的次数：
（品类，点击总数）（品类，下单总数）（品类，支付总数） 





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

