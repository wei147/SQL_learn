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

