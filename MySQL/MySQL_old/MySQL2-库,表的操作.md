# 库的操作

## 创建数据库

```mysql
create database db1;
create database db2 charset=utf8;  // 创建一个使用utf8字符集的 db2 数据库
create database db3 charset=utf8 collate utf8_general_ci;
// 创建一个使用utf字符集，并带校对规则的 db3 数据库。
```

> 说明：当我们创建数据库没有指定字符集和校验规则时，系统使用默认字符集：utf8，校验规则 是：utf8_ general_ ci

## 字符集和校验规则

xxx

## 查看数据库\显示创建语句

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db_base            |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> show create database db_base;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| db_base  | CREATE DATABASE `db_base` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 修改数据库

对数据库的修改主要指的是修改数据库的字符集，校验规则

```mysql
alter database db_base charset=gbk collate gbk_chinese_ci;

mysql> show create database db_base;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| db_base  | CREATE DATABASE `db_base` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> alter database db_base charset=gbk collate gbk_chinese_ci;
Query OK, 1 row affected (0.00 sec)

mysql> show create database db_base;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| db_base  | CREATE DATABASE `db_base` /*!40100 DEFAULT CHARACTER SET gbk */ |
+----------+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```

## 删除数据库

```mysql
mysql> drop database test_db;
```

## 数据库的备份和恢复

xxx

## 查看连接情况

xxx

# 表的操作

## 创建表

语法

```mysql
CREATE TABLE table_name (
field1 datatype,
field2 datatype,
field3 datatype
) character set 字符集 collate 校验规则 engine 存储引擎;
```

> 说明:
>
> - field 表示列名 
> - datatype 表示列的类型 
> - character set 字符集，如果没有指定字符集，则以所在数据库的字符集为准 
> - collate 校验规则，如果没有指定校验规则，则以所在数据库的校验规则为准

```mysql
create table users (
    id int,
    name varchar(20) comment '用户名',
    password char(32) comment '密码是32位的md5值',
    birthday date comment '生日'
) character set utf8 engine MyISAM;
```

### 有关存储引擎

xxx

## 查看表的结构

description

```mysql
mysql> desc users;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | YES  |     | NULL    |       |
| name     | varchar(20) | YES  |     | NULL    |       |
| password | char(32)    | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

## 修改表

```mysql
alter table users add assets varchar(100) comment '图片路径' after birthday;
// alter table xxx add xxx int after yyy;
alter table users modify name varchar(60);  // 修改列类型
alter table users drop password;
alter table users rename [to] employee;
alter table employee change name xingming varchar(60); // 把employee列改为xingming varchar(60)，列名&列类型都改
```

## 删除表

```mysql
drop table users;
```