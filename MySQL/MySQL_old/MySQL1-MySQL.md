# 服务器，数据库，表

所谓安装**数据库服务器**，只是在机器上安装了一个**数据库管理系统程序，这个管理程序可以管理多个数据库**，一般开发人员会针对每一个应用创建一个数据库(database)。

为保存应用中实体的数据，一般会在数据库中创建多个表，以保存程序中实体的数据。

数据库服务器、数据库和表的关系如下

![image-20231121170523119](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231121170523119.png)

# MySQL是一套网络服务

也就是存在MySQL服务端，存在MySQL客户端。

![image-20231121170617649](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231121170617649.png)

上图为正处于Listen状态的MySQL服务端，我们通过MySQL客户端连接MySQL服务端，然后通过SQL指令去向MySQL服务端发起请求

# SQL分类

DDL【data definition language】 数据定义语言，用来维护存储数据的结构 

代表指令: create, drop, alter 

DML【data manipulation language】 数据操纵语言，用来对数据进行操作 

代表指令： insert，delete，update 

- DML中又单独分了一个DQL，数据查询语言，代表指令： select 

DCL【Data Control Language】 数据控制语言，主要负责权限管理和事务

代表指令： grant，revoke，commit