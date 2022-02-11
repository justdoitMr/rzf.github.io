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