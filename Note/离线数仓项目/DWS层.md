## DWS层

dwd层建模是以**业务为驱动**，有什么业务线就建立什么事实表，业务和那些维度有关，就和维度关联即可，维度建模不需要考虑需求。

dws和dwt层数据宽表层，建模的时候有需求驱动，再dws或dwt层建立宽表的时候，我们是根据维度建立宽表，因为我们后续再做统计分析的时候，更多的时候是根据维度进行统计聚合，分组字段往往是维度id。

### 业务术语

#### **用户**

用户以设备为判断标准，在移动统计中，每个独立设备认为是一个独立用户。Android系统根据IMEI号，IOS系统根据OpenUDID来标识一个独立用户，每部手机一个用户，我们以设备id为标准。因为如果用户不登陆，我们无法获取用户的id，这个也可以表示访客，也就是用户没有登录，但是访问了网站，所以可以用来记录访客数量。

#### **新增用户**

首次联网使用应用的用户。如果一个用户首次打开某APP，那这个用户定义为新增用户；卸载再安装的设备，不会被算作一次新增。新增用户包括日新增用户、周新增用户、月新增用户。

#### **活跃用户**

打开应用的用户即为活跃用户，不考虑用户的使用情况。每天一台设备打开多次会被计为一个活跃用户，有日活跃，周活跃，月活跃。

#### **周（月）活跃用户**

某个自然周（月）内启动过应用的用户，该周（月）内的多次启动只记一个活跃用户。

#### **月活跃率**

月活跃用户与截止到该月累计的用户总和之间的比例。

#### **沉默用户**

用户仅在安装当天（次日）启动一次，后续时间无再启动行为。该指标可以反映新增用户质量和用户与APP的匹配程度。

#### **版本分布**

不同版本的周内各天新增用户数，活跃用户数和启动次数。利于判断APP各个版本之间的优劣和用户行为习惯。

#### **本周回流用户**

上周未启动过应用，本周启动了应用的用户。也就是一段时间没有使用产品，但是一段时间之后，又开始使用。

#### **连续n周活跃用户**

连续n周，每周至少启动一次。

#### **忠诚用户**

连续活跃5周以上的用户

#### **连续活跃用户**

连续2周及以上活跃的用户

#### **近期流失用户**

连续n（2<= n <= 4）周没有启动应用的用户。（第n+1周没有启动过）

#### **留存用户**

某段时间内的新增用户，经过一段时间后，仍然使用应用的被认作是留存用户；这部分用户占当时新增用户的比例即是留存率。

例如，5月份新增用户200，这200人在6月份启动过应用的有100人，7月份启动过应用的有80人，8月份启动过应用的有50人；则5月份新增用户一个月后的留存率是50%，二个月后的留存率是40%，三个月后的留存率是25%。

#### **用户新鲜度**

每天启动应用的新老用户比例，即新增用户数占活跃用户数的比例。

#### **单次使用时长**

每次启动使用的时间长度。

#### **日使用时长**

累计一天内的使用时间长度。

#### **启动次数计算标准**

IOS平台应用退到后台就算一次独立的启动；Android平台我们规定，两次启动之间的间隔小于30秒，被计算一次启动。用户在使用过程中，若因收发短信或接电话等退出应用30秒又再次返回应用中，那这两次行为应该是延续而非独立的，所以可以被算作一次使用行为，即一次启动。业内大多使用30秒这个标准，但用户还是可以自定义此时间间隔。

再dws中和dwt层的主题宽表对应的是dwd层的维度，一个维度对应一个主题。

### DWS层宽表统计

#### 每日设备行为

每日设备行为，主要按照设备id统计，但是并没有设备维度表。

设备信息直接融入到宽表当中。

```sql
drop table if exists dws_uv_detail_daycount;
create external table dws_uv_detail_daycount
(
    `mid_id`      string COMMENT '设备id',
    `brand`       string COMMENT '手机品牌',
    `model`       string COMMENT '手机型号',
    `login_count` bigint COMMENT '活跃次数',
    `page_stats`  array<struct<page_id:string,page_count:bigint>> COMMENT '页面访问统计'
) COMMENT '每日设备行为表'
partitioned by(dt string)
stored as parquet
location '/warehouse/gmall/dws/dws_uv_detail_daycount'
tblproperties ("parquet.compression"="lzo");
```

每一行数据代表一个设备累计的一天的行为。

汇总值：活跃次数和页面访问统计。

每一天每一个分区存储的是当天活跃的设备。

活跃次数来自于启动日志表。

页面访问统计来自于页面日志表。

> hive中只要进行分组，那么下面可以选择的字段有以下暗中情况：分组的字段，聚合函数聚合的字段，常量值三种。

dwd_pag_log中的数据，一行数据表示一条浏览记录。我们需要求出每一个设备浏览每一个页面的次数。所以首先按照设备id+page id进行分组。

#### 每日会员行为

再dws层是每日行为，以天为单位进行聚合操作。

再dwt层是累计值。

```sql
drop table if exists dws_user_action_daycount;
create external table dws_user_action_daycount
(   
    user_id string comment '用户 id',
    login_count bigint comment '登录次数',
    cart_count bigint comment '加入购物车次数',
    order_count bigint comment '下单次数',
    order_amount    decimal(16,2)  comment '下单金额',
    payment_count   bigint      comment '支付次数',
    payment_amount  decimal(16,2) comment '支付金额',
    order_detail_stats array<struct<sku_id:string,sku_num:bigint,order_count:bigint,order_amount:decimal(20,2)>> comment '下单明细统计'
) COMMENT '每日会员行为'
PARTITIONED BY (`dt` string)
stored as parquet
location '/warehouse/gmall/dws/dws_user_action_daycount/'
tblproperties ("parquet.compression"="lzo");
```

表中的一条数据表示一个用户再当天的所有行为的聚合值，一个用户一行数据，唯一。

表中所有字段是和维度表user相关的事实表的度量值字段。

**字段数据来源：**

用户id:

登录次数：来自于启动日志表，再dwd层的启动日志中，每一行代表一个启动记录，所以对数据按照用户id进行分组，就可以得到每一个用户的登录次数。

加入购物车次数：去action_log中获取数据。，因为没有事实表可以计算该参数。先把加购物车的数据过滤出来，然后按照用户id进行分组count()即可。

下单次数：来自于订单表

下单金额：订单表中的最终下单金额。

支付次数：来自于支付事实表

支付金额：来自于支付事实表

下单明细统计：一个用户每天可能下多个订单，订单中多个商品，所以字段中一个结构体表示一个商品，数组中存在多个商品。

> 表按照天进行分区，分区中存储的是当天活跃用户的各种行为。

**求登录次数**

按照userId进行分区，这个值可能是null,也就是有的用户没有登录就直接使用了，如果直接按照userid进行分区，那么所有null都会分到一个分区中，可能产生数据倾斜。所以直接过滤掉null。

#### 每日商品行为

每一条数据表示一个关于商品的行为，表中的字段全部是关于商品的度量值。

向表中插入数据需要join几个子查询，我们可以使用别的方法进行插入，提高效率，但是中优化仅仅适用于表中的所有字段全部都是数字类型的表。

我们可以首先对子查询中的字段补齐，也就是原始表中有几个字段，我们就跟着补齐为几个字段，如果一个子查询中没有计算的字段，我们可以使用0代替，这样所有子查询中字段的个数是一样的，我们就可以使用union进行上下合并，然后对这张合并过的大表进行分组操作，然后对每一个字段累加，和我们使用join的效果是一样的。

#### 每日活动行为

每一行数据表示一个活动当天的汇总行为，也就是和活动维度表相关的事实。

```sql
drop table if exists dws_activity_info_daycount;
create external table dws_activity_info_daycount(
    `id` string COMMENT '编号',
    `activity_name` string  COMMENT '活动名称',
    `activity_type` string  COMMENT '活动类型',
    `start_time` string  COMMENT '开始时间',
    `end_time` string  COMMENT '结束时间',
    `create_time` string  COMMENT '创建时间',
    `display_count` bigint COMMENT '曝光次数',
    `order_count` bigint COMMENT '下单次数',
    `order_amount` decimal(20,2) COMMENT '下单金额',
    `payment_count` bigint COMMENT '支付次数',
    `payment_amount` decimal(20,2) COMMENT '支付金额'
) COMMENT '每日活动统计'
PARTITIONED BY (`dt` string)
stored as parquet
location '/warehouse/gmall/dws/dws_activity_info_daycount/'
tblproperties ("parquet.compression"="lzo");
```

字段来自活动维度表中的维度字段。

曝光次数也是和活动相关的一个事实。

下单次数和下单金额是参与该活动的商品的下单次数和下单金额。

支付次数和支付金额：参与该活动并且支付的订单。

曝光次数来自于曝光日志。

下单次数和下单金额来自于订单事实表。

支付次数和支付金额可以根据订单表得出，统计支付了的订单，然后再统计支付金额和次数。

### DWT层宽表数据

累计值，dws是按照天进行汇总的，而dwt是对历史数据的累计。

#### 设备主题宽表

```sql
drop table if exists dwt_uv_topic;
create external table dwt_uv_topic
(
    `mid_id` string comment '设备id',
    `brand` string comment '手机品牌',
    `model` string comment '手机型号',
    `login_date_first` string  comment '首次活跃时间',
    `login_date_last` string  comment '末次活跃时间',
    `login_day_count` bigint comment '当日活跃次数',
    `login_count` bigint comment '累积活跃天数'
) COMMENT '设备主题宽表'
stored as parquet
location '/warehouse/gmall/dwt/dwt_uv_topic'
tblproperties ("parquet.compression"="lzo");
```

设备主题宽表不是一个分区表，也是记录累计值，记录的是全量数据信息，指的是包含所有的设备id。一行数据就是一个访客记录或者设备。

前三个字段是设备信息。

累计值是：

- 首次活跃时间：
- 末次活跃时间：
- 当日活跃次数：
- 累计活跃次数：

更新表里面的数据思路：首先获取到表中的所有历史数据，然后再获取到今天的活跃设备，然后进行一个全外连接，更新历史数据中活跃的设备和新增设备即可。

数据来自于前端的埋点数据，如果前端没有进行埋点的话，就无法获取数据进行初始化操作，主要会影响首次活跃时间字段，累计活跃天数如果没有历史数据，无法计算，我们可以从数仓搭建起开始那天计算。

首次活跃时间可以这样做，和数仓中历史数据进行对比，如果没有关联上，就算作新增数据。

#### 会员主题宽表

宽表字段怎么来？维度关联的事实表度量值+开头、结尾+累积+累积一个时间段。

**建表语句**

```sql
drop table if exists dwt_user_topic;
create external table dwt_user_topic
(
    user_id string  comment '用户id',
    login_date_first string  comment '首次登录时间',
    login_date_last string  comment '末次登录时间',
    login_count bigint comment '累积登录天数',
    login_last_30d_count bigint comment '最近30日登录天数',
    order_date_first string  comment '首次下单时间',
    order_date_last string  comment '末次下单时间',
    order_count bigint comment '累积下单次数',
    order_amount decimal(16,2) comment '累积下单金额',
    order_last_30d_count bigint comment '最近30日下单次数',
    order_last_30d_amount bigint comment '最近30日下单金额',
    payment_date_first string  comment '首次支付时间',
    payment_date_last string  comment '末次支付时间',
    payment_count decimal(16,2) comment '累积支付次数',
    payment_amount decimal(16,2) comment '累积支付金额',
    payment_last_30d_count decimal(16,2) comment '最近30日支付次数',
    payment_last_30d_amount decimal(16,2) comment '最近30日支付金额'
)COMMENT '会员主题宽表'
stored as parquet
location '/warehouse/gmall/dwt/dwt_user_topic/'
tblproperties ("parquet.compression"="lzo");
```

dwt层的数据是一个宽表，一行数据是一个用户，并且表中的字段是对用户各种操作的累计值，比如累计登录天数，累计30天登录天数。

涉及的表有：dwd_start_log，dws层订单事实表，dws用户行为表。

这张表没有进行分区，因为记录的都是历史数据的聚合，每天需要进行依次聚合，然后再写入原表中，所以表每一天都需要更新数据。

表中的字段计算方法来自于不同的表，比如登录日志，每一天都会有增加，如果我们每一天都进行计算，随着时间增长，数据量也会大大的增加。比如目前有100个分区，计算了依次，第二天增加3个分区，还要对103个分区重新计算，计算量大大增加不说，还有很多重复计算。

所以可以保存前面100个分区计算的结果，然后比如今天又增加三个分区，那么就计算者三个分区，然后去更新表中的值。所以最终的做法是先从表中查询记录，然后根据今天的值去更新操作。

> dwt层所有的表都是全量表，比如user_topic就应该包含所有的user。需要对所有的dwt层表进行一个初始化。

> 全量表一般不用做初始化，事实表一般也不用做初始化，拉链表第一天使用需要做初始化，因为拉链表记录的是新增和变化的数据。dwt层的表第一次使用需要做初始化。

需要拿到每一天的活跃用户，然后和原始表中的数据进行join比较操作，join之后，数据又join上的部分，也有没有join上的部分，代表的数据是：老用户并且今天活跃，老用户今天没有活跃，新增用户。

- 首次登录时间：对于老用户，这个字段不哟个改变，对于新用户，记录为今天的时间。
- 末次登录时间：对于老用户并且今天活跃和新用户的末次登录时间，修改为今天即可。
- 累计登录天数：原始用户并且没有活跃，不改变，原始用户并且今天活跃和新增用户，累计登录天数+1；
- 累计30天登录天数：需要获取30天之前的登录时间，如果30天之前登录了，数据需要-1，然后今天登录了+1，简单粗暴的就是直接重新计算最近30天登录数量。

其他字段更新思路一致。
![20211222091219](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211222091219.png)

#### 商品主题宽表

商品主题宽表中需要有全量的sku。

表的初始化，我们根据字段，去原始表中获取所有的数据，然后使用子查询全量导入查询。

表中每一天的更新逻辑，首先获取所有的历史数据，然后获取最近一天商品的各种行为，然后通过join两张表，更新累计值。

#### 活动主题宽表

活动主题宽表拥有全量的维度数据。

一行数据表示一个活动的各种累计值，每个活动一行数据。字段来自活动维度表的字段，，当日的字段，再dws层可以直接统计，累计等字段，更新思路是，先获取表中的历史数据，然后获取到当天的各种累计值，然后相加。