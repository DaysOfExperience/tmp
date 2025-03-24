1.fork子进程继承父进程内存数据，2. 继承文件描述符表 3. 写时拷贝。  所以，不会很低效。

![image-20230902125943153](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902125943153.png)







手动持久化一下 bgsave，发现rdb文件确实变化了。

而重启redis服务器之后，就会发现他会自动根据rdb文件进行内存中的数据恢复。我们用client查看key都是恢复好的。

![image-20230902134458430](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902134458430.png)

![image-20230902135606499](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902135606499.png)





![image-20230902162605715](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902162605715.png)

这里在fork之后，父进程也就是Redis服务端处理客户端的命令的那个进程。是要同时写两个缓冲区的![image-20230902172309581](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902172309581.png)

混合持久化





有关信号的理解：可以向捕捉信号的方向思考。



----

原子性其实可以理解为是事务的初心了。





---

理解为什么watch的实现中这个方案是一个乐观锁。



就是它预估出现冲突的概率较低，所以没有执行串型的加锁方案，而是预估概率较小，可以“并发”执行，如果确实冲突了，再不执行最终的exec命令即可。





![image-20230902210446509](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902210446509.png)

这里要和一个机器所能提供的硬件资源联系，包括但不限于CPU资源，内存资源，硬盘资源，网络带宽资源。



注意，主从结构下，并发量上升，但是从只能读哦。并且可用性提高，但是问题是，从节点挂了，问题不大，此时从主或者其他从同步即可。但是主如果挂了，此时就没法写了，只能提供读了。所以这也是个问题。

![image-20230902212122210](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902212122210.png)

写的并发和可用性没有提高！！！！![image-20230902212851523](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902212851523.png)

![image-20230902213632156](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902213632156.png)

![image-20230902214056018](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902214056018.png)

这两个是配套的，其实就是上方指令利用了下方功能。

![image-20230902215131672](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902215131672.png)

![image-20230902220213296](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230902220213296.png)

官网有介绍~



全量同步：先把master的当前的全部数据都同步给slave，然后后续的每一条指令再逐个同步给slave，这就是增量同步。



replicationid描述了数据的来源，offset描述数据的同步进度。

也就是说，如果现在有两个从节点，它们的repliid和offset都一样，则它们存储的数据也就是一样的。如果有一个不一样，则不同。

![image-20230903203053533](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903203053533.png)

![image-20230903203238499](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903203238499.png)







部分复制的积压缓冲区：一个数组结构的环形队列？ 哈哈 等我之后看了源码再来考证！





![image-20230903212536746](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903212536746.png)



2主要是：主机挂了，写就无法进行了。





![image-20230903213101466](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903213101466.png)

![image-20230903213122016](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903213122016.png)

![image-20230903213241010](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903213241010.png)

主节点起不了了（继承上节课的最后情况）

![image-20230903214132275](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903214132275.png)

![image-20230903214157306](C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230903214157306.png)