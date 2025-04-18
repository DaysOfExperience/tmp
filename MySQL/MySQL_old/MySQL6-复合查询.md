> 前言：函数略了

---

**前面我们讲解的mysql表的查询都是对一张表进行查询，在实际开发中这远远不够。**

# 复合查询

```mysql
-- 查询工资高于500或岗位为MANAGER的雇员，同时还要满足他们的姓名首字母为大写的J
mysql> select * from emp
    -> where (sal > 500 or job='MANAGER') and ename like 'J%';
+--------+-------+---------+------+---------------------+---------+------+--------+
| empno  | ename | job     | mgr  | hiredate            | sal     | comm | deptno |
+--------+-------+---------+------+---------------------+---------+------+--------+
| 007566 | JONES | MANAGER | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
| 007900 | JAMES | CLERK   | 7698 | 1981-12-03 00:00:00 |  950.00 | NULL |     30 |
+--------+-------+---------+------+---------------------+---------+------+--------+
2 rows in set (0.00 sec)
-- 按照部门号升序而雇员的工资降序排序
mysql> select * from emp order by deptno, sal desc;
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| empno  | ename  | job       | mgr  | hiredate            | sal     | comm    | deptno |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| 007839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
| 007782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
| 007934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
| 007788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 |    NULL |     20 |
| 007902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
| 007566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
| 007876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100.00 |    NULL |     20 |
| 007369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
| 007698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
| 007499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
| 007844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
| 007521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
| 007654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
| 007900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
14 rows in set (0.00 sec)
-- 使用年薪进行降序排序
select ename, sal*12+ifnull(comm,0) as '年薪' from EMP order by 年薪 desc;

mysql> select * from emp order by sal * 12 desc
    -> ;
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| empno  | ename  | job       | mgr  | hiredate            | sal     | comm    | deptno |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
| 007839 | KING   | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 |    NULL |     10 |
| 007788 | SCOTT  | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 |    NULL |     20 |
| 007902 | FORD   | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 |    NULL |     20 |
| 007566 | JONES  | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 |    NULL |     20 |
| 007698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 |    NULL |     30 |
| 007782 | CLARK  | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 |    NULL |     10 |
| 007499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 00:00:00 | 1600.00 |  300.00 |     30 |
| 007844 | TURNER | SALESMAN  | 7698 | 1981-09-08 00:00:00 | 1500.00 |    0.00 |     30 |
| 007934 | MILLER | CLERK     | 7782 | 1982-01-23 00:00:00 | 1300.00 |    NULL |     10 |
| 007521 | WARD   | SALESMAN  | 7698 | 1981-02-22 00:00:00 | 1250.00 |  500.00 |     30 |
| 007654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 00:00:00 | 1250.00 | 1400.00 |     30 |
| 007876 | ADAMS  | CLERK     | 7788 | 1987-05-23 00:00:00 | 1100.00 |    NULL |     20 |
| 007900 | JAMES  | CLERK     | 7698 | 1981-12-03 00:00:00 |  950.00 |    NULL |     30 |
| 007369 | SMITH  | CLERK     | 7902 | 1980-12-17 00:00:00 |  800.00 |    NULL |     20 |
+--------+--------+-----------+------+---------------------+---------+---------+--------+
14 rows in set (0.01 sec)

-- 显示工资最高的员工的名字和工作岗位
mysql> select ename, job from emp where ename = (select ename from emp order by sal desc limit 1);
+-------+-----------+
| ename | job       |
+-------+-----------+
| KING  | PRESIDENT |
+-------+-----------+
1 row in set (0.00 sec)
-- 下面这个更严谨正确，因为工资最高的可能不止一个
mysql> select ename, job from emp where sal = (select max(sal) from emp);
+-------+-----------+
| ename | job       |
+-------+-----------+
| KING  | PRESIDENT |
+-------+-----------+
1 row in set (0.00 sec)
-- 显示工资高于平均工资的员工信息
mysql> select * from emp where sal > (select avg(sal) from emp);
+--------+-------+-----------+------+---------------------+---------+------+--------+
| empno  | ename | job       | mgr  | hiredate            | sal     | comm | deptno |
+--------+-------+-----------+------+---------------------+---------+------+--------+
| 007566 | JONES | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
| 007698 | BLAKE | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 | NULL |     30 |
| 007782 | CLARK | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 | NULL |     10 |
| 007788 | SCOTT | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 | NULL |     20 |
| 007839 | KING  | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 | NULL |     10 |
| 007902 | FORD  | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 | NULL |     20 |
+--------+-------+-----------+------+---------------------+---------+------+--------+

-- 显示每个部门的平均工资和最高工资
mysql> select deptno, avg(sal), max(sal) from emp group by deptno;
+--------+-------------+----------+
| deptno | avg(sal)    | max(sal) |
+--------+-------------+----------+
|     10 | 2916.666667 |  5000.00 |
|     20 | 2175.000000 |  3000.00 |
|     30 | 1566.666667 |  2850.00 |
+--------+-------------+----------+
3 rows in set (0.00 sec)
mysql> select deptno, format(avg(sal), 2) , max(sal) from emp group by deptno;
+--------+---------------------+----------+
| deptno | format(avg(sal), 2) | max(sal) |
+--------+---------------------+----------+
|     10 | 2,916.67            |  5000.00 |
|     20 | 2,175.00            |  3000.00 |
|     30 | 1,566.67            |  2850.00 |
+--------+---------------------+----------+
3 rows in set (0.00 sec)
-- 显示平均工资低于2000的部门号和它的平均工资
mysql> select deptno, avg(sal) from emp group by deptno having avg(sal) < 2000;  // having!!!~~~
+--------+-------------+
| deptno | avg(sal)    |
+--------+-------------+
|     30 | 1566.666667 |
+--------+-------------+
1 row in set (0.00 sec)
-- 显示每种岗位的雇员总数，平均工资
mysql> select job, count(*), avg(sal) from emp group by job;
+-----------+----------+-------------+
| job       | count(*) | avg(sal)    |
+-----------+----------+-------------+
| ANALYST   |        2 | 3000.000000 |
| CLERK     |        4 | 1037.500000 |
| MANAGER   |        3 | 2758.333333 |
| PRESIDENT |        1 | 5000.000000 |
| SALESMAN  |        4 | 1400.000000 |
+-----------+----------+-------------+
5 rows in set (0.00 sec)
```

## 多表查询

**实际开发中数据往往来自不同的表，所以需要多表查询。**用一个简单的公司管理系统，有三张表EMP,DEPT,SALGRADE来演示如何进行多表查询。

```mysql
-- 显示雇员名、雇员工资以及所在部门的名字
-- 因为上面的数据来自EMP和DEPT表，因此要联合查询

mysql> select emp.ename, emp.sal, dept.dname from emp, dept where emp.deptno=dept.deptno;
+--------+---------+------------+
| ename  | sal     | dname      |
+--------+---------+------------+
| SMITH  |  800.00 | RESEARCH   |
| ALLEN  | 1600.00 | SALES      |
| WARD   | 1250.00 | SALES      |
| JONES  | 2975.00 | RESEARCH   |
| MARTIN | 1250.00 | SALES      |
| BLAKE  | 2850.00 | SALES      |
| CLARK  | 2450.00 | ACCOUNTING |
| SCOTT  | 3000.00 | RESEARCH   |
| KING   | 5000.00 | ACCOUNTING |
| TURNER | 1500.00 | SALES      |
| ADAMS  | 1100.00 | RESEARCH   |
| JAMES  |  950.00 | SALES      |
| FORD   | 3000.00 | RESEARCH   |
| MILLER | 1300.00 | ACCOUNTING |
+--------+---------+------------+
14 rows in set (0.00 sec)
-- 显示部门号为10的部门名，员工名和工资

mysql> select dname, ename, sal from emp,dept where emp.deptno = dept.deptno and deptno=10;
ERROR 1052 (23000): Column 'deptno' in where clause is ambiguous
mysql> select dname, ename, sal from emp,dept where emp.deptno = dept.deptno and emp.deptno=10;
+------------+--------+---------+
| dname      | ename  | sal     |
+------------+--------+---------+
| ACCOUNTING | CLARK  | 2450.00 |
| ACCOUNTING | KING   | 5000.00 |
| ACCOUNTING | MILLER | 1300.00 |
+------------+--------+---------+
3 rows in set (0.00 sec)
-- 显示各个员工的姓名，工资，及工资级别
mysql> select * from salgrade;
+-------+-------+-------+
| grade | losal | hisal |
+-------+-------+-------+
|     1 |   700 |  1200 |
|     2 |  1201 |  1400 |
|     3 |  1401 |  2000 |
|     4 |  2001 |  3000 |
|     5 |  3001 |  9999 |
+-------+-------+-------+
5 rows in set (0.01 sec)
select ename, sal, grade from EMP, SALGRADE where EMP.sal between losal and hisal;
mysql> select ename, sal, grade from emp,salgrade where sal > losal and sal < hisal;
+--------+---------+-------+
| ename  | sal     | grade |
+--------+---------+-------+
| SMITH  |  800.00 |     1 |
| ALLEN  | 1600.00 |     3 |
| WARD   | 1250.00 |     2 |
| JONES  | 2975.00 |     4 |
| MARTIN | 1250.00 |     2 |
| BLAKE  | 2850.00 |     4 |
| CLARK  | 2450.00 |     4 |
| KING   | 5000.00 |     5 |
| TURNER | 1500.00 |     3 |
| ADAMS  | 1100.00 |     1 |
| JAMES  |  950.00 |     1 |
| MILLER | 1300.00 |     2 |
+--------+---------+-------+
12 rows in set (0.00 sec)
```

## 自连接

**自连接是指在同一张表连接查询**

显示员工FORD的上级领导的编号和姓名（mgr是员工领导的编号--empno）

```mysql
mysql> select * from emp;
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

-- 使用的子查询 真的更简单方便一些
mysql> select empno, ename from emp where empno = (select mgr from emp where ename = 'FORD');
+--------+-------+
| empno  | ename |
+--------+-------+
| 007566 | JONES |
+--------+-------+
1 row in set (0.00 sec)

-- 使用多表查询（自查询）
-- 使用到表的别名
-- from emp leader, emp worker，给自己的表起别名，因为要先做笛卡尔积，所以别名可以先识别

mysql> select empno, ename from emp e1, emp e2 where e1.mgr = e2.empno and e1.ename = 'FORD';
ERROR 1052 (23000): Column 'empno' in field list is ambiguous
mysql> select e1.empno, e1.ename from emp e1, emp e2 where e1.mgr = e2.empno and e1.ename = 'FORD';
+--------+-------+
| empno  | ename |
+--------+-------+
| 007902 | FORD  |
+--------+-------+
1 row in set (0.00 sec)

mysql> select e1.empno, e1.ename from emp e1, emp e2 where e1.mgr = e2.empno;
+--------+--------+
| empno  | ename  |
+--------+--------+
| 007788 | SCOTT  |
| 007902 | FORD   |
| 007499 | ALLEN  |
| 007521 | WARD   |
| 007654 | MARTIN |
| 007844 | TURNER |
| 007900 | JAMES  |
| 007934 | MILLER |
| 007876 | ADAMS  |
| 007566 | JONES  |
| 007698 | BLAKE  |
| 007782 | CLARK  |
| 007369 | SMITH  |
+--------+--------+
13 rows in set (0.00 sec)

mysql> select e2.empno, e2.ename from emp e1, emp e2 where e1.mgr = e2.empno and e1.ename = 'FORD';
+--------+-------+
| empno  | ename |
+--------+-------+
| 007566 | JONES |
+--------+-------+
1 row in set (0.00 sec)

-- 最佳方式，所以说，这种情况下的别名很重要~
select leader.empno,leader.ename from emp leader, emp worker where
leader.empno = worker.mgr and worker.ename='FORD';
```

## 子查询

**子查询是指嵌入在其他sql语句中的select语句，也叫嵌套查询**

- 单行子查询是指**子查询只返回单列单行数据**；
- 多行**子查询是指返回单列多行数据**，都是针对单列而言的
- 多列子查询则是指**子查询返回多列数据的子查询语句**

### 单行子查询

```mysql
-- 显示SMITH同一部门的员工
mysql> select * from emp where deptno = (select deptno from emp where ename = 'SMITH');
+--------+-------+---------+------+---------------------+---------+------+--------+
| empno  | ename | job     | mgr  | hiredate            | sal     | comm | deptno |
+--------+-------+---------+------+---------------------+---------+------+--------+
| 007369 | SMITH | CLERK   | 7902 | 1980-12-17 00:00:00 |  800.00 | NULL |     20 |
| 007566 | JONES | MANAGER | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
| 007788 | SCOTT | ANALYST | 7566 | 1987-04-19 00:00:00 | 3000.00 | NULL |     20 |
| 007876 | ADAMS | CLERK   | 7788 | 1987-05-23 00:00:00 | 1100.00 | NULL |     20 |
| 007902 | FORD  | ANALYST | 7566 | 1981-12-03 00:00:00 | 3000.00 | NULL |     20 |
+--------+-------+---------+------+---------------------+---------+------+--------+
5 rows in set (0.01 sec)
```

### 多行子查询

```mysql
-- in all any 关键字 --
-- 查询和10号部门的工作岗位相同的雇员的名字，岗位，工资，部门号，但是不包含10自己的

mysql> select ename, job, sal, deptno from emp where job in (select job from emp where deptno = 10) and deptno != 10;
+-------+---------+---------+--------+
| ename | job     | sal     | deptno |
+-------+---------+---------+--------+
| JONES | MANAGER | 2975.00 |     20 |
| BLAKE | MANAGER | 2850.00 |     30 |
| SMITH | CLERK   |  800.00 |     20 |
| ADAMS | CLERK   | 1100.00 |     20 |
| JAMES | CLERK   |  950.00 |     30 |
+-------+---------+---------+--------+
5 rows in set (0.00 sec)
-- 显示工资比部门30的所有员工的工资高的员工的姓名、工资和部门号

mysql> select ename, sal, deptno from emp where sal > all(select sal from emp where deptno = 30);
+-------+---------+--------+
| ename | sal     | deptno |
+-------+---------+--------+
| JONES | 2975.00 |     20 |
| SCOTT | 3000.00 |     20 |
| KING  | 5000.00 |     10 |
| FORD  | 3000.00 |     20 |
+-------+---------+--------+
4 rows in set (0.00 sec)

mysql> select ename, sal, deptno from emp where sal > (select max(sal) from emp where deptno = 30);
+-------+---------+--------+
| ename | sal     | deptno |
+-------+---------+--------+
| JONES | 2975.00 |     20 |
| SCOTT | 3000.00 |     20 |
| KING  | 5000.00 |     10 |
| FORD  | 3000.00 |     20 |
+-------+---------+--------+
4 rows in set (0.00 sec)

-- 显示工资比部门30的任意员工的工资高的员工的姓名、工资和部门号（包含自己部门的员工）

mysql> select ename, sal, deptno from emp where sal > any(select sal from emp where deptno = 30);
+--------+---------+--------+
| ename  | sal     | deptno |
+--------+---------+--------+
| ALLEN  | 1600.00 |     30 |
| WARD   | 1250.00 |     30 |
| JONES  | 2975.00 |     20 |
| MARTIN | 1250.00 |     30 |
| BLAKE  | 2850.00 |     30 |
| CLARK  | 2450.00 |     10 |
| SCOTT  | 3000.00 |     20 |
| KING   | 5000.00 |     10 |
| TURNER | 1500.00 |     30 |
| ADAMS  | 1100.00 |     20 |
| FORD   | 3000.00 |     20 |
| MILLER | 1300.00 |     10 |
+--------+---------+--------+
12 rows in set (0.00 sec)

mysql> select ename, sal, deptno from emp where sal > (select min(sal) from emp where deptno = 30);
+--------+---------+--------+
| ename  | sal     | deptno |
+--------+---------+--------+
| ALLEN  | 1600.00 |     30 |
| WARD   | 1250.00 |     30 |
| JONES  | 2975.00 |     20 |
| MARTIN | 1250.00 |     30 |
| BLAKE  | 2850.00 |     30 |
| CLARK  | 2450.00 |     10 |
| SCOTT  | 3000.00 |     20 |
| KING   | 5000.00 |     10 |
| TURNER | 1500.00 |     30 |
| ADAMS  | 1100.00 |     20 |
| FORD   | 3000.00 |     20 |
| MILLER | 1300.00 |     10 |
+--------+---------+--------+
12 rows in set (0.00 sec)

mysql> select ename, sal, deptno from emp where sal > (select min(sal) from emp where deptno = 30) and deptno != 30;
+--------+---------+--------+
| ename  | sal     | deptno |
+--------+---------+--------+
| JONES  | 2975.00 |     20 |
| CLARK  | 2450.00 |     10 |
| SCOTT  | 3000.00 |     20 |
| KING   | 5000.00 |     10 |
| ADAMS  | 1100.00 |     20 |
| FORD   | 3000.00 |     20 |
| MILLER | 1300.00 |     10 |
+--------+---------+--------+
7 rows in set (0.01 sec)
```

### 多列子查询

where (name, job) = (select xxxx)

多列嘛

```mysql
-- 查询和SMITH的部门和岗位完全相同的所有雇员，不含SMITH本人

mysql> select * from emp where (deptno, job) = (select deptno, job from emp where ename = 'SMITH') and ename != 'SMITH';
+--------+-------+-------+------+---------------------+---------+------+--------+
| empno  | ename | job   | mgr  | hiredate            | sal     | comm | deptno |
+--------+-------+-------+------+---------------------+---------+------+--------+
| 007876 | ADAMS | CLERK | 7788 | 1987-05-23 00:00:00 | 1100.00 | NULL |     20 |
+--------+-------+-------+------+---------------------+---------+------+--------+
1 row in set (0.00 sec)
```

### 在from子句中使用子查询

子查询语句出现在from子句中, **把一个子查询当做一个临时表使用。**

```mysql
-- 显示每个高于自己部门平均工资的员工的姓名、部门、工资、平均工资

mysql> select e2.ename, e2.deptno, e2.sal, e1.av from (select deptno, avg(sal) av from emp group by deptno) e1, emp e2;
+--------+--------+---------+-------------+
| ename  | deptno | sal     | av          |
+--------+--------+---------+-------------+
| SMITH  |     20 |  800.00 | 2916.666667 |
| SMITH  |     20 |  800.00 | 2175.000000 |
| SMITH  |     20 |  800.00 | 1566.666667 |
| ALLEN  |     30 | 1600.00 | 2916.666667 |
| ALLEN  |     30 | 1600.00 | 2175.000000 |
| ALLEN  |     30 | 1600.00 | 1566.666667 |
| WARD   |     30 | 1250.00 | 2916.666667 |
| WARD   |     30 | 1250.00 | 2175.000000 |
| WARD   |     30 | 1250.00 | 1566.666667 |
| JONES  |     20 | 2975.00 | 2916.666667 |
| JONES  |     20 | 2975.00 | 2175.000000 |
| JONES  |     20 | 2975.00 | 1566.666667 |
| MARTIN |     30 | 1250.00 | 2916.666667 |
| MARTIN |     30 | 1250.00 | 2175.000000 |
| MARTIN |     30 | 1250.00 | 1566.666667 |
| BLAKE  |     30 | 2850.00 | 2916.666667 |
| BLAKE  |     30 | 2850.00 | 2175.000000 |
| BLAKE  |     30 | 2850.00 | 1566.666667 |
| CLARK  |     10 | 2450.00 | 2916.666667 |
| CLARK  |     10 | 2450.00 | 2175.000000 |
| CLARK  |     10 | 2450.00 | 1566.666667 |
| SCOTT  |     20 | 3000.00 | 2916.666667 |
| SCOTT  |     20 | 3000.00 | 2175.000000 |
| SCOTT  |     20 | 3000.00 | 1566.666667 |
| KING   |     10 | 5000.00 | 2916.666667 |
| KING   |     10 | 5000.00 | 2175.000000 |
| KING   |     10 | 5000.00 | 1566.666667 |
| TURNER |     30 | 1500.00 | 2916.666667 |
| TURNER |     30 | 1500.00 | 2175.000000 |
| TURNER |     30 | 1500.00 | 1566.666667 |
| ADAMS  |     20 | 1100.00 | 2916.666667 |
| ADAMS  |     20 | 1100.00 | 2175.000000 |
| ADAMS  |     20 | 1100.00 | 1566.666667 |
| JAMES  |     30 |  950.00 | 2916.666667 |
| JAMES  |     30 |  950.00 | 2175.000000 |
| JAMES  |     30 |  950.00 | 1566.666667 |
| FORD   |     20 | 3000.00 | 2916.666667 |
| FORD   |     20 | 3000.00 | 2175.000000 |
| FORD   |     20 | 3000.00 | 1566.666667 |
| MILLER |     10 | 1300.00 | 2916.666667 |
| MILLER |     10 | 1300.00 | 2175.000000 |
| MILLER |     10 | 1300.00 | 1566.666667 |
+--------+--------+---------+-------------+
42 rows in set (0.01 sec)

mysql> select e2.ename, e2.deptno, e2.sal, e1.av from (select deptno, avg(sal) av from emp group by deptno) e1, emp e2 where e1.deptno = e2.deptno;
+--------+--------+---------+-------------+
| ename  | deptno | sal     | av          |
+--------+--------+---------+-------------+
| SMITH  |     20 |  800.00 | 2175.000000 |
| ALLEN  |     30 | 1600.00 | 1566.666667 |
| WARD   |     30 | 1250.00 | 1566.666667 |
| JONES  |     20 | 2975.00 | 2175.000000 |
| MARTIN |     30 | 1250.00 | 1566.666667 |
| BLAKE  |     30 | 2850.00 | 1566.666667 |
| CLARK  |     10 | 2450.00 | 2916.666667 |
| SCOTT  |     20 | 3000.00 | 2175.000000 |
| KING   |     10 | 5000.00 | 2916.666667 |
| TURNER |     30 | 1500.00 | 1566.666667 |
| ADAMS  |     20 | 1100.00 | 2175.000000 |
| JAMES  |     30 |  950.00 | 1566.666667 |
| FORD   |     20 | 3000.00 | 2175.000000 |
| MILLER |     10 | 1300.00 | 2916.666667 |
+--------+--------+---------+-------------+
14 rows in set (0.00 sec)

mysql> select e2.ename, e2.deptno, e2.sal, e1.av from (select deptno, avg(sal) av from emp group by deptno) e1, emp e2 where e1.deptno = e2.deptno and e2.sal > e1.av;
+-------+--------+---------+-------------+
| ename | deptno | sal     | av          |
+-------+--------+---------+-------------+
| ALLEN |     30 | 1600.00 | 1566.666667 |
| JONES |     20 | 2975.00 | 2175.000000 |
| BLAKE |     30 | 2850.00 | 1566.666667 |
| SCOTT |     20 | 3000.00 | 2175.000000 |
| KING  |     10 | 5000.00 | 2916.666667 |
| FORD  |     20 | 3000.00 | 2175.000000 |
+-------+--------+---------+-------------+
6 rows in set (0.00 sec)
-- 参考
select ename, deptno, sal, format(asal,2) from EMP,
(select avg(sal) asal, deptno dt from EMP group by deptno) tmp
where EMP.sal > tmp.asal and EMP.deptno=tmp.dt;


-- 查找每个部门工资最高的人的姓名、工资、部门、最高工资

mysql> select etable.ename, etable.sal, etable.deptno, mtable.maxSal from (select deptno, max(sal) maxSal from emp group by deptno) mtable, emp etable where mtable.deptno = etable.deptno and etable.sal = mtable.maxSal;
+-------+---------+--------+---------+
| ename | sal     | deptno | maxSal  |
+-------+---------+--------+---------+
| BLAKE | 2850.00 |     30 | 2850.00 |
| SCOTT | 3000.00 |     20 | 3000.00 |
| KING  | 5000.00 |     10 | 5000.00 |
| FORD  | 3000.00 |     20 | 3000.00 |
+-------+---------+--------+---------+
4 rows in set (0.00 sec)
// select EMP.ename, EMP.sal, EMP.deptno, ms from EMP,
(select max(sal) ms, deptno from EMP group by deptno) tmp
where EMP.deptno=tmp.deptno and EMP.sal=tmp.ms;

-- 显示每个部门的信息（部门名，编号，地址）和人员数量
mysql> show tables;
+-----------------+
| Tables_in_scott |
+-----------------+
| dept            |
| emp             |
| salgrade        |
+-----------------+
3 rows in set (0.00 sec)

mysql> select * from dept;
+--------+------------+----------+
| deptno | dname      | loc      |
+--------+------------+----------+
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |
+--------+------------+----------+
4 rows in set (0.00 sec)

--  使用子查询的方法
mysql> select d.dname, d.deptno, d.loc, cnt from (select deptno, count(*) cnt from emp group by deptno) c, dept d where d.deptno = c.deptno;
+------------+--------+----------+-----+
| dname      | deptno | loc      | cnt |
+------------+--------+----------+-----+
| ACCOUNTING |     10 | NEW YORK |   3 |
| RESEARCH   |     20 | DALLAS   |   5 |
| SALES      |     30 | CHICAGO  |   6 |
+------------+--------+----------+-----+
3 rows in set (0.00 sec)
// -- 1. 对EMP表进行人员统计
select count(*), deptno from EMP group by deptno;
-- 2. 将上面的表看作临时表
select DEPT.deptno, dname, mycnt, loc from DEPT,
(select count(*) mycnt, deptno from EMP group by deptno) tmp
where DEPT.deptno=tmp.deptno;

-- 使用多表的方法
mysql> select d.dname, d.deptno, d.loc, count(*) 部门人数 from dept, emp where dept.deptno = emp.deptno group by deptno;
ERROR 1054 (42S22): Unknown column 'd.dname' in 'field list'
mysql> select d.dname, d.deptno, d.loc, count(*) 部门人数 from dept, emp where dept.deptno = emp.deptno group by d.deptno;
ERROR 1054 (42S22): Unknown column 'd.dname' in 'field list'
mysql> select d.dname, d.deptno, d.loc, count(*) 部门人数 from dept d, emp e where d.deptno = e.deptno group by d.deptno;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'scott.d.dname' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
mysql> select d.dname, d.deptno, d.loc, count(*) 部门人数 from dept d, emp e where d.deptno = e.deptno group by d.deptno, d.dname, d.loc;
+------------+--------+----------+--------------+
| dname      | deptno | loc      | 部门人数     |
+------------+--------+----------+--------------+
| ACCOUNTING |     10 | NEW YORK |            3 |
| RESEARCH   |     20 | DALLAS   |            5 |
| SALES      |     30 | CHICAGO  |            6 |
+------------+--------+----------+--------------+
3 rows in set (0.00 sec)
```

针对最后这个例子 : **注意：SELECT指定的字段必须是“分组依据字段”，其他字段若想出现在SELECT中则必须包含在聚合函数中。**

## 合并查询

在实际应用中，为了合并多个select的执行结果，可以使用集合操作符 union，union all

**union** 该操作符用于取得两个结果集的并集。当使用该操作符时，**会自动去掉结果集中的重复行。**

**union all** 该操作符用于取得两个结果集的并集。当使用该操作符时，**不会去掉结果集中的重复行。**

```mysql
-- 将工资大于2500或职位是MANAGER的人找出来

mysql> select * from emp where sal > 2500 
    -> union
    -> select * from emp where job = 'MANAGER';
+-------+-------+-----------+------+---------------------+---------+------+--------+
| empno | ename | job       | mgr  | hiredate            | sal     | comm | deptno |
+-------+-------+-----------+------+---------------------+---------+------+--------+
|  7566 | JONES | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
|  7698 | BLAKE | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 | NULL |     30 |
|  7788 | SCOTT | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 | NULL |     20 |
|  7839 | KING  | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 | NULL |     10 |
|  7902 | FORD  | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 | NULL |     20 |
|  7782 | CLARK | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 | NULL |     10 |
+-------+-------+-----------+------+---------------------+---------+------+--------+
6 rows in set (0.00 sec)

mysql> select * from emp where sal > 2500 union all select * from emp where job = 'MANAGER';
+-------+-------+-----------+------+---------------------+---------+------+--------+
| empno | ename | job       | mgr  | hiredate            | sal     | comm | deptno |
+-------+-------+-----------+------+---------------------+---------+------+--------+
|  7566 | JONES | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
|  7698 | BLAKE | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 | NULL |     30 |
|  7788 | SCOTT | ANALYST   | 7566 | 1987-04-19 00:00:00 | 3000.00 | NULL |     20 |
|  7839 | KING  | PRESIDENT | NULL | 1981-11-17 00:00:00 | 5000.00 | NULL |     10 |
|  7902 | FORD  | ANALYST   | 7566 | 1981-12-03 00:00:00 | 3000.00 | NULL |     20 |
|  7566 | JONES | MANAGER   | 7839 | 1981-04-02 00:00:00 | 2975.00 | NULL |     20 |
|  7698 | BLAKE | MANAGER   | 7839 | 1981-05-01 00:00:00 | 2850.00 | NULL |     30 |
|  7782 | CLARK | MANAGER   | 7839 | 1981-06-09 00:00:00 | 2450.00 | NULL |     10 |
+-------+-------+-----------+------+---------------------+---------+------+--------+
8 rows in set (0.00 sec)
```

