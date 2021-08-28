# Redis

[TOC]

## Redis简介

### 传统关系型数据库存在的现象

- 性能瓶颈：磁盘IO性能低下
- 扩展瓶颈：数据关系复杂，扩展性差，不便于大规模集群

**如何解决**

- Nosql
  - 降低磁盘IO次数，越低越好，内存存储
  - 去除数据间关系，越简单越好，不存储关系，仅存储数据

### Nosql

- NoSQL：即 Not-Only SQL（ 泛指非关系型的数据库），作为关系型数据库的补充。
- 作用：应对基于海量用户和海量数据前提下的数据处理问题。

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

### 思维导图

![1619590597102](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/28/141638-814448.png)

## Redis数据类型

### redis 数据存储格式

- redis 自身是一个 Map，其中所有的数据都是采用 key : value 的形式存储
- 数据类型指的是存储的数据的类型，也就是 value 部分的类型，**key 部分永远都是字符串**

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

#### String类型的扩展操作

**业务场景1**

大型企业级应用中，分表操作是基本操作，使用多张表存储同类型数据，但是对应的主键 id 必须保证统一性，不能重复。Oracle 数据库具有 sequence 设定，可以解决该问题，但是 MySQL数据库并不具有类似的机制，那么如何解决？

![1617938716851](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/28/141738-493220.png)

可以让主键不重复，也就是key唯一，每一张表中的key都不相同。

**解决方案**

1. 设置数值数据增加指定范围的值

~~~ java
//自增操作，可以对字符串进行操作，如果字符串是数值的话

incr key //增加1
incrby key increment //增加一个指定的值
incrbyfloat key increment //加浮点型

//案例
127.0.0.1:6379> incr num
(integer) 2
  127.0.0.1:6379> incrby num 10
(integer) 13
~~~

2. 设置数值数据减少指定范围的值

~~~ java
//自减操作

decr key decrby key increment

//案例
127.0.0.1:6379> decr num
(integer) 3
~~~

**业务场景2**

- “最强女生”启动海选投票，只能通过微信投票，每个微信号每 4 小时只能投1票。
- 电商商家开启热门商品推荐，热门商品不能一直处于热门期，每种商品热门期维持3天，3天后自动取消热门。
- 新闻网站会出现热点新闻，热点新闻最大的特征是时效性，如何自动控制热点新闻的时效性。

**解决方案**

~~~ java
//设置数据具有指定的生命周期
setex key seconds value //秒
psetex key milliseconds value //毫秒

//案例
127.0.0.1:6379> setex key 10 1 //设置key存活10s,十秒后就会自动销毁
  
//如果在10s内执行set key 3,那么会把上面的做覆盖操作
~~~

> redis 控制数据的生命周期，通过数据是否失效控制业务行为，适用于所有具有时效性限定控制的操作

#### String类型小结

**string 作为数值操作**

- string在redis内部存储默认就是一个字符串，当遇到增减类操作incr，decr时会转成数值型进行计算。

- redis所有的操作都是原子性的，采用单线程处理所有业务，命令是一个一个执行的，因此无需考虑并发带来的据影响。

- 注意：按数值进行操作的数据，如果原始数据不能转成数值，或超越了redis 数值上限范围，将报错。 9223372036854775807（java中long型数据最大值，Long.MAX_VALUE）

> redis用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性
>
> 此方案适用于所有数据库，且支持数据库集群

**注意事项**

- 数据操作不成功的反馈与数据正常操作之间的差异
  - 表示运行结果是否成功
    - (integer) 0 → false 失败
    - (integer) 1 → true 成功
  - 表示运行结果值
    - (integer) 3 → 3 3个
    - (integer) 1 → 1 1个
- 数据未获取到
  - （nil）等同于null
- 数据最大存储量
  - 512MB
- 数值计算最大范围（java中的long的最大值）

#### 案例

**业务场景**

主页高频访问信息显示控制，例如新浪微博大V主页显示粉丝数与微博数量

**解决方案**

1. 在redis中为大V用户设定用户信息，以用户主键和属性值作为key，后台设定定时刷新策略即可

~~~ java
eg: user:id:3506728370:fans → 12210947 
eg: user:id:3506728370:blogs → 6164 
eg: user:id:3506728370:focuss → 83
  
//user:表名
//id:主键
//fans:粉丝数
127.0.0.1:6379> set user:id:000789:fans 123456
OK
127.0.0.1:6379> set user:id:000789:blogs 789
~~~

2. 在redis中以json格式存储大V用户信息，定时刷新（也可以使用hash类型）

~~~ java
user:id:3506728370 → {"id":3506728370,"name":"春晚","fans":12210862,"blogs":6164, "focus":83}

//案例
127.0.0.1:6379> set user:id:000789 {id:000789,blogs:789,fans:122344}
//把粉丝的数量增加一个
127.0.0.1:6379> incr user:id:000789:fans
(integer) 123457
~~~

> redis应用于各种结构型和非结构型高热度数据访问加速

**key 的设置约定**

数据库中的热点数据key命名惯例

![1617940861660](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/120102-523810.png)

### Hash类型

对象类数据的存储如果具有较频繁的更新需求操作会显得笨重

![1617943351921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/124232-883987.png)

#### hash 

- 新的存储需求：对一系列存储的数据进行编组，方便管理，典型应用存储对象信息
- 需要的存储结构：一个存储空间保存多个键值对数据
- hash类型：底层使用哈希表结构实现数据存储

![1617943407403](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/124329-116036.png)

1. 添加或者修改数据

~~~ java
hset key field value

//案例 id是：user
127.0.0.1:6379> hset user name zhangsan
(integer) 1
127.0.0.1:6379> hset user age 39
(integer) 1
127.0.0.1:6379> hset user weight 80
~~~

2. 获取数据

~~~ java
hget key field  //获取键值下面的属性值
hgetall key //根据键值获取全部数据

//案例
hgetall user //获取全部的数据
~~~

3. 删除数据

~~~ java
hdel key field1 [field2]

//案例
hdel user age //删除年龄数据
~~~

4. 添加或者修改多个数据

~~~ java
hmset key field1 value1 field2 value2 …

//案例
hmset user name xiaorui weigth 120
~~~

5. 获取多个数据

~~~ java
hmget key field1 field2 …

//案例
 hmget user name weight
~~~

6. 获取哈希表中字段的数量

~~~ java
hlen key

//案例
 hlen user
~~~

7. 获取哈希表中是否存在指定的字段

~~~ java
hexists key field

//案例
hexists user name
~~~

####  Hash类型的扩展操作

1. 获取哈希表中所有的字段名或字段值

~~~ java
hkeys key 
hvals key

//案例
 hkeys user //获取键值
 hvals user //获取值集合
 //键不可以重复，但是值可以重复
~~~

2. 设置指定字段的数值数据增加指定范围的值

~~~ java
hincrby key field increment 
hincrbyfloat key field increment

//案例
hincrby user weight 10 //给体重增加10
~~~

#### hash 类型数据操作的注意事项

- hash类型下的value只能存储字符串，不允许存储其他数据类型，不存在嵌套现象。如果数据未获取到，对应的值为（nil）
- 每个 hash 可以存储 232 - 1 个键值对
- hash类型十分贴近对象的数据存储形式，并且可以灵活添加删除对象属性。但hash设计初衷不是为了存储大量对象而设计的，切记不可滥用，更不可以将hash作为对象列表使用
- hgetall 操作可以获取全部属性，如果内部field过多，遍历整体数据效率就很会低，有可能成为数据访问瓶颈

#### Hash类型应用场景

电商网站购物车设计与实现

- 仅分析购物车的redis存储模型 添加、浏览、更改数量、删除、清空
- 购物车于数据库间持久化同步（不讨论）
- 购物车于订单间关系（不讨论） 提交购物车：读取数据生成订单 商家临时价格调整：隶属于订单级别
- 未登录用户购物车信息存储（不讨论） cookie存储

**解决方案**

- 以客户id作为key，每位客户创建一个hash存储结构存储对应的购物车信息
- 将商品编号作为field，购买数量作为value进行存储
- 添加商品：追加全新的field与value
- 浏览：遍历hash
- 更改数量：自增/自减，设置value值
- 删除商品：删除field
- 清空：删除key
- 此处仅讨论购物车中的模型设计
- 购物车与数据库间持久化同步、购物车与订单间关系、未登录用户购物车信息存储不进行讨论

**案例实现**

~~~ java
//两个用户的购物车
127.0.0.1:6379> hmset 001 g01 100 go2 200
OK
127.0.0.1:6379> hmset 002 g02 1 g04 7 g05 100
  
//查看001购物车所有数据
hgetall 001
  

~~~

**当前设计是否加速了购物车的呈现**

当前仅仅是将数据存储到了redis中，并没有起到加速的作用，商品信息还需要二次查询数据库

- 每条购物车中的商品记录保存成两条field
- field1专用于保存购买数量

~~~ java
命名格式：商品id:nums 
保存数据：数值
~~~

field2专用于保存购物车中显示的信息，包含文字描述，图片地址，所属商家信息等 命名格式：

~~~ java
商品id:info 
保存数据：json
~~~

~~~ java
hsetnx key field value

 hmset 003 g01:nums 100 g02:info {...}
 hmset 004 g01:nums 100 g02:info {...}

//下面会返回失败，因为g01属性已经存在
127.0.0.1:6379> hsetnx 003 g01:nums 500
(integer) 0

//下面语句会执行成功，因为g05属性不存在
127.0.0.1:6379> hsetnx 003 g05:nums 500
(integer) 1
~~~

> hsetnx：如果某一个属性已经存在，那么执行此指令会失败，但是如果不存在，那么就会执行成功

场景2

双11活动日，销售手机充值卡的商家对移动、联通、电信的30元、50元、100元商品推出抢购活动，每种商品抢购上限1000张

**解决方案**

- 以商家id作为key
- 将参与抢购的商品id作为field
- 将参与抢购的商品数量作为对应的value
- 抢购时使用降值的方式控制产品数量
- 实际业务中还有超卖等实际问题，这里不做讨论

**String存储对象（json）和Hash存储对象对比**

String存储讲究的是整体性，要么一次性全部存储，要不一次性全部取出，而hash可以使用属性把数据隔离开，所以对数据的更新比较的方便。

### List类型

- 数据存储需求：存储多个数据，并对数据进入存储空间的顺序进行区分
- 需要的存储结构：一个存储空间保存多个数据，且通过数据可以体现进入顺序
- list类型：保存多个数据，底层使用双向链表存储结构实现
- 两端都可以插入或者输出数据

Redis中的list

![1617948142062](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/140223-537788.png)

#### List类型的基本操作

1. 添加修改数据

~~~ java
lpush key value1 [value2] ……
rpush key value1 [value2] ……

//案例
lpush list1 spark java scala //从左边添加数据
//查询数据 发现输出是倒序输出的，也就是如果数据从左边进，那么输出的顺序是从右边输出
127.0.0.1:6379> lrange list1 0 2
1) "scala"
2) "java"
3) "spark"
  
//从右边添加数据
127.0.0.1:6379> rpush list2 a b c
//查询数据，数据从右边进，输出顺序是按照输入顺序输出
127.0.0.1:6379> lrange list2 0 2
1) "a"
2) "b"
3) "c"
  
//不知道列表中有几个数据，可以使用-1,倒数第一个数据就是-1，倒数第二个元素就是-2
lrange list2 0 -1
~~~

2. 查看数据

~~~ java
lrange key start stop
lindex key index
llen key

//根据索引查看数据
 lindex list1 1
  
//查看数据列表的长度
llen list1
~~~

3. 获取并且移除元素

~~~ java
lpop key
rpop key

//案例
lpush list3 a b c d e
lpop list3 //输出e,从右边输出，也就是输出最右边的数据
 rpop list3  //输出a,删除最左边的数据
~~~

#### List类型的扩展操作

规定时间内获取并移除数据,相当于阻塞的list

~~~ java
blpop key1 [key2] timeout //key2表示可以从若干个列表中等待，如果第一个获取不到，可以从其他列表中获取
brpop key1 [key2] timeout
brpoplpush source destination timeout

blpop list1 10 //阻塞10秒钟，等待超过10秒后就获取不到数据
~~~

#### List应用场景

微信朋友圈点赞，要求按照点赞顺序显示点赞好友信息 如果取消点赞，移除对应好友信息

~~~ java
rpush friend a b c d e
~~~

1. 移除指定的元素

~~~ java
lrem key count value

//移中间的一个d元素，如果列表中同一个元素有多个，那么删除此元素的话会按照顺序删除
lrem friend 1 d
~~~

redis 应用于具有操作先后顺序的数据控制

**List应用场景2**

twitter、新浪微博、腾讯微博中个人用户的关注列表需要按照用户的关注顺序进行展示，粉丝列表需要将最近关注的粉丝列在前面

新闻、资讯类网站如何将最新的新闻或资讯按照发生的时间顺序展示？

企业运营过程中，系统将产生出大量的运营数据，如何保障多台服务器操作日志的统一顺序输出？

![1617962741851](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1617962741851.png)

redis可以将多台服务器的数据全部做聚合操作。

#### List数据类型注意事项

- list中保存的数据都是string类型的，数据总容量是有限的，最多232 - 1 个元素 (4294967295)。

- list具有索引的概念，但是操作数据时通常以队列的形式进行入队出队操作，或以栈的形式进行入栈出栈操作

- 获取全部数据操作结束索引设置为-1
- list可以对数据进行分页操作，通常第一页的信息来自于list，第2页及更多的信息通过数据库的形式加载

**应用场景说明**

1. 依赖list的数据具有顺序的特征对信息进行管理
2. 使用队列模型解决多路信息汇总合并的问题
3. 使用栈模型解决最新消息的问题

### Set类型

- 新的存储需求：存储大量的数据，在查询方面提供更高的效率
- 需要的存储结构：能够保存大量的数据，高效的内部存储机制，便于查询
- set类型：与hash存储结构完全相同，仅存储键，不存储值（nil），并且值是不允许重复的，也就是键不允许重复。

**存储结构**

![1617963260714](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/181422-192241.png)

#### Set类型的基本操作

1. 添加数据

~~~ java
sadd key member1 [member2]

//案例
sadd users xm
//注意，存储的顺序不是按照插入顺序来的
~~~

2. 获取全部数据

~~~ java
smembers key

//案例
smembers users
~~~

3. 删除数据

~~~ java
srem key member1 [member2]

//案例
srem users xr
~~~

4. 获取集合数据总量

~~~ java
scard key

//案例
scard users
~~~

5. 判断集合中是否包含指定数据

~~~ java
sismember key member

//案例
 sismember users xr
~~~

#### 业务场景

每位用户首次使用今日头条时会设置3项爱好的内容，但是后期为了增加用户的活跃度、兴趣点，必须让用户对其他信息类别逐渐产生兴趣，增加客户留存度，如何实现？

**方案实现**

- 系统分析出各个分类的最新或最热点信息条目并组织成set集合
- 随机挑选其中部分信息
- 配合用户关注信息分类中的热点信息组织成展示的全信息集合

1. 随机获取集合中指定数量的数据

~~~ java
srandmember key [count]

//案例 
sadd news n1
//随机获取若干条消息
srandmember news 3
~~~

2. 随机获取集合中的某个数据并将该数据移出集合

~~~ java
spop key [count]

//随机获取消息并且移除消息
spop news 2
~~~

redis 应用于随机推荐类信息检索，例如热点歌单推荐，热点新闻推荐，热卖旅游线路，应用APP推荐，大V推荐等

#### Set扩展操作

1. 求两个集合的交集，并集，差集

~~~ java
sinter key1 [key2] 
sunion key1 [key2] 
sdiff key1 [key2]
~~~

2. 求两个集合的交、并、差集并存储到指定集合中

~~~ java
sinterstore destination key1 [key2] 
sunionstore destination key1 [key2] 
sdiffstore destination key1 [key2]
~~~

3. 将指定数据从原始集合中移动到目标集合中

~~~ java
smove source destination member
~~~

**业务场景2**

集团公司共具有12000名员工，内部OA系统中具有700多个角色，3000多个业务操作，23000多种数据，每位员工具有一个或多个角色，如何快速进行业务操作的权限校验？

![1617966047248](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/09/190113-601094.png)

**代码**

~~~ java
127.0.0.1:6379> sadd rid:001 getall
(integer) 1
127.0.0.1:6379> sadd rid:001 getById
(integer) 1
127.0.0.1:6379> sadd rid:002 getCount
(integer) 1
127.0.0.1:6379> sadd rid:002 getall
(integer) 1
127.0.0.1:6379> sad rid:002 insert
(error) ERR unknown command 'sad'
127.0.0.1:6379> smember rid
(error) ERR unknown command 'smember'
127.0.0.1:6379> smembers rid
(empty list or set)
127.0.0.1:6379> smembers 001
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> scard rid
(integer) 0
//合并多个权限到一个集合中
127.0.0.1:6379> sunionstore uid:007 rid:001 rid:002
(integer) 3
127.0.0.1:6379> smembers uid:007
1) "getById"
2) "getCount"
3) "getall"
~~~

**场景3**

公司对旗下新的网站做推广，统计网站的PV（访问量）,UV（独立访客）,IP（独立IP）。

 PV：网站被访问次数，可通过刷新页面提高访问量 

UV：网站被不同用户访问的次数，可通过cookie统计访问量，相同用户切换IP地址，UV不变

 IP：网站被不同IP地址访问的总次数，可通过IP地址统计访问量，相同IP不同用户访问，IP不变

解决方案

- 利用set集合的数据去重特征，记录各种访问数据
- 建立string类型数据，利用incr统计日访问量（PV）
- 建立set模型，记录不同cookie数量（UV）
- 建立set模型，记录不同IP数量（IP）

``` JAVA
127.0.0.1:6379> sadd 7ips 1.2.3.4
(integer) 1
127.0.0.1:6379> sadd ips 1.2.3.4
(integer) 1
127.0.0.1:6379> sadd ips 2.3.4.5
(integer) 1
127.0.0.1:6379> scard ips
(integer) 2
127.0.0.1:6379> sadd ips 1.2.3.4
(integer) 0
127.0.0.1:6379> scard ips
(integer) 2
//可以看到，重复放入相同的ip，结果不计数
```

> redis 应用于同类型数据的快速去重

**场景4**

黑名单

 资讯类信息类网站追求高访问量，但是由于其信息的价值，往往容易被不法分子利用，通过爬虫技术，快速获取信息，个别特种行业网站信息通过爬虫获取分析后，可以转换成商业机密进行出售。例如第三方火车票、机票、酒店刷票代购软件，电商刷评论、刷好评。



#### set 类型数据操作的注意事项

- set 类型不允许数据重复，如果添加的数据在 set 中已经存在，将只保留一份
- set 虽然与hash的存储结构相同，但是无法启用hash中存储值的空间



### sorted_set类型

- 新的存储需求：数据排序有利于数据的有效展示，需要提供一种可以根据自身特征进行排序的方式
- 需要的存储结构：新的存储模型，可以保存可排序的数据
- sorted_set类型：在set的存储结构基础上添加可排序字段

![1618016891126](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/090812-564584.png)

#### sorted_set 类型数据的基本操作

1. 添加数据

~~~ java
zadd key score1 member1 [score2 member2]

//案例，94代表排序字段
zadd score 94 zs
~~~

2. 获取全部数据

~~~ java
zrange key start stop [WITHSCORES]
zrevrange key start stop [WITHSCORES]

//案例
zrange score 0 -1 //输出的结果是已经排好序的结果，默认排序是从小到大

 zrange score 0 -1 withscores //会把排序字段的值也进行输出
 
  zrevrange score 0 -1// 从大到小排序
~~~

3. 删除数据

~~~ java
zrem key member [member ...]

//案例，删除操作是按照值删除，而不是按照排序字段删除
zrem score li
~~~

4. 按条件获取数据

~~~ java
zrangebyscore key min max [WITHSCORES] [LIMIT]
zrevrangebyscore key max min [WITHSCORES] //反向查询

//案例
zrangebyscore score 80 89 //查询成绩在80-89之间的数据，这个是按照排序字段进行查询的
~~~

5. 按照条件删除数据

~~~ java
zremrangebyrank key start stop //按照索引删除数据
zremrangebyscore key min max //按照数据删除数据
~~~

> min与max用于限定搜索查询的条件，作用于排序字段
> start与stop用于限定查询范围，作用于索引，表示开始和结束索引
> offset与count用于限定查询范围，作用于查询结果，表示开始位置和数据总量

6. 获取集合数据总量

~~~ java
zcard key
zcount key min max
~~~

7. 集合交并操作

~~~ java
zinterstore destination numkeys key [key ...] //numkeys参数表示交集的集合的个数
zunionstore destination numkeys key [key ...]
~~~

#### 应用场景

票选广东十大杰出青年，

各类综艺选秀海选投票 各类资源网站TOP10（电影，歌曲，文档，电商，游戏等） 

聊天室活跃度统计 

游戏好友亲密度

**解决方案**

1. 获取数据对应的索引（排名）

~~~ java
zrank key member 
zrevrank key member

//案例
127.0.0.1:6379> zadd movies 147 aa 198 bb
(integer) 2
127.0.0.1:6379> zadd movies 123 cc
(integer) 1
127.0.0.1:6379> zrank movies bb
(integer) 2
//zrank是按照从小到大排序，输出bb的索引是2
  
//从大到小输出
127.0.0.1:6379> zrevrank movies bb
(integer) 0
~~~

2. score值获取与修改

~~~ java
//获取排序的值
zscore key member 
zincrby key increment member
~~~

redis 应用于计数器组合排序功能对应的排名

**应用场景2**

基础服务+增值服务类网站会设定各位会员的试用，让用户充分体验会员优势。例如观影试用VIP、游戏VIP体验、云盘下载体验VIP、数据查看体验VIP。当VIP体验到期后，如果有效管理此类信息。即便对于正式VIP用户也存在对应的管理方式。 网站会定期开启投票、讨论，限时进行，逾期作废。如何有效管理此类过期信息。

解决方案

- 对于基于时间线限定的任务处理，将处理时间记录为score值，利用排序功能区分处理的先后顺序
- 记录下一个要处理的时间，当到期后处理对应任务，移除redis中的记录，并记录下一个要处理的时间
- 当新任务加入时，判定并更新当前下一个要处理的任务时间
- 为提升sorted_set的性能，通常将任务根据特征存储成若干个sorted_set。例如1小时内，1天内，周内，月内，季内，年度等，操作时逐级提升，将即将操作的若干个任务纳入到1小时内处理的队列中

1. 获取当前系统的时间

~~~ java
time
~~~

redis 应用于定时任务执行顺序管理或任务过期管理

**应用场景3**

任务/消息权重设定应用 当任务或者消息待处理，形成了任务队列或消息队列时，对于高优先级的任务要保障对其优先处理，如何实现任务权重管理。

**方案**

~~~ java
127.0.0.1:6379> zadd task 4 order:id:005
(integer) 1
127.0.0.1:6379> zaddbtask 1 order:id:425  // order:id:425 是排序字段
(error) ERR unknown command 'zaddbtask'
127.0.0.1:6379> zadd task 1 order:id:425
(integer) 1
127.0.0.1:6379> zadd task 9 order:id:345
(integer) 1
127.0.0.1:6379> zreverange task 0 -1 withscores
(error) ERR unknown command 'zreverange'
127.0.0.1:6379> zrevrange task 0 -1 withscores
1) "order:id:345"
2) "9"
3) "order:id:005"
4) "4"
5) "order:id:425"
6) "1"
127.0.0.1:6379>

//下面从队列中删除元素
127.0.0.1:6379> ZREVRANGE task 0 0 //获取第一个数据
1) "order:id:345"
127.0.0.1:6379> zrem task order:id:345
(integer) 1
127.0.0.1:6379> ZREVRANGE task 0 -1 withscores
1) "order:id:005"
2) "4"
3) "order:id:425"
4) "1"
~~~

#### sorted_set 类型数据操作的注意事项

- score保存的数据存储空间是64位，如果是整数范围是-9007199254740992~9007199254740992
- score保存的数据也可以是一个双精度的double值，基于双精度浮点数的特征，可能会丢失精度，使用时候要慎重
- sorted_set 底层存储还是基于set结构的，因此数据不能重复，如果重复添加相同的数据，score值将被反复覆盖，保留最后一次修改的结果

### 数据类型案例

人工智能领域的语义识别与自动对话将是未来服务业机器人应答呼叫体系中的重要技术，百度自研用户评价语义识别服务，免费开放给企业试用，同时训练百度自己的模型。现对试用用户的使用行为进行限速，限制每个用户每分钟最多发起10次调用

![1618021541139](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/102541-665867.png)

**解决方案**

- 设计计数器，记录调用次数，用于控制业务执行次数。以用户id作为key，使用次数作为value
- 在调用前获取次数，判断是否超过限定次数
  - 不超过次数的情况下，每次调用计数+1 
  - 业务调用失败，计数-1
- 为计数器设置生命周期为指定周期，例如1秒/分钟，自动清空周期内使用次数

![1618021632185](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/102712-714016.png)

~~~ java
127.0.0.1:6379> get 415
(nil)
127.0.0.1:6379> setex 415 60 1
OK
127.0.0.1:6379> get 415
"1"
127.0.0.1:6379> incr 415
(integer) 2
127.0.0.1:6379> incrby 415 8
(integer) 10
~~~

但是这种做法需要在次数达到10之后，需要我们自己进行判断，我们可以利用最大值进行改进

- 取消最大值的判定，利用incr操作超过最大值抛出异常的形式替代每次判断是否大于最大值
- 判断是否为nil， 如果是，设置为Max-次数 
  - 如果不是，计数+1 
  - 业务调用失败，计数-1
- 遇到异常即+操作超过上限，视为使用达到上限

![1618021798427](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/102959-69208.png)

## Key通用指令

### Key基本操作

**key的特征**

key是一个字符串，通过key获取redis中保存的数据

**key应设计哪些操作**

- 对于key自身状态的相关操作，例如：删除，判定存在，获取类型等
- 对于key有效性控制相关操作，例如：有效期设定，判定是否有效，有效状态的切换等
- 对于key快速查询操作，例如：按指定策略查询key

1. 删除指定的key

~~~ java
127.0.0.1:6379> set str str
OK
127.0.0.1:6379> hset hash1 hash1 hash1
(integer) 1
127.0.0.1:6379> lpush list1 list1
(integer) 4
127.0.0.1:6379> sad set1 set1
(error) ERR unknown command 'sad'
127.0.0.1:6379> sadd set1 set1
(integer) 1
127.0.0.1:6379> zadd zset1 1 zsete1
//删除指定的key
del key

127.0.0.1:6379> del str
(integer) 1
~~~

2. 获取key是否存在

~~~ java
exists key

//案例
127.0.0.1:6379> exists str
(integer) 1
~~~

3. 获取key的类型

~~~ java
type key

//案例
127.0.0.1:6379> type zset1
zset
127.0.0.1:6379> type str
string
~~~

### Key的扩展操作（时效性控制）

1. 为指定key设置有效期

~~~ java
expire key seconds
pexpire key milliseconds //使用毫秒时间
expireat key timestamp //在linux中使用时间戳
pexpireat key milliseconds-timestamp

//案例，为str设置5秒有效期，超过5秒就失效
127.0.0.1:6379> expire str 5
(integer) 1
127.0.0.1:6379> get str
"str"
127.0.0.1:6379> get str
(nil)
~~~

2. 获取key的有效时间

~~~ java
ttl key 
pttl key //返回毫秒数

//案例
127.0.0.1:6379> expire list1 30
(integer) 1
127.0.0.1:6379> ttl list1
(integer) 27
127.0.0.1:6379> pttl list1
(integer) 20039

//如果设置的时间戳失效后，会返回-2，也就是nil值
127.0.0.1:6379> ttl str
(integer) -2
  
//既没有设置时间戳，也没有失效，会返回-1
127.0.0.1:6379> ttl hash1
(integer) -1
~~~

> ttl：key不存在或者失效，返回-2，key存在，那么返回-1，设置有效期并且没有失效，返回有效时间

3. 切换key从时效性转换为永久性

~~~ java
//修改指令的状态
persist key
~~~

### Key扩展操作（查询模式）

1. 查询key

~~~ java
keys pattern

//案例
keys * // 查询所有的key
~~~

**查询模式规则**

![1618024065335](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/10/110745-726872.png)

**为key改名字**

~~~ java
rename key newkey
renamenx key newkey

//案例
keys str str3
~~~

**对所有key进行排序**

~~~ java
sort //Sort the elements in a list, set or sorted set ，sort仅仅是做排序操作，没有变化元数据
//sort排序不可以对字符串进行排序
//案例
127.0.0.1:6379> lpush aa 111
(integer) 1
127.0.0.1:6379> lpush aa 222
(integer) 2
127.0.0.1:6379> lpush aa 8
(integer) 3
127.0.0.1:6379> lrange 0 -1
(error) ERR wrong number of arguments for 'lrange' command
127.0.0.1:6379> lrange aa 0 -1
1) "8"
2) "222"
3) "111"
127.0.0.1:6379> sort aaa
(empty list or set)
127.0.0.1:6379> sort aa
1) "8"
2) "111"
3) "222"
127.0.0.1:6379> sort aa desc
1) "222"
2) "111"
3) "8"
~~~

**其他key的通用操作**

~~~ java
help @generic
~~~

### 数据库通用指令

key的重复问题

- key是由程序员定义的
- redis在使用过程中，伴随着操作数据量的增加，会出现大量的数据以及对应的key
- 数据不区分种类、类别混杂在一起，极易出现重复或冲突

解决方案

- redis为每个服务提供有16个数据库，编号从0到15
- 每个数据库之间的数据相互独立

![1618640749825](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/142551-290607.png)

1. 切换数据库

~~~ java
select 15 //切换到数据库15
~~~

2. 其他操作

~~~ java
quit
ping
echo message  //打印信息
ping //测试客户端是否和服务器联通

//案例
127.0.0.1:6379[15]> ping abc
"abc"
~~~

3. 移动数据

~~~ java
//注意，移动数据必须保证数据库中有这个数据，并且移动操作必须保证另一个库中没有数据，如果有的话，会移动失败
move key db

//案例
127.0.0.1:6379> set name rzf
OK
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get name
(nil)
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> move name 1
(integer) 1
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get name
"rzf"
~~~

4. 清除数据

~~~ java
dbsize //查看数据库中keys的个数
flushdb //清空某一个数据库
flushall //清空所有的数据库
  
//案例
127.0.0.1:6379[1]> keys *
1) "name"
127.0.0.1:6379[1]> flushdb
OK
127.0.0.1:6379[1]> keys *
(empty list or set)
~~~

## Jedis

### 什么是Jedis

Java语言连接redis服务

![1618645810232](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/17/155013-434353.png)

**Java语言连接redis服务**

- Jedis 
- SpringData
- Redis Lettuce

**可视化连接redis客户端**

- Redis Desktop Manager 
- Redis Client 
- Redis Studio

### Java语言操作Redis

**导入依赖**

~~~ java
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
   <version>2.9.1</version>
</dependency>
~~~

1. 链接redis

~~~ java
//        1 链接redis
        Jedis jedis = new Jedis("127.0.0.1",6379);
~~~

2. 操作redis

~~~ java
//        2 操作redis

        jedis.set("name","rrr");

        String name = jedis.get("name");
        System.out.println(name);
~~~

3. 关闭redis

~~~ java
//        3 关闭redis
        jedis.close();
~~~

### java操作list集合

~~~ java
public class JedisTest_list {

    public static void main(String[] args) {

//        1 链接redis
        Jedis jedis = new Jedis("127.0.0.1",6379);

//        2 操作redis

        jedis.lpush("list","a","b","c","d");
        jedis.rpush("list","x");
        List<String> list = jedis.lrange("list", 0, -1);
        for(String s:list){
            System.out.println(s);
        }
//        输出list的长度
        System.out.println(jedis.llen("list"));


//        3 关闭redis
        jedis.close();
    }
}

~~~

### java操作hash

~~~ java
public class JedisTest_hash {

    public static void main(String[] args) {

//        1 链接redis
        Jedis jedis = new Jedis("127.0.0.1",6379);

//        2 操作redis
//        从redis取出来的数据都会转换为java中的数据类型进行展示

        jedis.hset("hash","a1","aaa");
        jedis.hset("hash","a2","bbb");
        jedis.hset("hash","a3","ccc");

        Map<String, String> hash = jedis.hgetAll("hash");
        System.out.println(hash);
        System.out.println(jedis.hlen("hash"));

//        3 关闭redis
        jedis.close();
    }
}
~~~

## Linux下的Redis

### 下载并且安装redis

**过程**

- 下载安装包
- 解压
- 编译：make
- 安装：make install

**下载redis**

~~~ java
wget https://download.redis.io/releases/redis-4.0.0.tar.gz
~~~

解压后的目录结构

![1619312831912](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619312831912.png)

**安装gcc环境**

由于redis是由C语言编写的，它的运行需要C环境，因此我们需要先安装gcc。安装命令如下：

~~~ JAVA
yum install gcc-c++
  
//如果出现以下错误，
  错误：Package: libstdc++-devel-4.4.7-17.el6.x86_64 (base)
          Requires: libstdc++(x86-64) = 4.4.7-17.el6
          已安装: libstdc++-4.4.7-23.el6.x86_64 (@anaconda-CentOS-201806291108.x86_64/6.10)
              libstdc++(x86-64) = 4.4.7-23.el6
          Available: libstdc++-4.4.7-17.el6.x86_64 (base)
              libstdc++(x86-64) = 4.4.7-17.el6
 You could try using --skip-broken to work around the problem
** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:
2:postfix-2.6.6-8.el6.x86_64 has missing requires of libmysqlclient.so.16()(64bit)
2:postfix-2.6.6-8.el6.x86_64 has missing requires of libmysqlclient.so.16(libmysqlclient_16)(64bit)
2:postfix-2.6.6-8.el6.x86_64 has missing requires of mysql-libs
[root@hadoop100 ~]# 
//执行下面四条命令
yum downgrade libgomp
yum downgrade libstdc++
yum downgrade libgcc
yum downgrade cpp
~~~

进入到 redis目录下，进行编译与安装

~~~ java
//执行make install命令进行安装，
//如果出现下面的错误
[rzf@hadoop100 redis]$ make
cd src && make all
make[1]: Entering directory `/opt/module/redis/src'
    CC adlist.o
在包含自 adlist.c：34 的文件中:
zmalloc.h:50:31: 错误：jemalloc/jemalloc.h：没有那个文件或目录
zmalloc.h:55:2: 错误：#error "Newer version of jemalloc required"
make[1]: *** [adlist.o] 错误 1
make[1]: Leaving directory `/opt/module/redis/src'
make: *** [all] 错误 2
//执行
make distclean

//出现下面错误
[rzf@hadoop100 src]$ make test
    CC Makefile.dep
You need tcl 8.5 or newer in order to run the Redis test
//执行
yum install tcl
~~~

**启动redis**

~~~ java
//进入redis目录中，执行下面的命令
redis-server
//执行下面命令，可以进入客户端
redis-cli
~~~

**redis启动界面**

![1619317808893](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/103010-24035.png)

**redis基础环境配置**

- 创建软连接

  - ln -s 原始目录名 快速访问目录名
- 创建配置文件管理目录
  - mkdir conf
- 创建日志存放目录
  - mkdir data  

### 指定端口号

在企业中，如果想要启动多个redis服务的话，可以更换端口操作。

~~~ java
redis-server --port 6380
//端口号可以自己指定
~~~

![1619318254431](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/103804-818582.png)

可以连接到指定端口的redis服务

~~~ java
redis-cli -p 6380
~~~

![1619318316833](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/103840-424828.png)

### 通过配置文件启动多台服务器

~~~ java
//过滤配置文件的信息，并且写入新的配置文件
cat redis.conf | grep -v "#" | grep -v "^$" > redis-6379.conf

//配置文件修改为一下内容
port 6379
daemonize yes //作为守护进程启动
logfile "6379.log" //日志存放的位置
dir /opt/module/redis/data //日志存放的路径
~~~

- daemonize yes 以守护进程方式启动，使用本启动方式，redis将以服务的形式存在，日志将不再打印到命令窗口中 

- port 6 设定当前服务启动端口号 
- dir “/自定义目录/redis/data“ 设定当前服务文件保存位置，包含日志文件、持久化文件（后面详细讲解）等 
- logfile "6.log“ 设定日志文件名，便于查阅

**使用配置文件的方式启动**

~~~ java
redis-server redis-6379.conf 
//查看进程
ps -ef | grep redis-
  
//直接连接redis服务即可
//这里要注意权限问题
~~~

可以把启动的配置文件存放到新建的conf目录下面，容易管理。

> 默认配置启动
>
> redis-server 
>
> redis-server –-port 6379 
>
> redis-server –-port 6380
>
> 指定配置文件启动
>
> redis-server redis.conf 
>
> redis-server redis-6379.conf 
>
> redis-server redis-6380.conf …… 
>
> redis-server conf/redis-6379.conf 
>
> redis-server config/redis-6380.conf ……
>
> redis客户端连接
>
> 默认连接 ：redis-cli
>
> 连接指定服务器：
>
> redis-cli -h 127.0.0.1 
>
> redis-cli –port 6379 
>
> redis-cli -h 127.0.0.1 –port 6379

## Redis持久化

### 持久化简介

**什么是持久化**

利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复的工作机制称为持久化。

**为什么要持久化**

防止数据的意外丢失，确保数据安全性

**持久化过程保存什么**

- 将当前**数据状态**进行保存，快照形式，存储数据结果，存储格式简单，关注点在数据
- 将数据的**操作过程**进行保存，日志形式，存储操作过程，存储格式复杂，关注点在数据的操作过程

![1619323113827](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/115835-54169.png)

在redis中，既有快照形式的数据结果存储，也就中间过程的步骤存储。

### RDB持久化

#### RDB启动方式

谁，什么时间，干什么事情

命令执行

- 谁：redis操作者（用户）
- 什么时间：即时（随时进行）
- 干什么事情：保存数据

**命令**

~~~ java
save
~~~

作用：执行save命令会在data目录下生成一个rdb结尾的文件，文件中存储的是当前数据库的快照信息。

**save相关指令的配置**

- dbfilename dump.rdb
  - 说明：设置本地数据库文件名，默认值为 dump.rdb 
  - 经验：通常设置为dump-端口号.rdb
- dir
  - 说明：设置存储.rdb文件的路径 
  - 经验：通常设置成存储空间较大的目录中，目录名称data
- rdbcompression yes
  - 说明：设置存储至本地数据库时是否压缩数据，默认为 yes，采用 LZF 压缩 
  - 经验：通常默认为开启状态，如果设置为no，可以节省 CPU 运行时间，但会使存储的文件变大（巨大）
- rdbchecksum yes 
  - 说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行 
  - 经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数据损坏风险

**修改配置文件**

在这里使用的是6379端口号的配置文件

~~~ java
port 6379
daemonize yes
logfile "6379.log"
dir /opt/module/redis/data

//添加下面内容
dbfilename dump-6379.rdb
rdbcompression yes
rdbchecksum yes
~~~

重新写入数据，发现持久化文件名字已经改变。

#### 数据恢复

现在退出redis客户端并且杀死redis服务器进程，现在重新启动服务，并且连接客户端，查询数据发现已经全部恢复，redis是在启动服务的时候，重新从持久化文件中加载数据。

#### save指令的工作原理

![1619324786412](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/25/122627-916427.png)

多个客户端连接同一个服务器，那么客户端发出的指令会按照先后的顺序排入一个队列中顺序发给服务器执行，但是如果rdb持久化过程很慢的话，可能会造成延迟过高，阻塞save后面的指令。

#### 后台执行

数据量过大，单线程执行方式造成效率过低如何处理？

**后台执行**

- 谁：redis操作者（用户）发起指令；redis服务器控制指令执行
- 什么时间：即时（发起）；合理的时间（执行）
- 干什么事情：保存数据

**命令**

~~~ java
bgsave
~~~

作用：手动启动后台保存操作，但不是立即执行

![1619325155397](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619325155397.png)

save指令是如果调用，那么就会立即执行持久化操作，但是bgsave的话会在后台使用folk命令创建一个子进程，子进程负责持久化数据。两个指令保存的文件都是同一个文件。

**工作原理**

![1619326806599](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619326806599.png)

注意： bgsave命令是针对save阻塞问题做的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式，save命令可以放弃使用。

**bgsave指令相关配置**

- dbfilename dump.rdb
- dir
- rdbcompression yes
- rdbchecksum yes
- stop-writes-on-bgsave-error yes 
  - 说明：后台存储过程中如果出现错误现象，是否停止保存操作 
  - 经验：通常默认为开启状态

#### 自动持久化操作

反复执行保存指令，忘记了怎么办？不知道数据产生了多少变化，何时保存？

自动执行

- 谁：redis服务器发起指令（基于条件）
- 什么时间：满足条件
- 干什么事情：保存数据

**指令**

~~~ java
save second changes
~~~

作用：满足限定时间范围内key的变化数量达到指定数量即进行持久化

参数

- second：监控时间范围 
- changes：监控key的变化量

位置

- 在conf文件中进行配置

案例

~~~ java
save 900 1 
save 300 10 
save 60 10000
~~~

也就是说在指定的时间内如果有change课key发生变化，那么在后台就会执行bgsave指令进行持久化操作。

**save配置**

![1619397751356](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619397751356.png)

> save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的
> save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系
> save配置启动后执行的是bgsave操作

#### RDB特殊启动方式

- 全量复制
  - 主从复制
- 服务器运行过程中重启

~~~ java
debug reload
~~~

- 关闭服务器时指定保存数据

~~~ java
shutdown save
~~~

> 默认情况下执行shutdown命令时，自动执行bgsave(如果没有开启AOF持久化功能)

#### RDB三种启动方式对比

![1619397991621](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/084633-118367.png)

**RDB优点**

- RDB是一个紧凑压缩的二进制文件，存储效率较高
- RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景
- RDB恢复数据的速度要比AOF快很多
- 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

**RDB缺点**

- RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
- bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
- Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象

### AOF持久化

#### RDB存在的缺点

- 存储数据量较大，效率较低
- 基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低
- 大数据量下的IO性能较低
- 基于fork创建子进程，内存产生额外消耗
- 宕机带来的数据丢失风险，是基于某一个时间点存储的数据，所以没有实时性，会丢失最新的数据。

**解决方案**

- 不写全数据，仅记录部分数据
- 降低区分数据是否改变的难度，改记录数据为记录操作过程
- 对所有操作均进行记录，排除丢失数据的风险

#### AOF概念

- AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程
- AOF的主要作用是解决了数据持久化的**实时性**，目前已经是Redis持久化的主流方式

#### AOF写数据过程

![1619398763365](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/085928-145728.png)

aof首先将从客户端接收的指令存储在缓存区中，然后等待时机同步到aof文件中，同步的时机分为下面三种情况。

#### AOF写数据三种策略(appendfsync)

- always(每次）
  - 每次写入操作均同步到AOF文件中，**数据零误差，性能较低**，不建议使用。
- everysec（每秒）
  - 每秒将缓冲区中的指令同步到AOF文件中，**数据准确性较高，性能较高在系统突然宕机的情况下丢失1秒内的数据**，，建议使用，也是默认配置
- no（系统控制）
  - 由操作系统控制每次同步到AOF文件的周期，整体**过程不可控**

#### AOF功能的开启

配置一

~~~ java
appendonly yes|no
~~~

作用:是否开启AOF持久化功能，默认为不开启状态

配置二

~~~ java
appendfsync always|everysec|no
~~~

作用:AOF写数据策略

配置三

~~~ java
appendfilename filename
~~~

作用:AOF持久化文件名，默认文件名未appendonly.aof，建议配置为appendonly-端口号.aof

配置四

~~~ java
dir
~~~

作用:AOF持久化文件保存路径，与RDB持久化文件保持一致即可

修改配置文件后重新启动，可以发现，已经产生新的文件

![1619399627734](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/091354-295888.png)

#### AOF重写

![1619399893082](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619399893082.png)

上面出现的问题是如果对一个数每次都增加1，增加3次最终值是3，但是aof文件中记录了三次累加操作，并且每次都是增加1，这样记录的话很占资源，所以采用的次略是合并这三条指令，直接把最终的值变为3，这样的功能在redis中叫做重写操作。

**aof重写**

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录。

**AOF重写作用**

- 降低磁盘占用量，提高磁盘利用率
- 提高持久化效率，降低持久化写时间，提高IO性能
- 降低数据恢复用时，提高数据恢复效率

**aof重写的规则**

- 进程内已超时的数据不再写入文件
- 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令
  如del key1、 hdel key2、srem key3、set key4 111、set key4 222等
- 对同一数据的多条写命令合并为一条命令
  如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。
- 为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素

**工作原理**

![1619401470988](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/094442-805296.png)

**手动重写**

~~~ java
bgrewriteaof
~~~

![1619400586336](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619400586336.png)

**自动重写**

自动重写触发条件

~~~ java
auto-aof-rewrite-min-size size//设定进行重写的最小文件大小，注意这个值和下面的aof_current_size配合使用，也就是说如果最小重写设置为60，如果缓冲区小于60，那么是不会重写的，但是如果大于60，那么就会进行重写
auto-aof-rewrite-percentage percent //自动重写百分比，达到这个百分比后就进行重写操作，需要和基础尺寸进行配合使用
~~~

自动重写触发比对参数（ 运行指令info Persistence获取具体信息 ）

~~~ java
aof_current_size//表示aof缓存区的大小
aof_base_size
~~~

自动重写触发条件

![1619401022779](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619401022779.png)

在客户端输入info命令可以查看到默认值的大小

![1619401197369](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/093959-585284.png)

**aof工作流程**

![1619401551356](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619401551356.png)

如果 是always,那么每一次客户端发送过来的指令，主进程执行命令，同时启动folk子进程，然后子进程把指令写入到aof文件中，

如果配置的是everysec,主进程首先会执行指令，然后创建一个子进程，子进程会把指令首先装入aof的一个缓冲区中，等待满足条件或者时间限制后（条件指的就是每秒中），就会把缓冲区中的指令写入aof文件中，如果发生丢失数据，丢失的数据就是缓冲区中的数据。

上面两种方式都是基于非重写，下面考虑重写功能。

![1619401871196](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/26/095114-663786.png)

基于everysec开启重写的功能，在子进程把指令写入缓存区时候，还会有一个重写缓存区域，专门用来执行重写操作。

![1619401899997](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619401899997.png)

如果执行了bgrewriteaof指令之后，首先会基于bgrewriteaof的主进程创建一个子进程，然后给出提示信息，然后重写新的aof文件，重写数据的来源是aof重写缓存区中的数据，然后重写完成后就会把重写的文件替换原来的重写后的aof文件，而基于bgrewriteaof指令产生的重写aof文件是一个临时文件。

AOF缓冲区同步文件策略，由参数appendfsync控制
系统调用write和fsync说明：

- write操作会触发延迟写（delayed write）机制，Linux在内核提供页缓冲区用来提高硬盘IO性能。write操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，列如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失。
- fsync针对单个文件操作（比如AOF文件），做强制硬盘同步，fsync将阻塞知道写入硬盘完成后返回，保证了数据持久化。

### RDB与AOF的区别

![1619402880464](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619402880464.png)

**两两者如何选择**

- 对数据非常敏感，建议使用默认的AOF持久化方案
  - AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出
    现问题时，最多丢失0-1秒内的数据。
  - 注意：由于AOF文件存储体积较大，且恢复速度较慢
- 数据呈现阶段有效性，建议使用RDB持久化方案
  - 数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段
    点数据恢复通常采用RDB方案
  - 注意：利用RDB实现紧凑的数据持久化会使Redis降的很低，慎重总结：
- 综合比对
  - RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊
  - 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF
  - 如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB
  - 灾难恢复选用RDB
  - 双保险策略，同时开启 RDB 和 AOF，重启后，Redis优先使用 AOF 来恢复数据，降低丢失数据的量

### 持久化的应用场景分析

- Tips 1：redis用于控制数据库表主键id，为数据库表主键提供生成策略，保障数据库表的主键唯一性
  - 不建议使用redis数据库，可能会导致主键重复
- Tips 3：redis应用于各种结构型和非结构型高热度数据访问加速
  - 不需要持久化
- Tips 4：redis 应用于购物车数据存储设计
  - 不需要
- Tips 5：redis 应用于抢购，限购类、限量发放优惠卷、激活码等业务的数据存储设计
  - 需要，因为是抢购速度很快，延迟非常的小，下次直接从持久化文件中加载数据即可。
- Tips 6：redis 应用于具有操作先后顺序的数据控制
  - 需要，提高访问速度
- Tips 7：redis 应用于最新消息展示
  - 需要
- Tips 9：redis 应用于同类信息的关联搜索，二度关联搜索，深度关联搜索
  - 不需要，可以从数据库中加载，内容多，不适合持久化
- Tips 12：redis 应用于基于黑名单与白名单设定的服务控制
  - 黑名单长期策略，那么适合在数据库中，如果是短期，那么适合持久化，白名单不需要持久化，一般在数据库中。
- Tips 13：redis 应用于计数器组合排序功能对应的排名
  - 建议做持久化
- Tips 15：redis 应用于即时任务/消息队列执行管理
  - 可以使用专门的队列解决
- Tips 16：redis 应用于按次结算的服务控制
  - 不需要持久化

**小结**

![1619403846977](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/084735-589107.png)

## 事务

### 什么是事务

Redis执行指令过程中，多条连续执行的指令被干扰，打断，插队

redis事务就是一个命令执行的队列，将一系列预定义命令包装成一个整体（一个队列）。当执行时，一次性按照添加顺序依次执行，中间不会被打断或者干扰。

一个队列中，一次性、顺序性、排他性的执行一系列命令

### 事务的基本操作

**开启事务**

~~~ java
//如果想开启一条事务，使用下面命令即可
multi
~~~

作用:设定事务的开启位置，此指令执行后，后续的所有指令均加入到事务中

**执行事务**

~~~ java
exec
~~~

 作用:设定事务的结束位置，同时执行事务。与multi成对出现，成对使用

> 注意：加入事务的命令暂时进入到任务队列中，并没有立即执行，只有执行exec命令才开始执行

**取消事务**

~~~ java
discard
~~~

作用:终止当前事务的定义，发生在multi之后，exec之前

> mult和exec应该是成对出现使用的，discard在开启事务和执行事务中间执行。
>
> 事务其实就是一个队列，只有当前的客户端才可以进行数据的操作，如果其他的客户端发送指令，不会加入队列执行。

### 事务的工作流程

![1619485035851](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/085719-436256.png)

### 事务的注意事项

定义事务的过程中，命令格式输入错误怎么办？

- 语法错误
  - 指命令书写格式有误
- 处理结果
  - 如果定义的事务中所包含的命令存在语法错误，整体事务中所有命令均不会执行。包括那些语法正确的命令。
  - 也可以认为是事务失效。

定义事务的过程中，命令执行出现错误怎么办？

- 运行错误
  - 指命令格式正确，但是无法正确的执行。例如对list进行incr操作
- 处理结果
  - 能够正确运行的命令会执行，运行错误的命令不会被执行

![1619485461213](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/090423-260867.png)

> 注意：已经执行完毕的命令对应的数据不会自动回滚，需要程序员自己在代码中实现回滚。

### 石手动进行事务回滚

记录操作过程中被影响的数据之前的状态

- 单数据：string
- 多数据：hash、list、set、zset

设置指令恢复所有的被修改的项

- 单数据：直接set（注意周边属性，例如时效）
- 多数据：修改对应值或整体克隆复制

### 基于特定条件的事务执行——锁

多个客户端监控一个key，如果key发生变化，那么其他客户端对key的事务执行将会被取消。注意是整个事务都不会被执行。

**对 key 添加监视锁，在执行exec前如果key发生了变化，终止事务执行**

~~~ java
watch key1 [key2……]
~~~

**取消对所有 key 的监视**

~~~ java
unwatch
//注意，这条语句是取消对全部key的监控
~~~

### 基于特定条件的事务执行——分布式锁

分布式锁意为在多线程下共享变量的线程安全问题。

可以比喻为java中的synchronized，也就是同一时间只能有一个线程获取到锁对数据进行操作，其他线程获取锁会阻塞。多个线程必须使用同一个锁对象。

**使用 setnx 设置一个公共锁**

~~~ java
setnx lock-key value
~~~

利用setnx命令的返回值特征，有值则返回设置失败，无值则返回设置成功

- 对于返回设置成功的，拥有控制权，进行下一步的具体业务操作
- 对于返回设置失败的，不具有控制权，排队或等待

操作完毕通过del操作释放锁,当前线程把锁释放掉之后，其他的线程才可以抢占获取锁。

> 注意：上述解决方案是一种设计概念，依赖规范保障，具有风险性

### 基于特定条件的事务执行——分布式锁改良

这里的死锁并不是操作系统层面的死锁概念，而是线程对共享变量加锁后忘记释放导致其他的线程不能获取到锁而导致的死锁问题。

使用 expire 为锁key添加时间限定，到时不释放，放弃锁

~~~ java
expire lock-key second
pexpire lock-key milliseconds
~~~

由于操作通常都是微秒或毫秒级，因此该锁定时间不宜设置过大。具体时间需要业务测试后确认。

- 例如：持有锁的操作最长执行时间127ms，最短执行时间7ms。
- 测试百万次最长执行时间对应命令的最大耗时，测试百万次网络延迟平均耗时
- 锁时间设定推荐：最大耗时*120%+平均网络延迟*110%
- 如果业务最大耗时<<网络平均延迟，通常为2个数量级，取其中单个耗时较长即可

## 删除策略

### Redis中的数据特征

Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过TTL指令获取其状态

- XX ：具有时效性的数据
- -1 ：永久有效的数据
- -2 ：已经过期的数据 或 被删除的数据 或 未定义的数据

### 删除数据的策略-内部实现

![1619489286669](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619489286669.png)

可以看到，左边的四个指令都可以给redis数据添加时效，也就是设定时效的时间，而在redis的内部，是以hash结构，也就是键值对的形式进行存储的，键存储的是键的地址，而值存储的是键的时效时间，以后就根据时效时间判断某一个键是否时效。

在内存占用与CPU占用之间寻找一种平衡，顾此失彼都会造成整体redis性能的下降，甚至引发服务器宕机或内存泄露

### 定时删除

创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作

- 优点：节约内存，到时就删除，快速释放掉不必要的内存占用
- 缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量
- 总结：用处理器性能换取存储空间（拿时间换空间）

### 惰性删除

数据到达过期时间，不做处理。等下次访问该数据时

- 如果未过期，返回数据
- 发现已过期，删除，返回不存在
- expireIfNeeded()此函数就是用来检查数据是否过期。

优点：节约CPU性能，发现必须删除的时候才删除
缺点：内存压力很大，出现长期占用内存的数据
总结：用存储空间换取处理器性能 ，（拿时间换空间）

### 定期删除

定期删除是定时删除和惰性删除的折中方案。

![1619491308128](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/104149-421292.png)

- serverCron()是轮训检查服务器，控制服务器每秒中践行多少次下面两种操作 
- databasesCron()是检查数据库
- activeExpireCycle()是检查数据库中的keys。
- current_db用于记录操作到第几个数据库

每秒钟需要进行若干次serverCron()操作，每次操作中轮训对数据库进行访问，对每一个数据库访问时候对其中的keys进行检查，去掉无效的key。访问的策略是随机挑选若干个key。



![1619490838956](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619490838956.png)

redis中有一块expires存储空间，里面存储的是过期的存储地址和时间，[0]表示第0个数据库。

启动redis时会读取server.hz文件中的值，每一秒对服务器进行轮训检查，然后继续对服务器中的每一个库进行轮训操作，接着会检查每一个库中的数据信息，也就是检查每一个库中的key，检查策略是随机挑选，挑去w个key，如果w中过期的key，如果发现上一次删除的key的数量比较大，那么就接着对剩下的key继续进行检测，还是随机挑选，然后删除。如果发现某一轮删除数据量比较少，那么就检查下一个数据库，逐个进行检查。

**定期删除小结**

周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度

- 特点1：CPU性能占用设置有峰值，检测频度可自定义设置
- 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理
- 总结：周期性抽查存储空间（随机抽查，重点抽查）

**删除策略对比**

![1619491959890](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/105240-360492.png)

在redis中使用的是定期删除+惰性删除策略。

### 逐出算法

当新数据进入redis时，如果内存不足怎么办？

- Redis使用内存存储数据，在执行每一个命令前，会调用freeMemoryIfNeeded()检测内存是否充足。如
  果内存不满足新加入数据的最低存储要求，redis要临时删除一些数据为当前指令清理存储空间。清理数据
  的策略称为逐出算法。
- 注意：逐出数据的过程不是100%能够清理出足够的可使用的内存空间，如果不成功则反复执行。当对所
  有数据尝试完毕后，如果不能达到内存清理的要求，将出现错误信息。

![1619496270357](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/120431-781490.png)

**影响数据逐出的相关配置**

最大可使用内存

~~~ java
maxmemory
~~~

占用物理内存的比例，默认值为0，表示不限制。生产环境中根据需求设定，通常设置在50%以上。

每次选取待删除数据的个数

~~~ java
maxmemory-samples
~~~

选取数据时并不会全库扫描，导致严重的性能消耗，降低读写性能。因此采用随机获取数据的方式作为待检测删除数据

删除策略

~~~ java
maxmemory-policy
~~~

 达到最大内存后的，对被挑选出来的数据进行删除的策略

**检测易失数据（可能会过期的数据集server.db[i].expires ）**

1. volatile-lru：挑选最近最少使用的数据淘汰
2. volatile-lfu：挑选最近使用次数最少的数据淘汰
3. volatile-ttl：挑选将要过期的数据淘汰
4. volatile-random：任意选择数据淘汰

**检测全库数据（所有数据集server.db[i].dict ）**

- allkeys-lru：挑选最近最少使用的数据淘汰
- allkeys-lfu：挑选最近使用次数最少的数据淘汰
- allkeys-random：任意选择数据淘汰

**放弃数据驱逐**

no-enviction（驱逐）：禁止驱逐数据（redis4.0中默认策略），会引发错误OOM（Out Of Memory）

> 需要在配置文件中配置
>
>  使用INFO命令输出监控信息，查询缓存 hit 和 miss 的次数，根据业务需求调优Redis配置
>
> 删除操作是一种加速运行效率的策略，目标是减少无效key的数量，提高性能，加速查询。

### 小结

![1619496674167](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/121116-336130.png)

## 服务器配置

### 服务器端的配置

1. 设置服务器以守护进程的方式运行

~~~ java
daemonize yes|no
~~~

2. 绑定主机地址

~~~ java
bind 127.0.0.1
~~~

如果绑定主机的ip地址，那么只能通过ip访问redis，如果不绑定，其他客户可以通过其他的ip进行访问。

3. 设置服务器端口号

~~~ java
port 6379
~~~

4. 设置数据库数量

~~~ java
databases 16
~~~

### 日志配置

1. 设置服务器以指定日志记录级别

~~~ java
loglevel debug|verbose|notice|warning
~~~

2. 日志记录文件名

~~~ java
logfile 端口号.log
~~~

> 注意：日志级别开发期设置为verbose即可，生产环境中配置为notice，简化日志输出量，降低写日志IO的频度

### 客户端配置

1. 设置同一时间最大客户端连接数，默认无限制。当客户端连接到达上限，Redis会关闭新的连接

~~~ java
maxclients 0
~~~

2. 客户端闲置等待最大时长，达到最大值后关闭连接。如需关闭该功能，设置为 0

~~~ java
timeout 300
~~~

### 多服务器快捷配置

1. 导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护

~~~ java
include /path/server-端口号.conf
~~~

## 高级数据类型

### Bitmaps

**存储需求**

![1619956063200](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/194746-354174.png)

**Bitmaps类型的基础操作**

- 获取指定key对应偏移量上的bit值

~~~ java
getbit key offset
~~~

- 设置指定key对应偏移量上的bit值，value只能是1或0

~~~ java
setbit key offset value
~~~

- 基本操作

~~~ java
127.0.0.1:6379> setbit bit 0 1
(integer) 0
127.0.0.1:6379> getbit bit 0
(integer) 1
~~~

**Bitmaps类型的扩展操作**

![1619957001120](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/200324-844339.png)



- 对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中

~~~ java
bitop op destKey key1 [key2...]
//这里的op代表是操作，可以使用下面的运算
and：交
or：并
not：非
xor：异或
//最后可以通过
bitcount destkey进行查看结果
~~~

- 统计指定key中1的数量

~~~ java
bitcount key [start end]
//表示统计一个bitmaps里面1的个数
~~~

### HyperLogLog

统计不重复数据的数量。

**统计独立UV**

- 原始方案：set
  - 存储每个用户的id（字符串）
- 改进方案：Bitmaps
  - 存储每个用户状态（bit）
- 全新的方案：Hyperloglog

**基数**

基数是数据集去重后元素个数

- HyperLogLog 是用来做基数统计的，运用了LogLog的算法

![1619957985726](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619957985726.png)

**HyperLogLog类型的基本操作**

- 添加数据

~~~ java
pfadd key element [element ...]
~~~

- 统计数据

~~~ java
pfcount key [key ...]
~~~

- 合并数据

~~~ java
pfmerge destkey sourcekey [sourcekey...]
~~~

**相关说明**

- 用于进行基数统计，不是集合，不保存数据，只记录数量而不是具体数据
- 核心是基数估算算法，最终数值存在一定误差
- 误差范围：基数估计的结果是一个带有 0.81% 标准错误的近似值
- 耗空间极小，每个hyperloglog key占用了12K的内存用于标记基数
- pfadd命令不是一次性分配12K内存使用，会随着基数的增加内存逐渐增大，但是上限是12kb。
- Pfmerge命令合并后占用的存储空间为12K，无论合并之前数据量多少

### GEO

这种类型是用来记录坐标的。

- 添加坐标点

~~~ java

geoadd key longitude latitude member [longitude latitude member ...]

127.0.0.1:6379> geoadd pos 1 1 a 
(integer) 1
127.0.0.1:6379> geoadd pos1 1 1 b 
(integer) 1
  
//添加的坐标是在一个容器中，不可以从一个容器中去取另外一个容器中的内容
~~~

- 获取点坐标

~~~ java
geopos key member [member ...]

127.0.0.1:6379> geopos pos a
1) 1) "0.99999994039535522"
   2) "0.99999945914297683"
127.0.0.1:6379> geopos pos1 b
1) 1) "0.99999994039535522"
   2) "0.99999945914297683"
~~~

- 计算坐标点距离

~~~ java
geodist key member1 member2 [unit]

127.0.0.1:6379> geodist pos a b
"157270.0561"

//必须在一个群组中计算距离，默认是以m为单位
//也可以使用千米为单位
127.0.0.1:6379> geodist pos a b km
"157.2701"
~~~

**其他操作**

- 添加坐标点

~~~ java
georadius key longitude latitude radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]

~~~

- 湖获取坐标点

~~~ java
georadiusbymember key member radius m|km|ft|mi [withcoord] [withdist] [withhash] [count count]
~~~

- 计算经纬度

~~~ java
geohash key member [member ...]
~~~

###  小结

![1619959345857](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/204535-206132.png)

## 集群-主从复制

### 主从复制简介

**互联网“三高”架构**

- 高并发：搭建的环境必须可以支持大量的用户同时访问我们的服务。
- 高性能：性能要求要好。
- 高可用

![1619500427287](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/131348-803494.png)

高可用，服务器宕机的时间占据总得时间比例越低越好。

#### 单机redis的风险与问题

- 问题1.机器故障
  - 现象：硬盘故障、系统崩溃
  - 本质：数据丢失，很可能对业务造成灾难性打击
  - 结论：基本上会放弃使用redis.

- 问题2.容量瓶颈
  - 现象：内存不足，从16G升级到64G，从64G升级到128G，无限升级内存
  - 本质：穷，硬件条件跟不上
  - 结论：放弃使用redis

- 结论：
  - 为了避免单点Redis服务器故障，准备多台服务器，互相连通。将数据复制多个副本保存在不同的服
    务器上，连接在一起，并保证数据是同步的。即使有其中一台服务器宕机，其他服务器依然可以继续
    提供服务，实现Redis的高可用，同时实现数据冗余备份。

#### 多台服务器连接方案

![1619500937605](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619500937605.png)

- 提供数据方：master 主服务器，主节点，主库 主客户端
- 接收数据方：slave 从服务器，从节点，从库 从客户端
- 需要解决的问题： 数据同步
- 核心工作： master的数据复制到slave中

主从复制即将master中的数据即时、有效的复制到slave中

特征：一个master可以拥有多个slave，一个slave只对应一个master

**职责：**

- master:
  - 写数据
  - 执行写操作时，将出现变化的数据自动同步到slave
  - 读数据（可忽略）
- slave:
  - 读数据
  - 写数据（禁止）

#### 高可用集群

![1619501276215](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619501276215.png)

**好处**

- 读写分离：master写、slave读，提高服务器的读写负载能力，也就是负载均衡。
- 负载均衡：基于主从结构，配合读写分离，由slave分担master负载，并根据需求的变化，改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量
- 故障恢复：当master出现问题时，由slave提供服务，实现快速的故障恢复
- 数据冗余：实现数据热备份，是持久化之外的一种数据冗余方式
- 高可用基石：基于主从复制，构建哨兵模式与集群，实现Redis的高可用方案

### 主从复制的工作流程

#### 总述

![1619501594867](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619501594867.png)

主从复制过程大体可以分为3个阶段

1. 建立连接阶段（即准备阶段），因为一个master可以对应多个slave节点，所以如果有新的slave节点上线，那么新节点首先会和master建立链接。
2. 数据同步阶段：第一步建立链接之后，slave和master会相互同步数据，把master中的数据同步到slave节点中，
3. 命令传播阶段：因为master节点一直在接受命令执行操作，所以第二部之后master还要和slave一直保持链接进行数据的同步工作。

![1619502629721](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/135031-458924.png)

#### 建立链接

建立slave到master的连接，使master能够识别slave，并保存slave端口号

![1619502703168](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/135145-985391.png)

**方式一**

客户端发送命令

~~~ java
slaveof <masterip> <masterport>
~~~

![1619506786436](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/145949-677653.png)

**方式二：启动服务器参数**

~~~ java
redis-server -slaveof <masterip> <masterport>
~~~

案例

y也可以把链接的master节点之间添加到配置文件中。

~~~ java
 redis-cli /opt/module/redis/conf/redis-6380.conf --slaveof 127.0.0.1 6379
~~~

**方式三：服务器配置**

~~~ java
slaveof <masterip> <masterport>
~~~

![1619506945989](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619506945989.png)

在配置文件中配置链接的master信息

**主从断开链接**

客户端发送命令

~~~ java
slaveof no one
~~~

说明： slave断开连接后，不会删除已有数据，只是不再接受master发送的数据

**授权访问**

1. master客户端发送命令设置密码

~~~ java
requirepass <password>
~~~

2. master配置文件设置密码

~~~ java
config set requirepass <password> 
config get requirepass
~~~

3. slave客户端发送命令设置密码

~~~ java
auth <password>
~~~

4. slave配置文件设置密码

~~~ java
masterauth <password>
~~~

5. slave启动服务器设置密码

~~~ java
redis-server –a <password>
~~~

#### 阶段二：数据同步阶段工作流程

- 在slave初次连接master后，复制master中的所有数据到slave
- 将slave的数据库状态更新成master当前的数据库状态

![1619508036884](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/27/152038-277318.png)

1. slave节首先需要向master节点发送psync2命令，请求同步数据
2. master接收到命令会执行bgsave操作，也就是备份操作，在执行bgsave的过程中，还会接受数据。在备份过程中中间还会接受数据，接收的数据存放在缓存中，所以缓存中的数据无法发送给slave节点。
3. master端生成的rdb文件最终会通过socket发送给slave节点。
4. slave节点在接收到rdb文件之后，不管slave之前有什么数据，他都会做清空工作，然后执行rdb文件恢复过程。然后返回信息给master节点，告诉master节点已经同步完数据。
5. slave发送同步完成的命令给master的作用就是请求同步缓冲区中的数据信息。
6. master会把缓冲区中的数据发送给slave节点，然后slave节点就会执行恢复数据。
7. 创建rdb数据同步文件是数据的全量复制，缓冲区中的数据是增量复制。

> 全量数据：获取从slave发送指令那一刻开始之前的全部数据。
>
> 部分复制：发送slave发送指令之后master节点在缓冲区中存放的数据
>
> 最终slave会获取和master全部一样的数据，而在master端也会记录当前slave节点已经同步完数据的位置。 

**数据同步阶段master说明**

- 如果master数据量巨大，数据同步阶段应避开流量高峰期，避免造成master阻塞，影响业务正常执行
- 复制缓冲区大小设定不合理，会导致数据溢出。如进行全量复制周期太长，进行部分复制时发现数据已经存在丢失的情况，必须进行第二次全量复制，致使slave陷入死循环状态。

~~~ java
//这个时候需要对master缓冲区的大小进行设置
repl-backlog-size 1mb
~~~

- master单机内存占用主机内存的比例不应过大，建议使用50%-70%的内存，留下30%-50%的内存用于执行bgsave命令和创建复制缓冲区

**数据同步阶段slave说明**

1. 为避免slave进行全量复制、部分复制时服务器响应阻塞或数据不同步，建议关闭此期间的对外服务

~~~ java
slave-serve-stale-data yes|no
~~~

2. 数据同步阶段，master发送给slave信息可以理解master是slave的一个客户端，主动向slave发送命令
3. 多个slave同时对master请求数据同步，master发送的RDB文件增多，会对带宽造成巨大冲击，如果master带宽不足，因此数据同步需要根据业务需求，适量错峰
4. slave过多时，建议调整拓扑结构，由一主多从结构变为树状结构，中间的节点既是master，也是slave。注意使用树状结构时，由于层级深度，导致深度越高的slave与最顶层master间数据同步延迟较大，数据一致性变差，应谨慎选择

#### 阶段三：命令传播阶段

- 当master数据库状态被修改后，导致主从服务器数据库状态不一致，此时需要让主从数据同步到一致的状态，同步的动作称为命令传播
- master将接收到的数据变更命令发送给slave，slave接收命令后执行命令

**命令传播阶段的部分复制**

- 命令传播阶段出现了断网现象
  - 网络闪断闪连，忽略 
  - 短时间网络中断，部分复制
  - 长时间网络中断， 全量复制
- 部分复制的三个核心要素
  - 服务器的运行 id（run id）
  - 主服务器的复制积压缓冲区
  - 主从服务器的复制偏移量

**服务器运行ID（runid）**

- 概念：服务器运行ID是每一台服务器每次运行的身份识别码，一台服务器多次运行可以生成多个运行id

- 组成：运行id由40位字符组成，是一个随机的十六进制字符

~~~ java
例如：fdc9ff13b9bbaab28db42b3d50f852bb5e3fcdce
~~~

- 作用：运行id被用于在服务器间进行传输，识别身份 如果想两次操作均对同一台服务器进行，必须每次操作携带对应的运行id，用于对方识别
- 实现方式：运行id在每台服务器启动时自动生成的，master在首次连接slave时，会将自己的运行ID发送给slave，slave保存此ID，通过info Server命令，可以查看节点的runid

**复制缓冲区**

概念：复制缓冲区，又名复制积压缓冲区，是一个先进先出（FIFO）的队列，用于存储服务器执行过的命
令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区，复制缓冲区是一个先进先出的队列。

![1619595172924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/28/153258-876303.png)

**复制缓冲区的内部工作原理**

![1619595210423](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/28/153333-276158.png)

offset的位置一定是在一个完整的指令处结束，offset来记录master和slave传输到哪一个位置。master和slavez都需要记录offset的位置，防止发生数据丢失。master记录的offset表示目前位置传输数据的位置，而slave记录的offset表示目前位置接收的数据的位置。

**复制缓冲区补充**

概念：复制缓冲区，又名复制积压缓冲区，是一个先进先出（FIFO）的队列，用于存储服务器执行过的命
令，每次传播命令，master都会将传播的命令记录下来，并存储在复制缓冲区

- 复制缓冲区默认数据存储空间大小是1M，由于存储空间大小是固定的，当入队元素的数量大于队
  列长度时，最先入队的元素会被弹出，而新元素会被放入队列

由来：每台服务器启动时，如果开启有AOF或被连接成为master节点，即创建复制缓冲区

作用：用于保存master收到的所有指令（仅影响数据变更的指令，例如set，select）

数据来源：当master接收到主客户端的指令时，除了将指令执行，会将该指令存储到缓冲区中

**主从服务器复制偏移量（offset）**

概念：一个数字，描述复制缓冲区中的指令字节位置

分类：

- master复制偏移量：记录发送给所有slave的指令字节对应的位置（多个）
- slave复制偏移量：记录slave接收master发送过来的指令字节对应的位置（一个）

数据来源：

- master端：发送一次记录一次
- slave端：接收一次记录一次

作用：同步信息，比对master与slave的差异，当slave断线后，恢复数据使用

**数据同步+命令传播阶段工作流程**

主从复制的工作流程

![1619670729091](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/29/123213-498637.png)

**心跳机制**

进入命令传播阶段候，master与slave间需要进行信息交换，使用心跳机制进行维护，实现双方连接保持在线

- master心跳：
  - 指令：PING
  - 周期：由repl-ping-slave-period决定，默认10秒
  - 作用：判断slave是否在线
  - 查询：INFO replication 获取slave最后一次连接时间间隔，lag项维持在0或1视为正常
  - master的心跳主要用来判断是否和slave连线。
-  slave心跳任务
  - 指令：REPLCONF ACK {offset}
  - 周期：1秒
  - 作用1：汇报slave自己的复制偏移量，获取最新的数据变更指令
  - 作用2：判断master是否在线

**心跳阶段注意事项**

当slave多数掉线，或延迟过高时，master为保障数据稳定性，将拒绝所有信息同步操作

~~~ java
min-slaves-to-write 2 //当slave个数小于这个参数的设定值的时候，停止同步信息
min-slaves-max-lag 8 //当延迟大于这个设定的值的时候，停止同步
~~~

slave数量少于2个，或者所有slave的延迟都大于等于10秒时，强制关闭master写功能，停止数据同步

- slave数量由slave发送REPLCONF ACK命令做确认
- slave延迟由slave发送REPLCONF ACK命令做确认

**主从复制完整工作流程**

![1619671737003](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619671737003.png)

1. 首先由slave节点发送replconf ack offset命令给master,然后master判断offset是否在缓冲区中，然后进行命令传播阶段的工作。
2. 如果一次传播完成，那么slave还会接着重新开始发送replconf ack offset指令，这里要和数据同步阶段使用的指令区分开，数据同步阶段使用的是psync2进行数据的同步。
3. 在这个过程中，master也会不停的发送ping命令来判断slave是否还在线。

### 主从复制常见的问题

#### **频繁的全量复制**

频繁的全量复制，会影响性能

伴随着系统的运行，master的数据量会越来越大，一旦master重启，runid将发生变化，会导致全部slave的
全量复制操作

**内部优化调整方案：**

1. master内部创建master_replid变量，使用runid相同的策略生成，长度41位，并发送给所有slave
2. 在master关闭时执行命令 shutdown save，进行RDB持久化,将runid与offset保存到RDB文件中
   1. repl-id，repl-offset 
   2. 通过redis-check-rdb命令可以查看该信息
3. master重启后加载RDB文件，恢复数据，重启后，将RDB文件中保存的repl-id与repl-offset加载到内存中
   1. master_repl_id = repl，master_repl_offset = repl-offset
   2. 通过info命令可以查看该信息
4. 作用：
   本机保存上次runid，重启后恢复该值，使所有slave认为还是之前的master

**导致频繁进行全量复制**

- 问题现象
  - 网络环境不佳，出现网络中断，slave不提供服务
- 问题原因
  - 复制缓冲区过小，断网后slave的offset越界，触发全量复制
- 最终结果
  -  slave反复进行全量复制
- 解决方案
  - 修改复制缓冲区大小

~~~ java
repl-backlog-size
~~~

- 建议设置如下：
  - 测算从master到slave的重连平均时长second
  - 获取master平均每秒产生写命令数据总量write_size_per_second
  - 最优复制缓冲区空间 = 2 * second * write_size_per_second

#### 频繁的网络中断

**问题现象一**

- 问题现象
  - master的CPU占用过高 或 slave频繁断开连接
- 问题原因
  - slave每1秒发送REPLCONF ACK命令到master
  - 当slave接到了慢查询时（keys * ，hgetall等），会大量占用CPU性能
  - master每1秒调用复制定时函数replicationCron()，比对slave发现长时间没有进行响应
- 最终结果
  - master各种资源（输出缓冲区、带宽、连接等）被严重占用
- 解决方案
  通过设置合理的超时时间，确认是否释放slave

~~~ java
repl-timeout
~~~

该参数定义了超时时间的阈值（默认60秒），超过该值，释放slave

**问题现象二**

- 问题现象
  - slave与master连接断开
- 问题原因
  - master发送ping指令频度较低
  - master设定超时时间较短
  - ping指令在网络中存在丢包
- 解决方案
  - 提高ping指令发送的频度

~~~ java
repl-ping-slave-period
~~~

超时时间repl-time的时间至少是ping指令频度的5到10倍，否则slave很容易判定超时

#### 数据不一致

- 问题现象
  - 多个slave获取相同数据不同步

- 问题原因
  - 网络信息不同步，数据发送有延迟
- 解决方案
  - 优化主从间的网络环境，通常放置在同一个机房部署，如使用阿里云等云服务器时要注意此现象
  - 监控主从节点延迟（通过offset）判断，如果slave延迟过大，暂时屏蔽程序对该slave的数据访问

~~~ java
slave-serve-stale-data yes|no
~~~

开启后仅响应info、slaveof等少数命令（慎用，除非对数据一致性要求很高）

### 小结

![1619676905058](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/29/141507-753602.png)

## 哨兵

### 哨兵简介

![1619677202376](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619677202376.png)

- 什么是哨兵？

简单来说，就是redis集群中master宕机的话，需要从slave节点中产生新的master节点，但是如何选择出新的master节点，如何选取master节点可以有效避免节点之间进行全量复制，所以这就需要一个哨兵，及时监控集群，等到master宕机后，及时选出一个合适的master进行工作。

- 如果master宕机后，需要做下面的工作
  - 关闭master和所有slave
  - 找一个slave作为master
  - 修改其他slave的配置，连接新的主
  - 启动新的master与slave
  - 全量复制*N+部分复制*N

- 关闭期间的数据服务谁来承接？
- 找一个主？怎么找法？
- 修改配置后，原始的主恢复了怎么办？

要回答上面三个问题，就需要哨兵。

**哨兵**

哨兵(sentinel) 是一个**分布式系统**，用于对主从结构中的每台服务器进行监控，当出现故障时通过投票机制选择新的master并将所有slave连接到新的master。

如下面这张图片所示，Sentinel是一台主机，里面多个线程监控redis集群，Sentinel有两个功能，第一个是监控功能，就是监控redis集群，第二个功能是选择，也就是master出问题的时候，及时选出新的master节点进行工作。

![1619677454848](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202104/29/142415-710086.png)

**哨兵的作用**

- 监控 
  - 不断的检查master和slave是否正常运行。 
  - master存活检测、master与slave运行情况检测
- 通知（提醒） 
  - 当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知
- 自动故障转移 
  - 断开master与slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址

> 注意： 哨兵也是一台redis服务器，只是不提供数据服务，通常哨兵配置数量为单数

#### 启动哨兵模式

**配置哨兵**

- 配置一拖二的主从结构
- 配置三个哨兵（配置相同，端口不同）

~~~ java
参看sentinel.conf
~~~

- 启动哨兵

~~~ java
redis-sentinel sentinel-端口号.conf
~~~

- 修改配置文件

~~~ java
[rzf@hadoop100 redis]$ cat sentinel.conf | grep -v "#" | grep -v "^$"
port 26379 哨兵也是一个服务，这个时对外服务的端口
dir /tmp  哨兵工作目录，存储信息的位置
sentinel monitor mymaster 127.0.0.1 6379 2 设置哨兵的监控的主节点 2表示如果有2个哨兵认为master挂掉的话，那么就认为master已经挂掉，通常设置为哨兵个数的一半+1
sentinel down-after-milliseconds mymaster 30000 表示主节点连接多长时间没有相应，就认为挂掉 mymaster表示主节点的名称
sentinel parallel-syncs mymaster 1 当主服务器挂掉时候，从同步数据的时候，每一次有多少个从同步数据，这个数字设置的越小，服务器压力越小，如果设置过大，服务器压力性能差
sentinel failover-timeout mymaster 180000 在同步的时候，多长时间同步完成算是有效，如果时间太长同步完成，认为是无效

//把信息写到配置文件中
 cat sentinel.conf | grep -v "#" | grep -v "^$" > ./conf/sentinel-26379.conf
 
 //复制并且重新创建两个配置文件
 [rzf@hadoop100 redis]$ sed 's/26379/26380/g' ./conf/sentinel-26379.conf > ./conf/sentinel-26380.conf 
[rzf@hadoop100 redis]$ sed 's/26379/26381/g' ./conf/sentinel-26379.conf > ./conf/sentinel-26381.conf 

//拷贝服务器配置文件
[rzf@hadoop100 conf]$ sed 's/6380/6381/g' redis-6380.conf > redis-6381.conf
~~~

### 哨兵工作原理

#### 主从切换

哨兵在进行主从切换过程中经历三个阶段

- 监控
- 通知
- 故障转移

#### 阶段一：监控阶段

![1619922176063](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/121058-623317.png)

- 用于同步各个节点的状态信息
  - 获取各个sentinel的状态（是否在线）
  - 获取master的状态
    - master属性
      - runid
      - role：master
    - 各个slave的详细信息
  - 获取所有slave的状态（根据master中的slave信息）
    - slave属性
      - runid
      -  role：slave
      - master_host、master_port
      - offset

**哨兵工作原理**

![1619922316306](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/102518-641332.png)

- sentinel上线以后首先连接master获取信息，会向master发送info信息，返回所有节点的详细信息，为了以后sentinel和master进行信息的交换，这个时候会建立一个cmd链接操作。
- 建立链接以后，同时在sentinel端会记录所有的链接状态，也就是记录所有在线节点的状态。在msater记录了redis实例的信息。但是内容稍微有点不同。
- 接下来sentinel会根据字节节点中存储的slave节点信息，链接每一个slave节点，然后发送info指令获取每一个节点的信息。到此为止，sentinel存储了每一个slave节点中的详细信息。
- 如果有新的sentinel节点上线，那么此时会去链接master然后发送info指令去获取节点信息，并且建立cmd链接通道，如果发现获取的信息中有其他的sentinel节点，那么此时会去链接其他的sentinel节点，保证所有的sentinel节点中信息的一致性。然后sentinel之间会建立cmd通道发送信息。sentinel之间还会相互发送ping命令，查看对方是否还存存在。
- 当有其他的sentinel节点加入进来，也会进行上面的操作，多个sentinel会建立smd通道，进行信息交换，并且还会发送ping命令。

#### 阶段二：通知阶段

通知阶段可以认为是信息的长期维护阶段。

![1619923154895](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/103917-692161.png)

多个哨兵之间通过cmd建立通道后，可以保持信息的互通，然后sentinel会通过和master和slave建立的链接，实时发送命令获取状态信息，master和slave会返回信息，然后获取到信息的哨兵节点会在自己的内部网络中通知其他哨兵同步信息。

#### 阶段三：故障转移阶段

![1619923365461](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/104248-237162.png)

如果哨兵给master发送命令，没有接收到master的返回信息，当多次发送都没有信息返回的时候，那么sentinel就会给master标记为sdown状态，并且会把master挂掉的信息在内网中同步给其他的哨兵，然后其他的哨兵也去向master发送命令查看是否挂掉，如果master没有信息返回，那么每一个哨兵都会把master挂的消息发送到内网之中，同步其他哨兵节点保存的信息。最后所有节点都认为master已经挂掉，那么master出的信息就修改为odown状态。也即是说一个哨兵标记完成为sdown（主观下线），所有哨兵标记完为odown。但是实际上是如果一半哨兵标记完就认为是odown（客观下线）。一旦master被标记为客观下线，那么就开启下一个阶段，清理队伍。

![1619923829691](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619923829691.png)

在多个sentinel之间也会选出一个主的sentinel，选择规则也是半数机制。然后选出的sentinel会在slave中选出一个备选的slave作为master节点。

sentinel选择master的原则如下：

 服务器列表中挑选备选master

- 在线的
- 响应慢的，相应慢的可能是因为网络差。
- 与原master断开时间久的，与master断开时间久可能是因为距离master相对较远。
- 优先原则
  - 优先级
  - offset
  - runid

发送指令（ sentinel ）

- 向新的master发送slaveof no one
- 向其他slave发送slaveof 新masterIP端口，然后slave和新的master建立链接。

致至此，故障转移阶段结束，前面建立的cmd通道，就是用来发送各个指令使用的。

切换master日志展示

![1619928441502](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/120750-544670.png)

**更换master**101

![1619928680890](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619928680890.png)

如果把挂掉的master节点重新启动，那么他就变成为slave节点。

![1619928755102](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619928755102.png)

### 小结

![1619928856809](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/121418-501925.png)

## 集群

### 集群的简介

集群就是使用网络将若干台计算机联通起来，并提供统一的管理方式，使其对外呈现单机的服务效果

![1619929173954](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/121936-849172.png)

**集群的作用**

- 分散单台服务器的访问压力，实现负载均衡
- 分散单台服务器的存储压力，实现可扩展性
- 降低单台服务器宕机带来的业务灾难

![1619929220652](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/122022-985274.png)

### Redis集群结构设计

![1619929410149](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/122332-129451.png)

- 首先会把redis集群进行划分，分为若干个存储区域，比如16384个，然后根据输入的key计算出一个值，这个值用于查找当前的值应该存储在那一台机器上面。

那么如果现在重新加入一台节点怎么办？

redis会把前面的所有节点的内容重新进行优化，从每一台节点中拿出一点数据存储到新加入的节点上面。下面展示的是扩展节点操作

![1619929746746](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/122909-676285.png)

**集群内部通讯设计**

![1619929891500](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/123137-785548.png)

各个集群的节点中都会存储其他节点中保存的数据的位置，如果客户端需要进行查询操作：

![1619930015306](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1619930015306.png)

客户端首先发出key的查询命令，然后经过计算去某台节点中查询，如果命中直接返回，如果没有命中，那么直接返回数据存储的具体位置，然后下一次客户端直接去数据存储的节点中查询数据。所以说最多两次就可以查询到数据。















## 企业级解决方案

### 缓存预热

**宕机**

服务器启动后迅速宕机

**问题排查**

1. 请求数量较高
2. 主从之间数据吞吐量较大，数据同步操作频度较高

**解决方案**

前置准备工作：

1. 日常例行统计数据访问记录，统计访问频度较高的热点数据
2. 利用LRU数据删除策略，构建数据留存队列 例如：storm与kafka配合

准备工作：

1. 将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据
2. 利用分布式多服务器同时进行数据读取，提速数据加载过程
3. 热点数据主从同时预热

实施：

1. 使用脚本程序固定触发数据预热过程
2. 如果条件允许，使用了CDN（内容分发网络），效果会更好

总结

缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

### 缓存雪崩

**数据库服务器崩溃（1）**

1. 系统平稳运行过程中，忽然数据库连接量激增
2. 应用服务器无法及时处理请求
3. 大量408，500错误页面出现
4. 客户反复刷新页面获取数据
5. 数据库崩溃
6. 应用服务器崩溃
7. 重启应用服务器无效
8. Redis服务器崩溃
9. Redis集群崩溃
10. 重启数据库后再次被瞬间流量放倒

**问题排查**

1. 在一个较短的时间内，缓存中较多的key集中过期，而用户又大量访问过期的数据。
2. 此周期内请求访问过期的数据，redis未命中，redis向数据库获取数据
3. 数据库同时接收到大量的请求无法及时处理
4. Redis大量请求被积压，开始出现超时现象
5. 数据库流量激增，数据库崩溃
6. 重启后仍然面对缓存中无数据可用
7. Redis服务器资源被严重占用，Redis服务器崩溃
8. Redis集群呈现崩塌，集群瓦解
9. 应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃
10. 应用服务器，redis，数据库全部重启，效果不理想

**问题分析**

- 短时间范围内

- 大量key集中过期

**解决方案（道）**

- 更多的页面静态化处理
- 构建多级缓存架构
  - Nginx缓存+redis缓存+ehcache缓存
- 检测Mysql严重耗时业务进行优化
  - 对数据库的瓶颈排查：例如超时查询、耗时较高事务等
- 灾难预警机制
  - 监控redis服务器性能指标
  - CPU占用、CPU使用率
  - 内存容量
  - 查询平均响应时间
  - 线程数
- 限流、降级 
  - 短时间范围内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后再逐步放开访问

**解决方案（术）**

- LRU与LFU切换
- 数据有效期策略调整
  - 根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟
  - 过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量
- 超热数据使用永久key
- 定期维护（自动+人工）
  - 对即将过期数据做访问量分析，确认是否延时，配合访问量统计，做热点数据的延时
- 加锁 慎用！

**小结**

缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免过期时间集中，可以有效解决雪崩现象的出现（约40%），配合其他策略一起使用，并监控服务器的运行数据，根据运行记录做快速调整。简单来说就是redis中的数据失效的很多，导致查询服务redis缓存不能及时返回，直接从数据库中获取数据，最终导致数据库的压力过大。

![1619941338144](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/154219-164956.png)

#### 缓存击穿

**数据库服务器崩溃（2）**

- 系统平稳运行过程中
- 数据库连接量瞬间激增
- Redis服务器无大量key过期
- Redis内存平稳，无波动
- Redis服务器CPU正常
- 数据库崩溃

**问题排查**

1. Redis中某个key过期，该key访问量巨大
2. 多个数据请求从服务器直接压到Redis后，均未命中
3. Redis在短时间内发起了大量对数据库中同一数据的访问

**问题分析**

- 单个key高热数据
- key过期

**解决方案（术）**

1. 预先设定 
   1. 以电商为例，每个商家根据店铺等级，指定若干款主打商品，在购物节期间，加大此类信息key的过期时长 注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势
2. 现场调整 
   1. 监控访问量，对自然流量激增的数据延长过期时间或设置为永久性key
3. 后台刷新数据 
   1. 启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失
4. 二级缓存 
   1. 设置不同的失效时间，保障不会被同时淘汰就行
5. 加锁 
   1. 分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重！

**总结**

缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量对同一数据的数据库访问，导致对数据库服务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监控测试与即时调整策略，毕竟单个key的过期监控难度较高，配合雪崩处理策略即可。

#### 缓存穿透

**数据库服务器崩溃（3）**

1. 系统平稳运行过程中
2. 应用服务器流量随时间增量较大
3. Redis服务器命中率随时间逐步降低
4. Redis内存平稳，内存无压力
5. Redis服务器CPU占用激增
6. 数据库服务器压力激增
7. 数据库崩溃

**问题排查**

- Redis中大面积出现未命中
- 出现非正常URL访问
- 也就是说访问请求是一些不正常的或者不存在的东西。大量访问不存在的内容，redis不会做缓存。

**问题分析**

- 获取的数据在数据库中也不存在，数据库查询未得到对应数据
- Redis获取到null数据未进行持久化，直接返回
- 下次此类数据到达重复上述过程
- 出现黑客攻击服务器

**解决方案（术）**

- 缓存null 
  - 对查询结果为null的数据进行缓存（长期使用，定期清理），设定短时限，例如30-60秒，最高5分钟

- 白名单策略
  - 提前预热各种分类数据id对应的bitmaps，id作为bitmaps的offset，相当于设置了数据白名单。当加载正常数据时，放行，加载异常数据时直接拦截（效率偏低）
  - 使用布隆过滤器（有关布隆过滤器的命中问题对当前状况可以忽略）
- 实施监控
  - 实时监控redis命中率（业务正常范围时，通常会有一个波动值）与null数据的占比
    - 非活动时段波动：通常检测3-5倍，超过5倍纳入重点排查对象
    - 活动时段波动：通常检测10-50倍，超过50倍纳入重点排查对象
  - 根据倍数不同，启动不同的排查流程。然后使用黑名单进行防控（运营）
- key加密
  - 问题出现后，临时启动防灾业务key，对key进行业务层传输加密服务，设定校验程序，过来的key校验 例
  - 如每天随机分配60个加密串，挑选2到3个，混淆到页面数据id中，发现访问key不满足规则，驳回数据访问

**小结**

- 缓存击穿访问了不存在的数据（不是说redis中不存在，而是整个数据库中不存在），跳过了合法数据的redis数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类数据的出现量是一个较低的值，当出现此类情况以毒攻毒，并及时报警。应对策略应该在临时预案防范方面多做文章。
- 无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。

### 性能指标监控

监控指标

- 性能指标：Performance
- 内存指标：Memory
- 基本活动指标：Basic activity
- 持久性指标：Persistence
- 错误指标：Error

**性能指标：Performance**

![1619947577112](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/172618-352263.png)

**内存指标：Memory**

![1619947606923](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/172648-715741.png)

**基本活动指标：Basic activity**

![1619947634908](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/172716-943198.png)

**持久性指标：Persistence**

![1619947730365](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/172851-283228.png)

**错误指标：Error**

![1619947757343](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/172917-555984.png)

**监控方式**

工具

- Cloud Insight Redis
- Prometheus
- Redis-stat
- Redis-faina
- RedisLive
- zabbix

**命令**

- benchmark
- redis cli
  - monitor
  - showlog

**benchmark**

- 命令

~~~ java
redis-benchmark [-h ] [-p ] [-c ] [-n <requests]> [-k ]
~~~

- 案例

~~~ java
redis-benchmark
//说明：50个连接，10000次请求对应的性能
redis-benchmark -c 100 -n 5000
//说明：100个连接，5000次请求对应的性能
~~~

![1619948571776](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/174253-123397.png)

**monitor**

- 命令

~~~ java
monitor
//打印服务器调试信息
~~~

**showlong**

- 命令

~~~ java
showlong [operator]
~~~

get ：获取慢查询日志
len ：获取慢查询日志条目数
reset ：重置慢查询日志

- 相关配置

~~~ java
slowlog-log-slower-than 1000 #设置慢查询的时间下线，单位：微妙 
slowlog-max-len 100 #设置慢查询命令对应的日志显示长度，单位：命令数
~~~

### 小结

![1619948822321](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202105/02/174704-15947.png)