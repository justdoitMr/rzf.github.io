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

