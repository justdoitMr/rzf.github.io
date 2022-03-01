## 搭建DWD层数据

### DWD层的任务

1. 对用户行为数据解析。
2. 对核心数据进行判空过滤。
3. **对业务数据采用维度模型重新建模**。

日志按照内容进行划分，一共有五类：

- 页面数据
- 事件日志
- 曝光日志
- 启动日志
- 错误日志

按照日志结构分类：

- 页面埋点日志
- 启动日志

> 本项目中，对数据的解析式按照内容进行解析。

### 日志数据的解析

get_json_object是hive中专门用来解析json对象的工具。

> 说明：数据采用parquet存储方式，是可以支持切片的，不需要再对数据创建索引。如果单纯的text方式存储数据，需要采用支持切片的，lzo压缩方式并创建索引。

#### 启动日志表 

启动日志解析思路：启动日志表中每行数据对应**一个启动记录**，一个启动记录应该包含日志中的**公共信息和启动信息**。先将所有包含start字段的日志过滤出来，然后使用get_json_object函数解析每个字段。启动日志表中的数据来自于启动日志。

> dwd,dws,dwt三层都是采用parquet列式存储格式，然后使用lzo进行数据的压缩。采用列式存储，对于数据的查询有好处，因为后面我们都是做聚合操作，需要对某一列做聚合操作，不需要查询其他的列，而列式存储刚好有利于对某一列进行查询。

> parquet存储+lzo压缩式什么格式呢？
>
> 文件的格式本质上还式parquet的格式，压缩指的是parquet内部每一个列压缩采用的格式。

> LZO支持切片，Snappy不支持切片。
>
> ORC和Parquet都是列式存储。
>
> ORC和Parquet 两种存储格式都是不能直接读取的，一般与压缩一起使用，可大大节省磁盘空间。
>
> 选择：**ORC文件支持Snappy压缩，但不支持lzo压缩，所以在实际生产中，使用Parquet存储 + lzo压缩的方式更为常见，这种情况下可以避免由于读取不可分割大文件引发的数据倾斜**。

##### 启动日志数据格式

- 公共字段
- start启动字段
- 时间戳

![20211217145428](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211217145428.png)

> 启动日志里面的每一条数据代表一条启动信息。

在这里所有表插入数据的时候，使用的式overwrite，为了就是保证**幂等性**，如果任务失败，重试写入数据的时候，不会发生数据的重复。

##### 创建表

![1640320528921](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/123605-719718.png)

##### 导入数据

数据来源与ods_log表。

#### 页面日志表

**页面日志解析思路：**页面日志表中每行数据对应**一个页面访问记录**，一个页面访问记录应该包含日志中的公共信息和页面信息。先将所有包含page字段的日志过滤出来，然后使用get_json_object函数解析每个字段。数据来自于页面埋点日志数据。

- 公共字段
- 页面字段

**数据格式**

![20211217150203](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211217150203.png)

页面日志中，用改包含common字段，page信息和ts时间戳字段信息。

> 页面日志中的一条记录表示一个页面访问。

##### 创建表

![1640320885891](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/124127-28282.png)

##### 数据导入

数据来源于ods_log表。

#### 动作日志表

动作日志解析思路：动作日志表中每行数据对应用户的**一个动作记录**，一个动作记录应当包含**公共信息、页面信息以及动作信息**。先将包含action字段的日志过滤出来，然后通过UDTF函数，将action数组“炸开”（类似于explode函数的效果），然后使用get_json_object函数解析每个字段。

**为什么这里需要使用自定义udtf函数，因为explode()函数接收的式一个数组或者map才可以炸裂，但是ods层存储的json以字符串形式存储，所以我们需要自定义udtf函数将字符串转换为json数组**。

- 公共字段
- action动作字段是一个数组。

##### udtf函数设计思路

![20211217180316](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211217180316.png)

##### 自定义udtf函数

initialize():

- 限定自定义函数的输入类型，这样就可以对数据做校验。
- 限定自定义函数的输出类型。

process():

- 对每一行数据的处理逻辑

forward():

- 将解析完成的数据进行输出。

在本项目中，initialize()方法中需要完成的功能：

1. 完成输入数据类型的校验。
2. 封装输出数据的类型

在process()中，完成字符串到数组的转换。

在hive中，临时函数只在本次的会话中有效，自定义函数的包放在hive所在的节点的本地即可，永久函数需要将自定义函数的jar包放在hdfs上面。

永久函数也有库的概念，如果在gmall库下面创建的函数，那么如果在其他数据库下面使用函数，需要使用gmall.进行引用。

##### 在hive中创建函数的语法

下面创建的是永久函数，

```java
create function explode_json_array as 'com.rzf.hive.udtf.ExplodeJSONArray' using jar 'hdfs://hadoop102:8020/user/hive/jars/hivefunction-1.0-SNAPSHOT.jar'
//usering jar指向hdfs上面jar包的路径;
```

经过炸裂函数，我们建立的虚表中有三个字段，line字段，dt分区字段和action字段。

**炸裂后虚表的结构**

![20211217175253](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211217175253.png)

##### 创建表

![1640321168624](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/124705-159083.png)

##### 导入数据

```sql
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_action_log partition(dt='2020-06-14')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.page.during_time'),
    get_json_object(line,'$.page.item'),
    get_json_object(line,'$.page.item_type'),
    get_json_object(line,'$.page.last_page_id'),
    get_json_object(line,'$.page.page_id'),
    get_json_object(line,'$.page.sourceType'),
    get_json_object(action,'$.action_id'),
    get_json_object(action,'$.item'),
    get_json_object(action,'$.item_type'),
    get_json_object(action,'$.ts')
from ods_log lateral view explode_json_array(get_json_object(line,'$.actions')) tmp as action
where dt='2020-06-14'
and get_json_object(line,'$.actions') is not null;
```

可以看到，曝光字段是从action的json对象中解析，这里没有使用ts时间戳字段，因为每一个动作中都自带时间戳字段。

需要注意的是，上面自定义函数传入的是一个json数组，但是这个数组是以字符串的形式存在的，我们自定义udtf函数的目的就是将字符串的数组转换为数组的格式，然后再炸裂开，**因为hive中的炸裂函数需要使用数组参数或者map参数**。

#### 曝光日志表

曝光日志解析思路：曝光日志表中每行数据对应**一个曝光记录**，一个曝光记录应当包含**公共信息、页面信息以及曝光信息**。先将包含display字段的日志过滤出来，然后通过UDTF函数，将display数组“炸开”（类似于explode函数的效果），然后使用get_json_object函数解析每个字段。

- 公共字段
- 页面信息
- 曝光信息

**过程**

![20211217180656](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211217180656.png)

> 但是在这里不需要再自定义一个udtf函数，因为上面写的解析动作日志的udtf函数很通用，我们可以将曝光字符串数组传入自定义的udtf函数，然后返回一个解析好的数组，最后使用炸裂函数。

> 曝光日志表中每行数据对应**一个曝光记录**

##### 创建表

![1640321330152](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/124857-698073.png)

#### 错误日志表

错误日志解析思路：错误日志表中每行数据对应**一个错误记录**，为方便定位错误，一个错误记录应当包含与之对应的**公共信息、页面信息、曝光信息、动作信息、启动信息以及错误信息**。先将包含err字段的日志过滤出来，然后使用get_json_object函数解析所有字段。

为什么错误字段包含这么多信息？

因为再启动，曝光和点击的时候，都有可能发生页面的错误。

再错误日志中，存在一对多的现象，因为再动作日志和曝光日志中可能存在多个动作和曝光，也就是一个错误对应一个页面，但是一个页面又对应者多个动作或者多个曝光。

> 再动作日志和曝光日志中都存在一对多的情况。

> 再本项目中，一个错误对应一个页面，但是并没有对动作信息和曝光信息进行解析。实际项目中，可以根据需求进行解析。

解决lzo索引失效问题：

```sql
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
```

再向表中插入数据的时候，首先需要设置以下上面的参数。

> 错误日志表中每行数据对应**一个错误记录**

##### 创建表

![1640321411231](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/125012-921669.png)

### 业务数据的解析

业务数据方面DWD层的搭建主要注意点在于维度建模，减少后续大量Join操作，dwd层的数据式粒度最细的数据。

> **一共有8张事实表，6张维度表**。
>
> 事务性事实表很简单，拿到新增的数据然后追加到表中即可。
>
> 加购和收藏是周期性事实表，因为我们并不是很关心数据的增删操作，做起来相对简单，
>
> 每日快照，累积性快照事实表，需要修改表中的数据，一条数据分多次写入表，等获取到新数据之后，还会修改数据，所以比较麻烦。

- 维度表的特点，数据量不是很大并且数据稳定，所以一般都是**全量导入**，还有一些采用**增量及变化**导入，使用增量导入方式很少，因为本来数据就少，变化的不多，所以一般都是用全量，另外对于一些特殊的维度表，只需要导入一次即可。

- 如果采用新增及变化，那么就需要考虑数据仓库中的数据和数据库中的数据同步问题，所以需要将新增及变化数据和前一天的数据进行一个整和，需要使用**拉链表**，这种维度表一般对应的数据量会很大，不适合使用全量同步，例如我们的用户表。

业务数据方面DWD层的搭建主要注意点在于维度建模，减少后续大量Join操作。

再本项目中，事实表和维度表之间的关系：

![1640260074711](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/194755-592372.png)

> 事实表和维度表是根据需要的业务，总结出来的，所以我们可以根据具体的业务线，总结出需要的业务然后形成事实表。
>
> 而维度表就是我们之后根据事实表进行分析的维度，说白了也就是group by的字段。

#### 商品维度表（全量）

商品维度表主要是将商品表SKU表、商品一级分类、商品二级分类、商品三级分类、商品品牌表和商品SPU表联接为商品表。

表字段来源：

- sku表
- 商品一级分类
- 商品二级分类
- 商品三级分类

##### 创建表

![1640260103210](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/194824-324645.png)

##### 数据导入

表中的数据来自：

1. ods_sku_info
2. ods_base_trademark
3. ods_spu_info
4. ods_base_category2
5. ods_base_category1
6. ods_base_category3

![1640325144375](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/135225-79581.png)

**代码实现**

~~~sql
-- 加载数据,在这里获取数据的时候，应该让每一张表的最新的分区之间去join操作，尽量不要全表join操作，应该先过滤操作
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_sku_info partition(dt='2020-06-22')
select
    sku.id,
    sku.spu_id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.tm_id,
    ob.tm_name,
    sku.category3_id,
    c2.id category2_id,
    c1.id category1_id,
    c3.name category3_name,
    c2.name category2_name,
    c1.name category1_name,
    spu.spu_name,
    sku.create_time
from
(
    -- 查询商品详情中的表
    select * from ods_sku_info where dt='2020-06-22'
)sku
join
(
    -- 查询品牌表中的数据
    select * from ods_base_trademark where dt='2020-06-22'
)ob on sku.tm_id=ob.tm_id
join
(
    -- 查询spu商品表中的数据
    select * from ods_spu_info where dt='2020-06-22'
)spu on spu.id = sku.spu_id
join
(
    -- 查询第三级分类中的数据
    select * from ods_base_category3 where dt='2020-06-22'
)c3 on sku.category3_id=c3.id
join
(
    -- 查询第二级分类中的数据
    select * from ods_base_category2 where dt='2020-06-22'
)c2 on c3.category2_id=c2.id
join
(
    --查询一级分类中的数据
    select * from ods_base_category1 where dt='2020-06-22'
)c1 on c2.category1_id=c1.id;
~~~

#### 优惠券维度表（全量）

再ods层，和优惠券有关的表是ods_coupon_info表，所以再dwd层，优惠券维度表和ods层的表字段相同，只需要将ods层关于优惠券表的数据稍作清洗然后导入优惠券维度表即可。

**字段来源**

- ods_coupon_info

##### 创建表

![1640322558373](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/130919-795356.png)

##### 数据导入

- ods_coupon_info

将ods_coupon_info表中的数据直接导入。此张表中即可。

~~~sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_coupon_info partition(dt='2020-06-22')
select
    id,
    coupon_name,
    coupon_type,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    create_time,
    range_type,
    spu_id,
    tm_id,
    category3_id,
    limit_num,
    operate_time,
    expire_time
from ods_coupon_info
where dt='2020-06-22';
~~~

#### 活动维度表（全量）

与活动有关的表有activity_info,activity_rule,activity_sku表，正常情况下需要将这三张表的字段合并在一起组成我们的活动维度表，但是活动规则中，一个商品可能参与多种活动，比如满减活动，也就是一个活动有多个优惠级别，所以将activity_info和activity_rule表进行join的话，一个活动会有多行数据。所以之后再将事实表订单表和活动表关联的时候，关联的字段是actovity_id，所以再关联的时候，会找到多个一样的活动id，因为上面已经说了，一个活动id有多个优惠级别。正常情况下，一个订单只能参与一个活动id的某一个优惠级别。所以我们应该再业务系统中获取订单以及订单的优惠级别，然后再关联。这个时候，订单和活动维度表再关联的时候，关联的id就有两个，活动id和优惠级别。

但是再本项目中，无法获取订单和优惠级别，所以就不在进行activity_info和activity_rule表进行关联，再活动维度表中只保留activity_info字段。

> 在做事实表和维度表进行关联的时候，只能是一对一关系。

**表中字段来源**

- ods_activity_info

##### 创建表

![1640323282947](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/132124-13174.png)

##### 导入数据

**数据来源**

- ods_activity_info

~~~ sql
--加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_activity_info partition(dt='2020-06-22')
select
    id,
    activity_name,
    activity_type,
    start_time,
    end_time,
    create_time
from ods_activity_info
where dt='2020-06-22';
~~~

#### 地区维度表（特殊）

地区维度表是一张特殊的表，因为变化很少，所以我们只需要加载一次即可。地区维度表中的数据字段来自ods_base_province和ods_base_region表，省份和省份中的地区。

- ods_base_province
- ods_base_region

##### 创建表

![1640323438881](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/132359-239462.png)

##### 导入数据

![1640325445854](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/135727-862511.png)

**数据来源**

- ods_base_province
- ods_base_region

~~~sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_base_province
select
    bp.id,
    bp.name,
    bp.area_code,
    bp.iso_code,
    bp.region_id,
    br.region_name
from
(
    select * from ods_base_province
) bp
join
(
    select * from ods_base_region
) br
on bp.region_id = br.id;
~~~

地区表是一张特殊表，表中的数据加载一次即可。

#### 时间维度表（特殊）

前面的所有的维度表都是从业务系统中导入过来的数据，但是时间维度表并不是从业务系统中倒过来的，业务系统中并没有这张表，这张表属于数据仓库中的表，因为业务系统中只需要保留最新数据，并不涉及历史数据，所以并不需要时间维度，而数据仓库中需要是因为保存有历史数据，需要时间维度去解释。

时间维度表每一年只需要导入一次，所以并不会将导入数据语句写入到脚本中。之所以需要时间维度，就是我们需要根据时间去分析数据仓库中的数据。

##### 创建表

![1640323536257](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/132539-575614.png)

#### 支付事实表（事务性事实表）

支付事实表一行数据所代表的含义：**代表一次支付事件**。

![20211219125110](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219125110.png)

支付事实表中的维度外键有：用户id,地区id，分区字段就代表时间id，度量值是payment_amount。

支付事实表的主要数据来源是payment_info,所以主要字段页来自于支付事实表，主要数据来源是payment_info，payment_type也可以作为一个度量字段。，

- ods_payment_info
- ods_order_info

##### 创建表

![1640323662907](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/132743-54142.png)

##### 导入数据

![1640325592328](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/135953-23325.png)

**数据来源**

- ods_payment_info
- ods_order_info

**度量值是支付金额**

但是province_id需要从ods_order_info表中去查询，所以需要使用到join。

~~~sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_payment_info partition(dt='2020-06-22')
select
    pi.id,
    pi.out_trade_no,
    pi.order_id,
    pi.user_id,
    pi.alipay_trade_no,
    pi.total_amount,
    pi.subject,
    pi.payment_type,
    pi.payment_time,
    oi.province_id
from
(
    select * from ods_payment_info where dt='2020-06-22'
)pi
join
(
    select id, province_id from ods_order_info where dt='2020-06-22'
)oi
on pi.order_id = oi.id;
~~~

#### 退款事实表（事务性事实表）

把ODS层ods_order_refund_info表数据导入到DWD层退款事实表，在导入过程中可以做适当的清洗。

![20211219132332](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219132332.png)

退款事实表和用户维度，时间维度，商品维度有关，分区字段作为时间维度。

度量值有：**退款的件数，退款的金额**。

退款事实表中数据主要来自ods_order_refund_info，因为退款事实表中的字段和ods_order_refund_info一样，所以不需要进行join操作。

- ods_order_refund_info

##### 创建表

![1640323767472](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/132928-738316.png)

##### 数据导入

数据主要来源

- ods_order_refund_info

~~~ sql
-- 导入数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_order_refund_info partition(dt='2020-06-22')
select
    id,
    user_id,
    order_id,
    sku_id,
    refund_type,
    refund_num,
    refund_amount,
    refund_reason_type,
    create_time
from ods_order_refund_info
where dt='2020-06-22';
~~~

#### 评价事实表(事务性事实表)

把ODS层ods_comment_info表数据导入到DWD层评价事实表，在导入过程中可以做适当的清洗。

![20211219132358](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219132358.png)

评价事实表中，一行数据指的是一条评价信息。涉及的维度有时间，用户，地区维度，商品维度等等。

**度量值是好评的条数。**

评价事实表中的数据和字段基本上来自于ods_comment_info表，所以可以直接将数据倒过来即可,我们只关心好评数，并不关心每天增加了多少条评论。

- ods_comment_info

##### 创建表

![1640262967106](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/203608-323886.png)

##### 导入数据

- ods_comment_info

~~~ sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_comment_info partition(dt='2020-06-22')
select
    id,
    user_id,
    sku_id,
    spu_id,
    order_id,
    appraise,
    create_time
from ods_comment_info
where dt='2020-06-22';
~~~

#### 订单明细事实表（事务型事实表）

![20211219134804](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219134804.png)

订单明细事实表中一条数据表示一个订单中的一个商品的明细。

维度有时间，用户，地区和商品。

**度量值有商品的件数，订单中总金额，分摊优惠，分摊运费，分摊最终。**

订单明细表中的数据主要来自ods_order_detail和ods_order_info表。

- 原始价格分摊：商品的单价乘以商品的数量。
- 优惠分摊：按照原价的比例进行分摊，比如说商品a,b,c原始价格20，30，50。然后满100元优惠20元，所以优惠20最终付费80元，那么分摊到每一个商品中，优惠的价格是4，6，10元，但是这里也会产生问题，会产生误差，比如说优惠10元，每一个商品分摊3.33元，但是最终分摊结果的总和相加没有10元，所以需要补偿这个误差，这里额做法是使用10减去分摊的金额，然后补充在某一个商品分摊的金额上，需要使用开创函数：

![20211219142823](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219142823.png)

- ods_order_detail
- ods_order_info

##### 创建表

![1640263312887](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/204154-980605.png)

##### 导入数据

- ods_order_detail
- ods_order_info

~~~ sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_order_detail partition(dt='2020-06-22')
select
    id,
    order_id,
    user_id,
    sku_id,
    sku_name,
    order_price,
    sku_num,
    create_time,
    province_id,
    source_type,
    source_id,
    original_amount_d,
    if(rn=1,final_total_amount -(sum_div_final_amount - final_amount_d),final_amount_d),
    if(rn=1,feight_fee - (sum_div_feight_fee - feight_fee_d),feight_fee_d),
    if(rn=1,benefit_reduce_amount - (sum_div_benefit_reduce_amount -benefit_reduce_amount_d), benefit_reduce_amount_d)
from
(
    select
        od.id,
        od.order_id,
        od.user_id,
        od.sku_id,
        od.sku_name,
        od.order_price,
        od.sku_num,
        od.create_time,
        oi.province_id,
        od.source_type,
        od.source_id,
        round(od.order_price*od.sku_num,2) original_amount_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.final_total_amount,2) final_amount_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.feight_fee,2) feight_fee_d,
        round(od.order_price*od.sku_num/oi.original_total_amount*oi.benefit_reduce_amount,2) benefit_reduce_amount_d,
        row_number() over(partition by od.order_id order by od.id desc) rn,--开窗
        oi.final_total_amount,
        oi.feight_fee,
        oi.benefit_reduce_amount,--分摊优惠之和
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.final_total_amount,2)) over(partition by od.order_id) sum_div_final_amount,
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.feight_fee,2)) over(partition by od.order_id) sum_div_feight_fee,
        sum(round(od.order_price*od.sku_num/oi.original_total_amount*oi.benefit_reduce_amount,2)) over(partition by od.order_id) sum_div_benefit_reduce_amount
    from
    (
        select * from ods_order_detail where dt='2020-06-22'
    ) od
    join
    (
        select * from ods_order_info where dt='2020-06-22'
    ) oi
    on od.order_id=oi.id
)t1;
~~~

#### 加购事实表（周期型快照事实表，每日快照）

每日全量表，因为我们关心的是购物车中的商品的件数，并不关系购物车中的商品每天新增或者减少。

由于购物车的数量是**会发生变化**，所以导增量不合适。

每天做一次快照，导入的数据是**全量**，区别于事务型事实表是每天导入**新增**。

周期型快照事实表劣势：存储的数据量会比较大。

解决方案：周期型快照事实表存储的数据比较讲究**时效性**，时间太久了的意义不大，可以删除以前的数据。

数据组要来源于：

- ods_cart_info

![20211219144352](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219144352.png)

一行数据代表，一个用户购物车中的一个商品。

度量值，一个用户的购物车中某一个商品一共有多少件，购物车中商品的总金额是多少（加入购物车商品的数量乘以单价）。

##### 创建表

![1640263557736](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/204559-865867.png)

##### 导入数据

数据主要来源

- ods_cart_info

~~~sql
-- 加载数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_cart_info partition(dt='2020-06-14')
select
    id,
    user_id,
    sku_id,
    cart_price,
    sku_num,
    sku_name,
    create_time,
    operate_time,
    is_ordered,
    order_time,
    source_type,
    source_id
from ods_cart_info
where dt='2020-06-22';
~~~

#### 收藏事实表（周期型快照事实表，每日快照）

![1640263725174](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/204846-877189.png)

收藏的标记，是否取消，会发生变化，做增量不合适。

每天做一次快照，导入的数据是全量，区别于事务型事实表是每天导入新增。

度量值，某个商品收藏的个数，通过累计行数确定。

数据主要来源：

- ods_favor_info

##### 创建表

![1640263697772](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/204818-97038.png)

##### 导入数据

数据来源

- ods_favor_info

~~~sql
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_favor_info partition(dt='2020-06-14')
select
    id,
    user_id,
    sku_id,
    spu_id,
    is_cancel,
    create_time,
    cancel_time
from ods_favor_info
where dt='2020-06-14';
~~~

#### 优惠券领用事实表（累积型快照事实表）

![20211219150224](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219150224.png)

优惠卷的生命周期：领取优惠卷-》用优惠卷下单-》优惠卷参与支付
累积型快照事实表使用：统计优惠卷领取次数、优惠卷下单次数、优惠卷参与支付次数。

维度字段，用户id，优惠券id，订单id，时间维度等，度量值是行数，表示一个优惠券被多少人领取。

对于事务性事实表，就是增量表，只需要将mysql中的增量数据存放到hive中当天的分区中。

周期性事实表也是将mysql中的全量数据放到hive表中当天的分区中。

累积性事实表在每一天向hive表中导入数据的时候，需要修改之前的数据，比如优惠券有领取时间，使用时间和过期时间，但是这几个时间不是一次性就直接可以写满，需要根据下单的过程进行去修改时间，那么mysql中修改过的时间需要同步到hive表中，所以需要修改历史数据。做法就是根据修改的数据，去hive表中查询数据然后进行更新操作，相当于修改数仓中的历史数据。

hive中也可以支持修改或者删除操作，只不过只有分桶表支持，因为如果是普通的表，因为hive支持一张表中数据量非常大，如果需要删除或者修改，会逐条判断，性能不高。所以hive采用分桶，将一张表中的数据分到多个桶中，也就是多个文件当中，首先定位文件，然后再小文件中修改操作。

但是我们一般不会进行分桶操作，而是采用另一种方法，先把hive表中的所有数据全部查询出来。然后对每一个字段逐个判断，使用if或者case when语句，判断哪一个字段需要修改，哪一个不需要修改，直接在select的过程中直接修改，然后再把查询出的结果写回去。

> 这种方式是便查询便修改，如果修改了，再把数据写会原表即可。

这一张表必须分区，如果不分区，那么比如修改表中100跳数据，就需要以整个表为单位进行查询然后修改，如果进行分区，那么就是以分区为单位进行查询和修改。

这里按照时间进行分区，但是每一个分区里面存储什么内容？

分区里面存储的是每一天新增的优惠券，如果是修改，那么就去之前的分区中，找到领取优惠券的那条记录，然后进行修改。

> 这个需求比较难

需要用到动态分区。

##### 创建表

![1640263776797](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/23/204937-537190.png)

##### 导入数据

![1640326269841](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/141120-474934.png)

数据来源：

- ods_coupon_use
- dwd_fact_coupon_use

~~~sql

-- 加载数据
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_coupon_use partition(dt)
select
    if(new.id is null,old.id,new.id),
    if(new.coupon_id is null,old.coupon_id,new.coupon_id),
    if(new.user_id is null,old.user_id,new.user_id),
    if(new.order_id is null,old.order_id,new.order_id),
    if(new.coupon_status is null,old.coupon_status,new.coupon_status),
    if(new.get_time is null,old.get_time,new.get_time),
    if(new.using_time is null,old.using_time,new.using_time),
    if(new.used_time is null,old.used_time,new.used_time),
    date_format(if(new.get_time is null,old.get_time,new.get_time),'yyyy-MM-dd')
from
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time
    from dwd_fact_coupon_use
    where dt in
    (
        select
            date_format(get_time,'yyyy-MM-dd')
        from ods_coupon_use
        where dt='2020-06-22'
    )
)old
full outer join
(
    select
        id,
        coupon_id,
        user_id,
        order_id,
        coupon_status,
        get_time,
        using_time,
        used_time
    from ods_coupon_use
    where dt='2020-06-22'
)new
on old.id=new.id;
~~~

#### 订单事实表（累积型快照事实表）

![20211219190217](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219190217.png)

订单生命周期：创建时间=》支付时间=》取消时间=》完成时间=》退款时间=》退款完成时间。

由于ODS层订单表只有创建时间和操作时间两个状态，不能表达所有时间含义，所以需要关联订单状态表。订单事实表里面增加了活动id，所以需要关联活动订单表。

表中的一条数据表示一个订单，用户每下一个单，表中就会产生一条数据与之对应。

维度外键，用户，省份，时间，活动id维度等等，度量值：原价金额，优惠金额，运费和订单金额。

这个表是一个典型的累积性宽窄事实表，从字段中可以看出，下一个订单之后，订单有很多的中间状态，没完成一部，都需要修改订单的状态。

用户下单之后，数据会放入数据仓库，但是下单这个操作也不是一次性完成操作，可能过两天后才完成下单这个操作，所以我们需要跟踪用户下单的操作，及时修改数据仓库中的数据。

数据按照分区存放，因为也需要修改数据，然后分区中存放当天下的订单信息，不存放修改数据，如果有修改数据，就去创建订单的那个分区中修改数据。

**更新操作**

从当天新增及变化的数据中找到变化的订单是那一天下的单，根据**create_time**进行判断，然后根据创建时间我们就可以找到数据所在的分区，因为订单存储再当天创建订单的时间分区里面，然后获取整个分区中的数据，但是我们可能只是修改整个分区中很少的数据，然后我们把拿到的分区中的数据和今天的新增及变化的数据做一个对比，也就是进行join操作，join之后，数据的对应关系也就很清晰了，因为有的数据可以join到，有的数据是新增数据，没有左边的数据，无法关联，有的是变化数据，右边没有无法关联。

![20211219192335](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219192335.png)

就像上图中所示，最上边是老的数据，但是没有发生变化，中间的是关联上的，也就是老数据发生了变化，最下面的是新增数据，那么我们需要找到的就是中间的老数据并且发生变化的，所以关联的时候，我们使用full outer join。

> 这里也需要使用动态分区。

##### 创建表

![1640263995770](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/133203-327182.png)

##### 导入数据

![1640326319568](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/141209-163303.png)

- ods_order_info
- ods_order_status_log
- ods_activity_order

~~~sql
-- 加载数据
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_fact_order_info partition(dt)
select
    if(new.id is null,old.id,new.id),
    if(new.order_status is null,old.order_status,new.order_status),
    if(new.user_id is null,old.user_id,new.user_id),
    if(new.out_trade_no is null,old.out_trade_no,new.out_trade_no),
    if(new.tms['1001'] is null,old.create_time,new.tms['1001']),--1001对应未支付状态
    if(new.tms['1002'] is null,old.payment_time,new.tms['1002']),
    if(new.tms['1003'] is null,old.cancel_time,new.tms['1003']),
    if(new.tms['1004'] is null,old.finish_time,new.tms['1004']),
    if(new.tms['1005'] is null,old.refund_time,new.tms['1005']),
    if(new.tms['1006'] is null,old.refund_finish_time,new.tms['1006']),
    if(new.province_id is null,old.province_id,new.province_id),
    if(new.activity_id is null,old.activity_id,new.activity_id),
    if(new.original_total_amount is null,old.original_total_amount,new.original_total_amount),
    if(new.benefit_reduce_amount is null,old.benefit_reduce_amount,new.benefit_reduce_amount),
    if(new.feight_fee is null,old.feight_fee,new.feight_fee),
    if(new.final_total_amount is null,old.final_total_amount,new.final_total_amount),
    date_format(if(new.tms['1001'] is null,old.create_time,new.tms['1001']),'yyyy-MM-dd')
from
(
    select
        id,
        order_status,
        user_id,
        out_trade_no,
        create_time,
        payment_time,
        cancel_time,
        finish_time,
        refund_time,
        refund_finish_time,
        province_id,
        activity_id,
        original_total_amount,
        benefit_reduce_amount,
        feight_fee,
        final_total_amount
    from dwd_fact_order_info
    where dt
    in
    (
        select
          date_format(create_time,'yyyy-MM-dd')
        from ods_order_info
        where dt='2020-06-22'
    )
)old
full outer join
(
    select
        info.id,
        info.order_status,
        info.user_id,
        info.out_trade_no,
        info.province_id,
        act.activity_id,
        log.tms,
        info.original_total_amount,
        info.benefit_reduce_amount,
        info.feight_fee,
        info.final_total_amount
    from
    (
        select
            order_id,
            str_to_map(concat_ws(',',collect_set(concat(order_status,'=',operate_time))),',','=') tms
        from ods_order_status_log
        where dt='2020-06-22'
        group by order_id
    )log
    join
    (
        select * from ods_order_info where dt='2020-06-14'
    )info
    on log.order_id=info.id
    left join
    (
        select * from ods_activity_order where dt='2020-06-14'
    )act
    on log.order_id=act.order_id
)new
on old.id=new.id;
~~~

#### 用户维度表（拉链表）

**用户表中的数据每日既有可能新增，也有可能修改，但修改频率并不高，属于缓慢变化维度，此处采用拉链表存储用户维度数据**。

##### **什么是拉链表**

> 维度表的特点一般是数据量不是很大，并且数据相对的稳定，基于这样的特点，我们维度表一般都采用**全量的同步法**。那么再数仓里面这张表就叫做每日全量表。
>
> 但是有些维度表，数据量很大，如果这样的表也做每日全量表，存储空间占用大，并且效率低，如果每天做全量，那么维度表本身数据变化很少，所以导过来了很多重复或者没用的数据。
>
> 所以当我们的维度表数据量很大，并且不方便做每日全量同步的话，这个时候，**我们可以使用每日新增及变化，但是使用每日新增及变化，拿到的数据需要和数仓维度表中已经存在的数据做一个整合，所谓的整合就是整合为拉链表**。
>
> **在这里使用拉链表是为了解决数据的存储问题，因为全量的话，重复数据很多**。

![20211219194347](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219194347.png)

##### **为什么需要做拉链表**

拉链表适合于：**数据会发生变化，但是大部分是不变的。（即：缓慢变化维）**

比如：用户信息会发生变化，但是每天变化的比例不高。如果数据量有一定规模，按照每日全量的方式保存效率很低。 比如：1亿用户*365天，每天一份用户信息。(做每日全量效率低)

![20211219194603](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219194603.png)

拉链表中一行数据是一个用户的一个状态，除了普通的字段之外，拉链表中还有两个时间的字段，一个是开始日期，一个是结束日期，代表的是状态的开始日期和结束日期。

拉链表可以降低存储是因为只对状态做一个开始时间和结束时间，因为始终保存的是状态，只要状态不发生改变，那么只会存储一份数据，这样再表中保存一份数据就可以了，如果使用全量同步，那么每天都需要存储一份。

**全量表可以实现的功能，拉链表都可以实现，只是拉链表写起来比较麻烦**。

##### **如何使用拉链表**

![20211219201028](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219201028.png)

比如需要获取全量的最新数据，可以使用结束时间进行过滤即可。因为一个用户只有一个状态是9999.

获取历史上某一天的全量数据，通过开始日期大于等于某一个日期，结束日期小于等于某一个日期的进行过滤。

同样使用事实表也是获取全量最新数据和历史某一天的数据。

##### **拉链表的制作过程**

1. 制作初始化导入，初始化起始时间。把mysql业务数据库中的数据一次性全部导入hive表中，做一次全量同步，结束时间都是9999，其实时间一般是根据实际情况指定一个时间。
2. 后续mysql业务数据库中的数据会发生新增和变化，需要把mysql中新增和变化的数据导入Mysql数据仓库中，通过crerate_time和operator_time两个字段获取新增和变化的数据。
3. 接下来将新增和变化的数据整合到拉链表中，首先将获取到的新增及变化数据补充上字段，开始时间就是今天，结束时间是9999，表示新增或者变化的数据是从今天开始，状态发生了变化。
4. 新增及变化数据处理好了，还需要处理原来拉链表中的历史数据，比如有些数据更新了，那么需要把过期的数据的结束日期修改下，一般修改为昨天的日期，必须保证一个用户所有状态之间的时间不会发生相互重合。
5. 最后将新增和变化的数据和处理过的历史数据进行unioin上下合并即可。
6. 再修改历史数据的过程中有涉及到修改数据的操作，在这里也是先查出数据，然后边查询边修改，最后再插入表中。

整个制作过程如下：

![20211219214012](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219214012.png)

![20211219214042](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211219214042.png)

在这里中间使用了一个临时表，是为了数据安全问题，因为我们再使用insert into或者overwrite的时候，是先删除原始表中的数据，然后再插入数据，但是如果再插入数据时候，任务失败了，那么原始数据也就丢失了。所以中间使用了一个临时表，原始数据并没有丢失。

> 但是实际上上面的考虑是多余的，因为Hive不是按照上面的做法做的，因为hive再插入数据的时候，也是先写入临时表，也就是先写入临时路径下的一个文件，等任务成功后，修改路径和文件名即可。

**第一步：初始化拉链表（首次独立执行）**

用户拉链表和ods层用户信息表字段基本一致，但是需要再添加两个字段，**一个是开始时间，一个是结束时间**。

> 拉链表为什么没有进行分区，因为我们再查询数据的时候，通常是根据时间进行查询的，也就是开始时间大于某一个时间，结束时间小于某一个时间，这样查询数据的话，如果分区，不能提高查询效率。

**步骤二：制作当日变动数据（包括新增，修改）每日执行**

因为本项目中只有2020-6-14号数据，所以基本上全是新增数据，没有变化数据，所以就从ods层中加载全部数据，然后添加上两个时间字段即可。

1. 如何获得每日变动表
   1. 最好表内有创建时间和变动时间（Lucky!）
   2. 如果没有，可以利用第三方工具监控比如canal，监控MySQL的实时变化进行记录（麻烦）。
   3. 逐行对比前后两天的数据，检查md5(concat(全部有可能变化的字段))是否相同(low)
   4. 要求业务数据库提供变动流水（人品，颜值）

**步骤三：先合并变动信息，再追加新增信息，插入到临时表中**

1. 先从ods层根据分区获取最新的的数据，然后对获取到的最新数据添加一个开始时间和结束时间，开始时间是就是获取数据那天的时间，结束时间是9999.
2. 处理老数据就是修改老数据的结束时间为9999，获取老数据是从dwd层的用户维度表。
3. 如何获取老数据，首先获取今天数据的新增和变化的数据，然后和原始表中的数据做一个对比，也就是进行full join，就会产生三部分数据，老数据没有修改，老数据并且修改，新增数据，我们只获取老数据并且修改的数据。

> 再hive中，可以进行union的条件是两边都必须是子查询。不管是几个select子查询union到一起，最终只当作一个select整体额子查询即可。

**步骤四：把临时表覆盖给拉链表**

先把查询的数据放到临时表中，然后整体再插入拉链表中。

##### 创建表

![1640264242908](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/133231-130733.png)

##### 导入数据

- ods_user_info

~~~sql
-- 第一步：初始化拉链表
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_user_info_his
select
    id,
    name,
    birthday,
    gender,
    email,
    user_level,
    create_time,
    operate_time,
    '2021-06-22',
    '9999-99-99'
from ods_user_info oi
where oi.dt='2021-06-22';

-- 制作当日临时数据（包括新增和修改数据）每日执行
-- 2021-06-23
--old_user_info
-- 先合并变动信息，再追加新增信息，插入到临时表中
--2021-06-23


--建立临时拉链表
drop table if exists dwd_dim_user_info_his_tmp;
create external table dwd_dim_user_info_his_tmp(
    `id` string COMMENT '用户id',
    `name` string COMMENT '姓名',
    `birthday` string COMMENT '生日',
    `gender` string COMMENT '性别',
    `email` string COMMENT '邮箱',
    `user_level` string COMMENT '用户等级',
    `create_time` string COMMENT '创建时间',
    `operate_time` string COMMENT '操作时间',
    `start_date`  string COMMENT '有效开始日期',
    `end_date`  string COMMENT '有效结束日期'
) COMMENT '订单拉链临时表'
stored as parquet
location '/warehouse/gmall/dwd/dwd_dim_user_info_his_tmp/'
tblproperties ("parquet.compression"="lzo");

-- 导入数据
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table dwd_dim_user_info_his_tmp
select * from
(
    select
        id,
        name,
        birthday,
        gender,
        email,
        user_level,
        create_time,
        operate_time,
        '2020-06-23' start_date,
        '9999-99-99' end_date
    from ods_user_info where dt='2020-06-23'

    union all
    select
        uh.id,
        uh.name,
        uh.birthday,
        uh.gender,
        uh.email,
        uh.user_level,
        uh.create_time,
        uh.operate_time,
        uh.start_date,
        if(ui.id is not null  and uh.end_date='9999-99-99', date_add(ui.dt,-1), uh.end_date) end_date
    from dwd_dim_user_info_his uh left join
    (
        select
            *
        from ods_user_info
        where dt='2020-06-23'
    ) ui on uh.id=ui.id
)his
order by his.id, start_date;

-- 导入数据
insert overwrite table dwd_dim_user_info_his select * from dwd_dim_user_info_his_tmp;



-- 日期转换为时间戳
select unix_timestamp('2021-06-22','yyyy-MM-dd');
-- 将时间戳转换为字符串
select from_unixtime(1624320000,'yyyy-MM-dd');
~~~

### 事实表的区别

![1646031002924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/28/145004-198542.png)