# Flume

## 第一章，Flume概述

## 1.1，`Flume`的定义

`Flume`是`Cloudera`提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。用来收集日志，`Flume`基于流式架构，灵活简单。**流式架构，处理数据的单位是按照行处理，而不是按照文件。**

1. `Flume`的作用。

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/122420-496110.png)

   - `flume`的作用就是实时的读取服务器或者本地磁盘的数据，将数据写入到`Hdfs`文件系统。

## 1.2，`flume`的优点

1. 可以和任意存储进程集成。
2. **输入的的数据速率大于写入目的存储的速率，flume会进行缓冲，减小hdfs的压力**。
3. flume中的事务基于channel，使用了两个事务模型（sender + receiver），确保消息被可靠发送。
4. Flume使用两个独立的事务分别负责从soucrce到channel，以及从channel到sink的事件传递。一旦事务中所有的数据全部成功提交到channel，那么source才认为该数据读取完成。同理，只有成功被sink写出去的数据，才会从channel中移除。

## 1.3，`flume`组成架构

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/122533-738341.png)

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/122536-119961.png)

- `flume`体现的是数据的可靠传输性，**而不是不重复性**。

#### 1.3.1，`agent`

- Agent是一个JVM进程，它以事件的形式将数据从源头送至目的。

- Agent主要有3个部分组成，Source、Channel、Sink。

#### 1.3.2，source

Source是数据的收集端，负责将数据捕获后进行特殊的格式化，将数据封装到事件（event） 里，然后将事件推入Channel中。

Flume提供了各种source的实现，包括Avro Source、**Exce Source**、**Spooling Directory  Source**、NetCat Source、Syslog Source、Syslog TCP Source、Syslog UDP  Source、HTTP Source、HDFS Source，etc。如果内置的Source无法满足需要， Flume还支持自定义Source。

source采用推的方式将数据传输到channel。

#### 1.3.3，channel

- Channel是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink运作在不同的速率上Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。

- Flume自带两种Channel：**Memory Channel和File Channel**。
  - Memory Channel是内存中的队列。Memory Channel在不需要关心数据丢失的情景下适用。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。
  - File Channel将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数据。

#### 1.3.4，sink

- Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。

- Sink是完全事务性的。在从Channel批量删除数据之前，每个Sink用Channel启动一个事务。批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务。事务一旦被提交，该Channel从自己的内部缓冲区删除事件。

- Sink组件目的地包括hdfs、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。

#### 1.3.5，event

- 传输单元，Flume数据传输的基本单元，以事件的形式将数据从源头送至目的地。  Event由可选的header和载有数据的一个byte array 构成。Header是容纳了key-value字符串对的HashMap。

  ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/122903-628622.png)

#### 1.3.6，组成小结

1. Flume组成，Put事务，Take事务
2. Taildir Source：断点续传、多目录。Flume1.6以前需要自己自定义Source记录每次读取文件位置，实现断点续传。
3. File Channel：数据存储在磁盘，宕机数据可以保存。但是传输速率慢。适合对数据传输可靠性要求高的场景，比如，金融行业。
4. Memory Channel：数据存储在内存中，宕机数据丢失。传输速率快。适合对数据传输可靠性要求不高的场景，比如，普通的日志数据。
5. Kafka Channel：减少了Flume的Sink阶段，省去sink阶段，但是下一阶段必须是kafka阶段。提高了传输效率。
6. Source到Channel是Put事务
7. Channel到Sink是Take事务

### 1.4，flume的拓扑结构

#### 1.4.1，拓扑结构一(Flume Agent连接)

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/122946-321067.png)

- 这种模式是将多个flume给顺序连接起来了，从最初的source开始到最终sink传送的目的存储系统。此模式不建议桥接过多的flume数量， flume数量过多不仅会影响传输速率，而且一旦传输过程中某个节点flume宕机，会影响整个传输系统。

#### 1.4.2，拓扑结构二(单source，多channel、sink)

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123001-609121.png)

- Flume支持将事件流向一个或者多个目的地。这种模式将数据源复制到多个channel中，每个channel都有相同的数据，sink可以选择传送的不同的目的地。

#### 1.4.3，拓扑结构三（Flume负载均衡）

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123004-913413.png)

- Flume支持使用将多个sink逻辑上分到一个sink组，flume将数据发送到不同的sink，主要解决负载均衡和故障转移问题。

#### 1.4.4，拓扑结构四（Flume Agent聚合）

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123006-937298.png)

- 这种模式是我们最常见的，也非常实用，日常web应用通常分布在上百个服务器，大者甚至上千个、上万个服务器。产生的日志，处理起来也非常麻烦。用flume的这种组合方式能很好的解决这一问题，每台服务器部署一个flume采集日志，传送到一个集中收集日志的flume，再由此flume上传到hdfs、hive、hbase、jms等，进行日志分析。

### 1.5，agent工作原理

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123009-21125.png)

说明：source端接收原始数据，接下来发送给channel选择器进行处理，channel把数据发送给拦截器，对数据做初始化拦截，在这里拦截器我们可以自己定义，在这里也可以不做拦截处理，对于处理后的数据，channel选择器会把数据发送到不同的channel,最后所有数据发送到sink选择器，sink选择处理器选择其中一个sink去获取channel的数据，并且将获取的数据写入到下一个阶段。

## 第二章，实战演练

### 2.1，Flume的安装

1. 安装下载地址

   - Flume官网地址

     ~~~java
     http://flume.apache.org/
     ~~~

   - 文档查看地址

     ~~~java
     http://flume.apache.org/FlumeUserGuide.html
     ~~~

   - 下载地址

     ~~~ java
     http://archive.apache.org/dist/flume/
     ~~~

2. 安装部署

   1. 将apache-flume-1.7.0-bin.tar.gz上传到linux的/opt/software目录下

   2. 解压apache-flume-1.7.0-bin.tar.gz到/opt/module/目录下

      ~~~ java
      tar -zxf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
      ~~~

   3. 修改apache-flume-1.7.0-bin的名称为flume

   4. 将flume/conf下的flume-env.sh.template文件修改为flume-env.sh，并配置flume-env.sh文件

      ~~~java
      mv flume-env.sh.template flume-env.sh
      vi flume-env.sh
      export JAVA_HOME=/opt/module/jdk1.8.0_144
      //每一个agent都有自己的配置，所以只需要配置JAVA_HOME即可。
      ~~~

      - 到此为止，我们的flume安装配置已经完成

### 2.2，**监控端口数据官方**案例

~~~ java
/*首先启动Flume任务，监控本机44444端口，服务端；
然后通过netcat工具向本机44444端口发送消息，客户端；
最后Flume将监听的数据实时显示在控制台。*/
~~~

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123117-937576.png)

1. 安装netcat工具

   ~~~ java
   sudo yum install -y nc
   ~~~

2. 判断44444端口是否被占用

   ~~~ java
   sudo netstat -tunlp | grep 44444
   ~~~

   - 功能描述：netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。 

   - 基本语法：netstat [选项]

   - 选项参数：

   ​	-t或--tcp：显示TCP传输协议的连线状况； 

   ​	-u或--udp：显示UDP传输协议的连线状况；

   ​	-n或--numeric：直接使用ip地址，而不通过域名服务器； 

   ​	-l或--listening：显示监控中的服务器的Socket； 

   ​	-p或--programs：显示正在使用Socket的程序识别码（PID）和程序名称；

3. 创建Flume Agent配置文件flume-netcat-logger.conf

   ~~~java
   //在flume目录下创建job文件夹并进入job文件夹。在job文件夹下创建Flume Agent配置文件flume-netcat-logger.conf
   //添加内容如下：
   # Name the components on this agent
   a1.sources = r1//a1标示agent1,agent1的sources是r1,agent1的sinks是k1，agent1的channels是c1.
   a1.sinks = k1
   a1.channels = c1
   
   
   #描述源配置信息
   # Describe/configure the source
   a1.sources.r1.type = netcat//r1的类型是netcat网络端口
   a1.sources.r1.bind = localhost//主机是localhost
   a1.sources.r1.port = 44444//端口是4444
   
   #描述输出端的配置信息
   # Describe the sink
   a1.sinks.k1.type = logger//sinks的类型是logger
   
   #描述channel端的配置信息
   # Use a channel which buffers events in memory
   a1.channels.c1.type = memory//标示类型
   a1.channels.c1.capacity = 1000//容量有1000
   a1.channels.c1.transactionCapacity = 100//事务有100条
   
   #把三个组件连接起来
   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1//连线
   a1.sinks.k1.channel = c1
   ~~~

4. 先开启flume监听端口

   ~~~ java
   //第一种写法：
   bin/flume-ng agent --conf conf/ --name a1 --conf-file job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
   //第二种写法：
   bin/flume-ng agent -c conf/ -n a1 –f job/flume-netcat-logger.conf -Dflume.root.logger=INFO,console
   ~~~

   - 参数说明：

     ​	--conf conf/  ：表示配置文件存储在conf/目录

     ​	--name a1	：表示给agent起名为a1

     ​	--conf-file job/flume-netcat.conf ：flume本次启动读取的配置文件是在job文件夹下的flume-telnet.conf文件。

     ​	-Dflume.root.logger==INFO,console ：-D表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括:log、info、warn、error。

5. 使用netcat工具向本机的44444端口发送内容

   ~~~java
   nc localhost 44444
   ~~~

6. 在Flume监听页面观察接收数据情况

### 2.3，**实时读取本地文件到HDFS案例**

1. 需求：

   实时监控Hive日志，并上传到HDFS中

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123122-640180.png)

2. Flume要想将数据输出到HDFS，必须持有Hadoop相关jar包

   ~~~ java
   将commons-configuration-1.6.jar、
   hadoop-auth-2.7.2.jar、
   hadoop-common-2.7.2.jar、
   hadoop-hdfs-2.7.2.jar、
   commons-io-2.4.jar、
   htrace-core-3.1.0-incubating.jar
   拷贝到/opt/module/flume/lib文件夹下。
   ~~~

3. 创建flume-file-hdfs.conf文件

   - 注：要想读取Linux系统中的文件，就得按照Linux命令的规则执行命令。由于Hive日志在Linux系统中所以读取文件的类型选择：exec即execute执行的意思。表示执行Linux命令来读取文件。

     ~~~java
     //配置文件内容
     # Name the components on this agent
     a2.sources = r2
     a2.sinks = k2
     a2.channels = c2
     
     # Describe/configure the source
     a2.sources.r2.type = exec
     a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
     a2.sources.r2.shell = /bin/bash -c
     
     # Describe the sink
     a2.sinks.k2.type = hdfs
     a2.sinks.k2.hdfs.path = hdfs://hadoop102:9000/flume/%Y%m%d/%H
     #上传文件的前缀
     a2.sinks.k2.hdfs.filePrefix = logs-
     #是否按照时间滚动文件夹
     a2.sinks.k2.hdfs.round = true
     #多少时间单位创建一个新的文件夹
     a2.sinks.k2.hdfs.roundValue = 1
     #重新定义时间单位
     a2.sinks.k2.hdfs.roundUnit = hour
     #是否使用本地时间戳
     a2.sinks.k2.hdfs.useLocalTimeStamp = true
     #积攒多少个Event才flush到HDFS一次
     a2.sinks.k2.hdfs.batchSize = 1000
     #设置文件类型，可支持压缩
     a2.sinks.k2.hdfs.fileType = DataStream
     #多久生成一个新的文件
     a2.sinks.k2.hdfs.rollInterval = 60
     #设置每个文件的滚动大小
     a2.sinks.k2.hdfs.rollSize = 134217700
     #文件的滚动与Event数量无关
     a2.sinks.k2.hdfs.rollCount = 0
     
     # Use a channel which buffers events in memory
     a2.channels.c2.type = memory
     a2.channels.c2.capacity = 1000
     a2.channels.c2.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a2.sources.r2.channels = c2
     a2.sinks.k2.channel = c2
     ~~~

     - **注意**：

       对于所有与时间相关的转义序列，Event Header中必须存在以 “timestamp”的key（除非hdfs.useLocalTimeStamp设置为true，此方法会使用TimestampInterceptor自动添加timestamp）。

4. 执行监控配置

   ~~~ java
   bin/flume-ng agent --conf conf/ --name a2 --conf-file job/flume-file-hdfs.conf
   --conf代表配置文件夹，可以用-c代替，--name代表agent的名字，可以用-n代替，--conf-file代表配置文件，可以用-f代替
   bin/flume-ng agent -c conf/ -n a2 -f job/flume-file-hdfs.conf 
   ~~~

5. 开启Hadoop和Hive并操作Hive产生日志

   ~~~java
   sbin/start-dfs.sh
   sbin/start-yarn.sh
   
   bin/hive
   hive (default)>
   ~~~

6. 在HDFS上面可以查看到文件。

### 2.4，**实时读取目录文件到HDFS案例**

1. 案例需求：使用Flume监听整个目录的文件

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123129-175582.png)

2. 创建配置文件flume-dir-hdfs.conf

   ~~~java
   //配置文件信息
   a3.sources = r3
   a3.sinks = k3
   a3.channels = c3
   
   # Describe/configure the source
   a3.sources.r3.type = spooldir//定义source类型是文件
   a3.sources.r3.spoolDir = /opt/module/flume/upload//定义监控的目录
   a3.sources.r3.fileSuffix = .COMPLETED//定义文件上传完的后缀
   a3.sources.r3.fileHeader = true//是否有文件头
   #忽略所有以.tmp结尾的文件，不上传
   a3.sources.r3.ignorePattern = ([^ ]*\.tmp)//忽略所有已tmp文件结尾的文件
   
   # Describe the sink
   a3.sinks.k3.type = hdfs//sink类型是hdfs
   a3.sinks.k3.hdfs.path = hdfs://hadoop102:9000/flume/upload/%Y%m%d/%H//文件上传到hdfs的目录
   #上传文件的前缀
   a3.sinks.k3.hdfs.filePrefix = upload-
   #是否按照时间滚动文件夹
   a3.sinks.k3.hdfs.round = true
   #多少时间单位创建一个新的文件夹
   a3.sinks.k3.hdfs.roundValue = 1
   #重新定义时间单位
   a3.sinks.k3.hdfs.roundUnit = hour
   #是否使用本地时间戳
   a3.sinks.k3.hdfs.useLocalTimeStamp = true
   #积攒多少个Event才flush到HDFS一次
   a3.sinks.k3.hdfs.batchSize = 100
   #设置文件类型，可支持压缩
   a3.sinks.k3.hdfs.fileType = DataStream
   #多久生成一个新的文件
   a3.sinks.k3.hdfs.rollInterval = 60
   #设置每个文件的滚动大小大概是128M
   a3.sinks.k3.hdfs.rollSize = 134217700
   #文件的滚动与Event数量无关
   a3.sinks.k3.hdfs.rollCount = 0
   
   # Use a channel which buffers events in memory
   a3.channels.c3.type = memory
   a3.channels.c3.capacity = 1000
   a3.channels.c3.transactionCapacity = 100
   
   # Bind the source and sink to the channel
   a3.sources.r3.channels = c3
   a3.sinks.k3.channel = c3
   ~~~

3. 启动监控文件夹命令

   ~~~ java
   bin/flume-ng agent --conf conf/ --name a3 --conf-file job/flume-dir-hdfs.conf
   ~~~

   - 说明：

     在使用Spooling Directory Source时

     1) 不要在监控目录中创建并持续修改文件

     2) 上传完成的文件会以.COMPLETED结尾

     3) 被监控文件夹每500毫秒扫描一次文件变动

4. 向upload文件夹中添加文件

   ~~~ java
   //在/opt/module/flume目录下创建upload文件夹
   touch atguigu.txt
   touch atguigu.tmp
   touch atguigu.log
   ~~~

5. 可以查看到hdfs文件上面上传的文件。

### 2.5，单数据源多出口案例(选择器)

- 单Source多Channel、Sink

  ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123134-161273.png)

1. 案例需求：使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123137-217465.png)

2. 在/opt/module/flume/job目录下创建group1文件夹

3. 在/opt/module/datas/目录下创建flume3文件夹

4. 创建flume-file-flume.conf

   配置1个接收日志文件的source和两个channel、两个sink，分别输送给flume-flume-hdfs和flume-flume-dir。

   ~~~ java
   # Name the components on this agent
   a1.sources = r1
   a1.sinks = k1 k2
   a1.channels = c1 c2
   # 将数据流复制给所有channel
   a1.sources.r1.selector.type = replicating//复制类型选择器
   
   # Describe/configure the source
   a1.sources.r1.type = exec
   a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log//监控的文件
   a1.sources.r1.shell = /bin/bash -c
   
   # Describe the sink
   # sink端的avro是一个数据发送者
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop102 
   a1.sinks.k1.port = 4141
   
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop102
   a1.sinks.k2.port = 4142
   
   # Describe the channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 1000
   a1.channels.c2.transactionCapacity = 100
   
   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1 c2
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ~~~

   - 注：Avro是由Hadoop创始人Doug Cutting创建的一种语言无关的数据序列化和RPC框架。

     注：RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

5. 创建flume-flume-hdfs.conf

   - 配置上级Flume输出的Source，输出是到HDFS的Sink。

     ~~~ java
     # Name the components on this agent
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     # Describe/configure the source
     # source端的avro是一个数据接收服务
     a2.sources.r1.type = avro
     a2.sources.r1.bind = hadoop102
     a2.sources.r1.port = 4141
     
     # Describe the sink
     a2.sinks.k1.type = hdfs
     a2.sinks.k1.hdfs.path = hdfs://hadoop102:9000/flume2/%Y%m%d/%H
     #上传文件的前缀
     a2.sinks.k1.hdfs.filePrefix = flume2-
     #是否按照时间滚动文件夹
     a2.sinks.k1.hdfs.round = true
     #多少时间单位创建一个新的文件夹
     a2.sinks.k1.hdfs.roundValue = 1
     #重新定义时间单位
     a2.sinks.k1.hdfs.roundUnit = hour
     #是否使用本地时间戳
     a2.sinks.k1.hdfs.useLocalTimeStamp = true
     #积攒多少个Event才flush到HDFS一次
     a2.sinks.k1.hdfs.batchSize = 100
     #设置文件类型，可支持压缩
     a2.sinks.k1.hdfs.fileType = DataStream
     #多久生成一个新的文件
     a2.sinks.k1.hdfs.rollInterval = 600
     #设置每个文件的滚动大小大概是128M
     a2.sinks.k1.hdfs.rollSize = 134217700
     #文件的滚动与Event数量无关
     a2.sinks.k1.hdfs.rollCount = 0
     
     # Describe the channel
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
     a2.channels.c1.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ~~~

6. 创建flume-flume-dir.conf

   - 配置上级Flume输出的Source，输出是到本地目录的Sink。

     ~~~ java
     # Name the components on this agent
     a3.sources = r1
     a3.sinks = k1
     a3.channels = c2
     
     # Describe/configure the source
     a3.sources.r1.type = avro
     a3.sources.r1.bind = hadoop102
     a3.sources.r1.port = 4142
     
     # Describe the sink
     a3.sinks.k1.type = file_roll
     a3.sinks.k1.sink.directory = /opt/module/data/flume3
     
     # Describe the channel
     a3.channels.c2.type = memory
     a3.channels.c2.capacity = 1000
     a3.channels.c2.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a3.sources.r1.channels = c2
     a3.sinks.k1.channel = c2
     //提示：输出的本地目录必须是已经存在的目录，如果该目录不存在，并不会创建新的目录。
     ~~~

7. 执行配置文件

   - 分别开启对应配置文件：flume-flume-dir，flume-flume-hdfs，flume-file-flume。

   ~~~ java
   bin/flume-ng agent --conf conf/ --name a3 --conf-file job/group1/flume-flume-dir.conf
   
   bin/flume-ng agent --conf conf/ --name a2 --conf-file job/group1/flume-flume-hdfs.conf
   
   bin/flume-ng agent --conf conf/ --name a1 --conf-file job/group1/flume-file-flume.conf
   ~~~

8. 启动Hadoop和Hive

   ~~~ java
   sbin/start-dfs.sh
   sbin/start-yarn.sh
   bin/hive
   hive (default)>
   ~~~

9. 检查/opt/module/datas/flume3目录中数据

### 2.6，**单数据源**多出口案例(Sink组)

- 单Source、Channel多Sink(负载均衡)

  ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123144-540352.png)

  1. 案例需求：使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3也负责存储到HDFS 

     ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123147-987451.png)

  2. 在/opt/module/flume/job目录下创建group2文件夹

  3. 创建flume-netcat-flume.conf

     ~~~java
     //配置1个接收日志文件的source和1个channel、两个sink，分别输送给flume-flume-console1和flume-flume-console2
     # Name the components on this agent
     a1.sources = r1
     a1.channels = c1
     a1.sinkgroups = g1
     a1.sinks = k1 k2
     
     # Describe/configure the source
     a1.sources.r1.type = netcat
     a1.sources.r1.bind = localhost
     a1.sources.r1.port = 44444
     
     a1.sinkgroups.g1.processor.type = load_balance//负载平衡
     a1.sinkgroups.g1.processor.backoff = true
     a1.sinkgroups.g1.processor.selector = round_robin
     a1.sinkgroups.g1.processor.selector.maxTimeOut=10000
     
     # Describe the sink
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop102
     a1.sinks.k1.port = 4141
     
     a1.sinks.k2.type = avro
     a1.sinks.k2.hostname = hadoop102
     a1.sinks.k2.port = 4142
     
     # Describe the channel
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 1000
     a1.channels.c1.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a1.sources.r1.channels = c1
     a1.sinkgroups.g1.sinks = k1 k2
     a1.sinks.k1.channel = c1
     a1.sinks.k2.channel = c1
     ~~~

     注：

     - Avro是由Hadoop创始人Doug Cutting创建的一种语言无关的数据序列化和RPC框架。

     - RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

  4. 创建flume-flume-console1.conf

     - 配置上级Flume输出的Source，输出是到本地控制台。

       ~~~ java
       # Name the components on this agent
       a2.sources = r1
       a2.sinks = k1
       a2.channels = c1
       
       # Describe/configure the source
       a2.sources.r1.type = avro
       a2.sources.r1.bind = hadoop101
       a2.sources.r1.port = 4141
       
       # Describe the sink
       a2.sinks.k1.type = logger
       
       # Describe the channel
       a2.channels.c1.type = memory
       a2.channels.c1.capacity = 1000
       a2.channels.c1.transactionCapacity = 100
       
       # Bind the source and sink to the channel
       a2.sources.r1.channels = c1
       a2.sinks.k1.channel = c1
       ~~~

  5. 创建flume-flume-console2.conf

     - 配置上级Flume输出的Source，输出是到本地控制台。

       ~~~ java
       # Name the components on this agent
       a3.sources = r1
       a3.sinks = k1
       a3.channels = c2
       
       # Describe/configure the source
       a3.sources.r1.type = avro
       a3.sources.r1.bind = hadoop101
       a3.sources.r1.port = 4142
       
       # Describe the sink
       a3.sinks.k1.type = logger
       
       # Describe the channel
       a3.channels.c2.type = memory
       a3.channels.c2.capacity = 1000
       a3.channels.c2.transactionCapacity = 100
       
       # Bind the source and sink to the channel
       a3.sources.r1.channels = c2
       a3.sinks.k1.channel = c2
       ~~~

  6. 执行配置文件

     - 分别开启对应配置文件：flume-flume-console2，flume-flume-console1，flume-netcat-flume。

       ~~~ java
       bin/flume-ng agent --conf conf/ --name a3 --conf-file job/group2/flume-flume-console2.conf -Dflume.root.logger=INFO,console
       
       bin/flume-ng agent --conf conf/ --name a2 --conf-file job/group2/flume-flume-console1.conf -Dflume.root.logger=INFO,console
       
       bin/flume-ng agent --conf conf/ --name a1 --conf-file job/group2/flume-netcat-flume.conf
       ~~~

  7. 使用netcat工具向本机的44444端口发送内容

     ~~~java
     nc localhost 44444
     ~~~

### 2.7，**多数据源汇总案例**

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123156-977504.png)

1. 案例需求：

   hadoop103上的Flume-1监控文件/opt/module/group.log，

   hadoop102上的Flume-2监控某一个端口的数据流，

   Flume-1与Flume-2将数据发送给hadoop104上的Flume-3，Flume-3将最终数据打印到控制台。

   ![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123200-85798.png)

2. 准备工作

   ~~~ java
   //分发Flume
   xsync flume
   //在hadoop102、hadoop103以及hadoop104的/opt/module/flume/job目录下创建一个group3文件夹。
   mkdir group3
   ~~~

3. 创建flume1-logger-flume.conf

   - 在hadoop103上创建配置文件并打开

   ~~~ java
   //配置Source用于监控hive.log文件，配置Sink输出数据到下一级Flume。
   //在hadoop103上创建配置文件并打开
   # Name the components on this agent
   a1.sources = r1
   a1.sinks = k1
   a1.channels = c1
   
   # Describe/configure the source
   a1.sources.r1.type = exec
   a1.sources.r1.command = tail -F /opt/module/group.log
   a1.sources.r1.shell = /bin/bash -c
   
   # Describe the sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop104
   a1.sinks.k1.port = 4141
   
   # Describe the channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   # Bind the source and sink to the channel
   a1.sources.r1.channels = c1
   a1.sinks.k1.channel = c1
   ~~~

4. 创建flume2-netcat-flume.conf

   - 配置Source监控端口44444数据流，配置Sink数据到下一级Flume：

     在hadoop102上创建配置文件并打开

     ~~~ java
     # Name the components on this agent
     a2.sources = r1
     a2.sinks = k1
     a2.channels = c1
     
     # Describe/configure the source
     a2.sources.r1.type = netcat
     a2.sources.r1.bind = hadoop102
     a2.sources.r1.port = 44444
     
     # Describe the sink
     a2.sinks.k1.type = avro
     a2.sinks.k1.hostname = hadoop104
     a2.sinks.k1.port = 4141
     
     # Use a channel which buffers events in memory
     a2.channels.c1.type = memory
     a2.channels.c1.capacity = 1000
     a2.channels.c1.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a2.sources.r1.channels = c1
     a2.sinks.k1.channel = c1
     ~~~

5. 创建flume3-flume-logger.conf

   - 配置source用于接收flume1与flume2发送过来的数据流，最终合并后sink到控制台。

     在hadoop104上创建配置文件并打开

     ~~~ java
     # Name the components on this agent
     a3.sources = r1
     a3.sinks = k1
     a3.channels = c1
     
     # Describe/configure the source
     a3.sources.r1.type = avro
     a3.sources.r1.bind = hadoop104
     a3.sources.r1.port = 4141
     
     # Describe the sink
     # Describe the sink
     a3.sinks.k1.type = logger
     
     # Describe the channel
     a3.channels.c1.type = memory
     a3.channels.c1.capacity = 1000
     a3.channels.c1.transactionCapacity = 100
     
     # Bind the source and sink to the channel
     a3.sources.r1.channels = c1
     a3.sinks.k1.channel = c1
     ~~~

6. 执行配置文件

   - 分别开启对应配置文件：flume3-flume-logger.conf，flume2-netcat-flume.conf，flume1-logger-flume.conf。

     ~~~ java
     bin/flume-ng agent --conf conf/ --name a3 --conf-file job/group3/flume3-flume-logger.conf -Dflume.root.logger=INFO,console
     
     bin/flume-ng agent --conf conf/ --name a2 --conf-file job/group3/flume2-netcat-flume.conf
     
     bin/flume-ng agent --conf conf/ --name a1 --conf-file job/group3/flume1-logger-flume.conf
     ~~~

7. 在hadoop103上向/opt/module目录下的group.log追加内容

   ~~~ java
   echo 'hello' > group.log
   ~~~

8. 在hadoop102上向44444端口发送数据

   ~~~ java
   telnet hadoop102 44444
   ~~~


## 第三章，企业面试

### 3.1，**你是如何实现**Flume数据传输的监控的

- 使用第三方框架Ganglia实时监控Flume。

### 3.2，**F**lume的Source，Sink，Channel的作用？你们Source是什么类型？

1. 作用
   1. Source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy
   2. Channel组件对采集到的数据进行缓存，可以存放在Memory或File中。
   3. Sink组件是用于把数据发送到目的地的组件，目的地包括Hdfs、Logger、avro、thrift、ipc、file、Hbase、solr、自定义。

### 3.3，**F**lume的Channel **S**electors

![](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/12/123257-794160.png)

### 3.4，**F**lume参数调优

1. Source

   增加Source个数（使用Tair Dir Source时可增加FileGroups个数）可以增大Source的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个Source 以保证Source有足够的能力获取到新产生的数据。

   batchSize参数决定Source一次批量运输到Channel的event条数，适当调大这个参数可以提高Source搬运Event到Channel时的性能。

2. Channel 

   type 选择memory时Channel的性能最好，但是如果Flume进程意外挂掉可能会丢失数据。type选择file时Channel的容错性更好，但是性能上会比memory channel差。

   使用file Channel时dataDirs配置多个不同盘下的目录可以提高性能。

   Capacity 参数决定Channel可容纳最大的event条数。transactionCapacity 参数决定每次Source往channel里面写的最大event条数和每次Sink从channel里面读的最大event条数。transactionCapacity需要大于Source和Sink的batchSize参数。

3. Sink 

   增加Sink的个数可以增加Sink消费event的能力。Sink也不是越多越好够用就行，过多的Sink会占用系统资源，造成系统资源不必要的浪费。

   batchSize参数决定Sink一次批量从Channel读取的event条数，适当调大这个参数可以提高Sink从Channel搬出event的性能。

### 3.5，**F**lume的事务机制

Flume的事务机制（类似数据库的事务机制）：

Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。

比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。

### 3.6，**Flume采集数据会丢失吗?**

不会，Channel存储可以存储在File中，数据传输自身有事务。















   



