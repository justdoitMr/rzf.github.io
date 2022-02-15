## 牛客网sql练习题目

### Sql大厂面试真题

#### **SQL1** 各个视频的平均完播率

~~~sql
SELECT 
    tb_user_video_log.video_id,
    round(sum(if(end_time - start_time>=tb_video_info.duration,1,0))/count(tb_user_video_log.video_id),3) as avg_ratio
from tb_user_video_log 
LEFT join tb_video_info 
on tb_user_video_log.video_id = tb_video_info.video_id
WHERE year(start_time)=2021
group by tb_user_video_log.video_id
order by avg_ratio desc;
~~~

round()函数：保留几位小数使用，本题目设置保留三位小数。

### Mysql开窗函数使用

窗口函数语法：其中[]中的内容可以省略

```sql
<窗口函数> over ([partition by <列清单>] order by <排序用列清单>)
```

窗口函数大体可以分为以下两种：

1. 能够作为窗口函数的聚合函数（sum，avg，count，max，min）
2. rank，dense_rank。row_number等专用窗口函数。

partition by 能够设定排序的对象范围，类似于group by语句，这里就是以product_type划分排序范围。

order by能够指定哪一列，何种顺序进行排序。也可以通过asc，desc来指定升序降序。

窗口函数兼具**分组和排序**两种功能。通过partition by分组后的记录集合称为窗口。

然而partition by不是窗口函数所必须的

由于窗口函数无需参数，因此通常括号里都是空的。

窗口函数的适用范围：只能在select子句中使用。

作为窗口函数使用的聚合函数：

sum：以累计的计算方式进行计算。

avg:以累计的方式计算窗口内部的平均。

#### 窗口函数的描述

窗口函数作用于一个**数据行集合**。窗口是标准的SQL术语，**用来描述SQL语句内OVER子句划定的内容**，这个内容就是**窗口函数的作用域**。而在OVER子句中，定义了窗口所覆盖的与当前行相关的数据行集、行的排序及其他的相关元素。

#### 窗口函数中的元素

**窗口函数的行为描述出现在函数的OVER子句中**，并涉及多个元素。3个核心元素分别是**分区、排序和框架**。不是所有的窗口函数都支持这3个元素。

##### 分区

分区元素是由PARTITION BY子句定义，并被所有的窗口函数支持。他对当前计算的窗口函数进行限制，仅仅那些在结果集的分区列中与当前行有相同值的行才能进入窗口。如果没有指定PARTITION BY子句，窗口就没有限制。

换种说法就是：如果没有显示指定分区，则默认分区就是把整个查询结果集当作一个分区。有一点不太明显，这里提出来：同一个查询中的不同函数，可能会有不同的分区描述。

##### 排序

排序元素定义计算的顺序，如果与分区有关，则是在分区内的顺序。在标准的SQL中，所有函数都支持排序元素。起初SQL SERVER不支持聚合函数中的排序元素，而仅仅支持分区。对聚合函数红排序的支持，是从SQL SERVER 2012 开始的。

有趣的是，针对不同的函数类别，排序元素有轻微的不同意义。对于排名函数，排序是直观的。而聚合窗口函数的排序和排名窗口的排序略有意义上的不同。在聚合中，与某些人认为的相反，排序与聚合的顺序无关；然而，排序元素为下面将描述的框架选项赋予一定的含义，换句话说，排序元素帮助限定在窗口内的行。

##### 框架

从本质上来说，框架是一个在分区内对行进行进一步限制的筛选器。它适用于聚合窗口函数，也适用于三个偏移函数：FIRST_VALUE、LAST_VALUE、NTH_VALUE。把这个窗口元素想成是基于给定的排序，在当前行所在分区中定义两个点，这两个点形成的框架之间的行才会参与计算。

在标准的框架描述中，包含一个ROWS或RANGE选项，用来定义框架的开始行和结束行，这两行也可以形成“框架外”（框架内的行被排除在计算外）窗口选项。SQL SERVER 2012 开始支持框架，完全实现ROWS选项，部分实现RANGE选项，尚未实现“框架外”窗口选项。

ROWS选项允许我们用相对当前行的偏移行数来指定框架的起点和终点。RANGE选项更具灵活性，可以以框架起终点的值于当前行的值的差异来定义偏移行数。“框架外”窗口选项用来定义如何对当前行及具有相同值的行进行处置。

#### 支持窗口元素的查询函数

并不是所有的查询子句都支持窗口函数，相反，仅仅SELECT和ORDER BY 子句支持窗口函数。为帮助理解，我们先看看SQL不同子句的执行顺序：

1、FROM

2、WHERE

3、GROUP BY

4、HAVING

5、SELECT

 5.1、Evalute Expressions（判断表达式）

 5.2、删除重复数据

6、ORDER BY

7、OFFSET-FETCH/TOP

只有SELECT和ORDER BY 子句直接支持窗口函数。做这个限制的原因是为了避免二义性，因此把（几乎是）查询的最终结果当作窗口的起点。如果窗口函数可以早于SELECT阶段出现，那么通过一些查询表单会无法得到正确的结果。

Mysql 开窗函数在Mysql8.0+ 中可以得以使用，实在且好用。

- row number() over
- rank() over
- dense rank()
- ntile() 

**测试数据**

~~~sql
/*测试数据*/
CREATE TABLE `school_score` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` char(1) DEFAULT NULL,
    `course` char(10) DEFAULT NULL,
  `score`  int (2) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ;

INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (1, 'A','Chinese',80);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (2, 'B','Chinese',90);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (3, 'C','Chinese',70);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (4, 'A','Math',70);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (5, 'B','Math',100);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (6, 'C','Math',80);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (7, 'A','English',90);
INSERT INTO `test`.`school_score`(`id`, `name`,`course`,`score`) VALUES (8, 'B','English',85);
INSERT INTO `test`.`school_score` (`id`, `name`,`course`,`score`) VALUES (9, 'C','English',99);
~~~

##### row_number()函数

~~~sql
-- row number() over
/*开窗函数和排名类函数结合，看每个课程的排名*/

SELECT
	NAME,
	course,
	score,
	row_number() over(PARTITION BY course ORDER BY score DESC) AS score_rank
FROM school_score;
~~~

**统计结果**

![1644479456760](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/155058-821693.png)

row_number()是开窗函数，以当前行进行开窗操作，over()函数是排名函数，partition by指明按照哪一个字段进行开窗操作，order by指明按照哪一个字段进行排序操作。

所以最终的结果是，按照course进行开窗，也就是分组，然后再按照score在每一个开窗内部进行排序操作

##### 使用开窗函数计算每一门课程的最高分

~~~sql
-- 使用开窗函数计算每一门课程的最高分

SELECT
	*
FROM 
(
	SELECT
		NAME,
		course,
		score,
		row_number() over(PARTITION BY course ORDER BY score DESC) AS score_rank
	FROM school_score
) AS temp

WHERE temp.score_rank =1;
~~~

在上面子查询的基础上，因为我们需要计算每一门功课的最高分，而每一门功课已经开窗口了，并且在窗口的内部从高到低进行排序，所以我们只需要取出每一个窗口内部的第一条数据即可。

**执行结果**

![1644480048559](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/160049-767133.png)

##### 开窗函数和聚合函数一起使用

~~~sql
/*第二部分：开窗函数和SUM() ，AVG() 等聚合函数结合*/
    
SELECT
    `name`,
    `course`,
    `score`,
    SUM( score ) over ( PARTITION BY `course` ) AS course_score_total ,
    round(AVG(score) over (PARTITION BY `course`),2)  as  course_score_avg
FROM
    `test`.`school_score`;
~~~

**执行结果**

![1644480084516](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/160125-279075.png)

和sum()聚合函数一起使用，那么会讲窗口内所有数据累加起来，然后放在每一条记录的后面。

如果和avg()聚合函数使用，那么会记录窗口内部数据的平均值。

~~~sql
/* SUM(score) over (PARTITION BY `course` ORDER BY score ASC)   如果执行这个语句，就是在每个
课程对分数进行累加*/

SELECT
    `name`,
    `course`,
    `score`,
    SUM(score) over (PARTITION BY `course` ORDER BY score ASC ) as course_score_total
FROM
`test`.`school_score`;
~~~

**执行结果**

![1644480202174](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/160323-489148.png)

 **有order by ,按照排序连续累加；无order by ,计算partition by 后的和；over() 中没有partition by ,计算所有数据总和**

**同时，order by 的asc 和 desc 的排序不同，有order by 的结果也不一样。**

#####  row number() over , rank() over ,dense rank() 三者对比。

**数据**

~~~sql
create table students_score(
    id int(4)  auto_increment primary key,
    name varchar(50) not null, 
    score int(4) not null
    );
    
    insert into students_score(name,score) values
    ('A', 300),
    ('B', 240),
    ('C', 250), 
    ('D', 280), 
    ('E', 240), 
    ('F', 200);
~~~

**执行sql语句**

~~~sql
SELECT
    `id`,
    `name`,
    rank ( ) over ( ORDER BY score DESC ) AS r,
    DENSE_RANK ( ) OVER ( ORDER BY score DESC ) AS dense_r,
    row_number ( ) OVER ( ORDER BY score DESC ) AS row_r 
FROM
    students_score;
~~~

**执行结果**

![1644480353578](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/160554-599731.png)

rank()排序：数据相同，排名相同，数据行总数不会少。

dense_rank()：数据相同，排名相同，但是数据行总数减少。

row_number():按照行数进行排序操作。

##### ntile()分组函数

ntile()函数是分组函数，里面的参数表示分多少组。

~~~sql
select ntile(3) over (order by score desc) as zu,
       name,
       score,
       province
from student
~~~

一共13条数据，现在我们分为3组：

![1644480651385](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/161052-328205.png)

一共13行数据，分三组，第一组就是5；

那么现在看看分5组如何。

![1644480714297](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/161155-975548.png)

分成五个组，前面1,2,3组是三个，后面两个组是2个。

还可以使用ntile() over (partition by province order by score desc),先按province分再分组

![u1644480752393](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/161233-648208.png)

### Mysql面试50题目

#### 首先创建表

>  学生表：student(学号,学生姓名,出生年月,性别)
>
> 成绩表：score(学号,课程号,成绩)
>
> 课程表：course(课程号,课程名称,教师号)
>
> 教师表：teacher(教师号,教师姓名)

 **学生表**

~~~sql
USE test;

CREATE TABLE student(
	`id` VARCHAR(10) ,
	`name` VARCHAR(10),
	`date` DATE,
	`sex` VARCHAR(10),
	PRIMARY KEY(id)
);
~~~

**分数表**

~~~sql
CREATE TABLE score
(
	id VARCHAR(10),
	deptno VARCHAR(10),
	score FLOAT,
	PRIMARY KEY(id,deptno)
);
~~~

**课程表**

~~~sql
CREATE TABLE course
(
	courseId VARCHAR(10),
	courseName VARCHAR(10),
	techNo VARCHAR(10),
	PRIMARY KEY(courseId)
);
~~~

**教师表**

~~~sql
CREATE TABLE teacher
(
	techNo VARCHAR(10),
	techName VARCHAR(10),
	PRIMARY KEY (techNo)
);
~~~

#### 表中插入数据

~~~sql
-- 向学生表中插入数据

INSERT INTO student VALUES('0001' , '猴子' , '1989-01-01' , '男');

INSERT INTO student VALUES('0002' , '猴子' , '1990-12-21' , '女');

INSERT INTO student VALUES('0003' , '马云' , '1991-12-21' , '男');

INSERT INTO student VALUES('0004' , '王思聪' , '1990-05-20' , '男');

-- 向成绩表中插入数据

INSERT INTO score VALUES('0001' , '0001' , 80);

INSERT INTO score VALUES('0001' , '0002' , 90);

INSERT INTO score VALUES('0001' , '0003' , 99);

INSERT INTO score VALUES('0002' , '0002' , 60);

INSERT INTO score VALUES('0002' , '0003' , 80);

INSERT INTO score VALUES('0003' , '0001' , 80);

INSERT INTO score VALUES('0003' , '0002' , 80);

INSERT INTO score VALUES('0003' , '0003' , 80);

-- 向课程表插入数据

INSERT INTO course VALUES('0001' , '语文' , '0002');

INSERT INTO course VALUES('0002' , '数学' , '0001');

INSERT INTO course VALUES('0003' , '英语' , '0003');

-- 向教师表中添加数据

INSERT INTO teacher VALUES('0001' , '孟扎扎');

INSERT INTO teacher VALUES('0002' , '马化腾');

-- 这里的教师姓名是空值（null）
INSERT INTO teacher VALUES('0003' , NULL);

-- 这里的教师姓名是空字符串（''）
INSERT INTO teacher VALUES('0004' , '');
~~~

#### 分析四张表关系

![1644482622890](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/164343-268250.png)

![1644482650836](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/10/164411-458885.png)

#### 简单查询

##### 查找姓猴的学生名单

~~~sql
SELECT
	NAME
FROM student
WHERE NAME LIKE "猴%";
~~~

##### 查找姓名中最后一个字是猴的学生名单

~~~sql
SELECT
	NAME
FROM student
WHERE NAME LIKE "%猴";
~~~

##### 查找姓名中带猴字的学生名单

~~~sql
SELECT
	NAME
FROM student
WHERE NAME LIKE "%猴%";
~~~

##### 查找姓孟的老师的个数

~~~sql
SELECT COUNT(techName)
FROM teacher
WHERE techName LIKE "孟%";
~~~

#### 汇总分析

##### 面试题：查询课程编号为“0002”的总成绩

~~~sql
--思路分析
*
分析思路
select 查询结果 [总成绩:汇总函数sum]
from 从哪张表中查找数据[成绩表score]
where 查询条件 [课程号是0002]
*/

SELECT
	deptno,
	SUM(score)
FROM score
WHERE deptno = '0002';
~~~

学会使用聚合函数：sum()

##### 查询选了课程的学生人数

**使用group by去重**

~~~sql
-- 查询选课的人
SELECT
	id,
	COUNT(id)
FROM score
GROUP BY id;

-- 使用子查询统计人数
SELECT
	COUNT(*) AS num
FROM
(
	SELECT
		id,
		COUNT(id)
	FROM score
	GROUP BY id
)tmp;
~~~

在这里使用的是子查询代替distinct()函数去重，使用group by先分组，然后统计分组的个数即可。

也可以使用distinct对数据进行去重，但是当数据量大时候，效率很低。

**使用distinct去重**

~~~sql
/*
这个题目翻译成大白话就是：查询有多少人选了课程
select 学号，成绩表里学号有重复值需要去掉
from 从课程表查找score;
*/
select count(distinct 学号) as 学生人数 
from score;
~~~

#### 分组练习

##### 查询各科成绩最高和最低的分

查询各科成绩最高和最低的分， 以如下的形式显示：课程号，最高分，最低分。

~~~sql
/*
分析思路
select 查询结果 [课程ID：是课程号的别名,最高分：max(成绩) ,最低分：min(成绩)]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [没有]
group by 分组 [各科成绩：也就是每门课程的成绩，需要按课程号分组];
*/
SELECT
	deptno,
	MAX(score),
	MIN(score)
FROM score
GROUP BY deptno;
~~~

##### 查询每门课程被选修的学生数

~~~sql
/*
分析思路
select 查询结果 [课程号，选修该课程的学生数：汇总函数count]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [没有]
group by 分组 [每门课程：按课程号分组];
*/
SELECT 
	deptno,
	COUNT(id) AS num
FROM score
GROUP BY deptno;
~~~

##### 查询男生、女生人数

> /*
> 分析思路
> select 查询结果 [性别，对应性别的人数：汇总函数count]
> from 从哪张表中查找数据 [性别在学生表中，所以查找的是学生表student]
> where 查询条件 [没有]
> group by 分组 [男生、女生人数：按性别分组]
> having 对分组结果指定条件 [没有]
> order by 对查询结果排序[没有];
> */

~~~sql

SELECT
	sex,
	COUNT(sex)
FROM student
GROUP BY sex;
~~~

**思路二**

首先给性别打上标签，然后使用if语句进行判断。

~~~sql
-- 使用if判断语句
SELECT
	*,
	IF(sex='男',1,0) AS target
FROM student;

-- 使用子查询统计男女生人数

SELECT
	SUM(IF(target=1,1,0)) AS man,
	SUM(IF(target=0,1,0))AS woman
FROM
(
	SELECT
		*,
		IF(sex='男',1,0) AS target
	FROM student
)AS tmp;
~~~

**思路三**

~~~sql
SELECT
	SUM(IF(sex ='男',1,0)) AS man,
	SUM(IF(sex='女',1,0)) AS woman
FROM student;
~~~

**思路四，使用case when**

~~~sql
SELECT
	SUM(CASE WHEN sex = '男' THEN 1 ELSE 0 END ) AS man,
	SUM(CASE WHEN sex = '女' THEN 1 ELSE 0 END ) AS woman
FROM student;
~~~

#### 分组结果的条件

##### 查询平均成绩大于60分学生的学号和平均成绩

~~~sql
/* 
题目翻译成大白话：
平均成绩：展开来说就是计算每个学生的平均成绩
这里涉及到“每个”就是要分组了
平均成绩大于60分，就是对分组结果指定条件

分析思路
select 查询结果 [学号，平均成绩：汇总函数avg(成绩)]
from 从哪张表中查找数据 [成绩在成绩表中，所以查找的是成绩表score]
where 查询条件 [没有]
group by 分组 [平均成绩：先按学号分组，再计算平均成绩]
having 对分组结果指定条件 [平均成绩大于60分]
*/
SELECT
	id,
	ROUND(AVG(score),3) AS avg_score
FROM score
GROUP BY id
HAVING avg_score>60;
~~~

##### 查询至少选修两门课程的学生学号

~~~sql
/* 
翻译成大白话：
第1步，需要先计算出每个学生选修的课程数据，需要按学号分组
第2步，至少选修两门课程：也就是每个学生选修课程数目>=2，对分组结果指定条件

分析思路
select 查询结果 [学号,每个学生选修课程数目：汇总函数count]
from 从哪张表中查找数据 [课程的学生学号：课程表score]
where 查询条件 [至少选修两门课程：需要先计算出每个学生选修了多少门课，需要用分组，所以这里没有where子句]
group by 分组 [每个学生选修课程数目：按课程号分组，然后用汇总函数count计算出选修了多少门课]
having 对分组结果指定条件 [至少选修两门课程：每个学生选修课程数目>=2]
*/
SELECT
	id,
	COUNT(deptno) AS con
FROM score
GROUP BY id
HAVING con>=2;
~~~

##### 查询同名同姓学生名单并统计同名人数

~~~sql
/* 
翻译成大白话，问题解析：
1）查找出姓名相同的学生有谁，每个姓名相同学生的人数
查询结果：姓名,人数
条件：怎么算姓名相同？按姓名分组后人数大于等于2，因为同名的人数大于等于2
分析思路
select 查询结果 [姓名,人数：汇总函数count(*)]
from 从哪张表中查找数据 [学生表student]
where 查询条件 [没有]
group by 分组 [姓名相同：按姓名分组]
having 对分组结果指定条件 [姓名相同：count(*)>=2]
order by 对查询结果排序[没有];
*/
SELECT
	NAME,
	COUNT(NAME) AS con
FROM student
GROUP BY NAME
HAVING con>=2;
~~~

##### 查询不及格的课程并按课程号从大到小排列

~~~sql
/* 
分析思路
select 查询结果 [课程号]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [不及格：成绩 <60]
group by 分组 [没有]
having 对分组结果指定条件 [没有]
order by 对查询结果排序[课程号从大到小排列：降序desc];
*/
SELECT 
	deptno,
	score
FROM score
WHERE score >= 60
ORDER BY deptno DESC;
~~~

##### 查询每门课程的平均成绩，结果按平均成绩升序排序，平均成绩相同时，按课程号降序排列

~~~sql
/* 
分析思路
select 查询结果 [课程号,平均成绩：汇总函数avg(成绩)]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [没有]
group by 分组 [每门课程：按课程号分组]
having 对分组结果指定条件 [没有]
order by 对查询结果排序[按平均成绩升序排序:asc，平均成绩相同时，按课程号降序排列:desc];
*/
SELECT
	deptno,
	ROUND(AVG(score)) AS avg_score
FROM score
GROUP BY deptno
ORDER BY avg_score ASC,deptno DESC;
~~~

##### 检索课程编号为“0004”且分数小于60的学生学号，结果按按分数降序排列

~~~sql
/* 
分析思路
select 查询结果 []
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [课程编号为“04”且分数小于60]
group by 分组 [没有]
having 对分组结果指定条件 []
order by 对查询结果排序[查询结果按按分数降序排列];
*/
SELECT
	id,
	deptno,
	score
FROM score
WHERE deptno = '0002' AND score <80
ORDER BY score DESC;
~~~

##### 统计每门课程的学生选修人数(超过2人的课程才统计)

~~~sql
/* 
分析思路
select 查询结果 [要求输出课程号和选修人数]
from 从哪张表中查找数据 []
where 查询条件 []
group by 分组 [每门课程：按课程号分组]
having 对分组结果指定条件 [学生选修人数(超过2人的课程才统计)：每门课程学生人数>2]
order by 对查询结果排序[查询结果按人数降序排序，若人数相同，按课程号升序排序];
*/
SELECT
	deptno,
	COUNT(id) AS num
FROM score
GROUP BY deptno
HAVING num>=2
ORDER BY num DESC,deptno ASC;
~~~

##### 首先查询有两门功课在80分以上的同学的学号

首先求出有那些同学有两门功课成绩大于80

~~~sql
SELECT
	id
FROM score
WHERE score >=80
GROUP BY id
HAVING COUNT(id)>=2;
~~~

然后求有两门功课成绩大于80分的同学的平均成绩。

~~~sql
SELECT
	id,
	ROUND(AVG(score),3) AS avg_score
FROM score
WHERE id IN
(
	SELECT
		id
	FROM score
	WHERE score >=80
	GROUP BY id
	HAVING COUNT(id)>=2
)
GROUP BY id;

~~~

- 使用in子查询。

**思路二**

~~~sql
/* 
第1步：得到每个学生的平均成绩，显示学号，平均成绩
select 查询结果 [学号,平均成绩：汇总函数avg(成绩)]
from 从哪张表中查找数据 [涉及到成绩：成绩表score]
where 查询条件 [没有]
group by 分组 [每个学生的平均：按学号分组]
having 对分组结果指定条件 [没有]
order by 对查询结果排序[没有];
*/
select 学号, avg(成绩) as 平均成绩
from score
group by 学号;


/* 
第2步：再加上限制条件：
1）不及格课程
2）两门以上[不及格课程]
select 查询结果 [学号,平均成绩：汇总函数avg(成绩)]
from 从哪张表中查找数据 [涉及到成绩：成绩表score]
where 查询条件 [限制条件：不及格课程，平均成绩<60]
group by 分组 [每个学生的平均：按学号分组]
having 对分组结果指定条件 [限制条件：课程数目>2,汇总函数count(课程号)>2]
order by 对查询结果排序[没有];
*/
SELECT
	id,
	ROUND(AVG(score),3) AS avg_score
FROM score
WHERE score >=80
GROUP BY id
HAVING COUNT(id)>=2;
~~~

#### 汇总分析

##### 查询学生的总成绩并进行排名

~~~sql
/*
分析思路
select 查询结果 [总成绩：sum(成绩), 学号]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [没有]
group by 分组 [学生的总成绩：按照每个学生学号进行分组]
order by 排序 [按照总成绩进行排序：sum(成绩)];
/*

SELECT
	id,
	SUM(score) AS sum_score
FROM score
GROUP BY id
ORDER BY sum_score DESC;
~~~

##### 查询平均成绩大于60分的学生的学号和平均成绩

~~~sql
/*
分析思路
select 查询结果 [学号, 平均成绩: avg(成绩)]
from 从哪张表中查找数据 [成绩表score]
where 查询条件 [没有]
group by 分组 [学号]
having 分组条件 [平均成绩大于60分：avg(成绩 ) >60]
order by 排序 [没有];
/*
SELECT
	id,
	AVG(score) AS avg_score
FROM score
GROUP BY id
HAVING avg_score >=70;
~~~

#### 复杂查询

##### 查询所有课程成绩大于等于80分学生的学号、姓名

首先查询所有成绩都大于80分的学生的信息

~~~sql
-- 查询所有课程成绩都大于80的学生
SELECT 
	*
FROM score
GROUP BY id
HAVING score >=80;
~~~

首先查询所有成绩都大于80分的学生的id

~~~sql
-- 查询所有课程成绩都大于80的学生的id
SELECT
	id
FROM
(
	SELECT 
		*
	FROM score
	GROUP BY id
	HAVING score >=80
)AS tmp;
~~~

使用内连接查询学生id的名字

~~~sql
-- 使用join查询名字
SELECT 
	student.id,
	student.`name`
FROM student
INNER JOIN
(
	SELECT
		id
	FROM
	(
		SELECT 
			*
		FROM score
		GROUP BY id
		HAVING score >=80
	)AS tmp
)tab
ON student.id = tab.id;
~~~

**思路二：使用in子句**

~~~sql
SELECT 
	id,
	NAME
FROM student
WHERE id IN
(
SELECT
	id
FROM
(
		SELECT 
			*
		FROM score
		GROUP BY id
		HAVING score >=80
	)AS tmp
)
~~~

> 这里注意一下，判断某一个学生所有成绩都大于80分，不能使用where判断，因为where子句只是一个过滤条件，所以我们需要先按照学号分组，然后判断每一个组内成绩是否大于80，所以需要使用having语句。

##### 查询没有学全所有课的学生的学号、姓名

```text
/*
查找出学号，条件：没有学全所有课，也就是该学生选修的课程数 < 总的课程数
【考察知识点】in，子查询
*/
```

1. 首先从course表中查询一共有多少门功课

~~~sql
-- 首先查询一共有多少们功课

SELECT
	COUNT(*) AS deptNum
FROM course;
~~~

2. 查询选修了所有功课的学生的学号

~~~sql
-- 查询选修课程数量超过3门的学生

SELECT
	id
FROM score
GROUP BY id
HAVING COUNT(deptno)>=(
	SELECT
		COUNT(*) AS deptNum
	FROM course
);
~~~

3. 从student表中使用in子句查询学生姓名。

~~~sql
-- 在student表中，使用in查询学生的名字

SELECT
	id,
	NAME
FROM student
WHERE id IN
(
	SELECT
		id
	FROM score
	GROUP BY id
	HAVING COUNT(deptno)>=(
		SELECT
			COUNT(*) AS deptNum
		FROM course
	)
);
~~~

##### 查询出只选修了两门课程的全部学生的学号和姓名|

1. 首先查询处选修两门课程的学生的id

~~~sql
SELECT
	id
FROM score
GROUP BY id
HAVING COUNT(deptno)=2;
~~~

2. 使用join内连接和student表关联查询出学生的姓名

~~~sql
-- 使用join关联学号查询姓名

SELECT
	tmp.id,
	student.name
FROM
(
	SELECT
		id
	FROM score
	GROUP BY id
	HAVING COUNT(deptno)=2
)tmp
JOIN student
ON tmp.id = student.id;
~~~

在这里做了一个优化操作，子查询是一个小表，使用小表关联大表。

#### 日期函数

![1644638232964](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/115715-402389.png)

##### 获取当前日期

~~~sql
-- 获取当前日期
SELECT CURRENT_DATE;
~~~

##### 获取当前系统时间

~~~sql
-- 获取当前系统时间
SELECT CURRENT_TIME;
~~~

##### 获取当前系统的时间和日期

~~~sql
-- 获取当前日通的日期和时间
SELECT CURRENT_TIMESTAMP;
~~~

##### 获取年份信息

~~~sql
-- 获取年份信息
SELECT YEAR('2022-02-12');
~~~

##### 获取月份信息

~~~sql
-- 获取月月份信息
SELECT MONTH('2022-02-12');
~~~

##### 获取具体日期

~~~sql
-- 获取日期day
SELECT DAY('2022-02-12');
~~~

##### 获取日期对应星期几

~~~sql
-- 获取日期对应的星期几
SELECT DAYNAME('2022-02-12');
~~~

##### 查询各科成绩前两名的记录

> 典型的top N问题

~~~sql
-- 查找0002好课程的前两名成绩，如果有多个课程，我们可以使用union all进行合并操作
(SELECT
	*
FROM score
WHERE deptno ='0001'
ORDER BY score DESC
LIMIT 2
)
UNION ALL
(
SELECT
	*
FROM score
WHERE deptno ='0002'
ORDER BY score DESC
LIMIT 2
)
UNION ALL
(
SELECT
	*
FROM score
WHERE deptno ='0003'
ORDER BY score DESC
LIMIT 2
)
~~~

##### 查询各学生的年龄（精确到月份）

~~~sql
/*
【知识点】时间格式转化​
*/
select 学号 ,timestampdiff(month ,出生日期 ,now())/12 
from student ;
-- 计算每一个学生的年龄

SELECT
	id,
	TIMESTAMPDIFF(MONTH,DATE,NOW())/12
FROM student;
~~~

TIMESTAMPDIFF(interval，time1_expr,time2_expr)：interval是时间的间隔，year,month,day，后面两个参数是时间表达式，除以12表示interval的单位。

##### 查询本月过生日的学生

~~~sql
-- 查询本月过生日的学生

SELECT
	*
FROM student
WHERE MONTH(DATE)=MONTH(NOW())+2;
~~~

#### Top N问题

> 需要了解一下关联子查询

##### 分组取每组最大值

**案例：按课程号分组取成绩最大值所在行的数据**

我们可以使用分组（group by）和汇总函数得到每个组里的一个值（最大值，最小值，平均值等）。但是无法得到成绩最大值所在行的数据。

~~~sql
select 课程号,max(成绩) as 最大成绩
from score 
group by 课程号;
SELECT
	id,
	deptno,
	MAX(score) AS max_score
FROM score
GROUP BY deptno;
~~~

##### 分组取每组最小值

**案例：按课程号分组取成绩最小值所在行的数据**

~~~sql
-- 分组每一组取最小值

SELECT
	id,
	deptno,
	MIN(score) AS min_score
FROM score
GROUP BY deptno;
~~~

##### 每组最大的N条记录

**案例：查询各科成绩前两名的记录**

第1步，查出有哪些组

我们可以按课程号分组，查询出有哪些组，对应这个问题里就是有哪些课程号

~~~sql
select 课程号,max(成绩) as 最大成绩
from score 
group by 课程号;

SELECT
	deptno,
	MAX(score) AS max_score
FROM score
GROUP BY deptno;
~~~

第2步：先使用order by子句按成绩降序排序（desc），然后使用limt子句返回topN（对应这个问题返回的成绩前两名）

~~~sql
-- 课程号'0001' 这一组里成绩前2名
select * 
from score 
where 课程号 = '0001' 
order by 成绩  desc 
limit 2;
~~~

同样的，可以写出其他组的（其他课程号）取出成绩前2名的sql

第3步，使用union all 将每组选出的数据合并到一起

~~~sql
-- 查找0002好课程的前两名成绩，如果有多个课程，我们可以使用union all进行合并操作
(SELECT
	*
FROM score
WHERE deptno ='0001'
ORDER BY score DESC
LIMIT 2
)
UNION ALL
(
SELECT
	*
FROM score
WHERE deptno ='0002'
ORDER BY score DESC
LIMIT 2
)
UNION ALL
(
SELECT
	*
FROM score
WHERE deptno ='0003'
ORDER BY score DESC
LIMIT 2
)
~~~

#### 多表查询

##### 查询所有学生的学号、姓名、选课数、总成绩

1. 首先查询每一个学生选修几门功课的信息

~~~sql
SELECT	
	id,
	COUNT(deptno) AS dept_num,
	SUM(score) AS sum_score
FROM score
GROUP BY id;
~~~

2. 和join表做内连接查询学生信息

~~~sql
-- 使用join内链接查询名字
SELECT
	student.`id`,
	student.`name`,
	tmp.dept_num,
	tmp.sum_score
FROM
(
	SELECT	
		id,
		COUNT(deptno) AS dept_num,
		SUM(score) AS sum_score
	FROM score
	GROUP BY id
)AS tmp
INNER JOIN student
ON tmp.id = student.id;
~~~

**第二种思路，使用left join**

~~~sql
selecta.学号,a.姓名,count(b.课程号) as 选课数,sum(b.成绩) as 总成绩
from student as a left join score as b
on a.学号 = b.学号
group by a.学号;

-- 使用左外链接

SELECT
	student.id,
	student.name,
	COUNT(score.deptno) AS dept_num,
	SUM(score.score) AS score_sum
FROM student
LEFT JOIN score
ON student.`id` = score.`id`
GROUP BY id;
~~~

##### 查询平均成绩大于85的所有学生的学号、姓名和平均成绩

使用左外链接。

~~~sql
-- 查询平均成绩大于85的所有学生的学号、姓名和平均成绩

SELECT
	student.`id`,
	student.`name`,
	AVG(score) AS avg_score
FROM student
INNER JOIN score
ON student.`id` = score.`id`
GROUP BY id
HAVING avg_score >=85;
~~~

##### 查询学生的选课情况：学号，姓名，课程号，课程名称

使用多表的内连接查询。

~~~sql
-- 查询学生的选课情况：学号，姓名，课程号，课程名称

SELECT
	student.`id`,
	student.`name`,
	score.`deptno`,
	course.`courseName`
FROM student
INNER JOIN score
ON student.`id` = score.`id`
INNER JOIN course
ON score.`deptno` = course.`courseId`;
~~~

##### 查询出每门课程的及格人数和不及格人数

考察case-when的用法

~~~sql
-- 查询出每门课程的及格人数和不及格人数

SELECT
	deptno,
	SUM(CASE WHEN score >=60 THEN 1 ELSE 0 END)AS p1,
	SUM(CASE WHEN score <60 THEN 1 ELSE 0 END)AS p2
FROM score
GROUP BY deptno;
~~~

##### 使用分段[100-85],[85-70],[70-60],[<60]来统计各科成绩，分别统计：各分数段人数，课程号和课程名称

~~~sql
-- 考察case表达式
select a.课程号,b.课程名称,
sum(case when 成绩 between 85 and 100 
	 then 1 else 0 end) as '[100-85]',
sum(case when 成绩 >=70 and 成绩<85 
	 then 1 else 0 end) as '[85-70]',
sum(case when 成绩>=60 and 成绩<70  
	 then 1 else 0 end) as '[70-60]',
sum(case when 成绩<60 then 1 else 0 end) as '[<60]'
from score as a right join course as b 
on a.课程号=b.课程号
group by a.课程号,b.课程名称;
~~~

**分段统计**

~~~sql
SELECT a.deptno,b.courseName,
SUM(CASE WHEN score BETWEEN 85 AND 100 
	 THEN 1 ELSE 0 END) AS '[100-85]',
SUM(CASE WHEN score >=70 AND score<85 
	 THEN 1 ELSE 0 END) AS '[85-70]',
SUM(CASE WHEN score>=60 AND score<70  
	 THEN 1 ELSE 0 END) AS '[70-60]',
SUM(CASE WHEN score<60 THEN 1 ELSE 0 END) AS '[<60]'
FROM score AS a RIGHT JOIN course AS b 
ON a.deptno=b.`courseId`
GROUP BY a.deptno,b.courseName;
~~~

##### 查询课程编号为0003且课程成绩在80分以上的学生的学号和姓名

~~~sql
-- 查询课程编号为0003且课程成绩在80分以上的学生的学号和姓名

SELECT
	student.`id`,
	student.`name`
FROM student
INNER JOIN score
ON student.`id`=score.`id`
WHERE score.`id`='0003' AND score >=80;
~~~

#### 行列转换

下面是学生的成绩表（表名score，列名：学号、课程号、成绩）

![1644645012906](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/135014-288812.png)

使用sql实现将该表行转列为下面的表结构

![1644645041686](C:\Users\MrR\AppData\Roaming\Typora\typora-user-images\1644645041686.png)

**第1步，使用常量列输出目标表的结构**

可以看到查询结果已经和目标表非常接近了

~~~sql
select 学号,'课程号0001','课程号0002','课程号0003'
from score;

SELECT id,'课程号0001','课程号0002','课程号0003'
FROM score;
~~~

![1644645153056](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/135234-508214.png)

**第2步，使用case表达式，替换常量列为对应的成绩**

~~~sql
-- 第2步，使用case表达式，替换常量列为对应的成绩

SELECT
	id,
	(CASE deptno WHEN '0001' THEN score ELSE 0 END) AS '0001',
	(CASE deptno WHEN '0002' THEN score ELSE 0 END) AS '0002',
	(CASE deptno WHEN '0003' THEN score ELSE 0 END) AS '0003'
FROM score;
~~~

![1644645415593](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/135657-726189.png)

在这个查询结果中，每一行表示了某个学生某一门课程的成绩。比如第一行是'学号0001'选修'课程号00001'的成绩，而其他两列的'课程号0002'和'课程号0003'成绩为0。

每个学生选修某门课程的成绩在下图的每个方块内。我们可以通过分组，取出每门课程的成绩。

![1644645448836](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/135730-195360.png)

**第3关，分组**

分组，并使用最大值函数max取出上图每个方块里的最大值

~~~sql
-- 分组
SELECT
	id,
	MAX((CASE deptno WHEN '0001' THEN score ELSE 0 END)) AS '0001',
	MAX((CASE deptno WHEN '0002' THEN score ELSE 0 END)) AS '0002',
	MAX((CASE deptno WHEN '0003' THEN score ELSE 0 END)) AS '0003'
FROM score
GROUP BY id;
~~~

![1644645590016](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/140049-569987.png)

##### mysql中case-when的两种用法

MySQL 的 case when 的语法有两种：

1. 简单函数 
   `CASE [col_name] WHEN [value1] THEN [result1]…ELSE [default] END`
2. 搜索函数 
   `CASE WHEN [expr] THEN [result1]…ELSE [default] END`

 **简单函数**

~~~sql
SELECT
    NAME '英雄',
    CASE NAME
        WHEN '德莱文' THEN
            '斧子'
        WHEN '德玛西亚-盖伦' THEN
            '大宝剑'
        WHEN '暗夜猎手-VN' THEN
            '弩'
        ELSE
            '无'
    END '装备'
FROM
    user_info;
~~~

**搜索函数**

`CASE WHEN [expr] THEN [result1]…ELSE [default] END`：搜索函数可以写判断，并且搜索函数只会返回第一个符合条件的值，其他`case`被忽略

~~~sql
# when 表达式中可以使用 and 连接条件
SELECT
    NAME '英雄',
    age '年龄',
    CASE
        WHEN age < 18 THEN
            '少年'
        WHEN age < 30 THEN
            '青年'
        WHEN age >= 30
        AND age < 50 THEN
            '中年'
        ELSE
            '老年'
    END '状态'
FROM
    user_info;
~~~

#### 多表连接

##### 检索"0001"课程分数小于60，按分数降序排列的学生信息

思路

![1644646148135](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/140910-751782.png)

~~~sql
-- 检索"0001"课程分数大于等于80，按分数降序排列的学生信息

SELECT
	student.id,
	student.name,
	score.score
FROM student
INNER JOIN score
ON student.`id` = score.`id` 
WHERE deptno = '0001' AND score >=80
ORDER BY score DESC;
~~~

##### 查询不同老师所教不同课程平均分从高到低显示

思路

![1644646537010](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/141539-371208.png)

~~~sql
-- 查询不同老师所教不同课程平均分从高到低显示

SELECT
	teacher.`techName`,
	ROUND(AVG(score)) AS avg_score
FROM teacher
INNER JOIN course
ON teacher.`techNo` = course.`techNo`
INNER JOIN score
ON course.`courseId` = score.`deptno`
GROUP BY teacher.`techNo`
ORDER BY avg_score DESC;
~~~

##### 查询课程名称为"数学"，且分数低于60的学生姓名和分数

思路

![1644646850766](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/12/142052-998848.png)

~~~sql
SELECT 
	student.`id`,
	student.`name`,
	score.`score`
FROM student
INNER JOIN score
ON student.`id` = score.`id`
INNER JOIN course
ON score.`deptno` = course.`courseId`
WHERE score <=60;
~~~

##### 查询任何一门课程成绩在70分以上的姓名、课程名称和分数（与上题类似）

~~~sql
select a.姓名,c.课程名称 ,b.成绩 
from student as ​a 
inner join score as b 
​​on a.学号=b.学号
inner join course c on b.课程号 =c.课程号 
where b.成绩 >70;
~~~

##### 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

> 分组+条件+多表连接
>
> 翻译成大白话:计算每个学号不及格分数个数，筛选出大于2个的学号并找出姓名，平均成绩，思路如图：

![1644728874878](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/130757-690971.png)

~~~sql
select b.姓名,avg(a.成绩),a.学号  
from score as​ a
inner join student as b 
​​on a.学号 =b.学号 
where a.成绩 <60
group by a.学号 
having count(a.学号 ) >=2;

-- 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

SELECT
	student.`id`,
	student.`name`,
	AVG(score) AS avg_score
FROM student
INNER JOIN score
ON student.`id` = score.`id`
WHERE score>=80
GROUP BY student.`id`
HAVING COUNT(student.`id`)>=2;
~~~

##### 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

> 重点问题，单张表相互做内连接

![1644729517695](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/131840-947545.png)

~~~sql
select distinct ​a.学号 ,a.成绩 ,a.课程号 
from score as​ a 
inner join score as b 
​​on a.学号 =b.学号 
where a.成绩 =b.成绩 and a.课程号 != b.课程号 ;
-- 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

SELECT
	DISTINCT(a.`id`),
	a.`deptno`,
	a.`score`
FROM score AS a
INNER JOIN score AS b
ON a.`id` = b.`id`
WHERE a.`score` = b.`score` AND a.`deptno` != b.`deptno`;
~~~

##### 查询课程编号为“0001”的课程比“0002”的课程成绩高的所有学生的学号

> 多表连接+条件，思路如图

![1644729784215](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/132306-977636.png)

~~~sql
select a.学号  
​from 
(select 学号 ,成绩 from score where 课程号=01) as a
inner join 
(select 学号 ,成绩 from score where 课程号=02) as b
on a.学号 =b.学号 
inner join student c on c.学号 =a.学号 
where a.成绩 >b.成绩 ;
~~~

##### 查询学过编号为“0001”的课程并且也学过编号为“0002”的课程的学生的学号、姓名

1. 首先查询选修了0001号课程和0002号课程的同学

~~~sql
-- 首先查询选修了0001和0002号课程的同学，使用的是同一张表的内连接

SELECT
	a.`id`
FROM score AS a
INNER JOIN score AS b
ON a.`id` = b.`id`
WHERE a.`deptno`='0001' AND b.`deptno`='0002';
~~~

2. 使用in子句查询学生的信息

~~~sql
-- 使用in条件查询姓名

SELECT
	id,
	NAME
FROM student
WHERE id IN
(
	SELECT
		a.`id`
	FROM score AS a
	INNER JOIN score AS b
	ON a.`id` = b.`id`
	WHERE a.`deptno`='0001' AND b.`deptno`='0002'
);
~~~

**第二种思路**

![1644730161758](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/132924-153412.png)

~~~sql
select a.学号  
​​from 
(select 学号 ,成绩 from score where 课程号=01) as a
inner join 
(select 学号 ,成绩 from score where 课程号=02) as b
on a.学号 =b.学号 
inner join student c on c.学号 =a.学号 
where a.成绩 >b.成绩 ;
~~~

##### 查询学过“孟扎扎”老师所教的所有课的同学的学号、姓名

![1644730720596](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/133843-101298.png)

> 没做出来

#### 窗口函数

##### 查询学生平均成绩及其名次

![1644731123683](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/134526-580124.png)

~~~sql
select 学号 ,avg(成绩),
row_number () over( order by avg(成绩) desc)
from score
group by 学号  ;
-- 查询学生平均成绩及其名次

SELECT
	id,
	row_number() over(ORDER BY AVG(score) DESC)
FROM score
GROUP BY id;
~~~

##### 按各科成绩进行排序，并显示排名

![1644731231353](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/134714-256049.png)

~~~sql
-- 按各科成绩进行排序，并显示排名

SELECT
	id,
	row_number() over(PARTITION BY deptno ORDER BY score DESC)
FROM score;
~~~

##### 查询每门功成绩最好的前两名学生姓名

![1644731653586](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202202/13/135415-564185.png)

~~~sql
SELECT
	student.id,
	student.`name`,
	tmp.ranking
FROM student
INNER JOIN
(
SELECT
	id,
	row_number() over(PARTITION BY deptno ORDER BY score DESC)AS ranking
FROM score
)AS tmp
ON student.`id` = tmp.id
WHERE ranking <3;
~~~

