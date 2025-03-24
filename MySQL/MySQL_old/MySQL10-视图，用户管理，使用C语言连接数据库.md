# 视图

**若某复杂查询经常进行且表结构变化不大，则可以创建视图方便查询。**

```mysql
-- 创建视图
create view 视图名 as select语句;

mysql> create view myview as select ename, emp.deptno, dname from emp inner join dept on emp.deptno=dept.deptno;
Query OK, 0 rows affected (0.00 sec)

mysql> create view mview as select ename, emp.deptno, dname from emp, dept where emp.deptno = dept.deptno;
Query OK, 0 rows affected (0.00 sec)

-- 删除视图
drop view 视图名;

mysql> drop view myview;
Query OK, 0 rows affected (0.00 sec)

mysql> drop view mview;
Query OK, 0 rows affected (0.00 sec)
```

- 修改了视图，对基表数据有影响. 修改了基表，对视图有影响

**视图规则和限制**

- 与表一样，命名必须唯一（不能出现同名视图或表） 
- 视图不能添加索引，也不能有关联的触发器或者默认值
- 视图可以提高安全性，必须具有足够的访问权限（user需要有相应的权限来访问视图）
- order by可以用在视图中，select视图时的order by与创建视图时的order by不冲突。
- 视图可以和表一起使用：可以将视图和表进行内联，外联（左联，右联）

# 用户管理

## 用户

如果我们只能使用root用户，这样存在安全隐患。这时，就需要使用MySQL的用户管理。

![image-20230830184002894](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230830184002894.png)

MySQL中的用户，都存储在系统数据库mysql的user表中。

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| db1                |
| db2                |
| db_base            |
| db_index           |
| db_transaction     |
| mysql              |
| performance_schema |
| scott              |
| sys                |
+--------------------+
10 rows in set (0.00 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
31 rows in set (0.00 sec)

mysql> select User, Host, authentication_string from user;
+---------------+-----------+-------------------------------------------+
| User          | Host      | authentication_string                     |
+---------------+-----------+-------------------------------------------+
| root          | localhost | *270D82E090E413A08A51EF3ECA653F031588A4FD |
| mysql.session | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys     | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
+---------------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)

mysql> desc user;
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                  | Type                              | Null | Key | Default               | Extra |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                   | char(60)                          | NO   | PRI |                       |       |
| User                   | char(32)                          | NO   | PRI |                       |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Insert_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Update_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Delete_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Create_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Drop_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Reload_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Shutdown_priv          | enum('N','Y')                     | NO   |     | N                     |       |
| Process_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| File_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Grant_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| References_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Index_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Show_db_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Super_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tmp_table_priv  | enum('N','Y')                     | NO   |     | N                     |       |
| Lock_tables_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Execute_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_slave_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_client_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Create_view_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Show_view_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Create_routine_priv    | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_routine_priv     | enum('N','Y')                     | NO   |     | N                     |       |
| Create_user_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Event_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Trigger_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tablespace_priv | enum('N','Y')                     | NO   |     | N                     |       |
| ssl_type               | enum('','ANY','X509','SPECIFIED') | NO   |     |                       |       |
| ssl_cipher             | blob                              | NO   |     | NULL                  |       |
| x509_issuer            | blob                              | NO   |     | NULL                  |       |
| x509_subject           | blob                              | NO   |     | NULL                  |       |
| max_questions          | int(11) unsigned                  | NO   |     | 0                     |       |
| max_updates            | int(11) unsigned                  | NO   |     | 0                     |       |
| max_connections        | int(11) unsigned                  | NO   |     | 0                     |       |
| max_user_connections   | int(11) unsigned                  | NO   |     | 0                     |       |
| plugin                 | char(64)                          | NO   |     | mysql_native_password |       |
| authentication_string  | text                              | YES  |     | NULL                  |       |
| password_expired       | enum('N','Y')                     | NO   |     | N                     |       |
| password_last_changed  | timestamp                         | YES  |     | NULL                  |       |
| password_lifetime      | smallint(5) unsigned              | YES  |     | NULL                  |       |
| account_locked         | enum('N','Y')                     | NO   |     | N                     |       |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
45 rows in set (0.00 sec)
```

目前，mysql中只有三个默认的用户，我还没有额外添加用户。

- `host`： 表示这个用户可以从哪个主机登陆，如果是localhost，表示只能从本机登陆
- `user`： 用户名
- `authentication_string`： 用户密码通过password函数加密后的密文
- `*_priv`： 用户拥有的权限

```mysql
-- 创建用户
create user '用户名'@'登陆主机/ip' identified by '密码';
-- 删除用户
drop user '用户名'@'主机名'
-- 修改用户密码
set password=password('新的密码');  -- 自己改自己的
set password for '用户名'@'主机名'=password('新的密码')； -- root修改某用户密码

mysql> create user 'yzl'@'localhost' identified by '111222zzz';
Query OK, 0 rows affected (0.00 sec)

mysql> create user 'yzl'@'%' identified by 'hhh';
Query OK, 0 rows affected (0.00 sec)

mysql> 
mysql> select User, Host, authentication_string from user;
+---------------+-----------+-------------------------------------------+
| User          | Host      | authentication_string                     |
+---------------+-----------+-------------------------------------------+
| root          | localhost | *270D82E090E413A08A51EF3ECA653F031588A4FD |
| mysql.session | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys     | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| yzl           | localhost | *6DC1FB654712186D86D7EA31ABBFF61B3721A6BA |
| yzl           | %         | *ADCB3111F8CE421E27114A63697CE697887F430F |
+---------------+-----------+-------------------------------------------+
5 rows in set (0.00 sec)
mysql> drop user 'yzl'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> drop user 'yzl'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

## 权限管理

MySQL数据库提供的权限列表：

![image-20230830185855394](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230830185855394.png)

刚创建的用户没有任何权限。需要给用户授权。

```mysql
-- 给用户授权
grant 权限列表 on 库.对象名 to '用户名'@'登陆位置' [identified by '密码'];

-- 说明：权限列表，多个权限用逗号分开
grant select on ...
grant select, delete, create on ....
grant all [privileges] on ... -- 表示赋予该用户在该对象（数据库.表）上的所有权限
*.* : 代表本系统中的所有数据库的所有对象（表，视图，存储过程等）
库.* : 表示某个数据库中的所有数据对象(表，视图，存储过程等)
identified by：可选。 如果用户存在，赋予权限的同时修改密码,如果该用户不存在，就是创建用户

-- 查看特定用户的权限
mysql> show grants for 'whb'@'%';

-- 如果发现赋权限后，没有生效，执行如下指令：
flush privileges;

-- 回收权限
revoke 权限列表 on 库.对象名 from '用户名'@'登陆位置';
```

# 使用C语言连接数据库

之前使用的都是Linux下的命令行式的MySQL客户端，实际开发中，并不会直接使用命令行式客户端连接数据库。而是在代码中进行连接。也就是使用语言的库去连接MySQL，比如C，C++，Java很多语言都有这样的库的支持。下面了解如何用C的三方库去连接数据库。

![image-20230830193235948](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230830193235948.png)

上方为动静态库，下方为头文件，都已经安装至系统默认搜索路径下了。

其中 include 包含所有的方法声明， lib 包含所有的方法实现（打包成库）

其中我们直接使用的是`/usr/include/mysql/mysql.h` 以及对应的动静态库

```C++
mysql_get_client_info();     -- 可用来验证是否引入成功
```

mysql接口介绍

```C++
MYSQL *mysql_init(MYSQL *mysql);    // 要使用库，必须先进行初始化！

// 初始化完毕之后，必须先链接数据库，在进行后续操作。（mysql网络部分是基于TCP/IP的）
MYSQL *mysql_real_connect(MYSQL *mysql, const char *host,
						const char *user,
						const char *passwd,
						const char *db,
                        unsigned int port,
                        const char *unix_socket,   // nullptr
                        unsigned long clientflag); // 0
//建立好链接之后，获取英文没有问题，如果获取中文是乱码：
//因为MySQL服务端设置链接的默认字符集是utf8，而语言客户端原始默认是latin1
mysql_set_character_set(myfd, "utf8");

// 第一个参数 MYSQL是 C api中一个非常重要的变量（mysql_init的返回值），里面内存非常丰富，有
// port,dbname,charset等连接基本参数。它也包含了一个叫 st_mysql_methods的结构体变量，该变量
// 里面保存着很多函数指针，这些函数指针将会在数据库连接成功以后的各种数据操作中被调用。

int mysql_query(MYSQL *mysql, const char *q); -- 向mysqld下发mysql命令
// 比如insert delete update select 还有begin commit，命令行怎么写，第二个参数就怎么写，且不需要加;

// 如果是其他指令，比如增删改，则只需要通过mysql_query的返回值判断是否执行成功即可。
// 而如果是查询指令，则需要进一步通过其他方法来获取select结果。

MYSQL_RES *mysql_store_result(MYSQL *mysql);

/*
该函数会调用MYSQL变量中的st_mysql_methods中的 read_rows 函数指针来获取查询的结果。同时该
函数会返回MYSQL_RES 这样一个变量，该变量主要用于保存查询的结果。同时该函数malloc了一片内
存空间来存储查询过来的数据，所以我们一定要记的释放这段空间,不然是肯定会造成内存泄漏的。 执行
完mysql_store_result以后，其实数据都已经在MYSQL_RES 变量中了，下面的api基本就是读取
MYSQL_RES 中的数据。
*/
my_ulonglong mysql_num_rows(MYSQL_RES *res);   // 获取结果行数mysql_num_rows
unsigned int mysql_num_fields(MYSQL_RES *res); // 获取结果列数mysql_num_fields
MYSQL_FIELD *mysql_fetch_fields(MYSQL_RES *res); // 获取列名mysql_fetch_fields
// 实际上，MYSQL_FIELD是一个结构体，要想获取列名，或者列的类型，是获取它的数据成员。
typedef struct st_mysql_field {
  char *name;                 /* Name of column */
  char *org_name;             /* Original column name, if an alias */
  char *table;                /* Table of column if column was a field */
  char *org_table;            /* Org table name, if table was an alias */
  char *db;                   /* Database for table */
  char *catalog;	      /* Catalog for table */
  char *def;                  /* Default value (set by mysql_list_fields) */
  unsigned long length;       /* Width of column (create length) */
  unsigned long max_length;   /* Max width for selected set */
  unsigned int name_length;
  unsigned int org_name_length;
  unsigned int table_length;
  unsigned int org_table_length;
  unsigned int db_length;
  unsigned int catalog_length;
  unsigned int def_length;
  unsigned int flags;         /* Div flags */
  unsigned int decimals;      /* Number of decimals in field */
  unsigned int charsetnr;     /* Character set */
  enum enum_field_types type; /* Type of field. See mysql_com.h for types */
  void *extension;
} MYSQL_FIELD;

MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);
typedef char **MYSQL_ROW;		/* return data as array of strings */
// 它会返回一个MYSQL_ROW变量，MYSQL_ROW其实就是char **
// 注意，此方法每次调用都会自动读取select返回结果(在MYSQL_RES中，也就是mysql_store_result的返回值)的下一行数据，也就是返回一个MYSQL_ROW，实际上，它就是char**类型，也就是一个char* []，我们可以直接根据列数，遍历这个char*[]，即可读取这一行的全部数据。每一列的数据都是一个字符串类型，而实际上表中并不一定是字符串，那么，其中这个列的实际类型，就在


void mysql_close(MYSQL *sock);   // 关闭mysql链接mysql_close

// 另外，mysql C api还支持事务等常用操作，
my_bool STDCALL mysql_autocommit(MYSQL * mysql, my_bool auto_mode);
my_bool STDCALL mysql_commit(MYSQL * mysql);
my_bool STDCALL mysql_rollback(MYSQL * mysql)
```



![image-20231124144209687](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231124144209687.png)

动态库路径以及要连接的动态库肯定是要指定的，因为这不是官方库。

而头文件，如果程序中写的路径是/usr/include/，下的绝对路径，则可以找到，如果不是，则需要-I指定一下头文件搜索路径~

其实还是动静态库的问题，emmm，有点惭愧