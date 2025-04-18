# 表的增删改查

CRUD : Create(创建), Retrieve(读取)，Update(更新)，Delete（删除）

> 注：这里表的增删改查，并不是建表，删表，修改表，查找表。而是对表的内容进行增删改查

## Create 增

语法

```mysql
INSERT [INTO] table_name [(column [, column] ...)]
VALUES (value_list) [, (value_list)] ...
value_list: value, [, value] ...
```

**单行数据+全列插入**

**多行数据+指定列插入**

```mysql
mysql> CREATE TABLE students (
    -> id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> sn INT NOT NULL UNIQUE COMMENT '学号',
    -> name VARCHAR(20) NOT NULL,
    -> qq VARCHAR(20)
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> 
mysql> desc students;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| sn    | int(11)          | NO   | UNI | NULL    |                |
| name  | varchar(20)      | NO   |     | NULL    |                |
| qq    | varchar(20)      | YES  |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

// 单行数据+全列插入
mysql> insert into students values (100, 10000, '杨', NULL);
Query OK, 1 row affected (0.00 sec)

mysql> insert into students values (101, 10001, '李', 1149367636);
Query OK, 1 row affected (0.00 sec)

mysql> select * from students;
+-----+-------+------+------------+
| id  | sn    | name | qq         |
+-----+-------+------+------------+
| 100 | 10000 | 杨   | NULL       |
| 101 | 10001 | 李   | 1149367636 |
+-----+-------+------+------------+
2 rows in set (0.00 sec)
// 多行数据+指定列插入
mysql> insert into students(id, sn, name) values (102, 10002, 'Yang'), (103,10003,'Lee');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from students;
+-----+-------+------+------------+
| id  | sn    | name | qq         |
+-----+-------+------+------------+
| 100 | 10000 | 杨   | NULL       |
| 101 | 10001 | 李   | 1149367636 |
| 102 | 10002 | Yang | NULL       |
| 103 | 10003 | Lee  | NULL       |
+-----+-------+------+------------+
4 rows in set (0.00 sec)
```

nsert into table values ...，此处不指定插入哪些列，则全列插入，value_list数量必须和定义表的列的数量及顺序一致 -- 注意，这里在插入的时候，也可以不用指定id, 因为id是auto_increment, 这时候就需要明确插入数据到哪些列了，那么mysql会使用默认的值进行自增。

指定列插入时，省略的列必须有默认值或者自增才可以省略，且必须指定要插入其他的列名，value_list必须和指定列数量及顺序一致。

**插入否则更新**

若由于主键或者唯一键对应的值已经存在而导致插入失败时，进行更新数据。

duplicate

- *v.*复制，复印；（无必要地）重复（某事）；使成倍增加
- *adj.*完全一样的，复制的；二重的
- *n.*完全一样的东西，复制品

```mysql
INSERT ... ON DUPLICATE KEY UPDATE
column = value [, column = value] ...
```

```mysql
mysql> select * from students;
+-----+-------+------+------------+
| id  | sn    | name | qq         |
+-----+-------+------+------------+
| 100 | 10000 | 杨   | NULL       |
| 101 | 10001 | 李   | 1149367636 |
| 102 | 10002 | Yang | NULL       |
| 103 | 10003 | Lee  | NULL       |
+-----+-------+------+------------+

mysql> insert into students values (100, 10004, 'ZZZ', null)
    -> on duplicate key update
    -> sn = 10004, name = 'zzz', qq = '123456';
Query OK, 2 rows affected (0.01 sec)

mysql> select * from students;
+-----+-------+------+------------+
| id  | sn    | name | qq         |
+-----+-------+------+------------+
| 100 | 10004 | zzz  | 123456     |
| 101 | 10001 | 李   | 1149367636 |
| 102 | 10002 | Yang | NULL       |
| 103 | 10003 | Lee  | NULL       |
+-----+-------+------+------------+

-- ON DUPLICATE KEY UPDATE - 当发生重复key的时候
```

**替换**

```mysql
-- 主键 或者 唯一键 没有冲突，则直接插入；
-- 主键 或者 唯一键 如果冲突，则删除后再插入
mysql> select * from students;
+-----+-------+------+------------+
| id  | sn    | name | qq         |
+-----+-------+------+------------+
| 100 | 10004 | zzz  | 123456     |
| 101 | 10001 | 李   | 1149367636 |
| 102 | 10002 | Yang | NULL       |
| 103 | 10003 | Lee  | NULL       |
+-----+-------+------+------------+
4 rows in set (0.00 sec)

mysql> replace into students (sn, name) values (10001, 'xxx');
Query OK, 2 rows affected (0.01 sec)

mysql> select * from students;
+-----+-------+------+--------+
| id  | sn    | name | qq     |
+-----+-------+------+--------+
| 100 | 10004 | zzz  | 123456 |
| 102 | 10002 | Yang | NULL   |
| 103 | 10003 | Lee  | NULL   |
| 104 | 10001 | xxx  | NULL   |
+-----+-------+------+--------+
4 rows in set (0.00 sec)
```

## Retrieve 查

```mysql
SELECT
    [DISTINCT] {* | {column [, column] ...}
    [FROM table_name]
    [WHERE ...]
    [ORDER BY column [ASC | DESC], ...]
    [LIMIT ...]
```

**全列查询**

通常情况下不建议使用 * 进行全列查询

1. 查询的列越多，意味着需要传输的数据量越大
2. 可能会影响到索引的使用。

```mysql
mysql> select * from exam_result;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
|  6 | 孙权      |      70 |   73 |      78 |
|  7 | 宋公明    |      75 |   65 |      30 |
+----+-----------+---------+------+---------+
```

**指定列查询**

```mysql
mysql> select id, name, math from exam_result;
+----+-----------+------+
| id | name      | math |
+----+-----------+------+
|  1 | 唐三藏    |   98 |
|  2 | 孙悟空    |   78 |
|  3 | 猪悟能    |   98 |
|  4 | 曹孟德    |   84 |
|  5 | 刘玄德    |   85 |
|  6 | 孙权      |   73 |
|  7 | 宋公明    |   65 |
+----+-----------+------+
7 rows in set (0.00 sec)

mysql> select id + 0 ID, name, chinese + math + english 总分 from exam_result;
+----+-----------+--------+
| ID | name      | 总分   |
+----+-----------+--------+
|  1 | 唐三藏    |    221 |
|  2 | 孙悟空    |    242 |
|  3 | 猪悟能    |    276 |
|  4 | 曹孟德    |    233 |
|  5 | 刘玄德    |    185 |
|  6 | 孙权      |    221 |
|  7 | 宋公明    |    170 |
+----+-----------+--------+
7 rows in set (0.00 sec)

mysql> select id + 0 ID, name, chinese + math + english 总分, 1111 from exam_result;
+----+-----------+--------+------+
| ID | name      | 总分   | 1111 |
+----+-----------+--------+------+
|  1 | 唐三藏    |    221 | 1111 |
|  2 | 孙悟空    |    242 | 1111 |
|  3 | 猪悟能    |    276 | 1111 |
|  4 | 曹孟德    |    233 | 1111 |
|  5 | 刘玄德    |    185 | 1111 |
|  6 | 孙权      |    221 | 1111 |
|  7 | 宋公明    |    170 | 1111 |
+----+-----------+--------+------+
7 rows in set (0.00 sec)
```

- 查询的字段可以为表达式 ： 10 + 20  ;   30 ; 666
- 表达式可以包含表中的某一个字段： math + 10
- 为查询结果**指定别名**：`SELECT column [AS] alias_name [...] FROM table_name;`   AS可以省略
- 结果可以用**distinct进行去重**

```mysql
mysql> select math from exam_result;
+------+
| math |
+------+
|   98 |
|   78 |
|   98 |
|   84 |
|   85 |
|   73 |
|   65 |
+------+
7 rows in set (0.00 sec)

mysql> select distinct math from exam_result;
+------+
| math |
+------+
|   98 |
|   78 |
|   84 |
|   85 |
|   73 |
|   65 |
+------+
```

### where条件查询

**比较运算符**

> \>, >=, <, <=               大于，大于等于，小于，小于等于 
>
> =                          等于，NULL 不安全，例如 NULL = NULL 的结果是 NULL 
>
> <=>                      等于，NULL 安全，例如 NULL <=> NULL 的结果是 TRUE(1) 
>
> !=, <>                   不等于 
>
> BETWEEN a0 AND a1 范围匹配，[a0, a1]，如果 a0 <= value <= a1，返回 TRUE(1) 
>
> IN (option, ...)     如果是 option 中的任意一个，返回 TRUE(1) 
>
> IS NULL               是 NULL
>
> IS NOT NULL      不是 NULL
>
> LIKE                     模糊匹配。% 表示任意多个（包括 0 个）任意字符；_ 表示任意一个字符

**逻辑运算符**

> AND 			多个条件必须都为 TRUE(1)，结果才是 TRUE(1) 
>
> OR 			   任意一个条件为 TRUE(1), 结果为 TRUE(1) 
>
> NOT			 条件为 TRUE(1)，结果为 FALSE(0)

```mysql
mysql> select id, name, math from exam_result where math >= 60 and math <= 100;
+----+-----------+------+
| id | name      | math |
+----+-----------+------+
|  1 | 唐三藏    |   98 |
|  2 | 孙悟空    |   78 |
|  3 | 猪悟能    |   98 |
|  4 | 曹孟德    |   84 |
|  5 | 刘玄德    |   85 |
|  6 | 孙权      |   73 |
|  7 | 宋公明    |   65 |
+----+-----------+------+
7 rows in set (0.00 sec)

mysql> select id, name, chinese from exam_result where chinese between 69 and 80;
+----+-----------+---------+
| id | name      | chinese |
+----+-----------+---------+
|  6 | 孙权      |      70 |
|  7 | 宋公明    |      75 |
+----+-----------+---------+
mysql> select id, name ,english from exam_result where english in (60,65,69, 70,72,73, 74, 80, 90);
+----+-----------+---------+
| id | name      | english |
+----+-----------+---------+
|  3 | 猪悟能    |      90 |
+----+-----------+---------+
1 row in set (0.00 sec)

mysql> select id, name from exam_result where english < math and chinese < math;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | 唐三藏    |
|  3 | 猪悟能    |
|  4 | 曹孟德    |
|  5 | 刘玄德    |
+----+-----------+
4 rows in set (0.00 sec)

SELECT name, chinese + math + english 总分 FROM exam_result
	WHERE chinese + math + english < 200;
SELECT name, chinese FROM exam_result
    WHERE chinese > 80 AND name NOT LIKE '孙%';
SELECT name, chinese, math, english, chinese + math + english 总分
FROM exam_result
WHERE name LIKE '孙_' OR (
chinese + math + english > 200 AND chinese < math AND english > 80
);
```

**NULL相关查询**

```mysql
SELECT name, qq FROM students WHERE qq IS NOT NULL;
SELECT NULL = NULL, NULL = 1, NULL = 0;
+-------------+----------+----------+
| NULL = NULL | NULL = 1 | NULL = 0 |
+-------------+----------+----------+
| NULL 	      |	  NULL   |     NULL |
+-------------+----------+----------+

SELECT NULL <=> NULL, NULL <=> 1, NULL <=> 0;
+---------------+------------+------------+
| NULL <=> NULL | NULL <=> 1 | NULL <=> 0 |
+---------------+------------+------------+
| 1             |     0      |      0     |
+---------------+------------+------------+
```

与NULL进行比较时，用<=>   IS NULL   IS NOT NULL

### 结果排序

> -- ASC 为升序（从小到大）  ascend  ascending
>
> -- DESC 为降序（从大到小）  descend  descending
>
> -- 默认为 ASC升序 
>
> SELECT ... FROM table_name [WHERE ...] 
> 	ORDER BY column [ASC|DESC], [...];

注意：没有 ORDER BY 子句的查询，返回的顺序是未定义的，永远不要依赖这个顺序

> NULL 视为比任何值都小，升序出现在最上面，降序出现在最下面

```mysql
mysql> select name, math from exam_result order by math;
+-----------+------+
| name      | math |
+-----------+------+
| 宋公明    |   65 |
| 孙权      |   73 |
| 孙悟空    |   78 |
| 曹孟德    |   84 |
| 刘玄德    |   85 |
| 唐三藏    |   98 |
| 猪悟能    |   98 |
+-----------+------+
7 rows in set (0.00 sec)

mysql> select name, math from exam_result order by math desc;
+-----------+------+
| name      | math |
+-----------+------+
| 唐三藏    |   98 |
| 猪悟能    |   98 |
| 刘玄德    |   85 |
| 曹孟德    |   84 |
| 孙悟空    |   78 |
| 孙权      |   73 |
| 宋公明    |   65 |
+-----------+------+
7 rows in set (0.00 sec)
mysql> select name, math, english, chinese from exam_result order by math desc, english, chinese desc;
+-----------+------+---------+---------+
| name      | math | english | chinese |
+-----------+------+---------+---------+
| 唐三藏    |   98 |      56 |      67 |
| 猪悟能    |   98 |      90 |      88 |
| 刘玄德    |   85 |      45 |      55 |
| 曹孟德    |   84 |      67 |      82 |
| 孙悟空    |   78 |      77 |      87 |
| 孙权      |   73 |      78 |      70 |
| 宋公明    |   65 |      30 |      75 |
+-----------+------+---------+---------+
7 rows in set (0.00 sec)
```

**-- ORDER BY 中可以使用表达式**

**-- ORDER BY 子句中可以使用列别名**

```mysql
mysql> select name, chinese+math+english 总分 from exam_result where math > 70 and name like '%%_' order by 总分 desc;
+-----------+--------+
| name      | 总分   |
+-----------+--------+
| 猪悟能    |    276 |
| 孙悟空    |    242 |
| 曹孟德    |    233 |
| 唐三藏    |    221 |
| 孙权      |    221 |
| 刘玄德    |    185 |
+-----------+--------+
6 rows in set (0.00 sec)

mysql> select name, chinese+math+english 总分 from exam_result where math > 70 and name like '___' order by 总分 desc;
+-----------+--------+
| name      | 总分   |
+-----------+--------+
| 猪悟能    |    276 |
| 孙悟空    |    242 |
| 曹孟德    |    233 |
| 唐三藏    |    221 |
| 刘玄德    |    185 |
+-----------+--------+
5 rows in set (0.00 sec)

// 查询姓孙的同学或者姓曹的同学数学成绩，结果按数学成绩由高到低显示
SELECT name, math FROM exam_result
	WHERE name LIKE '孙%' OR name LIKE '曹%'
	ORDER BY math DESC;
+-----------+--------+
| name     | math |
+-----------+--------+
| 曹孟德    | 84   |
| 孙悟空    | 78   |
| 孙权      | 73   |
+-----------+--------+
3 rows in set (0.00 sec)
```

### 筛选分页结果LIMIT

```mysql
-- 起始下标为 0
-- 从 0 开始，筛选 n 条结果
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT n;
-- 从 s 开始，筛选 n 条结果
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT s, n;
-- 从 s 开始，筛选 n 条结果，比第二种用法更明确，建议使用
SELECT ... FROM table_name [WHERE ...] [ORDER BY ...] LIMIT n OFFSET s;
-- LIMIT n offset s，可以理解为跳过s条记录，筛选出n条
```

建议：对未知表进行查询时，最好加一条 LIMIT 1，避免因为表中数据过大，查询全表数据导致数据库卡死

按 id 进行分页，每页 3 条记录，分别显示 第 1、2、3 页

```mysql
mysql> select * from exam_result order by id limit 3 offset 0;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
+----+-----------+---------+------+---------+
3 rows in set (0.00 sec)

mysql> select * from exam_result order by id limit 3 offset 3;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
|  6 | 孙权      |      70 |   73 |      78 |
+----+-----------+---------+------+---------+
3 rows in set (0.00 sec)

mysql> select * from exam_result order by id limit 3 offset 6;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  7 | 宋公明    |      75 |   65 |      30 |
+----+-----------+---------+------+---------+
1 row in set (0.00 sec)
```

## Update

```mysql
UPDATE table_name SET column = expr [, column = expr ...]
[WHERE ...] [ORDER BY ...] [LIMIT ...]
```

```mysql
select x from table
where order by limit

update table set x
where order by limit
-- select查询语句 VS update更新语句
```

```mysql
UPDATE exam_result SET math = 80 WHERE name = '孙悟空';
UPDATE exam_result SET math = 60, chinese = 70 WHERE name = '曹孟德';
-- 总分最低的三位同学的数学成绩+30分
UPDATE exam_result SET math = math + 30
ORDER BY chinese + math + english LIMIT 3;
-- 将所有同学的语文成绩更新为原来的 2 倍   注意：更新全表的语句慎用！
UPDATE exam_result SET chinese = chinese * 2;
-- 没有where筛选即更新全表
```

## Delete

```mysql
DELETE FROM table_name
[WHERE ...] [ORDER BY ...] [LIMIT ...]
```

```mysql
select x from table
where order by limit

update table set x
where order by limit

delete from table
where order by limit
-- select查询语句 VS update更新语句 VS delete删除语句
```

> 其实就是删除表中的数据时，可以先筛选（根据条件删除指定记录），可以排序（删除最后三名），limit（选择筛选指定数目记录）

```mysql
-- 删除数学小于90分的，最低的两位学生的记录
mysql> select * from exam_result;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
|  6 | 孙权      |      70 |   73 |      78 |
|  7 | 宋公明    |      75 |   65 |      30 |
+----+-----------+---------+------+---------+
7 rows in set (0.00 sec)

mysql> delete from exam_result where math < 90 order by math limit 2;
Query OK, 2 rows affected (0.00 sec)

mysql> select * from exam_result;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
+----+-----------+---------+------+---------+
5 rows in set (0.00 sec)
```

```mysql
delete from exam_result
-- 删除表中的所有记录（因为没有where筛选，所有记录都符合条件）  慎用
-- 这个和drop table有所不同，drop table是删除表，包含数据，delete from table是删除表的所有数据，剩下一个空表
```

**截断表**

```mysql
TRUNCATE [TABLE] table_name
```

注意：这个操作慎用

1. **只能对整表操作，不能像 DELETE 一样针对部分数据操作;**
2. 实际上 MySQL 不对数据操作，所以比 DELETE 更快
3. **TRUNCATE在删除数据的时候，并不经过真正的事务，所以无法回滚**
4. 会重置 AUTO_INCREMENT 项

## 插入查询结果

```mysql 
INSERT INTO table_name [(column [, column ...])] SELECT ...
INSERT INTO table_name [(column [, column ...])] VALUES ...
```

```mysql
mysql> create table for_copy(
    -> id int,
    -> name int
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> insert into for_copy values(1,100),(2,200),(3,300),(1,100),(2,200),(3,300);
Query OK, 6 rows affected (0.00 sec)
Records: 6  Duplicates: 0  Warnings: 0

mysql> select * from for_copy;
+------+------+
| id   | name |
+------+------+
|    1 |  100 |
|    2 |  200 |
|    3 |  300 |
|    1 |  100 |
|    2 |  200 |
|    3 |  300 |
+------+------+
6 rows in set (0.00 sec)

mysql> create table copy( id int, name int );
Query OK, 0 rows affected (0.01 sec)

mysql> insert into copy select distinct * from for_copy;
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from copy;
+------+------+
| id   | name |
+------+------+
|    1 |  100 |
|    2 |  200 |
|    3 |  300 |
+------+------+
3 rows in set (0.00 sec)

mysql> insert into copy select * from copy where id in(1,2);
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from copy;
+------+------+
| id   | name |
+------+------+
|    1 |  100 |
|    2 |  200 |
|    3 |  300 |
|    1 |  100 |
|    2 |  200 |
+------+------+
5 rows in set (0.00 sec)
```

## group by分组查询

**聚合函数（复合函数）**

通常对于统计数量，计算平均值，寻找最大值等操作，都可以通过复合函数来完成

---

| 函数                   | 说明                                        |
| ---------------------- | ------------------------------------------- |
| COUNT([DISTINCT] expr) | 返回查询到的数据的 数量                     |
| SUM([DISTINCT] expr)   | 返回查询到的数据的 总和，不是数字没有意义   |
| AVG([DISTINCT] expr)   | 返回查询到的数据的 平均值，不是数字没有意义 |
| MAX([DISTINCT] expr)   | 返回查询到的数据的 最大值，不是数字没有意义 |
| MIN([DISTINCT] expr)   | 返回查询到的数据的 最小值，不是数字没有意义 |

```mysql
mysql> select * from students;
+-----+-------+------+--------+
| id  | sn    | name | qq     |
+-----+-------+------+--------+
| 100 | 10004 | zzz  | 123456 |
| 102 | 10002 | Yang | NULL   |
| 103 | 10003 | Lee  | NULL   |
| 104 | 10001 | xxx  | NULL   |
+-----+-------+------+--------+
mysql> select count(1) from students;
+----------+
| count(1) |
+----------+
|        4 |
+----------+
mysql> select count(22) from students;
+-----------+
| count(22) |
+-----------+
|         4 |
+-----------+
mysql> select count(qq) from students;
+-----------+
| count(qq) |
+-----------+
|         1 |
+-----------+
1 row in set (0.00 sec)
mysql> select * from exam_result;
+----+-----------+---------+------+---------+
| id | name      | chinese | math | english |
+----+-----------+---------+------+---------+
|  1 | 唐三藏    |      67 |   98 |      56 |
|  2 | 孙悟空    |      87 |   78 |      77 |
|  3 | 猪悟能    |      88 |   98 |      90 |
|  4 | 曹孟德    |      82 |   84 |      67 |
|  5 | 刘玄德    |      55 |   85 |      45 |
+----+-----------+---------+------+---------+
5 rows in set (0.00 sec)
mysql> select SUM(math) from exam_result;
+-----------+
| SUM(math) |
+-----------+
|       443 |
+-----------+
mysql> select count(math) from exam_result;
+-------------+
| count(math) |
+-------------+
|           5 |
+-------------+
mysql> select count(distinct math) from exam_result;
+----------------------+
| count(distinct math) |
+----------------------+
|                    4 |
+----------------------+
mysql> select SUM(math) from exam_result where math < 90;
+-----------+
| SUM(math) |
+-----------+
|       247 |
+-----------+
mysql> select avg(chinese+math+english) from exam_result where math < 90;
+---------------------------+
| avg(chinese+math+english) |
+---------------------------+
|                       220 |
+---------------------------+
mysql> select max(english) from exam_result where english < 80;
+--------------+
| max(english) |
+--------------+
|           77 |
+--------------+
```

sum avg max min都很清晰。

其实count就是求数量的，和参数传进去什么东西没关系。有几个记录就返回几。注意count计数时，NULL不会计入结果。（没必要纠结一些没太大意义的细节，比如count传入什么参数）

---

在select中使用group by子句可以对指定列进行分组查询。

<u>**注意：SELECT指定的字段必须是“分组依据字段”，其他字段若想出现在SELECT中则必须包含在聚合函数中。**</u>

```mysql
mysql> show tables;
+-----------------+
| Tables_in_scott |
+-----------------+
| dept            |
| emp             |
| salgrade        |
+-----------------+
3 rows in set (0.00 sec)

mysql> select * from dept;    // 部门表: 部门编号, 部门名, 部门所在地址
+--------+------------+----------+
| deptno | dname      | loc      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)

mysql> select * from emp;    // 员工表: 员工编号, 名, 工作, 上司编号, 录用时间, 工资, 奖金, 所在部门编号
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| empno  | ename  | job       | mgr  | hiredate            | sal     | comm    | deptno |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| 007369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
| 007499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
| 007521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
| 007566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
| 007654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
| 007698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
| 007782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
| 007788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 |    NULL |     20 |
| 007839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
| 007844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
| 007876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100.00 |    NULL |     20 |
| 007900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
| 007902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
| 007934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
14 rows in set (0.00 sec)

mysql> select * from salgrade;   // 工资等级划分表
+-------+-------+-------+
| grade | losal | hisal |
+-------+-------+-------+
|     1 |   700 |  1200 |
|     2 |  1201 |  1400 |
|     3 |  1401 |  2000 |
|     4 |  2001 |  3000 |
|     5 |  3001 |  9999 |
+-------+-------+-------+
5 rows in set (0.00 sec)
```

```mysql
-- 按照部门编号进行分组，显示每个部门的平均工资和最高工资：deptno是分组依据字段，sal若想打印出来，则，必须通过聚合函数。
mysql> select deptno,avg(sal),max(sal) from emp group by deptno;
+--------+-------------+----------+
| deptno | avg(sal)    | max(sal) |
+--------+-------------+----------+
|     10 | 2916.666667 |  5000.00 |
|     20 | 2175.000000 |  3000.00 |
|     30 | 1566.666667 |  2850.00 |
+--------+-------------+----------+
3 rows in set (0.00 sec)
-- 按照部门编号分组之后再按工作岗位分组，显示最低工资和平均工资：job，deptno为分组依据字段
mysql> select avg(sal),min(sal),job,deptno from emp group by deptno,job;
+-------------+----------+-----------+--------+
| avg(sal)    | min(sal) | job       | deptno |
+-------------+----------+-----------+--------+
| 1300.000000 |  1300.00 | CLERK     |     10 |
| 2450.000000 |  2450.00 | MANAGER   |     10 |
| 5000.000000 |  5000.00 | PRESIDENT |     10 |
| 3000.000000 |  3000.00 | ANALYST   |     20 |
|  950.000000 |   800.00 | CLERK     |     20 |
| 2975.000000 |  2975.00 | MANAGER   |     20 |
|  950.000000 |   950.00 | CLERK     |     30 |
| 2850.000000 |  2850.00 | MANAGER   |     30 |
| 1400.000000 |  1250.00 | SALESMAN  |     30 |
+-------------+----------+-----------+--------+
9 rows in set (0.00 sec)
-- 错误!!!!!!
mysql> select avg(sal) from emp group by deptno where avg(sal) < 2000;    
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'where avg(sal) < 2000' at line 1
mysql> select avg(sal) from emp group by deptno;
+-------------+
| avg(sal)    |
+-------------+
| 2916.666667 |
| 2175.000000 |
| 1566.666667 |
+-------------+
3 rows in set (0.00 sec)

mysql> select avg(sal) from emp group by deptno having avg(sal) < 2000;
+-------------+
| avg(sal)    |
+-------------+
| 1566.666667 |
+-------------+
1 row in set (0.00 sec)

mysql> select avg(sal) avg_sal from emp group by deptno having avg_sal < 2000;
+-------------+
| avg_sal     |
+-------------+
| 1566.666667 |
+-------------+
1 row in set (0.00 sec)

mysql> select deptno, avg(sal) avg_sal from emp group by deptno having avg_sal < 2000;
+--------+-------------+
| deptno | avg_sal     |
+--------+-------------+
|     30 | 1566.666667 |
+--------+-------------+
1 row in set (0.00 sec)
```

- 使用GROUP BY进行分组，如果需要使用条件判断来过滤数据，就不能再使用WHERE，而是要使用**HAVING语法**

> 面试题：SQL查询中各个关键字的执行先后顺序 from > on> join > where > group by > with > having > select > distinct > order by > limit
