

```sql
1001	2021-08-01
1001	2021-08-02
1001	2021-08-03
1001	2021-08-05
1001	2021-08-06
1001	2021-08-07
1001	2021-08-10
1001	2021-08-12
1002	2021-08-01
1002	2021-08-02
1002	2021-08-03
1002	2021-08-07
1002	2021-08-09
1002	2021-08-11
1002	2021-08-13
1002	2021-08-15
```

计算每一个人连续登录的最大天数，断一天也算连续

## 思路一：等差数列

1.1 开窗，按照id分组的同时按照dt进行排序，求Rank，但是如果又重复数据，先要进行排序。

```sql
select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx; 
```
![20211201134347](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201134347.png)

1.2 将每一行日期减去rk值，如果之前是连续的日期，则相减之后为相同日期。

这种做法前提是两个序列的等差值相同

```sql
select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1;
```
![20211201134432](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201134432.png)

1.3 计算绝对连续的天数

```sql
select
	id,
	flag,
	count(*) days
from 
	(select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1)t2
group id,flag;
```
![20211201134450](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201134450.png)

1.4 计算连续问题

```sql
select
	id，
	flag,
	days,
	rank() over(partition by id order by flag) newFlag
from 
	(select
	id,
	flag,
	count(*) days
from 
	(select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1)t2
group id,flag)t3;
```
![20211201134624](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201134624.png)

1.5 将flag 减去newFlag

```sql
select 
	id，
	days,
	date_sub(flag,newFlag) flag
from
	(
select
	id，
	flag,
	days,
	rank() over(partition by id order by flag) newFlag
from 
	(select
	id,
	flag,
	count(*) days
from 
	(select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1)t2
group id,flag)t3)t4;
```

![20211201134753](https://vscodepic.oss-cn-beijing.aliyuncs.com/pic/20211201134753.png)

1.6 计算每一个用户连续的登录天数

```sql
select 
	id,
	flag,
	sum(days)+count(*)-1 days
from 
	(select 
	id，
	days,
	date_sub(flag,newFlag) flag
from
	(
select
	id，
	flag,
	days,
	rank() over(partition by id order by flag) newFlag
from 
	(select
	id,
	flag,
	count(*) days
from 
	(select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1)t2
group id,flag)t3)t4)t5
group by id,flag;
```

1.7 计算最大连续天数

```sql
select 
	id,
	max(days)
from 
	(select 
	id,
	flag,
	sum(days)+count(*)-1 days
from 
	(select 
	id，
	days,
	date_sub(flag,newFlag) flag
from
	(
select
	id，
	flag,
	days,
	rank() over(partition by id order by flag) newFlag
from 
	(select
	id,
	flag,
	count(*) days
from 
	(select
	id,
	dt,
	data_sub(dt,rk) flag
from 
	(select
	id,
	dt,
	rank() over(partition by id order by dt) rk
from tx)t1)t2
group id,flag)t3)t4)t5
group by id,flag)t6
group by id;
```