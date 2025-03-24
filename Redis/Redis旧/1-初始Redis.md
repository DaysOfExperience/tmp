# 认识Redis

![image-20230831154416932](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230831154416932.png)

开源，内存数据存储，被数百万的开发者用作数据库，缓存，流式引擎和消息代理。

Redis，最大的特点：**在内存中存储数据。**

> 那么，某进程内定义一个变量，不也是在内存中存储数据吗？是的，所以**Redis更适合于分布式系统中**。如果是单机程序，则直接通过变量存储数据的方式，是比Redis更优的选择。
>
> 而在分布式系统中，多个进程可能在多个机器上，因为进程的隔离性，此时进程间要想通信，就必须通过网络。
>
> Redis就是基于网络，将自己在内存中存储的变量（数据）给别的进程，甚至是别的主机的进程使用。

**Redis的用途**

1. **Redis用于database：使用Redis作为数据库。**

> 对比MySQL：MySQL最大的问题在于：访问的速度较慢，主要是因为MySQL将数据存储在磁盘中，磁盘这种硬件IO的效率相比内存就慢了很多了。加上提供的功能支持来说，MySQL比Redis复杂很多。所以，MySQL相比于Redis来说就很慢。

所以，Redis作为数据库时，它是一个内存级数据库，最大的特点就是快。（可以持久化）

而Redis对比MySQL来说，优势是快，劣势就是存储空间有限。

那么，如何结合Redis的快和MySQL的大呢？可以将两者结合起来使用。

2. **Redis作为cache - 缓存**

也就是将Redis作为cache缓存，利用二八原则（20%的热点数据，能满足80%的访问需求），将少量的热点数据放在Redis中，将全量数据放在MySQL中，这样，就可以高速访问大量的数据，如果遇到没有缓存的数据，再去MySQL中获取，也会大大提高整体效率。

但是，虽然这样做的好处是快，且存储空间大。但是坏处是：系统的复杂程度大大提高了，并且如果有数据发生修改，还涉及到Redis和MySQL的数据同步问题。

3. **streaming engine - 消息队列**

而对于第三点streaming engine：这是Redis的初心，也就是作为一个消息中间件（消息队列），即分布式系统下的生产者消费者模型。
但是，目前出现了很多比Redis更专业的消息中间件。所以Redis的实际应用场景中，database和cache比消息中间件更多。

# 关系型数据库 vs 非关系型数据库

关系型数据库（RDBMS）和非关系型数据库（NoSQL）是两种不同类型的数据库管理系统，它们在数据存储、查询和处理方面有很多区别。以下是它们之间的一些主要区别：

**1. 数据模型：**

- **关系型数据库：** 使用表格（表）来存储数据，每个表由行和列组成，数据之间的关系通过表之间的键（主键和外键）建立。
- **非关系型数据库：** 使用不同的数据模型，例如键值对、文档、列族或图形等，以适应不同类型的数据和数据关系。

**2. 数据结构：**

- **关系型数据库：** 数据表需要在设计时定义其结构，包括列名、数据类型和约束，使得**数据的格式和结构严格受控。**
- **非关系型数据库：** **通常更加灵活**，可以存储半结构化和非结构化数据，不需要严格的预定义模式。

**3. 数据一致性：**

- **关系型数据库：** 通常支持 ACID（原子性、一致性、隔离性、持久性）事务，确保数据的一致性和完整性。
- **非关系型数据库：** 一些非关系型数据库可能不提供强一致性，而是更注重可用性和分区容忍性。

**4. 扩展性：**

- **关系型数据库：** 垂直扩展（增加硬件资源，如CPU、内存）相对容易，但水平扩展（在多台机器上分布数据）可能较为复杂。
- **非关系型数据库：** 多数非关系型数据库设计用于水平扩展，能够更好地应对大规模数据和高并发。(分布式)

**5. 查询语言：**

- **关系型数据库：** 使用结构化查询语言 SQL 进行查询和操作。
- **非关系型数据库：** 查询语言因数据库类型而异，有些使用类似 SQL 的查询语言，而其他可能使用自定义的查询API。

**6. 适用场景：**

- **关系型数据库：** 适用于**需要严格数据结构和事务一致性的应用**，如银行系统、ERP系统等。
- **非关系型数据库：** 适用于大数据、实时数据处理、分布式系统、社交媒体、物联网等场景，其中数据结构和关系可能更加动态或复杂。

需要注意的是，关系型数据库和非关系型数据库并不是绝对的对立关系，而是根据不同的应用需求来选择的。有时候也会采用混合的方法，在适当的场景下同时使用两种类型的数据库。（28原则）

# 分布式

...

# Redis的特性（优点）

Redis是一个在内存中存储数据的中间件。(in-memory data store)，用于作为内存数据库或数据缓存（比如填补MySQL读取数据慢的缺点），且在分布式系统中才能大展拳脚。

![image-20230831160916115](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230831160916115.png)

核心能力-核心特性-核心优点

## In-memory data structures

Well-known as a "data structure server", with support for strings, hashes, lists, sets, sorted sets, streams, and more.

**在内存中存储数据。 支持多种数据结构 - 数据结构服务器**

> MySQL - 主要是以"**表**"的方式存储并组织数据，称之为**关系型数据库**。
>
> Redis - 主要是以"**键值对**的方式"存储并组织数据，称之为**非关系型数据库**。
>

基于键值对的数据结构服务器：与很多键值对数据库不同的是， Redis中的值不仅可以是字符串，⽽且还可以是具体的数据结构，这样不仅能便于在许多应⽤场景的开发，同时也能提⾼开发效率。

## Programmablity

**可编程性**

可以直接使用简单的交互式命令来对Redis进行操作，也可以通过一些脚本的方式，批量执行一些操作（可以带有一些逻辑）。这就是可编程性。

## Extensibility

A module API for building custom extensions to Redis in C, C++, and Rust.

**可扩展性：在Redis原有的功能基础上可进行扩展**

Redis提供了一个模块的API，可以用C，C++，Rust构建自定义的Redis扩展。（本质就是一个动态链接库）

比如在Redis已经提供的很多数据结构和命令的基础上，**编写扩展，让Redis支持更多的数据结构与命令。**

## Persistence

Keeps the dataset in memory for fast access, but can also persist all writes to permanent storage to survive reboots and system failures.

**持久性（可持久化）**：（原本）Redis将数据集保存在内存中以便快速访问，但内存是"易失"的，进程退出/系统重启/系统故障，则Redis在内存中存储的数据将丢失。则Redis也可以将数据保存在永久存储中（硬盘）上，以便在机器重启或系统发生故障时进行恢复数据。

内存为主，硬盘为辅。Redis的核心业务还是在内存中完成的，硬盘相当于对内存中的数据进行了备份，如果Redis重启了，就会在重启时加载硬盘中的备份数据，使Redis的内存恢复到重启前的状态。

通常将数据放在内存中是不安全的，⼀旦发⽣断电或者机器故障，重要的数据可能就会丢失，因此 Redis 提供了两种持久化⽅式：RDB 和 AOF，即可以⽤两种策略将内存的数据保存到硬盘中，这样就保证了数据的可持久性，后续我们将对 Redis 的持久化进⾏详细说明。

## Clustering

Horizontal scalability with hash-based sharding, scaling to millions of nodes with automatic re-partitioning when growing the cluster.

**集群**

Redis作为一个分布式系统中的中间件。支持集群功能是很重要的。

简单理解集群：一个Redis能存储的数据有限（内存嘛），而引入多个主机，部署多个Redis节点，就可以存储更多的数据。（上方的水平可扩展性，就类似于MySQL的分库分表...）

> - **关系型数据库：** 垂直扩展（增加硬件资源，如CPU、内存）相对容易，但水平扩展（在多台机器上分布数据）可能较为复杂。
> - **非关系型数据库：** 多数非关系型数据库设计用于水平扩展，能够更好地应对大规模数据和高并发。

## High availability

Replication with automatic failover for both standalone and clustered deployments.

**高可用性**
具有自动故障转移的复制，适用于独立部署和集群部署。

高可用，就是备份。

Redis支持主从结构。从节点可以充当主节点的备份，当主节点发生故障时，可以由从节点来代替主节点。也就是提高了可用性！

**主从复制**（Replication） ：Redis 提供了复制功能，实现了多个相同数据的Redis 副本（Replica），复制功能是分布式Redis的基础。后续我们会对Redis的复制功能进⾏详细演⽰。

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230831203053354.png" alt="image-20230831203053354" style="zoom:50%;" />

Redis 提供了⾼可⽤实现的 Redis 哨兵（Redis Sentinel），能够保证 Redis 结点的故障发现和故障⾃动转移。也提供了 Redis 集群（Redis Cluster），是真正的分布式实现，提供了⾼可⽤、读写和容量的扩展性。

## 快

Redis为什么快？

1. **Redis 的所有数据都是存放在内存中的**
2. **Redis提供的核心功能较简单，是比较简单的逻辑**，核心功能就是操作内存中的数据结构。而反观MySQL，比如外键，主键等等就需要更多的检查，更复杂的逻辑。
3. 从网络角度上，Redis使用了**IO多路复用的方式**（epoll）
4. **Redis使用的是单线程模型**，减少了不必要的线程间的竞争开销，避免了加锁，竞争锁，阻塞等。

   衍生问题：Redis为什么不用多线程？Redis什么情况下会用多线程？
   （其实还是因为Redis的业务场景不适合多线程：CPU密集型的任务可以充分利用CPU的多核资源，而Redis核心任务是直接操作内存中的数据结构，不会吃很多CPU，所以不适合多线程。）
5. **Redis使用C语言开发。**但其实MySQL也是使用C语言开发的。



## 单线程架构

**一、Redis 使⽤了单线程架构来实现⾼性能的内存数据库服务**

我们所谓的 Redis 是采⽤单线程模型执⾏命令的是指：虽然三个客⼾端看起来是同时要求 Redis 去执⾏命令的，但微观⻆度，这些命令还是采⽤线性⽅式去执⾏的，只是原则上命令的执⾏顺序是不确定的，但⼀定不会有两条命令被同步执⾏，不会发⽣并发问题，这个就是 Redis 的单线程执⾏模型。

图略了，不难理解。宏观上，多个客户端同时发送指令给服务端执行，但是微观上，这些指令一定是串型执行的，一个时刻只能执行一个命令。

**二、为什么单线程还能这么快**

1. **纯内存访问。**Redis将所有数据放在内存中，内存的响应时长⼤约为 100 纳秒，这是 Redis 达到每秒万级别访问的重要基础。 
2. **Redis的核心功能比MySQL简单。**（提供的功能更少）：比如MySQL在数据的插入删除时，数据库的各种约束，都要数据库做很多额外工作。

3. 处理网络IO时，**Redis 使⽤ epoll 这样的 I/O 多路复⽤技术**，再加上 Redis ⾃⾝的事件处理模型将 epoll 中的连接、读写、关闭都转换为事件，不在⽹络 I/O 上浪费过多的时间。
   主要是Redis的操作都是短平快的，简单操作一下内存数据结构，不会很消耗CPU，也就不需要多核。
4. **单线程避免了线程切换和竞态产⽣的消耗。**避免了一些不必要的线程竞争开销。单线程可以简化数据结构和算法的实现，让程序模型更简单；其次单线程避免了多线程在线程竞争同⼀份共享数据时带来的切换和等待消耗。

<img src="C:\Users\yangzilong\Desktop\markdown\CS\Redis\Redis旧\image-20230901120706482.png" alt="image-20230901120706482" style="zoom:67%;" />

虽然单线程给 Redis 带来很多好处，但还是有⼀个致命的问题：对于单个命令的执⾏时间都是有 要求的。如果某个命令执⾏过⻓，会导致其他命令全部处于等待队列中，迟迟等不到响应，造成客户端的阻塞，对于 Redis 这种⾼性能的服务来说是⾮常严重的，所以 Redis 是⾯向快速执⾏场景的数据库。

# Redis的应用场景

![image-20230831164601219](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230831164601219.png)

## Real-time data store

Redis' versatile in-memory data structures enable building data infrastructure for real-time applications that require low latency and high-throughput.   Redis的多用途内存数据结构可以为需要低延迟和高吞吐量的实时应用程序构建数据基础设施。

low latency and high-throughput：低延迟和高吞吐量。

内存数据库 - 适用于需要快速获取数据的数据存储的场景。

大多数情况，数据存储考虑的是大，而有些场景下需要快。而Redis在内存中存储的多数据结构数据就可以满足这样的需求。

> 比如搜索引擎的广告搜索，对性能的要求就很高，所以搜索系统中不适合使用MySQL这样的数据库。而适合使用Redis这样的内存数据库。

此处Redis作为一个内存数据库，存储的是全量数据，这样的数据是不允许轻易丢失的。

## Caching & session storage

Redis' speed makes it ideal for caching database queries, complex computations, API calls, and session state.

缓存数据库查询 & session 存储

**Caching**

之前说过了，利用二八原则，MySQL大，慢，而Redis快，小。即可将小部分热点数据存储在Redis中，而全量数据存储在MySQL中，这就是缓存，Caching

**<u>缓存机制几乎在所有大型网站都有使用，合理地使用缓存不仅可以加速数据的访问速度，而且能够有效地降低后端数据源的压力。</u>**
Redis 提供了键值过期时间设置，并且也提供了灵活控制最⼤内存和内存溢出后的淘汰策略。可以这么说，⼀个合理的缓存设计能够为⼀个⽹站的稳定保驾护航。

**session storage**

有关cookie 和 session

cookie保存用户身份信息，存储在浏览器客户端。而实际上身份验证需要session配合，session存储在服务器中，session中真正存储了用户数据。cookie只是在浏览器中存储了一个用户身份标识, sessionID

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230831165819938.png" alt="image-20230831165819938" style="zoom:50%;" />

比如上方的分布式情境中，应用服务器部署在多台机器上，此时客户端的请求通过负载均衡器负载均衡式的提交给应用服务器，此时需要cookie身份验证，而session的存储就是一个问题。

1. 通过UserID之类的方式将一个用户的请求全部提交给同一个应用服务器，这样session会话就可以存储在应用服务器中了。
2. 将会话session数据存储在单独一台机器的Redis中。这样多台应用服务器就可以统一去Redis中获取Session数据。且这样的好处是，即使应用程序重启了，也就是server重启，session依旧不丢失。

## Streaming & messaging

The stream data type enables high-rate data ingestion, messaging, event sourcing, and notifications.

消息队列（服务器）

也就是在分布式系统中，充当一个网络版本的生产者消费者模型。（发挥生产消费模型的优势：解耦合+削峰填谷）

业界的其他知名消息队列：RabbitMQ，Kafka，RocketMQ（message queue）比Redis充当消息队列时更专业一些。

## Redis的缺点

**不能存储大量数据**（根本原因就是In-memory data store）

站在数据规模的⻆度，虽然现在内存已经⾜够便宜，但是⼤规模数据使⽤ Redis 来存储的话，基本上是个⽆底洞，经济成本相当⾼。

站在数据冷热的⻆度，例如对于视频⽹站来说，单纯站在数据冷热的⻆度上看，视频信息属于热数据，⽤⼾观看记录属于冷数据。如果将这些冷数据放在 Redis 上，基本上是对于内存的⼀种浪费，但是对于⼀些热数据可以放在 Redis 中加速读写，也可以减轻后端存储的负载，可以说是事半功倍。   其实还是说, Redis - 内存存储, 所以空间有限, 如果存储一些冷数据, 就会造成内存浪费

---

# PDF

第⼀章 初识 Redis

1.1 盛赞Redis

1.2 Redis特性 - 八个点

1.3 Redis的使用场景

​	1.3.1 Redis可以做什么 - 5个例子

​	1.3.2 Redis不可以做什么 - 两个点

1.4 Redis 重大版本

1.5 安装并启动Redis
