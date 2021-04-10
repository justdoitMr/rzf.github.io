# Redis

## Redis简介

### 传统关系型数据库存在的现象

- 性能瓶颈：磁盘IO性能低下
- 扩展瓶颈：数据关系复杂，扩展性差，不便于大规模集群

**如何解决**

- Nosql
  - 降低磁盘IO次数，越低越好，内存存储
  - 去除数据间关系，越简单越好，不存储关系，仅存储数据

### Nosql

NoSQL：即 Not-Only SQL（ 泛指非关系型的数据库），作为关系型数据库的补充。
作用：应对基于海量用户和海量数据前提下的数据处理问题。

- 特征：
  - 可扩容，可伸缩
  - 大数据量下高性能
  - 灵活的数据模型
  - 高可用

- 常见 Nosql 数据库：
  - Redis
  - memcache
  - HBase
  - MongoDB

### Redis简介

概念：Redis (REmote DIctionary Server) 是用 C 语言开发的一个开源的高性能键值对（key-value）数据库。

- 特征：
  - 数据间没有必然的关联关系
  - 内部采用单线程机制进行工作
  - 高性能。官方提供测试数据，50个并发执行100000 个请求,读的速度是110000 次/s,写的速度是81000次/s。
  - 多数据类型支持
    - 字符串类型 string
    - 列表类型 list
    - 散列类型 hash
    - 集合类型 set
    - 有序集合类型 sorted_set
  - 持久化支持。可以进行数据灾难恢复

### Redis应用

- 为热点数据加速查询（主要场景），如热点商品、热点新闻、热点资讯、推广类等高访问量信息等
- 任务队列，如秒杀、抢购、购票排队等
- 即时信息查询，如各位排行榜、各类网站访问统计、公交到站信息、在线人数信息（聊天室、网站）、设备信号等
- 时效性信息控制，如验证码控制、投票控制等
- 分布式数据共享，如分布式集群架构中的 session 分离
- 消息队列
- 分布式锁

### Redis下载和安装

**Linux 版**（适用于企业级开发）

- Redis 高级开始使用
- 以4.0 版本作为主版本

**Windows 版本**

- Redis 入门使用
- 以 3.2 版本作为主版本
- 下载地址：`https://github.com/MSOpenTech/redis/tags`

**安装目录**

![1617868944542](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/160225-54710.png)

**启动Redis**

![1617868992561](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/160326-360570.png)

**客户端连接**

![1617869028719](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/160350-108604.png)

### Redis基本操作

#### 添加信息

功能：设置 key，value 数据

命令

~~~ java
set key value
//案例
set name rzf
~~~

#### 信息查询

功能：根据 key 查询对应的 value，如果不存在，返回空（nil）

命令

~~~ java
get key
//案例：redis是键值对数据库，所以需要根据key去查询
127.0.0.1:6379> get name
"rzf"
~~~

#### 清除屏幕信息

清除屏幕中的信息

~~~ java
clear
~~~

#### 退出客户端命令

~~~ java
quit 
exit
~~~

#### 帮助命令

功能：获取命令帮助文档，获取组中所有命令信息名称

~~~ java
help 命令名称 
help @组名
//案例
127.0.0.1:6379> help get

  GET key
  summary: Get the value of a key
  since: 1.0.0
  group: string
~~~

![1617870057006](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/162057-857256.png)

## Redis数据类型

### redis 数据存储格式

- redis 自身是一个 Map，其中所有的数据都是采用 key : value 的形式存储
- 数据类型指的是存储的数据的类型，也就是 value 部分的类型，key 部分永远都是字符串

![1617870736286](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/163217-467595.png)

### String类型

- 存储的数据：单个数据，最简单的数据存储类型，也是最常用的数据存储类型
- 存储数据的格式：一个存储空间保存一个数据
- 存储内容：通常使用字符串，如果字符串以整数的形式展示，可以作为数字操作使用

![1617870805331](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/08/163326-661341.png)

**基本操作**

- 添加或者修改数据

~~~ java
set key value
//案例
 set name rzf
 //如果已经有某一个键了，那么在此设置会做更新操作
~~~

- 获取数据

~~~ java
get key
//案例
get name
~~~

- 删除数据

~~~ java
//可以根据key去删除数据
del key
// 案例
127.0.0.1:6379> del name
(integer) 1
//在Redis中操作成功或者失败都是以1或者0表示
~~~

- 添加或者修改多个数据

~~~ java
mset key1 value1 key2 value2 …
//案例
 mset a 1 b 2 c 3
//对原有的键的值做更新操作，会做覆盖操作
~~~

- 获取多个数据

~~~ java
mget key1 key2 …
//获取多个数据
  127.0.0.1:6379> mget a b c
1) "1"
2) "2"
3) "3"
~~~

- 获取数据字符个数（字符串长度）

~~~ java
strlen key
//案例
127.0.0.1:6379> strlen name
(integer) 3
~~~

- 追加信息到原始信息后部（如果原始信息存在就追加，否则新建）

~~~ java
append key value
//案例
 append name hahaha
~~~

**单数据操作和多数据操作如何选择**

如果数据量太大，