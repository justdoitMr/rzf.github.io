## Leetcode sql面试题目

### [175. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

~~~sql
# Write your MySQL query statement below
select
    FirstName,
    LastName,
    City,
    State
from Person
left join Address
on Person.PersonId = Address.PersonId;
~~~

多表的联结又分为以下几种类型：

1）左联结（left join），联结结果保留左表的全部数据

2）右联结（right join），联结结果保留右表的全部数据

3）内联结（inner join），取两表的公共数据

**图解多表join**

![1646120399451](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/01/153959-880648.png)

### [176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

**使用排序+limit方法**

~~~sql
# Write your MySQL query statement below

select ifNull(
(select distinct salary
from Employee 
order by salary Desc
limit 1,1),null
) as SecondHighestSalary ;
--注意列的别名，不按要求无法通过
~~~

**使用max()方法**

~~~sql
select ifNull(
    (
        select max(salary)
        from Employee
        where salary <
        (
            select
                max(salary)
            from Employee
        )
    ),null
)as SecondHighestSalary;
~~~

#### 如何查询第N高的数据

> 找出语文课中成绩第二高的学生成绩。如果不存在第二高成绩的学生，那么查询应返回 null。

1. **找出所有选修了“语文”课的学生成绩**

~~~sql
select * 
from 成绩表
where 课程='语文';
~~~

查找语文课程成绩的第二名

2. 考虑到成绩可能有一样的值，所以使用distinct 成绩进行成绩去重。

##### **思路1**：

使用子查询找出语文成绩查询最大的成绩记为a，然后再找出小于a的最大值就是课程成绩的第二高值。
max(列名) 可以返回该列的最大值，可以用下面的sql语句得到语文课的最大值：

~~~sql
select max(distinct 成绩) 
from 成绩表
where 课程='语文';
~~~

然后再找出小于a的最大值就是课程成绩的第二高值。

~~~sql
select max(distinct 成绩) 
from 成绩表
where 课程='语文' and
      成绩 < (select max(distinct 成绩) 
              from 成绩表 
              where 课程='语文');
~~~

##### 思路2：使用 limit 和 offset

- limit n子句表示查询结果返回前n条数据
- limit：a,b：表示从第a条数据开始，读取b条数据，包括数据a，下表从0开始。

- offset n表示跳过x条语句

- limit y offset x ：分句表示查询结果跳过 x 条数据，读取前 y 条数据,包括第x条数据。

使用limit和offset，降序排列再返回第二条记录可以得到第二大的值。

~~~sql
select distinct 成绩  
from 成绩表
where 课程='语文'
order by 课程,成绩 desc
limit 1,1;
~~~

**考虑特殊情况**

题目要求，如果没有第二高的成绩，返回空值，所以这里用判断空值的函数（ifnull）函数来处理特殊情况。

ifnull(a,b)函数解释：

- 如果value1不是空，结果返回a

- 如果value1是空，结果返回b

对于本题的sql就是：

~~~sql
select ifnull(第2步的sql,null) as '语文课第二名成绩';
~~~

我们把第2步的sql语句套入上面的sql语句，本题最终sql如下：

~~~sql
select ifnull(
(select max(distinct 成绩) from 成绩表
where 成绩<(select max(成绩) from 成绩表 where 课程='语文')
and 课程='语文')
,null) as '语文课第二名成绩';
~~~

### [177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

> 这种解法没有完全通过

~~~sql
select ifnull(
    (
        select
      distinct salary
      from Employee
      order by salary limit n,1
    ),null
)
~~~

### [178. 分数排名](https://leetcode-cn.com/problems/rank-scores/)

~~~sql
select
    score,
    dense_rank()  over(order by score desc) as 'rank'
from Scores;
~~~

#### ROW_NUMBER()

Row_number() 在排名是序号 连续 不重复，即使遇到表中的两个一样的数值亦是如此

~~~sql
select *,row_number() OVER(order by number ) as row_num
from num 
~~~

![1646124409797](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1646124409797.png)

> 记住row_numer就是进行编号操作。

注意：在使用row_number() 实现分页时需要特别注意一点，over子句中的order by 要与SQL排序记录中的order by保持一致，否则得到的序号可能不是连续的

~~~sql
select *,row_number() OVER(order by number ) as row_num
from num
ORDER BY id
~~~

![1646124481486](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/01/164802-357738.png)

#### rank()

Rank() 函数会把要求排序的值相同的归为一组且每组序号一样，排序不会连续执行

~~~sql
select *,rank() OVER(order by number ) as row_num
from num 
~~~

![1646124568186](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/01/164928-395332.png)

> rank排名总数据总个数不会减少

#### dense_rank()

Dense_rank() 排序是连续的，也会把相同的值分为一组且每组排序号一样

~~~sql
select *,dense_rank() OVER(order by number ) as row_num
from num 
~~~

![1646124626700](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/01/165027-440239.png)

> 排名连续。总数会减少

#### ntile()

Ntile(group_num) 将所有记录分成group_num个组，每组序号一样

~~~sql
select *,ntile(2) OVER(order by number ) as row_num
from num 
~~~

![1646124689362](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/01/165130-457268.png)

> 注意，ntile()里面的参数2表示数据分两组。

### [181. 超过经理收入的员工](https://leetcode-cn.com/problems/employees-earning-more-than-their-managers/)

~~~sql
select a.Name as Employee
from Employee as a
inner join Employee as b
on a.managerId = b.id
where  a.salary > b.salary;
~~~

- 单表进行内连接

### [182. 查找重复的电子邮箱](https://leetcode-cn.com/problems/duplicate-emails/)

~~~sql
select
    Email as Email
from Person
group by Email
having count(Email)>=2;
~~~

- 本质是查找重复项，首先分组，然后判断每一组内容的数量

### [183. 从不订购的客户](https://leetcode-cn.com/problems/customers-who-never-order/)

~~~sql
select
    Name as Customers 
from Customers
where id not in
(
    select
        CustomerId 
    from Orders
    group by CustomerId
);
~~~

### [184. 部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

#### 方法一

重点方法，可以保证相同的工资也可以选出来。

~~~sql
select 
    Department.name as Department , 
    Employee.name as Employee , 
    salary as Salary 
from Employee
inner join Department 
on Employee.departmentId = Department.id
where (departmentId, salary) 
in (
	select 
        departmentId, 
        max(salary) 
    from Employee 
    group by departmentId
);
~~~

#### 方法二

~~~sql
-- 使用窗口函数

with temp as (
    select 
        Department.name as Department, 
        Employee.name as Employee, 
        dense_rank() over(partition by Department.id order by Employee.salary desc) as rank
    from Employee
    inner join Department 
    on Employee.departmentId = Department.id
)select Department,Employee,salary from temp where rank = 1;
~~~

- 使用 with 建立临时表
- 连接两表 department 和 employee 使用 dense_rank() 对 salary 进行排序。partition by 的作用是分区

**第二种写法**

~~~sql
select
    d.Name as Department,
    m.Name as Employee,
    Salary
from (
    select
        Name,
        DepartmentId,
        Salary,
        rank() over( partition by DepartmentId order by Salary desc) as rk 
    from Employee
) as m left join Department d on m.DepartmentId = d.Id
where m.rk = 1
~~~

利用开窗函数可以取每个部门最高，也可以取前二高，前三高，也可以只取第一第三，都OK的

每一个部门前两高

~~~sql
-- 每个部门前2高
SELECT S.NAME, S.EMPLOYEE, S.SALARY
  FROM (SELECT D.NAME,
               T.NAME EMPLOYEE,
               T.SALARY,
               ROW_NUMBER() OVER(PARTITION BY T.DEPARTMENTID ORDER BY T.SALARY DESC) RN
          FROM EMPLOYEE T
          LEFT JOIN DEPARTMENT D
            ON T.DEPARTMENTID = D.ID) S
 WHERE S.RN <= 2
~~~

每一个部门第三稿

~~~sql
-- 每个部门第一第三高
SELECT S.NAME, S.EMPLOYEE, S.SALARY
  FROM (SELECT D.NAME,
               T.NAME EMPLOYEE,
               T.SALARY,
               ROW_NUMBER() OVER(PARTITION BY T.DEPARTMENTID ORDER BY T.SALARY DESC) RN
          FROM EMPLOYEE T
          LEFT JOIN DEPARTMENT D
            ON T.DEPARTMENTID = D.ID) S
 WHERE S.RN = 1 OR S.RN = 3
~~~



**使用dense_rank()窗口函数**

dense_rank 排名是密集连续，相同工资的排名是并列的 只取工资排名前三的员工

~~~sql
-- 首先按照部门进行开窗，然后按照工资进行排序
SELECT 
    Department, 
    Employee, 
    salary
FROM(
    SELECT 
    b.Name as 'Department',
    a.Name as 'Employee', 
    a.salary,
    dense_rank() over(partition by b.Name ORDER by a.salary DESC) as 'rank'
FROM Employee a 
left join Department b 
ON a.DepartmentId = b.Id
) temp
WHERE temp.rank <= 3
~~~

**第二种写法**

~~~sql
with temp as
(
    select
        Employee.departmentId as Department,
        Employee.name as Employee,
        salary,
        dense_rank() over(partition by Department.id order by Employee .salary desc) rank
    from Employee
    inner join Department
    on Employee.departmentId = Department.id
)select Department,Employee,salary from temp where rank <=3;
~~~

### 求topn问题

top n问题又要分组，有需要进行排序操作，所以可以使用开窗操作，按照某一个字段进行开窗操作，相当于分组，然后按照某一个字段进行排序操作，然后取出序号小于某个值的前几个数据即可。

**模板**

~~~sql
# topN问题 sql模板
select *
from (
   select *, 
          row_number() over (partition by 要分组的列名
          order by 要排序的列名 desc) as 排名
   from 表名) as a
where 排名 <= N;
~~~

### [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

> 练习删除操作

使用内连接，连接条件是表中电子邮件号码相等的，并且id大的。

~~~sql
delete p1
from Person p1
join Person p2
on p1.email = p2.email and p1.id>p2.id;
~~~

**思路二，使用开窗函数**

根据题意可知该题希望保留的是同一邮箱下Id较小的那条数据，反之就是需删除同一邮箱下Id较大的数据条目，这里使用开窗函数来解决：

- 针对同一邮箱，对其Id序号进行排序rn；
- 依据题意，只保留排序rn=1的数据，其余数据均需删除

~~~sql
-- 首先按照email进行离开床，然后按照id进行排序
select 
    id,
    dense_rank() over(partition by email order by id desc)
from Person

//然后选取id大于1的那些名单id
select
    id
from
(
    select 
        id,
        dense_rank() over(partition by email order by id desc) as rank
    from Person
)where rank >1

-- 总的sql

delete 
from Person
where Id in
(
    select Id
    from
        (
            select Id,
                row_number() over(partition by Email order by Id) rn
            from Person
        ) t1
    where rn>1
)
--下面sql没走桶
delete from Person 
where id in
(
select
    id
from
(
        select 
            id,
            row_number() over(partition by email order by id) as rank
        from Person
    )temp
    where rank >1
)
~~~

### [197. 上升的温度](https://leetcode-cn.com/problems/rising-temperature/)

**思路一：使用内连接**

只有一张表，现在要找出今天温度比昨天温度高的日期 id 。

所以需要用**自连接**，也就是把 weather 和 weather 进行自身连接。

在自连之后，需要将自连后的表取个别名 w1 ，如果不取别名的话，两个 weather 表名会冲突。这里把 weather 作为今天表， w1 作为昨天表。

**两表自连之后需要有连接条件，连接条件是 今天和昨天的日期**。

MySQL 提供了datediff 函数，用来比较两个日期之间的时间差，如果两个时间之间相差 1 天，那么就是今天和做题。

~~~sql
select
    w1.id
from Weather w1
inner join Weather w2
--下面是子链接条件
on datediff(w1.recordDate,w2.recordDate)=1
and w1.Temperature > w2.Temperature;
~~~

**思路二：使用date_add()函数**

思路和方法一的思路是一样的，区别在于计算今天和昨天的方法不一样。

这里使用 MySQL 提供的 `adddate` 函数。这个函数是将日期函数一个规律进行偏移。	

~~~sql
select weather.id from weather join weather w1 
on weather.recordDate = adddate(w1.recordDate, interval 1 day) 
and weather.temperature > w1.temperature;

-- 使用自联结	
select
    w1.id
from Weather w1
inner join Weather w2
on w1.recordDate=adddate(w1.recordDate, interval 1 day) 
and w1.Temperature > w2.Temperature;
~~~

`adddate(w1.recordDate, interval 1 day)`会遍历表中的每一条数据，然后对日期添加1天，

这样的话，w2表中如果有一条数据+1和w1表中的时间一样，那么说明w2中 的这一条数据是w1表中的前一条数据。

**方法三**

使用lead()函数。使用窗口函数 `lead` ，它是从后往前偏移，偏移量为 `1` 天。

~~~sql
select id from (
	select
	temperature, 
	recordDate ,
	lead(id, 1) over(order by recordDate) as id,
	lead(recordDate, 1) over(order by recordDate) as 'nextDate',
	lead(temperature, 1) over(order by recordDate) as 'nextTemp'
	from weather
) temp
where nextTemp > temperature and datediff(nextDate, recordDate) = 1;
~~~

![1647416616924](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202203/16/154337-305590.png)

~~~sql
select
	temperature, 
	recordDate ,
	lead(id, 1) over(order by recordDate) as nextId,
	lead(recordDate, 1) over(order by recordDate) as 'nextDate',
	lead(temperature, 1) over(order by recordDate) as 'nextTemp'
from weather;
~~~

这里说一下，窗口函数还有一个 lag 是从前往后偏移的，用法和 lead 是一样的。这里就用 lead 来举例。

前三列是 weather 原数据，后三列是使用窗口函数 lead 算出来的数据。

为什么是偏移 1 呢？

因为比较的是今天和昨天，而且这里日期是连续的，所以用 1 。

然后查询出来的数据作为一个临时表 temp 。

筛选条件就是 nextTemp > temperature ，最后使用 datediff 比较两个日期差可写可不写，因为这里日期是连续的。

### mysql中的lead()函数和lag()函数

#### lead()函数

返回后面的若干行。

`LEAD()`函数是一个[窗口函数](https://www.begtut.com/mysql/mysql-window-functions.html)，允许您向前看多行并从当前行访问行的数据。

与`LAG()`函数类似，`LEAD()`函数对于计算同一结果集中当前行和后续行之间的差异非常有用。

以下显示了`LEAD()`函数的语法：

~~~sql
LEAD(<expression>[,offset[, default_value]]) OVER (
    PARTITION BY (expr)
    ORDER BY (expr)
) 
~~~

##### expression

`LEAD()`函数返回的值`expression`从`offset-th`有序分区排。

##### offset

`offset`是从当前行向前行的行数，以获取值。

`offset`必须是一个非负整数。如果`offset`为零，则`LEAD()`函数计算`expression`当前行的值。

如果省略  `offset`，则`LEAD()`函数默认使用一个。

##### default_value

如果没有后续行，则`LEAD()`函数返回`default_value`。例如，如果`offset`是1，则最后一行的返回值为`default_value`。

如果您未指定`default_value`，则函数返回  `NULL` 。

##### PARTITION BY子句

`PARTITION BY`子句将结果集中的行划分`LEAD()`为应用函数的分区。

如果`PARTITION BY`未指定子句，则结果集中的所有行都将被视为单个分区。

##### ORDER BY子句

`ORDER BY`子句确定`LEAD()`应用函数之前分区中行的顺序。

#### lag()函数

`LAG()`函数是一个[窗口函数](https://www.begtut.com/mysql/mysql-window-functions.html)，允许您回顾多行并从当前行访问行的数据。

以下说明了`LAG()`函数的语法：

~~~sql
LAG(<expression>[,offset[, default_value]]) OVER (
    PARTITION BY expr,...
    ORDER BY expr [ASC|DESC],...
) 
~~~

expression

`LAG()`函数返回`expression`当前行之前的行的值，其值为`offset` 其分区或结果集中的行数。

##### offset

`offset`是从当前行返回的行数，以获取值。`offset`必须是零或文字正整数。如果`offset`为零，则`LAG()`函数计算`expression`当前行的值。如果未指定`offset`，则`LAG()`默认情况下函数使用一个。

##### default_value

如果没有前一行，则`LAG()`函数返回`default_value`。例如，如果offset为2，则第一行的返回值为`default_value`。如果省略`default_value`，则默认`LAG()`返回函数`NULL`。

##### `PARTITION BY` 子句

`PARTITION BY`子句将结果集中的行划分`LAG()`为应用函数的分区。如果省略`PARTITION BY`子句，`LAG()`函数会将整个结果集视为单个分区。

##### `ORDER BY` 子句

`ORDER BY`子句指定在`LAG()`应用函数之前每个分区中的行的顺序。

`LAG()`函数可用于计算当前行和上一行之间的差异。

### [511. 游戏玩法分析 I](https://leetcode-cn.com/problems/game-play-analysis-i/)

使用窗口函数，按照id分组，然后排序，取出第一个时间即可。

~~~sql
select
    player_id,
    event_date as first_login 
from
(
    select
        player_id,
        event_date,
        row_number() over(partition by player_id order by event_date) as num
    from Activity
)temp
where num =1;  
~~~

**思路二**

~~~sql
select player_id, min(event_date) as first_login from activity
group by player_id;
~~~

- 按照 `player_id` 将 `activity` 分组
- 使用 `min` 函数，求出日期的最小值

### [569. 员工薪水中位数](https://leetcode-cn.com/problems/median-employee-salary/)

#### 思路一

- 使用 row_number() 计算排名，并按照 company 分组， salary 升序
- 按照 company 分组，并计算总数
- 现在只需要筛选出中位数就可以了
  - 筛选条件 floor((total + 1) / 2) 和 floor((total + 2) / 2) ， floor 是想下取整
    - 当 total = 6，中位数是 3 和 4 ，这里计算的结果正是 3 和 4
    - 当 total = 5，中位数是 3，这里计算的两个值分别是 3 和 3
  - 筛选条件也可以使用 where 排名 >= total / 2 and 排名 <= total / 2 + 1
    - 当 total = 6，中位数是 3 和 4 ， 排名 ≥ 3 and 排名 ≤ 4 ，筛选出来的是 3 和 4
    - 当 total = 5，中位数是 3 ， 排名 ≥ 2.5 and 排名 ≤ 3.5 ，筛选出来的就是 3

~~~sql
select id, company, salary from (
	select
		id, company, salary,
		row_number() over(partition by company order by salary) as 排名,
		count(id) over(partition by company) as total
	from employee
) as temp
where temp.排名 in (floor((total + 1) / 2), floor((total + 2) / 2));
~~~

**第二种写法**

~~~sql
select  
    id,
    company,
    salary
from
(
    select
        id,
        company,
        salary,
        row_number() over(partition by company order by salary) as nk,
        count(id) over(partition by company)as total
    from Employee
)temp
where nk>=total/2 and nk <=total/2+1;
~~~

