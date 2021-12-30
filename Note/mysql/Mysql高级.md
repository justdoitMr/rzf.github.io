
<!-- TOC -->

- [Mysql架构介绍](#mysql架构介绍)
    - [Mysql简介](#mysql简介)
        - [Mysql概述](#mysql概述)
        - [Mysql高级学习方向](#mysql高级学习方向)
        - [Mysql的linus版本安装](#mysql的linus版本安装)
        - [Mysql主要配置文件](#mysql主要配置文件)
        - [Mysql逻辑架构](#mysql逻辑架构)
        - [Mysql存储引擎](#mysql存储引擎)
    - [基本概念](#基本概念)
        - [事务](#事务)
        - [锁粒度](#锁粒度)
- [索引优化分析](#索引优化分析)
    - [性能下降SQL慢，执行时间长，等待时间长](#性能下降sql慢执行时间长等待时间长)
    - [常见通用的Join查询](#常见通用的join查询)
        - [SQL执行顺序](#sql执行顺序)
        - [Join图](#join图)
        - [建表](#建表)
        - [7中Join理论](#7中join理论)
    - [索引简介](#索引简介)
        - [什么是索引](#什么是索引)
        - [索引的优势](#索引的优势)
        - [索引的劣势](#索引的劣势)
        - [索引的分类](#索引的分类)
        - [Mysql索引结构](#mysql索引结构)
            - [**B+Tree索引**](#btree索引)
            - [MyISAM索引实现](#myisam索引实现)
            - [Innodb索引原理](#innodb索引原理)
            - [聚簇索引与非聚簇索引](#聚簇索引与非聚簇索引)
            - [full-text全文索引](#full-text全文索引)
            - [Hash索引](#hash索引)
            - [R-Tree索引](#r-tree索引)
        - [可以使用B+树索引的查询类型](#可以使用b树索引的查询类型)
        - [B-树索引的限制](#b-树索引的限制)
        - [**那些情况需要建立索引**](#那些情况需要建立索引)
        - [**哪些情况不要创建索引**](#哪些情况不要创建索引)
    - [性能分析](#性能分析)
        - [可以使用B树索引的查询类型](#可以使用b树索引的查询类型)
        - [B-树索引的限制](#b-树索引的限制-1)
        - [MySQL常见瓶颈](#mysql常见瓶颈)
        - [Explain关键字](#explain关键字)
            - [是什么(查看执行计划)](#是什么查看执行计划)
            - [做什么用](#做什么用)
            - [如何使用](#如何使用)
            - [各字段解释](#各字段解释)
                - [id(重要)](#id重要)
                - [select_type](#select_type)
                - [table](#table)
                - [type（重要）](#type重要)
                - [possible_keys](#possible_keys)
                - [key(重要)](#key重要)
                - [key_len](#key_len)
                - [ref](#ref)
                - [rows（重要）](#rows重要)
                - [Extra](#extra)
    - [查询优化](#查询优化)
        - [单表查询优化](#单表查询优化)
        - [两表查询优化](#两表查询优化)
        - [三标查询优化](#三标查询优化)
        - [小结](#小结)
        - [使用索引](#使用索引)
            - [创建表](#创建表)
            - [全部匹配索引最好](#全部匹配索引最好)
            - [最佳左前缀法则](#最佳左前缀法则)
            - [不做计算](#不做计算)
            - [少用范围查找](#少用范围查找)
            - [使用覆盖索引](#使用覆盖索引)
            - [不等号的使用](#不等号的使用)
            - [is null](#is-null)
            - [like使用](#like使用)
            - [字符串不加单引号索引失效](#字符串不加单引号索引失效)
            - [少用or,用它来连接时会索引失效](#少用or用它来连接时会索引失效)
            - [小结](#小结-1)
        - [一般性建议](#一般性建议)
    - [查询截取分析](#查询截取分析)
        - [子查询优化](#子查询优化)
            - [order by关键字优化](#order-by关键字优化)
            - [双路排序](#双路排序)
            - [单路排序](#单路排序)
            - [使用单路排序优化策略](#使用单路排序优化策略)
            - [分页查询的优化---limit](#分页查询的优化---limit)
            - [GROUP BY关键字优化](#group-by关键字优化)
            - [去重优化](#去重优化)
    - [慢查询日志](#慢查询日志)
        - [**查看是否开启及如何开启**](#查看是否开启及如何开启)
        - [开启](#开启)
        - [日志分析工具](#日志分析工具)
    - [Show Profile](#show-profile)
        - [是什么](#是什么)
        - [分析步骤](#分析步骤)
    - [Mysql锁](#mysql锁)
        - [锁的分类](#锁的分类)
        - [表锁(偏读)](#表锁偏读)
        - [行锁(偏写)](#行锁偏写)
        - [优化建议](#优化建议)

<!-- /TOC -->

> 面试如何讲mysql优化：
> 
> 首先写出sql
> 
> 然后开启慢查询日志
> 
> 执行sql
> 
> 找出执行慢的sql
> 
> 使用exPlain执行计划进行分析
> 
> 最后使用profile进行分析
> 
> 提升硬件资源

## Mysql架构介绍

### Mysql简介

#### Mysql概述

1. MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。
2. MySQL是一种关联数据库管理系统，将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。
3. Mysql是开源的，所以你不需要支付额外的费用。
4. Mysql是可以定制的，采用了GPL协议，你可以修改源码来开发自己的Mysql系统。
5. Mysql支持大型的数据库。可以处理拥有上千万条记录的大型数据库。
6. MySQL使用标准的SQL数据语言形式。
7. Mysql可以允许于多个系统上，并且支持多种语言。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等。
8. MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。

#### Mysql高级学习方向

1. 数据库内部结构和原理（内部结构，层次结构）
2. 数据库建模优化
3. 数据库索引建立（重点）
4. SQL语句优化（重点）
5. SQL编程
6. mysql服务器的安装配置
7. 数据库的性能监控分析与系统优化
8. 各种参数常量设定
9. 主从复制
10. 分布式架构搭建、垂直切割和水平切割
11. 数据迁移
12. 容灾备份和恢复
13. shell或python等脚本语言开发
14. 对开源数据库进行二次开发

#### Mysql的linus版本安装

1. **官网下载地址**

~~~ java
http://dev.mysql.com/downloads/mysql/
~~~

2. **检查当前系统是否安装过mysql**

~~~ java
//检查是否安装
rpm -qa|grep mysql
//执行卸载 卸载所有依赖
rpm -e --nodeps  mysql-libs
~~~

3. **检查/tmp文件夹权限**

~~~ java
//由于mysql安装过程中，会通过mysql用户在/tmp目录下新建tmp_db文件，所以请给/tmp较大的权限
执行 ：chmod -R 777 /tmp
~~~

4. **安装mysql**

~~~ java
//在mysql的安装文件目录下执行：
rpm -ivh MySQL-server-5.5.54-1.linux2.6.x86_64.rpm
rpm -ivh MySQL-client-5.5.54-1.linux2.6.x86_64.rpm
~~~

5. 查看MySQL安装版本

~~~ java
//可以执行 mysqladmin --version命令，类似java -version如果打出消息，即为成功。
~~~

6. 查看用户组

~~~ java
//安装mysql后，系统会创建mysql用户组
//查看所有用户组
[rzf@hadoop01 etc]$ cat group
//查看是否有mysql用户组
[root@hadoop01 etc]# cat group |grep mysql
mysql:x:993:
~~~

7. mysql服务的启+停

~~~ java
//启动服务
service mysql start
//关闭服务
service mysql stop
//查看状态
service mysql status
~~~

8. 登录mysql服务

安装完成后会提示出如下的提示：在mysql首次登录前要给 root 账号设置密码

![1612012166353](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/30/210927-222083.png)

**设置密码**

~~~ java
//启动服务后，执行命令 
/usr/bin/mysqladmin -u root  password 'root'
//然后通过 mysql -uroot -proot进行登录
~~~

9. 设置mysql服务自启动

~~~ java
[root@hadoop01 etc]# chkconfig mysql on
//查看自动启动是否成功开启
[root@hadoop01 etc]# chkconfig --list
//也可以使用管道进行过滤
[root@hadoop01 etc]# chkconfig --list | grep mysql
[root@hadoop01 etc]# cat /etc/inittab 
~~~

**输出的六种状态**

![1612012601155](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/081226-829589.png)

**状态含义**

![1612012624112](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/30/211705-867086.png)

- 也就是只有2,3,4,5四种情况重启。

10. MySQL的安装位置

~~~ java
//在linux下查看安装目录  ps -ef|grep mysql
//如果你只要查询文件的运行文件所在地址，直接用下面的命令就可以了(还是以mysql为例)：
[root@hadoop01 /]# which mysql
~~~

| 参数         | 路径                      | 解释                         | 备注                       |
| ------------ | ------------------------- | ---------------------------- | -------------------------- |
| --basedir    | /usr/bin                  | 相关命令目录                 | mysqladmin mysqldump等命令 |
| --datadir    | /var/lib/mysql/           | mysql数据库文件的存放路径    |                            |
| --plugin-dir | /usr/lib64/mysql/plugin   | mysql插件存放路径            |                            |
| --log-error  | /var/lib/mysql/           | mysql错误日志路径            |                            |
| --pid-file   | /var/lib/mysql/           | 进程pid文件                  |                            |
| --socket     | /var/lib/mysql/mysql.sock | 本地连接时用的unix套接字文件 |                            |
|              | /usr/share/mysql          | 配置文件目录                 | mysql脚本及配置文件        |
|              | /etc/init.d/mysql         | 服务启停相关脚本             |                            |

**图示**

![1612013563664](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/30/213244-511258.png) 

> 查看mysql的安装位置：whereis mysql

11. 修改字符集问题

**查看字符集**

~~~ java
show variables like 'character%'; 
show variables like '%char%';

mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
//发现服务端的字符集是latin1，默认的是客户端和服务器都用了latin1，所以会乱码。

~~~

**修改my.cnf**

~~~ java
/*在/usr/share/mysql/ 中找到my.cnf的配置文件，
拷贝其中的my-huge.cnf 到 /etc/  并命名为my.cnf 
mysql 优先选中 /etc/ 下的配置文件
然后修改my.cnf:*/
//添加下面内容
[client]
default-character-set=utf8
[mysqld]
character_set_server=utf8
character_set_client=utf8
collation-server=utf8_general_ci
[mysql]
default-character-set=utf8

~~~

- 重新启动mysql，但是原库的设定不会发生变化，参数修改之对新建的数据库生效
- 已生成的库表字符集如何变更

~~~ java
//修改数据库的字符集
mysql> alter database mytest character set 'utf8';
//修改数据表的字符集
mysql> alter table user convert to  character set 'utf8';
//但是原有的数据如果是用非'utf8'编码的话，数据本身不会发生改变。
~~~

![1612016301805](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/30/221823-273496.png)

#### Mysql主要配置文件

1. 二进制日志log-bin
   - log-bin 中存放了所有的操作记录(写？)，可以用于恢复。相当于 Redis 中的 AOF，my.cnf中的log-bin配置(默认关闭)
2. 错误日志log-error
   - 默认是关闭的,记录严重的警告和错误信息，每次启动和关闭的详细信息等。
3. 慢查询日志log
   - 默认关闭，记录查询的sql语句，如果开启会减低mysql的整体性能，因为记录日志也是需要消耗系统资源的
   - 可自定义“慢”的概念:0-10秒之间的一个数。慢查询日志会将超过这个查询事件的查询记录下来，方便找到需要优化的 sql 。用于优化sql语句是使用。 
4. 数据文件
   - 两系统
     - windows
       - ....\MySQLServer5.5\data目录下很多数据库文件
     - linux
       - 默认路径：/var/lib/mysql
       - 可在配置文件中更改 /usr/share/mysql/  下的 my-huge.cnf,此文件已经被复制到/etc目录下面。
   - Myisam存放方式：show create table mydb 查看创建 mydb 表的基本信息，其中包括了数据引擎。自带的库 mysql 库中所有的表都是以 MyIsam 引擎存的。通过 myisam 引擎存的表都是 一式三份，放在库同名的文件夹下 /var/lib/mysql，mysql使用的是文件系统的目录和文件来保存数据库和表的定义，
     - frm文件(framework)：**存放表结构**，也可以说是scheme。
     - myd文件(data)：存放表数据
     - myi文件(index)：存放表索引
   - innodb存放方式
     - ibdata1：Innodb引擎将所有表的的数据都存在这里面 /usr/share/mysql/ibdata1  而frm文件存放在库同名的包下
     - frm文件：存放表结构
     - 单独存放

![1612052851875](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/082734-498545.png)

~~~ java
set innodb_file_per_table=on 
create table mmm (id int(20) auto_increment ,name varchar(20),primary key(id));
//设在为 on 后 单独以 table名.ibd 
//查询表的定义
mysql> show table status like 't1';
~~~

![1612052896706](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/082818-63829.png)

5. 如何配置
   - windows:my.ini文件(配置文件)
   - Linux:/etc/my.cnf文件(配置文件)

#### Mysql逻辑架构

和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。**主要体现在存储引擎的架构上，插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。**

**架构图**

![1612054333070](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/182010-878770.png)

1. 连接层

   最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

2. 服务层

   - Management Serveices & Utilities： 系统管理和控制工具  
   - **SQL Interface: SQL接口**，接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface
   - **Parser: 解析器**，SQL命令传递到解析器的时候会被解析器验证和解析，解析为解析树。 
   - **Optimizer**: 查询优化器。
     - SQL语句在查询之前会使用查询优化器对查询进行优化。 
     -  用一个例子就可以理解： select uid,name from user where  gender= 1;
     - 优化器来决定先投影还是先过滤。
   - **Cache和Buffer**： 查询缓存。
     - 如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。
     - 这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等
     - 缓存是负责读，缓冲负责写。
   - **引擎层**，存储引擎层，存储引擎真正的负责了MySQL中数据的**存储和提取**，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，上层的API屏蔽了不同存储引擎之间的差异，这样我们可以根据自己的实际需要进行选取。后面介绍MyISAM和InnoDB,mysql的创建索引是在存储引擎层实现的，而不是服务层面。
   - **存储层**，数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互。

   **查询过程**

   ![1612054827787](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/090029-951186.png)

**说明**

1. mysql客户端通过协议与mysql服务器建连接，发送查询语句，先检查查询缓存，如果命中(一模一样的sql才能命中)，直接返回结果，否则进行语句解析,也就是说，在解析查询之前，服务器会先访问查询缓存(query cache)——它存储SELECT语句以及相应的查询结果集。如果某个查询结果已经位于缓存中，服务器就不会再对查询进行解析、优化、以及执行。它仅仅将缓存中的结果返回给用户即可，这将大大提高系统的性能。
2. 语法解析器和预处理：首先mysql通过关键字将SQL语句进行解析，并生成一颗对应的“解析树”。mysql解析器将使用mysql语法规则验证和解析查询；预处理器则根据一些mysql规则进一步检查解析数是否合法。
3. 查询优化器当解析树被认为是合法的了，并且由优化器将其转化成执行计划。一条查询可以有很多种执行方式，最后都返回相同的结果。优化器的作用就是找到这其中最好的执行计划。
4. 然后，mysql默认使用的BTREE索引，并且一个大致方向是:无论怎么折腾sql，至少在目前来说，mysql最多只用到表中的一个索引。

#### Mysql存储引擎

InnoDB是Mysql默认的**事务性**存储引擎:
1. 他被用来设计处理大量的短期事务，短期事务中大部分情况是正常提交的，很少被回滚
2. InnoDB的数据存储在表空间中，表空间是InnoDB管理的一个黑盒子，由一系列的文件组成，在后续的版本中，Mysql将每一个表的数据和索引存储在单独的文件中
3. InnoDB默认是采用聚簇索引的，聚簇索引对主键的查询有很高的性能
4. InnoDB存储引擎对索引文件没有使用压缩的方式存储，而是按照原来的数据格式进行存储，但是MyISAM存储引擎对索引数据使用前缀压缩技术使得索引文件更小。

MyISAM存储引擎**不支持事务和行级锁**:
1. MyISAM存储引擎会将数据和索引存储在两个单独的文件中，数据文件和索引文件，
2. InnoDB存储引擎是根据主键的引用被索引的行，但是MyISAM是通过数据的物理位置引用被索引的行。

看你的mysql现在已提供什么存储引擎:

~~~ java
mysql> show engines;
~~~

![1612055016833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/090337-340791.png)

~~~ java
//第二种方式查看
mysql> show variables like '%storage_engine%';
//mysql中默认存储引擎是InnoDB
~~~

![1612055191216](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/090632-800800.png)

**MyISAM和InnoDB**

| 对比项         | MyISAM                                                   | InnoDB                                                       |
| -------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 主外键         | 不支持                                                   | 支持                                                         |
| 事务           | 不支持                                                   | 支持                                                         |
| 行表锁         | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 | 行锁,操作时只锁某一行，不对其它行有影响，适合高并发的操作    |
| 缓存           | 只缓存索引，不缓存真实数据                               | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间         | 小                                                       | 大                                                           |
| 关注点         | 性能（读数据更有利）                                     | 事务                                                         |
| 默认安装       | Y                                                        | Y                                                            |
| 用户表默认使用 | N                                                        | Y                                                            |
| 自带系统表使用 | Y                                                        | N                                                            |

> **innodb 索引 使用 B+TREE， myisam 索引使用 b+tree**
> innodb 主键为聚簇索引，基于聚簇索引的增删改查效率非常高。
>
> 为什么可以这样说，是因为innodb引擎吧数据存储在叶子节点，而非叶子节点只存储索引，所以执行增删改产的时候，只需要对叶子节点进行，并且查找一个节点很方便，但是myidam中非叶子节点和叶子节点都存储有值，对数据进行增删改产之后，还需要对树进行调整操作。

**阿里巴巴使用的存储引擎**

![1612055989199](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/091954-381917.png)

- Percona 为 MySQL 数据库服务器进行了改进，在功能和性能上较 MySQL 有着很显著的提升。该版本提升了在高负载情况下的 InnoDB 的性能、为 DBA 提供一些非常有用的性能诊断工具；另外有更多的参数和命令来控制服务器行为。
- 该公司新建了一款存储引擎叫xtradb完全可以替代innodb,并且在性能和并发上做得更好,
- 阿里巴巴大部分mysql数据库其实使用的percona的原型加以修改。
- AliSql+AliRedis（阿里巴巴对开源社区的贡献）

Mysql拥有分层的架构，上层是服务器层的服务和查询的执行引擎，下层则是存储引擎。

### 基本概念

#### 事务

事务就是一组原子性的sql语句，在一个事务中的语句，要么全部执行成功，要么全部执行失败。

**事务的ACID特性**

原子性：一个事务必须被视为一个不可分割的最小执行单元，整个事务中的所有操作要么全部提交执行成功，要么全部回滚失败，对于一个事务来说，不可能仅仅执行其中的一部分，这就是事务的原子性。

一致性：数据库总是可以从一种一致性状态转换到另一种一致性状态。

隔离性：一个事务所做的修改在最终提交之前，对其他的事务是不可见的。

持久性：一旦事务提交并且执行成功，他所做的修改就会永久的保存在数据库中。即使系统崩溃，修改的数据也不会丢失。

#### 锁粒度

表锁：表锁是Mysql数据库中最基本的锁策略，并且是开销最小的策略。读锁之间不会相互阻塞，而是共享的，也就是说读锁之间可以并发执行，读锁是共享的，但是读锁和写锁是相互互斥的，具有排他性，写锁比读锁有更高的优先级。

行级锁：行级锁可以最大程度上的支持并发处理，innodb存储引擎实现了行级锁，行级锁只是在存储引擎的层面进行实现的。

## 索引优化分析

### 性能下降SQL慢，执行时间长，等待时间长
1. **查询数据过多**

能不能拆，条件过滤尽量少

2. **关联了太多的表，太多join**

join 原理。用  A 表的每一条数据 扫描 B表的所有数据。所以尽量先过滤。

3. **没有利用到索引**

- 索引针对 列 建索引。但并不可能每一列都建索引
- 索引并非越多越好。当数据更新了，索引会进行调整。也会很消耗性能。（因为索引的底层是B+树，我们对索引进行调整，破坏了树的结构，所以始终需要调整满足B+树的特性，所以在调整树的过程中，也很耗费性能）
- 且 mysql 并不会把所有索引都用上，只会根据其算法挑一个索引用。所以建的准很重要。

- 单值索引：create index indexName on table(“列名”)
- 复合索引：create index indexName on table(“列名1”，“列名2”)
  - 条件多时，可以建共同索引(混合索引)。混合索引一般会偶先使用。
  - 有些情况下，就算有索引具体执行时也不会被使用。
- 索引命名规则：idx_表名_列名

4. **服务器调优及各个参数设置（缓冲、线程数等）(不重要DBA的工作)**

### 常见通用的Join查询

#### SQL执行顺序

通常我们程序员理解的sql执行顺序

![1612057833921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/135321-946786.png)

**mysql优化器理解的执行顺序**

随着Mysql版本的更新换代，其优化器也在不断的升级，优化器会分析不同执行顺序产生的性能消耗不同而动态调整执行顺序。

![1612057870908](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/135325-516749.png)

**小结**

sql执行顺序图

![1612057907121](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/095147-545574.png)

#### Join图

![min1612058158001](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/095559-591921.png)

~~~ sql
select <select_list> from TableA A inner join TableB B on A.key=B.key;

select <select_list> from TableA A left join TableB B on A.key=B.key;

select <select_list> from TableA A right join TableB B on A.key=B.key;

select <select_list> from TableA A left join TableB B on A.key=B.key where B.key is null;

select <select_list> from TableA A right join TableB B on A.key=B.key where A.key is null;

select <select_list> from TableA A full outer Join TableB B on A.key=B.key;

select <select_list> from TableA A full outer Join TableB B on A.key=B.key where A.key is null or B.key is null;
/*什么叫共有，什么叫独有？
共有：满足 a.deptid = b.id 的叫共有
A独有:  A 表中所有不满足  a.deptid = b.id  连接关系的数据
同时参考 join 图*/
~~~

#### 建表

~~~ sql
CREATE TABLE `t_dept` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `deptName` VARCHAR(30) DEFAULT NULL,
 `address` VARCHAR(40) DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
CREATE TABLE `t_emp` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `name` VARCHAR(20) DEFAULT NULL,
  `age` INT(3) DEFAULT NULL,
 `deptId` INT(11) DEFAULT NULL,
 PRIMARY KEY (`id`),
 KEY `fk_dept_id` (`deptId`)
 #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `t_dept` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
 
 
INSERT INTO t_dept(deptName,address) VALUES('华山','华山');
INSERT INTO t_dept(deptName,address) VALUES('丐帮','洛阳');
INSERT INTO t_dept(deptName,address) VALUES('峨眉','峨眉山');
INSERT INTO t_dept(deptName,address) VALUES('武当','武当山');
INSERT INTO t_dept(deptName,address) VALUES('明教','光明顶');
INSERT INTO t_dept(deptName,address) VALUES('少林','少林寺');
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('风清扬',90,1);
INSERT INTO t_emp(NAME,age,deptId) VALUES('岳不群',50,1);
INSERT INTO t_emp(NAME,age,deptId) VALUES('令狐冲',24,1);
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('洪七公',70,2);
INSERT INTO t_emp(NAME,age,deptId) VALUES('乔峰',35,2);
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('灭绝师太',70,3);
INSERT INTO t_emp(NAME,age,deptId) VALUES('周芷若',20,3);
 
 
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('张三丰',100,4);
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('张无忌',25,5);
 
INSERT INTO t_emp(NAME,age,deptId) VALUES('韦小宝',18,null);
--t_emp表
mysql> select * from t_emp;
+----+--------------+------+--------+
| id | name         | age  | deptId |
+----+--------------+------+--------+
|  1 | 风清扬       |   90 |      1 |
|  2 | 岳不群       |   50 |      1 |
|  3 | 令狐冲       |   24 |      1 |
|  4 | 洪七公       |   70 |      2 |
|  5 | 乔峰         |   35 |      2 |
|  6 | 灭绝师太     |   70 |      3 |
|  7 | 周芷若       |   20 |      3 |
|  8 | 张三丰       |  100 |      4 |
|  9 | 张无忌       |   25 |      5 |
| 10 | 韦小宝       |   18 |   NULL |
+----+--------------+------+--------+
-- t_dept
mysql> select * from t_dept;
+----+----------+-----------+
| id | deptName | address   |
+----+----------+-----------+
|  1 | 华山     | 华山      |
|  2 | 丐帮     | 洛阳      |
|  3 | 峨眉     | 峨眉山    |
|  4 | 武当     | 武当山    |
|  5 | 明教     | 光明顶    |
|  6 | 少林     | 少林寺    |
+----+----------+-----------+
~~~

#### 7中Join理论

1. A、B两表共有

~~~ sql
mysql> select * from t_emp inner join t_dept on t_emp.deptId=t_dept.id;
+----+--------------+------+--------+----+----------+-----------+
| id | name         | age  | deptId | id | deptName | address   |
+----+--------------+------+--------+----+----------+-----------+
|  1 | 风清扬       |   90 |      1 |  1 | 华山     | 华山      |
|  2 | 岳不群       |   50 |      1 |  1 | 华山     | 华山      |
|  3 | 令狐冲       |   24 |      1 |  1 | 华山     | 华山      |
|  4 | 洪七公       |   70 |      2 |  2 | 丐帮     | 洛阳      |
|  5 | 乔峰         |   35 |      2 |  2 | 丐帮     | 洛阳      |
|  6 | 灭绝师太     |   70 |      3 |  3 | 峨眉     | 峨眉山    |
|  7 | 周芷若       |   20 |      3 |  3 | 峨眉     | 峨眉山    |
|  8 | 张三丰       |  100 |      4 |  4 | 武当     | 武当山    |
|  9 | 张无忌       |   25 |      5 |  5 | 明教     | 光明顶    |
+----+--------------+------+--------+----+----------+-----------+
-- 可以给表添加别名，例如a,b，
select * from t_emp a inner join t_dept b on a.deptId = b.id;
~~~

2. A、B两表共有+A的独有

~~~ sql
mysql> select * from t_emp left join t_dept on t_emp.deptId=t_dept.id;
+----+--------------+------+--------+------+----------+-----------+
| id | name         | age  | deptId | id   | deptName | address   |
+----+--------------+------+--------+------+----------+-----------+
|  1 | 风清扬       |   90 |      1 |    1 | 华山     | 华山      |
|  2 | 岳不群       |   50 |      1 |    1 | 华山     | 华山      |
|  3 | 令狐冲       |   24 |      1 |    1 | 华山     | 华山      |
|  4 | 洪七公       |   70 |      2 |    2 | 丐帮     | 洛阳      |
|  5 | 乔峰         |   35 |      2 |    2 | 丐帮     | 洛阳      |
|  6 | 灭绝师太     |   70 |      3 |    3 | 峨眉     | 峨眉山    |
|  7 | 周芷若       |   20 |      3 |    3 | 峨眉     | 峨眉山    |
|  8 | 张三丰       |  100 |      4 |    4 | 武当     | 武当山    |
|  9 | 张无忌       |   25 |      5 |    5 | 明教     | 光明顶    |
| 10 | 韦小宝       |   18 |   NULL | NULL | NULL     | NULL      |
+----+--------------+------+--------+------+----------+-----------+
-- t_emp表中的记录的deptId不管是否是空，都会保留下来
~~~

3. A、B两表共有+B的独有

~~~ sql
mysql> select * from t_emp right join t_dept on t_emp.deptId=t_dept.id;    
+------+--------------+------+--------+----+----------+-----------+
| id   | name         | age  | deptId | id | deptName | address   |
+------+--------------+------+--------+----+----------+-----------+
|    1 | 风清扬       |   90 |      1 |  1 | 华山     | 华山      |
|    2 | 岳不群       |   50 |      1 |  1 | 华山     | 华山      |
|    3 | 令狐冲       |   24 |      1 |  1 | 华山     | 华山      |
|    4 | 洪七公       |   70 |      2 |  2 | 丐帮     | 洛阳      |
|    5 | 乔峰         |   35 |      2 |  2 | 丐帮     | 洛阳      |
|    6 | 灭绝师太     |   70 |      3 |  3 | 峨眉     | 峨眉山    |
|    7 | 周芷若       |   20 |      3 |  3 | 峨眉     | 峨眉山    |
|    8 | 张三丰       |  100 |      4 |  4 | 武当     | 武当山    |
|    9 | 张无忌       |   25 |      5 |  5 | 明教     | 光明顶    |
| NULL | NULL         | NULL |   NULL |  6 | 少林     | 少林寺    |
+------+--------------+------+--------+----+----------+-----------+
-- t_dept表中的记录的id不管在表t_emp中是否是空，都会保留下来，用null填充
~~~

4. A的独有

~~~ java
mysql> select * from t_emp left join t_dept on t_emp.deptId=t_dept.id where t_dept.id is null;
+----+-----------+------+--------+------+----------+---------+
| id | name      | age  | deptId | id   | deptName | address |
+----+-----------+------+--------+------+----------+---------+
| 10 | 韦小宝    |   18 |   NULL | NULL | NULL     | NULL    |
+----+-----------+------+--------+------+----------+---------+
~~~

5. B的独有

~~~ sql
mysql> select * from t_emp a right join t_dept b on a.deptId = b.id where a.deptId is null;  
+------+------+------+--------+----+----------+-----------+
| id   | name | age  | deptId | id | deptName | address   |
+------+------+------+--------+----+----------+-----------+
| NULL | NULL | NULL |   NULL |  6 | 少林     | 少林寺    |
+------+------+------+--------+----+----------+-----------+
~~~

6. AB全有

~~~ sql
-- MySQL Full Join的实现 因为MySQL不支持FULL JOIN,下面是替代方法
left join + union(可去除重复数据)+ right join
SELECT * FROM t_emp A LEFT JOIN t_dept B ON A.deptId = B.id
UNION
SELECT * FROM t_emp A RIGHT JOIN t_dept B ON A.deptId = B.id
--这里因为要联合的缘故，不能考虑到小表驱动大表的情况。只能用right join。要保证查询出来的数字要一致。
mysql> SELECT * FROM t_emp A LEFT JOIN t_dept B ON A.deptId = B.id
    -> UNION
    -> SELECT * FROM t_emp A RIGHT JOIN t_dept B ON A.deptId = B.id;
+------+--------------+------+--------+------+----------+-----------+
| id   | name         | age  | deptId | id   | deptName | address   |
+------+--------------+------+--------+------+----------+-----------+
|    1 | 风清扬       |   90 |      1 |    1 | 华山     | 华山      |
|    2 | 岳不群       |   50 |      1 |    1 | 华山     | 华山      |
|    3 | 令狐冲       |   24 |      1 |    1 | 华山     | 华山      |
|    4 | 洪七公       |   70 |      2 |    2 | 丐帮     | 洛阳      |
|    5 | 乔峰         |   35 |      2 |    2 | 丐帮     | 洛阳      |
|    6 | 灭绝师太     |   70 |      3 |    3 | 峨眉     | 峨眉山    |
|    7 | 周芷若       |   20 |      3 |    3 | 峨眉     | 峨眉山    |
|    8 | 张三丰       |  100 |      4 |    4 | 武当     | 武当山    |
|    9 | 张无忌       |   25 |      5 |    5 | 明教     | 光明顶    |
|   10 | 韦小宝       |   18 |   NULL | NULL | NULL     | NULL      |
| NULL | NULL         | NULL |   NULL |    6 | 少林     | 少林寺    |
+------+--------------+------+--------+------+----------+-----------+
~~~

7. A的独有+B的独有

~~~ sql
mysql> select * FROM t_emp A LEFT JOIN t_dept B ON A.deptId = B.id WHERE B.`id` IS NULL
    -> UNION
    -> SELECT * FROM t_emp A RIGHT JOIN t_dept B ON A.deptId = B.id WHERE A.`deptId` IS NULL;
+------+-----------+------+--------+------+----------+-----------+
| id   | name      | age  | deptId | id   | deptName | address   |
+------+-----------+------+--------+------+----------+-----------+
|   10 | 韦小宝    |   18 |   NULL | NULL | NULL     | NULL      |
| NULL | NULL      | NULL |   NULL |    6 | 少林     | 少林寺    |
+------+-----------+------+--------+------+----------+-----------+
~~~

### 索引简介

#### 什么是索引

- MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。可以得到索引的本质：**索引是数据结构**。是存储引擎用于快速查找到记录的一种数据结构。
- 在Mysql数据库中，**索引的实现是在引擎层而不是服务层**.
- 你可以简单理解为“**排好序的快速查找数据结构**”。
  - 在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，
  - 这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。下图就是一种可能的索引方式示例：

![1612097652519](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/205414-285856.png)

- 左边是数据表，一共有两列七条记录，最左边的是数据记录的物理地址
- 为了加快Col2的查找（这里是以col2列建立的索引），可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。
- 数据本身之外，数据库还维护着一个满足特定查找算法的数据结构，这些数据结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是索引。

> 二叉树弊端之一：二叉树很可能会发生两边不平衡的情况。
> B-TREE: (B:balance)  会自动根据两边的情况自动调节，使两端无限趋近于平衡状态。可以使性能最稳定。


- 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的**磁盘上**
- 我们平常所说的索引，如果没有特别指明，都是指B+树(多路搜索树，并不一定是二叉的)结构组织的索引。其中聚集索引，次要索引，覆盖索引，复合索引，前缀索引，唯一索引默认都是使用**B+树索引**，统称索引。当然，除了B+树这种类型的索引之外，还有哈稀索引(hash index)等。
- **索引有排序和查找数据两大功能**，解决where后面的字段查找速度问题和order by排序的时候如何查找的快的问题。

> 为什么说索引有排序和查找两大功能，这是根据索引底层的数据结构来说的，底层的B+树本身就是一颗多路搜索树，而树中间节点中的值，是相对有序的。


#### 索引的优势

- 类似大学图书馆建书目索引，提高数据检索的效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗

#### 索引的劣势

- 实际上索引也是一张表，该表保存了**主键与索引字段**，并指向实体表的记录，所以索引列也是要占用空间的。
- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。
  因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息
- **索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句**

#### 索引的分类

**单值索引**

- 即一个索引只包含单个列，一个表可以有多个单列索引

- 索引建立成哪种索引类型？
  - 根据数据引擎类型自动选择的索引类型
  - 除开 innodb 引擎主键默认为聚簇索引 外。 innodb 的索引都采用的 B+TREE
  - myisam 也都采用的 B+TREE索引

- 一般多值索引优于单值索引，一张表建立的索引最多不要超过5个。

**单值索引或者主键索引**

~~~ sql
--随表一起建索引：
CREATE TABLE customer (
id INT(10) UNSIGNED  AUTO_INCREMENT ,
customer_no VARCHAR(200),
customer_name VARCHAR(200),
PRIMARY KEY(id),
KEY (customer_name)  
);
--随表一起建立的索引 索引名同 列名(customer_name)
--单独建单值索引：
CREATE  INDEX idx_customer_name ON customer(customer_name); 
 
--删除索引：
DROP INDEX idx_customer_name ;
~~~

**唯一索引**

- 索引列的值必须唯一，但允许有空值

~~~ sql
--随表一起建索引：
CREATE TABLE customer (
	id INT(10) UNSIGNED  AUTO_INCREMENT ,
  customer_no VARCHAR(200),
  customer_name VARCHAR(200),
  PRIMARY KEY(id),
  KEY (customer_name),
  UNIQUE (customer_no)
);
--建立 唯一索引时必须保证所有的值是唯一的（除了null），若有重复数据，会报错。  
 
--单独建唯一索引：
CREATE UNIQUE INDEX idx_customer_no ON customer(customer_no); 
 
--删除索引：
DROP INDEX idx_customer_no on customer ;
~~~

**复合索引**

- 即一个索引包含多个列
- 在数据库操作期间，复合索引比单值索引所需要的开销更小(对于相同的多个列建索引)
- 当表的行数远大于索引列的数目时可以使用复合索引
- 复合索引与单值索引有什么区别？
  - 复合索引：create index idx_no_name on emp(no,name); 
  -  // no 与  name 有同一个索引 idx_no_name
  - 单值索引：create index idx_no on emp(no);
                     create index idx_name on emp(name);

~~~ sql
--随表一起建索引：
CREATE TABLE customer (
  id INT(10) UNSIGNED  AUTO_INCREMENT ,
  customer_no VARCHAR(200),
  customer_name VARCHAR(200),
  PRIMARY KEY(id),
  KEY (customer_name),
  UNIQUE (customer_name),
  KEY (customer_no,customer_name)
);
 
--单独建索引：
CREATE  INDEX idx_no_name ON customer(customer_no,customer_name); 
 
--删除索引：
DROP INDEX idx_no_name  on customer ;
~~~

**主键索引**

- 设定为主键后数据库会自动建立索引，innodb为聚簇索引

~~~ sql
--随表一起建索引：
CREATE TABLE customer (
  id INT(10) UNSIGNED  AUTO_INCREMENT ,-- 索引列
  customer_no VARCHAR(200),
  customer_name VARCHAR(200),
  PRIMARY KEY(id) -- 主键作为索引
);
unsigned (无符号的)
--使用  AUTO_INCREMENT 关键字的列必须有索引(只要有索引就行)。
 
CREATE TABLE customer2 (
  id INT(10) UNSIGNED   ,
  customer_no VARCHAR(200),
  customer_name VARCHAR(200),
  PRIMARY KEY(id) 
);
 
 --单独建主键索引：
ALTER TABLE customer 
 add PRIMARY KEY customer(customer_no);  
 
--删除建主键索引：
ALTER TABLE customer 
 drop PRIMARY KEY ;  
 
--修改建主键索引：
--必须先删除掉(drop)原索引，再新建(add)索引
~~~

**基本语法**

~~~sql
--创建索引
ALTER mytable ADD  [UNIQUE(表示唯一索引) ]  INDEX [indexName] ON (columnname(length)（如果多个列名，就是复合索引）) 
create [unique] index indexName on mytable(columnname(length))
-- 删除
DROP INDEX [indexName] ON mytable; 
-- 查看索引
SHOW INDEX FROM table_name\G
-- 索引命名规则
idx_表名_字段名
~~~

使用ALTER命令

~~~ sql
--有四种方式来添加数据表的索引：
ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): --该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
 
ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): --这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
 
ALTER TABLE tbl_name ADD INDEX index_name (column_list): --添加普通索引，索引值可出现多次。
 
ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):--该语句指定了索引为 FULLTEXT ，用于全文索引。
~~~

查看索引

![1612099758240](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/30/133209-189216.png)

- non_unique: 是否是唯一索引  1：是   0：不是
- seq_in_index:列在索引中的序列。针对符合索引(一个索引对应多个列)。针对同一个复合索引 按照创建复合索引时的顺序进行排序
- collation:
- cardinality:
- sub_part:
- packed:
- Null:是否允许 null 值
- comment:
- index_comment:

#### Mysql索引结构

大多数Mysql的存储引擎都支持B+Tree这种索引结构，MyISAM索引通过数据的物理位置引用被索引的行，InnoDB则是根据主键引用被索引的行。

##### **B+Tree索引**

![1612100529804](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202101/31/214211-778947.png)

【初始化介绍】 

- 一颗b树，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），
- 如磁盘块1包含数据项17和35，包含指针P1、P2、P3，
- P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。
- 真实的数据存在于叶子节点即3、5、9、10、13、15、28、29、36、60、75、79、90、99。
- 非叶子节点不存储真实的数据，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表中。

【查找过程】

- 如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。
- 真实的情况是，3层的b+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常高。

**时间复杂度**

同一问题可用不同算法解决，而一个算法的质量优劣将影响到算法乃至程序的效率。算法分析的目的在于选择合适算法和改进算法。

![1612100659289](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/30/133340-729596.png)

![1640842438388](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/30/133359-791458.png)

- N  logN 分别表示数据与查询次数之间的关系。
- 常数  1*c 表示查询最快的方式。查询次数不随数据的增加而增加*
- 变量 N 表示查询次数随数据数量的增加而增加
- *数 logN 表示查询次数与数据数量成对数关系。 介于常数与 N 之间。*
- *n*logN 表示使用的复合方法。

##### MyISAM索引实现

MyISAM引擎使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。下图是MyISAM索引的原理图：

![20211230133646](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211230133646.png)

这里设表一共有三列，假设我们以Col1为主键，则图是一个MyISAM表的主索引（Primary key）示意。可以看出MyISAM的索引文件仅仅保存数据记录的地址。在MyISAM中，**主索引和辅助索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复**。如果我们在Col2上建立一个辅助索引，则此索引的结构如下图所示：

![20211230133824](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211230133824.png)

同样也是一颗B+Tree，data域保存数据记录的地址。因此，MyISAM中索引检索的算法为首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址，读取相应数据记录。

MyISAM的索引方式也叫做“非聚集”的，之所以这么称呼是为了与InnoDB的聚集索引区分。

##### Innodb索引原理

虽然InnoDB也使用B+Tree作为索引结构，但具体实现方式却与MyISAM截然不同。

第一个重大区别是**InnoDB的数据文件本身就是索引文件**。从上文知道，MyISAM索引文件和数据文件是分离的，索引文件仅保存数据记录的地址。而在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。

![20211230133958](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211230133958.png)

图是InnoDB主索引（同时也是数据文件）的示意图，可以看到叶节点包含了完整的数据记录。这种索引叫做聚集索引。因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

第二个与MyISAM索引的不同是InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，InnoDB的所有辅助索引都引用主键作为data域。例如，图11为定义在Col3上的一个辅助索引：

![20211230134109](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211230134109.png)

这里以英文字符的ASCII码作为比较准则。聚集索引这种实现方式使得按主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

了解不同存储引擎的索引实现方式对于正确使用和优化索引都非常有帮助，例如知道了InnoDB的索引实现后，就很容易明白为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。再例如，用非单调的字段作为主键在InnoDB中不是个好主意，因为InnoDB数据文件本身是一颗B+Tree，非单调的主键会造成在插入新记录时数据文件为了维持B+Tree的特性而频繁的分裂调整，十分低效，而使用自增字段作为主键则是一个很好的选择。

**B树和B+树的区别**

B+Tree与B-Tree 的区别：结论在内存有限的情况下，B+TREE 永远比 B-TREE好。无限内存则后者方便

1. B树的关键字和记录是放在一起的，叶子节点可以看作外部节点，不包含任何信息；B+树叶子节点中只有关键字和指向下一个节点的索引，记录只放在叶子节点中。(一次查询可能进行两次i/o操作)，这也决定了两种树的高度不同，同一量级的数据，B+树种非叶子节点只存储索引，所以可以存储更多的数据，但是对于B树，节点不仅要存储索引，还要存储数据，所以B树建立欸都树更高。
2. 在B-树中，越靠近根节点的记录查找时间越快，只要找到关键字即可确定记录的存在；而B+树中每个记录的查找时间基本是一样的，都需要从根节点走到叶子节点，而且在叶子节点中还要再比较关键字。从这个角度看B-树的性能好像要比B+树好，而在实际应用中却是B+树的性能要好些。因为B+树的非叶子节点不存放实际的数据，这样每个节点可容纳的元素个数比B-树多，树高比B-树小，这样带来的好处是减少磁盘访问次数。尽管B+树找到一个记录所需的比较次数要比B-树多，但是一次磁盘访问的时间相当于成百上千次内存比较的时间，因此实际中B+树的性能可能还会好些，而且B+树的叶子节点使用指针连接在一起，方便顺序遍历（例如查看一个目录下的所有文件，一个表中的所有记录等），这也是很多数据库和文件系统使用B+树的缘故。 

思考：为什么说B+树比B-树更适合实际应用中操作系统的文件索引和数据库索引？ 

   - B+树的磁盘读写代价更低 
     B+树的内部结点并没有指向关键字具体信息的指针。因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。 
   - B+树的查询效率更加稳定 
     由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

##### 聚簇索引与非聚簇索引

- 聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。

- 术语‘聚簇’表示数据行和相邻的键值存储在一起。

- innodb存储引擎使用的是聚簇索引

- 如下图，**左侧的索引就是聚簇索引，因为数据行在磁盘的排列和索引排序保持一致。**

   ![1612101303481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/30/133405-344183.png)

**聚簇索引的好处：**

- 按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于数据都是紧密相连，数据库不用从多个数据块中提取数据，所以节省了大量的io操作。
- 可以把相关的数据全部保存在一起。
- 数据访问的更快聚簇索引将索引和数据保存在同一颗树中。
- 使用覆盖索引扫描的查询可以直接使用叶节点中的主键值

> 最重要的一点，讲索引和数据保存在一起，也就是如果你找到索引的话，那么等于说数据也找到了，因为两者是存储在一起的，但是非聚簇索引就不一样了，找到所以是获取到了数据的存放地址，还需要拿到这个地址去找数据。

**缺点**

- 聚簇索引最大限度的提高了io密集型应用的性能，但是如果数据全部存放在内存中，那么访问顺序就没有那么重要了，聚簇索引失去优势
- 插入数据严重依赖于插入顺序，如果是按照主键顺序插入，速度会很快，但是如果不是按照主键的顺序插入，有损失性能，因为INNODB引擎使用主键将数据聚集，也就是根据主键建立索引。
- 更新聚簇索引的代价很高，因为强制innodb将每一个被更新的行移动到新的位置

**聚簇索引的限制：**

- 对于mysql数据库目前只有innodb数据引擎支持聚簇索引，而Myisam并不支持聚簇索引。
- 由于数据物理存储排序方式只能有一种，所以每个Mysql的表只能有一个聚簇索引。一般情况下就是该表的主键。
- 为了充分利用聚簇索引的聚簇的特性，所以innodb表的主键列尽量选用有序的顺序id，而不建议用无序的id，比如uuid这种。（参考聚簇索引的好处。）

这里说明了主键索引为何采用自增的方式：

1、业务需求，有序。

2、能使用到聚簇索引

##### full-text全文索引

全文索引（也称全文检索）是目前搜索引擎使用的一种关键技术。它能够利用【分词技术】等多种算法智能分析出文本文字中关键词的频率和重要性，然后按照一定的算法规则智能地筛选出我们想要的搜索结果。

~~~ sql
CREATE TABLE `article` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(200) DEFAULT NULL,
  `content` text,
  PRIMARY KEY (`id`),
  FULLTEXT KEY `title` (`title`,`content`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
~~~

不同于like方式的的查询：

~~~ sql
SELECT * FROM article WHERE content LIKE ‘%查询字符串%’;
--全文索引用match+against方式查询：
SELECT * FROM article WHERE MATCH(title,content) AGAINST (‘查询字符串’);
~~~

明显的提高查询效率。

- 限制：
  - mysql5.6.4以前只有Myisam支持，5.6.4版本以后innodb才支持，但是官方版本不支持中文分词，需要第三方分词插件。
  - 5.7以后官方支持中文分词。
- 随着大数据时代的到来，关系型数据库应对全文索引的需求已力不从心，逐渐被 solr,elasticSearch等专门的搜索引擎所替代。

##### Hash索引

- Hash索引只有Memory（并且是唯一哈希索引）, NDB两种引擎支持，Memory引擎默认支持Hash索引，如果多个hash值相同，出现哈希碰撞，那么索引以链表方式存储。
- NoSql采用此索引结构。

**补充内容**

哈希索引是基于哈希表实现的，只有精确匹配所有列的查询才有效。对于每一行数据，存储引擎都会计算一个哈希码，不同键值得哈希码计算出来的值也不一样，哈希索引将所有的哈希码存储在索引中，同时在哈希表中保存指向每一个数据的行指针。

**哈希索引存在的局限性**

1. 哈希索引只包括哈希值和行指针，而不存储字段值，所以不能使用索引中的值来避免读取行
2. 哈希索引数据并不是按照索引的顺序读取的，所以也无法用于排序
3. 哈希索引也不支持部分索引列的匹配查找，因为哈希索引始终是使用索引列的全部内容来计算的哈希值
4. 哈希索引只支持等值比较查询，包括=，in(),<=>，也不支持任何的范围查询。
5. 访问哈希索引的数据非常快，除非是有很多哈希冲突，当有哈希冲突的时候，需要对链表逐个进行比较，所以会降低效率。
6. 如果哈希冲突很高的话，一些索引的维护操作代价也会很高。

##### R-Tree索引

- R-Tree在mysql很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有myisam、bdb、innodb、ndb、archive几种。
- 相对于b-tree，r-tree的优势在于范围查找。

**补充内容**

#### 可以使用B+树索引的查询类型

1. **全值匹配**，全值匹配值=指的是和单值索引或者复合索引中的列全部匹配。
2. **匹配最左前缀**：使用单值索引或者复合索引最左边的列索引。
3. **匹配列前缀**：可以匹配某一列的值得开头部分，一般用于laike的模糊查询，“abc%”，但是如果吧%放在左边，索引就失效了，不能匹配。
4. **匹配范围值**：匹配某一个范围内的值
5. **精确匹配某一列并且范围匹配另外一列**，也就是说某一类是精确匹配，但是另外若干列是范围匹配。

> B树通常可以支持“只访问索引的查询“，既查询只需要访问索引，而无需访问数据行

#### B-树索引的限制

1. 如果建立的是复合索引，但是如果查找不是从最左边开始查找，那么建立的复合索引会失效。
2. 建立复合索引的话，不能跳过复合索引的中间列，否则跳过的列后面的索引全部会失效。
3. 如果查询中某一个索引使用的是范围查找，那么此索引右面的索引列全部会失效。

#### **那些情况需要建立索引**

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引(where 后面的语句)
3. 查询中与其它表关联的字段，外键关系建立索引
   - A 表关联 B 表：A join B  。  on 后面的连接条件 既 A 表查询 B 表的条件。所以 B 表被关联的字段建立索引能大大提高查询效率，因为在 join 中，join 左边的表会用每一个字段去遍历 B 表的所有的关联数据，相当于一个查询操作
4. 单键/组合索引的选择问题，who？(在高并发下倾向创建组合索引)
5. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
   - group by 和 order by 后面的字段有索引大大提高效率
6. 查询中统计或者分组字段x。

#### **哪些情况不要创建索引**

1. 表**记录太少**
2. 经常**增删改**的表
   - Why:提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。
   - 因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件
3. Where条件里用不到的字段不创建索引
   - 索引建多了影响 增删改 的效率
4. 数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

![1612157627618](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/133348-271321.png)

### 性能分析

#### 可以使用B树索引的查询类型

1. 全值匹配，全值匹配值=指的是和单值索引或者复合索引中的列全部匹配。
2. 匹配最左前缀：使用单值索引或者复合索引最左边的列索引。
3. 匹配列前缀：可以匹配某一列的值得开头部分
4. 匹配范围值：匹配某一个范围内的值
5. 精确匹配某一列并且范围匹配另外一列，也就是说某一类是精确匹配，但是另外若干列是范围匹配。

> B树通常可以支持“只访问索引的查询“，既查询只需要访问索引，而无需访问数据行

#### B-树索引的限制

1. 如果建立的是复合索引，但是如果查找不是从最左边开始查找，那么建立的复合索引会失效。
2. 建立复合索引的话，不能跳过复合索引的中间列，否则跳过的列后面的索引全部会失效。
3. 如果查询中某一个索引使用的是范围查找，那么此索引右面的索引列全部会失效。

#### MySQL常见瓶颈

1. CPU

SQL中对大量数据进行比较、关联、排序、分组，最大的压力在于比较。

2. IO：

实例内存满足不了缓存数据或排序等需要，导致产生大量 物理 IO。

查询执行效率低，扫描过多数据行。

3. 锁

不适宜的锁的设置，导致线程阻塞，性能下降。

死锁，线程之间交叉调用资源，导致死锁，程序卡住。

4. 服务器硬件的性能瓶颈：top,free, iostat和vmstat来查看系统的性能状态

#### Explain关键字

##### 是什么(查看执行计划)

使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈

##### 做什么用

- 表的读取顺序<也就是表的id>
- 哪些索引可以使用<possible_keys>
- 数据读取操作的操作类型<select_type>
- 哪些索引被实际使用<keys>
- 表之间的引用<ref>
- 每张表有多少行被优化器查询<rows>

##### 如何使用

- Explain + SQL语句
- 执行计划包含的信息

![1612158587245](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/134948-128976.png)

> 重点是：
>
> id：子表和主表的读取顺序。
>
> type：查询数据的类型，尽量避免all全表扫描。
>
> key：查询种实际使用的索引。
>
> rows列显示MySQL认为它执行查询时必须检查的行数。越少越好
>
> **id**
>
> system：查询的结果只有一条数据，那么就是system级别。
>
> const:唯一索引或者主键索引查询是const，因为只有一条记录。
>
> eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描，通常表A和表B种的数据是一对一关系。
>
> ref:非唯一性索引扫描，返回匹配某个单独值的**所有行**.
> 本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体,我们在join字段上面建立索引。 
>
> range:只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引
> 一般就是在你的where语句中出现了between、<、>、in等的查询
> 这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。
>
> index:Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）,但是需要在查询字段上面建立索引。
>
> all:遍历全表扫描。
>
> index_merge：在查询过程中需要多个索引组合使用，通常出现在有 or 的关键字的sql中
>
> ref_or_null：对于某个字段既需要关联条件，也需要null值得情况下。查询优化器会选择用ref_or_null连接查询。
>
> index_subquery：利用索引来关联子查询，不再全表扫描，而是扫描一个范围。
>
> unique_subquery ：该联接类型类似于index_subquery。 子查询中的唯一索引。

##### 各字段解释

**建表语句**

~~~ sql
CREATE TABLE t1(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t2(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t3(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 CREATE TABLE t4(id INT(10) AUTO_INCREMENT,content  VARCHAR(100) NULL ,  PRIMARY KEY (id));
 
 
INSERT INTO t1(content) VALUES(CONCAT('t1_',FLOOR(1+RAND()*1000)));
 
INSERT INTO t2(content) VALUES(CONCAT('t2_',FLOOR(1+RAND()*1000)));
  
INSERT INTO t3(content) VALUES(CONCAT('t3_',FLOOR(1+RAND()*1000)));
    
INSERT INTO t4(content) VALUES(CONCAT('t4_',FLOOR(1+RAND()*1000)));
~~~

###### id(重要)

**含义**

select查询的序列号,包含一组数字，表示查询中执行select**子句或操作表**的顺序

**三种情况**

id相同，执行顺序由上至下

~~~ sql
mysql> explain select *  from t1,t2,t3 where t1.id=t2.id and t1.id=t3.id; 
~~~

![1612159590490](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/140631-910143.png)

- id相同，执行顺序由上至下 ，程序员理解加载表的顺序是t1,t2,t3，mysql优化器优化后加载的顺序也是t1,t2,t3，但是有时候可能和程序员想的加载顺序不同，
- 此例中 先执行where 后的第一条语句 t1.id = t2.id 通过 t1.id 关联 t2.id 。 而  t2.id 的结果建立在 t2.id=t3.id 的基础之上。

id不同

如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行，如果id相同，那么就是顺序执行

![1612160172746](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/141614-655814.png)

t3表在最里面，所以t3是一个子查询，id序号3最大，所以最先被执行。接下来加载执行t1表，最后是t2表。id相同的话，会顺序加载。

id相同不同，同时存在

![1612160464470](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/142105-140938.png)

id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行，所以执行顺序是t3，t1，t2（t3优先级最大，最先执行，t1,t2优先级一样，顺序执行），derived2中的2就是代表t3对应的id2,也就是说延伸表来自于t3后的，t3对应序号是2。

衍生表 = derived2 --> derived + 2 （2 表示由 id =2 的查询衍生出来的表。type 肯定是 all ，因为衍生的表没有建立索引）

###### select_type

不同类型表示从优化器的角度理解查询的语句的类型。

**值**

![1612164443910](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/135808-705266.png)

**查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询**

**SIMPLE**： 简单的 select 查询,查询中不包含子查询或者UNION

![1612164574317](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/135820-680621.png)

**PRIMARY**：查询中若包含任何复杂的子部分，**最外层查询则被标记为Primary**，也就是最后加载的那张表的查询

![1612164653519](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/153055-160600.png)

外面的查询是从衍生表种查询。

**SUBQUERY**：在SELECT或WHERE列表中包含了子查询，**这个仅仅是子查询，而不是依赖子查询**，因为子查询中返回的结果是单值，也就是说只有一个结果，但是依赖子查询返回的结果又多个结果，

![1612164725633](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/153210-787302.png)

**DERIVED**：**在FROM列表中包含的子查询被标记为DERIVED(衍生)**，MySQL会递归执行这些子查询, 把结果放在临时表里。

![1612164793087](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/153323-849188.png)

- 这里要记住一点，是from子句中加载的表是子查询。比如上面的t1表是在from子句中加载的，那么就是DERIVED。

**UNION**：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED

![1612165193743](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/30/142136-737302.png)

UNION RESULT 两个语句执行完后的结果，也就是union两边最后合并的结果，放在最后。

**UNION RESULT**:从UNION表获取结果的SELECT

![1612165539494](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/154540-610450.png)

**DEPENDENT SUBQUERY**:在SELECT或WHERE列表中包含了子查询,子查询基于外层

![1612165602916](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/154643-562339.png)

- dependent subquery 与 subquery 的区别
  - 依赖子查询 ： 子查询结果为 多值
  - 子查询：查询结果为 单值 

**UNCACHEABLE SUBQUREY**:无法被缓存的子查询

![1612165722068](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/154850-259940.png)

图1 中的 @@ 表示查的环境参数 。没办法缓存

###### table

显示这一行的数据是关于哪张表的。

###### type（重要）

**可以取得值**

![1612165910240](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/155152-755719.png)

**type显示的是访问类型，是较为重要的一个指标，结果值从最好到最坏依次是：** 

~~~ sql
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range(尽量保证) > index > ALL 
system>const>eq_ref>ref>range>index>ALL
~~~

**一般来说，得保证查询至少达到range级别，最好能达到ref。**

> all的意思就是说检索的结果全表扫描。

**system**

表只有一行记录（等于系统表，mysql出厂带的表），这是const类型的特列，平时不会出现，这个也可以忽略不计，也就是说查询的表中只有一条记录。

**const**

- 表示通过索引一次就找到了,const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。
- 如将主键置于where列表中，MySQL就能将该查询转换为一个常量，也就是某一列的值是唯一的，那么就是const。

![1612166398694](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140016-905839.png)

这里可以看到,子查询中id=1已经写死了，只有一条记录，所以type=const，延伸表是system，因为子查询查询的结果是一张只有一条记录的表。所以是system。

**eq_ref**

唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

![1612166657506](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/160802-526613.png)

t1表是驱动表，所以需要全表扫描，但是t2表中只有一条记录和t1表对应，所以是eq_ref，也就是说t1表和t2表是一对一的关系。

**ref**

非唯一性索引扫描，返回匹配某个单独值的**所有行**.
本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

![1612167277478](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/161439-922704.png)

可以看到，在没有建立索引之前，两个表基本都是全表扫描，但是在给t2表建立了索引之后，因为t1,t2表有多条记录匹配，所以t2就类型就是ref，但是表t1还是要全表扫描。

**range**

只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引
一般就是在你的where语句中出现了between、<、>、in等的查询
这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。

![1612167536836](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140027-413395.png)

**index**

Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

![1612167593553](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162008-162926.png)

id是主键的索引。也就是扫描索引。

**all**

Full Table Scan，将遍历全表以找到匹配的行

![1612167667407](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162108-889109.png)

**index_merge**

在查询过程中需要多个索引组合使用，通常出现在有 or 的关键字的sql中

![1612167751530](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162233-401413.png)

**ref_or_null**

对于某个字段既需要关联条件，也需要null值得情况下。查询优化器会选择用ref_or_null连接查询。

![1612167791087](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162312-905299.png)

**index_subquery**

利用索引来关联子查询，不再全表扫描。

![1612167829824](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162350-367197.png)

**unique_subquery **

该联接类型类似于index_subquery。 子查询中的唯一索引

![1612167862961](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/162430-435765.png)

> 备注：一般来说，得保证查询至少达到range级别，最好能达到ref。

###### possible_keys

> 理论上可能用到的索引

显示可能应用在这张表中的索引，一个或多个。
查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。

如果程序员建立了多条索引，那么Mysql在执行查询的时候，可能不会用到建立的全部索引，有的索引可能用不到。

###### key(重要)

实际使用的索引。如果为NULL，则没有使用索引

查询中若使用了覆盖索引（覆盖索引就是待查询的字段和建立索引的字段相互吻合了），则该索引和查询的select字段重叠

![1612168816799](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140119-763431.png)

![1612168370200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/163251-760005.png)

###### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。 

![1612169165317](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/164606-94633.png)

**计算方法**

![1612169237076](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140127-765798.png)

总结一下：char(30) utf8 --> key_len = 30*3 +1  表示 utf8 格式需要  *3 (跟数据类型有关)   
允许为 NULL  +1  ，不允许 +0
动态类型 +2  (动态类型包括 : varchar , detail text() 截取字符窜)

![1612169282145](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140129-330799.png)

key_len字段能够帮你检查是否充分的利用上了索引

###### ref

显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

![1612169759385](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/165600-677374.png)

t2表的col1列被引用了，并且还引用了一个常量值ac。

###### rows（重要）

rows列显示MySQL认为它执行查询时必须检查的行数。越少越好

![1612170221210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/01/170341-474471.png)

###### Extra

包含不适合在其他列中显示但十分重要的额外信息

1. Using filesort 

   说明mysql会对数据使用一个**外部的索引排序**，而不是按照表内的索引顺序进行读取。MySQL中无法利用索引完成的排序操作称为“文件排

~~~ java
// \G表示字段竖着显示
 explain select * from t_dept\G;
~~~

建立索引之后，排序的时候，最好是按照索引的顺序进行排序。

![1613454513718](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/140144-25523.png)

第一种查询操作，排序没有用到全部的索引，也就是col2，所以mysql会在外部重新建立索引进行排序操作，所以在extra中会出现using filesort字段。

但是第二种排序直接使用了索引的全部字段，mysql也就没有产生外部文件的索引，所以查询效率很高。

查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度。

分情况：当通过前面的查询语句 筛选大部分条件后，只剩下很少的数据。using filesort 性能影响不大。需要综合考虑

> 使用索引就是为了加快查询和排序。

2. Using temporary 

   使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。

![1613454938066](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/16/135539-683086.png)

查询一在clo1和col2列上建立复合索引，但是在分组的时候，只是按照col2进行分组，所以会使用临时表，查询二按照建立索引的列进行分组。创建临时表很伤系统的性能。

3. USING index

表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！
如果同时出现using where，表明索引被用来执行索引键值的查找;
如果没有同时出现using where，表明索引只是用来读取数据而非利用索引执行查找。

**覆盖索引(Covering Index)**

索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据;当能通过读取索引就可以得到想要的数据，那就不需要读取行了。

1. 一个索引
2. 包含了(或覆盖了)[select子句]与查询条件[Where子句]中
3. 所有需要的字段就叫做覆盖索引。

上句理解：
         select id , name from t_xxx where age=18;
有一个组合索引  idx_id_name_age_xxx 包含了(覆盖了)，id,name,age三个字段。查询时直接将建立了索引的列读取出来了，而不需要去查找所在行的其他数据。所以很高效。         (个人认为：在数据量较大，固定字段查询情况多时可以使用这种方法。)

注意：
如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select *，
因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。

4. Using where

表明使用了where过滤

5. using join buffer

使用了连接缓存：

![1613456200097](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/16/141647-142503.png)

出现在当两个连接时
驱动表(被连接的表,left join 左边的表。inner join 中数据少的表) 没有索引的情况下。
给驱动表建立索引可解决此问题。且 type 将改变成 ref

1. impossible where

where子句的值总是false，不能用来获取任何元组

![1613456267896](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/16/141748-142390.png)

7. select tables optimized away

在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者
对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，
查询执行计划生成的阶段即完成优化。

**在innodb中：**

![1613456360958](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/16/141928-343900.png)

**在Myisam中：**

![1613456386895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/16/141954-363155.png)

myisam 中会维护 总行数 (还有其他参数)这个参数，所以在执行查询时不会进行全表扫描。而是直接读取这个数。
但会对增删产生一定的影响。根据业务情况决定谁好谁坏
innodb 中没有这个机制。

### 查询优化

#### 单表查询优化

**创建表**

~~~ sql
CREATE TABLE IF NOT EXISTS `article` (
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`author_id` INT(10) UNSIGNED NOT NULL,
`category_id` INT(10) UNSIGNED NOT NULL,
`views` INT(10) UNSIGNED NOT NULL,
`comments` INT(10) UNSIGNED NOT NULL,
`title` VARBINARY(255) NOT NULL,
`content` TEXT NOT NULL
);
 
INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES
(1, 1, 1, 1, '1', '1'),
(2, 2, 2, 2, '2', '2'),
(1, 1, 3, 3, '3', '3');
 
SELECT * FROM article;
~~~

**案例**

~~~ sql
-- #查询 category_id 为1 且  comments 大于 1 的情况下,views 最多的 article_id。 
EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
~~~

![1613537499820](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/125140-60934.png)

- 结论：很显然,type 是 ALL,即最坏的情况,可以说是全表扫描。Extra 里还出现了 Using filesort,也是最坏的情况，使用外部的索引排序。优化是必须的。

由于上面的查询使用到category_id，comments,和views等三个字段，所以创建复合索引。

~~~ sql
create index idx_article_ccv on article(category_id,comments,views);
//查看创建的索引
mysql> show index from article;
~~~

![1613537881592](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/125802-904006.png)

再次查看执行结果

![1613538016269](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/130055-429095.png)

结论：

- type 变成了 range,这是可以忍受的。但是 extra 里使用 Using filesort 仍是无法接受的。
- 但是我们已经建立了索引,为啥没用呢?
- 这是因为按照 BTree 索引的工作原理,
- 先排序 category_id,
- 如果遇到相同的 category_id 则再排序 comments,如果遇到相同的 comments 则再排序 views。
- 当 comments 字段在联合索引里处于中间位置时,
- 因comments > 1 条件是一个范围值(所谓 range),
- MySQL 无法利用索引再对后面的 views 部分进行检索,即 range 类型查询字段后面的索引无效。

> 范围以后的索引全部会失效

- 可以看到，如果我们把comments>1修改为comments=1，就可以使用comments索引了。也就是说我们建立的复合索引只用到了category_id第一个字段，而没有使用后面的两个字段。建立复合索引，字段的顺序会影响索引的查询结果。

![1613538187792](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/130355-734305.png)

删除创建的idx_article_ccv索引，重新建立索引

~~~ java
 drop index idx_article_ccv on article; 
//重新再catagery_id和views上创建索引
 create index idx_article_cv on article(category_id,views);
~~~

![1613538830059](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/131415-401668.png)

重新执行查询

![1613538878763](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/131459-644520.png)

结论：可以看到,type 变为了 ref,Extra 中的 Using filesort 也消失了,结果非常理想。检索和排序同时用到我们的索引

> 单表优化的重点，让检索和排序都使用到我们建立的索引。

#### 两表查询优化

**建表**

~~~ sql
 
CREATE TABLE IF NOT EXISTS `class` (
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`id`)
);
CREATE TABLE IF NOT EXISTS `book` (
`bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`bookid`)
);
 
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
 
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));

~~~

**案例**

~~~ sql
explain select * from class left join book on class.card=book.card;
~~~

![1613539597612](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/132637-142132.png)

结论：type 有All,也就是两张表全部是全表扫描

- 接下来添加索引

~~~ sql
 -- 首先添加索引在右表上
 create index Y on book(card);
~~~

![1613539973734](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/133339-933304.png)

删除索引，然后添加索引到class表上面

~~~ sql
drop index Y on book; 
-- 添加索引到class，也就是左表
create index Y on class(card);
explain select * from class left join book on class.card=book.card;
~~~

![1613540114207](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/133521-328072.png)

- 可以看到第二行的 type 变为了 ref,rows 也变成了优化比较明显。
- 这是由左连接特性决定的。LEFT JOIN 条件用于确定如何从右表搜索行,左边一定都有,
- 所以右边是我们的关键点,一定需要建立索引。
- 所以如果左链接，我们一定要在右表上建立链接，右链接在左表上建立索引

如下所示，在book上面建立的链接，进行右链接

![1613540579832](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/134301-25486.png)

#### 三标查询优化

**创建表**

~~~ sql
CREATE TABLE IF NOT EXISTS `phone` (
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`id`)
);

INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));

~~~

**案例分析**

~~~ sql
explain select * from class inner join book on class.card=book.card inner join phone on book.card=phone.card;
~~~

![1613541495197](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/135815-75585.png)

**创建索引优化**

~~~sql
create index z on phone(card); 
create index y on book(card); 
~~~

![1613541790598](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/140311-752740.png)

索引添加在后两张表中，并且索引最好添加在需要经常查询的字段中

#### 小结

1. 保证被驱动表的join字段已经被索引，被驱动表  join 后的表为被驱动表  (需要被查询)
2. left join 时，选择小表作为驱动表，大表作为被驱动表，因为大表需要被查询，所以在大表上建立索引可以提高查询效率， left join 时一定是左边是驱动表，右边是被驱动表
3. inner join 时，mysql会自己帮你把小结果集的表选为驱动表。mysql 自动选择。小表作为驱动表。因为 驱动表无论如何都会被全表扫描？。所以扫描次数越少越好
4. 子查询尽量不要放在被驱动表，有可能使用不到索引。

~~~ sql
select a.name ,bc.name from t_emp a left join
         (select b.id , c.name from t_dept b
         inner join t_emp c on b.ceo = c.id)bc 
         on bc.id = a.deptid.
--上段查询中用到了子查询，必然 bc 表没有索引。肯定会进行全表扫描
--上段查询 可以直接使用 两个 left join 优化
select a.name , c.name from t_emp a
    left outer join t_dept b on a.deptid = b.id
    left outer join t_emp c on b.ceo=c.id
--所有条件都可以使用到索引
 
--若必须用到子查询，可将子查询设置为驱动表，，因为驱动表的type 肯定是 all，而子查询返回的结果表没有索引，必定也是all
~~~

#### 使用索引

##### 创建表

~~~ sql
 
CREATE TABLE staffs (
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR (24)  NULL DEFAULT '' COMMENT '姓名',
  age INT NOT NULL DEFAULT 0 COMMENT '年龄',
  pos VARCHAR (20) NOT NULL DEFAULT '' COMMENT '职位',
  add_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
) CHARSET utf8 COMMENT '员工记录表' ;
 
 
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES('2000',23,'dev',NOW());
INSERT INTO staffs(NAME,age,pos,add_time) VALUES(null,23,'dev',NOW());
SELECT * FROM staffs;
 
 -- 创建复合索引
ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name, age, pos);
~~~

##### 全部匹配索引最好

~~~ sql
--索引  idx_staffs_nameAgePos 建立索引时 以 name ， age ，pos 的顺序建立的。全值匹配表示 按顺序匹配的
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
~~~

![1613542826246](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/142111-881067.png)

type是决定访问类别的，ref决定性能的好坏。

~~~ sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25;
~~~

![1613542974385](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/142254-646676.png)

各个字段相同的情况下，key_len的长度越短越好。可以发现查询使用的属性个数增加，也就是两个const常量，所以必然引起key_len长度增加

~~~ sql
EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25 AND pos = 'dev';
~~~

![1613543186335](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/142626-72828.png)

~~~ sql
EXPLAIN SELECT * FROM staffs WHERE age = 25 AND pos = 'dev';    
~~~

![1613543306983](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/142827-928279.png)

发现索引全部失效，所以在查询的时候，最好不要跳过索引列。

##### 最佳左前缀法则

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。

- and 忽略左右关系。既即使没有按顺序 由于优化器的存在，会自动优化。
- 经过试验结论  建立了 idx_nameAge 索引  id 为主键
  - 当使用覆盖索引的方式时，(select name/age/id from staffs where age=10 (后面没有其他没有索引的字段条件))，即使不是以 name 开头，也会使用 idx_nameAge 索引。既 select 后的字段 有索引，where 后的字段也有索引，则无关执行顺序。
  - 除开上述条件 才满足最左前缀法则。

![1613544161528](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/144242-540900.png)

##### 不做计算

不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描

~~~ sql
EXPLAIN SELECT * FROM staffs WHERE NAME= 'July'; 
~~~

![1613544521400](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/144842-22263.png)

~~~ sql
EXPLAIN SELECT * FROM staffs WHERE left(NAME,4) = 'July';
~~~

![1613544559944](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/144921-777281.png)

type类型是all，做了全表扫描。

##### 少用范围查找

存储引擎不能使用索引中范围条件右边的列

范围 若有索引则能使用到索引，范围条件右边的索引会失效(范围条件右边与范围条件使用的同一个组合索引，右边的才会失效。若是不同索引则不会失效)

~~~ sql
explain select * from staffs where NAME='July' and age >11 and pos='manager'; 
~~~

![1613544862371](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/145422-375243.png)

age索引后的索引全部会失效，会全表扫描。

##### 使用覆盖索引

尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少select *

![1613545168009](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/150228-341282.png)

##### 不等号的使用

mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描

![1613545588670](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202102/17/150629-529667.png)

- 索引  idx_nameAgeJob，idx_name
- 使用 != 和 <> 的字段索引失效( != 针对数值类型。 <> 针对字符类型
- 前提 where and 后的字段在混合索引中的位置比比当前字段靠后  where age != 10 and name='xxx'  ,这种情况下，mysql自动优化，将 name='xxx' 放在 age ！=10 之前，name 依然能使用索引。只是 age 的索引失效)

##### is null

is not null 也无法使用索引,但是is null是可以使用索引的

![1613545787802](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1613545787802.png)

##### like使用

like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作.

一般情况下将%放在字符串的右边进行匹配。

![1640763313781](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/154128-861390.png)

那么如果工作中，字符串两端都需要使用%怎么办？

这里推荐使用覆盖索引解决。

如果不建立索引，使用%aa%进行匹配的话，几乎会全表扫描：

![1640763860515](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/154421-148415.png)

我们可以建立一个覆盖索引，在需要查询的字段上面建立索引，这个索引的字段和我们需要查询的字段，个数最好一致，这样就可以使用%进行两边匹配，既可以避免全表扫描，还可以使用索引。

##### 字符串不加单引号索引失效

![1640764696206](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640764696206.png)

name在这里是vchar类型。

![1640764847262](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640764847262.png)

如果不添加引号，那么在mysql底层会自动进行隐式转换，相当于做了计算。

##### 少用or,用它来连接时会索引失效

![1640764992566](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/160315-33421.png) 

理论上可以查询结果，但是没有使用索引。

##### 小结

![1640765154098](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/160556-561450.png)

#### 一般性建议

对于单键索引，尽量选择针对当前query过滤性更好的索引

在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。(避免索引过滤性好的索引失效)

在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引

尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的

### 查询截取分析

面试时候，说使用过explain关键字。

**生产中调优的步骤**

![1640765787951](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/161629-335667.png)

#### 子查询优化

用in 还是 exists

> 永远小表驱动大表。
>
> 如果B表数据集小于A表数据，我们使用In
>
> 如果B表数据大于A表数据集，我们使用exists。

![1640766370476](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/162813-40808.png)

![1640766494637](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/162814-731174.png)

![1640766760459](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/163241-145291.png)

##### order by关键字优化

ORDER BY子句，尽量使用Index方式排序,避免使用FileSort方式排序

**建表**

![1640766888711](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/163504-273942.png)

![1640767107069](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/163827-415794.png)

因为建立了索引，所以第一条sql使用到了age索引，是覆盖索引，没有产生filesort.

第二个sql也使用到索引，因为建立索引使用的是age,birth，所以在查询的时候，排序的时候，都用到了索引。

第三条sql使用到了filesort。

第四条sql排序字段和索引字段顺序不同，所以产生fiLe sort。

> 索引着重于排序和查找功能。但是在第三条sql中，建立好索引为age和birth，但是排序的时候，顺序没有按照建立索引的顺序来，所以失效。

MySQL支持二种方式的排序，FileSort和Index，Index效率高.它指MySQL扫描索引本身完成排序。FileSort方式效率较低。

ORDER BY满足两情况，会使用Index方式排序:

- 语句使用索引最左前列
- 使用Where子句与Order BY子句条件列组合满足索引最左前列
- where子句中如果出现索引的范围查询(即explain中出现range)会导致order by 索引失效。

尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

![1640767713105](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/164833-980416.png)

如果不在索引列上，filesort有两种算法：mysql就要启动**双路排序和单路排序**

##### 双路排序

MySQL 4.1之前是使用双路排序,字面意思就是两次扫描磁盘，最终得到数据，读取行指针和orderby列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出

多路排序需要借助 磁盘来进行排序。所以 取数据，排好了取数据。两次 io操作。比较慢，单路排序 ，将排好的数据存在内存中，省去了一次 io 操作，所以比较快，但是需要内存空间足够。

从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。

取一批数据，要对磁盘进行了两次扫描，众所周知，I\O是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序。

##### 单路排序

从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，

它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO,但是它会使用更多的空间，
因为它把每一行都保存在内存中了。

什么情况下单路排序性能不好？

在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出, 所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取取sort_buffer容量大小，再排……从而多次I/O。

本来想省一次I/O操作，反而导致了大量的I/O操作，反而得不偿失。

##### 使用单路排序优化策略

增大sort_buffer_size参数的设置：用于单路排序的内存大小

增大max_length_for_sort_data参数的设置：单次排序字段大小。(单次排序请求)

去掉select 后面不需要的字段：select 后的多了，排序的时候也会带着一起，很占内存，所以去掉没有用的

提高Order By的速度

1. Order by时select * 是一个大忌只Query需要的字段， 这点非常重要。在这里的影响是：
   1.1 当Query的字段大小总和小于max_length_for_sort_data 而且排序字段不是 TEXT|BLOB 类型时，会用改进后的算法——单路排序， 否则用老算法——多路排序。
     1.2 两种算法的数据都有可能超出sort_buffer的容量，所以如果使用select *更容易昂缓冲区满，超出之后，会创建tmp文件进行合并排序，导致多次I/O，但是用单路排序算法的风险会更大一些,所以要提高sort_buffer_size。
2. 尝试提高 sort_buffer_size
   不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的
3. 尝试提高 max_length_for_sort_data
   提高这个参数， 会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率. 

**小结**

![1640775106705](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/185252-238071.png)

##### 分页查询的优化---limit

实践证明： ①、order by 后的字段（XXX）有索引 ②、sql 中有 limit 时，
    当 select id 或 XXX字段索引包含字段时 ，显示 using index
    当 select 后的字段含有 bouder by 字段索引不包含的字段时，将显示 using filesort

##### GROUP BY关键字优化

group by实质是先排序后进行分组，遵照索引建的最佳左前缀

当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置

where高于having，能写在where限定的条件就不要去having限定了。

其他的和order by规则类似。

##### 去重优化

尽量不要使用 distinct 关键字去重：优化

![1640775577323](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/185938-830451.png)

### 慢查询日志

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。

具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句。

由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。

#### **查看是否开启及如何开启**

默认情况下：

```sql
SHOW VARIABLES LIKE '%slow_query_log%';
```

默认情况下slow_query_log的值为OFF，表示慢查询日志是禁用的，可以通过设置slow_query_log的值来开启

SHOW VARIABLES LIKE '%slow_query_log%';

![1640775766160](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/190247-935697.png)

#### 开启

```sql
set global slow_query_log=1;
--在本次会话中开启，可以在配置文件中修改
```

使用set global slow_query_log=1开启了慢查询日志只对当前数据库生效，
如果MySQL重启后则会失效。

那么开启了慢查询日志后，什么样的SQL才会记录到慢查询日志里面呢？

![1640776136506](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/190856-721707.png)

查看当前多少秒算慢

```sql
SHOW VARIABLES LIKE 'long_query_time%';
```

设置慢的阙值时间:

为什么设置后看不出变化？

1. 需要重新连接或新开一个会话才能看到修改值。 SHOW VARIABLES LIKE 'long_query_time%';
2. 或者通过set session long_query_time=1来改变当前session变量;

记录慢SQL并后续分析

查询当前系统中有多少条慢查询记录

![1640776498847](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/191517-159301.png)

通过分析日志，我们直到有那些sql执行的缓慢。

**配置永久生效**

```sql
【mysqld】下配置：

slow_query_log=1;
slow_query_log_file=/var/lib/mysql/atguigu-slow.log
long_query_time=3;
log_output=FILE
```

#### 日志分析工具

日志分析工具mysqldumpslow

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow

查看mysqldumpslow的帮助信息

![1640776679257](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/191813-933537.png)

**常用参考**

```sql
得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log
 
得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log
 
得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log
 
另外建议在使用这些命令时结合 | 和more 使用 ，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```

### Show Profile

工作中如何优化sql:

- 开启慢查询日志，抓取执行缓慢的sqlyuju 。

- 采用explain进行分析。

- 使用show profile进行分析。
- 调整硬件参数，内存资源。

#### 是什么

是什么：是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量

默认情况下，参数处于关闭状态，并保存最近15次的运行结果

#### 分析步骤

1. 是否支持，看看当前的mysql版本是否支持

```sql
Show  variables like 'profiling';
默认是关闭，使用前需要开启
```

2. 开启功能，默认是关闭，使用前需要开启

```sql
show variables like 'profiling';
 
set profiling=1;
```

3. 执行sql

```sql
select * from emp group by id%10 limit 150000;
select * from emp group by id%20  order by 5
```

4. 查看执行结果

![1640777339549](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/192920-624034.png)

5. 诊断SQL，show profile cpu,block io for query  n  (n为上一步前面的问题SQL数字号码);

![1640777385690](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/192946-948506.png)

**参数说明**

![1640777407498](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/193008-704392.png)

**日常开发需要注意的结论**

1. converting HEAP to MyISAM 查询结果太大，内存都不够用了往磁盘上搬了。
2. Creating tmp table 创建临时表
   1. 拷贝数据到临时表
   2. 用完在删除数据
3. Copying to tmp table on disk 把内存中临时表复制到磁盘，危险！！！
4. locked

### Mysql锁

#### 锁的分类

从对数据操作的类型（读\写）分

- 读锁(共享锁)：针对同一份数据，多个读操作可以同时进行而不会互相影响。
- 写锁（排它锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。

从对数据操作的粒度分

- 表锁
- 行锁

#### 表锁(偏读)

**特点**

偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。

**结论**

**MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。** 

MySQL的表级锁有两种模式：

- 表共享读锁（Table Read Lock）
- 表独占写锁（Table Write Lock）

![1640779334154](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/200214-787250.png)

结论：

结合上表，所以对MyISAM表进行操作，会有以下情况： 

1. 对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求。只有当读锁释放后，才会执行其它进程的写操作。 
2. 对MyISAM表的写操作（加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作。

>  简而言之，就是读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞

#### 行锁(偏写)

**特点**

偏向InnoDB存储引擎，开销大，加锁慢；**会出现死锁**；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。

InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁

**Select也可以加锁**

读锁

共享锁(Share Lock)

共享锁又称读锁，是读取操作创建的锁。其他用户可以并发读取数据，但任何事务都不能对数据进行修改（获取数据上的排他锁），直到已释放所有共享锁。

如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不能加排他锁。获准共享锁的事务只能读数据，不能修改数据。

用法
SELECT ... LOCK IN SHARE MODE;

在查询语句后面增加 LOCK IN SHARE MODE ，Mysql会对查询结果中的每行都加共享锁，当没有其他线程对查询结果集中的任何一行使用排他锁时，可以成功申请共享锁，否则会被阻塞。其他线程也可以读取使用了共享锁的表（行？），而且这些线程读取的是同一个版本的数据。

写锁

排他锁（eXclusive Lock）

共享锁又称写锁，如果事务T对数据A加上排他锁后，则其他事务不能再对A加任任何类型的封锁。获准排他锁的事务既能读数据，又能修改数据。

用法

SELECT ... FOR UPDATE;

 在查询语句后面增加 FOR UPDATE ，Mysql会对查询结果中的每行都加排他锁，当没有其他线程对查询结果集中的任何一行使用排他锁时，可以成功申请排他锁，否则会被阻塞。

**无索引，行锁升级为表锁**

![1640779993907](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/201315-584017.png)

**间歇锁**

![1640779757302](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/29/200918-512761.png)

【什么是间隙锁】
当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（GAP Lock）。

【危害】
因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

**小结**

 Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了。

  但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

#### 优化建议

1. 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
2. 尽可能较少检索条件，避免间隙锁
3. 尽量控制事务大小，减少锁定资源量和时间长度
4. 锁住某行后，尽量不要去调别的行或表，赶紧处理被锁住的行然后释放掉锁。
5. 涉及相同表的事务，对于调用表的顺序尽量保持一致。
6. 在业务环境允许的情况下,尽可能低级别事务隔离