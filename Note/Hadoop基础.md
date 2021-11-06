# Hadoop基础

<!-- TOC -->

- [Hadoop基础](#hadoop基础)
  - [Hadoop基础概述](#hadoop基础概述)
    - [Hadoop是什么](#hadoop是什么)
    - [Hadoop发展史](#hadoop发展史)
    - [Hadoop的三大发行版本](#hadoop的三大发行版本)
    - [Hadoop的四大优势](#hadoop的四大优势)
    - [Hadoop的组成(重点)](#hadoop的组成重点)
    - [大数据生态体系](#大数据生态体系)
    - [推荐系统框架图](#推荐系统框架图)
  - [Hadoop运行环境搭建](#hadoop运行环境搭建)
    - [虚拟机环境准备](#虚拟机环境准备)
    - [安装JDk](#安装jdk)
  - [JAVA_HOME](#java_home)
    - [安装Hadoop](#安装hadoop)
  - [HADOOP_HOME](#hadoop_home)
    - [Hadoop的目录结构](#hadoop的目录结构)
  - [Hadoop运行模式](#hadoop运行模式)
    - [本地运行模式](#本地运行模式)
      - [官方WordCount案例](#官方wordcount案例)
    - [伪分布模式](#伪分布模式)
      - [启动HDFS并运行MapReduce程序](#启动hdfs并运行mapreduce程序)
      - [启动YARN并运行MapReduce程序](#启动yarn并运行mapreduce程序)
      - [配置历史服务器](#配置历史服务器)
      - [配置日志的聚集](#配置日志的聚集)
      - [配置文件说明](#配置文件说明)
    - [完全分布式模式](#完全分布式模式)
      - [准备工作](#准备工作)
      - [编写集群分发脚本xsync](#编写集群分发脚本xsync)
      - [集群配置](#集群配置)
      - [集群的单点启动](#集群的单点启动)
      - [ssh无密码登录设置](#ssh无密码登录设置)
      - [群起集群](#群起集群)
      - [集群的基本测试](#集群的基本测试)
      - [集群启动/停止方式总结](#集群启动停止方式总结)

<!-- /TOC -->

## Hadoop基础概述

### Hadoop是什么

1. `Hadoop`是一个`Apache`基金会开发的分布式系统的架构。
2. 主要解决海量数据的存储和分析计算问题。
3. 广义上说，`Hadoop`通常指`Hadoop`生态圈。

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190414-589264.png)

### Hadoop发展史

`Google`是`Hadoop`的思想来源

- `GFS—>HDFS`

- `Map Reduce—>MR`

- `Big Table—>HBase`
### Hadoop的三大发行版本

`Hadoop`三大发行版本：`Apache`、`Cloudera`、`Hortonworks`。

1. `Apache`版本最原始（最基础）的版本，对于入门学习最好。`apache`版。

   官网地址：http://hadoop.apache.org/releases.html

   下载地址：https://archive.apache.org/dist/hadoop/common/

2. `Cloudera`在大型互联网企业中用的较多,收费。`CDH`版。

   官网地址：https://www.cloudera.com/downloads/cdh/5-10-0.html

   下载地址：http://archive-primary.cloudera.com/cdh5/cdh/5/

3. `Hortonworks`文档较好。

   官网地址：https://hortonworks.com/products/data-center/hdp/

   下载地址：https://hortonworks.com/downloads/#data-platform

### Hadoop的四大优势

1. 高可靠性，`hadoop`底层维护了多个数据的副本，所以即使`hadoop`某个节点的数据出现故障，也不会导致数据的丢失。
2. 高扩展性：在集群之间分配任务，可以很方便扩展大量的节点。‘
3. 高效性：在`mapreducer`的思想下，`hadoop`是==并行==工作的，加快了数据的处理速度。
4. 高容错性：能够自行将失败的任务重新分配。

### Hadoop的组成(重点)

- `hadoop1`和`hadoop2`的不同之处：

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190637-143979.png)

1. `Hdfs`介绍：
   1. `namenode`:存储文件的元数据，如文件名，文件属性（文件生成时间，权限等），文件目录结构，以及每个文件块信息和文件块所在的`datenode`地址。
   2. `Datenode`:在本地文件系统存储文件块的数据，以及数据块的校验和。
   3. `Secondary Namenode`:用来监控`HDFS`状态的辅助后台程序，每隔一段时间获取`hdfs`的快照。
2. `Yarn`架构概述：

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190640-177397.png)

- `ResourceManager`负责分配服务器的整个资源

- `ApplicationMaster`是一个一个的job

- `NodeManager`单个节点上的老大，管理单个节点上面的资源

- `Container`封装一个节点上面的资源，为某一个节点上的`ApplicationMaster`提供资源，可以理解为`vmare`。

3. `MapReduce`架构概述

   `MapReduce`将计算过程分为两个阶段：`Map`和`Reduce`，如图所示

   - `Map`阶段并行处理输入数据
   - `Reduce`阶段对`Map`结果进行汇总

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190642-430972.png)

### 大数据生态体系

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190644-192964.png)

1. `Sqoop`：`Sqoop`是一款开源的工具，主要用于在`Hadoop`、`Hive`与传统的数据库(`MySql`)间进行数据的传递，可以将一个关系型数据库（例如 ：`MySQL`，`Oracle` 等）中的数据导进到`Hadoop`的`HDFS`中，也可以将`HDFS`的数据导进到关系型数据库中。

2. `Flume`：`Flume`是`Cloudera`提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，`Flume`支持在日志系统中定制各类数据发送方，用于收集数据；同时，`Flume`提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

3. `Kafka`：`Kafka`是一种高吞吐量的分布式发布订阅消息系统，有如下特性：

   1. 通过`O(1)`的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。

   2. 高吞吐量：即使是非常普通的硬件`Kafka`也可以支持每秒数百万的消息。

   3. 支持通过`Kafka`服务器和消费机集群来分区消息。

   4. 支持`Hadoop`并行数据加载。

4. `Storm`：`Storm`用于“连续计算”，对数据流做连续查询，在计算时就将结果以流的形式输出给用户。

5. `Spark`：`Spark`是当前最流行的开源大数据内存计算框架。可以基于`Hadoop`上存储的大数据进行计算。

6. `Oozie`：`Oozie`是一个管理`Hdoop`作业（`job`）的工作流程调度管理系统。

7. `Hbase`：`HBase`是一个分布式的、面向列的开源数据库。`HBase`不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。

8. `Hive`：`Hive`是基于`Hadoop`的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的`SQL`查询功能，可以将`SQL`语句转换为`MapReduce`任务进行运行。 其优点是学习成本低，可以通过类`SQL`语句快速实现简单的`MapReduce`统计，不必开发专门的`MapReduce`应用，十分适合数据仓库的统计分析。

9. `R`语言：`R`是用于统计分析、绘图的语言和操作环境。`R`是属于`GNU`系统的一个自由、免费、源代码开放的软件，它是一个用于统计计算和统计制图的优秀工具。

10. `Mahout`：`Apache Mahout`是个可扩展的机器学习和数据挖掘库。

11. `ZooKeeper`：`Zookeeper`是`Google`的`Chubby`一个开源的实现。它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、 分布式同步、组服务等。`ZooKeeper`的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

### 推荐系统框架图

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/12/190647-128896.png)

## Hadoop运行环境搭建

### 虚拟机环境准备

如果使用的是虚拟机，那么首先需要完成下面的步骤

1. 克隆虚拟机                                                       
2. 修改克隆虚拟机的静态IP
3. 修改主机名
4. 关闭防火墙
5. 创建用户
6. 配置用户具有root权限
7. 同时需要在`windows`系统的`/etc/hosts`文件中添加虚拟机的`ip`地址和主机名形成映射。
8. 在`/opt`目录下创建文件夹,用于安装我们的软件。

~~~ java
//创建成功后文件夹的所有者和所有者的组都是root
[rzf@VM-8-10-centos opt]$ sudo mkdir module
[rzf@VM-8-10-centos opt]$ sudo mkdir software
//sudo rm -rf wdsx/   删除文件夹指令
~~~

9. 修改文件夹所属的主和组

~~~ java
[rzf@VM-8-10-centos opt]$ sudo chown rzf:group software/ module/
//修改文件所属的主是rzf,所属的组是group
[rzf@VM-8-10-centos opt]$ ll
total 16
drwxr-xr-x  4 root root  4096 Aug  5 16:11 mellanox
drwxr-xr-x  2 rzf  group 4096 Dec 13 13:55 module
drwxr-xr-x. 2 root root  4096 Oct 31  2018 rh
drwxr-xr-x  2 rzf  group 4096 Dec 13 13:52 software
~~~

10. 把`JDK`和编译过的`hadoop`包拷贝到`software`目录下面。

### 安装JDk

~~~ java
//解压jdk包到module文件夹
[rzf@VM-8-10-centos software]$ tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt/module/
//获取jdk的安装路径，配置环境变量
[rzf@VM-8-10-centos jdk1.8.0_144]$ pwd
/opt/module/jdk1.8.0_144
//开始配置环境变量
[rzf@VM-8-10-centos jdk1.8.0_144]$ sudo vim /etc/profile
//配置下面的环境变量
## JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
//别忘记source
[rzf@VM-8-10-centos jdk1.8.0_144]$ source /etc/profile
//测试一下
[rzf@VM-8-10-centos jdk1.8.0_144]$ java -version
//注意：重启（如果java -version可以用就不用重启）
sync
sudo reboot
~~~

### 安装Hadoop

~~~ java
//解压hadoop到modele目录
[rzf@VM-8-10-centos software]$ tar -zxvf hadoop-2.7.2.tar.gz -C /opt/module/ 
//获取安装路径，配置环境变量
/opt/module/hadoop-2.7.2
//打开vim /etc/profile配置环境变量，添加下面内容
## HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/sbin
//最后进行source /etc/profile
//验证
[rzf@VM-8-10-centos root]$ hadoop
~~~

### Hadoop的目录结构

~~~ java
[rzf@VM-8-10-centos hadoop-2.7.2]$ ll
total 52
drwxr-xr-x 2 rzf group  4096 May 22  2017 bin //可执行的命令
drwxr-xr-x 3 rzf group  4096 May 22  2017 etc //存放配置文件信息
drwxr-xr-x 2 rzf group  4096 May 22  2017 include
drwxr-xr-x 3 rzf group  4096 May 22  2017 lib //本地库文件
drwxr-xr-x 2 rzf group  4096 May 22  2017 libexec
-rw-r--r-- 1 rzf group 15429 May 22  2017 LICENSE.txt
-rw-r--r-- 1 rzf group   101 May 22  2017 NOTICE.txt
-rw-r--r-- 1 rzf group  1366 May 22  2017 README.txt
drwxr-xr-x 2 rzf group  4096 May 22  2017 sbin //集群启动停止命令
drwxr-xr-x 4 rzf group  4096 May 22  2017 share //存放说明文档，案例
~~~

- 重要目录说明

~~~ java
（1）bin目录：存放对Hadoop相关服务（HDFS,YARN）进行操作的脚本
（2）etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
（3）lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
（4）sbin目录：存放启动或停止Hadoop相关服务的脚本
（5）share目录：存放Hadoop的依赖jar包、文档、和官方案例
~~~

## Hadoop运行模式

Hadoop运行模式包括：本地模式、伪分布式模式以及完全分布式模式。

Hadoop官方网站：<http://hadoop.apache.org/>

### 本地运行模式

#### 官方WordCount案例

1. 创建在hadoop-2.7.2文件下面创建一个wcinput文件夹

~~~ java
[rzf@VM-8-10-centos hadoop-2.7.2]$ mkdir wcinput
~~~

2. 在wcinput文件下创建一个wc.input文件

~~~ java
[rzf@VM-8-10-centos wcinput]$ touch wc.input
[rzf@VM-8-10-centos wcinput]$ ll
total 0
-rw-r--r-- 1 rzf group 0 Dec 13 15:10 wc.input
~~~

3. 编辑wc.input文件

~~~ java
//输入下面内容
hadoop yarn
hadoop mapreduce
rzf
rzf
~~~

6. 回到Hadoop目录/opt/module/hadoop-2.7.2，执行程序

~~~ java
[rzf@VM-8-10-centos hadoop-2.7.2]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput
~~~

7. 查看计算结果

~~~ java
[rzf@VM-8-10-centos wcoutput]$ cat part-r-00000 
hadoop  2
mapreduce       1
rzf     2
yarn    1
~~~

### 伪分布模式

#### 启动HDFS并运行MapReduce程序

- 分析：
  - 配置集群
  - 启动、测试集群增、删、查
  - 执行WordCount案例

1. 配置集群

   - 配置：`hadoop-env.sh`

   ~~~ java
   //Linux系统中获取JDK的安装路径：
   echo $JAVA_HOME
   //修改JAVA_HOME 路径：
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   - 配置：`core-site.xml`

   ~~~ java
   <configuration>
       <property>
       <!-- 指定HDFS中NameNode的地址 -->
       <name>fs.defaultFS</name>
           <value>hdfs://VM-8-10-centos:9000</value>
       </property>
   
       <property>
       <!-- 指定Hadoop运行时产生文件的存储目录 -->
               <name>hadoop.tmp.dir</name>
               <value>/opt/module/hadoop-2.7.2/data/tmp</value>
       </property>
   </configuration>
   ~~~

   - 配置：`hdfs-site.xml`

   ~~~ java
   <!-- 指定HDFS副本的数量 -->
   <property>
   	<name>dfs.replication</name>
   	<value>1</value>
   </property>
   ~~~

2. 启动集群

   - **格式化NameNode**（第一次启动时格式化，以后就不要总格式化）

   ~~~ java
   [rzf@VM-8-10-centos hadoop-2.7.2]$ bin/hdfs namenode -format
   ~~~

   - 启动`NameNode`

   ~~~ java
   [rzf@VM-8-10-centos hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
   //hadoop-daemon.sh是hadoop的一个守护进程
   ~~~

   - 启动`DataNode`

   ~~~ java
   [rzf@VM-8-10-centos hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode
   ~~~

   - 查看是否启动成功

   ~~~ java
   [rzf@VM-8-10-centos hadoop-2.7.2]$ jps
   2641 DataNode
   2782 Jps
   2350 NameNode
   //jps是属于java命令
   //注意：jps是JDK中的命令，不是Linux命令。不安装JDK不能使用jps
   ~~~

3. 查看集群

   - `web`端查看`HDFS`文件系统

   - 查看产生的`log`日志进行排查错误。

     说明：在企业中遇到`Bug`时，经常根据日志提示信息去分析问题、解决`Bug`。

     当前目录：`/opt/module/hadoop-2.7.2/logs`

   - **思考：为什么不能一直格式化NameNode，格式化NameNode**，要注意什么？

     - 注意：格式化`NameNode`，会产生新的集群`id`,导致`NameNode`和`DataNode`的集群`id`不一致，集群找不到已往数据。所以，格式`NameNode`时，一定要先删除`data`数据和`log`日志，然后再格式化`NameNode`。

     ~~~ java
     [rzf@hadoop01 dfs]$ pwd
     /opt/module/hadoop-2.7.2/data/tmp/dfs
     [rzf@hadoop01 dfs]$ ll
     total 8
     drwx------ 3 rzf group 4096 Dec 14 09:46 data
     drwxr-xr-x 3 rzf group 4096 Dec 14 09:46 name
     //上面的文件夹下面，有两个文件夹，data和name，其中name存储的就是namenode节点的数据信息，data存储的就是datdanode的节点信息
     //在name中的version文件中，有集群的唯一标识
     [rzf@hadoop01 dfs]$ cd name/
     [rzf@hadoop01 name]$ ll
     total 8
     drwxr-xr-x 2 rzf group 4096 Dec 14 09:46 current
     -rw-r--r-- 1 rzf group   13 Dec 14 09:46 in_use.lock
     [rzf@hadoop01 name]$ cd current/
     [rzf@hadoop01 current]$ ll
     total 1040
     -rw-r--r-- 1 rzf group 1048576 Dec 14 09:56 edits_inprogress_0000000000000000001
     -rw-r--r-- 1 rzf group     350 Dec 14 09:46 fsimage_0000000000000000000
     -rw-r--r-- 1 rzf group      62 Dec 14 09:46 fsimage_0000000000000000000.md5
     -rw-r--r-- 1 rzf group       2 Dec 14 09:46 seen_txid
     -rw-r--r-- 1 rzf group     202 Dec 14 09:46 VERSION
     [rzf@hadoop01 current]$ pwd
     /opt/module/hadoop-2.7.2/data/tmp/dfs/name/current
     [rzf@hadoop01 current]$ cat VERSION 
     #Mon Dec 14 09:46:28 CST 2020
     namespaceID=1549727902
     clusterID=CID-98711717-9e44-4934-8311-02d6a51d5300//集群的唯一标识
     cTime=0
     storageType=NAME_NODE
     blockpoolID=BP-1112205128-10.0.8.10-1607910388669
     layoutVersion=-63
     [rzf@hadoop01 current]$ 
     //在data中也有一个集群的id
     [rzf@hadoop01 current]$ cat VERSION 
     #Mon Dec 14 09:46:58 CST 2020
     storageID=DS-00572539-6788-4c22-865e-4e2d74f960d4
     clusterID=CID-98711717-9e44-4934-8311-02d6a51d5300//集群id号码
     cTime=0
     datanodeUuid=13060fd6-f9e9-4860-a4ff-ee2a48da5a74
     storageType=DATA_NODE
     layoutVersion=-56
     [rzf@hadoop01 current]$ pwd
     /opt/module/hadoop-2.7.2/data/tmp/dfs/data/current
     //也就是说我们要保证namenode的id和datanode的id号码要一致
     ~~~

![1607912045974](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/14/101407-713337.png)

4. 操作集群

   1. 在HDFS文件系统上**创建**一个input文件夹

   ~~~ java
   //启动集群，在集群创建文件夹，-p表示创建多级文件目录
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -mkdir -p /rzf/user/input
   ~~~

   2. 将测试文件内容**上传**到文件系统上

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -put wcinput/wc.input /rzf/user/input
   //bin/hdfs dfs是hadoop特定的指令，-put表示上传
   ~~~

   3. **查看**上传的文件是否正确

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -ls /rzf/user/input
   ~~~

   4. 运行MapReduce程序

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /rzf/user/input /rzf/user/output
   //后面一个是输入路径，一个是输出路径
   ~~~

   5. 查看输出的结果

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -cat /rzf/user/output/part-r-00000
   hadoop  2
   mapreduce       1
   rzf     2
   yarn    1
   ~~~

   5. 将测试文件内容**下载**到本地

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ sudo bin/hdfs dfs -get /rzf/user/output/part-r-00000 E:/
   //将hdfs文件系统的文件下载到本地的e盘
   ~~~

   6. 删除输出的结果

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ sudo bin/hdfs dfs -rm -r /rzf/user/output/
   ~~~

#### 启动YARN并运行MapReduce程序

- 分析
  1. 配置集群在YARN上运行MR
  2. 启动、测试集群增、删、查
  3. 在YARN上执行WordCount案例

1. 配置集群

   1. 配置`yarn-env.sh`

   ~~~ java
   //配置一下JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   2. 配置`yarn-site.xml`

   ~~~ java
   <!-- Reducer获取数据的方式 -->
   <property>
    		<name>yarn.nodemanager.aux-services</name>
    		<value>mapreduce_shuffle</value>
   </property>
   
   <!-- 指定YARN的ResourceManager的地址 -->
   <property>
   <name>yarn.resourcemanager.hostname</name>
   <value>hadoop101</value>
   </property>
   
   ~~~

   3. 配置：`mapred-env.sh`

   ~~~ java
   //配置一下JAVA_HOME
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   4. 配置： (对`mapred-site.xml.template`重新命名为)` mapred-site.xml`

   ~~~ java
   mv mapred-site.xml.template mapred-site.xml
   <!-- 指定MR运行在YARN上 -->
   <property>
   		<name>mapreduce.framework.name</name>
   		<value>yarn</value>
   </property>
   ~~~

2. 启动集群

   启动前必须保证`NameNode`和`DataNode`已经启动

   1. 启动`ResourceManager`

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh start resourcemanager
   starting resourcemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-rzf-resourcemanager-hadoop01.out
   [rzf@hadoop01 hadoop-2.7.2]$ jps
   11928 Jps
   3066 DataNode
   2956 NameNode
   11709 ResourceManager
   ~~~

   2. 启动`NodeManager`

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
   starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-rzf-nodemanager-hadoop01.out
   [rzf@hadoop01 hadoop-2.7.2]$ jps
   12081 NodeManager
   3066 DataNode
   2956 NameNode
   12173 Jps
   11709 ResourceManager
   ~~~

   3. 端口说明

   ~~~ java
   50070 //查看hdfs
   8088 //查看maperreducer
   ~~~

3. 集群操作

   1. `YARN`的浏览器页面查看，如图2-35所示

   ~~~ java
   http://hadoop01:8088/cluster//使用的是8088端口
   ~~~

   2. 删除文件系统上的`output`文件

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -rm -r /rzf/user/output
   如果配置了hadoop的bin目录，那么可以直接用：hdfs dfs -rm -r /rzf/user/output命令
   ~~~

   3. 执行`MapReduce`程序

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /rzf/user/input /rzf/user/output
   ~~~

   4. 查看计算结果

   ~~~ java
   [rzf@hadoop01 hadoop-2.7.2]$ bin/hdfs dfs -cat /rzf/user/output/part-r-00000
   hadoop  2
   mapreduce       1
   rzf     2
   yarn    1
   ~~~

#### 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体配置步骤如下：

1. 配置`mapred-site.xml`,在该文件里面增加如下配置。

~~~ java
<!-- 历史服务器端地址 -->
<property>
<name>mapreduce.jobhistory.address</name>
<value>hadoop01:10020</value>
</property>
<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop01:19888</value>
</property>
~~~

2. 启动历史服务器

~~~ java
[rzf@hadoop01 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
starting historyserver, logging to /opt/module/hadoop-2.7.2/logs/mapred-rzf-historyserver-hadoop01.out
[rzf@hadoop01 hadoop-2.7.2]$ jps
12081 NodeManager
16006 JobHistoryServer
3066 DataNode
2956 NameNode
16060 Jps
11709 ResourceManager
~~~

3. 查看历史服务器

~~~ java
http://hadoop01:19888/jobhistory
~~~

4. 关闭历史服务器

~~~ java
[rzf@hadoop01 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh stop historyserver
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop resourcemanager
~~~

#### 配置日志的聚集

日志聚集概念：应用运行完成以后，将程序运行日志信息上传到`HDFS`系统上。

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试。

注意：开启日志聚集功能，需要重新启动`NodeManager 、ResourceManager和HistoryManager。`

开启日志聚集功能具体步骤如下：

1. 配置`yarn-site.xml`

~~~ java
<!-- 日志聚集功能使能 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>

<!-- 日志保留时间设置7天 -->
<property>
<name>yarn.log-aggregation.retain-seconds</name>
<value>604800</value>
</property>

~~~

2. 关闭`NodeManager 、ResourceManager和HistoryServer`

~~~ java
[rzf@hadoop01 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh stop historyserver
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop resourcemanager
~~~

3. 启动`NodeManager 、ResourceManager和HistoryServer`

~~~ java
[rzf@hadoop01 hadoop-2.7.2]$ sbin/mr-jobhistory-daemon.sh start historyserver
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
[rzf@hadoop01 hadoop-2.7.2]$ sbin/yarn-daemon.sh start resourcemanager
~~~

4. 重新执行`wordcount`程序，可以看到日志信息

~~~ java
http://hadoop01:19888/jobhistory
~~~

![1607916099022](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/14/112140-152339.png)

![1607916106318](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/14/155857-744104.png)

![1607916113894](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/14/112155-35265.png)

#### 配置文件说明

`Hadoop`配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值。

1. 默认配置文件

| 要获取的默认文件     | 文件存放在Hadoop的jar包中的位置                              |
| -------------------- | ------------------------------------------------------------ |
| [core-default.xml]   | hadoop-common-2.7.2.jar/   core-default.xml                  |
| [hdfs-default.xml]   | hadoop-hdfs-2.7.2.jar/   hdfs-default.xml                    |
| [yarn-default.xml]   | hadoop-yarn-common-2.7.2.jar/   yarn-default.xml             |
| [mapred-default.xml] | hadoop-mapreduce-client-core-2.7.2.jar/   mapred-default.xml |

2. 自定义配置文件

   `core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml`四个配置文件存放在`$HADOOP_HOME/etc/hadoop`这个路径上，用户可以根据项目需求重新进行修改配置。

### 完全分布式模式

#### 准备工作

- 准备3台客户机（关闭防火墙、静态ip、主机名称）
- 安装JDK
- 配置环境变量
- 安装Hadoop
- 配置环境变量
- 配置集群
- 单点启动
- 配置ssh
- 群起并测试集群

#### 编写集群分发脚本xsync

1. `scp（secure copy）`安全拷贝

~~~ java
(1) scp定义：
scp可以实现服务器与服务器之间的数据拷贝。（from server1 to server2）
（2）基本语法
scp    -r          $pdir/$fname              $user@hadoop$host:$pdir/$fname
命令   递归       要拷贝的文件路径/名称           目的用户@主机:目的路径/名称

~~~

2. 案例演示

~~~ java
（a）在hadoop101上，将hadoop101中/opt/module目录下的软件拷贝到hadoop102上。
[rzf@hadoop101 /]$ scp -r /opt/module  root@hadoop102:/opt/module

（b）在hadoop103上，将hadoop101服务器上的/opt/module目录下的软件拷贝到hadoop103上。
[rzf@hadoop103 opt]$sudo scp -r rzf@hadoop101:/opt/module root@hadoop103:/opt/module

（c）在hadoop103上操作将hadoop101中/opt/module目录下的软件拷贝到hadoop104上。
[rzf@hadoop103 opt]$ scp -r rzf@hadoop101:/opt/module root@hadoop104:/opt/module

//注意：拷贝过来的/opt/module目录，别忘了在hadoop102、hadoop103、hadoop104上修改所有文件的，所有者和所有者组。sudo chown rzf:group -R /opt/module

（d）将hadoop101中/etc/profile文件拷贝到hadoop102的/etc/profile上。
[rzf@hadoop101 ~]$ sudo scp /etc/profile root@hadoop102:/etc/profile
（e）将hadoop101中/etc/profile文件拷贝到hadoop103的/etc/profile上。
[rzf@hadoop101 ~]$ sudo scp /etc/profile root@hadoop103:/etc/profile
（f）将hadoop101中/etc/profile文件拷贝到hadoop104的/etc/profile上。
[rzf@hadoop101 ~]$ sudo scp /etc/profile root@hadoop104:/etc/profile
//注意：拷贝过来的配置文件别忘了source一下/etc/profile，。

~~~

3. `rsync `远程同步工具

   `rsync`主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

   `rsync`和`scp`区别：用`rsync`做文件的复制要比`scp`的速度快，`rsync`只对差异文件做更新。`scp`是把所有文件都复制过去。

- 基本语法

~~~ java
（1）基本语法
rsync    -av       $pdir/$fname              $user@hadoop$host:$pdir/$fname
命令   选项参数  -   要拷贝的文件路径/名称    目的用户@主机:目的路径/名称
~~~

- 选项参数说明

| 选项 | 功能         |
| ---- | ------------ |
| -a   | 归档拷贝     |
| -v   | 显示复制过程 |

- 案例演示

  把`hadoop101`机器上的`/opt/software`目录同步到`hadoop102`服务器的`root`用户下的`/opt/`目录

~~~ java
rsync -av /opt/software/ hadoop102:/opt/software
~~~

4. `xsync`集群分发脚本

~~~ java
（1）需求：循环复制文件到所有节点的相同目录下
	（2）需求分析：
		（a）rsync命令原始拷贝：
		rsync  -av     /opt/module  		 root@hadoop103:/opt/
		（b）期望脚本：
		xsync 要同步的文件名称
		（c）说明：在/home/rzf/bin这个目录下存放的脚本，rzf用户可以在系统任何地方直接执行。
（3）脚本实现
（a）在/home/rzf目录下创建bin目录，并在bin目录下xsync创建文件，文件内容如下：
    [rzf@hadoop102 ~]$ mkdir bin
    [rzf@hadoop102 ~]$ cd bin/
    [rzf@hadoop102 bin]$ touch xsync
    [rzf@hadoop102 bin]$ vi xsync
    //添加如下代码
    #!/bin/bash
    #1 获取输入参数个数，如果没有参数，直接退出
    pcount=$# # 获取参数输入的个数
    if ((pcount==0)); then
    echo no args;
    exit;
    fi

    #2 获取文件名称
    p1=$1 # 获取第一个参数
    fname=`basename $p1`# 获取文件名
    echo fname=$fname # 输出展示文件名

    #3 获取上级目录到绝对路径
    pdir=`cd -P $(dirname $p1); pwd`# 拿到p1文件所对应的路径
    echo pdir=$pdir

    #4 获取当前用户名称
    user=`whoami`

    #5 循环
    for((host=103; host<105; host++)); do
            echo ------------------- hadoop$host --------------
            rsync -av $pdir/$fname $user@hadoop$host:$pdir
            -          原始文件的路径  用户  那一台主机    路径
    done
（b）修改脚本 xsync 具有执行权限

[rzf@hadoop102 bin]$ chmod 777 xsync

（c）调用脚本形式：xsync 文件名称

[rzf@hadoop102 bin]$ xsync /home/rzf/bin

注意：如果将xsync放到/home/rzf/bin目录下仍然不能实现全局使用，可以将xsync移动到/usr/local/bin目录下。
~~~

#### 集群配置

1. 集群规划部署

|      | hadoop102           | hadoop103                     | hadoop104                    |
| ---- | ------------------- | ----------------------------- | ---------------------------- |
| HDFS | NameNode   DataNode | DataNode                      | SecondaryNameNode   DataNode |
| YARN | NodeManager         | ResourceManager   NodeManager | NodeManager                  |

2. 配置集群

   - 核心配置文件

   1. 配置core-site.xml

   ~~~ java
   <!-- 指定HDFS中NameNode的地址 -->
   <property>
   		<name>fs.defaultFS</name>
         <value>hdfs://hadoop102:9000</value>
   </property>
   
   <!-- 指定Hadoop运行时产生文件的存储目录 -->
   <property>
   		<name>hadoop.tmp.dir</name>
   		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
   </property>
   
   ~~~

   - HDFS配置文件

   1. 配置`hadoop-env.sh`

   ~~~ java
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   2. 配置`hdfs-site.xml`

   ~~~ java
   <property>
   		<name>dfs.replication</name>
   		<value>3</value>
   </property>
   
   <!-- 指定Hadoop辅助名称节点主机配置 -->
   <property>
         <name>dfs.namenode.secondary.http-address</name>
         <value>hadoop104:50090</value>
   </property>
   
   ~~~

   - `YARN`配置文件

   1. 配置`yarn-env.sh`

   ~~~ java
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   2. 配置`yarn-site.xml`

   ~~~ java
   <!-- Reducer获取数据的方式 -->
   <property>
   		<name>yarn.nodemanager.aux-services</name>
   		<value>mapreduce_shuffle</value>
   </property>
   
   <!-- 指定YARN的ResourceManager的地址 -->
   <property>
   		<name>yarn.resourcemanager.hostname</name>
   		<value>hadoop103</value>
   </property>
   
   ~~~

   - `MapReduce`配置文件

   1. 配置`mapred-env.sh`

   ~~~ java
   export JAVA_HOME=/opt/module/jdk1.8.0_144
   ~~~

   2. 配置`mapred-site.xml`

   ~~~ java
   //先修改名字
   cp mapred-site.xml.template mapred-site.xml
   <!-- 指定MR运行在Yarn上 -->
   <property>
   		<name>mapreduce.framework.name</name>
   		<value>yarn</value>
   </property>
   ~~~

3. 在集群上分发配置好的`Hadoop`配置文件

   ~~~ java
   xsync /opt/module/hadoop-2.7.2/
   ~~~

4. 查看文件分发情况

   ~~~ java
   cat /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml
   ~~~

#### 集群的单点启动

~~~ java
（1）如果集群是第一次启动，需要格式化NameNode
    [rzf@hadoop102 hadoop-2.7.2]$ hdfs namenode -format
（2）在hadoop102上启动NameNode
    [rzf@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start namenode
    [rzf@hadoop102 hadoop-2.7.2]$ jps
	3461 NameNode
（3）在hadoop102、hadoop103以及hadoop104上分别启动DataNode
    [rzf@hadoop102 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
    [rzf@hadoop102 hadoop-2.7.2]$ jps
    3461 NameNode
    3608 Jps
    3561 DataNode
    [rzf@hadoop103 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
    [rzf@hadoop103 hadoop-2.7.2]$ jps
    3190 DataNode
    3279 Jps
    [rzf@hadoop104 hadoop-2.7.2]$ hadoop-daemon.sh start datanode
    [rzf@hadoop104 hadoop-2.7.2]$ jps
    3237 Jps
    3163 DataNode
~~~

#### ssh无密码登录设置

![1607936671479](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202012/14/170433-194728.png)

`ssh`原理说明：服务器A会生成一对公钥和私钥，配合使用，把服务器A的公钥拷贝给服务器B，那么服务器A在传输数据的时候，用私钥对数据进行加密，当服务器B接受到数据的时候，使用A的公钥对数据解密操作即可读取数据。

1. 配置ssh

~~~ java
（1）基本语法
ssh另一台电脑的ip地址
（2）ssh连接时出现Host key verification failed的解决方法
[rzf@hadoop102 opt] $ ssh 192.168.1.103
The authenticity of host '192.168.1.103 (192.168.1.103)' can't be established.
RSA key fingerprint is cf:1e:de:d7:d0:4c:2d:98:60:b4:fd:ae:b1:2d:ad:06.
Are you sure you want to continue connecting (yes/no)? 
Host key verification failed.
（3）解决方案如下：直接输入yes
~~~

2. 无密钥配置

~~~ java
1,生成公钥和私钥：
[rzf@hadoop102 .ssh]$ ssh-keygen -t rsa
//然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）
2,将公钥拷贝到要免密登录的目标机器上
[rzf@hadoop102 .ssh]$ ssh-copy-id hadoop102
//虽然在hadoop102机器上生成密钥，但是还是需要向hadoop机器上拷贝一次
[rzf@hadoop102 .ssh]$ ssh-copy-id hadoop103
[rzf@hadoop102 .ssh]$ ssh-copy-id hadoop104
//注意：
//还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104；
su root
//还需要在hadoop103上采用rzf账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。
//也就是在hadoop103上生成公钥和私钥，分别配置到hadoop102,hadoop104
~~~

3. `.ssh`文件夹下（`~/.ssh`）的文件功能解释

| known_hosts     | 记录ssh访问过计算机的公钥(public   key) |
| --------------- | --------------------------------------- |
| id_rsa          | 生成的私钥                              |
| id_rsa.pub      | 生成的公钥                              |
| authorized_keys | 存放授权过得无密登录服务器公钥          |

#### 群起集群

1. 配置slaves

~~~ java
/opt/module/hadoop-2.7.2/etc/hadoop/slaves
[rzf@hadoop102 hadoop]$ vi slaves
在该文件中增加如下内容：
hadoop102
hadoop103
hadoop104
//注意，该文件中不能有空格
//同步所有节点配置文件
[rzf@hadoop102 hadoop]$ xsync slaves
~~~

2. 启动集群

~~~ java
//如果集群是第一次启动，需要格式化NameNode（注意格式化之前，一定要先停止上次启动的所有namenode和datanode进程，然后再删除data和log数据）
bin/hdfs namenode -format
//启动hdfs
[rzf@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[rzf@hadoop102 hadoop-2.7.2]$ jps
4166 NameNode
4482 Jps
4263 DataNode
[rzf@hadoop103 hadoop-2.7.2]$ jps
3218 DataNode
3288 Jps
[rzf@hadoop104 hadoop-2.7.2]$ jps
3221 DataNode
3283 SecondaryNameNode
3364 Jps
//启动yarn
sbin/start-yarn.sh
//注意：NameNode和ResourceManger如果不是同一台机器，不能在NameNode上启动 YARN，应该在ResouceManager所在的机器上启动YARN。
~~~

#### 集群的基本测试

~~~ java
（1）上传文件到集群
上传小文件
[rzf@hadoop102 hadoop-2.7.2]$ hdfs dfs -mkdir -p /user/rzf/input
[rzf@hadoop102 hadoop-2.7.2]$ hdfs dfs -put wcinput/wc.input /user/rzf/input
上传大文件
[rzf@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -put
 /opt/software/hadoop-2.7.2.tar.gz  /user/rzf/input
（2）上传文件后查看文件存放在什么位置
（a）查看HDFS文件存储路径
[rzf@hadoop102 subdir0]$ pwd
/opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/BP-938951106-192.168.10.107-1495462844069/current/finalized/subdir0/subdir0
（b）查看HDFS在磁盘存储文件内容
[rzf@hadoop102 subdir0]$ cat blk_1073741825
hadoop yarn
hadoop mapreduce 
rzf
rzf
（3）拼接
-rw-rw-r--. 1 rzf rzf 134217728 5月  23 16:01 blk_1073741836
-rw-rw-r--. 1 rzf rzf   1048583 5月  23 16:01 blk_1073741836_1012.meta
-rw-rw-r--. 1 rzf rzf  63439959 5月  23 16:01 blk_1073741837
-rw-rw-r--. 1 rzf rzf    495635 5月  23 16:01 blk_1073741837_1013.meta
[rzf@hadoop102 subdir0]$ cat blk_1073741836>>tmp.file
[rzf@hadoop102 subdir0]$ cat blk_1073741837>>tmp.file
[rzf@hadoop102 subdir0]$ tar -zxvf tmp.file
（4）下载
[rzf@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -get
 /user/rzf/input/hadoop-2.7.2.tar.gz ./

~~~

#### 集群启动/停止方式总结

~~~ java
1.	各个服务组件逐一启动/停止
	（1）分别启动/停止HDFS组件
		hadoop-daemon.sh  start / stop  namenode / datanode / secondarynamenode
	（2）启动/停止YARN
		yarn-daemon.sh  start / stop  resourcemanager / nodemanager
2.	各个模块分开启动/停止（配置ssh是前提）常用
	（1）整体启动/停止HDFS
		start-dfs.sh   /  stop-dfs.sh
	（2）整体启动/停止YARN
		start-yarn.sh  /  stop-yarn.sh
~~~

