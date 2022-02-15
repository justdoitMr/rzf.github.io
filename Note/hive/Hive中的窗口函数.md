## Hive中的窗口函数

### 什么是窗口函数

窗口函数是 SQL 中一类特别的函数。和聚合函数相似，窗口函数的输入也是多行记录。不 同的是，**聚合函数的作用于由 GROUP BY 子句聚合的组，而窗口函数则作用于一个窗口**， 这里，窗口是由一个 OVER 子句 定义的多行记录。

聚合函数对其所作用的每一组记录输出一条结果，而窗口函数对其所作用的窗口中的每一行记录输出一条结果。一些聚合函 数，如 sum, max, min, avg,count 等也可以当作窗口函数使用。

### 窗口函数的实现原理

在用group-by处理数据分组时，每一行只能进入一个分组。窗口函数基于称为框(f r a m e)的一组行，计算表的每一输入行的返回值，每一行可以属于一个或多个框。

常见用例就是查看某些值的滚动平均值，其中每一行代表一天，那么每行属于7个不同的框。

如下图所示，每一行是如何匹配多个窗口框的。

![1644732939916](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/141541-676041.png)

### 窗口函数的使用场景

1. 分组排序，如取某年级每个班学习成绩排名前10的学生。
2. 分组聚合

### 基本语法

![1644732986165](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/141626-33441.png)

窗口函数的语法分为**四个部分**：

**函数子句**：指明具体操作，如sum-求和，first_value-取第一个值；

**partition by子句**：指明分区字段，如果没有，则将所有数据作为一个分区；

**order by子句**：指明了每个分区排序的字段和方式,也是可选的，没有就是按照表中的顺序；

**窗口子句**：指明相对当前记录的计算范围，可以向上（preceding），可以向下（following）,也可以使用between指明，上下边界的值，没有的话默认为当前分区。

**ROWS BETWEEN，也叫做window子句**数字+PRECEDING 向前n条数字+FOLLOWING 向后n条+CURRENT ROW 当前行UNBOUNDED 无边界，表示从最前面的起点开始，表示到最后面的终点UNBOUNDED PRECEDING 向前无边界UNBOUNDED FOLLOWING 向后无边界。

### 窗口函数有那些

窗口函数的功能分为：**聚合**、**取值**、**排名**、**序列**四种，前三种的使用场景比较常见，容易理解，最后一种(序列)的使用场景比较少。

**聚合**

- count 统计条数
- sum 求和
- avg 求平均值
- max 求最大值
- min 求最小值

**取值**

- **first_value 取窗口中的第一值**
- **last_value 取窗口中的最后一个值**
- **lag(col, n, DEFAULT)** 用于统计窗口内向上第n行的值
  - col ：列名 
  - n：向上n行，[可选，默认为1] 
  - DEFAULT ：当向上n行为NULL时，取默认值；如果不指定，则为NULL。
- **lead(col, n, DEFAULT)** 用于统计窗口内向下第n行的值，和lag相反
  - col ：列名
  -  n：向下n行，[可选，默认为1]
  -  DEFAULT ：当向上n行为NULL时，取默认值；如果不指定，则为NULL

**排序**

- **rank** 排序**，**有相同分数，排名相同并对后续跳过，如分数5，5，8，9，则得到的结果未1，1，3，4
- **dense_rank** 排序，有相同的分数排名相同，但后续接上，如分数5，5，8，9，则得到的排序结果未1，1，2，3**、**
- **row_number** 排序，相同分数按先来后到排序，无重复排序，如分数5，5，8，9，得到的结果为1，2，3，4

### 实战

**在深入研究Over字句之前，一定要注意：在SQL处理中，窗口函数都是最后一步执行，而且仅位于Order by字句之前。**

#### 数据准备

我们准备一张order表,字段分别为name,orderdate,cost.数据内容如下:

~~~text
jack,2015-01-01,10
tony,2015-01-02,15
jack,2015-02-03,23
tony,2015-01-04,29
jack,2015-01-05,46
jack,2015-04-06,42
tony,2015-01-07,50
jack,2015-01-08,55
mart,2015-04-08,62
mart,2015-04-09,68
neil,2015-05-10,12
mart,2015-04-11,75
neil,2015-06-12,80
mart,2015-04-13,94
~~~

#### 聚合函数+over()

假如说我们想要查询在2015年4月份购买过的顾客及总人数,我们便可以使用窗口函数去去实现:

~~~sql
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'

-- 结果
name    count_window_0
mart    5
mart    5
mart    5
mart    5
jack    5
-- 5表示有5次购买记录
~~~

可见其实在2015年4月一共有5次购买记录,mart购买了4次,jack购买了1次.事实上,大多数情况下,我们是只看去重后的结果的.针对于这种情况,我们有两种实现方式

**第一种，使用distinct**

~~~sql
select distinct name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
~~~

**第二种，使用group by**

~~~sql
select name,count(*) over ()
from t_window
where substring(orderdate,1,7) = '2015-04'
group by name

-- 执行结果
name count_window_0 
mart 2 
jack 2
-- 后面的表示购买记录的次数
~~~

#### partition by子句

Over子句之后第一个提到的就是Partition By.Partition By子句也可以称为**查询分区子句**，非常类似于Group By，都是将数据按照边界值分组，**而Over之前的函数在每一个分组之内进行**，如果超出了分组，则函数会重新计算.

实例

我们想要去看顾客的购买明细及月购买总额,可以执行如下的sql：

~~~sql
select name,orderdate,cost,sum(cost) over(partition by month(orderdate))
from t_window

-- 执行结果
name    orderdate   cost    sum_window_0
jack    2015-01-01  10  205
jack    2015-01-08  55  205
tony    2015-01-07  50  205
jack    2015-01-05  46  205
tony    2015-01-04  29  205
tony    2015-01-02  15  205
jack    2015-02-03  23  23
mart    2015-04-13  94  341
jack    2015-04-06  42  341
mart    2015-04-11  75  341
mart    2015-04-09  68  341
mart    2015-04-08  62  341
neil    2015-05-10  12  12
neil    2015-06-12  80  80
--按照month进行分区，在month单位分区内部将金额进行累加即可
~~~

#### order By子句

上述的场景,假如我们想要将cost按照月进行累加.这时我们引入order by子句.

order by子句会让输入的数据强制排序（文章前面提到过，窗口函数是SQL语句最后执行的函数，因此可以把SQL结果集想象成输入数据）。Order By子句对于诸如Row_Number()，Lead()，LAG()等函数是必须的，因为如果数据无序，这些函数的结果就没有任何意义。因此如果有了Order By子句，则Count()，Min()等计算出来的结果就没有任何意义。

我们在上面的代码中加入order by：

~~~sql
select name,orderdate,cost,sum(cost) over(partition by month(orderdate) order by orderdate )
from t_window

-- 得到的结果如下：(order by默认情况下聚合从起始行到当前行的数据)

name    orderdate   cost    sum_window_0
jack    2015-01-01  10  10
tony    2015-01-02  15  25 //10+15
tony    2015-01-04  29  54 //10+15+29
jack    2015-01-05  46  100 //10+15+29+46
tony    2015-01-07  50  150
jack    2015-01-08  55  205
jack    2015-02-03  23  23
jack    2015-04-06  42  42
mart    2015-04-08  62  104
mart    2015-04-09  68  172
mart    2015-04-11  75  247
mart    2015-04-13  94  341
neil    2015-05-10  12  12
neil    2015-06-12  80  80

-- 窗口是按照月进行分组，然后把每一个窗口内部的数据进行累加即可
~~~

#### window子句

我们在上面已经通过使用partition by子句将数据进行了分组的处理.如果我们想要更细粒度的划分，我们就要引入window子句了.

我们首先要理解两个概念: 

- **如果只使用partition by子句,未指定order by的话,我们的聚合是分组内的聚合**. 
- **使用了order by子句,未使用window子句的情况下,默认从起点到当前行**.

**当同一个select查询中存在多个窗口函数时,他们相互之间是没有影响的.每个窗口函数应用自己的规则.**

window子句： 
- PRECEDING：往前 ，preceding
- FOLLOWING：往后 ,following
- CURRENT ROW：当前行 ,current row
- UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING：表示到后面的终点

我们按照name进行分区,按照购物时间进行排序,做cost的累加. 

如下我们结合使用window子句进行查询:

~~~sql
select name,orderdate,cost,
sum(cost) over() as sample1,--所有行相加，也就是所有cost进行累加，因为没有指定分组字段
sum(cost) over(partition by name) as sample2,--按name分组，组内数据相加，因为没有指定order by子句
sum(cost) over(partition by name order by orderdate) as sample3,--按name分组，组内数据累加，因为执行了order by子句
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING and current row )  as sample4 ,--和sample3一样,由起点到当前行的聚合，这里是累加聚合，因为指定了order by
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   and current row) as sample5, --当前行和前面一行做聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING   AND 1 FOLLOWING  ) as sample6,--当前行和前边一行及后面一行
sum(cost) over(partition by name order by orderdate rows between current row and UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
from t_window;

-- 执行结果

name    orderdate   cost    sample1 sample2 sample3 sample4 sample5 sample6 sample7
jack    2015-01-01  10  661 176 10  10  10  56  176
jack    2015-01-05  46  661 176 56  56  56  111 166
jack    2015-01-08  55  661 176 111 111 101 124 120
jack    2015-02-03  23  661 176 134 134 78  120 65
jack    2015-04-06  42  661 176 176 176 65  65  42
mart    2015-04-08  62  661 299 62  62  62  130 299
mart    2015-04-09  68  661 299 130 130 130 205 237
mart    2015-04-11  75  661 299 205 205 143 237 169
mart    2015-04-13  94  661 299 299 299 169 169 94
neil    2015-05-10  12  661 92  12  12  12  92  92
neil    2015-06-12  80  661 92  92  92  92  92  80
tony    2015-01-02  15  661 94  15  15  15  44  94
tony    2015-01-04  29  661 94  44  44  44  94  79
tony    2015-01-07  50  661 94  94  94  79  79  50
~~~

#### 窗口函数中的序列函数

主要序列函数是不支持window子句的.

hive中常用的序列函数有下面几个:

##### NTILE

- NTILE(n)，用于将分组数据按照顺序切分成n片，返回当前切片值
- **NTILE不支持ROWS BETWEEN**， 
  - 比如 NTILE(2) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW)
- 如果切片不均匀，默认增加第一个切片的分布

这个函数用什么应用场景呢?假如我们想要每位顾客购买金额前1/3的交易记录,我们便可以使用这个函数.

~~~sql
select name,orderdate,cost,
       ntile(3) over() as sample1 , --全局数据切片
       ntile(3) over(partition by name), -- 按照name进行分组,在分组内将数据切成3份
       ntile(3) over(order by cost),--全局按照cost升序排列,数据切成3份
       ntile(3) over(partition by name order by cost ) --按照name分组，在分组内按照cost升序排列,数据切成3份
from t_window

-- 执行结果
name    orderdate   cost    sample1 sample2 sample3 sample4
jack    2015-01-01  10  3   1   1   1
jack    2015-02-03  23  3   1   1   1
jack    2015-04-06  42  2   2   2   2
jack    2015-01-05  46  2   2   2   2
jack    2015-01-08  55  2   3   2   3
mart    2015-04-08  62  2   1   2   1
mart    2015-04-09  68  1   2   3   1
mart    2015-04-11  75  1   3   3   2
mart    2015-04-13  94  1   1   3   3
neil    2015-05-10  12  1   2   1   1
neil    2015-06-12  80  1   1   3   2
tony    2015-01-02  15  3   2   1   1
tony    2015-01-04  29  3   3   1   2
tony    2015-01-07  50  2   1   2   3
~~~

##### 排序函数

- row_number
- rank
- dense_rank

这三个窗口函数的使用场景非常多 

- row_number()从1开始，按照顺序，生成分组内记录的序列,row_number()的值不会存在重复,当排序的值相同时,按照表中记录的顺序进行排列 
- RANK() 生成数据项在分组中的排名，排名相等会在名次中留下空位 ，数据总行数不变
- DENSE_RANK() 生成数据项在分组中的排名，排名相等会在名次中不会留下空位，数据总行数减少。

**注意： 
rank和dense_rank的区别在于排名相等时会不会留下空位.**

~~~sql
SELECT 
  cookieid,
  createtime,
  pv,
  RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn1,
  DENSE_RANK() OVER(PARTITION BY cookieid ORDER BY pv desc) AS rn2,
  ROW_NUMBER() OVER(PARTITION BY cookieid ORDER BY pv DESC) AS rn3 
FROM lxw1234 
WHERE cookieid = 'cookie1';
 
cookieid day           pv       rn1     rn2     rn3 
 
cookie1 2015-04-12      7       1       1       1
cookie1 2015-04-11      5       2       2       2
cookie1 2015-04-15      4       3       3       3
cookie1 2015-04-16      4       3       3       4
cookie1 2015-04-13      3       5       4       5
cookie1 2015-04-14      2       6       5       6
cookie1 2015-04-10      1       7       6       7

rn1: 15号和16号并列第3, 13号排第5
rn2: 15号和16号并列第3, 13号排第4
rn3: 如果相等，则按记录值排序，生成唯一的次序，如果所有记录值都相等，或许会随机排吧。
~~~

##### LAG和LEAD函数

- 这两个函数为常用的窗口函数,可以返回上下数据行的数据. 
- 以我们的订单表为例,假如我们想要查看顾客上次的购买时间可以这样去查询

~~~sql
select name,orderdate,cost,
lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate ) as time1,
lag(orderdate,2) over (partition by name order by orderdate) as time2
from t_window;

-- 执行结果
name    orderdate   cost    time1   time2
jack    2015-01-01  10  1900-01-01  NULL
jack    2015-01-05  46  2015-01-01  NULL
jack    2015-01-08  55  2015-01-05  2015-01-01
jack    2015-02-03  23  2015-01-08  2015-01-05
jack    2015-04-06  42  2015-02-03  2015-01-08
mart    2015-04-08  62  1900-01-01  NULL
mart    2015-04-09  68  2015-04-08  NULL
mart    2015-04-11  75  2015-04-09  2015-04-08
mart    2015-04-13  94  2015-04-11  2015-04-09
neil    2015-05-10  12  1900-01-01  NULL
neil    2015-06-12  80  2015-05-10  NULL
tony    2015-01-02  15  1900-01-01  NULL
tony    2015-01-04  29  2015-01-02  NULL
tony    2015-01-07  50  2015-01-04  2015-01-02
~~~

time1取的为按照name进行分组,分组内升序排列,**取上一行数据的值**，见下图。

time2取的为按照name进行分组，分组内升序排列,**取上面2行的数据的值**,注意当lag函数未设置行数值时,默认为1行.设定取不到时的默认值时,取null值.

lead函数与lag函数方向相反,取向下的数据.

![1644734945481](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/144907-658918.png)

##### first_value和last_value

- first_value取分组内排序后，截止到当前行，第一个值 
- last_value取分组内排序后，截止到当前行，最后一个值

~~~sql
select name,orderdate,cost,
first_value(orderdate) over(partition by name order by orderdate) as time1,
last_value(orderdate) over(partition by name order by orderdate) as time2
from t_window

-- 执行结果
name    orderdate   cost    time1   time2
jack    2015-01-01  10  2015-01-01  2015-01-01
jack    2015-01-05  46  2015-01-01  2015-01-05
jack    2015-01-08  55  2015-01-01  2015-01-08
jack    2015-02-03  23  2015-01-01  2015-02-03
jack    2015-04-06  42  2015-01-01  2015-04-06
mart    2015-04-08  62  2015-04-08  2015-04-08
mart    2015-04-09  68  2015-04-08  2015-04-09
mart    2015-04-11  75  2015-04-08  2015-04-11
mart    2015-04-13  94  2015-04-08  2015-04-13
neil    2015-05-10  12  2015-05-10  2015-05-10
neil    2015-06-12  80  2015-05-10  2015-06-12
tony    2015-01-02  15  2015-01-02  2015-01-02
tony    2015-01-04  29  2015-01-02  2015-01-04
tony    2015-01-07  50  2015-01-02  2015-01-07
~~~

#### 扩展

**row_numbe**r的用途非常广泛，排序最好用它，它会为查询出来的每一行记录生成一个序号，依次排序且不会重复，注意使用row_number函数时必须要用over子句选择对某一列进行排序才能生成序号。

**rank**函数用于返回结果集的分区内每行的排名，行的排名是相关行之前的排名数加一。简单来说rank函数就是对查询出来的记录进行排名，与row_number函数不同的是，rank函数考虑到了over子句中排序字段值相同的情况，如果使用rank函数来生成序号，over子句中排序字段值相同的序号是一样的，后面字段值不相同的序号将跳过相同的排名号排下一个，也就是相关行之前的排名数加一，可以理解为根据当前的记录数生成序号，后面的记录依此类推。

**dense_rank**函数的功能与rank函数类似，dense_rank函数在生成序号时是连续的，而rank函数生成的序号有可能不连续。dense_rank函数出现相同排名时，将不跳过相同排名号，rank值紧接上一次的rank值。在各个分组内，rank()是跳跃排序，有两个第一名时接下来就是第四名，dense_rank()是连续排序，有两个第一名时仍然跟着第二名。

##### 案例

**原始数据**

![1644735206471](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/145328-431756.png)

**现在需要按照课程对学生的成绩进行排序：**

~~~sql
--row_number() 顺序排序
select name,course,row_number() over(partition by course order by score desc) rank from student;
~~~

![1644735254164](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/145414-43255.png)

~~~sql
--rank() 跳跃排序，如果有两个第一级别时，接下来是第三级别
select name,course,rank() over(partition by course order by score desc) rank from student;
~~~

![1644735277698](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/145438-183396.png)

**取得每门课程的第一名：**

~~~sql
--每门课程第一名只取一个： 
select * from (select name,course,row_number() over(partition by course order by score desc) rank from student) where rank=1;
--每门课程第一名取所有： 
select * from (select name,course,dense_rank() over(partition by course order by score desc) rank from student) where rank=1;
--每门课程第一名取所有：
select * from (select name,course,rank() over(partition by course order by score desc) rank from student) where rank=1;
~~~

**关于Parttion by：**

Parttion by关键字是Oracle中分析性函数的一部分，用于给结果集进行分区。它和聚合函数Group by不同的地方在于它只是将原始数据进行名次排列，能够返回一个分组中的多条记录（记录数不变），而Group by是对原始数据进行聚合统计，**一般只有一条反映统计值的结果（每组返回一条）**。

TIPS：

使用rank over()的时候，空值是最大的，如果排序字段为null, 可能造成null字段排在最前面，影响排序结果。

可以这样： rank over(partition by course order by score desc nulls last)

**总结：**

在使用排名函数的时候需要注意以下三点：

1. 排名函数必须有 OVER 子句。
2. 排名函数必须有包含 ORDER BY 的 OVER 子句。
3. 分组内从1开始排序。