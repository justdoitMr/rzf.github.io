### ADS层

> union:会对数据进行去重操作
>
> union all:不会对数据进行去重操作
>
> group by分组后具有去重功能

ADS层数据也分为若干个主题，但是ADS层不涉及数据建模，建表根据我们的需求决定，所以也不需要考虑表中都需要什么字段，一切根据需求而定，在dws和dwt层建表对应我们的维度表字段。

- 设备主题，统计流量的一些指标，默认都以设备id作为唯一的标识用户。
- 会员主题
- 商品主题
- 营销主题
- 地区主题

#### 设备主题

##### 活跃设备数（日、周、月）

- 日活：当日活跃的**设备数**

- 周活：当周活跃的**设备数**

- 月活：当月活跃的**设备数**

需求定义：默认都是以设备的id作为统一的标示（计算周活，日活，月活都需要对数据去重，都是以设备id为唯一标识）

> 只要今天用户登录了，那么就算作一个活跃用户。

日活：当日活跃的设备数，一天一个数，表示当天活跃设备数量。每天计算一次。如果从dwd层获取数据，dwd层每一行代表一个启动日志，所以需要去重操作，从dws层也可以，在天的粒度上已经去重了。

周活：当周活跃的设备数，当周一计算上周的周活。可以从dwd获取数据，需要去重，dws层也可以，但是需要去重，因为dws是对于天去重，但是我们需要计算一周，所以需在周的粒度上去重。

月活：当月活跃的设备数，下个月的1号计算上一个月的月活，对于月活也需要在月的粒度上进行去重操作。

> 以上我们的周，月都是指自然周和月，而不是指最近7天或者最近一个月。

再本项目中，对日活，周活和月活的计算进行统一，都是每天计算一次，这样的话，计算周活，本周一计算上周的周活，本周二计算本周的周一日活，周三计算本周一二的周活。直到下周一才可以计算完本周的周活，中间计算的周活不是完整的周活数据。

这个表中不需要进行分区，因为每天只需要向表中插入一条数据。为什么没有进行parquet列式存储呢？

因为ads层的表式最终的结果数据了，我们后期直接使用sqoop将全表导入mysql中，而parquet使用场景是我们需要查询若干个列的时候，效率高，我们导入的是全表，所以没必要使用列式存储。也没必要进行压缩，因为数据一天插入一条，一年300多条，数据量少。

**计算日活：**

从dwt层的dwt_uv_topic中根据**login_data_last**判断，如果末次登录时间等于今天，说明今天活跃。

如果我们想看历史上某一天是否是活跃，就不能使用上面方法判断，因为dwt层表中的最后登录时间会一直的进行更新。也就是说从dwt层计算日活的时候，只能计算最新一天的日活，历史上某一天无法计算，之所以我i发计算是因为login_data_last一直会更新，保存的永远是最新的数据。

如果从dws层计算，我们只需要找到最新一天的分区数据，这个分区中存储的就是当天的活跃设备，如果从dwd层计算，数据量太大，所以在本项目中，我们是从dws层计算。

**计算周活：**

首先找到本周一和本周日时间，然后对数据进行过滤。也是从dws层计算，也可以从dwt层计算，只要末次登录时间再本周一和本周日时间之间，就可以计算，算周活从dwt计算。

**计算月活：**

主要对比从dwt层计算还是从dws层计算，主要是看我们数据量的多少，尽量从数据量少的地方计算。

本项目中，三个指标都是从dwt层数据计算，可以放在一起计算，只是过滤条件不同。

###### 创建表

表中的字段也没有特殊的要求，需要根据我们的需求而定。

日周月设备活跃数量，因为我们需要每天计算日活，周活和月活，所以设计表的时候，增加了两个字段，这里多添加了两个字段，是否是周末或者是月末，因为周活和月活只有一个是准确的值，所以使用这两个字段作为标识，区分完整的周和月。

![1640411520376](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/135201-556468.png)

###### 装载数据

**计算周一和周日的日期**

```sql
--计算本周一
select data_add(next_day('2020-06-14','monday'),-7)

-- 计算本周日
select data_add(next_day('2020-06-14','monday'),-1)
```

###### 优化

- 但是这里有一个问题，如果使用into进行追加，如果任务失败的话，重新跑任务会导致数据的重复
- 追加操作是针对hive表来说的，但是对于hdfs来说，就是创建一个文件，所以对于ads层会产生很多的小文件
  - 解决办法：在插入数据之前，先将表中的数据查询出来，然后将今天的一条数据和查询出来的历史数据做一个union操作，然后将今天的一条数据和查询出来的历史数据做一个union操作，然后整体重新写入文件中，这样就避免了小文件的产生.
  - 对于数据的重复问题，使用overwrite+union，重新写入数据，避免重复数据。

**完整sql**

```sql
insert overwrite ads_uv_count
select * from ads_uv_count
union  -- union会进行去重操作，所以解决了任务失败重启时候，数据重复问题
select 
	'2020-06-14',
	sum(if(login_date_last == '2020-06-14',1,0)),--如果末次登录时间是今天，就返回1，否则返回0
	sum(if(login_date_last>=select data_add(next_day('2020-06-14','monday'),-7) and login_date_last <=select data_add(next_day('2020-06-14','monday'),-1)),1,0)
	-- 计算月活，只要末次登录日期位于本月之内即可,在这里对日期进行格式化，只要年月相同，就在同一个月内
	sum(if(date_format(login_date_last,'yyyy-MM')=date_format('2020-06-14','yyyy-MM'),1,0),
	if('2020-06-14' = data_add(next_day('2020-06-14','monday'),-1),'Y','N'),
	-- 判断今天是否是月末
	if('2020-06-14' = last_day('2020-06-14'),'Y','N')
from dwt_uv_topic;
```

具体的针对时间字段，我们在计算的时候，需要判断当天死否是周末或者是月末。

##### 每日新增设备

在dwt层的dwt_uv_topic表中有一个首次登录日期，可以用于计算，如果首次登录时间等于当天日期，那么就是新增设备。

> **通用的计算新增的方法，首先获取当天的活跃用户，然后获取表中的所有历史数据，做一个全外连接，全外连接中的数据分为三部分，历史用户没有活跃，历史用户活跃，新增用户，也可以计算新增。**

###### 创建表

![1640412488757](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/140810-146368.png)

###### 装载数据

- 新增用户的个数，根据时间判断
- 在这里也可以使用sum(if())，但是更加推荐使用where过滤条件，这样效率更高，会过滤很大一部分数据
- 在这里也需要考虑小文件和数据重复问题

**完整sql**

这里采用的是时间过滤法，而没有采用通用的方法。

~~~sql
insert overwrite table ads_new_mid_count
select * from ads_new_mid_count
union
select
    '2020-06-14',
    count(*)
from dwt_uv_topic
where login_date_first='2021-06-22';
~~~

##### 留存率

![20211223133708](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211223133708.png)

在这里我们计算每一天的三日留存率,注意一点是某一天计算的留存率指标，可能是前面几天的某几天的留存率。

###### 创建表

![1640413097574](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/141819-790483.png)

>  留存率=留存数量/新增数量
>
> 统计日期指的是计算日期，不重要，指是计算日期的前一天。
>
> 在这里标识一个留存是用：设备新增日期和截至当前日期留存天数来唯一标识。
>
> 表中的一行数据，表示一个留存。

计算留存，我们必须明确是在那一天的几日留存，

###### 案例

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

所以在18号这一天，我们计算的是14号的三日留存率，15号的2日留存率，16号的1日留存率。

###### 数据装载

我们以18号需要计算的留存来写sql。

- 在18号我们计算的内容有：
  - 14号的三日留存率
  - 15号的2日留存率
  - 16号的1日留存率

**首先计算14号的新增数据**

~~~sql
select
	count(*)
from dwt_uv_topic
where login_date_first = '2020-06-14'
~~~

**计算14号的三日留存**

~~~sql
select
	count(*)
from dwt_uv_topic
where login_date_first='2020-06-14'
and login_date_last = '2020-06-17'
~~~

在这里我们可以根据首次登录使劲啊和莫从登录时间进行计算，首次登录在14号，末次登录在17号，寿命15，16，17号都登录了，也就是三日留存。

**在这里需要使用join操作连接两个子查询，但是我们在这里sum if操作。**

~~~ sql
select
	sum(if(login_date_first = '2020-06-14',1,0),
	sum(if(login_date_first='2020-06-14'and login_date_last = '2020-06-17',1,0))
from dwt_uv_topic
~~~

**向表中插入数据**

~~~sql
select
	'2020-06-17',
	'2020-06-14', --计算14号
	3,--3日的禄存率
	sum(if(login_date_first='2020-06-14'and login_date_last = '2020-06-17',1,0)),
	sum(if(login_date_first = '2020-06-14',1,0),
	sum(if(login_date_first='2020-06-14'and login_date_last = '2020-06-17',1,0))/sum(if(login_date_first = '2020-06-14',1,0)
from dwt_uv_topic
~~~

###### **优化**

对于上面的sum(if)操作，我们可以使用where对数据进行先过滤操作，这样会大大的减少数据量。

~~~sql
-- 优化1，可以直接把14号数据过滤出来

--第一条sql对于每一个数据都需要进行判断，但是下面的优化，直接过滤掉大部分的数据，数据量更小
select
	'2020-06-17',
	'2020-06-14', --计算14号
	3,--3日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-14'
~~~

**计算2020-06-15日2日留存率**

有了14号留存用户的计算方法，对于15号和16号的计算，会方便很多，我们只需要修改时间即可。

~~~sql
select
	'2020-06-17',--统计日期不变。因为在17号才可以获取完整的15日留存用户
	'2020-06-15', --计算15号
	2,--2日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-15'
~~~

**计算2020-06-16日1日留存率**

~~~sql
select
	'2020-06-17',--统计日期不变。因为在17号才可以获取完整的15日留存用户
	'2020-06-16', --计算15号
	1,--2日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-16'
~~~

**向表中插入数据，对于本案例，也就是18号计算了三个留存，向表中插入三条数据**

~~~sql
--- 将上面sql语句union起来插入表中，相当于每天向表中插入三条数据
insert into ads_user_retention_day_rate
select
	'2020-06-17',
	'2020-06-14', --计算14号
	3,--3日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-14'
union all --不需要进行去重操作
select
	'2020-06-17',--统计日期不变。因为在17号才可以获取完整的15日留存用户
	'2020-06-15', --计算15号
	2,--2日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-15'
union all
select
	'2020-06-17',--统计日期不变。因为在17号才可以获取完整的15日留存用户
	'2020-06-16', --计算15号
	1,--2日的禄存率
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算14号新增数据
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first='2020-06-16'
~~~

观察上面的sql，计算14，15，16三个日期的留存用户的时候，需要三次从dwt_uv_topic表中读取数据，很影响效率，所以我们可以进一步优化，只向表中读取一次数据。

###### **减少表中读取次数优化**

需要按照login_date_first对数据进行分组操作.

~~~sql
select
	'2020-06-17',--统计日期不变。因为在17号才可以获取完整的15日留存用户
	login_date_first,--计算那一天的留存指标
	datediff('2020-06-17',login_date_first)--计算两个日期相隔的天数，我们要求分组后，选取的字段必须是常量值，分组字段或者聚合函数字段，但是这个
	--日期函数是一进一出，并且第一个参数是常量值，第二个参数是分组字段，所以说也可以选择
	sum(if(login_date_last = '2020-06-17',1,0)),--计算留存
	count(*)--计算新增
	sum(if(login_date_last = '2020-06-17',1,0))/count(*)
from dwt_uv_topic
where login_date_first in('2020-06-14','2020-06-15','2020-06-16')
group by login_date_first --按照三天的日期对数据进行分组
~~~

##### 沉默用户数

需求定义：

沉默用户：只在安装当天启动过，且启动时间是在7天前

第一次登录的时间等于最后一次的登录时间，

dwt_uv_topic中有一个累计活跃天数字段，如果字段为1，标识指在安装当天启动过，之后再也没有登陆过。还需要保证登录时间在7天之前

###### 创建表

![1640415263829](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/145425-800275.png)

###### 装载数据

思路很简单，首次登录时间==末次登录时间即可，说明仅仅在安装当天登录了一次。

~~~sql
select
    '2021-06-17',
    count(*)
from
    dwt_uv_topic
where login_date_first =login_date_last -- 安装时间等于启动时间，这个值为1 ，但是还需要保证在7天之前
and login_date_first<=date_add('2021-06-14',-7);
~~~

##### 本周回流用户

需求定义：

本周回流用户：上周未活跃，本周活跃的设备，且不是本周新增设备

###### 创建表

![1640415443444](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640415443444.png)

###### 装载数据

应该是以周为计算单位，因为本周过完后，才可以统计回流了几个用户。但是本项目中为了统一，也是按照天计算。

**本周回流用户肯定出自于本周活跃用户**，然后从种去掉本周新增用户和上周活跃用户，剩余的数据就是本周回流用户。

> 总体思路：先求出本周活跃用户，除掉本周新增用户，除掉上周活跃用户。

**首先获取本周活跃用户**

~~~sql
select
	mid_id
from dws_uv_detail_daycount
where dt>=data_add(next_day('2020-06-14','monday'),-7)
and dt<=data_add(next_day('2020-06-14','monday'),-1)
and login_count>0 --可能存在有的用户在前一天11：59登录，但是在第二天12:00后使用
group by mid_id
~~~

**获取本周新增用户**

~~~sql
select
	mid_id
from dwt_uv_topic
where login_date_first>=data_add(next_day('2020-06-14','monday'),-7)
and login_date_first<=data_add(next_day('2020-06-14','monday'),-1)
~~~

下面一条sql效果和上面两个子查询的join结果一样，先查询本周活跃用户，然后去掉本周新增用户

也可以使用一条sql搞定，从dwt层求出当周的活跃，通过最后登录时间求出，最后还需要减去上周活跃过的用户

~~~sql
select
	mid_id
from dwt_uv_topic
where login_date_last>=data_add(next_day('2020-06-14','monday'),-7)
and login_date_last<=data_add(next_day('2020-06-14','monday'),-1)--还需要过滤掉本周新增的数据
and login_date_first < data_add(next_day('2020-06-14','monday'),-7)
~~~

**计算上周活跃用户**

~~~sql
select
	mid_id
from dws_uv_detail_daycount
where dt>=data_add(next_day('2020-06-14','monday'),-7-7)
and dt<=data_add(next_day('2020-06-14','monday'),-1-7)
and login_count>0
group by mid_id
~~~

将上面的sql和上周活跃用户的数据进行left join操作，最终结果是老用户并且上周没有活跃，中间的是老用户并且上周活跃，最下面的是新增用户(实际就是将本周活跃用户 left join 上周活跃用户)

也可以使用not in 语法

~~~sql
-- 合并操作

(
	select
		mid_id
	from dwt_uv_topic
	where login_date_last>=data_add(next_day('2020-06-14','monday'),-7)
	and login_date_last<=data_add(next_day('2020-06-14','monday'),-1)--还需要过滤掉本周新增的数据
	and login_date_first < data_add(next_day('2020-06-14','monday'),-7)
)current_wk
left join
(
	select
		mid_id

	from dws_uv_detail_daycount
	where dt>=data_add(next_day('2020-06-14','monday'),-7-7)
	and dt<=data_add(next_day('2020-06-14','monday'),-1-7)
	and login_count>0
	group by mid_id
)last_wk
on current_wk.mid_id = last_wk.mid_id
~~~

**完整sql**

~~~sql
-- 从子查询种去选择数据

select
	'2020-06-14',
	concat(date_add(next_day('2021-06-22','monday'),-7),'_',date_add(next_day('2021-06-22','monday'),-1)),
	count(*)
from
(
	(
		select
			mid_id
		from dwt_uv_topic
		where login_date_last>=data_add(next_day('2020-06-14','monday'),-7)
		and login_date_last<=data_add(next_day('2020-06-14','monday'),-1)--还需要过滤掉本周新增的数据
		and login_date_first < data_add(next_day('2020-06-14','monday'),-7)
	)current_wk
	left join
	(
		select
			mid_id

		from dws_uv_detail_daycount
		where dt>=data_add(next_day('2020-06-14','monday'),-7-7)
		and dt<=data_add(next_day('2020-06-14','monday'),-1-7)
		and login_count>0
		group by mid_id
	)last_wk
	on current_wk.mid_id = last_wk.mid_id
)
where last_wk.mid_id is null;

-- 在这个需求中，减法操作使用not in实现也可以使用left join方法实现
~~~

> 减法操作使用not in实现也可以使用left join方法实现



##### 流式用户数

需求定义：

流失用户：最近7天未活跃的设备，也就是最后一次活跃时间在7天前。

###### 创建表

![1640416226996](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/151036-558085.png)

###### 数据装载

~~~sql
select
     '2020-06-25',
     count(*)
from 
(
    select 
        mid_id
    from dwt_uv_topic
    where login_date_last<=date_add('2020-06-25',-7)
    group by mid_id
)t1;
~~~

##### 最近连续三周活跃用户数

###### 创建表

![1640416388326](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/151310-28148.png)

本质上是找三个周内都活跃的用户：

**第一种思路**

**找共有的部分，最简单的就是将子查询进行内连接，返回的结果就是多个子查询的公共部分。**

如果使用login_data_last求，只能够求出当天，当周或者当月的，不能够求出历史数据。

**第二种思路**

先求出三个周的活跃用户，但是求出来之后并不是进行join操作，而是使用union all操作，之后形成了一个大的虚表，不会去重，然后从虚表种查询数据，按照设备的id进行分组，这样相同的设备id分为同一组，然后对每一组进行count(`*`)操作， 只要count(*)的结果是3，那么就表明连续三周活跃。

为什么可以这样做？

因为我们在分别求这三周种活跃用户的时候，分别按照设备的mid进行分组操作了，每一个设备id如果活跃了就只有一条数据，也就是说一组种一个设备id只能出现一次，已经进行去重操作。然后我们进行union all操作，然后再次分组，如果一组种个数是3，那么说明连续三周都活跃。这样做不会涉及多个表join操作，效率高。

###### 数据装载

首先查找本周的活跃用户，在dws层数据，每一个设备的行为占据一行，已经对数据进行去重，是按照天的单位去重。

但是我们在这里考虑的是一周，所以还需要考虑周去重问题,group by 为什么可以去重，group by之后，一组只能有一行数据，相当于去重
distinct在hive中不经常用，因为distinct是全局去重，所有数据分发到一个reduce中，效率低，而group by可以将一组数据发送到一个reduce中

###### 思路一

**本周活跃用户**

~~~sql
select
	mid_id
from dws_uv_detail_daycount
where dt>=data_add(next_day('2020-06-14','monday'),-7)
and dt<=data_add(next_day('2020-06-14','monday'),-1)
and login_count>0 --可能存在有的用户在前一天11：59登录，但是在第二天12:00后使用
group by mid_id
~~~

**上周活跃用户**

~~~sql
select
	mid_id
from dws_uv_detail_daycount
where dt>=data_add(next_day('2020-06-14','monday'),-7-7)
and dt<=data_add(next_day('2020-06-14','monday'),-1-7)
and login_count>0
group by mid_id
~~~

**上上周活跃用户**

~~~sql
select
	mid_id
from dws_uv_detail_daycount
where dt>=data_add(next_day('2020-06-14','monday'),-7-7-7)
and dt<=data_add(next_day('2020-06-14','monday'),-1-7-7)
and login_count>0
group by mid_id
~~~

**使用内连接找共有部分**

~~~sql
select
       '2021-06-22',
       concat(date_add(next_day('2021-06-22','monday'),-7-7-7),'_',date_add(next_day('2021-06-22','monday'),-1)),
       count(*)
from
(
    select
        mid_id
    from dws_uv_detail_daycount
    where dt>=date_add(next_day('2021-06-22','monday'),-7)
    and dt<=date_add(next_day('2021-06-22','monday'),-1)
    and login_count>0
    group by mid_id
)t1
join
(
-- 上周的数据
    select
        mid_id
    from dws_uv_detail_daycount
    where dt>=date_add(next_day('2021-06-22','monday'),-7-7)
    and dt<=date_add(next_day('2021-06-22','monday'),-1-7)
    and login_count>0
    group by mid_id
)t2
on t1.mid_id=t2.mid_id
(
    -- 上上周数据
    select
        mid_id
    from dws_uv_detail_daycount
    where dt>=date_add(next_day('2021-06-22','monday'),-7-7-7)
    and dt<=date_add(next_day('2021-06-22','monday'),-1-7-7)
    and login_count>0
    group by mid_id
)t3
on t1.mid_id=t3.mid_id
~~~

上面这种方法效率不高，因为每一个子查询数据量都不是很小，并且join很多

###### 思路二

将三周的数据不进行join操作，而是进行union all操作，然后再按照设备id进行分组操作，统计每组中数据的条数。

~~~sql
-- 第二种思路，使用union all
insert into table ads_continuity_wk_count
select
    '2020-06-25',
    concat(date_add(next_day('2020-06-25','MO'),-7*3),'_',date_add(next_day('2020-06-25','MO'),-1)),
    count(*)
from
(
    select
        mid_id
    from
    (
        select
            mid_id
        from dws_uv_detail_daycount
        where dt>=date_add(next_day('2020-06-25','monday'),-7)
        and dt<=date_add(next_day('2020-06-25','monday'),-1)
        group by mid_id

        union all

        select
            mid_id
        from dws_uv_detail_daycount
        where dt>=date_add(next_day('2020-06-25','monday'),-7*2)
        and dt<=date_add(next_day('2020-06-25','monday'),-7-1)
        group by mid_id

        union all

        select
            mid_id
        from dws_uv_detail_daycount
        where dt>=date_add(next_day('2020-06-25','monday'),-7*3)
        and dt<=date_add(next_day('2020-06-25','monday'),-7*2-1)
        group by mid_id
    )t1
    group by mid_id
    having count(*)=3
)t2;
~~~

##### 最近七天内连续三天活跃用户数

###### 创建表

![1640416914303](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/152259-59395.png)

wk_dt:使用七天中的第一天的日期拼接第七天的日期。

###### 查找连续的数据

首先获取每一个用户7天内的活跃记录，一个人一天的登录记录只有一个，所以需要去重。

![1640308302643](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/091143-587366.png)

**第一种思路**

找连续三天的，可以先找到中间一行数据，然后和前一条数据和后一条数据相比，也可以找到每一个日期下边的两个日期进行比较，因为我们的日期是经过排序的，并且经过去重的，如果第一个日期和第三个日期相差2，那么就一定是连续的。

![1640308780768](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/091942-243745.png)

**第二种思路**

![1640310103843](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/094145-112550.png)

###### 数据装载

###### 方案一

**首先找到所有人最近七天内的登录记录**

~~~ sql
--1、 首先找到所有人最近七天内的登录记录

--2、判断死否是连续三天，使用开窗函数lead获取某一行的下边某几行某几个字段的值

select
	count(*)
from
(
	select
		mid_id,
	from
	(
		select
			mid_id,
			datediff(lead2,dt) diff

		from
		(
			select
				mid_id,
				dt,
				lead(dt,2,'1970-01-01')over(partition by mid_id order by dt) lead2 -- 分组开窗
			from dws_uv_detail_daycount
			where dt>=data_add('2020-06-14',-6)
			and dt<=data_add('2020-06-14',-1)
			and login_count>0
		)t1
	)t2
	where diff =2
	group by mid_id -- 去重操作
)t3

-- 查询最近七天登录的用户
select
    count(*)
from
(
    select
        mid_id
    from
    (
        select
            mid_id,
            datediff(dt,lead2) diff
        from
        (
            select
                mid_id,
                dt,
                lead(dt,2,'1970-01-01') over(partition by mid_id order by dt)lead2
            from dws_uv_detail_daycount
            where dt>=date_add('2021-06-22',-6)
            and dt<='2021-06-22'
            and login_count>0
        )t1
    )t2
    where diff=2
    group by mid_id
)t3;
~~~

###### 方案二

开窗-=-》排序--》dt-rank-->根据相减的结果的差值是否相同，插值相同的并且有3行，说明是连续登录

~~~sql
-- 思路二
insert into table ads_continuity_uv_count
select
    '2020-06-22',
    concat(date_add('2020-06-22',-6),'_','2020-06-22'),
    count(*)
from
(
    select mid_id
    from
    (
        select mid_id
        from
        (
            select
                mid_id,
                date_sub(dt,rank) date_dif -- dt-rank的结果
            from
            (
                select
                    mid_id,
                    dt,
                    rank() over(partition by mid_id order by dt) rank
                from dws_uv_detail_daycount
                where dt>=date_add('2020-06-22',-6) and dt<='2020-06-22'
            )t1
        )t2
        group by mid_id,date_dif
        having count(*)>=3 --分组计算个数
    )t3 
    group by mid_id --按照mid进行分组
)t4;

~~~

#### 会员主题

##### 会员信息

###### 建表

![1640310556506](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/24/094917-677400.png)

- 活跃会员数：指的是当天活跃的用户
- 新增会员：指的是新增用户
- 新增消费会员：统计的当天，如果用户是第一次支付，那么就算新增消费会员。
- 总付费会员数：统计总的付过费的用户一共有多少。
- 总会员数：统计到目前为止一共有多少个用户。
- 会员活跃率：活跃会员/总会员数
- 会员付费率：总付费会员数/总会员数
- 会员新鲜度：当日的新增/当日的活跃

###### 装载数据

- 求新增会员和活跃会员都是从dwt层计算
- 活跃人数最后登录时间等于今天
- 当天新增会员：首次登录时间等于今天
- 新增消费会员数：首次支付时间等于今天即可
- 总付费会员，求一共付费的用户有多少人

~~~ sql
insert into table ads_user_topic
select
    '2020-06-22',
    sum(if(login_date_last='2020-06-22',1,0)),--当天活跃人数
    sum(if(login_date_first='2020-06-22',1,0)),--当天新增人数
    sum(if(payment_date_first='2020-06-22',1,0)),-- 当天消费会员，首次支付时间等于今天即可
    sum(if(payment_count>0,1,0)),--总付费会员
    count(*),--总会员数
    sum(if(login_date_last='2020-06-22',1,0))/count(*),
    sum(if(payment_count>0,1,0))/count(*),
    sum(if(login_date_first='2020-06-22',1,0))/sum(if(login_date_last='2020-06-22',1,0))
from dwt_user_topic;
~~~

##### 漏斗分析

统计：“浏览首页->浏览商品详情页->加入购物车->下单->支付”的转化率

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

###### 数据装载

**首先计算浏览首页的人数**

~~~sql
select
	count(*)
from
(
	select
		mid_id
	from dwd_page_log
	where dt='2020-06-14'
	and page_id='home'
	group by mid_id -- 对用户进行去重
)t1
~~~

**计算浏览了商品详情页的人数**

~~~sql
select
	count(*)
from
(
	select
		mid_id
	from dwd_page_log
	where dt='2020-06-14'
	and page_id='good_detail'
	group by mid_id -- 对用户进行去重
)t1
~~~

**合并两个子查询**

~~~sql
select
	sum(if(page_id='home',1,0),
	sum(if(page_id='good_detail',1,0))
from
(
	select
		mid_id,
		page_id
	from dwd_page_log
	where dt in('home','good_detail')
	group by mid_id,page_id
)t2
~~~

**计算加购物车，订单数和支付数量**

~~~sql
select
	sum(if(cart_count>0,1,0)),
	sum(if(order_count>0,1,0)),
	sum(if(payment_count>0,1,0))
from dws_user_action_daycount
where dt='2020-06-14'
~~~

**完整sql**

~~~sql
with
tmp_uv as
(
    select
        '2020-06-14' dt,
        sum(if(array_contains(pages,'home'),1,0)) home_count,
        sum(if(array_contains(pages,'good_detail'),1,0)) good_detail_count
    from
    (
        select
            mid_id,
            collect_set(page_id) pages
        from dwd_page_log
        where dt='2020-06-14'
        and page_id in ('home','good_detail')
        group by mid_id
    )tmp
),
tmp_cop as
(
    select
        '2020-06-14' dt,
        sum(if(cart_count>0,1,0)) cart_count,
        sum(if(order_count>0,1,0)) order_count,
        sum(if(payment_count>0,1,0)) payment_count
    from dws_user_action_daycount
    where dt='2020-06-14'
)
insert into table ads_user_action_convert_day
select
    tmp_uv.dt,
    tmp_uv.home_count,
    tmp_uv.good_detail_count,
    tmp_uv.good_detail_count/tmp_uv.home_count*100,
    tmp_cop.cart_count,
    tmp_cop.cart_count/tmp_uv.good_detail_count*100,
    tmp_cop.order_count,
    tmp_cop.order_count/tmp_cop.cart_count*100,
    tmp_cop.payment_count,
    tmp_cop.payment_count/tmp_cop.order_count*100
from tmp_uv
join tmp_cop
on tmp_uv.dt=tmp_cop.dt;
~~~

###### 第二种写法

~~~sql
select
    mid_id,
    page_struct.page_id
from dws_uv_detail_daycount lateral view explode(page_stats) tmp as page_struct --page_struct是炸出来的那一列的列明
where dt='2021-06-22';

-- 替换子查询
select
    sum(if(page_id='home',1,0)),
    sum(if(page_id='good_detail',1,0))
from
(
    select
        mid_id,
        page_struct.page_id
    from dws_uv_detail_daycount lateral view explode(page_stats) tmp as page_struct --page_struct是炸出来的那一列的列明
    where dt='2021-06-22'
)t1;
~~~

#### 商品主题

##### 商品个数信息

###### 创建表

![1640395737855](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/092859-918187.png)

###### 导入数据

~~~ sql
-- 装载数据,但是一般不推荐使用distinct进行去重操作，使用distinct去重会把所有数据发送一个reducer进行去重，
-- 所以一般使用group by
select
    count(*),
    count(distinct (spu_id))
from dwt_sku_topic;
~~~

我们一般不会使用distinct进行去重操作。下面使用group by进行去重

~~~ sql
-- 使用Group by进行去重\
select
    count(*)
from dwt_sku_topic;


-- 计算spu的个数
select
    count(*)
from
(
    select
        spu_id
    from dwt_sku_topic
    group by spu_id
)t1;
~~~

join上面两个子查询：

~~~ sql
-- join表中的数据，文档中的写法是阿静两个子查询join到一起，但是效率很低，因为从dwt_sku_topic表中读取了两次数据
insert into table ads_product_info
select
    '2020-06-22' dt,
    sku_num,
    spu_num
from
(
    select
        '2020-06-22' dt,
        count(*) sku_num
    from
        dwt_sku_topic
) tmp_sku_num
join
(
    select
        '2020-06-22' dt,
        count(*) spu_num
    from
    (
        select
            spu_id
        from
            dwt_sku_topic
        group by
            spu_id
    ) tmp_spu_id
) tmp_spu_num
on tmp_sku_num.dt=tmp_spu_num.dt;
~~~

优化sql查询：

~~~ sql
-- 第二种写法

select
    spu_id,
    count(*) sku_num -- 计算每一个分组中的个数
from dwt_sku_topic
group by spu_id

select
    '2021-06-22',
    count(*), --spu个数
    sum(sku_num) --sku的个数
from
(
    select
        spu_id,
        count(*) sku_num -- 计算每一个分组中的个数
    from dwt_sku_topic
    group by spu_id
)t1;
~~~

##### 商品销量排名

###### 创建表

![1640395995313](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/093316-997264.png)

涉及排名的需求，一般不会进行全局排序操作，一般求top k问题。

计算商品的排名，我们统一使用sku。时间统一按照当天计算。

商品销量排名，我们统一按照支付金额进行计算。

计算每一个品牌下面销量前10的sku。

###### 装载数据

~~~ sql
-- 装载数据
-- 求每一个品牌下面销量前十的spu，在这里我们按照支付金额统计

-- 1、获取当天的每一个sku的销量，按照销量排序

select
    '2021-06-22',
    sku_id,
    payment_amount
from dws_sku_action_daycount
where dt='2021-06-22'
order by payment_amount desc
limit 10;
~~~

##### 商品收藏排名

按照商品被收藏的次数计算。

###### 创建表

![1640396358312](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640396358312.png)

###### 装载数据

~~~sql
-- 装载数据
insert into table ads_product_favor_topN
select
    '2020-06-22' dt,
    sku_id,
    favor_count
from
    dws_sku_action_daycount
where
    dt='2020-06-22'
order by favor_count desc
limit 10;
~~~

##### 商品加入购物车排名

按照商品被加入购物车的次数。

###### 创建表

![1640396395133](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1640396395133.png)

###### 装载数据

~~~sql
-- 装载数据
insert into table ads_product_cart_topN
select
    '2020-06-22' dt,
    sku_id,
    cart_count
from
    dws_sku_action_daycount
where
    dt='2020-06-22'
order by cart_count desc
limit 10;
~~~

##### 商品退款率排名（最近30天）

退款率：某一件商品退还次数/付款次数

如果只计算当天的退款率没有意义，因为今天退款的商品可能不是今天支付的，退款和支付之间有一个时间差。所以我们通常算最近30天的数据。

###### 创建表

![1640397361981](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/095603-318249.png)

###### 装载数据

~~~ sql
-- 装载数据
insert into table ads_product_refund_topN
select
    '2020-06-22',
    sku_id,
    refund_last_30d_count/payment_last_30d_count*100 refund_ratio
from dwt_sku_topic
order by refund_ratio desc
limit 10;
~~~

##### 商品差评率

也是按照差评率排序，然后求top 10。

差评率：差评数/总评价次数

###### 创建表

![1640396554363](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/094235-74114.png)

###### 装载数据

~~~ sql
-- 数据装载
insert into table ads_appraise_bad_topN
select
    '2020-06-14' dt,
    sku_id,
appraise_bad_count/(appraise_good_count+appraise_mid_count+appraise_bad_count+appraise_default_count) appraise_bad_ratio
from
    dws_sku_action_daycount
where
    dt='2020-06-14'
order by appraise_bad_ratio desc
limit 10;
~~~

##### 分组top N问题

开窗+where过滤

> 每一个品牌下销量前十的sku

1. 首先获取支付金额

~~~ sql
--获取支付金额
        select
            '2021-06-22',
            sku_id,
            payment_amount
        from dws_sku_action_daycount
        where dt='2021-06-22'
~~~

2. 获取品牌信息

~~~ sql
        select
            id,
            tm_id
        from
            dwd_dim_sku_info
        where dt='2021-06-22'
    )sku_info
~~~

3. join两个结果

因为我们需要上面的sql返回的结果，所以使用的是left join

~~~ sql
    (
	--获取支付金额
        select
            '2021-06-22',
            sku_id,
            payment_amount
        from dws_sku_action_daycount
        where dt='2021-06-22'
    )sku_pay
    left join--因为我们需要上一个子查询的结果，所以使用left join
    -- 获取商品维度表，从中获取品牌信息
    (
        select
            id,
            tm_id
        from
            dwd_dim_sku_info
        where dt='2021-06-22'
    )sku_info
    on sku_pay.sku_id=sku_info.id
~~~

4. 开窗聚合

~~~ sql
    select
        tm_id, --获取品牌信息
        sku_id, -- 获取sku_id
        payment_amount,--获取支付金额
        rank() over (partition by tm_id order by payment_amount desc )rk --按照tm_id进行分组开窗，然后每一个窗口按照payment_amount进行排序操作
    from
    (
	--获取支付金额
        select
            '2021-06-22',
            sku_id,
            payment_amount
        from dws_sku_action_daycount
        where dt='2021-06-22'
    )sku_pay
    left join--因为我们需要上一个子查询的结果，所以使用left join
    -- 获取商品维度表，从中获取品牌信息
    (
        select
            id,
            tm_id
        from
            dwd_dim_sku_info
        where dt='2021-06-22'
    )sku_info
    on sku_pay.sku_id=sku_info.id
~~~

**完整sql**

~~~ sql
-- 分组+topN
-- 固定写法：开窗+过滤


-- 每一个品牌下销量前十的sku

-- 1、首先获取当天每一个sku的销量
-- 2、找到每一个sku所属的品牌
-- 3、对品牌进行开窗+rank()操作（partition by 品牌 order by 销量）

select
    '2021-06-22',
    tm_id,
    sku_id,
    payment_amount,--获取支付金额
    rk
from
(
    select
        tm_id, --获取品牌信息
        sku_id, -- 获取sku_id
        payment_amount,--获取支付金额
        rank() over (partition by tm_id order by payment_amount desc )rk --按照tm_id进行分组开窗，然后每一个窗口按照payment_amount进行排序操作
    from
    (
	--获取支付金额
        select
            '2021-06-22',
            sku_id,
            payment_amount
        from dws_sku_action_daycount
        where dt='2021-06-22'
    )sku_pay
    left join--因为我们需要上一个子查询的结果，所以使用left join
    -- 获取商品维度表，从中获取品牌信息
    (
        select
            id,
            tm_id
        from
            dwd_dim_sku_info
        where dt='2021-06-22'
    )sku_info
    on sku_pay.sku_id=sku_info.id
)t1
where rk <=10;--去排名前十
~~~

#### 营销主题（用户+商品+购买行为）

##### 下单数目统计

需求分析：统计每日下单数，下单金额及下单用户数。

###### 创建表

![1640398576178](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/101617-652874.png)

- 下单笔数：当天所有人下单次数之和
- 单日下单金额：统计当天下单金额的总和，GMV.
- 单日下单用户数：统计当日有多少用户下单

> gmv指的是一段时间内的成交总额，用于电商行业，一般包括拍下的未支付的订单金额。

###### 装载数据

~~~ sql
-- 三个字段都可以从dwd层的订单事实表中去计算，但是dwd层是最明细的数据，需要对订单进行去重操作
-- 也可以去dws层的用户主题表中计算，一行数据标识一个用户当天的各种行为，对每一个人的下单笔数和下单金额相加，就是总的下单金额和下单笔数
-- 在这里从dws层计算数据，相当于已经做过当天的汇总数据


-- 装载数据

select
    '2021-06-22',
    sum(order_count),
    sum(order_amount),
    sum(if(order_count>0,1,0))
from dws_user_action_daycount
where dt='2021-06-22';
~~~

##### 支付信息统计

每日支付金额、支付人数、支付商品数、支付笔数以及下单到支付的平均时长（取自DWD）

###### 创建表

![1640398979981](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/102305-882956.png)

单日支付商品数，在这里统计一共有多少sku被支付。

单日支付人数：统计当天一共有多少个人支付了。

下单到支付的平均时长：首先获取下单时间，然后获取支付时间，最后取品均值即可。

###### 装载数据

**计算单日支付笔数，支付金额，支付人数**

~~~sql
-- 还是从dws层的用户行为表中统计，只需要累加即可
-- 装载数据
select
    '2021-06-22' dt,
    sum(payment_count),
    sum(payment_amount),
    sum(if(payment_count>0,1,0))
from dws_user_action_daycount
where dt='2021-06-22';
~~~

**计算单日支付商品数有，根据sku**

~~~sql
-- 查询被支付的商品数，也就是一共有多少个sku被支付
-- 在这里也可以使用sum if，但是我们使用where，因为可以过滤，减少很大一部分数据
select
    '2021-06-22' dt,
    count(*)
from dws_sku_action_daycount
where dt='2021-02-22'
and payment_count>0;
~~~

**计算订单时间和支付时间**

在dwd_fact_order_info中有订单的创建时间和支付时间，但是有的订单可能创建了，但是最终并没有被支付，所以需要过滤掉没有被支付的订单。

~~~sql
select
    '2021-06-22' dt,
    avg(unix_timestamp(payment_time)-unix_timestamp(create_time))/60
from dwd_fact_order_info
where dt='2021-06-22'
and payment_time is not null;
~~~

**最终sql**

~~~sql
--最终sql
select
    tmp_payment.dt,
    tmp_payment.payment_count,
    tmp_payment.payment_amount,
    tmp_payment.payment_user_count,
    tmp_skucount.payment_sku_count,
    tmp_time.payment_avg_time
from
(
    select
        '2020-06-14' dt,
        sum(payment_count) payment_count,
        sum(payment_amount) payment_amount,
        sum(if(payment_count>0,1,0)) payment_user_count
    from dws_user_action_daycount
    where dt='2020-06-14'
)tmp_payment
join
(
    select
        '2020-06-14' dt,
        sum(if(payment_count>0,1,0)) payment_sku_count
    from dws_sku_action_daycount
    where dt='2020-06-14'
)tmp_skucount on tmp_payment.dt=tmp_skucount.dt
join
(
    select
        '2020-06-14' dt,
        sum(unix_timestamp(payment_time)-unix_timestamp(create_time))/count(*)/60 payment_avg_time
    from dwd_fact_order_info
    where dt='2020-06-14'
    and payment_time is not null
)tmp_time on tmp_payment.dt=tmp_time.dt;
~~~

##### 品牌复购率

重复购买就是复购，通常统计一段时间内的复购率。可能以周，月，季度为单位。

> 计算方法：**购买过该品牌的人数（下单次数>=1）/重复购买过该品牌的人数(下单次数>=2)**单词复购率
>
> 购买过该品牌的人数（下单次数>=2）/重复购买过该品牌的人数(下单次数>=3)多次复购率

###### 创建表

在这里统计的是一个自然月，而不是最近30天。

![1640399025605](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/102346-825664.png)

统计每个category1品类下的每一个tm_id下的单次复购率和多次复购率。

![1640404886289](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/120128-911742.png)

我们需要计算的，是购买品牌A次数大于等于1的人数，购买品牌B次数大于等于2的有多少人。因为有多个品牌，所以我们需要求多个品牌。

###### 装载数据

dwd_order_detail层的用户订单表中有用户和sku之间的关系，用户没有和品牌有直接的关系，所以我们需要将dwd_order_detail表和品牌表进行join操作。

1. 计算每一个用户购买过的sku

~~~ sql
select
	user_id,
	sku_id
from dwd_fact_order_detail
where data_format(dt,'yyyy-MM')=data_format('2020-06-14','yyyy-MM')
~~~

2. 获取品牌id,品牌名字，categoryid和一级种类名

~~~sql
select
	id,
	tm_id,
	category1_id,
	category1_name
from dwd_dim_sku_info
where dt='2020-06-14'
~~~

一行数据代表一个用户购买的商品，后面还有商品品牌和种类。
我们要统计每一个用户购买某一个品牌的次数，所以我们直接按照userid+品牌id进行分组即可

每一个用户购买每一个以及品牌下面一个品牌商品的次数

3. 因为需要第一个sql的结果，所以使用left join

~~~sql
select
	user_id,
	tm_id,
	category1_id,
	category1_name
	count(*)--统计购买次数
from 
(
	select
		user_id,
		sku_id
	from dwd_fact_order_detail
	where data_format(dt,'yyyy-MM')=data_format('2020-06-14','yyyy-MM')
)od
left join
(
	select
		id,
		tm_id,
		category1_id,
		category1_name
	from dwd_dim_sku_info
	where dt='2020-06-14'
)sku
on od.sku_id=sku.sku_id
group by user_id,tm_id,category1_id,category1_name
~~~

**完整sql**

~~~sql
select
	 tm_id,
    category1_id,
    category1_name,
    sum(if(order_count>=1,1,0)) buycount,
    sum(if(order_count>=2,1,0)) buyTwiceLast,
    sum(if(order_count>=2,1,0))/sum( if(order_count>=1,1,0)) buyTwiceLastRatio,
    sum(if(order_count>=3,1,0))  buy3timeLast  ,
    sum(if(order_count>=3,1,0))/sum( if(order_count>=1,1,0)) buy3timeLastRatio ,
    date_format('2020-06-14' ,'yyyy-MM') stat_mn,
    '2020-06-14' stat_date

from 
(
	select
		user_id,
		tm_id,
		category1_id,
		category1_name
		count(*)  order_count--统计购买次数
	from 
	(
		select
			user_id,
			sku_id
		from dwd_fact_order_detail
		where data_format(dt,'yyyy-MM')=data_format('2020-06-14','yyyy-MM')
	)od
	left join
	(
		select
			id,
			tm_id,
			category1_id,
			category1_name
		from dwd_dim_sku_info
		where dt='2020-06-14'
	)sku
	on od.sku_id=sku.sku_id
	group by user_id,tm_id,category1_id,category1_name
)t1
group by tm_id,category1_id,category1_name -- 相同品牌，相同品类的分到一组


with
tmp_order as
(
    select
        user_id,
        order_stats_struct.sku_id sku_id,
        order_stats_struct.order_count order_count
    from dws_user_action_daycount lateral view explode(order_detail_stats) tmp as order_stats_struct
    where date_format(dt,'yyyy-MM')=date_format('2020-06-14','yyyy-MM')--过滤出一整个月的数据
),
tmp_sku as
(
    select
        id,
        tm_id,
        category1_id,
        category1_name
    from dwd_dim_sku_info
    where dt='2020-06-14'
)
~~~

上面的思路中，最开始是去dwd_fact_order_detail最明细的数据，并且是一个月的数据，数据量比较的大，在dws_user_action_daycount中有订单的详细信息，是以数组的形式存在的，表示每一个user购买过的sku信息。也就是从这一张表中可以获取一个user购买过的sku信息。数据计算量小。

我们最终想要得到的结果是每一个user购买过某一个品牌的次数，我们先可以间接的获取每一个user购买过的每一个sku的次数，然后在和商品维度表进行join操作，然后在按照品牌分组，数据量相对较小。

**完整sql**

~~~sql

with
tmp_order as
(
    select
        user_id,
        order_stats_struct.sku_id sku_id,
        order_stats_struct.order_count order_count
    from dws_user_action_daycount lateral view explode(order_detail_stats) tmp as order_stats_struct -- 首先把结构体数组炸裂开
    where date_format(dt,'yyyy-MM')=date_format('2020-06-14','yyyy-MM')--过滤出一整个月的数据
),
tmp_sku as
(
-- 商品维度表
    select
        id,
        tm_id,
        category1_id,
        category1_name
    from dwd_dim_sku_info
    where dt='2020-06-14'
)


-- 装载数据

insert into table ads_sale_tm_category1_stat_mn
select
    tm_id,
    category1_id,
    category1_name,
    sum(if(order_count>=1,1,0)) buycount,
    sum(if(order_count>=2,1,0)) buyTwiceLast,
    sum(if(order_count>=2,1,0))/sum( if(order_count>=1,1,0)) buyTwiceLastRatio,
    sum(if(order_count>=3,1,0))  buy3timeLast  ,
    sum(if(order_count>=3,1,0))/sum( if(order_count>=1,1,0)) buy3timeLastRatio ,
    date_format('2020-06-14' ,'yyyy-MM') stat_mn,
    '2020-06-14' stat_date
from
(
    select
        tmp_order.user_id,
        tmp_sku.category1_id,
        tmp_sku.category1_name,
        tmp_sku.tm_id,
        sum(order_count) order_count
    from tmp_order
    join tmp_sku
    on tmp_order.sku_id=tmp_sku.id
    group by tmp_order.user_id,tmp_sku.category1_id,tmp_sku.category1_name,tmp_sku.tm_id
)tmp
group by tm_id, category1_id, category1_name;
~~~

#### 地区主题

##### 地区主题信息

###### 创建表

![1640399065390](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/102426-684635.png)