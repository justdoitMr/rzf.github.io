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

- 使用 with 建立临
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

