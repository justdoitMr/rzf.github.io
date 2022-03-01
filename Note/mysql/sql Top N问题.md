## sql Top N问题

### 关联子查询

**取最大值或者最小值**

~~~sql
SELECT * 
FROM score AS t1 
WHERE 成绩 = (SELECT max/min(成绩) 
			 FROM score AS t2 
			 WHERE t1.课程号 = t2.课程号);
~~~

**取前两名**

~~~sql
SELECT *
FROM score t1
WHERE (SELECT COUNT( * ) 
	   FROM score t2 
	   WHERE t1.课程号 = t2.课程号 AND t1.成绩 < t2.成绩 ) < 2
ORDER BY t1.课程号, t1.成绩 DESC; #排序便于观察
~~~

**或者使用union all**

~~~sql
(SELECT * FROM score WHERE 课程号 = '0001' ORDER BY 成绩  DESC LIMIT 2)
UNION ALL
(SELECT * FROM score WHERE 课程号 = '0002' ORDER BY 成绩  DESC LIMIT 2)
UNION ALL
(SELECT * FROM score WHERE 课程号 = '0003' ORDER BY 成绩  DESC LIMIT 2);
~~~

### 窗口函数

根据需求选取ROW_NUMBER() OVER()、RANK() OVER()、DENSE_RANK() OVER()函数：

1. ROW_NUMBER() OVER()：依次排序且不重复，不关注值
2. RANK() OVER()：排序字段值相同的序号相同，且跳过相同的排名号，跳跃排序。例如[100,100,90]，则对应的排名分别为1,1,3。
3. DENSE_RANK() OVER()：排序字段值相同的序号相同，不跳过相同的排名号，连续排序。例如[100,100,90]，则对应的排名分别为1,1,2。

**top k万能模板**

~~~sql
#【TOPN万能模板】
SELECT * 
FROM (SELECT *, 
			 row_number() OVER (PARTITION BY 要分组的列名 ORDER BY 要排序的列名 DESC) AS ranking 
	  FROM 表名)  AS t  
WHERE ranking <= N;
~~~

**查找每个学生成绩最高的两个科目**

~~~sql
SELECT * 
FROM (SELECT *, 
			 row_number() OVER (PARTITION BY 学号 ORDER BY 成绩 DESC) AS ranking 
	  FROM score)  AS t1  
WHERE ranking <= 2;
~~~

### limit语句

**查找工资排前N的员工（不分组）**

~~~sql
# LeetCode 176-177题
Select ifnull(
          (SELECT DISTINCT Salary 
           from Employee 
           order by Salary desc 
           limit N-1,1) , null));
~~~

#### limit用法

获取前n条数据

limit关键字用来在查询结果集中，选择指定的行返回，常常用来实现翻页功能。

**例1**

~~~sql
select * from table limit 10;	// limit n;		返回查询结果的前n条数据
//等同于
select * from table limit 0,10;	//limit offset, n;	返回从offset + 1开始的n条数据
~~~

**例2：limit 和 offset**

~~~sql
//从数据库中第三条开始查询，取一条数据，即第三条数据读取，一二条跳过
selete * from testtable limit 2,1;

//是从数据库中的第二条数据开始查询两条数据，即第二条和第三条。
selete * from testtable limit 2 offset 1;
~~~

**注意：**

1. 数据库数据计算是从0开始的
2. `offset` X是跳过X个数据，limit Y是选取Y个数据
3. limit X,Y 中X表示跳过X个数据，读取Y个数据