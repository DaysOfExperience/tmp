## 学生表+成绩表

索隆的各个科目的成绩

```mysql
mysql> select sname, cname, score from student inner join result on student.sid = result.sid and sname = '索隆'
    -> ;
+--------+--------+-------+
| sname  | cname  | score |
+--------+--------+-------+
| 索隆   | 语文   |    80 |
| 索隆   | 数学   |    30 |
| 索隆   | 英语   |    50 |
+--------+--------+-------+
3 rows in set (0.00 sec)
```

路飞的数学成绩

`mysql> select sname, cname, score from student inner join result on student.sid = result.sid where cname = '数学' and sname = '路飞';`

1班数学前两名的学生信息

```mysql
mysql> select classid, cname, score, sname from student inner join result on student.sid = result.sid where classid = 1 and cname = '数学' order by score desc limit 2;
+---------+--------+-------+--------+
| classid | cname  | score | sname  |
+---------+--------+-------+--------+
|       1 | 数学   |    99 | 路西   |
|       1 | 数学   |    80 | 山治   |
+---------+--------+-------+--------+
2 rows in set (0.00 sec)
```

**各个科目前三名的学生**

**分组topk**

两个表先连接一下, 每个同学的每科的表就出现了, 然后筛选的一个条件是: 再拿一张result表, 课程相同的情况下, 里面这张result表中成绩比外面大表中成绩高的数量<3, 那么外面这张表中的这条记录就是这个科目的前三名, 其实就是借助where条件中再找张表去进行统计.

关键是子查询中的表可以用外查询中的表, 进行比较 

```mysql
mysql> select r.sid, sname, courseid, cname, score from student s inner join result r on r.sid = s.sid where (select count(*) from result where result.courseid = r.courseid and result.score > r.score)
< 3 order by courseid, score desc;
+-------+-----------+----------+--------+-------+
| sid   | sname     | courseid | cname  | score |
+-------+-----------+----------+--------+-------+
| 10009 | 路西      |        1 | 语文   |    99 |
| 10008 | 乌索普    |        1 | 语文   |    97 |
| 10003 | 山治      |        1 | 语文   |    90 |
| 10009 | 路西      |        2 | 数学   |    99 |
| 10005 | 乔巴      |        2 | 数学   |    96 |
| 10004 | 娜美      |        2 | 数学   |    88 |
| 10007 | 布洛克    |        2 | 数学   |    88 |
| 10008 | 乌索普    |        2 | 数学   |    88 |
| 10009 | 路西      |        3 | 英语   |    99 |
| 10003 | 山治      |        3 | 英语   |    96 |
| 10008 | 乌索普    |        3 | 英语   |    96 |
+-------+-----------+----------+--------+-------+
11 rows in set (0.00 sec)
```

每个班级的总分

```mysql
mysql> select classid, sum(score) 成绩总分 from student inner join result on student.sid = result.sid group by classid ;
+---------+--------------+
| classid | 成绩总分     |
+---------+--------------+
|       1 |          878 |
|       2 |          523 |
|       3 |          492 |
|       4 |          281 |
+---------+--------------+
4 rows in set (0.01 sec)

```

每门课程的最大分数

```mysql
mysql> select max(score), courseid, cname from result group by cname, courseid;
+------------+----------+--------+
| max(score) | courseid | cname  |
+------------+----------+--------+
|         99 |        2 | 数学   |
|         99 |        3 | 英语   |
|         99 |        1 | 语文   |
+------------+----------+--------+
3 rows in set (0.00 sec)

mysql> select max(score), courseid, cname from result group by cname, courseid order by courseid;
+------------+----------+--------+
| max(score) | courseid | cname  |
+------------+----------+--------+
|         99 |        1 | 语文   |
|         99 |        2 | 数学   |
|         99 |        3 | 英语   |
+------------+----------+--------+
3 rows in set (0.00 sec)

mysql> select max(score), courseid from result group by courseid;
+------------+----------+
| max(score) | courseid |
+------------+----------+
|         99 |        1 |
|         99 |        2 |
|         99 |        3 |
+------------+----------+
3 rows in set (0.00 sec)
```

**1班每位姓路的学生的课程总成绩**

<u>第二个sql找出来之后, 不能直接sum, 要先分组, 再sum. 统计函数一般都配合着分组使用的</u>

sum之前必须group by, 不然怎么sum?

```mysql
mysql> select sum(score), classid, sname from student inner join result on result.sid = student.sid and classid = 1 and sname like '路%';
ERROR 1140 (42000): In aggregated query without GROUP BY, expression #3 of SELECT list contains nonaggregated column 'new_db_1.student.sname'; this is incompatible with sql_mode=only_full_group_by
mysql> select *from student inner join result on result.sid = student.sid and classid = 1 and sname like '路%';
+----+-------+---------+--------+----+-------+----------+--------+-------+
| id | sid   | classid | sname  | id | sid   | courseid | cname  | score |
+----+-------+---------+--------+----+-------+----------+--------+-------+
|  1 | 10001 |       1 | 路飞   |  1 | 10001 |        1 | 语文   |    75 |
|  1 | 10001 |       1 | 路飞   |  2 | 10001 |        2 | 数学   |    60 |
|  1 | 10001 |       1 | 路飞   |  3 | 10001 |        3 | 英语   |    20 |
|  9 | 10009 |       1 | 路西   | 25 | 10009 |        1 | 语文   |    99 |
|  9 | 10009 |       1 | 路西   | 26 | 10009 |        2 | 数学   |    99 |
|  9 | 10009 |       1 | 路西   | 27 | 10009 |        3 | 英语   |    99 |
+----+-------+---------+--------+----+-------+----------+--------+-------+
6 rows in set (0.00 sec)

mysql> select classid, sname, sum(score) from student inner join result on result.sid = student.sid and classid = 1 and sname like '路%' group by classid, sname;
+---------+--------+------------+
| classid | sname  | sum(score) |
+---------+--------+------------+
|       1 | 路西   |        297 |
|       1 | 路飞   |        155 |
+---------+--------+------------+
2 rows in set (0.01 sec)
```

取总成绩 最高的三位 学生, 输出 : 学号, 总成绩

<u>总成绩要sum, 必须分组. 最高要order, 三位要limit</u>

```mysql
mysql> select sum(score) 总成绩, student.sid from student inner join result on student.sid = result.sid group by result.sid order by 总成绩 desc limit 3;
+-----------+-------+
| 总成绩    | sid   |
+-----------+-------+
|       297 | 10009 |
|       281 | 10008 |
|       268 | 10005 |
+-----------+-------+
3 rows in set (0.01 sec)

```

平均分低于80的同学的姓名

<u>分组之后select出来的必须是分组依据, 否则就是函数汇总的结果</u>

(严格来说不应该输出平均分啊)

```mysql
mysql> select sname from student inner join result on student.sid = result.sid group by sname having avg(score) < 80;
+--------+
| sname  |
+--------+
| 索隆   |
| 罗宾   |
| 路飞   |
+--------+
3 rows in set (0.00 sec)

mysql> select sname, avg(score) from student inner join result on student.sid = result.sid group by sname having avg(score) < 80;
+--------+------------+
| sname  | avg(score) |
+--------+------------+
| 索隆   |    53.3333 |
| 罗宾   |    77.0000 |
| 路飞   |    51.6667 |
+--------+------------+
3 rows in set (0.00 sec)
```

每个班级有多少学生

```mysql
mysql> select count(*), classid from student group by classid;
+----------+---------+
| count(*) | classid |
+----------+---------+
|        4 |       1 |
|        2 |       2 |
|        2 |       3 |
|        1 |       4 |
+----------+---------+
4 rows in set (0.00 sec)
```

输出一个班级里同名的学生的姓名

```mysql
mysql> select s1.sname from student s1 inner join student s2 where s1.classid = s2.classid and s1.sid != s2.sid and s1.sname = s2.sname;
Empty set (0.00 sec)
// 两个同样的表连接, 班级一样, 学号不一样, 名字一样, 看是否存在
mysql> select sname from student group by classid, sname having count(1) >= 2;
Empty set (0.00 sec)
mysql> select sname, count(*) as cnt from student group by classid, sname having cnt >= 2;
Empty set (0.00 sec)
```

## 学生班级成绩表

**求各班前十名的同学**

**TOPK**

student表: id, name(姓名), class_id(班级), score(成绩)

> 这种就是, 分组topK, 也就是要先分组, 然后求每组的前若干名
>
> 就像前面那个题, 求各个科目的前三名, 这个是每个班的前十名
>
> 这种不能用group by, 因为分组之后, 没法求前几名

> (这是一个单表题)

> 还是那个思路: where条件中用一个表, 班级和外表的班级相同, 代表是一个班的, 同时成绩比外表高的记录(人)的数量要小于<10 筛选出来就是了

```mysql
select class_id, name, score from student s where (select count(*) from student where student.class_id = s.class_id and student.score > s.score) < 10 order by class_id, score desc;
```

这里面的关键是, 其实是用外表中的每条记录, 也就是每个人的成绩去用这个条件进行筛选, 看是否符合. 条件就是, 内表中和外面这个人的班级, 且成绩比外表这个人的成绩高的人的数量要小于10. 最后都筛选出来, 再order by, 先班级, 再分数

## 学生选课成绩表

查询选了4门课及以上的学生的id和name

```mysql
mysql>  select student_id, name from t_student group by name, student_id having count(*) >= 4;
+------------+------+
| student_id | name |
+------------+------+
|       1001 | jj   |
+------------+------+
1 row in set (0.00 sec)
```

查询有不及格课程的学生的各科的平均分和姓名, 按照各科平均分的成绩的倒序输出

```mysql
mysql> select avg(course_score) avg_score, name from t_student where name in(select distinct name from t_student where course_score < 60) group by name order by avg_score desc;
+--------------------+------+
| avg_score          | name |
+--------------------+------+
|               70.5 | yy   |
|                 58 | uu   |
| 55.333333333333336 | tt   |
+--------------------+------+
3 rows in set (0.00 sec)
mysql> select avg(course_score), name from t_student group by name having min(course_score) < 60 order by avg(course_score) desc;
+--------------------+------+
| avg(course_score)  | name |
+--------------------+------+
|               70.5 | yy   |
|                 58 | uu   |
| 55.333333333333336 | tt   |
+--------------------+------+
3 rows in set (0.00 sec)
```

先按照名字分组, 然后having筛选条件是要有成绩不及格 的组 就筛选出来了, 每个组就是一个人的各科成绩, 这些组(人)求名字和平均成绩即可, 最后按照平均成绩排序输出

也就是 group by 分完组之后 having可以按照某个条件进行筛选出符合条件的组, 这个筛选条件其实挺多样的,  合理即可, 自己思考

查找两门及以上的科目分数相同的学生

```mysql
mysql> select distinct t1.name from t_student t1 inner join t_student t2 on t1.name = t2.name and t1.course_id != t2.course_id and t1.course_score = t2.course_score;
+------+
| name |
+------+
| jj   |
| uu   |
+------+
2 rows in set (0.00 sec)
```

每门科目都在80以上的学生的学号

```mysql
mysql> select student_id from t_student group by student_id having min(course_score) > 80 
    -> ;
+------------+
| student_id |
+------------+
|       1001 |
|       1002 |
+------------+
2 rows in set (0.00 sec)

```

## 用户信息表

大于18岁用户的个数

```mysql
mysql> select count(1) from user where age > 18;
+----------+
| count(1) |
+----------+
|        6 |
+----------+
1 row in set (0.00 sec)
```

每个城市下面年龄最大的员工的信息

```mysql
mysql> select * from user where (age, city) in (select max(age), city from user group by city)
    -> ;
+----+---------+------+--------+
| id | name    | age  | city   |
+----+---------+------+--------+
|  1 | Emma    |   20 | 深圳   |
|  3 | Noah    |   25 | 广州   |
|  4 | Jackson |   15 | 上海   |
|  6 | Aiden   |   47 | 北京   |
+----+---------+------+--------+
4 rows in set (0.00 sec)

mysql> select max(age), city from user group by city;
+----------+--------+
| max(age) | city   |
+----------+--------+
|       15 | 上海   |
|       47 | 北京   |
|       25 | 广州   |
|       20 | 深圳   |
+----------+--------+
4 rows in set (0.00 sec)
```

统计每个城市的用户数量

```mysql
mysql> select city, count(*) user_cnt from user group by city;
+--------+----------+
| city   | user_cnt |
+--------+----------+
| 上海   |        1 |
| 北京   |        3 |
| 广州   |        2 |
| 深圳   |        2 |
+--------+----------+
4 rows in set (0.00 sec)
```

年龄段分组统计人数

case????????????

查询出现次数最多的人数

```mysql
mysql> select city, count(*) cnt from user group by city order by cnt desc limit 1;
+--------+-----+
| city   | cnt |
+--------+-----+
| 北京   |   3 |
+--------+-----+
1 row in set (0.00 sec)
```





---

最后几个看看, 都挺有意义
