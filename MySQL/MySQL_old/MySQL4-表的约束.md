# 约束

真正约束字段的是数据类型，但是数据类型约束很单一，需要有一些**额外的约束**，更好的**保证数据的合法性**，从业务逻辑角度**保证数据的正确性。**

表的约束很多，这里主要介绍如下几个： `null/not null,default, comment, zerofill，primary key，auto_increment，unique key` 

## 1. 空属性not null

- 两个值：`null`（默认的）和`not null`（不为空）
- 数据库默认字段基本都是字段为空，但是实际开发时，尽可能保证字段不为空，因为数据为空不能参与运算。

```mysql
mysql> select 1+null;
+--------+
| 1+null |
+--------+
|   NULL |
+--------+
1 row in set (0.00 sec)

mysql> select null-null;
+-----------+
| null-null |
+-----------+
|      NULL |
+-----------+
1 row in set (0.00 sec)
```

案例：

创建一个班级表，包含班级名和班级所在的教室。站在正常的业务逻辑中： 如果班级没有名字，你不知道你在哪个班级；如果教室名字为空，就不知道在哪上课。

所以我们在设计数据库表的时候，一定要在表中进行限制，不满足上面条件的数据就不能插入到表中。这就是“约束”。

```mysql
mysql> create table class(
    -> class_name int not null,
    -> class_room int not null
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> desc class;
+------------+---------+------+-----+---------+-------+
| Field      | Type    | Null | Key | Default | Extra |
+------------+---------+------+-----+---------+-------+
| class_name | int(11) | NO   |     | NULL    |       |
| class_room | int(11) | NO   |     | NULL    |       |
+------------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

```mysql
mysql> create table test(
    -> age int,
    -> name int
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> desc test;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| age   | int(11) | YES  |     | NULL    |       |
| name  | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

对比之下，test table的两个列字段可以为Null：YES，而class的class_room class_name不能为null：NO

**这就是对字段的一种约束，简单来说就是：约束某一个字段不可以为Null。建表时，在字段属性中加上not null约束即可**

```mysql
mysql> insert into class(class_name) values('509');
ERROR 1364 (HY000): Field 'class_room' doesn't have a default value
```

## 2. 默认值default

默认值：某一种数据会经常性的出现某个具体的值，可以在一开始就指定好，在需要真实数据的时候， 用户可以选择性的使用默认值。

```mysql
mysql> create table tt (
    -> name varchar(20) not null,
    -> age tinyint unsigned default 0,
    -> sex char(2) default '男'
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> desc tt;
+-------+---------------------+------+-----+---------+-------+
| Field | Type                | Null | Key | Default | Extra |
+-------+---------------------+------+-----+---------+-------+
| name  | varchar(20)         | NO   |     | NULL    |       |
| age   | tinyint(3) unsigned | YES  |     | 0       |       |
| sex   | char(2)             | YES  |     | 男      |       |
+-------+---------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

其实很简单，比如我们现在在创建一个程序员信息表，此时sex就可以设置一个default值，也就是'男'。此时desc table时，该字段的Default列显示的就不是NULL了，而是设定好的default值。

**默认值的生效**：数据在插入的时候不给该字段赋值，就使用默认值

```mysql
mysql> insert into tt(name) values ('yzl');
Query OK, 1 row affected (0.00 sec)

mysql> select * from tt;
+------+------+------+
| name | age  | sex  |
+------+------+------+
| yzl  |    0 | 男   |
+------+------+------+
1 row in set (0.00 sec)
```

注意：只有设置了default的列，才可以在插入值的时候，对列进行省略。

若某列没有设定default值，则插入新数据时，不指定它，那么MySQL服务端也不知道这一列该插入什么数据呀。

> 突发没什么意义的奇想：可以设定为default NULL吗？也就是设定默认值为NULL，NULL也算默认值吗？插入的时候也就可以省略该字段了？
>
> 结果：是可以的....
>
> ```mysql
> mysql> alter table tt modify name varchar(20) default NULL;
> Query OK, 0 rows affected (0.05 sec)
> Records: 0  Duplicates: 0  Warnings: 0
> 
> mysql> desc tt;
> +-------+---------------------+------+-----+---------+-------+
> | Field | Type                | Null | Key | Default | Extra |
> +-------+---------------------+------+-----+---------+-------+
> | name  | varchar(20)         | YES  |     | NULL    |       |
> | age   | tinyint(3) unsigned | YES  |     | 0       |       |
> | sex   | char(2)             | YES  |     | 男      |       |
> +-------+---------------------+------+-----+---------+-------+
> 3 rows in set (0.00 sec)
> 
> mysql> insert into tt values();
> Query OK, 1 row affected (0.00 sec)
> 
> mysql> select * from tt;
> +------+------+------+
> | name | age  | sex  |
> +------+------+------+
> | yzl  |    0 | 男   |
> | NULL |    0 | 男   |
> +------+------+------+
> 2 rows in set (0.00 sec)
> ```

## 3. 列描述comment

列描述：comment，没有实际含义，专门用来描述字段，会根据表创建语句保存，用来给程序员或DBA来进行了解（该字段）。

```mysql
mysql> create table tt12 (
    -> name varchar(20) not null comment '姓名',
    -> age tinyint unsigned default 0 comment '年龄',
    -> sex char(2) default '男' comment '性别'
    -> );
Query OK, 0 rows affected (0.02 sec)

// 通过desc查看不到注释信息
mysql> desc tt12;
+-------+---------------------+------+-----+---------+-------+
| Field | Type                | Null | Key | Default | Extra |
+-------+---------------------+------+-----+---------+-------+
| name  | varchar(20)         | NO   |     | NULL    |       |
| age   | tinyint(3) unsigned | YES  |     | 0       |       |
| sex   | char(2)             | YES  |     | 男      |       |
+-------+---------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

// 通过show create table xxx可以看到comment信息
mysql> show create table tt12;

| tt12  | CREATE TABLE `tt12` (
  `name` varchar(20) NOT NULL COMMENT '姓名',
  `age` tinyint(3) unsigned DEFAULT '0' COMMENT '年龄',
  `sex` char(2) DEFAULT '男' COMMENT '性别'
) ENGINE=InnoDB DEFAULT CHARSET=gbk
```

可能会有疑惑，comment算什么约束，可以理解为它是一个软约束，也就是约束程序员在insert数据的时候，要按照comment描述的该字段来进行insert，比如年龄，你就不应该insert一个负数~

## 4. zerofill

刚开始学习数据库时，很多人对数字类型后面的长度很迷茫。通过show看看test表的建表语句：

```Mysql
mysql> show create table test;
+-------+-----------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                          |
+-------+-----------------------------------------------------------------------------------------------------------------------+
| test  | CREATE TABLE `test` (
  `age` int(11) DEFAULT NULL,
  `name` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gbk |
+-------+-----------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

可以看到int(11),这个代表什么意思呢？整型不是4字节码？这个10又代表什么呢？**其实没有zerofill这个 属性，括号内的数字是毫无意义的。**

```mysql
mysql> create table test3(
    -> age int(4) unsigned zerofill
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> insert into test3 values (1), (22);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from test3;
+------+
| age  |
+------+
| 0001 |
| 0022 |
+------+
2 rows in set (0.00 sec)
```

这次可以看到a的值由原来的1变成0001，这就是zerofill属性的作用，**如果宽度小于设定的宽度（这里设置的是4），自动填充0。**
要注意的是，这只是最后显示的结果，在MySQL中实际存储的还是1，0001只是设置了zerofill属性后的一种格式化输出而已。

## 5. 主键primary key

主键约束：primary key

用来唯一的约束该字段里面的数据，**不能重复，不能为空，一张表中最多只能有一个主键；主键所在的列通常是整数类型。**

```mysql
mysql> create table tt2( 
    -> id int unsigned primary key comment '学号，不能为NULL',
    -> name varchar(20) not null );
Query OK, 0 rows affected (0.02 sec)

mysql> desc tt2;
+-------+------------------+------+-----+---------+-------+
| Field | Type             | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+-------+
| id    | int(10) unsigned | NO   | PRI | NULL    |       |
| name  | varchar(20)      | NO   |     | NULL    |       |
+-------+------------------+------+-----+---------+-------+
2 rows in set (0.01 sec)
```

**主键约束：主键对应的字段中不能重复，一旦重复，操作失败。**

```mysql
mysql> insert into tt2 values (11, 'zzz');
Query OK, 1 row affected (0.01 sec)

mysql> insert into tt2 values (11, 'xxx');
ERROR 1062 (23000): Duplicate entry '11' for key 'PRIMARY'
```

当表创建好以后但是没有主键的时候，**可以再次追加主键**

```mysql
mysql> desc users;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| id       | int(11)      | YES  |     | NULL    |       |
| name     | varchar(20)  | YES  |     | NULL    |       |
| password | char(32)     | YES  |     | NULL    |       |
| birthday | date         | YES  |     | NULL    |       |
| assets   | varchar(100) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
5 rows in set (0.00 sec)

mysql> alter table users add primary key (id, password);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> desc users;
+----------+--------------+------+-----+---------+-------+
| Field    | Type         | Null | Key | Default | Extra |
+----------+--------------+------+-----+---------+-------+
| id       | int(11)      | NO   | PRI | NULL    |       |
| name     | varchar(20)  | YES  |     | NULL    |       |
| password | char(32)     | NO   | PRI | NULL    |       |
| birthday | date         | YES  |     | NULL    |       |
| assets   | varchar(100) | YES  |     | NULL    |       |
+----------+--------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

**删除主键**

```mysql
mysql> alter table users drop primary key;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

**复合主键**

在创建表的时候，在所有字段之后，使用primary key(主键字段列表)来创建主键，如果有多个字段作为主键，可以使用复合主键。

```mysql
mysql> create table tt3(
    -> id int,
    -> name varchar(20),
    -> primary key(id, name)
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> desc tt3;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   | PRI | NULL    |       |
| name  | varchar(20) | NO   | PRI | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.01 sec)
```

## 6. 自增长auto increment

**auto_increment约束**：当对应的字段不给值时，会自动的被系统触发，系统会从当前字段中已经有的最大值+1操作，得到一个新的不同的值。通常和主键搭配使用，作为逻辑主键。

自增长的特点: 

- 任何一个字段要做自增长，前提是本身是一个索引**（key一栏有值）**
- 自增长字段**必须是整数**
- 一张表**最多只能有一个自增长**

```mysql
mysql> create table tt21(
    -> id int unsigned primary key auto_increment,
    -> name varchar(10) not null default ''
    -> );
mysql> insert into tt21(name) values('a');
mysql> insert into tt21(name) values('b');
mysql> select * from tt21;
+----+------+
| id | name |
+----+------+
| 1  | a    |
| 2  | b    |
+----+------+
```

> 索引：
>
> 在关系数据库中，索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。 **索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。** 
>
> 索引提供指向存储在表的指定列中的数据值的指针，然后根据您指定的排序顺序对这些指针排序。 数据库使用索引以找到特定值，然后顺指针找到包含该值的行。这样可以使对应于表的SQL语句执行得 更快，可快速访问数据库表中的特定信息。

## 7. 唯一键unique

**一张表中有往往有很多字段需要唯一性**，数据不能重复，但是一张表中只能有一个主键

唯一键就可以解决表中有多个字段需要**唯一性约束**的问题。

唯一键的本质和主键差不多，唯一键允许为空，而且**可以多个为空，空字段不做唯一性比较。**

关于唯一键和主键的区别：我们可以简单理解成，主键更多的是标识唯一性的。而**唯一键更多的是保证在业务上，不要和别的信息出现重复。**

```mysql
mysql> create table student (
    -> id char(10) unique comment '学号，不能重复，但可以为空',
    -> name varchar(10)
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> insert into student(id, name) values('01', 'aaa');
Query OK, 1 row affected (0.00 sec)

mysql> insert into student(id, name) values('01', 'bbb'); --唯一约束不能重复
ERROR 1062 (23000): Duplicate entry '01' for key 'id'

mysql> insert into student(id, name) values(null, 'bbb'); -- 但可以为空
Query OK, 1 row affected (0.00 sec)

mysql> select * from student;
+------+------+
| id   | name |
+------+------+
| 01   | aaa  |
| NULL | bbb  |
+------+------+
```

## 8. 外键foreign key (x) references t(y)

**外键用于定义主表和从表之间的关系**：外键约束主要定义在从表上，主表则必须是有**主键约束或unique约束**。当定义外键后，要求**外键列数据必须在主表的对应列存在或为null。**

语法：`foreign key (字段名) references 主表(列)`

![image-20231121204820045](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231121204820045.png)

作用/意义 : 如果将班级表中的数据都设计在每个学生表的后面，那就会出现**数据冗余**，所以我们只要设计成让stu->class_id和myclass->id形成关联的关系=>**外键约束**

```mysql
// 创建主表
mysql> create table myclass(
    -> id int primary key,
    -> name varchar(20) not null unique comment'班级名'
    -> );
Query OK, 0 rows affected (0.02 sec)
// 创建从表
mysql> create table stu (
    -> id int primary key,
    -> name varchar(10) not null comment '学生名',
    -> class_id int unique,
    -> foreign key (class_id) references myclass(id)     // 外键约束
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> insert into myclass values(509, '花班'), (501, '朵班');
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> insert into stu values(329, 'yzl', 509);
Query OK, 1 row affected (0.01 sec)

mysql> insert into stu values(330, 'zzz', 501);
Query OK, 1 row affected (0.01 sec)

mysql> insert into stu values(333, 'hhh', 555);    // 555班级不存在
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`db_base`.`stu`, CONSTRAINT `stu_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `myclass` (`id`))
mysql> insert into stu values(333, 'hhh', NULL);   // 新生暂时没有班级，是可以的，外键可以为NULL
Query OK, 1 row affected (0.00 sec)
```

> 如何理解外键：
>
> 首先我们承认，这个世界是数据很多都是相关性的。 理论上，上面的例子，我们不创建外键约束，就正常建立学生表，以及班级表，该有的字段我们都有。 此时，在实际使用的时候，可能会出现什么问题？ 有没有可能插入的学生信息中有具体的班级，但是该班级却没有在班级表中？
>
> 因为此时**两张表在业务上是有相关性的**，但是在业务上**没有建立约束关系**，**那么就可能出现问题。**
>
> 解决方案就是通过外键完成的。建立外键的本质其实就是把相关性交给mysql去审核了，提前告诉mysql表之间的约束关系，**那么当用户插入不符合业务逻辑的数据的时候，mysql不允许你插入。**
