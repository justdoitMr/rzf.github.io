### ADS层

ADS层数据也分为若干个主题，但是ADS层不涉及数据建模，建表根据我们的需求决定，所以也不需要考虑表中都需要什么字段，一切根据需求而定，再dws和dwt层建表对应我们的维度表字段。

- 设备主题，统计流量的一些指标，默认都以设备id作为唯一的标识用户。
- 会员主题
- 商品主题
- 营销主题
- 地区主题

#### 设备主题

##### 活跃设备数（日、周、月）

需求定义：默认都是以设备的id作为统一的标示

> 只要今天用户登录了，那么就算作一个活跃用户。

日活：当日活跃的设备数，一天一个数，表示当天活跃设备数量。每天计算一次。如果从dwd层获取数据，dwd层每一行代表一个启动日志，所以需要去重操作，从dws层也可以，已经去重了。

周活：当周活跃的设备数，当周一计算上周的周活。可以从dwd获取数据，需要去重，dws层也可以，但是需要去重，因为dws是对于天去重，但是我们需要计算一周，所以需要去重。

月活：当月活跃的设备数，下个月的1号计算上一个月的月活。

> 以上我们的周，月都是指自然周和月，而不是指最近7天或者最近一个月。

再本项目中，对日活，周活和月活的计算进行统一，都是每天计算一次，这样的话，计算周活，本周一计算上周的周活，本周二计算本周的周一日活，周三计算本周一二的周活。直到下周一才可以计算完本周的周活，

这个表中不需要进行分区，因为每天只需要向表中插入一条数据。为什么没有进行parquet列式存储呢？

因为ads层的表式最终的结果数据了，我们后期直接使用sqoop将全表导入mysql中，二parquet使用场景式我们需要查询若干个列的时候，效率高，我们导入的式全表，所以没必要使用列式存储。也没必要进行压缩，因为数据一天插入一条，一年300多条，数据量少。

计算日活：从dwt层的dwt_uv_topic中根据login_data_last判断，如果末次登录时间等于今天，说明今天活跃。

如果我们向看历史上某一天是否式活跃，就不能使用上面方法判断，因为dwt层表中的最后登录时间会一致的进行更新。也就是说从dwt层计算日活的时候，只能计算最新一天的日活，历史上某一天无法计算。

如果从dws层计算，我们只需要找到最新一天的分区数据，这个分区中存储的就是当天的活跃设备，如果从dwd层计算，数据量太大。

计算周活：首先找到本周一和本周日时间，然后对数据进行过滤。也是从dws层计算，也可以从dwt层计算，只要末次登录时间再本周一和本周日时间之间，就可以计算，算周活从dwt计算。

计算月活：主要对比从dwt层计算还式从dws层计算，主要是看我们数据量的多少，尽量从数据量少的地方计算。

本项目中，三个指标都是从dwt层数据计算，可以放在一起计算，只是过滤条件不同。

##### 每日新增设备

在dwt层的dwt_uv_topic表中有一个首次登录日期，可以用于计算。

> 通用的计算新增的方法，首先获取当天的活跃用户，然后获取表中的所有历史数据，做一个全外连接，全外连接中的数据分为三部分，历史用户没有活跃，历史用户活跃，新增用户，也可以计算新增。

```sql
drop table if exists ads_new_mid_count;
create external table ads_new_mid_count
(
    `create_date`     string comment '创建时间' ,
    `new_mid_count`   BIGINT comment '新增设备数量' 
)  COMMENT '每日新增设备数量'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_new_mid_count/';
```

##### 留存率

![20211223133708](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211223133708.png)

在这里我们计算每一天的三日留存率。

```sql
drop table if exists ads_user_retention_day_rate;
create external table ads_user_retention_day_rate 
(
     `stat_date`          string comment '统计日期',
     `create_date`       string  comment '设备新增日期',
     `retention_day`     int comment '截止当前日期留存天数',
     `retention_count`    bigint comment  '留存数量',
     `new_mid_count`     bigint comment '设备新增数量',
     `retention_ratio`   decimal(16,2) comment '留存率'
)  COMMENT '留存率'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_user_retention_day_rate/';
```

留存率=留存数量/新增数量

统计日期指的是计算日期，不重要，值是计算日期的前一天。

在这里标识一个留存是用：设备新增日期和截至当前日期留存天数。

表中的一行数据，表示一个留存。

计算留存，我们必须明确是在那一天的几日留存，

假设现在计算2020-06-14号的1日，2日，3日留存率：

**1日留存率：**

6-14号的新增用户，14号的留存用户出自15号的活跃用户，所以需要15号的活跃用户。

**2日留存率**

需要14号的新增用户，需要16号的活跃用户

**3日留存率**

需要14号的新增用户，17号的活跃用户。

但是14号1日留存率在16号才能计算，2日留存率在17号能计算，3日留存在18号可以计算。所以计算每天的3个留存指标，并不是在某一天就直接可以计算出来。

在18号我们计算的内容有：

- 14号的三日留存率
- 15号的2日留存率
- 16号的1日留存率

可以发现，这三个指标都需要17号的活跃人数。

![20211223163356](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211223163356.png)





##### 沉默用户数

需求定义：

沉默用户：只在安装当天启动过，且启动时间是在7天前

第一次登录的时间等于最后一次的登录时间，

dwt_uv_topic中有一个累计活跃天数字段，如果字段为1，标识指在安装当天启动过，之后再也没有登陆过。还需要保证登录时间在7天之前

##### 本周回流用户

需求定义：

本周回流用户：上周未活跃，本周活跃的设备，且不是本周新增设备

```sql
drop table if exists ads_back_count;
create external table ads_back_count( 
    `dt` string COMMENT '统计日期',
    `wk_dt` string COMMENT '统计日期所在周',
    `wastage_count` bigint COMMENT '回流设备数'
) COMMENT '本周回流用户数'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_back_count';
```

应该是以周为计算单位，因为本周过完后，才可以统计回流了几个用户。但是本项目中为了统一，也是按照天计算。

本周回流用户肯定出自于本周活跃用户，然后从种去掉本周新增用户和上周活跃用户，剩余的数据就是本周回流用户。



##### 流式用户数

需求定义：

流失用户：最近7天未活跃的设备，也就是最后一次活跃时间在7天前。



##### 最近连续三周活跃用户数

```sql
drop table if exists ads_continuity_wk_count;
create external table ads_continuity_wk_count( 
    `dt` string COMMENT '统计日期,一般用结束周周日日期,如果每天计算一次,可用当天日期',
    `wk_dt` string COMMENT '持续时间',
    `continuity_count` bigint COMMENT '活跃用户数'
) COMMENT '最近连续三周活跃用户数'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_continuity_wk_count';
```

**找共有的部分，最简单的就是将子查询进行内连接，返回的结果就是多个子查询的公共部分。**

如果使用login_data_last求，只能够求出当天，当周或者当月的，不能够求出历史数据。

**第二种思路**

先求出三个周的活跃用户，但是求出来之后并不是进行join操作，而是使用union all操作，之后形成了一个大的虚表，不会去重，然后从虚表种查询数据，按照设备的id进行分组，这样相同的设备id分为同一组，然后对每一组进行count(`*`)操作， 只要count(*)的结果是3，那么就表明连续三周活跃。

为什么可以这样做？

因为我们在分别求这三周种活跃用户的时候，分别按照设备的mid进行分组操作了，每一个设备id如果活跃了就只有一条数据，也就是说一组种一个设备id只能出现一次。然后我们进行union alla操作，然后再次分组，如果一组种个数是3，那么说明连续三周都活跃。这样做不会涉及多个表join操作，效率高。

##### 最近七天内连续三天活跃用户数

```sql
drop table if exists ads_continuity_uv_count;
create external table ads_continuity_uv_count( 
    `dt` string COMMENT '统计日期',
    `wk_dt` string COMMENT '最近7天日期',
    `continuity_count` bigint
) COMMENT '最近七天内连续三天活跃用户数'
row format delimited fields terminated by '\t'
location '/warehouse/gmall/ads/ads_continuity_uv_count';
```

wk_dt:使用七天中的第一天的日期拼接第七天的日期。

###### 查找连续的数据

首先获取每一个用户7天内的活跃记录，一个人一天的登录记录只有一个，所以需要去重。

![1640308302643](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/091143-587366.png)

找连续三天的，可以先找到中间一行数据，然后和前一条数据和后一条数据相比，也可以找到每一个日期下边的两个日期进行比较，因为我们的日期是经过排序的，并且经过去重的，如果第一个日期和第三个日期相差2，那么就一定是连续的。

![1640308780768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/091942-243745.png)

**文档思路**

![1640310103843](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/094145-112550.png)

#### 会员主题

##### 会员信息

###### 建表

![1640310556506](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/094917-677400.png)

- 活跃会员数：指的是当天活跃的用户
- 新增会员指的是新增用户
- 新增消费会员：统计的当天，如果用户是第一次支付，那么就算新增消费会员。
- 总付费会员数：统计总的付过费的用户一共有多少。
- 总会员数：统计到目前为止一共有多少个用户。
- 会员活跃率：活跃会员/总会员数
- 会员付费率：总付费会员数/总会员数
- 会员新鲜度：当日的新增/当日的活跃



##### 漏斗分析

统计“浏览首页->浏览商品详情页->加入购物车->下单->支付”的转化率

思路：统计各个行为的人数，然后计算比值。

> 统计完整的购物过程中，每一个阶段的人数。比如浏览首页的人数，浏览商品详情页的人数等等。
>
> 在这里统计的是每一个行为的人数，所以不需要统计个人信息。
>
> 浏览首页和浏览商品详情页可以使用设备id和用户id进行标识，但是加购物车和下单，支付需要使用用户id标识一个人。

###### 建表

![1640311324380](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/100204-359430.png)

- 浏览首页人数，浏览详情页人数都可以从dwd_page_log求出，dws的设备主题表中有page_status，记录哪一个设备浏览了哪一个页面，也可以统计计算。
- 当天加过购物车人数：可以从动作日志中求，
- 当天下单人数：去订单事实表中计算，记录了当天所有的订单
- 当天支付人数：去支付事实表中，获取当天分区数据，然后按照用户id分组去重统计人数
  - 上面三个字段也可以从dws_user_action_daycount计算，根据cart_count,order_count,payment_count计算，如果大于0，说明今天进行了加购物车，下订单和支付操作。本项目中从dws层计算，因为dws层数据相对较小，然后从一张表中读取一次数据既可以计算，性能好。

