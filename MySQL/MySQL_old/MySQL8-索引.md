# 索引

## 索引定义

MySQL官方对索引的定义为 : **索引（Index）是帮助MySQL高效获取数据的数据结构**。索引的本质：**索引是数据结构。各类索引有各自的数据结构实现。**

> 索引并非一定是B+结构....

## InnoDB为什么选择B+树作为索引结构

InnoDB 在建立索引结构来管理数据的时候，为什么选择B+树结构？

- 链表？线性遍历，根本不行

- 二叉搜索树？退化问题，可能退化成为线性结构

- AVL || 红黑树？虽然是平衡或者近似平衡，但是毕竟是二叉结构，相比较多阶B+，意味着**树整体更高（瘦高型树）**，大家都是自顶向下找，层高越低，经过的Page越少，意味着系统与硬盘更少的IO Page交互。

- Hash？官方的索引实现方式中， MySQL 是支持HASH的，不过 InnoDB 和 MyISAM 并不支持.
  Hash算法特征，决定了**虽然有时候按照key进行检索时很快:O(1)，不过，在面对范围查找就明显不行**，另外还有其他差别...
  ![image-20231122185144589](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122185144589.png)

- B树？最值得比较的是 InnoDB 为何不用B树作为底层索引结构？

  B+树

  ![](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/v2-d1648a55f3cdfa616b7b2bd9f28835c5_r.jpg)
  B树

  ![img](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/667853-20201209215015379-24958929.png)

  - B树的非叶子Page内部会存储data，而B+树只有叶子Page会存储data，非叶子节点全部都是目录Page。（B树的所有节点既有数据，又有Page指针，而B+，只有叶子节点有数据，其他目录页，只有键值和Page指针）
  - B+叶子节点，全部相连，而B没有。

  为何选择B+？

  - B+的目录Page不存储data，这样一个目录Page就可以存储更多的目录项 : 键值+Page指针，**可以使得树更矮**，找到目的记录的**IO操作次数更少，效率更高。**
  - **叶子节点相连，更便于进行范围查找。**（叶子结点相连就决定了B+结构在范围遍历时更有优势）

> **B+Tree是在B-Tree基础上的一种优化。目录Page只存储键信息，这样不存数据可以腾出空间放更多的键信息，让树层数越小**

## 聚簇索引 VS 非聚簇索引(辅助索引)

### MyISAM存储引擎 - 非聚簇索引

MyISAM 引擎与InnoDB一样使用B+树作为索引结构，**不同的是MyISAM的B+树的叶子Page的data域存放的是数据记录的地址，而不是数据记录。**

下图为 MyISAM 表的主键索引， Col1为主键。

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/3829bea81468dba3da62f831b6614b7a.png" alt="image.png" style="zoom:80%;" />

当然， MySQL 除了可以建立**主键索引**外，也有可能按照用户指定的其他列字段信息建立索引，一般这种索引可以叫做**辅助（普通）索引。**(主键不能为NULL, 不能重复, 辅助索引可以为NULL, 可以重复)

对于 MyISAM ，建立辅助（普通）索引和主键索引没有差别, 下图就是基于 MyISAM 的 Col2 建立的索引，和主键索引没有差别, 因为这是非聚簇索引, 它们的叶子Page存的是数据记录的地址, 而不是数据记录

![img](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/u%3D4097609815%2C2313979017%26fm%3D253%26app%3D138%26f%3DJPEG)

其中， MyISAM 最大的特点是，将**索引Page和数据Page分离**. 也就是索引的叶子Page不存数据，只有对应数据记录的地址。

> 数据记录还是要存在Page内部的, 都是Page, 都是16KB空间，只是数据Page内部存储的是数据库中表的数据记录。

### InnoDB存储引擎 - 聚簇索引

**相较于MyISAM的非聚簇索引， InnoDB 是将索引结构和数据记录放在一起的。**

> 也就是叶子Page的内部存储有数据记录（当然还有目录，便于检索叶子Page内部的数据记录）

![image.png](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/ae4303dd27435f95af8c9c11f774276c.png)

同样， InnoDB 除了主键索引，用户也会建立辅助（普通）索引，表中Col3 建立对应的辅助索引如下图：

![img](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/667853-20201209220717856-957975411.png)

**可以看到， InnoDB 的辅助索引中叶子节点并没有数据记录，而只有对应记录的key值。所以通过辅助（普通）索引，找到目标记录，需要两遍索引：1. 首先检索辅助索引获得主键值，然后用主键值到主键索引中检索获得数据记录。这个过程，叫做<u>回表查询</u>**

为何 InnoDB 针对这种辅助（普通）索引的场景，不采用主键索引的结构, 给叶子Page也附上数据记录呢？很简单：因为太浪费空间了, 相较于回表查询的性能消耗，将数据记录在辅助索引的叶子Page中也存储一份是非常不值的.

**其中， MyISAM 这种用户数据记录与索引数据分离的索引方案，叫做非聚簇索引**, 即数据记录Page和B+结构的索引Page是分开存储的

**其中， InnoDB 这种用户数据记录与索引数据放在一个B+结构的索引方案，叫做聚簇索引**, 叶子Page内部存储数据记录

```mysql
mysql> create table myisam(
    -> id int primary key,
    -> name varchar(20)
    -> )engine=MyISAM;      --使用engine=MyISAM
Query OK, 0 rows affected (0.00 sec)

mysql> create table innodb(
    -> id int primary key,
    -> name varchar(20)
    -> )engine=InnoDB;      --使用engine=InnoDB
Query OK, 0 rows affected (0.02 sec)


[root@hecs-296309 db_index]# ll
total 248
-rw-r----- 1 mysql mysql    61 Aug 25 18:42 db.opt
-rw-r----- 1 mysql mysql  8586 Aug 25 21:08 innodb.frm  --表结构数据
-rw-r----- 1 mysql mysql 98304 Aug 25 21:08 innodb.ibd	--该表对应的主键索引
-rw-r----- 1 mysql mysql  8586 Aug 25 21:08 myisam.frm	--表结构数据
-rw-r----- 1 mysql mysql     0 Aug 25 21:08 myisam.MYD	--该表对应的记录数据，当前没有数据，所以是0
-rw-r----- 1 mysql mysql  1024 Aug 25 21:08 myisam.MYI	--该表对应的主键索引
-rw-r----- 1 mysql mysql  8558 Aug 25 20:20 test.frm
-rw-r----- 1 mysql mysql  8614 Aug 25 19:31 user.frm
-rw-r----- 1 mysql mysql 98304 Aug 25 19:33 user.ibd
```

这也就是为什么innodb和myisam作为表的存储引擎时，对应创建出的文件数量不同：就是因为一个是聚簇索引，一个是非聚簇索引。

## 复合索引（联合索引）

按照字段的个数来对索引进行分类时，可以将索引分为两类

1.**单列索引**：一个索引只包含了一个列，一个表里面可以有多个单列索引。
其实就是，按照某一个字段构建索引，比如主键索引，唯一键索引，普通索引，都属于单列索引。即按照某字段值构建索引，提高检索效率。比如按照id构建单列索引，在`select * from table where id=300;`时就会提高搜索效率。

2.**复合索引(联合索引&组合索引)（Compound Index）：**包含多个列的索引。

```mysql
mysql> create table staffs( id int, name varchar(10), age int);
Query OK, 0 rows affected (0.02 sec)
mysql> create index id_name_age_index on staffs(id,name,age);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table staffs\G;
*************************** 1. row ***************************
       Table: staffs
Create Table: CREATE TABLE `staffs` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(10) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  KEY `id_name_age_index` (`id`,`name`,`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```

如上，对表staffs中的三个列创建了一个复合索引。我们知道索引必须要有键值，并且索引会按照键值对记录进行排序。InnoDB下会根据键值构建B+索引，**而复合索引就会先根据第一个（最左侧）字段对记录进行排序，当第一个字段有序时，再按照第二个字段对记录排序，其次再根据第三个字段对记录进行排序。**

所以，构建复合索引（id，name，age）**相当于**构建了以下的三个索引：（效果等同于构建三个索引的效果，而非真的构建了三个索引，实际上还是一个索引结构）

1. （id）

2. （id，name）
3. （id，name，age）

> 举例来说，此复合索引可以优化以下的查询条件：
>
> select * from staffs where id=1;
> select * from staffs where id=1 and name = 'xxxx';
> select * from staffs where id=1 and name = 'xxxx' and age=1;
> select * from staffs where id=1 and name = 'xxxx' and age>10;
>
> 上方三条select都可以利用根据（id，name，age）构建的复合索引来优化查询，因为这个索引是根据id进行排序的，而id有序时name也是有序的，id，name有序时，age也是有序的。因此可以很快检索到目的记录。
>
> select * from staffs where id=1 and age=1;
>
> 此select也可以根据复合索引进行优化，因为索引中的id是有序的。但age相对于id来说是无序的，只有id是有序的，所以他只能使用联合索引中的id索引。
>
> select * from staffs where name = 'xxxx';
> select * from staffs where age=1;
> select * from staffs where name = 'xxxx' and age=1;
>
> 上方三条语句，因为不符合最左匹配原则，所以没有用到联合索引。
>
> select * from staffs where name = 'xxxx' and age=1 and id=3;
> select * from staffs where name = 'xxxx' and id=1 and age=1;
> select * from staffs where id=1 and age=1 and name = 'xxxx';
>
> 注意，上方三条语句，虽然并不是严格按照最左匹配原则进行检索，但是因为MySQL中有**查询优化器explain**，所以sql语句中字段的顺序不需要和联合索引定义的字段顺序相同，查询优化器会判断并纠正这条SQL语句以什么样的顺序执行效率高，最后生成真正的执行计划，所以不论以何种顺序都可使用到联合索引。
>
> **因此，构建复合索引时，尽量将唯一性较强的，比较频繁作为搜索条件的字段放在左边。**

**复合索引的好处是什么？**

- **高效**：比如我们在select id =1 and name = 'xxx'时，或者按照多个字段进行select时，如果没有复合索引，则会涉及多个，多次辅助索引的查找，再加上回表，性能相对于复合索引来说是更低的。
- **覆盖索引**：如果一个复合索引涵盖了查询中所需的所有列，那么查询可以直接通过复合索引进行，而不需要访问主键索引。这种情况称为"覆盖索引"，它可以显著提高查询性能，因为避免了额外的数据行访问操作。（减少了IO！）
- **排序和分组操作**：如果复合索引的列顺序与常见的排序或分组操作相匹配，那么数据库引擎可以利用这种索引的有序性，加速排序和分组操作的执行。

## 索引的最左匹配原则

最左前缀原则，又称最左匹配原则，是指在**使用联合索引（复合索引）进行查询时，where的查询条件需要遵循索引中列的顺序，从左到右进行匹配。只有当查询条件满足最左前缀原则时，才能充分利用联合索引的优势，提高查询性能。**

1. 在查询时，如果不满足最左列的查询条件，索引将无法被充分利用。例如，对于一个(a, b, c)的联合索引，如果查询条件没有涉及到列a，则索引的优势无法被充分发挥。

2. 查询条件可以部分使用联合索引。例如，在(a, b, c)的联合索引中，如果查询条件仅涉及到列a，则只有列a部分的索引会被利用。如果查询条件涉及到列a和b，那么列a和b部分的索引都会被利用。

3. 使用最左前缀原则时，可以使用EXPLAIN命令来分析查询计划，以确保索引被正确使用。

为什么要遵循最左匹配原则来进行select呢？其实就是因为复合索引结构是按照从左到右的字段顺序对记录进行排序的。

## 索引覆盖

只需要在一个辅助索引上就能获取SQL所需的所有列数据，无需回表，速度更快。

覆盖索引即从辅助索引中就可以得到查询需要的数据，而不需要根据辅助索引中获取的主键值来回表查询主键索引中的记录。
使用覆盖索引的一个好处就是由于辅助索引中不包含整行的所有记录，所以它的大小要远远小于聚簇索引，**因此可以减少大量的I/O操作。**

比如：由于辅助索引中叶子节点存放的数据就是主键，所以当我们要查找主键，或者通过主键来统计数量的时候，就可以使用覆盖索引来完成。

再比如：我们根据（id，name）构建了复合索引，当`select name from table where id=3`时，也可以从复合索引中获取SQL所需的所有列数据（name），而不需要回表（回主键索引），也可以提高效率。不过需要注意，其实索引覆盖并不是仅限于复合索引的场景。只要我们所需的数据在辅助索引中能够获取到，那么它就覆盖了主键索引，即，索引覆盖。

## 索引优缺点与适用场景

**优点**

- 大大**加快了数据检索的速度**
- **所有的列类型都可以被索引**，也就是可以给任意字段设置索引

**缺点**

- **索引需要占用物理空间**，建立的索引越多则需要的空间越大(早就说过了，索引≈目录，都是空间换时间的思想~)
- **创建和维护索引需要耗费时间**，并且时间随着数据量的增加而增加
- 当对表中的数据进行增加、删除、修改的时候，索引也要动态的维护，降低了数据的维护速度（与第二点重复）

总结索引：消耗一定的空间+时间，大幅提高数据检索速度。（数据量大的时候是很有必要的！）

## 索引创建原则

- **数据量较大，并且比较频繁作为查询条件**的字段应该创建索引（select * from x where zzz='yyy'频繁，则zzz适合创建索引）
- **唯一性太差的字段不适合**单独创建索引，即使频繁作为查询条件。
- **更新非常频繁的字段不适合**作创建索引（B+的修改效率低，查询效率高）
- 不会出现在where子句中的字段不该创建索引（与第一点重复。索引的作用本身就是为了提高检索效率啊，他妈的某字段都不检索，还创建索引干嘛？）

# 索引的理解

> 包含mysql，os，还有磁盘之间的关系（因为要搞清楚mysql的数据的存储方式：Page）；
> 没有索引有什么问题？提出没有索引的问题 -> 问题驱动式地逐步解决问题，最终的解决方案其实就是InnoDB的索引结构。

## MySQL，OS，磁盘

**MySQL与存储**

MySQL 给用户提供存储服务，而存储的都是数据，数据需要存储在磁盘这个外设当中。磁盘是计算机中的一个机械设备，相比于计算机其他电子元件，磁盘效率是比较低的，在加上IO本身的特征，可以知道，如何提高效率，是 MySQL 的一个重要话题。

**认识磁盘**

磁头，音圈马达... 略了，主要是磁盘是一摞盘片组成的。而一个盘片是由很多**扇区**组成的，未来的文件等数据都是存储在一个个扇区中的，文件大就多占几个扇区，文件小就少占几个扇区。

![image-20231122155505804](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122155505804.png)

在Linux操作系统中，大部分的目录或者文件都是存储在硬盘（磁盘的一种）中的（除了一些内存级文件，不考虑），也就是说，OS要想进行数据的持久化，就必须将数据保存在磁盘中，也就是磁盘的一个个扇区中。MySQL作为一款数据库软件，是客户端服务端的网络模式的，mysqld也就是数据库的服务端, 本质也是一个在OS内运行的进程。

如上图

1. **数据库中的每一个database都是一个目录**
2. **database中的每一个table都是若干个文件**（每个table对应文件的数量取决于table的存储引擎engine是InnoDB还是MyISAM，InnoDB对应两个文件，MyISAM对应三个文件）

```mysql
mysql> show create table users\G;
*************************** 1. row ***************************
       Table: users
Create Table: CREATE TABLE `users` (
  `id` int(11) NOT NULL,
  `name` varchar(20) DEFAULT NULL COMMENT '用户名',
  `password` char(32) NOT NULL COMMENT '密码是32位的md5值',
  `birthday` date DEFAULT NULL COMMENT '生日',
  `assets` varchar(100) DEFAULT NULL COMMENT '图片路径'
) ENGINE=MyISAM DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

ERROR: 
No query specified

mysql> show create table tt\G;
*************************** 1. row ***************************
       Table: tt
Create Table: CREATE TABLE `tt` (
  `name` varchar(20) DEFAULT NULL,
  `age` tinyint(3) unsigned DEFAULT '0',
  `sex` char(2) DEFAULT '男'
) ENGINE=InnoDB DEFAULT CHARSET=gbk
1 row in set (0.00 sec)

ERROR: 
No query specified

-rw-r----- 1 mysql mysql   8616 Aug 22 19:43 tt.frm
-rw-r----- 1 mysql mysql  98304 Aug 22 19:43 tt.ibd
-rw-r----- 1 mysql mysql   8746 Aug 22 20:29 users.frm
-rw-r----- 1 mysql mysql     44 Aug 22 20:29 users.MYD
-rw-r----- 1 mysql mysql   1024 Aug 22 20:29 users.MYI
```

因此，得出结论，**MySQL的table和database本质都是文件/目录，而要想持久化就需要持久化到磁盘中。**

所以，**找到一个文件**的全部，本质，就是在磁盘找到所有保存文件的扇区。而我们能够**定位任何一个扇区**，就能找到所有扇区，因为查找方式是一样的。磁盘如何定位扇区：略

---

操作系统与磁盘IO的基本单位不是扇区的512字节, **操作系统与磁盘IO，是以块为单位的，基本单位是`4KB `。**

**MySQL与磁盘IO交互的基本单位**

而 MySQL 作为一款应用软件，可以想象成一种特殊的文件系统。它有着更高的IO场景与IO需求
为了提高IO效率， MySQL 与磁盘IO的基本单位是`16KB`（不同于OS与磁盘之间的4KB）

> **磁盘随机访问(Random Access)与连续访问(Sequential Access)**
>
> **随机访问**：本次IO所给出的扇区地址和上次IO给出扇区地址不连续，这样的话磁头在两次IO操作之间需要做比较大的移动动作才能定位新的扇区, 重新开始读/写数据。**（耗时，效率低）**
>
> **连续访问**：如果当次IO给出的扇区地址与上次IO结束的扇区地址是连续的，那磁头就能很快的定位目标扇区并开始这次IO操作，这样的多个IO操作称为连续访问。**（效率高）**
>
> > 磁盘是通过机械运动进行寻址的，连续访问不需要过多的定位，故效率相对更高。

```mysql
mysql> SHOW GLOBAL STATUS LIKE 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_page_size | 16384 |   -- 16384 = 16 * 1024
+------------------+-------+
1 row in set (0.01 sec)
```

不过也要注意，MySQL作为一款应用层软件，要想和磁盘这种硬件外设进行IO，一定还是要通过OS来完成的。（OS向下管理好硬件，向上对软件提供一个良好的运行环境。）

也就是说，磁盘这个硬件设备的基本单位是 512 字节(扇区)，而 MySQL`InnoDB引擎` 使用 16KB 进行IO交互(页 Page)。 这个基本数据单元，在 MySQL 这里叫做**page**（注意和系统的page区分）。**也就是说，MySQL在存取数据时都是按照Page16KB为基本单位进行的。**

- MySQL 中的数据文件，是以page为单位（16KB）保存在磁盘当中的。
- MySQL 的 CURD 操作，都需要通过计算，找到对应的插入位置，或者找到对应要修改或者查询的数据。而只要涉及计算，就需要CPU参与，而为了便于CPU参与，一定要能够先将数据移动（加载load）到内存当中。(因为内存访问速度比磁盘快得多，通过在内存中操作数据可以显著提高数据库的性能。
- 所以在特定时间内，数据一定是磁盘中有，内存中也有。后续操作完内存数据之后，以特定的刷新策略，刷新到磁盘。而这时，就涉及到磁盘和内存的数据交互，也就是IO了。而此时IO的基本单位 就是Page。
- 为了更好的进行上面的操作， **MySQL 服务器在内存中运行的时候**，在服务器内部，就**申请了被称为 `Buffer Pool` 的的大内存空间**，来进行各种缓存。其实就是一段很大的内存空间，来和磁盘数据进行IO交互。
- 为了更高的效率，**一定要尽可能的减少系统和磁盘IO的次数。**

## 为何IO交互要是16KB的Page

为何MySQL和磁盘进行IO交互的时候，要采用Page的方案进行交互呢？用多少，加载多少不香吗? 

如上面的5条记录，如果MySQL要查找id=2的记录，第一次加载id=1，第二次加载id=2，一次一条记录，那 么就需要2次IO。如果要找id=5，那么就需要5次IO。 
但，如果这5条(或者更多)都被保存在一个Page中(16KB，能保存很多记录)（mysql对于表中的数据是以page为单位存储在磁盘中的），那么第一次IO查找id=2的时候，整个Page会被加载到MySQL的Buffer Pool中，这里完成了一次IO。但是往后如果在查找id=1,3,4,5 等，完全不需要进行IO了，而是直接在内存中进行了。所以**大大减少了IO的次数。**

你怎么保证，用户一定下次找的数据，就在这个Page里面？我们不能严格保证，但是有很大概率，因为有**局部性原理。**

往往IO效率低下的最主要矛盾不是单次IO数据量的大小，而是IO的次数。利用局部性原理，一次IO加载的多一些，就可以减少IO次数，提高IO效率。

> 正如上方最后一点所说的~

## 没有索引，可能会有什么问题

当构建出一个含有八百万条记录的没有索引的表之后，在按照某字段进行select时, `select * from EMP where empno=998877;`会耗时严重，慢的离谱。（四五秒，五六秒）

而我们`alter table EMP add index(empno);`为empno字段构建索引之后，再次执行`select * from EMP where empno=123456;`此时快到发昏。

所以，如果我们构建的database中的table的数据量或记录很多时，**在没有索引时会检索很慢，而索引就会显著提高select检索速度。**

## 现象与问题

```mysql
mysql> create table if not exists user ( 
mysql>    id int primary key, 
mysql>    age int not null,
mysql>    name varchar(16) not null 
mysql> );
Query OK, 0 rows affected (0.01 sec)

mysql> show create table user\G;
*************************** 1. row ***************************
       Table: user
Create Table: CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `age` int(11) NOT NULL,
  `name` varchar(16) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> insert into user (id, age, name) values(4, 16, '小龙女');
Query OK, 1 row affected (0.00 sec)

mysql>  insert into user (id, age, name) values(2, 26, '黄蓉');
Query OK, 1 row affected (0.00 sec)

mysql>  insert into user (id, age, name) values(5, 36, '郭靖');
Query OK, 1 row affected (0.00 sec)

mysql> insert into user (id, age, name) values(1, 56, '欧阳锋');
Query OK, 1 row affected (0.00 sec)

mysql> select * from user;
+----+-----+-----------+
| id | age | name      |
+----+-----+-----------+
|  1 |  56 | 欧阳锋    |
|  2 |  26 | 黄蓉      |
|  3 |  18 | 杨过      |
|  4 |  16 | 小龙女    |
|  5 |  36 | 郭靖      |
+----+-----+-----------+
5 rows in set (0.00 sec)
```

现象：一个table含有primary key，我们进行乱序insert，最后select时，竟然是有序的。也就是MySQL自动对这些记录按照主键进行了排序，为什么要这样做呢？

## 理解单个Page

MySQL 中要管理很多表文件，我们目前可以简单理解成每个独立文件是由若干个Page构成的。而要管理好这些文件，就需要管理好一个个Page，即对Page进行先描述，再组织。

![image-20231122165824661](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122165824661.png)

Page，先描述，再组织，先描述：用结构体或类：内部包含两个类似于双向链表的指针，还有要存储的数据记录。（不同的Page ，在 MySQL 中，都是16KB ）再组织：使用 prev 和 next 构成双向链表结构，将大量的Page组织起来。

因为有主键， **MySQL会按照主键给我们的数据记录进行排序**，从上面Page内的数据记录可以看出，数据记录是有序且彼此关联的。

## 理解多个Page

上面页模式中，只有一个功能，就是在查询某条数据记录的时候直接将一整页的数据加载到内存中，以减少硬盘IO次数，从而提高性能。

- 但是，现在的单个Page内部，实际上是采用了链表的结构，前一条记录指向后一条记录，单个Page内查找记录时本质还是通过记录的逐条比较来取出特定的数据。
- 如果有1千万条数据记录，一定需要多个Page来保存1千万条数据记录，而多个Page彼此之间又是使用双链表链接起来的，而且每个Page内部的数据记录也是基于有序链表的。**那么，查找特定一条记录，也一定是线性查找。效率就会非常低。**
  （这就是为什么最初8000000条记录在进行select时，耗时严重：因为没有主键，select需要线性遍历查找）

那么，现在有什么办法进行优化吗？（所以下面的内容本质就是逐步引出并介绍索引，从而解决select效率低的问题）

### 页目录

> 我们在看《谭浩强C程序设计》这本书的时候，如果我们要看<指针章节>，找到该章节有两种做法：1. 从头逐页的向后翻，直到找到目标内容。2. 通过书提供的目录，发现指针章节在234页(假设)，那么我们便直接翻到234页。同时，查找目录的方案，可以顺序找，不过因为目录肯定少，所以可以快速提高定位。
>
> 本质上，书中的目录，是多花了纸张的，但是却提高了效率。所以，**目录，是一种“空间换时间的做法”**

### 单页引入目录

因为单页page内部的若干记录是按照某key值顺序存储的，所以现在仅考虑单page进行检索的情况下，是需要O(N)时间复杂度的，效率低，**因此针对上面的单页Page，我们可以引入目录**

![image-20231122170550089](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122170550089.png)

那么当前，在一个Page内部，我们引入了目录。比如，我们要查找id=4记录，之前必须线性遍历4次， 才能拿到结果。现在直接通过目录2[3]，直接进行定位新的起始位置，提高了效率。

现在我们可以正式回答上面的问题了，**为何 MySQL 会根据键值将记录自动排序？为了配合目录，提高单Page的检索效率！**

### 多页情况

![image-20231122170721582](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122170721582.png)

多个Page的效率问题：**在Page之间，也是需要遍历的**，遍历意味着依旧需要进行大量的IO：将下一个Page加载到内存，进行检测。这样就显得我们之前的Page内部的目录，有点杯水车薪了。
（比如，在8000000条记录中，select778899号记录时，虽然单Page内部可以高效检索，但是一个Page最多只能存储16KB记录，而**多个Page之间又需要遍历检索**，所以如果迟迟没有找到目标记录，MySQL就需要Load很多磁盘中的Page到内存中来进行检索，这就意味着大量的IO，再加上多个Page之间又需要遍历检索（相比于大量IO来说可能并不算什么，所以效率低的主要原因还是需要大量IO将Page加载到内存中）

那么如何解决呢？解决方案：**给多个Page也带上目录。**

- 开辟目录Page, 使用目录Page的目录项来指向某存储数据记录的Page，而这个目录项存放的就是指向的Page中存放的最小数据记录的键值和指向Page的地址(指针)。
- 和下方存储记录数据的Page内的目录不同的地方在于，这种目录项管理的是Page，而Page内的目录管理的是数据记录。
- 其中，**每个目录项的构成是：键值+指针。**（图中没有画全）

![image-20231122171036827](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231122171036827.png)

而当下方存储数据记录的Page越来越多，就需要越来越多的目录Page，而在select时就需要遍历上方的目录Page来找到目标Page，而当目录Page越来越多时，又应该怎么解决呢？答：再在目录Page的上层添加目录Page。（如下图）

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230825200708561.png" alt="image-20230825200708561" style="zoom:80%;" />

所以，像这样层层向上添加目录Page，最终这种存储的数据结构就是**B+树**

至此，我们已经给我们的表user构建完了**主键索引**。主键索引最终在内存中存储时就是如上图所示的结构。

此时再随便select一个id=x, 我们发现，现在查找的Page数一定减少了，也就意味着IO次数减少了，**那么效率也就提高了！**

>总结：Page分为目录页和数据页。目录页的目录项存储各个下级Page的最小键值及Page地址。
>
>查找的时候，自顶向下找，只需要加载部分页（包含目录Page和数据Page）到内存，即可完成算法的整个查找过程，大大减少了IO次数，大大提高了效率！

----

# 索引操作

## 创建索引

**主键索引 primary key**

```mysql
-- 1
mysql> create table test2(
    -> id int primary key,
    -> name varchar(20) not null
    -> );
Query OK, 0 rows affected (0.01 sec)
-- 2
mysql> create table test3(
    -> id int,
    -> name varchar(20),
    -> primary key(id)
    -> );
Query OK, 0 rows affected (0.02 sec)
-- 3. 创建表以后再添加主键
mysql> alter table test add primary key(age);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

主键索引的特点

- 一个表中，最多有一个主键索引，当然也可以是复合主键
- 因为主键不可为NULL, 不能重复, 故主键索引的效率相对其他索引来说更高
- 主键索引的列基本上是int类型

**唯一键索引 unique**

```mysql
-- 1
mysql> create table test4(
    -> id int unique,
    -> name varchar(20)
    -> );
Query OK, 0 rows affected (0.01 sec)
-- 2
mysql> create table test5(
    -> id int,
    -> name varchar(20),
    -> unique(id)
    -> );
Query OK, 0 rows affected (0.02 sec)
-- 3
mysql> create table test6(
    -> id int);
Query OK, 0 rows affected (0.01 sec)

mysql> alter table test6 add unique(id);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

唯一索引的特点：

- 一个表中，可以有多个唯一索引
- 唯一键不可以重复, 可以为NULL, 查询效率较普通索引更高
- 如果一个唯一索引上指定not null，等价于主键索引。

**普通索引/辅助索引 index**

```mysql
-- 1
mysql> create table test7(
    -> id int,
    -> name varchar(20),
    -> index(id)
    -> );
Query OK, 0 rows affected (0.01 sec)
-- 2
mysql> create table test8(id int, name varchar(20));
Query OK, 0 rows affected (0.10 sec)

mysql> alter table test8 add index(id);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
-- 3
mysql> create table test9(
    -> id int primary key,
    -> name varchar(20),
    -> email varchar(30)
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> create index nor_index on test9(email);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

普通索引的特点：

- 一个表中可以有多个普通索引，普通索引在实际开发中用的比较多
- 如果某列需要创建索引，但是该列有重复的值，那么我们就应该使用普通索引

### **全文索引**

当对文章字段或有大量文字的字段进行检索时，会使用到全文索引。

```````mysql
mysql> create table articles(
    -> id int primary key auto_increment,
    -> title varchar(20),
    -> body text,
    -> fulltext(title, body)
    -> )engine=MyISAM;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO articles (title,body) VALUES
    -> ('MySQL Tutorial','DBMS stands for DataBase ...'),
    -> ('How To Use MySQL Well','After you went through a ...'),
    -> ('Optimizing MySQL','In this tutorial we will show ...'),
    -> ('1001 MySQL Tricks','1. Never run mysqld as root. 2. ...'),
    -> ('MySQL vs. YourSQL','In the following database comparison ...'),
    -> ('MySQL Security','When configured properly, MySQL ...');
Query OK, 6 rows affected, 1 warning (0.00 sec)
Records: 6  Duplicates: 0  Warnings: 1

mysql> select * from articles where body like '%database%';
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)

mysql> explain select * from articles where body like '%database%'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: articles
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL   <== key为null表示没有用到索引
      key_len: NULL
          ref: NULL
         rows: 6
     filtered: 16.67
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
-- 使用全文索引
mysql> select * from articles
    -> where match (title, body) against ('database');
+----+-------------------+------------------------------------------+
| id | title             | body                                     |
+----+-------------------+------------------------------------------+
|  5 | MySQL vs. YourSQL | In the following database comparison ... |
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)

mysql> explain select * from articles where match (title, body) against ('database')\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: articles
   partitions: NULL
         type: fulltext
possible_keys: title
          key: title  <= key用到了title
      key_len: 0
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```````

```mysql
CREATE FULLTEXT INDEX 索引名 ON 表名(字段名);
where MATCH(要匹配的列) AGAINST(要查找的内容) 
```

## 查询索引

```mysql
-- 1
mysql> show keys from test9;
+-------+------------+-----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name  | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed 
| test9 |          0 | PRIMARY   |            1 | id          | A         |           0 |     NULL | NULL   | test9 |          1 | nor_index |            1 | email       | A         |           0 |     NULL | NULL   

| Null | Index_type | Comment | Index_comment |
|      | BTREE      |         |               |
| YES  | BTREE      |         |               |
+-------+------------+-----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)
-- 2
mysql> show index from test9\G;
*************************** 1. row ***************************
        Table: test9
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 2. row ***************************
        Table: test9
   Non_unique: 1
     Key_name: nor_index
 Seq_in_index: 1
  Column_name: email
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment: 
Index_comment: 
2 rows in set (0.00 sec)

ERROR: 
No query specified
-- 3
mysql> desc test9;
```

## 删除索引

```mysql
-- 1
mysql> alter table test9 drop primary key;
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

-- 2
mysql> alter table test9 drop index nor_index;   -- 注意这里的indexname是创建普通索引时指定的索引名，而不是字段名。
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

-- 3
mysql> drop index id on test8;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

> 补充：
>
> ```mysql
> mysql> create table test10(
>  -> id int primary key,        -- 主键索引
>     -> name varchar(20) not null,
>     -> person_id int unique,	  -- 唯一键索引
>     -> sal int,
>     -> index(sal)				  -- 普通索引
>     -> );
>    Query OK, 0 rows affected (0.02 sec)
> 
> mysql> show index from test10\G;
> *************************** 1. row ***************************
>      Table: test10
>    Non_unique: 0
>      Key_name: PRIMARY
>    Seq_in_index: 1
>  Column_name: id
>    Collation: A
>    Cardinality: 0
>     Sub_part: NULL
>        Packed: NULL
>          Null: 
>    Index_type: BTREE
>       Comment: 
>    Index_comment: 
> *************************** 2. row ***************************
>      Table: test10
>    Non_unique: 0
>      Key_name: person_id
>    Seq_in_index: 1
>  Column_name: person_id
>    Collation: A
>    Cardinality: 0
>     Sub_part: NULL
>        Packed: NULL
>          Null: YES
>    Index_type: BTREE
>       Comment: 
>    Index_comment: 
> *************************** 3. row ***************************
>      Table: test10
>    Non_unique: 1
>      Key_name: sal
>    Seq_in_index: 1
>  Column_name: sal
>    Collation: A
>    Cardinality: 0
>     Sub_part: NULL
>        Packed: NULL
>          Null: YES
>    Index_type: BTREE
>       Comment: 
>    Index_comment: 
> 3 rows in set (0.00 sec)
> 
> ERROR: 
> No query specified
> ```
> 
>本质上唯一键索引和普通索引的不同仅在于唯一键索引的Non_unique为0，也就是不能重复，且可以为null（Null： YES）。而普通索引键值可以重复且可以为null。
> 
>故，本质上唯一键索引B+树和普通索引B+树的结构是一样的，实现也是一样的，它们的叶子Page都不存储数据记录，而是对应的主键值。(InnoDB)

# Other

下方为扩展内容，用于加深理解，大致了解即可（有小部分内容与上方有重复）（🤭）

## MySQL存储引擎

### 什么是存储引擎？

数据库存储引擎是数据库底层软件组件，数据库管理系统使用数据引擎进行创建、查询、更新和删除数据操作。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎还可以获得特定的功能。

现在许多数据库管理系统都支持多种不同的存储引擎。MySQL 的核心就是存储引擎。

- InnoDB：事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键。MySQL 5.5.5 之后，InnoDB 作为默认存储引擎。
- MyISAM 是基于 ISAM 的存储引擎，并对其进行扩展，是在 Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM 拥有较高的插入、查询速度，但不支持事务。
- MEMORY 存储引擎将表中的数据存储到内存中，为查询和引用其他数据提供快速访问。

### InnoDB

**使用InnoDB时，会将数据表分为`.frm` 和 `.ibd` 两个文件进行存储。**

- **.frm : 存储表结构**
- **.ibd : 存储表数据和索引**

> **innodb的所有数据文件（后缀为ibd的文件），他的大小始终都是16384（16k）的整数倍。**

**InnoDB索引实现(聚簇)**

- 表数据文件本身就是按B+Tree组织的一个索引结构文件
- **聚簇索引-叶节点包含了完整的数据记录**
- **为什么InnoDB表必须有主键，并且推荐使用整型的自增主键？**
- **为什么非主键索引结构叶子节点存储的是主键值？(一致性和节省存储空间)**

#### InnoDB一棵主键索引B+树可以存放多少行数据？

**这个问题的简单回答是：约2千万**。**为什么是这么多呢？因为这是可以算出来的**，要搞清楚这个问题，我们先从InnoDB索引数据结构、数据组织方式说起。

我们都知道计算机在存储数据的时候，有最小存储单元，这就好比我们今天进行现金的流通最小单位是一毛。在计算机中磁盘存储数据最小单元是扇区，一个扇区的大小是512字节，而文件系统（例如XFS/EXT4）他的最小单元是块，一个块的大小是4k，而对于我们的InnoDB存储引擎也有自己的最小储存单元——页（Page），一个页的大小是16K。

> 下面几张图可以帮你理解最小存储单元：
>
> 文件系统中一个文件大小只有1个字节，但不得不占磁盘上4KB的空间。
>
> <img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/667853-20201209221556175-212431100.png" alt="img" style="zoom:50%;" />
>
> **innodb的所有数据文件（后缀为ibd的文件），他的大小始终都是16384（16k）的整数倍。**
>
> <img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/3bbecff2e182c9c19b30c51288aba79a.png" alt="image.png" style="zoom:50%;" />
>
> 磁盘扇区、文件系统、InnoDB存储引擎都有各自的最小存储单元。
>
> ![img](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/667853-20201209221718628-2070547995.png)

**数据表中的数据都是存储在页中的，所以一个页中能存储多少行数据呢？假设一行数据的大小是1k，那么一个页可以存放16行这样的数据。**

那么回到我们开始的问题，通常一棵B+树可以存放多少行数据？

这里我们先假设B+树高为2，即存在一个根节点和若干个叶子节点，那么这棵B+树的存放总记录数为：根节点指针数*单个叶子节点记录行数。

上文我们已经说明单个叶子节点（页）中的记录数=16K/1K=16。（**这里假设一行记录的数据大小为1k，实际上现在很多互联网业务数据记录大小通常就是1K左右**）。

那么现在我们需要计算出非叶子节点能存放多少指针，其实这也很好算，我们假设主键ID为bigint类型，长度为8字节，而指针大小在InnoDB源码中设置为6字节，这样一共14字节，我们一个页中能存放多少这样的单元，其实就代表有多少指针，即`16384/14=1170`。那么可以算出一棵高度为2的B+树，能存放`1170*16=18720`条这样的数据记录。

根据同样的原理我们可以算出一个高度为3的B+树可以存放：`1170*1170*16=21902400`条这样的记录。**所以在InnoDB中B+树高度一般为1-3层，它就能满足千万级的数据存储。在查找数据时一次页的查找代表一次IO，所以通过主键索引查询通常只需要1-3次IO操作即可查找到数据。**

#### 怎么得到InnoDB主键索引B+树的高度？

上面我们通过推断得出B+树的高度通常是1-3，下面我们从另外一个侧面证明这个结论。**在InnoDB的表空间文件中，约定page number为3的代表主键索引的根页，而在根页偏移量为64的地方存放了该B+树的page level**。如果page level为1，树高为2，page level为2，则树高为3。即B+树的高度=page level+1；下面我们将从实际环境中尝试找到这个page level。

在实际操作之前，你可以通过InnoDB元数据表确认主键索引根页的page number为3，你也可以从《InnoDB存储引擎》这本书中得到确认。

#### 为什么InnoDB表必须有主键，并且推荐使用整型的自增主键？

> 在MySQL的InnoDB存储引擎下，如果您创建一个表并没有显式地指定主键，InnoDB会自动生成一个隐藏的主键。这个隐藏的主键通常是一个名为"GEN_CLUST_INDEX"的聚集索引（Clustered Index），在物理上实际上是用于组织表数据的B-tree结构。
>
> 这个隐藏的主键在背后实际上是一个包含递增整数值的列，用于唯一标识每一行数据。如果没有显式定义主键，InnoDB会自动为这个隐藏列创建一个主键，确保每一行都有一个唯一的标识。
>
> 需要注意的是，由于这个隐藏主键实际上是InnoDB引擎内部生成的，您不能对其进行直接的操作，也不需要在查询或操作时显式地指定这个主键。但是，如果您后续需要根据某个列来进行唯一标识和操作数据，最好还是显式地定义一个主键，这样可以更清晰地控制表的结构和数据完整性。

1. **如果用户设置了主键，那么InnoDB会根据指定的主键创建主键索引，如果没有显式定义主键，则InnoDB会选择第一个不包含有NULL值的唯一键作为主键、如果也没有这样的唯一键，则InnoDB会选择内置6字节长的ROWID作为隐含的键值并构建对应的聚集索引(ROWID随着行记录的写入而逐渐递增)。**
2. **如果表使用自增主键，**那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，主键的顺序按照数据记录的插入顺序排列，自动有序。当一页写满，就会自动开辟一个新的页。
3. **如果使用非自增主键（如身份证号或学号等），**由于每次插入主键的值近似于随机，因此每次新纪录都要被插到现有索引页得中间某个位置，此时MySQL不得不为了将新记录插到合适位置而移动数据，甚至目标页面可能已经从缓存中清掉并被回写到磁盘上，此时又要从磁盘上读回来，这增加了很多开销。
