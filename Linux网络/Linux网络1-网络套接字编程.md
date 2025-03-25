## 字节序问题

网络字节序是大端模式

**为使网络程序具有可移植性,使同样的C代码在大端和小端计算机上编译后都能正常运行**,可以调用以下库函数做网络字节序和主机字节序的转换。

```C++
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);//将无符号长整型的主机字节序转换为网络字节序

uint16_t htons(uint16_t hostshort);//将无符号短整型的主机字节序转换为网络字节序

uint32_t ntohl(uint32_t netlong);//将无符号长整型的网络字节序转换为主机字节序

uint16_t ntohs(uint16_t netshort);//将无符号短整型的网络字节序转换为主机字节序

// h代表主机字节序，n代表网络字节序，l代表长整型，s代表短整型
// 例如htonl表示将32位的长整数从主机字节序转换为网络字节序,例如将IP地址转换后准备发送。
// 如果主机是小端字节序,这些函数将参数做相应的大小端转换然后返回;
// 如果主机是大端字节序,这些函数不做转换,将参数原封不动地返回。
```

# socket编程

## socket 常见API

```C++
// 创建 socket 文件描述符 (TCP/UDP, 客户端 + 服务器)
int socket(int domain, int type, int protocol);
// 绑定端口号 (TCP/UDP, 服务器) 
int bind(int socket, const struct sockaddr *address,
 socklen_t address_len);
// 开始监听socket (TCP, 服务器)
int listen(int socket, int backlog);
// 接收请求 (TCP, 服务器)
int accept(int socket, struct sockaddr* address,
 socklen_t* address_len);
// 建立连接 (TCP, 客户端)
int connect(int sockfd, const struct sockaddr *addr,
 socklen_t addrlen);
```

前面讲过,struct sockaddr *是一个通用指针类型,myaddr参数实际上可以接受多种协议的sockaddr结构体,而它们的长度各不相同,所以需要第三个参数addrlen指定结构体的长度;

## sockaddr结构

socket API是一层抽象的网络编程接口（程序通用性，兼容性比较好）,适用于各种底层网络协议,如IPv4、IPv6,以及后面要讲的UNIX Domain Socket. 然而, 各种网络协议的地址格式并不相同.

<img src="C:\Users\yangzilong\AppData\Roaming\Typora\typora-user-images\image-20230812232034204.png" alt="image-20230812232034204" style="zoom:80%;" />

- IPv4和IPv6的地址格式定义在netinet/in.h中,IPv4地址用sockaddr_in结构体表示,包括16位地址类型, 16 位端口号和32位IP地址.

- IPv4、IPv6地址类型分别定义为常数AF_INET、AF_INET6. 这样,只要取得某种sockaddr结构体的首地址, 不需要知道具体是哪种类型的sockaddr结构体,就可以根据地址类型字段确定结构体中的内容.
- socket API可以都用struct sockaddr *类型表示, 在使用的时候需要强制转化成sockaddr_in; 这样的好 处是程序的通用性, 可以接收IPv4, IPv6, 以及UNIX Domain Socket各种类型的sockaddr结构体指针做为参数;

## 地址转换函数

本节只介绍基于IPv4的socket网络编程,sockaddr_in中的成员struct in_addr sin_addr表示32位 的IP 地址 但是我们通常用点分十进制的字符串表示IP 地址,**以下函数可以在字符串表示和in_addr表示之间转换;**

`inet_addr(_ip.c_str())`  服务端bind

`inet_ntoa(client.sin_addr)`  服务端accept获取客户端连接

.............

# 简单的UDP/TCP网络程序

[hhhhhh](https://github.com/DaysOfExperience/Linux_Network_Programming)

> 结合代码稍微一看就行了，不难且大概率不考~

## TCP socket API 详解

`int socket(int domain, int type, int protocol)`

- socket()打开一个网络通讯端口,如果成功的话,就像open()一样返回一个文件描述符;**应用程序可以像读写文件一样用read/write在网络上收发数据。（一切皆文件）**

`int bind(int socket, const struct sockaddr *address, socklen_t address_len)`

- 服务器程序所监听的网络地址和端口号通常是固定不变的,客户端程序得知服务器程序的地址和端口号后 就可以向服务器发起连接; **服务器需要调用bind绑定一个固定的网络地址和端口号;**

- **bind()的作用是将参数sockfd和myaddr绑定在一起, 使sockfd这个用于网络通讯的文件描述符监听myaddr所描述的地址和端口号;**

- 网络地址为INADDR_ANY, 这个宏表示本地的任意IP地址,因为服务器可能有多个网卡,每个网卡也可能绑定多个IP 地址, 这样设置可以在所有的IP地址上监听,直到与某个客户端建立了连接时才确定下来到底用哪个IP地址;

`int listen(int socket, int backlog)`

- listen()声明sockfd处于监听状态, 并且最多允许有**backlog**个客户端处于连接等待状态, 如果接收到更多的连接请求就忽略, 具体细节同学们课后深入研究;

`int accept(int socket, struct sockaddr* address, socklen_t* address_len)`

- 三次握手完成后, 服务器调用accept()接受连接; 
- 如果服务器调用accept()时还没有客户端的连接请求,就阻塞等待直到有客户端连接上来;
- addr是一个输出型参数,accept()返回时传出客户端的地址和端口号;
- 如果给addr参数传NULL,表示不关心客户端的地址;
- addrlen参数是一个传入传出参数(value-result argument), 传入的是调用者提供的, 缓冲区addr的长度 以避免缓冲区溢出问题, 传出的是客户端地址结构体的实际长度(有可能没有占满调用者提供的缓冲区);

`int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`

- 客户端需要调用connect()连接服务器
- connect和bind的参数形式一致, 区别在于bind的参数是自己的地址, 而connect的参数是对方的地址;

补充说明：关于bind函数的调用

- 由于客户端不需要固定的端口号,因此不必调用bind(),客户端的端口号由内核自动分配.
- 客户端不是不允许调用bind(), 只是没有必要调用bind()固定一个端口号. 否则如果在同一台机器上启动多个客户端, 就会出现端口号被占用导致不能正确建立连接;
- 服务器也不是必须调用bind(), 但如果服务器不调用bind(), 内核会自动给服务器分配监听端口, 每次启动服务器时端口号都不一样, 客户端要连接服务器就会遇到麻烦;