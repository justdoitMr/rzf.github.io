## Rides底层数据结构

> String：最大存储512M
>
> Hash：每个 hash 可以存储**2^32 - 1个键值对**，大概40亿

### 每一种数据结构的实现方式

![1640934142912](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/150224-988796.png)

### 底层实现

Redis底层的所有数据结构：都是基于对象的；RedisObject

- 类型；
- 编码；
- 指向实际数据的指针；

~~~c++
typedef struct redisObject{
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 指向底层实现数据结构的指针
    void *ptr;
}
~~~

- type：记录是什么类型（String，List，Hah表，set，Zset）
- encoding：对象使用的编码；
- *ptr：数据指针（比如String，ptr就指向了一个SDS动态字符串）

一个String类型数据如下图：

![1640931070644](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141112-109515.png)

> Object 系统包含五种Object：
>
> - String：字符串对象
> - List：列表对象
> - Hash：哈希对象
> - Set：集合对象
> - ZSet：有序集合

Redis使用对象来表示数据库中的键和值，即每新建一个键值对，至少创建有两个对象，而且使用对象的具有以下好处：

1. redis可以在执行命令前会根据对象的类型判断一个对象是否可以执行给定的命令
2. 针对不同的使用场景，为对象设置不同的数据结构实现，从而优化对象的不同场景下的使用效率
3. 对象系统还可以基于引用计数计数的内存回收机制，自动释放对象所占用的内存，或者还可以让多个数据库键共享同一个对象来节约内存。
4. redis对象带有访问时间记录信息，使用该信息可以进行优化空转时长较大的key，进行删除！

对象的ptr指针指向对象的底层现实数据结构，而这些数据结构由对象的encoding属性决定，对应关系：

![1640931302753](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141515-641256.png)

每种Object对象至少有两种不同的编码，对应关系：

![1640931338152](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141540-810965.png)

## Rides数据结构

我们都知道，在 redis 中一共有5种数据结构，那每种数据结构的使用场景都是什么呢？

![1640927052735](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/130413-582839.png)

- String——字符串
-  Hash——字典
-  List——列表
-  Set——集合
-  Sorted Set——有序集合

### String(字符串)

#### String介绍

String 数据结构是简单的 key-value 类型，value 不仅可以是 String，也可以是数字（当数字类型用 Long 
可以表示的时候encoding 就是整型，其他都存储在 sdshdr 当做字符串）。

**简单来说有这三种实现方式**

![1640934466138](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/150747-696448.png)

**结构图上显示，String类型有三种实现方式**：

> - 使用整数值实现的字符串对象
> - 使用 embstr 编码的动态字符串实现的字符串对象
> - 动态字符串实现的字符串对象
>
> 使用 String类型，可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受 Redis 的定时持久化（可以选择 RDB 模式或者 AOF 模式），操作日志及 Replication 等功能。

#### 原理

embstr ：是专门用于保存短字符串的一种优化编码方式，跟正常的字符编码相比，字符编码会调用两次内存分配函数来分别创建 redisObject 和 sdshdr 结构（动态字符串结构），而  embstr 编码则通过调用一次内存分配函数来分配一块连续的内存空间，空间中包含 redisObject 和  sdshdr（动态字符串）两个结构，两者在同一个内存块中。从 Redis 3.0 版本开始，字符串引入了 embstr 编码方式，长度小于  OBJ_ENCODING_EMBSTR_SIZE_LIMIT(39) 的字符串将以EMBSTR方式存储。

> **注意**： 在Redis 3.2 之后，就不是以 39 为分界线，而是以 44 为分界线，主要与 Redis  中内存分配使用的是 jemalloc 有关。（ jemalloc 分配内存的时候是按照 8、16、32、64 作为 chunk  的单位进行分配的。为了保证采用这种编码方式的字符串能被 jemalloc 分配在同一个 chunk  中，该字符串长度不能超过64，故字符串长度限制：

![1640934793223](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/151407-159176.png)

> 动态字符串 ：Redis 自己构建的一种名为 简单动态字符串（simple dynamic string，SDS）的抽象类型，并将 SDS 作为 Redis 的默认字符串表示。先简单了解概念，后面看详细解析

字符串的编码可以是 int，raw 或者 embstr。

如果一个字符串内容可转为 long，那么该字符串会被转化为 long 类型，对象  ptr 指向该 long，并且对象类型也用 int 类型表示。普通的字符串有两种 embstr 和 raw。如果字符串对象的长度小于 39  字节，就用 embstr，否则用 raw。

> 也就是说，Redis 会根据当前值的类型和长度决定使用内部编码实现：恍然大悟

```text
int：8个字节的长整型
embstr：小于等于39个字节的字符串
raw：大于39个字节的字符串
```

#### 动态字符串

Redis的字符串底层使用SDS（动态字符串），而不是C语言的字符串；

~~~java
// SDS的定义
strcut sdshdr{
    // 记录字符串长度
    int len;
    // 记录未使用的buf；
    int free;
    // 字符串数组，存放实际的字符串
    char buf[];
}
~~~

用 SDS 保存字符串 “Redis” 具体结构如下图：

![1640931440096](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141722-768554.png)

![1640935581146](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/152623-346700.png)

那么为什么不使用c语言的字符串呢？

![1640931483300](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141804-519433.png)

![1640935633816](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/152715-278513.png)

字符串对象编码可以int 、raw或者embstr，如果保存的值为整数值且这个值可以用long类型表示，使用int编码，其他编码类似。

##### 获取字符串长度复杂度

sdshdr 中由于 len 属性的存在，获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)，而对于 C 语言来说，获取字符串的长度通常是遍历字符串计数来实现的，时间复杂度为 O(n)。

##### API安全性与缓冲区溢出

缓冲区溢出（buffer overflow）：是这样的一种异常，当程序将数据写入缓冲区时，会超过缓冲区的边界，并覆盖相邻的内存位置。在 C 语言中使用 strcat 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出，如

![1640935770022](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640935770022.png)

![1640935788768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/152949-131673.png)

##### 字符串的内存重分配次数

C 语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存。
SDS 实现了空间预分配和惰性释放两种策略：

1. 空间预分配：当 SDS 的 API 对一个 SDS 进行修改，并且需要对 SDS 进行空间扩展的时候，程序不仅会为 SDS 分配修改所必须的空间，还会为 SDS 分配额外的未使用空间，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
2. 惰性释放：当 SDS 的 API 需要对 SDS 保存的字符串进行缩短时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用 free 属性将这些字节的数量记录起来，并等待将来使用，如

![1640935872457](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/153115-264354.png)

~~~java
sdstrim(s, "XY"); // 移除 SDS 字符串中的所有 'X' 和 'Y' 
~~~

![1640935894962](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/153136-118432.png)

##### 二进制数据安全

二进制安全（binary-safe）：指能处理任意的二进制数据，包括非 ASCII 和 null 字节。
C 字符串以空字符 '\0'，作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串'\0'，导致程序读入的空字符会被误认为是字符串的结尾，因此C字符串无法正确存取二进制数据；

SDS 的 API 都是以处理二进制的方式来处理 buf 里面的元素，并且 SDS 不是以空字符串'\0'来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束，

> 因此 Redis 不仅可以保存文本数据，还可以保存任意格式的二进制数据。

##### C字符串函数兼容

 SDS 的buf数组会以'\0'结尾，这样可以重用 C 语言库<string.h> 中的一部分函数，避免了不必要的代码重复。

> - String 类型对象三种实现方式，int，embstr，raw
> - 字符串内容可转为 long，采用 int 类型，否则长度<39（3.2版本前39,3.2版本后分界线44） 用 embstr，其他用 raw
> - SDS 是Redis自己构建的一种简单动态字符串的抽象类型，并将 SDS 作为 Redis 的默认字符串表示
> - SDS 与 C 语言字符串结构相比，具有五大优势

比如：int编码的String Object：

~~~java
redis> set number 520 
 ok
 redis> OBJECT ENCODING number 
"int"
~~~

![1640931567523](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/141929-303721.png)

**String 对象之间的编码转换**

int编码的字符串对象和embstr编码的字符串对象在条件满足的情况下，会被转换为raw编码的字符串对象。   

比如：对int编码的字符串对象进行append命令时，就会使得原来是int变为raw编码字符串

> Redis的字符串是动态字符串，是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配，如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

#### String类型的操作

除了提供与 Memcached 一样的 get、set、incr、decr 等操作外，Redis 还提供了下面一些操作：

~~~java
1.LEN niushuai：O(1)获取字符串长度

2.APPEND niushuai redis：往字符串 append 内容，而且采用智能分配内存（每次2倍）

3.设置和获取字符串的某一段内容

4.设置及获取字符串的某一位（bit）

5.批量设置一系列字符串的内容

6.原子计数器

7.GETSET 命令的妙用，请于清空旧值的同时设置一个新值，配合原子计数器使用
~~~

初始化字符串 需要提供「变量名称」和「变量的内容」

~~~java
set ireader beijing.zhangyue.keji.gufen.youxian.gongsi
~~~

获取字符串的内容 提供「变量名称」

```
`> get ireader``"beijing.zhangyue.keji.gufen.youxian.gongsi"`
```

获取字符串的长度 提供「变量名称」

```
`> strlen ireader``(``integer``) 42`
```

获取子串 提供「变量名称」以及开始和结束位置[start, end]

```
`> getrange ireader 28 34``"youxian"`
```

覆盖子串 提供「变量名称」以及开始位置和目标子串

```
`> setrange ireader 28 wooxian``(``integer``) 42 # 返回长度``> get ireader``"beijing.zhangyue.keji.gufen.wooxian.gongsi"`
```

追加子串

```
`> append ireader .hao``(``integer``) 46 # 返回长度``> get ireader``"beijing.zhangyue.keji.gufen.wooxian.gongsi.hao"`
```

遗憾的是字符串没有提供字串插入方法和子串删除方法。

计数器 如果字符串的内容是一个整数，那么还可以将字符串当成计数器来使用。

```
`> ``set` `ireader 42``OK``> get ireader``"42"``> incrby ireader 100``(``integer``) 142``> get ireader``"142"``> decrby ireader 100``(``integer``) 42``> get ireader``"42"``> incr ireader # 等价于incrby ireader 1``(``integer``) 43``> decr ireader # 等价于decrby ireader 1``(``integer``) 42`
```

计数器是有范围的，它不能超过Long.Max，不能低于Long.MIN

```
`> ``set` `ireader 9223372036854775807``OK``> incr ireader``(error) ERR increment ``or` `decrement would overflow``> ``set` `ireader -9223372036854775808``OK``> decr ireader``(error) ERR increment ``or` `decrement would overflow`
```

过期和删除 字符串可以使用del指令进行主动删除，可以使用expire指令设置过期时间，到点会自动删除，这属于被动删除。可以使用ttl指令获取字符串的寿命。

```
`> expire ireader 60``(``integer``) 1 # 1表示设置成功，0表示变量ireader不存在``> ttl ireader``(``integer``) 50 # 还有50秒的寿命，返回-2表示变量不存在，-1表示没有设置过期时间``> del ireader``(``integer``) 1 # 删除成功返回1``> get ireader``(nil) # 变量ireader没有了`
```

#### 使用场景

##### **分布式id解决方案**

我们说value类型也可以表示int类型，所以可以作为分布式id的一种解决方案。

所以我们可以每一次让value指定加1，也可以指定加一个固定值。

~~~java
incr key //增加1
incrby key increment //增加一个指定的值
incrbyfloat key increment //加浮点型
~~~

##### **自动销毁热门的数据**

redis 控制数据的生命周期，通过数据是否失效控制业务行为，适用于所有具有时效性限定控制的操作

redis应用于各种结构型和非结构型高热度数据访问加速

#### 注意事项

注意：按数值进行操作的数据，如果原始数据不能转成数值，或超越了redis 数值上限范围，将报错。 9223372036854775807（java中long型数据最大值，Long.MAX_VALUE）

**数据最大存储量**

- 512MB

### Hash(字典)

1、ziplist 编码的哈希对象使用**压缩列表**作为底层实现
2、hashtable 编码的哈希对象使用**字典**作为底层实现

> 字典是一种什么样的结构？
>
> **字典**， 又称符号表（symbol table）、关联数组（associative  array）或者映射（map）， 是一种用于保存键值对（key-value pair）的抽象数据结构。在字典中，  一个键（key）可以和一个值（value）进行关联（或者说将键映射为值）， 这些关联的键和值就被称为键值对。
>
> 字典中的每个键都是独一无二的， 程序可以在字典中根据键查找与之关联的值， 或者通过键来更新值， 又或者根据键来删除整个键值对， 等等。

#### Hash介绍

在 Memcached 中，我们经常将一些结构化的信息打包成 hashmap，在客户端序列化后存储为一个字符串的值（一般是 JSON 格式），比如用户的昵称、年龄、性别、积分等。

这时候在需要修改其中某一项时，通常需要将字符串（JSON）取出来，然后进行反序列化，修改某一项的值，再序列化成字符串（JSON）存储回去。简单修改一个属性就干这么多事情，消耗必定是很大的，也不适用于一些可能并发操作的场合（比如两个并发的操作都需要修改积分）。

而Redis 的 Hash 结构可以使你像在数据库中 Update 一个属性一样只修改某一项属性值。

![1640927547523](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/131227-764591.png)

哈希等价于Java语言的HashMap或者是Python语言的dict，在实现结构上它使用二维结构，**第一维是数组，第二维是链表**，**hash的内容key和value存放在链表中，数组里存放的是链表的头指针**。通过key查找元素时，先计算key的hashcode，然后用hashcode对数组的长度进行取模定位到链表的表头，再对链表进行遍历获取到相应的value值，链表的作用就是用来将产生了「hash碰撞」的元素串起来。Java语言开发者会感到非常熟悉，因为这样的结构和HashMap是没有区别的。**哈希的第一维数组的长度也是2^n**。

![1640927613110](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/131334-602897.png)

哈希在很多编程语言中都有着很广泛的应用，而在Redis中也是如此，在redis中，哈希类型是指Redis键值对中的值本身又是一个键值对结构，形如`value=[{field1，value1}，...{fieldN，valueN}]`
，其与Redis字符串对象的区别如下图所示:

![1640946414980](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/182656-473584.png)

哈希类型的内部编码有两种：ziplist(压缩列表),hashtable(哈希表)。只有当存储的数据量比较小的情况下，Redis 才使用压缩列表来实现字典类型。具体需要满足两个条件：

- 当哈希类型元素个数小于hash-max-ziplist-entries配置（默认512个）
- 所有值都小于hash-max-ziplist-value配置（默认64字节）
   `ziplist`使用更加紧凑的结构实现多个元素的连续存储，所以在节省内存方面比`hashtable`更加优秀。当哈希类型无法满足`ziplist`的条件时，Redis会使用`hashtable`作为哈希的内部实现，因为此时`ziplist`的读写效率会下降，而`hashtable`的读写时间复杂度为O（1）。
   有关ziplist和hashtable这两种redis底层数据结构的具体实现可以参考我的另外两篇文章。

Hash对象的编码可以是ziplist或者hashtable
其中，ziplist底层使用**压缩列表**实现：

- 保存同一键值对的两个节点**紧靠相邻**，键key在前，值vaule在后
- 先保存的键值对在压缩列表的表头方向，后来在表尾方向

hashtable底层使用字典实现，Hash对象种的每个键值对都使用一个字典键值对保存：

- 字典的键为字符串对象，保存键key
- 字典的值也为字符串对象，保存键值对的值

**比如：HSET命令**

~~~java
redis>HSET author name  "Ccww"
(integer)

redis>HSET author age  18
(integer)

redis>HSET author sex  "male"
(integer)
~~~

#### **ziplist的底层结构：**

![1640933854942](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145748-9291.png)

#### **hashtable的底层结构：**

![1640933866977](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/164212-759418.png)

RedisObject中的ptr指针，指向的就是哈希对象：

~~~java
typedef struct dict{
    // 指向对哈希表操作的函数
    dictType *type;
    // 私有数据
    void *privdata;
    // ht[1]指向真正的哈希表结构,ht[2]用于备用扩容，指向正在扩容的哈希表
    dictht ht[2];
    // 是否在rehash：如果不在rehash，此值为-1；
    int trehashidx;
}
~~~

#### **哈希表**

~~~java
typedef struct dictht{
    // 哈希数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 记录尾部：size-1
    unsigned long sizemask;
    // 已使用的大小
    unsigned long used;
}
~~~

- table 属性是一个数组， 数组中的每个元素都是一个指向 dict.h/dictEntry 结构的指针， 每个 dictEntry 结构保存着一个键值对。
- size 属性记录了哈希表的大小， 也即是 table 数组的大小， 而 used 属性则记录了哈希表目前已有节点（键值对）的数量。

**结构图，一个空的哈希表**

![1640947635634](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/184721-47947.png)

#### **哈希表节点**

一个哈希表里面可以有多个哈希表节点，那么每个哈希表节点的结构以及多个哈希表节点之间的存储关系是怎么样的呢？

**哈希表节点结构定义** （dictEntry）：

~~~java
typedef struct dictEntry{
    // key
    void *key;
    // value
    union{
        void *val;
        unit64_tu64;
        int64_ts64;
    } v;
    // next指针
    struct dictEntry *next;
} dictEntry;
~~~

- key 属性保存着键值对中的键， 而 v 属性则保存着键值对中的值， 其中键值对的值可以是一个指针， 
  或者是一个 uint64_t 整数， 又或者是一个 int64_t 整数。

- next 属性是指向另一个哈希表节点的指针， 这个指针可以将多个哈希值相同的键值对连接在一次， 

以此来解决键冲突（collision）的问题。

结构图解：多个哈希值相同的键值对存储结构，解决键冲突

![1640947747311](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/184908-278525.png)

#### 字典结构实现

**字典结构定义** （dict.h/dict）：

~~~java
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */

} dict;
~~~

**描述**：type 属性和 privdata 属性是针对不同类型的键值对， 为创建多态字典而设置的

- type 属性是一个指向 dictType 结构的指针， 每个 dictType 结构保存了一簇用于操作特定类型键值对的函数，
- Redis 会为用途不同的字典设置不同的类型特定函数。
- privdata 属性则保存了需要传给那些类型特定函数的可选参数。

ht属性是一个包含两个项的数组， 数组中的每个项都是一个 dictht 哈希表， 一般情况下， 字典只使用 ht[0] 哈希表， ht[1] 哈希表只会在对 ht[0] 哈希表进行 rehash 时使用。

除了 ht[1] 之外， 另一个和 rehash 有关的属性就是 rehashidx ： 它记录了 rehash 目前的进度， 如果目前没有在进行 rehash ， 那么它的值为 -1 。

结构图解：普通状态下（没有进行 rehash）的字典

![1640947961839](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/185243-548504.png)

#### Hash算法

当要将一个新的键值对添加到字典里面时， 程序需要先根据键值对的键计算出哈希值和索引值， 然后再根据索引值， 将包含新键值对的哈希表节点放到哈希表数组的指定索引上面。

Redis 计算哈希值和索引值的方法如下

~~~java
# 使用字典设置的哈希函数，计算键 key 的哈希值
hash = dict->type->hashFunction(key);

# 使用哈希表的 sizemask 属性和哈希值，计算出索引值
# 根据情况不同， ht[x] 可以是 ht[0] 或者 ht[1]
index = hash & dict->ht[x].sizemask;
~~~

sizemark记录尾部元素值：![1640948267079](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/185748-549121.png)

#### 键冲突解决

当有两个或以上数量的键被分配到了哈希表数组的同一个索引上面时， 我们称这些键发生了冲突（collision）。

Redis 的哈希表使用链地址法（separate chaining）来解决键冲突： 每个哈希表节点都有一个 next 指针，  多个哈希表节点可以用 next 指针构成一个单向链表， 被分配到同一个索引上的多个节点可以用这个单向链表连接起来， 这就解决了键冲突的问题。

> > 1.字典 ht 属性是包含两个哈希表项的数组，一般情况下， 字典只使用 ht[0]， ht[1] 哈希表只会在对 ht[0] 哈希表进行 rehash (下节分析) 时使用
>
> > 2.哈希表使用链地址法（separate chaining）来解决键冲突
>
> > 3.键值对添加到字典的过程， 先根据键值对的键计算出哈希值和索引值， 然后再根据索引值， 将包含新键值对的哈希表节点放到哈希表数组的指定索引上面

#### **Hash扩容**

![1640933180759](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/144622-611325.png)

**对比：java中的hashmap，当数据数量达到阈值的时候(0.75)，就会发生rehash，hash表长度变为原来的二倍，将原hash表数据全部重新计算hash地址，重新分配位置，达到rehash目的**

**redis中的hash表采用的是渐进式hash的方式：**

1. **redis字典（hash表）底层有两个数组，还有一个rehashidx用来控制rehash**

![1640933559745](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145242-374524.png)

![1640933576056](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145258-773908.png)

2. **初始默认hash长度为4，当元素个数与hash表长度一致时，就发生扩容，hash长度变为原来的二倍**

![1640933609516](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145348-874434.png)

3. **redis中的hash则是执行的单步rehash的过程：**

**每次的增删改查，rehashidx+1，然后执行对应原hash表rehashidx索引位置的rehash**

总结：

在扩容和收缩的时候，如果哈希字典中有很多元素，一次性将这些键全部rehash到ht[1]的话，可能会导致服务器在一段时间内停止服务。所以，采用渐进式rehash的方式，详细步骤如下：

1. 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表
2. 将rehashindex的值设置为0，表示rehash工作正式开始
3. 在rehash期间，每次对字典执行增删改查操作是，程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在rehashindex索引上的所有键值对rehash到ht[1]，当rehash工作完成以后，rehashindex的值+1
4. 随着字典操作的不断执行，最终会在某一时间段上ht[0]的所有键值对都会被rehash到ht[1]，这时将rehashindex的值设置为-1，表示rehash操作结束

渐进式rehash采用的是一种分而治之的方式，将rehash的操作分摊在每一个的访问中，避免集中式rehash而带来的庞大计算量。

> 需要注意的是在渐进式rehash的过程，如果有增删改查操作时，如果index大于rehashindex，访问ht[0]，否则访问ht[1].

#### Hash操作

**增加元素** 可以使用hset一次增加一个键值对，也可以使用hmset一次增加多个键值对。

~~~java
> hset ireader go fast
(integer) 1
> hmset ireader java fast python slow
OK
~~~

**获取元素** 可以通过hget定位具体key对应的value，可以通过hmget获取多个key对应的value，可以使用hgetall获取所有的键值对，可以使用hkeys和hvals分别获取所有的key列表和value列表。这些操作和Java语言的Map接口是类似的。

~~~ java
> hmset ireader go fast java fast python slow
OK
> hget ireader go
"fast"
> hmget ireader go python
1) "fast"
2) "slow"
> hgetall ireader
1) "go"
2) "fast"
3) "java"
4) "fast"
5) "python"
6) "slow"
> hkeys ireader
1) "go"
2) "java"
3) "python"
> hvals ireader
1) "fast"
2) "fast"
3) "slow"
~~~

**删除元素** 可以使用hdel删除指定key，hdel支持同时删除多个key

~~~java
> hmset ireader go fast java fast python slow
OK
> hdel ireader go
(integer) 1
> hdel ireader java python
(integer) 2
~~~

**判断元素是否存在** 通常我们使用hget获得key对应的value是否为空就直到对应的元素是否存在了，不过如果value的字符串长度特别大，通过这种方式来判断元素存在与否就略显浪费，这时可以使用hexists指令。

~~~ java
> hmset ireader go fast java fast python slow
OK
> hexists ireader go
(integer) 1
~~~

**计数器** hash结构还可以当成计数器来使用，对于内部的每一个key都可以作为独立的计数器。如果value值不是整数，调用hincrby指令会出错。

~~~java
> hincrby ireader go 1
(integer) 1
> hincrby ireader python 4
(integer) 4
> hincrby ireader java 4
(integer) 4
> hgetall ireader
1) "go"
2) "1"
3) "python"
4) "4"
5) "java"
6) "4"
> hset ireader rust good
(integer) 1
> hincrby ireader rust 1
(error) ERR hash value is not an integer
~~~

**扩容** 当hash内部的元素比较拥挤时(hash碰撞比较频繁)，就需要进行扩容。扩容需要申请新的两倍大小的数组，然后将所有的键值对重新分配到新的数组下标对应的链表中(rehash)。如果hash结构很大，比如有上百万个键值对，那么一次完整rehash的过程就会耗时很长。这对于单线程的Redis里来说有点压力山大。所以Redis采用了渐进式rehash的方案。它会同时保留两个新旧hash结构，在后续的定时任务以及hash结构的读写指令中将旧结构的元素逐渐迁移到新的结构中。这样就可以避免因扩容导致的线程卡顿现象。

缩容 Redis的hash结构不但有扩容还有缩容，从这一点出发，它要比Java的HashMap要厉害一些，Java的HashMap只有扩容。缩容的原理和扩容是一致的，只不过新的数组大小要比旧数组小一倍。

#### Hash对象的编码转换

当list对象可以同时满足以下两个条件时，list对象使用的是ziplist编码：

1. list对象保存的所有字符串元素的长度都小于64字节
2. list对象保存的元素数量小于512个，

不能满足这两个条件的hash对象需要使用hashtable编码

> **Note**：这两个条件的上限值是可以修改的，可查看配置文件hash-max-zaiplist-value和hash-max-ziplist-entries

#### 使用场景

##### 电商网站购物车设计与实现：

- 用户id作为键，商品的id作为key，商品的件数作为value.

##### 促销活动

- 将商家的id作为键

- 将商品的id作为key

- 将优惠商品的件数作为value

### List(列表)

#### List介绍

List 说白了就是链表（redis 使用**双端链表**实现的 List），相信学过数据结构知识的人都应该能理解其结构。使用 List 结构，我们可以轻松地实现最新消息排行等功能（比如新浪微博的 TimeLine ）。List 的另一个应用就是消息队列，可以利用 List 的 PUSH 操作，将任务存在 List 中，然后工作线程再用 POP 操作将任务取出进行执行。

Redis 还提供了操作 List 中某一段元素的API，你可以直接查询，删除 List 中某一段的元素。

![1640928014515](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/132015-561416.png)

Redis将列表数据结构命名为list而不是array，**是因为列表的存储结构用的是链表而不是数组**，而且链表还是双向链表。因为它是链表，所以随机定位性能较弱，首尾插入删除性能较优。如果list的列表长度很长，使用时我们一定要关注链表相关操作的时间复杂度。

负下标 链表元素的位置使用自然数0,1,2,…n-1表示，还可以使用负数-1,-2,…-n来表示，-1表示「倒数第一」，-2表示「倒数第二」，那么-n就表示第一个元素，对应的下标为0。

队列／堆栈 链表可以从表头和表尾追加和移除元素，结合使用rpush/rpop/lpush/lpop四条指令，可以将链表作为队列或堆栈使用，左向右向进行都可以

list对象实现方式可以为ziplist或者为linkedlist，对应底层实现ziplist为压缩列表，linkedlist为双向列表。

~~~java
Redis>RPUSH numbers “Ccww” 520 1
~~~

#### **用ziplist编码的List对象结构：**

![1640934071126](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/150112-890347.png)

#### **用linkedlist编码的List对象结构：  **

![1640934091736](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/150132-351278.png)

#### 压缩列表与双端链表是什么样的结构？

##### 压缩列表（ziplist）

压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列**特殊编码的连续内存块**组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

**结构**

![1640945844115](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/181726-301102.png)

压缩列表的每个节点构成如下

![1640945895504](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/181817-831644.png)

1）**previous_entry_ength**：以字节为单位，记录了压缩列表中前一个字节的长度。previous_entry_ength 的长度可以是1字节或者5字节：
如果前一节点的长度小于254字节，那么 previous_entry_ength 属性的长度为1字节，前一节点的长度就保存在这一个字节里面。
如果前一个节点的长度大于等于254，那么 previous_entry_ength 属性的长度为5字节，其中属性的第一字节会被设置为0xFE（十进制254），而之后的四个字节则用于保存前一节点的长度。
利用此原理即当前节点位置减去上一个节点的长度即得到上一个节点的起始位置，压缩列表可以从尾部向头部遍历，这么做很有效地减少了内存的浪费。
 2）**encoding**：记录了节点的 content 属性所保存数据的类型以及长度。
 3）**content**：保存节点的值，节点的值可以是一个字节数组或者整数，值的类型和长度由节点的 encoding 属性决定。

##### 双端链表（linkedlist）

链表是一种常用的数据结构，C 语言内部是没有内置这种数据结构的实现，所以Redis自己构建了链表的实现。
 链表节点定义：

~~~c++、
typedef  struct listNode{
       //前置节点
       struct listNode *prev;
       //后置节点
       struct listNode *next;
       //节点的值
       void *value;  
}listNode
~~~

![1640946020185](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/182039-942175.png)

另外Redis还提供了操作链表的数据结构：

~~~java
typedef struct list{
     //表头节点
     listNode *head;
     //表尾节点
     listNode *tail;
     //链表所包含的节点数量
     unsigned long len;
     //节点值复制函数
     void (*free) (void *ptr);
     //节点值释放函数
     void (*free) (void *ptr);
     //节点值对比函数
     int (*match) (void *ptr,void *key);
}list;
~~~

list结构为链表提供了表头指针 head ，表尾指针 tail 以及链表长度计数器 len ，dup、free、match 成员则是用于实现多态链表所需的类型特定函数。

**Redis链表实现的特性**：

> - 双端：链表节点带有 prev 和 next 指针，获取某个节点的前置节点和后置节点复杂度都是O(1)。
> - 无环：表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL，对链表的访问以NULL为终点。
> - 带表头指针和表尾指针：通过list结构的 head 和 tail 指针，程序获取链表的表头节点和表尾结点的复杂度都是O(1)。
> - 带链表长度计数器：程序使用 list 结构的 len属性对 list持有的链表节点进行计数，程序获取链表中节点数量的复杂度为O(1)。
> - 多态：链表节点使用 void* 指针来保存节点值，并且通过 list 结构的 dup、 free、match 三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。

#### List对象的编码转换：

当list对象可以同时满足以下两个条件时，list对象使用的是ziplist编码：

1. list对象保存的所有字符串元素的长度都小于64字节
2. list对象保存的元素数量小于512个，

不能满足这两个条件的list对象需要使用linkedlist编码。

#### list操作

~~~java
# 右进左出
> rpush ireader go
(integer) 1
> rpush ireader java python
(integer) 3
> lpop ireader
"go"
> lpop ireader
"java"
> lpop ireader
"python"
# 左进右出
> lpush ireader go java python
(integer) 3
> rpop ireader
"go"
...
# 右进右出
> rpush ireader go java python
(integer) 3
> rpop ireader 
"python"
...
# 左进左出
> lpush ireader go java python
(integer) 3
> lpop ireader
"python"
...
~~~

在日常应用中，列表常用来作为异步队列来使用。

长度 使用llen指令获取链表长度:

~~~java
> rpush ireader go java python
(integer) 3
> llen ireader
(integer) 3
~~~

**随机读** 可以使用lindex指令访问指定位置的元素，使用lrange指令来获取链表子元素列表，提供start和end下标参数

~~~java
> rpush ireader go java python
(integer) 3
> lindex ireader 1
"java"
> lrange ireader 0 2
1) "go"
2) "java"
3) "python"
> lrange ireader 0 -1 # -1表示倒数第一
1) "go"
2) "java"
3) "python"
~~~

使用lrange获取全部元素时，需要提供end_index，如果没有负下标，就需要首先通过llen指令获取长度，才可以得出end_index的值，有了负下标，使用-1代替end_index就可以达到相同的效果。

**修改元素** 使用lset指令在指定位置修改元素。

~~~java
> rpush ireader go java python
(integer) 3
> lset ireader 1 javascript
OK
> lrange ireader 0 -1
1) "go"
2) "javascript"
3) "python"

~~~

**插入元素** 使用linsert指令在列表的中间位置插入元素，有经验的程序员都知道在插入元素时，我们经常搞不清楚是在指定位置的前面插入还是后面插入，所以antirez在linsert指令里增加了方向参数before/after来显示指示前置和后置插入。不过让人意想不到的是linsert指令并不是通过指定位置来插入，而是通过指定具体的值。这是因为在分布式环境下，列表的元素总是频繁变动的，意味着上一时刻计算的元素下标在下一时刻可能就不是你所期望的下标了。

~~~java
> rpush ireader go java python
(integer) 3
> linsert ireader before java ruby
(integer) 4
> lrange ireader 0 -1
1) "go"
2) "ruby"
3) "java"
4) "python"

~~~

到目前位置，我还没有在实际应用中发现插入指定的应用场景。

**删除元素** 列表的删除操作也不是通过指定下标来确定元素的，你需要指定删除的最大个数以及元素的值

~~~java
> rpush ireader go java python
(integer) 3
> lrem ireader 1 java
(integer) 1
> lrange ireader 0 -1
1) "go"
2) "python"

~~~

**定长列表** 在实际应用场景中，我们有时候会遇到「定长列表」的需求。比如要以走马灯的形式实时显示中奖用户名列表，因为中奖用户实在太多，能显示的数量一般不超过100条，那么这里就会使用到定长列表。维持定长列表的指令是ltrim，需要提供两个参数start和end，表示需要保留列表的下标范围，范围之外的所有元素都将被移除。

~~~java
> rpush ireader go java python javascript ruby erlang rust cpp
(integer) 8
> ltrim ireader -3 -1
OK
> lrange ireader 0 -1
1) "erlang"
2) "rust"
3) "cpp"

~~~

如果指定参数的end对应的真实下标小于start，其效果等价于del指令，因为这样的参数表示需要需要保留列表元素的下标范围为空。

![1640928371390](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/132612-291089.png)

如果再深入一点，你会发现Redis底层存储的还不是一个简单的linkedlist，而是称之为快速链表quicklist的一个结构。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。所以Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

#### 使用场景

1. 依赖list的数据具有顺序的特征对信息进行管理
2. 使用队列模型解决多路信息汇总合并的问题
3. 使用栈模型解决最新消息的问题

> 1、lpush+lpop=Stack（栈）
>  2、lpush+rpop=Queue（队列）
>  3、lpush+ltrim=Capped Collection（有限集合）
>  4、lpush+brpop=Message Queue（消息队列）
>  5、排行榜，数据最新列表等等

### Set(集合)

#### Set介绍

Set 就是一个集合，集合的概念就是一堆**不重复值**的组合。利用 Redis 提供的 Set  数据结构，可以存储一些集合性的数据。比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。因为 Redis  非常人性化的为集合提供了求交集、并集、差集等操作，那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

1. 共同好友、二度好友
2. 利用唯一性，可以统计访问网站的所有独立 IP
3. 好友推荐的时候，根据 tag 求交集，大于某个 threshold 就可以推荐

Java程序员都知道HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

Set对象的编码可以为intset或者hashtable

- intset编码：使用整数集合作为底层实现，set对象包含的所有元素都被保存在intset整数集合里面
- hashtable编码：使用字典作为底层实现，字典键key包含一个set元素，而字典的值则都为null

#### inset编码Set对象结构：

~~~java
redis> SAD number  1 3 5 
~~~

![1640933934961](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145903-333280.png)

#### hashtable编码Set对象结构：

另一方面， hashtable 编码的集合对象使用**字典**作为底层实现， **字典的每个键都是一个字符串对象， 每个字符串对象包含了一个集合元素， 而字典的值则全部被设置为 NULL** 。

~~~java
redis> SAD Dfruits  “apple”  "banana" " cherry"
~~~

![1640933964059](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/145936-170350.png)

#### Set对象的编码转换：

当集合对象可以同时满足以下两个条件时， 对象使用 intset 编码：

1. set对象保存的所有元素都是整数值
2. set对象保存的元素数量不超过512个

不能满足这两个条件的Set对象使用hashtable编码

**注意 :** 第二个条件的上限值是可以修改的， 具体请看配置文件中关于 set-max-intset-entries 选项的说明。对于使用 intset 编码的集合对象来说， 当使用 intset 编码所需的两个条件的任意一个不能被满足时， 对象的编码转换操作就会被执行： 原本保存在整数集合中的所有元素都会被转移并保存到字典里面， 并且对象的编码也会从 intset 变为 hashtable。

#### Set操作

**增加元素** 可以一次增加多个元素

~~~java
> sadd ireader go java python
(integer) 3
~~~

**读取元素** 使用smembers列出所有元素，使用scard获取集合长度，使用srandmember获取随机count个元素，如果不提供count参数，默认为1

~~~java
> sadd ireader go java python
(integer) 3
> smembers ireader
1) "java"
2) "python"
3) "go"
> scard ireader
(integer) 3
> srandmember ireader
"java"

~~~

**删除元素** 使用srem删除一到多个元素，使用spop删除随机一个元素

~~~java
> sadd ireader go java python rust erlang
(integer) 5
> srem ireader go java
(integer) 2
> spop ireader
"erlang"

~~~

**判断元素是否存在** 使用sismember指令，只能接收单个元素

~~~java
> sadd ireader go java python rust erlang
(integer) 5
> sismember ireader rust
(integer) 1
> sismember ireader javascript
(integer) 0

~~~

> > （1）集合对象的编码可以是 intset 或者 hashtable 。
>
> > （2）intset 编码的集合对象使用整数集合作为底层实现。
>
> > （3）hashtable 编码的集合对象使用字典作为底层实现。
>
> > （4）intset 与 hashtable 编码之间，符合条件的情况下，可以转换。

### Sorted Set(有序集合)

#### Sorted Sets简介

和Sets相比，Sorted Sets是将 Set 中的元素增加了一个权重参数 score，使得集合中的元素能够按 score  进行有序排列，比如一个存储全班同学成绩的 Sorted Sets，其集合 value 可以是同学的学号，而 score  就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。另外还可以用 Sorted Sets 来做带权重的队列，比如普通消息的  score 为1，重要消息的 score 为2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。

1. 带有权重的元素，比如一个游戏的用户得分排行榜
2. 比较复杂的数据结构，一般用到的场景不算太多

![1640928695856](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/133137-460046.png)

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, 
Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层实现使用了两个数据结构，**第一个是hash，第二个是跳跃列表**，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。跳跃列表的目的在于给元素value排序，根据score的范围获取元素列表。

> ZSet对象的编码 可以为ziplist或者skiplist
> ziplist编码，每个集合元素使用相邻的两个压缩列表节点保存，一个保存元素成员，一个保存元素的分值，然后根据分数进行从小到大排序。

**zipList编码的ZSet对象结构**

![1640932417080](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/143445-98984.png)

**skiplist编码的ZSet对象使用了zset结构，包含一个字典和一个跳跃表**

~~~java
Type struct zset{

    Zskiplist *zsl；
    dict *dict；
    ...
}

skiplist编码的ZSet对象结构
~~~

![1640932533572](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/143535-705327.png)

**ZSet对象的编码转换**

当ZSet对象同时满足以下两个条件时，对象使用ziplist编码

```apache
1. 有序集合保存的元素数量小于128个
2. 有序集合保存的所有元素的长度都小于64字节
```

不能满足以上两个条件的有序集合对象将使用skiplist编码。  

#### 原理

跳跃表是一种有序数据结构，通过每个节点中维持多个指向其他节点指针，达到快速访问；

多个节点指针：意味着跳跃表的层数；

![1640932271622](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640932271622.png)

- 层：每个节点的多个指针，是一个指针集合List；同层之前指针关联；
- 前进指针：每一层的从头指向后面的指针；
- 跨度：记录两个节点间的距离；
- 后退指针：每个节点，只有一个后退指针，也就是全部后退指针相当于一个单链表；
- 分值Score：跳跃表，按照Score大小排序；
- 成员：也就是此节点的value；

复杂度：增删改查O（logn），最坏O（n）

**源码**

~~~java
// zset结构
typedef struct zskiplist{
    // 头节点，尾节点
    structz skiplisNode *header , *tail;
    // 节点数量
    unsigned long length;
    // 最大的节点层数；
    int level;
}

// 跳跃表的结构
typedef struct skiplisNode{
	// 层：是一个节点集合
    struct zskiplistLevel{
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsigned int span;
    } level[];
	// 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    // 成员对象
    robj *obj;

}
~~~

#### Sorted Set操作

**增加元素** 通过zadd指令可以增加一到多个value/score对，score放在前面

~~~java
> zadd ireader 4.0 python
(integer) 1
> zadd ireader 4.0 java 1.0 go
(integer) 2
~~~

**长度** 通过指令zcard可以得到zset的元素个数

```java
`> zcard ireader``(``integer``) 3`
```

**删除元素** 通过指令zrem可以删除zset中的元素，可以一次删除多个

```java
`> zrem ireader go python``(``integer``) 2`
```

**计数器** 同hash结构一样，zset也可以作为计数器使用。

```java
`> zadd ireader 4.0 python 4.0 java 1.0 go``(``integer``) 3``> zincrby ireader 1.0 python``"5"`
```

**获取排名和分数** 通过zscore指令获取指定元素的权重，通过zrank指令获取指定元素的正向排名，通过zrevrank指令获取指定元素的反向排名[倒数第一名]。正向是由小到大，负向是由大到小。

```java
`> zscore ireader python``"5"``> zrank ireader go # 分数低的排名考前，rank值小``(``integer``) 0``> zrank ireader java``(``integer``) 1``> zrank ireader python``(``integer``) 2``> zrevrank ireader python``(``integer``) 0`
```

根据排名范围获取元素列表 通过zrange指令指定排名范围参数获取对应的元素列表，携带withscores参数可以一并获取元素的权重。通过zrevrange指令按负向排名获取元素列表[倒数]。正向是由小到大，负向是由大到小。

```java
`> zrange ireader 0 -1 # 获取所有元素``1) ``"go"``2) ``"java"``3) ``"python"``> zrange ireader 0 -1 withscores``1) ``"go"``2) ``"1"``3) ``"java"``4) ``"4"``5) ``"python"``6) ``"5"``> zrevrange ireader 0 -1 withscores``1) ``"python"``2) ``"5"``3) ``"java"``4) ``"4"``5) ``"go"``6) ``"1"`
```

**根据score范围获取列表** 通过zrangebyscore指令指定score范围获取对应的元素列表。通过zrevrangebyscore指令获取倒排元素列表。正向是由小到大，负向是由大到小。参数-inf表示负无穷，+inf表示正无穷。

```java
`> zrangebyscore ireader 0 5``1) ``"go"``2) ``"java"``3) ``"python"``> zrangebyscore ireader -inf +inf withscores``1) ``"go"``2) ``"1"``3) ``"java"``4) ``"4"``5) ``"python"``6) ``"5"``> zrevrangebyscore ireader +inf -inf withscores # 注意正负反过来了``1) ``"python"``2) ``"5"``3) ``"java"``4) ``"4"``5) ``"go"``6) ``"1"`
```

根据范围移除元素列表 可以通过排名范围，也可以通过score范围来一次性移除多个元素

```java
`> zremrangebyrank ireader 0 1``(``integer``) 2 # 删掉了2个元素``> zadd ireader 4.0 java 1.0 go``(``integer``) 2``> zremrangebyscore ireader -inf 4``(``integer``) 2``> zrange ireader 0 -1``1) ``"python"`
```

**跳跃列表** zset内部的排序功能是通过「跳跃列表」数据结构来实现的，它的结构非常特殊，也比较复杂。这一块的内容深度读者要有心理准备。

因为zset要支持随机的插入和删除，所以它不好使用数组来表示。我们先看一个普通的链表结构。
 ![1640929003859](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/133644-208990.png)

我们需要这个链表按照score值进行排序。这意味着当有新元素需要插入时，需要定位到特定位置的插入点，这样才可以继续保证链表是有序的。通常我们会通过二分查找来找到插入点，但是二分查找的对象必须是数组，只有数组才可以支持快速位置定位，链表做不到，那该怎么办？

想想一个创业公司，刚开始只有几个人，团队成员之间人人平等，都是联合创始人。随着公司的成长，人数渐渐变多，团队沟通成本随之增加。这时候就会引入组长制，对团队进行划分。每个团队会有一个组长。开会的时候分团队进行，多个组长之间还会有自己的会议安排。公司规模进一步扩展，需要再增加一个层级——部门，每个部门会从组长列表中推选出一个代表来作为部长。部长们之间还会有自己的高层会议安排。

跳跃列表就是类似于这种层级制，最下面一层所有的元素都会串起来。然后每隔几个元素挑选出一个代表来，再将这几个代表使用另外一级指针串起来。然后在这些代表里再挑出二级代表，再串起来。最终就形成了金字塔结构。

想想你老家在世界地图中的位置：亚洲–>中国->安徽省->安庆市->枞阳县->汤沟镇->田间村->xxxx号，也是这样一个类似的结构。
 ![1640929092410](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/133813-916848.png)

「跳跃列表」之所以「跳跃」，是因为内部的元素可能「身兼数职」，比如上图中间的这个元素，同时处于L0、L1和L2层，可以快速在不同层次之间进行「跳跃」。

定位插入点时，先在顶层进行定位，然后下潜到下一级定位，一直下潜到最底层找到合适的位置，将新元素插进去。你也许会问那新插入的元素如何才有机会「身兼数职」呢？

跳跃列表采取一个随机策略来决定新元素可以兼职到第几层，首先L0层肯定是100%了，L1层只有50%的概率，L2层只有25%的概率，L3层只有12.5%的概率，一直随机到最顶层L31层。绝大多数元素都过不了几层，只有极少数元素可以深入到顶层。列表中的元素越多，能够深入的层次就越深，能进入到顶层的概率就会越大。

#### 使用场景

各类综艺选秀海选投票 各类资源网站TOP10（电影，歌曲，文档，电商，游戏等） 

### 跳跃表

#### 业务场景

场景来自小灰的算法之旅，我们需要做一个拍卖行系统，用来查阅和出售游戏中的道具，类似于魔兽世界中的拍卖行那样，还有以下需求：

拍卖行拍卖的商品需要支持四种排序方式，分别是：按价格、按等级、按剩余时间、按出售者ID排序，**排序查询要尽可能地快。还要支持输入道具名称的精确查询和不输入名称的全量查询。**

这样的业务场景所需要的数据结构该如何设计呢？

拍卖行商品列表是线性的，最容易表达线性结构的是**数组和链表**。

假如用有序数组，虽然查找的时候可以使用二分法（时间复杂度O(logN)），但是插入的时间复杂度是O(N)，总体时间复杂度是O(N)；

而如果要使用有序链表，虽然插入的时间复杂度是O(1)，但是查找的时间复杂度是O(N)，总体还是O(N)。

那有没有一种数据结构，查找时，有二分法的效率，插入时有链表的简单呢？有的，就是 跳表。

#### skiplist

skiplist，即跳表，又称跳跃表，也是一种数据结构，**用于解决算法问题中的查找问题。**

一般问题中的查找分为两大类，一种是基于各种平衡术，时间复杂度为O(logN)，一种是基于哈希表，时间复杂度O(1)。但是skiplist比较特殊，没有在这里面

#### 跳表简介

跳表也是链表的一种，是在链表的基础上发展出来的，我们都知道，链表的插入和删除只需要改动指针就行了，时间复杂度是O(1

但是插入和删除必然伴随着查找，而查找需要从头/尾遍历，时间复杂度为O(N)，如下图所示是一个有序链表（最左侧的灰色表示一个空的头节点）（图片来自网络，以下同）：

![1640929713797](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/134840-836526.png)

链表中，每个节点都指向下一个节点，想要访问下下个节点，必然要经过下个节点，即无法跳过节点访问，假设，现在要查找22，我们要先后查找 3->7->11->19->22，需要五次查找。

但是如果我们能够实现跳过一些节点访问，就可以提高查找效率了，所以对链表进行一些修改，如下图：

![1640929735458](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640929735458.png)

我们每个一个节点，都会保存指向下下个节点的指针，这样我们就能跳过某个节点进行访问，这样，我们其实是构造了两个链表，新的链表之后原来链表的一半。

我们姑且称原链表为第一层，新链表为第二层，第二层是在第一层的基础上隔一个取一个。假设，现在还是要查找22，我们先从第二层查找，从7开始，7小于22，再往后，19小于22，再往后，26大于22，所以从节点19转到第一层，找到了22，先后查找  7->19->26->22，只需要四次查找。

以此类推，如果再提取一层链表，查找效率岂不是更高，如下图：

![1640929753621](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/134921-832065.png)

现在，又多了第三层链表，第三层是在第二层的基础上隔一个取一个，假设现在还是要查找22，我们先从第三层开始查找，从19开始，19小于22，再往后，发现是空的，则转到第二层，19后面的26大于22，转到第一层，19后面的就是22，先后查找  19->26>22，只需要三次查找。

由上例可见，在查找时，跳过多个节点，可以大大提高查找效率，skiplist 就是基于此原理。

上面的例子中，每一层的节点个数都是下一层的一半，这种查找的过程有点类似二分法，查找的时间复杂度是O(logN)，但是例子中的多层链表有一个致命的缺陷，就是一旦有节点插入或者删除，就会破坏这种上下层链表节点个数是2:1的结构，如果想要继续维持，则需要在插入或者删除节点之后，对后面的所有节点进行一次重新调整，这样一来，插入/删除的时间复杂度就变成了O(N)。

#### 跳表层级之间的关系

如上所述，跳表为了解决插入和删除节点时造成的后续节点重新调整的问题，引入了随机层数的做法。相邻层数之间的节点个数不再是严格的2:1的结构，而是为每个新插入的节点赋予一个随机的层数。下图展示了如何通过一步步的插入操作从而形成一个跳表：

![1640929822526](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/31/135024-998401.png)

每一个节点的层数都是随机算法得出的，插入一个新的节点不会影响其他节点的层数，因此，插入操作只需要修改插入节点前后的指针即可，避免了对后续节点的重新调整。这是跳表的一个很重要的特性，也是跳表性能明显由于平衡树的原因，因为平衡树在失去平衡之后也需要进行平衡调整。

上图最后的跳表中，我们需要查找节点22，则遍历到的节点依次是：7->37->19->22，可见，这种随机层数的跳表的查找时可能没有2:1结构的效率，但是却解决了插入/删除节点的问题。

#### 跳表的复杂度

跳表搜索的时间复杂度平均 O(logN)，最坏O(N)，空间复杂度O(2N)，即O(N)