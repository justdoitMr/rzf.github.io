HBase
<!-- TOC -->

- [HBase简介](#hbase简介)
  - [HBase 定义](#hbase-定义)
  - [HBase数据模型](#hbase数据模型)
  - [HBase的逻辑结构](#hbase的逻辑结构)
  - [HBase 物理存储结构](#hbase-物理存储结构)
  - [数据模型](#数据模型)
    - [Name Space](#name-space)
    - [Region](#region)
    - [Row](#row)
    - [Column](#column)
    - [Time Stamp](#time-stamp)
    - [Cell](#cell)
  - [HBase基本架构](#hbase基本架构)
- [HBase 快速入门](#hbase-快速入门)
  - [HBase 安装部署](#hbase-安装部署)
    - [Zookeeper 正常部署](#zookeeper-正常部署)
    - [Hadoop 正常部署](#hadoop-正常部署)
    - [HBase 部署](#hbase-部署)
    - [HBase 远程发送到其他集群](#hbase-远程发送到其他集群)
    - [HBase 服务的启动](#hbase-服务的启动)
    - [查看 HBase 页面](#查看-hbase-页面)
  - [HBase Shell 操作](#hbase-shell-操作)
    - [基本操作](#基本操作)
    - [表的操作](#表的操作)
    - [DML操作](#dml操作)
- [HBase高级](#hbase高级)
  - [HBase架构原理](#hbase架构原理)
  - [HBase写数据流程](#hbase写数据流程)
    - [MemStore Flush](#memstore-flush)
  - [数据读流程](#数据读流程)
  - [StoreFile Compaction](#storefile-compaction)
  - [Region Split](#region-split)
- [HBase的API操作](#hbase的api操作)
  - [环境准备](#环境准备)
  - [HBase的API操作](#hbase的api操作-1)
    - [DDL操作](#ddl操作)
      - [判断表是否存在API](#判断表是否存在api)
      - [创建表API](#创建表api)
      - [删除表API](#删除表api)
      - [关闭资源API](#关闭资源api)
      - [创建命名空间](#创建命名空间)
      - [代码小结](#代码小结)
    - [DML操作](#dml操作-1)
      - [插入数据](#插入数据)
      - [获取数据](#获取数据)
      - [扫描全表](#扫描全表)

<!-- /TOC -->
## HBase简介

### HBase 定义

HBase 是一种分布式、可扩展、支持海量数据存储的 NoSQL 数据库。

### HBase数据模型

逻辑上，HBase 的数据模型同关系型数据库很类似，数据存储在一张表中，有行有列。
但从 HBase 的底层物理存储结构（K-V）来看，HBase 更像是一个 multi-dimensional map。

### HBase的逻辑结构

![1616727865285](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/113753-827036.png)

### HBase 物理存储结构

![1616729924546](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616729924546.png)

### 数据模型

#### Name Space

命名空间，类似于关系型数据库的 DatabBase 概念，每个命名空间下有多个表。HBase有两个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default 表是用户默认使用的命名空间。

可以简单的把命名空间理解为mysql中的数据库。

#### Region

类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明**列族**即可，不需要声明具体的列。这意味着，往 HBase 写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景。

#### Row

HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey的**字典顺序**存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要。

#### Column

HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要。

#### Time Stamp 

用于标识数据的不同版本（version），每条数据写入时，如果不指定时间戳，系统会自动为其加上该字段，其值为写入 HBase 的时间。除了版本和值不同以外，其他的部分必须相同

#### Cell

由{rowkey, column Family：column Qualifier, time Stamp} 唯一确定的单元。cell 中的数据是没有类型的，全部是字节码形式存贮。

![1616731515536](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/26/120516-10077.png)

> 一个大表可以切分为多个Region,Region在表中叫做切片，接下来是列族，列族下面是列，接下来是值，但是值可能是覆盖其他数据的值，所以在HBase要想确定一个数据，需要根据列族，列row_key和时间戳可以唯一确定一条数据记录。根据这些条件唯一确定的数据记录又叫做一个cell。单元格中所有的数据都没有数据类型，在底层全部是字节数组存储。表和Region有对应关系，因为一张表可以进行横向和纵向的切分操作，形成切片，最小可以切分为cell。

### HBase基本架构

**不完整版架构图**

![1616732277685](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616732277685.png)

- Region Server 
  - Region Server 为 Region 的管理者，其实现类为 HRegionServer，主要作用如下:
  - 对于数据的操作：get, put, delete；
  - 对于 Region 的操作：splitRegion、compactRegion。
  - 主要管理DML操作，也就是主要对数据进行操作。
- Master 
  - Master 是所有 Region Server 的管理者，其实现类为 HMaster，主要作用如下：
  - 对于表的操作：create, delete, alter,也就是对所有表的元数据进行管理。
  - 对于 RegionServer的操作：分配 regions到每个RegionServer，监控每个 RegionServer的状态，负载均衡和故障转移。如果master某一个时间挂掉的话，那么做数据级别的增删改查是可以的，但是不可以做表级别的修改。
  - 主要管理DDL的操作，对表进行操作。
  - master自带HA高可用机制。
- Zookeeper 
  - HBase 通过 Zookeeper 来做 Master 的高可用、RegionServer 的监控、元数据的入口以及集群配置的维护等工作。
- HDFS 
  - HDFS 为 HBase 提供最终的底层数据存储服务，同时为 HBase 提供高可用的支持。

## HBase 快速入门

### HBase 安装部署

#### Zookeeper 正常部署

首先保证 Zookeeper 集群的正常部署，并启动之：

~~~ java
[rzf@hadoop100 zookeeper-3.4.10]$ bin/zkServer.sh start
~~~

#### Hadoop 正常部署

Hadoop 集群的正常部署并启动：

~~~ java
//在这里只需要启动namenode即可，不需要启动yarn
[rzf@hadoop100 hadoop-2.7.2]$ sbin/start-dfs.sh 
~~~

#### HBase 部署

**解压HBase**

1. 解压 Hbase 到指定目录：

**HBase 的配置文件**

修改 HBase 对应的配置文件。

1. hbase-env.sh 修改内容：

~~~ java
export JAVA_HOME=/opt/module/jdk1.6.0_144
//设置关闭hbase自带的高可用机制，使用外部的zookeeper高可用机制
export HBASE_MANAGES_ZK=false 
~~~

2. hbase-site.xml 修改内容：

~~~ java
<configuration>
<property>
<name>hbase.rootdir</name>
<value>hdfs://hadoop100:9000/HBase</value>
</property>
<property>
<name>hbase.cluster.distributed</name>
<value>true</value>
</property>
 <!-- 0.98 后的新变动，之前版本没有.port,默认端口为 60000 -->
<property>
<name>hbase.master.port</name>
<value>16000</value>
</property>
<property> 
<name>hbase.zookeeper.quorum</name>
 <value>hadoop100,hadoop101,hadoop102</value>
 </property>
<property> 
<name>hbase.zookeeper.property.dataDir</name>
 <value>/opt/module/zookeeper-3.4.10/data/zkData</value>
</property>
</configuration>
~~~

3. regionservers

~~~ java
hadoop100
hadoop101
hadoop102
~~~

4. 软连接 hadoop 配置文件到 HBase：

~~~ java
[rzf@hadoop100 module]$ ln -s /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml /opt/module/hbase/conf/core-site.xml
[rzf@hadoop100 module]$ ln -s /opt/module/hadoop-2.7.2/etc/hadoop/hdfs-site.xml /opt/module/hbase/conf/hdfs-site.xml
~~~

####  HBase 远程发送到其他集群

~~~ java
xsync hbase
~~~

#### HBase 服务的启动

1. 启动方式一

~~~ java
//在哪一个节点上启动，那一个节点就作为master节点
[rzf@hadoop100 hbase]$ bin/hbase-daemon.sh start master
//启动regionserver进程
[rzf@hadoop100 hbase]$ bin/hbase-daemon.sh start regionserver
~~~

- 提示：如果集群之间的节点时间不同步，会导致 regionserver 无法启动，抛出ClockOutOfSyncException 异常。
  修复提示：

  - 同步时间服务
  - 属性：hbase.master.maxclockskew 设置更大的值

~~~ java
<property>
 <name>hbase.master.maxclockskew</name>
 <value>180000</value>
 <description>Time difference of regionserver from 
master</description>
</property>
~~~

2. 启动方式二，群起集群

~~~ java
//启动集群
[rzf@hadoop100 hbase]$ bin/start-hbase.sh 
//停止集群
[rzf@hadoop100 hbase]$ bin/stop-hbase.sh 
~~~

#### 查看 HBase 页面

查看端口是16010，hbase内部服务器端口是16000

~~~ java
http://hadoop100:16010/master-status
~~~

### HBase Shell 操作

#### 基本操作

1. 进入 HBase 客户端命令行

~~~ java
[rzf@hadoop100 hbase]$ bin/hbase shell
~~~

2. 查看帮助命令

~~~ java
hbase(main):005:0> help
~~~

3. 查看当前数据库中有哪些表 

~~~ java
//默认是看不到系统表的
hbase(main):006:0> list
~~~

#### 表的操作

1. 创建表

创建表的时候，至少有一个列族信息

~~~ java
//创建一个student表并且查看，create 后面直接添加表名，表名后面紧跟着的是列族
hbase(main):007:0> create 'student','info'
0 row(s) in 2.8880 seconds

=> Hbase::Table - student
hbase(main):008:0> list
TABLE                                                                                                                               
student                                                                                                                             
1 row(s) in 0.0220 seconds

=> ["student"]

//创建具有两个列族的表
hbase(main):009:0> create 'stu','info1','info2'
~~~

**创建的表在hbase的web界面上可以查看到信息**

![1616820051835](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/124053-235200.png)

2. 查看表的结构

~~~ java
hbase(main):012:0> describe 'student'
Table student is ENABLED                                                                                                            
student                                                                                                                             
COLUMN FAMILIES DESCRIPTION                                                                                                         
{NAME => 'info', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 
'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE 
=> '0'}                                                                                                                             
1 row(s) in 0.2540 seconds

hbase(main):013:0> describe 'stu'
Table stu is ENABLED                                                                                                                
stu                                                                                                                                 
COLUMN FAMILIES DESCRIPTION                                                                                                         
{NAME => 'info1', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE
 => '0'}                                                                                                                            
{NAME => 'info2', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE
 => '0'}                                                                                                                            
2 row(s) in 0.0500 seconds
~~~

3. 向表中插入数据

~~~ java

~~~

4. 修改表的信息

列族是最小的修改单元

~~~ java
//将 info1 列族中的数据存放 3 个版本：也就是版本修改为3
hbase(main):003:0> alter 'stu',{NAME=>'info1',VERSIONS=>3}
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 3.8080 seconds

//查看表结构，发现info1列族版本为3
hbase(main):004:0> describe 'stu'
Table stu is ENABLED                                                                                                                
stu                                                                                                                                 
COLUMN FAMILIES DESCRIPTION                                                                                                         
{NAME => 'info1', BLOOMFILTER => 'ROW', VERSIONS => '3', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE
 => '0'}                                                                                                                            
{NAME => 'info2', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING =>
 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE
 => '0'}                                                                                                                            
2 row(s) in 0.0560 seconds
~~~

5. 删除表

删除表的话先要使表变为不在线的状态

~~~ java
//使表变为不在线状态
hbase(main):005:0> disable 'student'
//然后在删除表
  hbase(main):006:0> drop 'student'
0 row(s) in 2.3680 seconds

hbase(main):007:0> list
TABLE                                                                                                                               
stu                                                                                                                                 
1 row(s) in 0.0170 seconds

=> ["stu"]
~~~

在web界面也看不到表的信息

![1616820202967](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616820202967.png)

6. 查看命名空间，相当于数据库

~~~ java
hbase(main):008:0> list_namespace
NAMESPACE                                                                                                                           
default                                                                                                                             
hbase   

//创建命名空间
hbase(main):009:0> create_namespace 'bigTable'
0 row(s) in 0.9330 seconds

hbase(main):010:0> list_namespace
NAMESPACE                                                                                                                           
bigTable                                                                                                                            
default                                                                                                                             
hbase                                                                                                                               
3 row(s) in 0.0450 seconds

~~~

7. 在命名空间下创建一张表

~~~ java
hbase(main):011:0> create "bigTable:stu","info"
0 row(s) in 2.3340 seconds

=> Hbase::Table - bigTable:stu

//查看
hbase(main):012:0> list
TABLE                                                                                                                               
bigTable:stu                                                                                                                        
stu                                                                                                                                 
2 row(s) in 0.0170 seconds

=> ["bigTable:stu", "stu"]
~~~

如果表的前面没有命名空间名字，那么就是说表在default命名空间下

8. 删除命名空间

不能删除非空的命名空间，要想删除，必须先把命名空间中的表全部下线，然后在删除命名空间

~~~ java
//先让命名空间下的表全部下线
 hbase(main):013:0> disable "bigTable:stu"
0 row(s) in 2.3380 seconds
//然后在删除命名空间下的表
hbase(main):014:0> drop "bigTable:stu"
0 row(s) in 1.3060 seconds

hbase(main):015:0> list
TABLE                                                                                                                               
stu                                                                                                                                 
1 row(s) in 0.0190 seconds

=> ["stu"]

//最后在删除命名空间
hbase(main):016:0> drop_namespace "bigTable"
0 row(s) in 0.9300 seconds

hbase(main):017:0> list_namespace
NAMESPACE                                                                                                                           
default                                                                                                                             
hbase                   
~~~

#### DML操作

1. 向表中put数据

~~~ java
//需要四个参数信息
hbase> put 'ns1:t1', 'r1', 'c1', 'value'
'ns1:t1'：命名空间下的表名
r1:表示row_KEY
c1:表示列信息，需要指出列族，列族：列名
value:插入的数据

//向stu表的info1列族下的name列插入名字张三
hbase(main):021:0> put "default:stu",'1001','info1:name',"张三"
  
  
//插入一些数据
  hbase(main):034:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                       
 1001                              column=info1:sex, timestamp=1616821744141, value=male                                            
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing  
 
 //有多少个row_key就有多少条数据
 3 row(s) in 0.0580 second
~~~

2. 查询数据

- 第一种查询，直接扫描全表

~~~ java
hbase(main):022:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89  //发生中文乱码
 
 //scan也可以进行范围扫描,区间是左闭右开
 hbase(main):040:0> scan 'stu',{STARTROW=>'1001',STOPROW=>'1003'}
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                       
 1001                              column=info1:sex, timestamp=1616821744141, value=male                                            
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456 
   
//重新插入数据 10010
   hbase(main):041:0> put 'stu','10010','info1:name','banzhang'
0 row(s) in 0.0280 seconds

hbase(main):042:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                       
 1001                              column=info1:sex, timestamp=1616821744141, value=male                                            
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing   
 //插入是按照row_key的字典顺序进行插入
~~~

- 第二种查询，使用get

~~~ java
hbase(main):024:0> get 'stu','1001'
COLUMN                             CELL                                                                                             
 info1:name                        timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89   
 
 //get案例
 hbase> t.get 'r1'
  hbase> t.get 'r1', {TIMERANGE => [ts1, ts2]}
  hbase> t.get 'r1', {COLUMN => 'c1'}
  hbase> t.get 'r1', {COLUMN => ['c1', 'c2', 'c3']}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
  hbase> t.get 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
  hbase> t.get 'r1', 'c1'
  hbase> t.get 'r1', 'c1', 'c2'
  hbase> t.get 'r1', ['c1', 'c2']
  hbase> t.get 'r1', {CONSISTENCY => 'TIMELINE'}
  hbase> t.get 'r1', {CONSISTENCY => 'TIMELINE', REGION_REPLICA_ID => 1}

//查询row_key=1001的数据，显示的是三个列，每一个列都对应有自己的时间戳
hbase(main):035:0> get 'stu','1001'
COLUMN                             CELL                                                                                             
 info1:name                        timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                                          
 info1:sex                         timestamp=1616821744141, value=male                                                              
 info2:addr1                       timestamp=1616821799187, value=shanghai 
 
//也可以指定查看某一个列族下的列
hbase(main):037:0> get 'stu','1001','info1:name'
COLUMN                             CELL                                                                                             
 info1:name                        timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89  
 
//查看指定列族下的所有列和其值
hbase(main):038:0> get 'stu','1001','info1'
COLUMN                             CELL                                                                                             
 info1:name                        timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                                          
 info1:sex                         timestamp=1616821744141, value=male                  
~~~

> get可以根据row_key查看某一条数据的全部信息，可以指定：列族:列查看某列的信息，也可以只指定：列族查看列族下的某条数据全部信息

从hdfs上查看我们存储的数据

![1616823092330](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/133135-111775.png)

Region下面是我们的列族信息，一个Region下面可以有多个列族信息

![1616823120043](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/27/133200-304356.png)

3. 修改数据

hbase里面没有直接的alter更改数据操作，但是可以通过put对数据记性更新操作

~~~  java
//通过put更新操作修改数据
hbase(main):043:0> put 'stu','1001','info1:name','zhangsansan'
0 row(s) in 0.0280 seconds

hbase(main):044:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616823351586, value=zhangsansan                                    
 1001                              column=info1:sex, timestamp=1616821744141, value=male                                            
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing        //但是此时数据并没有被真正的删除，VERSIONS => 10表示查看10个版本内的数据
   hbase(main):046:0> scan 'stu', {RAW => true, VERSIONS => 10}
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616823351586, value=zhangsansan                                    
 1001                              column=info1:name, timestamp=1616821229741, value=\xE5\xBC\xA0\xE4\xB8\x89                       
 1001                              column=info1:sex, timestamp=1616821744141, value=male                                            
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:name, timestamp=1616821919699, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1002                              column=info1:phone1, timestamp=1616821919738, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing                                        
4 row(s) in 0.0320 seconds
//但是使用get查询的话只能查到最新的数据

//修改时间戳插入一条数据
hbase(main):048:0> put 'stu','1001','info1:name','lisansan' ,1616823351587
0 row(s) in 0.0330 seconds
//查询数据
hbase(main):050:0> get 'stu','1001'
COLUMN                             CELL                                                                                             
 info1:name                        timestamp=1616823351587, value=lisansan                                                          
 info1:sex                         timestamp=1616821744141, value=male                                                              
 info2:addr1                       timestamp=1616821799187, value=shanghai 
 //可以看到，返回的是最新时间戳的数据，也就是我们修改数据的时候，时间戳一定要比之前的时间戳新，因为get每一次返回的是最新时间戳的数据
~~~

4. 删除数据

~~~ java
delete 'ns1:t1', 'r1', 'c1', ts1
ns1:t1:某一个命名空间下的一张表
r1:表示row_key信息
c1:表示列名
ts1:表示时间戳，可以没有

//删除row_key为1001的sex信息
hbase(main):052:0> delete 'stu','1001','info1:sex'
0 row(s) in 0.0600 seconds

hbase(main):053:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616823351587, value=lisansan                                       
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing 
 
 //现在name属性有三个版本，删除name后发现三个版本全部删除，因为插入的时候是覆盖操作，所以如果删除那么前面的版本数据也会全部删除
 hbase(main):054:0> delete 'stu','1001','info1:name'
0 row(s) in 0.0190 seconds

hbase(main):055:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing   
~~~

在查询name列的时候，发现已经打上Delete标记，所以就不会返回结果信息

![1616824585092](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616824585092.png)

现在根据已经删除的列的时间戳去查询，发现无法查询

~~~ java
get 'stu','1001','info1:sex',1616824168593
~~~

现在在name列最新的时间戳之前插入一条数据,查询之后发现无法查出，由此看出hbase是根据时间戳查询的

~~~ java
hbase(main):063:0> put 'stu','1001','info1:name','baozhu',1616824265643
0 row(s) in 0.0310 seconds

hbase(main):064:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616824895193, value=1616824265643                                  
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 10010                             column=info1:name, timestamp=1616822799179, value=banzhang                                       
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing                                        
4 row(s) in 0.0300 seconds
~~~

**根据数据的时间戳删除数据**

~~~ java
//根据时间戳删除数据，时间戳必须大于等于数据的时间戳，如果小于，无法删除数据
hbase(main):065:0> delete 'stu','10010','info1:name',1616822799179
0 row(s) in 0.0230 seconds

hbase(main):066:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1001                              column=info1:name, timestamp=1616824895193, value=1616824265643                                  
 1001                              column=info2:addr1, timestamp=1616821799187, value=shanghai                                      
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing 
~~~

> 注意：在命令行中如果只指出列族，而不指出具体的列信息，那么是无法删除数据的，但是在api中可以进行删除

deleteall可以根据指定的row_key删除全部记录信息

~~~ java
hbase> deleteall 'ns1:t1', 'r1'
  ns1:t1:表示某一个命名空间下的表
  r1:表示row_key
  
 hbase(main):069:0> deleteall 'stu','1001'
0 row(s) in 0.0160 seconds

hbase(main):070:0> scan 'stu'
ROW                                COLUMN+CELL                                                                                      
 1002                              column=info1:name, timestamp=1616821926859, value=lisi                                           
 1002                              column=info1:phone1, timestamp=1616821930949, value=123456                                       
 1003                              column=info2:addr, timestamp=1616821970138, value=beijing 
~~~

如果表中还有数据，想直接删除表的话使用truncate语句

~~~ java
hbase(main):071:0> truncate 'stu'
~~~

> 所有的增删改查都是根据时间戳进行的

5. 查询多个版本数据信息

~~~ java
//创建表
hbase(main):071:0> create 'stu2','info'
//插入数据
hbase(main):073:0> put 'stu2','1005','info:name','zhangsan'
//修改表的信息,版本修改为2
hbase(main):076:0> alter 'stu2',{NAME=>'info',version=>2}
//重新插入数据，只不过是两个版本
hbase(main):077:0> put 'stu2','1001','info:name','zhangsan'
0 row(s) in 0.0430 seconds

hbase(main):078:0> put 'stu2','1001','info:name','lisi'
0 row(s) in 0.0200 seconds

//指定查询多少个版本的数据
hbase(main):084:0> get 'stu2','1001',{COLUMN=>'info:name',VERSIONS=>3}
~~~

> 版本说明：修改后列族的版本后，就是表示将来hbase为我们存储几个版本，查询时候指定的版本数量要和设定的版本数量一致，否则无法查询，比如设置的是5个版本，那么将来hbase就为我们存储5个版本的数据，如果我们put进去的数据版本数大于5的话，其他的版本hbase认为是将来会删除的，不会为我们存储，我们设定多少个版本，hbase为我们存储多少个版本。
>
> 一句话说明：就是如果你put了10个版本的数据，但是设定的版本数是5，那么hbase只为我们保存5个版本的数据，其他5个版本最终会删除，保存的版本一定是时间戳最新的。

## HBase高级

### HBase架构原理

**结构图**

![1616828162065](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1616828162065.png)

HBase两个核心的组件，HMastermHRegionServer两个进程，HBase依赖于zookeeper和hdfs。

Hmaster管理ddl操作，master决定region给哪一个RegionServer进行维护

RegionServer管理DML操作。

HLogs相当于hdfs中的Edits文件,叫做预写入文件，目的是防止内存挂掉，所以专门用来记录操作，进行数据的恢复，他会记录每一步操作，写入hdfs中。

一个RegionServer中可以有很多Regions

HRegion和表有对应关系，一张表有一个或者多个Regions,按照行进行切分出来的

Store就相当于表中的列族，Store在hdfs上对应的是文件夹，可以类比hive中分区的划分，可以加速查询，查询数据可以扫描对应的文件夹即可，不需要全表扫描，一个Region下面可以有多个列族，也就是sTORE，存储结构是以文件夹的形式存在

列族下面存储的是数据，对于HBase来说，列族下面的列就相当于数据，因为列是在插入数据时候进行指定的。

存储数据在内存或者磁盘，从内存到磁盘的过程叫做Flush操作，刷写一次会形成一个文件，所以会产生很多小文件，小文件会自动进行合并操作，如果文件过大，还会进行再一次的切分操作。

最后存储到磁盘，文件会以HFile格式进行存储，是以键值对形式存储，HFile是一种文件格式。

刷写过程中会调用hdfs的客户端把数据存储在hdfs上面，这一些列的操作都是由RegionServer进行完成的2，HLog会一直进行落盘操作

如果做的是DML操作，那么即使master挂掉也没问题，因为DML操作是和zookeeper进行交互的，这就是zookeeper帮助master分担的工作。但是DDL操作的话，master挂掉之后就无法操作。

所有的操作会首先连接zookeeper，

- StoreFile 
  保存实际数据的物理文件，StoreFile 以 HFile 的形式存储在 HDFS 上。每个 Store 会有一个或多个 StoreFile（HFile），数据在每个 StoreFile 中都是有序的。
- MemStore 
  写缓存，由于 HFile 中的数据要求是有序的，所以数据是先存储在 MemStore 中，排好序后，等到达刷写时机才会刷写到 HFile，每次刷写都会形成一个新的 HFile。 
- WAL 
  由于数据要经 MemStore 排序后才能刷写到 HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入 MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。

### HBase写数据流程

HBase是一个读比写入慢的框架

![1617003040368](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/153859-975661.png)

- Client 先访问 zookeeper，获取 hbase:meta 表位于哪个 Region Server。客户端访问数据首先和zookeeper进行交互操作。
- 访问对应的 Region Server，获取 hbase:meta 表，也就是获取命名空间的位置，根据读请求的 namespace:table/rowkey，查询出目标数据位于哪个 Region Server 中的哪个 Region 中。并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。
- 与目标 Region Server 进行通讯；也就是访问目标RegionServer
- 将数据顺序写入（追加）到 WAL；首先写入日志文件中，然后在预写到内存中。但是实际上写入复杂的多
- 将数据写入对应的 MemStore，数据会在 MemStore 进行排序；
- 向客户端发送 ack；
- 等达到 MemStore 的刷写时机后，将数据刷写到 HFile。

#### MemStore Flush

![1617005680075](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/29/161440-241014.png)

每一次进行刷写操作都会形成一个文件，同一个Region写的不同store是不同的列族，不同的列族刷写到hdfs上面是不同的文件夹。最后合并文件的时候只会在相同的列族下面合并文件。

**MemStore 刷写时机：**

1. 当某个 memstroe 的大小达到了 hbase.hregion.memstore.flush.size（默认值 128M），其所在 region 的所有 memstore 都会刷写。
2. 当 memstore 的大小达到了hbase.hregion.memstore.flush.size（默认值 128M）hbase.hregion.memstore.block.multiplier（默认值 4）时，会阻止继续往该 memstore 写数据。这个时候往往是因为写入的速度太快，所以要阻止写入操作。
3. 当 region server 中 memstore 的总大小达到java_heapsize，hbase.regionserver.global.memstore.size（默认值 0.4）hbase.regionserver.global.memstore.size.lower.limit（默认值 0.95），region 会按照其所有memstore 的大小顺序（由大到小）依次进行刷写。直到 region server中所有 memstore 的总大小减小到上述值以下。
4. 当 region server 中 memstore 的总大小达java_heapsize*hbase.regionserver.global.memstore.size（默认值 0.4）时，会阻止继续往所有的 memstore 写数据。
5. 到达自动刷写的时间，也会触发 memstore flush。自动刷新的时间间隔由该属性进行
   配置 hbase.regionserver.optionalcacheflushinterval（默认 1 小时）。
6. 当 WAL 文件的数量超过 hbase.regionserver.max.logs，region 会按照时间顺序依次进行刷写，直到 WAL 文件数量减小到 hbase.regionserver.max.log 以下（该属性名已经废弃，现无需手动设置，最大值为 32）。

### 数据读流程

![1617008718053](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/30/142542-741974.png)

**读数据流程**

- Client 先访问 zookeeper，获取 hbase:meta 表位于哪个 Region Server。也就是先获取命名空间找到元数据表的存储位置，所有表的元数据信息都存储在meta表中。
- 访问对应的 Region Server，获取 hbase:meta 表，根据读请求的 namespace:table/rowkey，查询出目标数据位于哪个 Region Server 中的哪个 Region 中。并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache，方便下次访问。
- 与目标 Region Server 进行通讯；
- 分别在 Block Cache（读缓存），MemStore 和 Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的（Put/Delete）。
- 将从文件中查询到的数据块（Block，HFile 数据存储单元，默认大小为 64KB）缓存到Block Cache。
- 将合并后的最终结果返回给客户端。 

> 注意：读取的数据的时候是根据时间戳来的，不管是读取内存，还是读取磁盘，永远读取的是最新的数据，也就是时间戳最大的数据，会把内存中的数据和磁盘中的数据拿出来进行比较操作，返回时间戳最大的数据，也就是最新的数据。每一次都进行磁盘的扫描，时间很慢，所以就添加block cache缓存空间。block cache仅仅对磁盘中的文件有效，新写入的数据一定会写入洗盘，然后下一次读取数据的时候，会首先看block cache中是否有数据，如果有的话直接就返回，否则就从磁盘中读取，然后再把从磁盘中读取的数据添加到block cache文件中，供下次读取使用。
>
> 读比写慢的原因是无论如何都要进行读磁盘操作。

从内存把数据刷到hdfs上面

~~~ java
flush 'stu'
~~~

### StoreFile Compaction

- 由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（Put/Delete）有可能会分布在不同的 HFile 中，因此查询时需要遍历所有的 HFile。为了减少 HFile 的个数，以及清理掉过期和删除的数据，会进行 StoreFile Compaction。
- Compaction 分为两种，分别是 Minor Compaction 和 Major Compaction。Minor Compaction
  会将临近的若干个较小的 HFile 合并成一个较大的 HFile，但不会清理过期和删除的数据。
  Major Compaction 会将一个 Store 下的所有的 HFile 合并成一个大 HFile，并且会清理掉过期和删除的数据。但是不会立即清理掉所有合并过的文件，因为需要保证数据的一致性问题，所以还会等待一会，然后才会删除合并过的文件。

![1617087893462](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/30/150455-476504.png)

> 从上面可以发现，数据的读写全程没有master的参与，所以如果仅仅进行dml的操作，那么就不需要master参与也是可以完成的，但是长期这样做很不安全，因为当数据量大的情况下，一个表会切分为很多的Region，这样，如果master不可以工作，那么所有的Region会分布在一台机器上，所以不是很安全，因为所有的Region需要被master调度到其他的节点上面存储。

**删除数据**

22集

在大合并的时候，会删除数据，小合并不会删除数据

Flush或者major大合并的时候会删除数据

- flush删除数据只有数据在内存中的话才会删除，不能删除跨越多个文件的数据，flush仅仅会操作内存，不会操作磁盘。
- 合并操作会把所有的数据加载到内存当中，然后重写的时候，会把过时的数据删除，也就是删除时间戳小的数据。
- 删除标记是在大合并的时候删除

### Region Split

默认情况下，每个 Table 起初只有一个 Region，随着数据的不断写入，Region 会自动进行拆分。刚拆分时，两个子 Region 都位于当前的 Region Server，但处于负载均衡的考虑，HMaster 有可能会将某个 Region 转移给其他的 Region Server。

**Region Split 时机：**

- 当1个region中的某个Store下所有StoreFile的总大小超过hbase.hregion.max.filesize，该 Region 就会进行拆分（0.94 版本之前）。
- 当 1 个 region 中 的 某 个 Store 下所有 StoreFile 的 总 大 小 超 过 Min(R^2 * 
  "hbase.hregion.memstore.flush.size",hbase.hregion.max.filesize")，该 Region 就会进行拆分，其
  中 R 为当前 Region Server 中属于该 Table 的个数（0.94 版本之后）。

![1617093353939](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202103/30/163554-61650.png)

## HBase的API操作

### 环境准备

**添加依赖**

~~~ java
 <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.3.1</version>
        </dependency>
~~~

### HBase的API操作

#### DDL操作

~~~ java
//获取配置信息，使用静态代码块初始化配置信息
public class TestApi01 {

    private static Connection connection=null;
    private static Admin admin=null;

//    初始化静态变量
    static {

        try {
//        使用新的api创建配置信息
          //使用 HBaseConfiguration 的单例方法实例化
            Configuration conf = HBaseConfiguration.create();
//        配置集群信息,zookeeper默认使用的客户端端口是2181，所以在这里不需要进行配置
            conf.set("hbase.zookeeper.quorum","hadoop100,hadoop101,hadoop102");
//        使用新的api操作
            connection = ConnectionFactory.createConnection(conf);

            // 使用connection创建admin对象,admin对象是操作表的，也就是对应ddl语言
            admin = connection.getAdmin();
        }catch (Exception e){
            e.printStackTrace();
        }

    }
}
~~~

##### 判断表是否存在API

~~~ java
/**
     * 判断表是否存在
     * @param tableName 表名
     * @return 返回boolean表示表是否存在
     */
    public static boolean isExists(String tableName) throws IOException {
//        使用全局的admin对象
        /*
        执行程序，首先会加载执行静态代码块，然后admin connection两个对象就创建出来了
         */
        boolean exists = admin.tableExists(TableName.valueOf(tableName));


        return exists;
    }
~~~

##### 创建表API

创建表必须提供表名和列族信息

~~~ java
private static void createTable(String tableName,String ...arg ) throws IOException {
//       1 首先判断是否存在列族信息
       if(arg.length <= 0){
           System.out.println("请设置列族信息 ");
           return;
       }
//       2 判断表是否存在
       if(isExists(tableName)){
           System.out.println("表已经存在");
           return;
       }

//       3 创建一个表描述器
       HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf(tableName));

//       4 添加列族信息
       for (String temp:arg) {
//           创建一个列族描述器
           HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(temp);
//           添加列族信息
           descriptor.addFamily(hColumnDescriptor);
       }
//      5 创建表
       admin.createTable(descriptor);

   }
~~~

##### 删除表API

~~~ java
 /*
   删除表操作
    */
   private static void deleTable(String tableName) throws IOException {
//       首先判断表是否存在
       if(!isExists(tableName)){
           System.out.println("表不存在");
           return;
       }
//       然后使表下线
       admin.disableTable(TableName.valueOf(tableName));

//       删除表
       admin.deleteTable(TableName.valueOf(tableName));
   }

~~~

##### 关闭资源API

~~~ java
 /**
     * 单独关闭资源
     */
   public static void close() throws IOException {
        if(admin != null){
            admin.close();
        }

        if(connection != null){
            connection.close();
        }
   }

//测试代码

    public static void main(String[] args) throws IOException {
//        测试表是否存在
//        String tableName="stu";
//        boolean exists = isExists(tableName);
//        System.out.println(exists);
//        创建表测试
//        createTable("stud","info1","info2");

//        删除表测试
        deleTable("stud");
//        关闭资源
        close();
    }
~~~

##### 创建命名空间

~~~ java
  public static void createNameSpace(String ns)  {
//       先创建一个builder，通过builder去创建一个namespace
//        创建命名空间描述器
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(ns);
        NamespaceDescriptor build = builder.build();

//        创建命名空间
        try {
            admin.createNamespace(build);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
~~~

##### 代码小结

~~~ java
public class TestApi01 {

    private static Connection connection=null;
    private static Admin admin=null;

//    初始化静态变量
    static {

        try {
//        使用新的api创建配置信息
            Configuration conf = HBaseConfiguration.create();
//        配置集群信息,zookeeper默认使用的客户端端口是2181，所以在这里不需要进行配置
            conf.set("hbase.zookeeper.quorum","hadoop100,hadoop101,hadoop102");
//        使用新的api操作
            connection = ConnectionFactory.createConnection(conf);

            // 使用connection创建admin对象,admin对象是操作表的，也就是对应ddl语言
            admin = connection.getAdmin();
        }catch (Exception e){
            e.printStackTrace();
        }

    }
    

    /*
    DDL:
        1 判断表是否存在
        2 创建表
        3 创建命名空间
        4 表的增和删

    DMNL操作
        1 数据的增删改查 get scan
     */


    /**
     * 判断表是否存在
     * @param tableName 表名
     * @return 返回boolean表示表是否存在
     */
    public static boolean isExists(String tableName) throws IOException {
//        使用全局的admin对象
        /*
        执行程序，首先会加载执行静态代码块，然后admin connection两个对象就创建出来了
         */
        boolean exists = admin.tableExists(TableName.valueOf(tableName));


        return exists;
    }

    /**
     * 单独关闭资源
     */
   public static void close() throws IOException {
        if(admin != null){
            admin.close();
        }

        if(connection != null){
            connection.close();
        }
   }

   private static void createTable(String tableName,String ...arg ) throws IOException {
//       1 首先判断是否存在列族信息
       if(arg.length <= 0){
           System.out.println("请设置列族信息 ");
           return;
       }
//       2 判断表是否存在
       if(isExists(tableName)){
           System.out.println("表已经存在");
           return;
       }

//       3 创建一个表描述器
       HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf(tableName));

//       4 添加列族信息
       for (String temp:arg) {
//           创建一个列族描述器
           HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(temp);
//           添加列族信息
           descriptor.addFamily(hColumnDescriptor);
       }
//      5 创建表
       admin.createTable(descriptor);

   }

   /*
   删除表操作
    */
   private static void deleTable(String tableName) throws IOException {
//       首先判断表是否存在
       if(!isExists(tableName)){
           System.out.println("表不存在");
           return;
       }
//       然后使表下线
       admin.disableTable(TableName.valueOf(tableName));

//       删除表
       admin.deleteTable(TableName.valueOf(tableName));
   }

    public static void main(String[] args) throws IOException {
//        测试表是否存在
//        String tableName="stu";
//        boolean exists = isExists(tableName);
//        System.out.println(exists);
//        创建表测试
//        createTable("stud","info1","info2");

//        删除表测试
//        deleTable("stud");

//        创建一个命名空间
        createNameSpace("ns");

//        关闭资源
        close();
    }

    public static void createNameSpace(String ns)  {
//       先创建一个builder，通过builder去创建一个namespace
//        创建命名空间描述器
        NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(ns);
        NamespaceDescriptor build = builder.build();

//        创建命名空间
        try {
            admin.createNamespace(build);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~

#### DML操作

##### 插入数据

~~~ java
/**
     * 向表中插入数据
     * @param tableName 表名
     * @param rowKey 键
     * @param cf 列族
     * @param cn 列名
     * @param value 值
     */
    public static void insertData(String tableName,String rowKey,String cf,String cn
    ,String value) throws IOException {
//        首先获取表对象
        Table tab = connection.getTable(TableName.valueOf(tableName));

//        创建put对象
        Put put = new Put(Bytes.toBytes(rowKey));
//        给put对象进行赋值操作
        put.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn),Bytes.toBytes(value));

//        插入数据
        tab.put(put);
    }
~~~

##### 获取数据

~~~ java
 /**
     * 获取单行数据
     * @param tableName 表名
     * @param rowKey rowkey
     * @param cf 列族
     * @param cn 列名
     */
    public static void getData(String tableName,String rowKey,String cf,String cn) throws IOException {

//        获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

//        创建get对象
        Get get = new Get(Bytes.toBytes(rowKey));

//        获取指定的列族,里面的参数表示列族信息
        get.addFamily(Bytes.toBytes(cf));

//        指定列族和列
        get.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn));

//        设置获取数据的版本数,这里可以根据数据的版本数量进行自己设置
        get.setMaxVersions();


//        调用get方法获取数据
        Result res = table.get(get);

//        解析res结果,获取cell数组
        Cell[] cells = res.rawCells();

//        遍历cell打印数据,cell是由多个行组成的
        for(Cell s:cells){
            System.out.println("cf"+Bytes.toString(CellUtil.cloneFamily(s))
            +"cn"+Bytes.toString((CellUtil.cloneQualifier(s)))
            +"value"+Bytes.toString(CellUtil.cloneValue(s)));
        }

//        关闭表连接
        table.close();

    }
~~~

##### 扫描全表

~~~ java
/**
     * 获取数据（scan)
     */

    public static void scanTab(String tableName) throws IOException {
//        1 获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

//        2 创建一个scan对象,空参数表示扫描的是全表
        Scan scan = new Scan();

//        扫描全表
        ResultScanner scanner = table.getScanner(scan);

//        4 解析返回的结果
        for (Result result : scanner) {

//            解析并且打印
            for(Cell cell:result.rawCells()){
                System.out.println("cf"+Bytes.toString(CellUtil.cloneFamily(cell))
                        +"cn"+Bytes.toString((CellUtil.cloneQualifier(cell)))
                        +"value"+Bytes.toString(CellUtil.cloneValue(cell)));
            }

        }

//        关闭资源
        table.close();
    }
~~~

