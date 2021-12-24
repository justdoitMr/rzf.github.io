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

##### 创建表

![1640324438938](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/134040-412735.png)

> 每一行数据代表一个设备累计的一天的行为。
>
> 汇总值：活跃次数和页面访问统计，都是统计一天的行为。
>
> 每一天每一个分区存储的是当天活跃的设备，分区内部的数据已经按照设备的id进行去重，每一行标识一个设备当天的行为。

活跃次数来自于启动日志表：dwd_start_log

页面访问统计来自于页面日志表：dwd_page_log

> hive中只要进行分组，那么下面可以选择的字段有以下三中情况：分组的字段，聚合函数聚合的字段，常量值三种。

dwd_page_log中的数据，一行数据表示一条浏览记录。我们需要求出每一个设备浏览每一个页面的次数。所以首先按照设备id+page id进行分组。

##### 插入数据

![1640324831579](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/134712-894069.png)

##### 导入数据

- dwd_start_log
- dwd_page_log

**计算前四个字段**

~~~sql
-- 计算前四个字段
-- hive中只要进行分组，那么下面可以选择的字段有以下三中情况：分组的字段，聚合函数聚合的字段，常量值三种。
select
    mid_id,
    brand,
    model,
    count(*) login_count --当天活跃的次数
from
dwd_start_log
where dt='2021-06-22'
group by mid_id, brand, model;
~~~

**计算页面访问统计**

~~~ sql
-- 封装为结构体
select 
    mid_id,
    brand,
    model,
    collect_list(named_struct('page_id',page_id,'page_count',page_count)) page_status  --封装结构体
from
(
    select
        mid_id,
        brand,
        model,
        page_id,
        count(*) as page_count --每一个页面的访问次数
    from dwd_page_log
    where dt='2021-06-22'
    group by mid_id,brand,model,page_id
)tmp
group by mid_id,brand,model;
~~~

**完整sql**

~~~sql
-- 计算前四个字段
-- hive中只要进行分组，那么下面可以选择的字段有以下三中情况：分组的字段，聚合函数聚合的字段，常量值三种。
select
    mid_id,
    brand,
    model,
    count(*) login_count --当天活跃的次数
from
dwd_start_log
where dt='2021-06-22'
group by mid_id, brand, model;

-- 封装为结构体
select 
    mid_id,
    brand,
    model,
    collect_list(named_struct('page_id',page_id,'page_count',page_count)) page_status  --封装结构体
from
(
    select
        mid_id,
        brand,
        model,
        page_id,
        count(*) as page_count --每一个页面的访问次数
    from dwd_page_log
    where dt='2021-06-22'
    group by mid_id,brand,model,page_id
)tmp
group by mid_id,brand,model;

--对上面的sql进行拼接
with
tmp_start as
(
    select
        mid_id,
        brand,
        model,
        count(*) login_count
    from dwd_start_log
    where dt='2020-06-22'
    group by mid_id,brand,model
),
tmp_page as
(
    select
        mid_id,
        brand,
        model,
        collect_set(named_struct('page_id',page_id,'page_count',page_count)) page_status
    from
    (
        select
            mid_id,
            brand,
            model,
            page_id,
            count(*) page_count
        from dwd_page_log
        where dt='2020-06-22'
        group by mid_id,brand,model,page_id
    )tmp
    group by mid_id,brand,model
)

insert overwrite table dws_uv_detail_daycount partition(dt='2020-06-22')
select
    nvl(tmp_start.mid_id,tmp_page.mid_id),
    nvl(tmp_start.brand,tmp_page.brand),
    nvl(tmp_start.model,tmp_page.model),
    nvl(tmp_start,login_count,0),
    tmp_page.page_status
from tmp_start
full outer join tmp_page
on tmp_start.mid_id=tmp_page.mid_id
and tmp_start.brand=tmp_page.brand
and tmp_start.model=tmp_page.model;
~~~

#### 每日会员行为

在dws层是每日行为，以天为单位进行聚合操作，在dwt层是累计值。

##### 建表

![1640328604052](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/145004-736877.png)

表中的一条数据表示一个用户再当天的所有行为的聚合值，一个用户一行数据，唯一。

表中所有字段是和维度表user相关的事实表的度量值字段。

**字段数据来源：**

**用户id**:

**登录次数**：来自于**启动日志表**，再dwd层的启动日志中，每一行代表一个启动记录，所以对数据按照用户id进行分组，就可以得到每一个用户的登录次数。

**加入购物车次数**：去action_log中获取数据。，因为没有事实表可以计算该参数。先把加购物车的数据过滤出来，然后按照用户id进行分组count()即可。

**下单次数**：来自于订单表

**下单金额**：订单表中的最终下单金额。

**支付次数**：来自于支付事实表

**支付金额**：来自于支付事实表

**下单明细统计**：一个用户每天可能下多个订单，订单中多个商品，所以字段中一个结构体表示一个商品，数组中存在多个商品。

> 表按照天进行分区，分区中存储的是当天活跃用户的各种行为。

##### 插入数据

![1640329596513](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/150637-846718.png)

- dwd_start_log
- dwd_action_log
- dwd_fact_order_info
- dwd_fact_payment_info
- dwd_fact_order_detail

**求登录次数**

按照userId进行分区，这个值可能是null,也就是有的用户没有登录就直接使用了，如果直接按照userid进行分区，那么所有null都会分到一个分区中，可能产生数据倾斜。所以直接过滤掉null。

~~~ sql
 select
		user_id,
		count(*) as login_count
from dwd_start_log
 where dt='2021-06-22' and user_id is not null
group by user_id
~~~

**求加购物车的次数**

~~~sql
-- 2 求每一个用户加入购物车的次数
select
    user_id,
    count(*) as card_count
from dwd_action_log
where dt='2021-06-22' and action_id='cart_add' and user_id is not null
group by user_id;
~~~

**求下单次数和下单金额**

~~~ sql
-- 3 求下单次数和下单金额，求的是每一个用户的下单次数和下单金额
-- 这个不需要对userid进行去重操作，因为用户只有登录了才可以进行下单操作

select
        user_id,
        count(*) as order_count,
        sum(final_total_amount) as order_amount -- 使用最终金额
from dwd_fact_order_info
where dt='2021-06-22'
group by user_id;
~~~

**求支付次数和支付金额**

~~~ sql
-- 4 求支付次数和支付金额
select
    user_id,
    count(*) as payment_count,
    sum(payment_amount) as payment_amount
from dwd_fact_payment_info
where dt='2021-06-22'
group by user_id;
~~~

**求每一个用户的订单详情**

~~~ sql
--5 求最后一个字段，每一个用户一天当中所购买所有的sku的id，件数，金额
-- 一行数据代表一个用户购买的一个sku
select
        user_id,
        sku_id,
        sum(sku_num) sku_num,
        count(*) order_count,
        sum(final_amount_d) order_amount
from dwd_fact_order_detail
where dt='2021-06-22'
group by user_id,sku_id;

-- 把数据封装到结构体中
-- named_struct:把多个列封装为一个结构体
select
    user_id,
    named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'order_amount',order_amount)
from
(
    select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            sum(final_amount_d) order_amount
    from dwd_fact_order_detail
    where dt='2021-06-22'
    group by user_id,sku_id
)tmp;

-- 把结构体放到数组当中，按照user_id进行分组操作
-- 没有去重要求，就使用collect_list
select
    user_id,
    collect_list(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'order_amount',order_amount)) as order_details_status
from
(
    select
            user_id,
            sku_id,
            sum(sku_num) sku_num,
            count(*) order_count,
            sum(final_amount_d) order_amount
    from dwd_fact_order_detail
    where dt='2021-06-22'
    group by user_id,sku_id
)tmp
group by user_id;
~~~

**完整sql**

~~~ sql
-- 新的语法结构
-- with 子查询表别名 as (子查询),但是这种语法没有优化效果
-- 多个表明之间用一个with即可
with
tmp_login as
(
    select
        user_id,
        count(*) as login_count
    from dwd_start_log
    where dt='2021-06-22' and user_id is not null
    group by user_id
),
tmp_cart as
(
    select
        user_id,
        count(*) as card_count
    from dwd_action_log
    where dt='2021-06-22' and action_id='cart_add' and user_id is not null
    group by user_id
),
tmp_order as
(
    select
        user_id,
        count(*) as order_count,
        sum(final_total_amount) as order_amount
    from dwd_fact_order_info
    where dt='2021-06-22'
    group by user_id
),
tmp_payment as
(
    select
        user_id,
        count(*) as payment_count,
        sum(payment_amount) as payment_amount
    from dwd_fact_payment_info
    where dt='2021-06-22'
    group by user_id
),
tmp_detail as
(
    select
        user_id,
        collect_list(named_struct('sku_id',sku_id,'sku_num',sku_num,'order_count',order_count,'order_amount',order_amount)) as order_details_status
    from
    (
        select
                user_id,
                sku_id,
                sum(sku_num) sku_num,
                count(*) order_count,
                sum(final_amount_d) order_amount
        from dwd_fact_order_detail
        where dt='2021-06-22'
        group by user_id,sku_id
    )tmp
    group by user_id
)

-- 拼接所有的字段，把数据放入表中

-- 把数据插入表中
insert overwrite table dws_user_action_daycount partition (dt='2021-06-22')
select
    coalesce(tmp_login.user_id,tmp_cart.user_id,tmp_order.user_id,tmp_payment.user_id,tmp_detail.user_id),
       nvl(login_count,0),
       nvl(card_count,0),
       nvl(order_count,0),
       nvl(order_amount,0.0),
       nvl(payment_count,0),
       nvl(payment_amount,0.0),
       order_details_status
from tmp_login
full outer join tmp_cart on tmp_login.user_id = tmp_cart.user_id
full outer join tmp_order on nvl(tmp_login.user_id,tmp_cart.user_id)=tmp_order.user_id
full outer join tmp_payment on coalesce(tmp_login.user_id,tmp_cart.user_id,tmp_order.user_id) = tmp_payment.user_id
full outer join tmp_detail on coalesce(tmp_login.user_id,tmp_cart.user_id,tmp_order.user_id,tmp_payment.user_id) = tmp_detail.user_id
;
~~~

#### 每日商品行为

每一条数据表示一个关于商品的行为，表中的字段全部是关于商品的度量值。

向表中插入数据需要join几个子查询，我们可以使用别的方法进行插入，提高效率，但是中优化仅仅适用于表中的所有字段全部都是数字类型的表。

我们可以首先对子查询中的字段补齐，也就是原始表中有几个字段，我们就跟着补齐为几个字段，如果一个子查询中没有计算的字段，我们可以使用0代替，这样所有子查询中字段的个数是一样的，我们就可以使用union进行上下合并，然后对这张合并过的大表进行分组操作，然后对每一个字段累加，和我们使用join的效果是一样的。

##### 创建表

![1640330803250](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/152644-988623.png)

##### 导入数据

注意：如果是23点59下单，支付日期跨天。需要从订单详情里面取出支付时间是今天，且订单时间是昨天或者今天的订单。

- dwd_fact_order_detail
- dwd_fact_payment_info
- dwd_fact_order_refund_info
- dwd_action_log

**计算2，3，4个字段**

~~~ sql
-- 1 获取第234个字段的信息,统计每一件商品的被下单次数，被下单件数和下单金额，需要按照sku_id进行分组操作
-- 求次数，每一行就是一次下单
--下面sql可以求出今天被支付的次数，件数和金额
select
    sku_id,
    count(*) order_count,
    sum(sku_num) order_num,
    sum(final_amount_d) order_amount
from dwd_fact_order_detail
where (dt='2021-06-22'
or dt=date_add('2021-06-22',-1))
and order_id in
(
    select
        order_id
    from dwd_fact_payment_info
    where dt='2021-06-22'
)
group by sku_id;
~~~

**求退款的件数和退款金额**

~~~sql
-- 求退款次数和退款的件数，按照sku_id进行分组，然后分组求件数和金额
select
    sku_id,
    count(*),
    sum(refund_num)
from dwd_fact_order_refund_info
where dt='2021-06-22'
group by sku_id;
~~~

**求加购物车次数**

~~~sql
-- 求加购物车的次数

select
    item sku_id,
    count(*) cart_count
from dwd_action_log
where dt='2021-06-22'
and action_id='cart_add'
group by item;
~~~

**计算收藏**

~~~sql
--计算收藏
select
    item sku_id,
    count(*) favor_count
from dwd_action_log
where dt='2021-06-22'
and action_id='favor_add'
group by item;
~~~

**优化**

~~~sql
-- 合并上面两条sql语句
--求加购物车的次数和求收藏的次数
select
    item sku_id,
    sum(if(action_id='cart_add',1,0)),
    sum(if(action_id='favor_add',1,0))
from dwd_action_log
where dt='2021-06-22'
and action_id in('cart_add','favor_add')
group by item;
~~~

**计算评价字段**

~~~sql
-- 评价字段

select
    sum(if(appraise='1201',1,0)),
    sum(if(appraise='1202',1,0)),
    sum(if(appraise='1203',1,0)),
    sum(if(appraise='1204',1,0))
from dwd_fact_comment_info
where dt='2021-06-22'
group by sku_id;
~~~

**union所有子查询结果**

~~~sql
-- union上面的字段
with
tmp_order as
(
    select
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(final_amount_d) order_amount
    from dwd_fact_order_detail
    where dt='2020-06-14'
    group by sku_id
),
tmp_payment as
(
    select
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(final_amount_d) payment_amount
    from dwd_fact_order_detail
    where (dt='2020-06-14'
    or dt=date_add('2020-06-14',-1))
    and order_id in
    (
        select
            id
        from dwd_fact_order_info
        where (dt='2020-06-14'
        or dt=date_add('2020-06-14',-1))
        and date_format(payment_time,'yyyy-MM-dd')='2020-06-14'
    )
    group by sku_id
),
tmp_refund as
(
    select
        sku_id,
        count(*) refund_count,
        sum(refund_num) refund_num,
        sum(refund_amount) refund_amount
    from dwd_fact_order_refund_info
    where dt='2020-06-14'
    group by sku_id
),
tmp_cart as
(
    select
        item sku_id,
        count(*) cart_count
    from dwd_action_log
    where dt='2020-06-14'
    and user_id is not null
    and action_id='cart_add'
    group by item
),tmp_favor as
(
    select
        item sku_id,
        count(*) favor_count
    from dwd_action_log
    where dt='2020-06-14'
    and user_id is not null
    and action_id='favor_add'
    group by item
),
tmp_appraise as
(
select
    sku_id,
    sum(if(appraise='1201',1,0)) appraise_good_count,
    sum(if(appraise='1202',1,0)) appraise_mid_count,
    sum(if(appraise='1203',1,0)) appraise_bad_count,
    sum(if(appraise='1204',1,0)) appraise_default_count
from dwd_fact_comment_info
where dt='2020-06-14'
group by sku_id
)
~~~

##### 优化

- 因为表中的所有字段全是数字类型，使用union可以将所有查询的字段拼接再一起，但是还有更高效的方法
- 将每一个子查询补充其，使其和原来表中的字段个数相同，其余的没有值的字段使用0补充，然后对所有的子查询使用union操作
- 这个时候，每一个子查询的字段个数都一样，可以使用union进行上下拼接，这个大表有很多的行，然后我们对这个大表进行一个子查询，分组聚合操作
- 按照sku_id进行分组聚合操作，然后对每一个字段进行分组求和，这样做的结果和使用全外联结果是一样的，使用全外连接会有null值出现

~~~ sql
insert overwrite table dws_sku_action_daycount partition(dt='2020-06-14')
select
-- 下面是向表中的插入字段
    sku_id,
    sum(order_count),
    sum(order_num),
    sum(order_amount),
    sum(payment_count),
    sum(payment_num),
    sum(payment_amount),
    sum(refund_count),
    sum(refund_num),
    sum(refund_amount),
    sum(cart_count),
    sum(favor_count),
    sum(appraise_good_count),
    sum(appraise_mid_count),
    sum(appraise_bad_count),
    sum(appraise_default_count)
from
(

-- 下面是整体的子查询
    select
        sku_id,
        order_count,
        order_num,
        order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_order
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        payment_count,
        payment_num,
        payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_payment
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        refund_count,
        refund_num,
        refund_amount,
        0 cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_refund
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        cart_count,
        0 favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_cart
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        favor_count,
        0 appraise_good_count,
        0 appraise_mid_count,
        0 appraise_bad_count,
        0 appraise_default_count
    from tmp_favor
    union all
    select
        sku_id,
        0 order_count,
        0 order_num,
        0 order_amount,
        0 payment_count,
        0 payment_num,
        0 payment_amount,
        0 refund_count,
        0 refund_num,
        0 refund_amount,
        0 cart_count,
        0 favor_count,
        appraise_good_count,
        appraise_mid_count,
        appraise_bad_count,
        appraise_default_count
    from tmp_appraise
)tmp
group by sku_id;--最终按照sku_id进行分组操作，然后求和
~~~

#### 每日活动行为

每一行数据表示一个活动当天的汇总行为，也就是和活动维度表相关的事实。

![1640331833511](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/154354-789300.png)

字段来自活动维度表中的维度字段。

曝光次数也是和活动相关的一个事实。

下单次数和下单金额是参与该活动的商品的下单次数和下单金额。

支付次数和支付金额：参与该活动并且支付的订单。

曝光次数来自于曝光日志。

下单次数和下单金额来自于订单事实表。

支付次数和支付金额可以根据订单表得出，统计支付了的订单，然后再统计支付金额和次数。

##### 数据导入

![1640331868306](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/154432-498899.png)

- dwd_fact_order_info
- dwd_dim_activity_info

~~~ sql

with
tmp_op as
(
    select
        activity_id,
        sum(if(date_format(create_time,'yyyy-MM-dd')='2020-06-14',1,0)) order_count,--计算6-14日订单的个数
        sum(if(date_format(create_time,'yyyy-MM-dd')='2020-06-14',final_total_amount,0)) order_amount,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='2020-06-14',1,0)) payment_count,--计算24号支付的订单个数
        sum(if(date_format(payment_time,'yyyy-MM-dd')='2020-06-14',final_total_amount,0)) payment_amount
    from dwd_fact_order_info--是一个累积性快照事实表，里面有每一个订单每一个阶段的时间
    where (dt='2020-06-14' or dt=date_add('2020-06-14',-1))--因为要统计支付订单今天支付的订单可能是昨天下的单
    and activity_id is not null -- 活动id不可以为null，因为我们统计的是参与活动的订单
    group by activity_id  --按照活动订单进行分组，这样的话，参与同一个活动的订单会分到一起，这里数据是两天数据，也就是会将两天内参与同一个活动的订单
	-- 分到一组中
),
tmp_display as
(
    select
        item activity_id,
        count(*) display_count
    from dwd_display_log
    where dt='2020-06-14'
    and item_type='activity_id' --先把活动曝光过滤出来
    group by item --按照曝光对象进行分组
),
tmp_activity as
(
    select -- 获取活动表当天的分区，我们只获取最新数据，因为这张表是每日全量表 
        *
    from dwd_dim_activity_info
    where dt='2020-06-14'
)
insert overwrite table dws_activity_info_daycount partition(dt='2020-06-14')
select
    nvl(tmp_op.activity_id,tmp_display.activity_id),
    tmp_activity.activity_name,
    tmp_activity.activity_type,
    tmp_activity.start_time,
    tmp_activity.end_time,
    tmp_activity.create_time,
    tmp_display.display_count,
    tmp_op.order_count,
    tmp_op.order_amount,
    tmp_op.payment_count,
    tmp_op.payment_amount
from tmp_op
full outer join tmp_display on tmp_op.activity_id=tmp_display.activity_id
left join tmp_activity on nvl(tmp_op.activity_id,tmp_display.activity_id)=tmp_activity.id;
~~~

#### 每日地区统计

##### 创建表

![1640332091300](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/154813-926373.png)

##### 导入数据

- dwd_start_log
- dwd_fact_order_info
- dwd_dim_base_province

~~~ sql
with 
tmp_login as
(
    select
        area_code,
        count(*) login_count
    from dwd_start_log
    where dt='2020-06-14'
    group by area_code
),
tmp_op as
(
    select
        province_id,
        sum(if(date_format(create_time,'yyyy-MM-dd')='2020-06-14',1,0)) order_count,
        sum(if(date_format(create_time,'yyyy-MM-dd')='2020-06-14',final_total_amount,0)) order_amount,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='2020-06-14',1,0)) payment_count,
        sum(if(date_format(payment_time,'yyyy-MM-dd')='2020-06-14',final_total_amount,0)) payment_amount
    from dwd_fact_order_info
    where (dt='2020-06-14' or dt=date_add('2020-06-14',-1))
    group by province_id
)
insert overwrite table dws_area_stats_daycount partition(dt='2020-06-14')
select
    pro.id,
    pro.province_name,
    pro.area_code,
    pro.iso_code,
    pro.region_id,
    pro.region_name,
    nvl(tmp_login.login_count,0),
    nvl(tmp_op.order_count,0),
    nvl(tmp_op.order_amount,0.0),
    nvl(tmp_op.payment_count,0),
    nvl(tmp_op.payment_amount,0.0)
from dwd_dim_base_province pro
left join tmp_login on pro.area_code=tmp_login.area_code
left join tmp_op on pro.id=tmp_op.province_id;
~~~

### DWT层宽表数据

累计值，dws是按照天进行汇总的，而dwt是对历史数据的累计。

在dwt层的表，一般都需要进行初始化操作，并且dwt层的表中数据一般都包含全量的数据。

#### 设备主题宽表

##### 创建表

![1640329874586](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/151115-806814.png)

设备主题宽表不是一个分区表，也是记录累计值，记录的是**全量数据信息**，指的是包含所有的设备id。一行数据就是一个访客记录或者设备，也就是说一行数据式一个设备的各种访客信息。

前三个字段是设备信息。

累计值是：

- 首次活跃时间：不需要进行更新操作
- 末次活跃时间：这个值我们一般需要每天进行更新操作。
- 当日活跃次数：
- 累计活跃次数：

更新表里面的数据思路：首先获取到表中的所有**历史数据**，然后再获取到**今天的活跃设备**，然后进行一个**全外连接**，通过全外连接，数据被分为三部分，**历史用户没有活跃，活跃的历史用户，新增的用户**。

数据来自于前端的埋点数据，如果前端没有进行埋点的话，就无法获取数据进行初始化操作，主要会影响首次活跃时间字段，累计活跃天数如果没有历史数据，无法计算，我们可以从数仓搭建起开始那天计算。

首次活跃时间可以这样做，和数仓中历史数据进行对比，如果没有关联上，就算作新增数据。

##### 字段更新思路

更新首次活跃时间，这个时间主要是针对新增的用户，如果一个用户是新增，那么我们就把这个字段更新为当天的日期即可。

末次活跃时间：这个时间需要我们频繁的更新操作，在上面三部分数据中，需要更新的是活跃的历史用户，针对这部分数据，我们需要把他的末次登录时间更新为当天计算的时间即可。

当日活跃次数：在dws_uv_detail_daycount表中有当日活跃次数，直接获取即可，dws中是以天为单位计算。

累计活跃次数：这个字段需要使用历史的累计活跃次数+今天的活跃次数。

##### 导入数据

![1640327082608](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/142443-252305.png)

~~~ sql
insert overwrite table dwt_uv_topic
select
	nvl(new.mid_id,old.mid_id),
	nvl(new.brand,old.mid_id),
	nvl(new.model,old.model),
	if(old.mid is null,'2020-06-14',old.login_date_first),
	--nvl(old.login_date_first,'2020-06-14')--首次登录时间
	if(new.mid_id is null,old.login_date_last,'2020-06-14'),--末次登录时间
	nvl(new.login_count,0),--登录次数
	nvl(old.login_count,0)+if(new.mid_id is null,1,0)
	
from
dwt_uv_topic old
full outer join
(
	select
		mid_id,
		brand,
		model,
		login_count
	from dws_uv_detail_daycount
	where dt='2020-06-14'
)new
on old.mid_id == new.mid_id
~~~

#### 会员主题宽表

宽表字段怎么来？维度关联的事实表度量值+开头、结尾+累积+累积一个时间段。

##### 建表语句

![1640329807171](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/151008-442757.png)

dwt层的数据是一个宽表，一行数据是一个用户，并且表中的字段是对用户各种操作的累计值，比如累计登录天数，累计30天登录天数。

涉及的表有：dwd_start_log，dws层订单事实表，dws用户行为表。

**这张表没有进行分区，因为记录的都是历史数据的聚合，每天需要进行依次聚合，然后再写入原表中，所以表每一天都需要更新数据。**

表中的字段计算方法来自于不同的表，比如登录日志，每一天都会有增加，如果我们每一天都进行计算，随着时间增长，数据量也会大大的增加。比如目前有100个分区，计算了依次，第二天增加3个分区，还要对103个分区重新计算，计算量大大增加不说，还有很多重复计算。

所以可以保存前面100个分区计算的结果，然后比如今天又增加三个分区，那么就计算者三个分区，然后去更新表中的值。所以最终的做法是先从表中查询记录，然后根据今天的值去更新操作。

> dwt层所有的表都是全量表，比如user_topic就应该包含所有的user。需要对所有的dwt层表进行一个初始化。

> 全量表一般不用做初始化，事实表一般也不用做初始化，拉链表第一天使用需要做初始化，因为拉链表记录的是新增和变化的数据。dwt层的表第一次使用需要做初始化。

需要拿到每一天的活跃用户，然后和原始表中的数据进行join比较操作，join之后，数据又join上的部分，也有没有join上的部分，代表的数据是：老用户并且今天活跃，老用户今天没有活跃，新增用户。

- 首次登录时间：对于老用户，这个字段不会改变，对于新用户，记录为今天的时间。
- 末次登录时间：对于老用户并且今天活跃和新用户的末次登录时间，修改为今天即可。
- 累计登录天数：原始用户并且没有活跃，不改变，原始用户并且今天活跃和新增用户，累计登录天数+1；
- 累计30天登录天数：需要获取30天之前的登录时间，如果30天之前登录了，数据需要-1，然后今天登录了+1，简单粗暴的就是直接重新计算最近30天登录数量。

其他字段更新思路一致。

![20211222091219](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211222091219.png)

##### 导入数据

- dws_user_action_daycount

**计算最近一天登录次数**

~~~ sql
-- 求最近一天的登录次数
-- dwd_start_log记录的是最明细的数据，里面每一条数据表示一次启动，如果从这里计算，那么每一天还需要去重操作，数据量比较大
--dws层每一条数据表示一个用户的使用记录，所以从dws层计算数据

select
    user_id
from dws_user_action_daycount
where dt='2021-06-22' and login_count >0
~~~

**最近30天，每一个用户登录天数**

~~~ sql
-- 计算最近30天，每一个用户的登录天数
select
    user_id,
    -- count(*),这里考虑0的情况
    sum(if(login_count >0,1,0))-- if里面第一个参数是一个判断条件，第二个和第三个值是返回的结果
from dws_user_action_daycount
where dt > date_add('2021-06-22',-29) and dt <= '2021-06-22'
group by user_id;
~~~

**合并两条sql**

~~~ sql
-- 合并上面两条sql的功能为一条sql
select
    user_id,
      sum(if(dt='2021-06-22',login_count,0)),--判断当天 是否登录
    -- count(*),这里考虑0的情况
    sum(if(login_count >0,1,0))-- 最近30天登录次数
from dws_user_action_daycount
where dt > date_add('2021-06-22',-29) and dt <= '2021-06-22'
group by user_id;
~~~

**完整sql**

~~~sql
----------------完整sql
insert overwrite table dwt_user_topic
select
    nvl(new.user_id,old.user_id),
    if(old.login_date_first is null and new.login_count>0,'2020-06-22',old.login_date_first),
    if(new.login_count>0,'2020-06-22',old.login_date_last),
    nvl(old.login_count,0)+if(new.login_count>0,1,0),
    nvl(new.login_last_30d_count,0),
    if(old.order_date_first is null and new.order_count>0,'2020-06-22',old.order_date_first),
    if(new.order_count>0,'2020-06-22',old.order_date_last),
    nvl(old.order_count,0)+nvl(new.order_count,0),
    nvl(old.order_amount,0)+nvl(new.order_amount,0),
    nvl(new.order_last_30d_count,0),
    nvl(new.order_last_30d_amount,0),
    if(old.payment_date_first is null and new.payment_count>0,'2020-06-22',old.payment_date_first),
    if(new.payment_count>0,'2020-06-22',old.payment_date_last),
    nvl(old.payment_count,0)+nvl(new.payment_count,0),
    nvl(old.payment_amount,0)+nvl(new.payment_amount,0),
    nvl(new.payment_last_30d_count,0),
    nvl(new.payment_last_30d_amount,0)
from
dwt_user_topic old --查出老数据
full outer join
(
    select
        user_id,
        sum(if(dt='2020-06-22',login_count,0)) login_count,
        sum(if(dt='2020-06-22',order_count,0)) order_count,
        sum(if(dt='2020-06-22',order_amount,0)) order_amount,
        sum(if(dt='2020-06-22',payment_count,0)) payment_count,
        sum(if(dt='2020-06-22',payment_amount,0)) payment_amount,
        sum(if(login_count>0,1,0)) login_last_30d_count,
        sum(order_count) order_last_30d_count,
        sum(order_amount) order_last_30d_amount,
        sum(payment_count) payment_last_30d_count,
        sum(payment_amount) payment_last_30d_amount
    from dws_user_action_daycount
    where dt>=date_add( '2020-06-22',-30)
    group by user_id
)new
on old.user_id=new.user_id;
~~~

#### 商品主题宽表

商品主题宽表中需要有全量的sku。

表的初始化，我们根据字段，去原始表中获取所有的数据，然后使用子查询全量导入查询。

表中每一天的更新逻辑，首先获取所有的历史数据，然后获取最近一天商品的各种行为，然后通过join两张表，更新累计值。

##### 创建表

![1640331398738](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/153640-128336.png)

##### 导入数据

- dws_sku_action_daycount

~~~sql
insert overwrite table dwt_sku_topic
select
    nvl(new.sku_id,old.sku_id),
    sku_info.spu_id,
    nvl(new.order_count30,0),
    nvl(new.order_num30,0),
    nvl(new.order_amount30,0),
    nvl(old.order_count,0) + nvl(new.order_count,0),
    nvl(old.order_num,0) + nvl(new.order_num,0),
    nvl(old.order_amount,0) + nvl(new.order_amount,0),
    nvl(new.payment_count30,0),
    nvl(new.payment_num30,0),
    nvl(new.payment_amount30,0),
    nvl(old.payment_count,0) + nvl(new.payment_count,0),
    nvl(old.payment_num,0) + nvl(new.payment_count,0),
    nvl(old.payment_amount,0) + nvl(new.payment_count,0),
    nvl(new.refund_count30,0),
    nvl(new.refund_num30,0),
    nvl(new.refund_amount30,0),
    nvl(old.refund_count,0) + nvl(new.refund_count,0),
    nvl(old.refund_num,0) + nvl(new.refund_num,0),
    nvl(old.refund_amount,0) + nvl(new.refund_amount,0),
    nvl(new.cart_count30,0),
    nvl(old.cart_count,0) + nvl(new.cart_count,0),
    nvl(new.favor_count30,0),
    nvl(old.favor_count,0) + nvl(new.favor_count,0),
    nvl(new.appraise_good_count30,0),
    nvl(new.appraise_mid_count30,0),
    nvl(new.appraise_bad_count30,0),
    nvl(new.appraise_default_count30,0)  ,
    nvl(old.appraise_good_count,0) + nvl(new.appraise_good_count,0),
    nvl(old.appraise_mid_count,0) + nvl(new.appraise_mid_count,0),
    nvl(old.appraise_bad_count,0) + nvl(new.appraise_bad_count,0),
    nvl(old.appraise_default_count,0) + nvl(new.appraise_default_count,0)
from
dwt_sku_topic old
full outer join
(
    select
        sku_id,
        sum(if(dt='2020-06-14', order_count,0 )) order_count, -- 6-14号，某一个sku被下单的次数
        sum(if(dt='2020-06-14',order_num ,0 ))  order_num,--当日下单的件数
        sum(if(dt='2020-06-14',order_amount,0 )) order_amount ,--当日下单的金额
        sum(if(dt='2020-06-14',payment_count,0 )) payment_count,--当日支付的金额
        sum(if(dt='2020-06-14',payment_num,0 )) payment_num,--支付数量
        sum(if(dt='2020-06-14',payment_amount,0 )) payment_amount,--支付总金额
        sum(if(dt='2020-06-14',refund_count,0 )) refund_count,
        sum(if(dt='2020-06-14',refund_num,0 )) refund_num,
        sum(if(dt='2020-06-14',refund_amount,0 )) refund_amount,
        sum(if(dt='2020-06-14',cart_count,0 )) cart_count,
        sum(if(dt='2020-06-14',favor_count,0 )) favor_count,
        sum(if(dt='2020-06-14',appraise_good_count,0 )) appraise_good_count,
        sum(if(dt='2020-06-14',appraise_mid_count,0 ) ) appraise_mid_count ,
        sum(if(dt='2020-06-14',appraise_bad_count,0 )) appraise_bad_count,
        sum(if(dt='2020-06-14',appraise_default_count,0 )) appraise_default_count,
        sum(order_count) order_count30 ,
        sum(order_num) order_num30,
        sum(order_amount) order_amount30,
        sum(payment_count) payment_count30,
        sum(payment_num) payment_num30,
        sum(payment_amount) payment_amount30,
        sum(refund_count) refund_count30,
        sum(refund_num) refund_num30,
        sum(refund_amount) refund_amount30,
        sum(cart_count) cart_count30,
        sum(favor_count) favor_count30,
        sum(appraise_good_count) appraise_good_count30,
        sum(appraise_mid_count) appraise_mid_count30,
        sum(appraise_bad_count) appraise_bad_count30,
        sum(appraise_default_count) appraise_default_count30
    from dws_sku_action_daycount
    where dt >= date_add ('2020-06-14', -30)
    group by sku_id
)new
on new.sku_id = old.sku_id
left join
(select * from dwd_dim_sku_info where dt='2020-06-14') sku_info
on nvl(new.sku_id,old.sku_id)= sku_info.id;
~~~

#### 活动主题宽表

活动主题宽表拥有全量的维度数据。

一行数据表示一个活动的各种累计值，每个活动一行数据。字段来自活动维度表的字段，，当日的字段，再dws层可以直接统计，累计等字段，更新思路是，先获取表中的历史数据，然后获取到当天的各种累计值，然后相加。

##### 创建表

![1640332378617](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/155300-26254.png)

##### 导入数据

- dws_activity_info_daycount

![1640332399313](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640332399313.png)

~~~ sql
insert overwrite table dwt_activity_topic
select
    nvl(new.id,old.id),
    nvl(new.activity_name,old.activity_name),
    nvl(new.activity_type,old.activity_type),
    nvl(new.start_time,old.start_time),
    nvl(new.end_time,old.end_time),
    nvl(new.create_time,old.create_time),
    nvl(new.display_count,0),
    nvl(new.order_count,0),
    nvl(new.order_amount,0.0),
    nvl(new.payment_count,0),
    nvl(new.payment_amount,0.0),
    nvl(new.display_count,0)+nvl(old.display_count,0),
    nvl(new.order_count,0)+nvl(old.order_count,0),
    nvl(new.order_amount,0.0)+nvl(old.order_amount,0.0),
    nvl(new.payment_count,0)+nvl(old.payment_count,0),
    nvl(new.payment_amount,0.0)+nvl(old.payment_amount,0.0)
from
(
    select
        *
    from dwt_activity_topic
)old
full outer join
(
    select
        *
    from dws_activity_info_daycount
    where dt='2020-06-14'
)new
on old.id=new.id;
~~~

#### 地区主题宽表

##### 创建表

![1640332525275](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640332525275.png)

##### 导入数据

- dws_area_stats_daycount

~~~sql
insert overwrite table dwt_area_topic
select
    nvl(old.id,new.id),
    nvl(old.province_name,new.province_name),
    nvl(old.area_code,new.area_code),
    nvl(old.iso_code,new.iso_code),
    nvl(old.region_id,new.region_id),
    nvl(old.region_name,new.region_name),
    nvl(new.login_day_count,0),
    nvl(new.login_last_30d_count,0),
    nvl(new.order_day_count,0),
    nvl(new.order_day_amount,0.0),
    nvl(new.order_last_30d_count,0),
    nvl(new.order_last_30d_amount,0.0),
    nvl(new.payment_day_count,0),
    nvl(new.payment_day_amount,0.0),
    nvl(new.payment_last_30d_count,0),
    nvl(new.payment_last_30d_amount,0.0)
from 
(
    select
        *
    from dwt_area_topic
)old
full outer join
(
    select
        id,
        province_name,
        area_code,
        iso_code,
        region_id,
        region_name,
        sum(if(dt='2020-06-14',login_count,0)) login_day_count,
        sum(if(dt='2020-06-14',order_count,0)) order_day_count,
        sum(if(dt='2020-06-14',order_amount,0.0)) order_day_amount,
        sum(if(dt='2020-06-14',payment_count,0)) payment_day_count,
        sum(if(dt='2020-06-14',payment_amount,0.0)) payment_day_amount,
        sum(login_count) login_last_30d_count,
        sum(order_count) order_last_30d_count,
        sum(order_amount) order_last_30d_amount,
        sum(payment_count) payment_last_30d_count,
        sum(payment_amount) payment_last_30d_amount
    from dws_area_stats_daycount
    where dt>=date_add('2020-06-14',-30)
    group by id,province_name,area_code,iso_code,region_id,region_name
)new
on old.id=new.id;
~~~

