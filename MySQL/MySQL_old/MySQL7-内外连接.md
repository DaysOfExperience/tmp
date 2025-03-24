**表的连接分为内连和外连**

# 内连接

**内连接实际上就是利用where子句对两种表形成的笛卡儿积进行筛选**，我们前面学习的查询都是内连接，也是在开发过程中使用的最多的连接查询。

`select 字段 from 表1 inner join 表2 on 连接条件 and 其他条件；`

> 备注：前面学习的都是内连接

```mysql
-- 显示SMITH的名字和部门名称
-- 用前面的写法
mysql> select ename, dname from emp, dept where emp.deptno = dept.deptno and emp.ename='SMITH';
+-------+----------+
| ename | dname    |
+-------+----------+
| SMITH | RESEARCH |
+-------+----------+
1 row in set (0.00 sec)

-- 用标准的内连接写法
mysql> select ename, dname from emp inner join dept on emp.deptno = dept.deptno and ename = 'SMITH';
+-------+----------+
| ename | dname    |
+-------+----------+
| SMITH | RESEARCH |
+-------+----------+
1 row in set (0.00 sec)
-- 后面还可以加where ... and ...
```

# 外连接

**外连接分为左外连接和右外连接**

**如果联合查询，左侧的表完全显示我们就说是左外连接。**

**如果联合查询，右侧的表完全显示我们就说是右外连接。**

```mysql
-- 左外连接
select 字段 from 表1 left join 表2 on 连接条件
-- 右外连接
select 字段 from 表1 right join 表2 on 连接条件

-- 内连接 - 多表查询
select 字段 from 表1 inner join 表2 on 连接条件 and 其他条件；
```

```mysql
mysql> select * from stu;
+------+------+
| id   | name |
+------+------+
|    1 | jack |
|    2 | tom  |
|    3 | kity |
|    4 | nono |
+------+------+
4 rows in set (0.00 sec)

mysql> select * from exam
    -> ;
+------+-------+
| id   | grade |
+------+-------+
|    1 |    56 |
|    2 |    76 |
|   11 |     8 |
+------+-------+
3 rows in set (0.00 sec)

mysql> select * from stu left join exam where stu.id = exam.id;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'where stu.id = exam.id' at line 1

-- 查询所有学生的成绩，如果这个学生没有成绩，也要将学生的个人信息显示出来
-- 当左边表和右边表没有匹配时，也会显示左边表的数据
mysql> select * from stu left join exam on stu.id = exam.id;
+------+------+------+-------+
| id   | name | id   | grade |
+------+------+------+-------+
|    1 | jack |    1 |    56 |
|    2 | tom  |    2 |    76 |
|    3 | kity | NULL |  NULL |
|    4 | nono | NULL |  NULL |
+------+------+------+-------+
4 rows in set (0.00 sec)

-- 对stu表和exam表联合查询，把所有的成绩都显示出来，即使这个成绩没有学生与它对应，也要显示出来
mysql> select * from stu right join exam on stu.id=exam.id;
+------+------+------+-------+
| id   | name | id   | grade |
+------+------+------+-------+
|    1 | jack |    1 |    56 |
|    2 | tom  |    2 |    76 |
| NULL | NULL |   11 |     8 |
+------+------+------+-------+
3 rows in set (0.00 sec)

mysql> use scott;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

-- 列出部门名称和这些部门的员工信息，同时列出没有员工的部门
mysql> select dname, emp.* from dept left join emp on dept.deptno = emp.deptno;
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
| dname      | empno  | ename  | job       | mgr  | hiredate            | sal     | comm    | deptno |
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
| RESEARCH   | 007369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
| SALES      | 007499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
| SALES      | 007521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
| RESEARCH   | 007566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
| SALES      | 007654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
| SALES      | 007698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
| ACCOUNTING | 007782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
| RESEARCH   | 007788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 |    NULL |     20 |
| ACCOUNTING | 007839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
| SALES      | 007844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
| RESEARCH   | 007876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100.00 |    NULL |     20 |
| SALES      | 007900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
| RESEARCH   | 007902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
| ACCOUNTING | 007934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
| OPERATIONS |   NULL | NULL   | NULL      | NULL | NULL                |    NULL |    NULL |   NULL |
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
15 rows in set (0.00 sec)

mysql> select dname, emp.* from emp right join dept on dept.deptno = emp.deptno;
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
| dname      | empno  | ename  | job       | mgr  | hiredate            | sal     | comm    | deptno |
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
| RESEARCH   | 007369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
| SALES      | 007499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
| SALES      | 007521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
| RESEARCH   | 007566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
| SALES      | 007654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
| SALES      | 007698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
| ACCOUNTING | 007782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
| RESEARCH   | 007788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 |    NULL |     20 |
| ACCOUNTING | 007839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
| SALES      | 007844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
| RESEARCH   | 007876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100.00 |    NULL |     20 |
| SALES      | 007900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
| RESEARCH   | 007902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
| ACCOUNTING | 007934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
| OPERATIONS |   NULL | NULL   | NULL      | NULL | NULL                |    NULL |    NULL |   NULL |
+------------+--------+--------+-----------+------+---------------------+---------+---------+--------+
15 rows in set (0.00 sec)
```