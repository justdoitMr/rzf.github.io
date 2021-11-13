## Flink部署
<!-- TOC -->

- [Flink部署](#flink部署)
    - [Local本地模式安装](#local本地模式安装)
        - [原理](#原理)
        - [操作](#操作)
        - [测试](#测试)
    - [Standalone 模式](#standalone-模式)
        - [原理，](#原理)
        - [安装](#安装)
        - [StandAlone的HA模式](#standalone的ha模式)
    - [Flink-On-Yarn-模式](#flink-on-yarn-模式)
        - [原理](#原理-1)
        - [两种模式](#两种模式)
        - [集群部署](#集群部署)
            - [Session会话模式](#session会话模式)
            - [Job分离模式--用的更多](#job分离模式--用的更多)

<!-- /TOC -->
Flink支持多种安装模式

- Local—本地单机模式，学习测试时使用
- Standalone—独立集群模式，Flink自带集群，开发测试环境使用
- StandaloneHA—独立集群高可用模式，Flink自带集群，开发测试环境使用
- On Yarn—计算资源统一由Hadoop YARN管理，生产环境使用

### Local本地模式安装

#### 原理

![1621563914228](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/132832-31487.png)

本地模式下面是以多线程去模拟Flink集群资源。

1. Flink程序由JobClient进行提交
2. JobClient将作业提交给JobManager
3. JobManager负责协调资源分配和作业执行。资源分配完成后，任务将提交给相应的TaskManager
4. TaskManager启动一个线程以开始执行。TaskManager会向JobManager报告状态更改,如开始执行，正在进行或已完成。 
5. 作业执行完成后，结果将发送回客户端(JobClient)

#### 操作

1.下载安装包

https://archive.apache.org/dist/flink/

2.上传flink-1.12.0-bin-scala_2.12.tgz到node1的指定目录

3.解压

```java
tar -zxvf flink-1.12.0-bin-scala_2.12.tgz 
```

4.如果出现权限问题，需要修改权限

```java
chown -R root:root /export/server/flink-1.12.0
```

5.改名或创建软链接

```java
mv flink-1.12.0 flink

ln -s /export/server/flink-1.12.0 /export/server/flink
```

#### 测试

1.准备文件/root/words.txt

```
vim /root/words.txt

hello me you her
hello me you
hello me
hello
```

2.启动Flink本地“集群”

```java
/export/server/flink/bin/start-cluster.sh
```

3.使用jps可以查看到下面两个进程

```java
  \- TaskManagerRunner

  \- StandaloneSessionClusterEntrypoint
```

4. 访问Flink的Web UI

![1621564188615](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621564188615.png)

slot在Flink里面可以认为是资源组，Flink是通过将任务分成子任务并且将这些子任务分配到slot来并行执行程序。

5. 执行官方案例

```java
/export/server/flink/bin/flink run /export/server/flink/examples/batch/WordCount.jar --input /root/words.txt --output /root/out
```

6. 停止Flink

```java
/export/server/flink/bin/stop-cluster.sh
```

### Standalone 模式

#### 原理，

![1621564602661](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/103729-430205.png)

![1621564329905](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170405-405689.png)

**规划**

1.集群规划:

\- 服务器: hadoop100(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop101(Slave): TaskManager

\- 服务器: hadoop102(Slave): TaskManager

#### 安装

1. 解压缩 flink-1.10.1-bin-scala_2.12.tgz，进入 conf 目录中。
2. 修改 flink/conf/flink-conf.yaml 文件：

```java
jobmanager.rpc.address: hadoop100 //配置jobmanager地址,相当于spark中的driver
taskmanager.numberOfTaskSlots: 2 //配置每一个taskmanager的slot数量
web.submit.enable: true //配置可用通过web ui进行访问

#历史服务器
jobmanager.archive.fs.dir: hdfs://hadoop100:8020/flink/completed-jobs/
historyserver.web.address: hadoop100
historyserver.web.port: 8082
historyserver.archive.fs.dir: hdfs://hadoop100:8020/flink/completed-jobs/

//修改masters
vim /export/server/flink/conf/masters
hadoop100:8081
```

3. 修改 /conf/slaves 文件：

```java
//添加如下内容，也就是集群节点的地址
hadoop101
hadoop102
```

4. 分发给另外两台机子：

```java
xsync flink-1.10.1
```

5. 启动flink

```java
[rzf@hadoop100 flink-1.10.1]$ bin/start-cluster.sh  //也可以进行单点启动
Starting cluster.
Starting standalonesession daemon on host hadoop100. //后面提交任务就是通过这个进程提交
Starting taskexecutor daemon on host hadoop101. //具体执行任务的节点
Starting taskexecutor daemon on host hadoop102.
```

6. 访问flink集群

```java
http://hadoop100:8081/#/overview //通过8081端口上面已经配置过8081端口

http://hadoop100:8081/#/overview

http://hadoop100:8082/#/overview //访问历史服务器
```

![1614308298297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/02/222551-596516.png)

7. 执行官方案例

```java
/export/server/flink/bin/flink run /export/server/flink/examples/batch/WordCount.jar
```

#### StandAlone的HA模式

![1621565322255](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170706-63710.png)

并不是说jobManager挂掉以后，另一个jobManager会立马上位，而是说还会有一个超时时间。

从之前的架构中我们可以很明显的发现 JobManager 有明显的单点问题(SPOF，single point of failure)。JobManager 肩负着任务调度以及资源分配，一旦 JobManager 出现意外，其后果可想而知。在 Zookeeper 的帮助下，一个 Standalone的Flink集群会同时有多个活着的 JobManager，其中只有一个处于工作状态，其他处于 Standby 状态。当工作中的 JobManager 失去连接后(如宕机或 Crash)，Zookeeper 会从 Standby 中选一个新的 JobManager 来接管 Flink 集群。

**操作**

1.集群规划

```java
\- 服务器: hadoop100(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop101(Master + Slave): JobManager + TaskManager

\- 服务器: hadoop102(Slave): TaskManager
```

2.启动ZooKeeper

```ja
zkServer.sh status

zkServer.sh stop

zkServer.sh start
```

3.启动HDFS

```java
/export/serves/hadoop/sbin/start-dfs.sh
```

4.停止Flink集群

```java
/export/server/flink/bin/stop-cluster.sh
```

5.修改flink-conf.yaml,增加如下内容G

```
state.backend: filesystem
state.backend.fs.checkpointdir: hdfs://hadoop100:8020/flink-checkpoints
high-availability: zookeeper
high-availability.storageDir: hdfs://hadoop100:8020/flink/ha/
high-availability.zookeeper.quorum: hadoop100:2181,hadoop101:2181,hadoop102:2181
```

6.修改masters

vim /export/server/flink/conf/masters

```java
hadoop100 8081
hadoop101 8081
```

7.同步

```
scp -r /export/server/flink/conf/flink-conf.yaml hadoop101:/export/server/flink/conf/
scp -r /export/server/flink/conf/flink-conf.yaml hadoop102:/export/server/flink/conf/
scp -r /export/server/flink/conf/masters hadoop101:/export/server/flink/conf/
scp -r /export/server/flink/conf/masters hadoop102:/export/server/flink/conf/
```

8.修改hadoop101上的flink-conf.yam

```
vim /export/server/flink/conf/flink-conf.yaml
jobmanager.rpc.address: hadoop101
```

9.重新启动Flink集群,node1上执行

```java
/export/server/flink/bin/stop-cluster.sh

/export/server/flink/bin/start-cluster.sh
```

![1621565780708](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565780708.png)

10.使用jps命令查看,发现没有Flink相关进程被启动

11.查看日志,发现如下错误

![1621565849864](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565849864.png)

因为在Flink1.8版本后,Flink官方提供的安装包里没有整合HDFS的jar

12. 下载jar包并在Flink的lib目录下放入该jar包并分发使Flink能够支持对Hadoop的操作
13. 放入lib目录

![1621565909778](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1621565909778.png)

14. 分发
15. 重新启动Flink集群,hadoop100上执行

```java
/export/server/flink/bin/stop-cluster.sh

/export/server/flink/bin/start-cluster.sh
```

16. 使用jps命令查看,发现三台机器已经ok

**测试**

1. 访问WebUI

```java
http://hadoop100:8081/#/job-manager/config

http://hadoop101:8081/#/job-manager/config
```

2. 执行wc

```java
/export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
```

3. kill掉其中一个master
4. 重新执行wc,还是可以正常执行

```java
/export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar

```

5. 停止集群

```java
/export/server/flink/bin/stop-cluster.sh
```

### Flink-On-Yarn-模式

#### 原理

![1621569017735](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/170739-620796.png)

**Flink和Yarn的交互过程**

![1621569052576](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/115214-814296.png)

1. 用户从yarn客户端上传jar包和配置文件到hdfs文件系统上面。
2. 找Resourcemanager注册并且申请资源。
3. resourcemanager会找到一个空闲的nodemanager启动applicationmaster作业。启动完毕之后，就相当于启动了一个jobManager进程，负责管理整个作业，jobManager和ApplicationMaster运行在同一个container上。一旦他们被成功启动，AppMaster就知道JobManager的地址(AM它自己所在的机器)。它就会为TaskManager生成一个新的Flink配置文件(他们就可以连接到JobManager)。这个配置文件也被上传到HDFS上。此外，AppMaster容器也提供了Flink的web服务接口。YARN所分配的所有端口都是临时端口，这允许用户并行执行多个Flink
4. applicationmaster向resourcemanager申请资源，也就是申请节点使用.
5. applicationmaster启动其余的节点，也就是taskManager
6. 所有节点启动开之后，每隔节点都去hdfs上面下载jar包和配置文件信息。

#### 两种模式

**Session会话模式**

在yarn集群中启动了一个Flink集群，并且会重复利用该集群

![1621569957189](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/120655-703592.png)

![1621661620440](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/133341-669155.png)

**Job分离模式**

![1621570170828](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/21/120932-662903.png)

![1621661649056](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/22/133434-869292.png)

对于每一个作业都是申请一个flink集群，作业执行完毕之后就关闭集群资源。

#### 集群部署

1. 关闭yarn的内存检查

```java
//因flink和spark一样，在运行的时候需要大量的内存
vim /export/server/hadoop/etc/hadoop/yarn-site.xml

<!-- 关闭yarn内存检查 -->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
```

2. 分发修改后的配置文件
3. 重启yarn

```java
/export/server/hadoop/sbin/stop-yarn.sh

/export/server/hadoop/sbin/start-yarn.sh
```

##### Session会话模式

在Yarn上启动一个Flink集群,并重复使用该集群,后续提交的任务都是给该集群,资源会被一直占用,除非手动关闭该集群----适用于大量的小任务

1.在yarn上启动一个Flink集群/会话，hadoop100上执行以下命令

```java
/export/server/flink/bin/yarn-session.sh -n 2 -tm 800 -s 1 -d
```

**说明:**

```java
- 申请2个CPU、1600M内存

-n 表示申请2个容器，这里指的就是多少个taskmanager

-tm 表示每个TaskManager的内存大小

-s 表示每个TaskManager的slots数量

-d 表示以后台程序方式运行
```

注意:

```java
该警告不用管

WARN  org.apache.hadoop.hdfs.DFSClient  - Caught exception 

java.lang.InterruptedException
```

2. 查看ui界面

```java
http://hadoop100:8088/cluster
```

3. 使用flink run提交任务：

```java
  /export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
```

4. 运行完之后可以继续运行其他的小任务

```java
  /export/server/flink/bin/flink run  /export/server/flink/examples/batch/WordCount.jar
```

5. 关闭yarn-session：关闭任务需要通过任务号关闭

yarn application -kill application_1609508087977_0005

##### Job分离模式--用的更多

针对每个Flink任务在Yarn上启动一个独立的Flink集群并运行,结束后自动关闭并释放资源,----适用于大任务

1.直接提交job

```java
/export/server/flink/bin/flink run -m yarn-cluster -yjm 1024 -ytm 1024 /export/server/flink/examples/batch/WordCount.jar
```

**说明**

```java
-m  jobmanager的地址

-yjm 1024 指定jobmanager的内存信息

-ytm 1024 指定taskmanager的内存信息
```

2.查看UI界面

`http://hadoop100:8088/cluster`