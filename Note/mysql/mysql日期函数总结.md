## mysql日期函数总结

### ==获取当前日期及格式化输出==

- 获取系统日期：`now()`
- 格式化日期：`date_format(date,format)`
- 返回系统日期。输出`2009-12-25 14:00:00`

```sql
select now(); #获取当前时间
select date_format(now(),'%Y-%m-%d %H:%I:%s'); #获取当前时间并格式化输出
```

#### format可选格式类型：

```sql
%S, %s 两位数字形式的秒（ 00,01, …, 59）
%I, %i 两位数字形式的分（ 00,01, …, 59）
%H 两位数字形式的小时，24 小时（00,01, …, 23）
%h 两位数字形式的小时，12 小时（01,02, …, 12）
%k 数字形式的小时，24 小时（0,1, …, 23）
%l 数字形式的小时，12 小时（1, 2, …, 12）
%T 24 小时的时间形式（hh:mm:ss）
%r 12 小时的时间形式（hh:mm:ss AM 或hh:mm:ss PM）
%p AM或PM
%W 一周中每一天的名称（Sunday, Monday, …, Saturday）
%a 一周中每一天名称的缩写（Sun, Mon, …, Sat）
%d 两位数字表示月中的天数（00, 01,…, 31）
%e 数字形式表示月中的天数（1, 2， …, 31）
%D 英文后缀表示月中的天数（1st, 2nd, 3rd,…）
%w 以数字形式表示周中的天数（ 0 = Sunday, 1=Monday, …, 6=Saturday）
%j 以三位数字表示年中的天数（ 001, 002, …, 366）
%U 周（0, 1, 52），其中Sunday 为周中的第一天
%u 周（0, 1, 52），其中Monday 为周中的第一天
%M 月名（January, February, …, December）
%b 缩写的月名（ January, February,…, December）
%m 两位数字表示的月份（01, 02, …, 12）
%c 数字表示的月份（1, 2, …, 12）
%Y 四位数字表示的年份
%y 两位数字表示的年份
%% 直接值“%”
```

#### ==常用格式==

~~~sql
select date_format(now(),'%Y-%m-%d %H:%I:%s'); #获取当前时间并格式化输出
date_format(now(),'%Y-%m') -- 获取年和月
-- 如果输出日期不要-的时候，那么可以使用下面这种，吧格式化字符串中间的-去掉
date_format(now(),'%Y%m') -- 获取年和月
~~~

> 除了now()外，MySql还提供了一下函数处理时间
>
> current_timestamp()
> localtime()
> localtimestamp()
> 等同于now()

#### 获得当前日期+时间：sysdate()

```sql
select sysdate();
2022-03-21 11:16:54
```

#### 获取当前日期：`curdate()`

```sql
select curdate();
2022-03-21
```

#### 获取当前时间：`curtime()`

```sql
select curtime();
11:17:22
```

### ==MySQL 日期转换函数、时间转换函数==

#### MySQL （时间、秒）转换函数：`time_to_sec(time)`, `sec_to_time(seconds)`

```sql
select time_to_sec( '01:00:05' ); -- 3605
select sec_to_time(3605); -- '01:00:05'
```

#### MySQL （日期、天数）转换函数：`to_days(date)`, `from_days(days)`

```sql
select to_days( '0000-00-00' ); -- 0
select to_days( '2008-08-08' ); -- 733627
select from_days(0); -- '0000-00-00'
select from_days(733627); -- '2008-08-08'
```

#### ==MySQL Str to Date （字符串转换为日期）函数：`str_to_date(str, format)`==

```sql
select str_to_date( '08/09/2008' , '%m/%d/%Y' ); -- 2008-08-09
select str_to_date( '2022-02-28 00:00:00' , '%Y-%m-%d %H:%I:%s' ); -- 2022-02-28 00:00:00
```

#### ==MySQL 日期/时间转换为字符串函数：`date_format(date,format)`, `time_format(time,format)`==

```sql
select date_format( '2008-08-08 22:23:00' , '%W %M %Y' ); --Friday August 2008
select date_format('2008-08-08 22:23:01', '%Y%m%d%H%i%s'); --20080808222301
select time_format('22:23:01', '%H.%i.%s'); --22.23.01
select DATE_FORMAT(NOW(),'%Y-%m-%d %H-%I-%s');##日期格式化
```

#### MySQL 获得国家地区时间格式函数：`get_format()`

```sql
--语法 ：get_format(date|time|datetime, 'eur'|'usa'|'jis'|'iso'|'internal'

select get_format( date , 'usa' ) ; -- '%m.%d.%Y'
select get_format( date , 'jis' ) ; -- '%Y-%m-%d'
select get_format( date , 'iso' ) ; -- '%Y-%m-%d'
select get_format( date , 'eur' ) ; -- '%d.%m.%Y'
select get_format( date , 'internal' ) ; -- '%Y%m%d'
select get_format(datetime, 'usa' ) ; -- '%Y-%m-%d %H.%i.%s'
select get_format(datetime, 'jis' ) ; -- '%Y-%m-%d %H:%i:%s'
select get_format(datetime, 'iso' ) ; -- '%Y-%m-%d %H:%i:%s'
select get_format(datetime, 'eur' ) ; -- '%Y-%m-%d %H.%i.%s'
select get_format(datetime, 'internal' ) ; -- '%Y%m%d%H%i%s'
select get_format( time , 'usa' ) ; -- '%h:%i:%s %p'
select get_format( time , 'jis' ) ; -- '%H:%i:%s'
select get_format( time , 'iso' ) ; -- '%H:%i:%s'
select get_format( time , 'eur' ) ; -- '%H.%i.%s'
select get_format( time , 'internal' ) ; -- '%H%i%s'
```

#### MySQL 拼凑日期、时间函数：`makdedate(year,dayofyear)`, `maketime(hour,minute,second)`

```sql
select makedate(2001,31); -- '2001-01-31'
select makedate(2001,32); -- '2001-02-01'
select maketime(12,15,30); -- '12:15:30'
```

### MySQL 时间戳（Timestamp）函数

1）MySQL 获得当前时间戳函数：`current_timestamp`, `current_timestamp()`

```sql
select current_timestamp , current_timestamp ();
```

2）MySQL （Unix 时间戳、日期）转换函数

```sql
unix_timestamp(),
unix_timestamp( date ),
from_unixtime(unix_timestamp),
from_unixtime(unix_timestamp,format)
```

3）MySQL 时间戳（timestamp）转换、增、减函数

```sql
timestamp ( date ) -- date to timestamp
timestamp (dt, time ) -- dt + time
timestampadd(unit,interval,datetime_expr) --
timestampdiff(unit,datetime_expr1,datetime_expr2) --
```

### ==计算两个时间相差的分钟数==

下面说明了`TIMESTAMPDIFF`函数的语法。

`TIMESTAMPDIFF(unit,begin,end)`

`TIMESTAMPDIFF`函数返回`begin-end`的结果，其中`begin`和`end`是[DATE](http://www.yiibai.com/mysql/date.html)或[DATETIME](http://www.yiibai.com/mysql/datetime.html)表达式。

`TIMESTAMPDIFF`函数允许其参数具有混合类型，例如，`begin`是`DATE`值，`end`可以是`DATETIME`值。 如果使用`DATE`值，则`TIMESTAMPDIFF`函数将其视为时间部分为`“00:00:00”`的`DATETIME`值。

`unit`参数是确定(`end-begin`)的结果的单位，表示为整数。 以下是有效单位：

- MICROSECOND
- SECOND
- MINUTE
- HOUR
- DAY
- WEEK
- MONTH
- QUARTER
- YEAR

~~~sql
TIMESTAMPDIFF(SECOND,start_time,end_time)-- 计算相差的分钟数
~~~

### ==日期时间Extract函数==

1）选取日期时间的各个部分：日期、时间、年、季度、月、日、小时、分钟、秒、微妙

```sql
set @dt = '2020-10-29 15:19:00.123456'

select date(@dt); --2020-10-29
select time(@dt); --15:19:00.123456
select year(@dt); --2020
select quarter(@dt); --4
select month(@dt); --10
select week(@dt); --43
select day(@dt); --29
select hour(@dt); --15
select minute(@dt); --19
select second(@dt); --00
select microsecond(@dt); --123456
```

2）MySql `Extract()`函数

```sql
set @dt = '2008-09-10 07:15:30.123456' ;
 
select extract( year from @dt); -- 2008
select extract(quarter from @dt); -- 3
select extract( month from @dt); -- 9
select extract(week from @dt); -- 36
select extract( day from @dt); -- 10
select extract( hour from @dt); -- 7
select extract( minute from @dt); -- 15
select extract( second from @dt); -- 30
select extract(microsecond from @dt); -- 123456
select extract(year_month from @dt); -- 200809
select extract(day_hour from @dt); -- 1007
select extract(day_minute from @dt); -- 100715
select extract(day_second from @dt); -- 10071530
select extract(day_microsecond from @dt); -- 10071530123456
select extract(hour_minute from @dt); -- 715
select extract(hour_second from @dt); -- 71530
select extract(hour_microsecond from @dt); -- 71530123456
select extract(minute_second from @dt); -- 1530
select extract(minute_microsecond from @dt); -- 1530123456
select extract(second_microsecond from @dt); -- 30123456
```

3）MySql dayof…函数：`dayofweek()`,`dayofmonth()`,`dayofyear()`
分别返回日期参数，在一周、一月、一年的位置

```sql
set @dt='2020-10-29 15:15:20'

select dayofweek(@dt); --周几
select dayofmonth(@dt);  --几月
select dayofyear(@dt);  --某天
```

4）MySql week…函数：`week()`,`weekofyear()`,`weekday()`,`yearweek()`

```sql
set @dt='2020-10-29 15:15:20'

select week(@dt); --第几周
select week(@dt,3);  --三天后是第几周
select weekofyear(@dt);  --一年中的第几周
select dayofweek(@dt);  --“某天”在一周中的位置
select weekday(@dt);  --3
select yearweek(@dt);  --那年的第几周  202043
```

```sql
weekday：(0 =Monday, 1 = Tuesday, …, 6 = Sunday)；
dayofweek：（1 =Sunday, 2 = Monday,…, 7 = Saturday）
```

5）MySql 返回星期和月份名称函数：`dayname()`,`monthname()`

```sql
set @dt='2020-10-29 15:15:20'

select dayname(@dt);  --Thursday
select monthname(@dt);  --October
```

#### MySQL `last_day()` 函数：返回月份中的最后一天

```sql
select last_day('2020-10-29 15:15:20'); ---2020-10-31
select day (last_day(now())) as days; --获取本月有多少天,也就是返回本个月的最后一天
```

### MySql日期时间计算函数

1）MySql 为日期增加一个时间间隔：`date_add()`

```sql
set @dt=now();

select date_add(@dt, interval 1 day ); -- 加一天
select date_add(@dt, interval 1 hour ); -- 加一小时
select date_add(@dt, interval 1 minute ); -- 加一分
select date_add(@dt, interval 1 second ); --加一秒
select date_add(@dt, interval 1 microsecond); --加一毫秒
select date_add(@dt, interval 1 week); --加一周
select date_add(@dt, interval 1 month ); --加一月
select date_add(@dt, interval 1 quarter); --加一季度
select date_add(@dt, interval 1 year );  --加一年
select date_add(@dt, interval -1 day ); -- 减一天

select date_add(@dt, interval '01:15:30' hour_second); --加01:15:30
select date_add(@dt, interval '1 01:15:30' day_second); --加1天01:15:30
```

2）MySQL 为日期减去一个时间间隔：`date_sub()`

```sql
set @dt=now();

select date_sub(@dt, interval '1 1:1:1' day_second); --减去1天1:1:1
```

3）MySQL 另类日期函数：`period_add(P,N)`, `period_diff(P1,P2)`

```sql
-- 函数参数“P” 的格式为“YYYYMM” 或者 “YYMM”，第二个参数“N” 表示增加或减去 N month（月）
select period_add(200808,2), period_add(20080808,-2) --日期加/减去2月
```

4）MySQL 日期、时间相减函数：`datediff(date1,date2)`, `timediff(time1,time2)`

```sql
select datediff( '2008-08-08' , '2008-08-01' ); -- 7 返回天数

select timediff( '2008-08-08 08:08:08' , '2008-08-08 00:00:00' ); -- 08:08:08 返回 time 差值
```

###  字符串截取

~~~sql
substring(event_time,1,7)：截取日期的年和月，表示截取1-7位，闭区间。
~~~

> 注意，左右都是闭区间