## MySQL面试



#### mysql最基础的优化

```
把数据文件放到一个独立的磁盘上。这个磁盘不受其他程序的干扰,由mysql独享,那么mysql写入数据的时候就一定会顺序的写。
```



## [MySQL经典练习题及答案，常用SQL语句练习50题](https://www.cnblogs.com/Diyo/p/11424844.html)



### 练习题和sql语句

#### 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数

```mysql
-- 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数
-- SELECT *,s_score from student,score where 
-- (SELECT )

1.自连接 
SELECT
c.*,
a.s_score s01,
b.s_score s02 
FROM
	score a,
	score b,
	student c 
WHERE
	a.c_id = '01' 
	AND b.c_id = '02' 
	AND a.s_id = b.s_id -- --a表和b表根据学号连接起来
	AND c.s_id = a.s_id -- --将学生表 c和a表根据学号连接起来
AND a.s_score > b.s_score;

2.长型数据变成宽型数据
学生编号  课程编号  分数	变成宽型数据(case when a.c_id='01' then a.s_score end)
	01  			01  		 80  		80
	01  			02  		 90  		null
	01  			03  		 99  		null

SELECT
s.*,
t.s01,t.s02
FROM
(select 
	a.s_id,
	max(case when a.c_id='01' then a.s_score end ) s01, -- 如果a.c_id='01'的话返回a.s_score,否则什么都不返回 case when a.c_id='01' then a.s_score else null end
	max(case when a.c_id='02' then a.s_score end ) s02
from
	score a
GROUP BY 
	a.s_id) t,student s
WHERE t.s01>t.s02
and t.s_id=s.s_id;  -- 连接条件 
```



#### 2、查询"01"课程比"02"课程成绩低的学生的信息及课程分数

```mysql
-- 2、查询"01"课程比"02"课程成绩低的学生的信息及课程分数
SELECT
c.*,
a.s_score s01,
b.s_score s02 
FROM
	score a,
	score b,
	student c 
WHERE
	a.c_id = '01' 
	AND b.c_id = '02' 
	AND a.s_id = b.s_id -- --a表和b表根据学号连接起来
	AND c.s_id = a.s_id -- --将学生表 c和a表根据学号连接起来
AND a.s_score < b.s_score;
```



#### 3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

```mysql

-- 3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
-- score AVG() GROUP BY 根据学号s_id排序 student
1.子查询
select 
	a.s_id,
-- 	这里用了子查询的方法
	(SELECT s_name FROM student s WHERE s.s_id=a.s_id) s_name,
	AVG(a.s_score) avg_s
from
score a
GROUP BY 
a.s_id 
HAVING avg_s>=60

1.两个表连接
select 
	a.s_id,
	s.s_name,
	AVG(a.s_score) avg_s
from
score a,student s
WHERE s.s_id = a.s_id
GROUP BY 
a.s_id 
HAVING avg_s>=60
```



#### 4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩

```mysql

-- 4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
        -- (包括有成绩的和无成绩的)
	SELECT
	a.s_id,s.s_name,
	avg(a.s_score) s_avg
	FROM 
	score a,student s
	where a.s_id=s.s_id
	GROUP BY a.s_id
	HAVING s_avg<=60
	
	2.用外连接做
	SELECT
-- 	这里是以右表student的属性为准 (右连接)
	s.s_id,s.s_name,
	IFNULL(avg(a.s_score),0) s_avg 
	FROM 
	score a
	RIGHT JOIN
	student s
	on 
	a.s_id=s.s_id
	GROUP BY a.s_id
	HAVING s_avg<=60
```



#### 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩

```mysql
-- 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
-- student score count GROUP BY s_id 根据学号来分组  总成绩sum

1.自己写的
SELECT 
a.s_id,
s.s_name,
Count(a.c_id) s_count,
sum(a.s_score) s_sum
from
student s,score a
WHERE a.s_id=s.s_id
GROUP BY a.s_id

2.内连接
SELECT 
a.s_id,
s.s_name,
Count(a.c_id) s_count,
sum(a.s_score) s_sum
from
	score a
-- 	内连接。但是有一个学生成绩为空的情况下拿不到她的值
INNER JOIN
	student s
on
	a.s_id=s.s_id
GROUP BY 
a.s_id

2.右连接
SELECT 
s.s_id,
s.s_name,
Count(a.c_id) s_count,
IFNULL(sum(a.s_score),0) s_sum
from
	score a
-- 	内连接。但是有一个学生成绩为空的情况下拿不到她的值
right JOIN
	student s
on
	a.s_id=s.s_id
GROUP BY 
s.s_id,
s.s_name
```

#### 6、查询"李"姓老师的数量 

```mysql
-- 6、查询"李"姓老师的数量 
SELECT count(t_name) count_t from teacher a WHERE a.t_name like '李%';
```



#### 7、查询学过"张"老师授课的同学的信息 

```mysql
SELECT
s.*,t.t_name '任课老师'
FROM 
course c,score a, student s,teacher t
WHERE
		t.t_id = c.t_id
and c.c_id = a.c_id
and a.s_id = s.s_id
and t.t_name='张老师'
```



#### 8、查询没学过"张"老师授课的同学的信息 

```mysql
-- 8、查询没学过"张"老师授课的同学的信息 
-- 这样是不行的？ 直接取反
SELECT
s.*,t.t_name '任课老师'
FROM 
course c,score a, student s,teacher t
WHERE
		t.t_id = c.t_id
and c.c_id = a.c_id
and a.s_id = s.s_id
and t.t_name!='张老师'

-- 1.把第七道题的结果当成条件
SELECT * from student WHERE s_id not in(
SELECT
a.s_id
FROM 
course c,score a,teacher t
WHERE
		t.t_id = c.t_id
and c.c_id = a.c_id
and t.t_name='张老师');

-- not exists  感觉这种方式看着复杂了一些
SELECT * from student WHERE not EXISTS(
SELECT 1 from 
	(SELECT
	a.s_id
	FROM 
	course c,score a,teacher t
	WHERE
			t.t_id = c.t_id
	and c.c_id = a.c_id
	and t.t_name='张老师') tabl
WHERE tabl.s_id = student.s_id);
```



#### 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息

```mysql
-- 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
course 01和02  student score

1.自连接的方式
SELECT
 c.*
FROM
	score a,score b,student c
WHERE
	a.c_id = '01'
and b.c_id='02'
and a.s_id = b.s_id -- 三张表的连接条件
and a.s_id = c.s_id

2.长型数据变成宽型数据
```

#### 10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息

```mysql
-- 10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
SELECT s.* from 
(SELECT
	a.s_id,
	max(case WHEN c_id ='01' THEN s_score END) s01, -- 01课程的成绩
	max(case WHEN c_id ='02' THEN s_score END) s02 
FROM
 score a
GROUP BY
	a.s_id) t,student s -- 这里把第二个查询看做一个子表
WHERE 
	t.s_id =s.s_id	-- 连接条件
	and t.s01 is not null	-- 这里的不为空·是这样写的。 null值不能用大于小于比较
	and	t.s02 is null
```

