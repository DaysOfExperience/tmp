# DNS

## DNS与域名

在我们访问某个网站的时候，使用的都不是IP地址，而是使用由符号和字母构成的一段字符串。例如https://openai.com/blog/chatgpt。但是按照前面的IP协议格式所描述的，并没有域名这一说，使用的都是IP地址，**那么是如何通过域名来获取IP地址呢？这便是借助到了DNS技术。**

**DNS**：Domain Name System - 域名系统，**用于存储IP地址与域名的映射关系，并且提供域名解析, 即通过域名来获取服务器IP地址。**
**域名**：服务器地址的别名，便于我们记忆，最终通过域名访问服务器的时候通过解析域名获取IP地址来访问服务器的。

具体的转换流程

![image-20231120194356961](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120194356961.png)

- DNS是应用层协议, DNS底层使用UDP进行域名解析
- 浏览器会缓存DNS结果，故不是每一次通过域名访问服务器时都需要先访问DNS域名服务器解析域名

## 域名服务器

上面提到了有一个叫做hosts的文件记录了域名以及IP地址的映射关系，**而存储这些映射关系的则是域名服务器。**(否则它没法解析啊..)

1. 首先产生的是**根域名服务器**，根域名服务器是最高级别域名服务器，但是为了分摊访问压力进行容灾处理全世界一共有13台根域名服务器，分布在世界各地。
2. 紧接着则是**顶级域名服务器**，这些服务器大多以所属的国家和所属组织的命名。例如代表中国的cn,代表政府的gov等。例如**.com  .org .gov .edu .cn .net等。**
3. 在紧接着，就是**二级域名服务器**，这类服务器往往是某一企业或者服务商所管理。例如.csdn.net   .qq.com   .baidu.com等等
   这些服务器还能再继续往下授权，可以授权出三级域名服务器，如百度知道的域名：zhidao.baidu.com。

例如百度知道

![image-20231120194637584](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120194637584.png)

通常情况下，当我们要访问某个域名时，会按照域名服务器的等级一层一层向下查询。

## 域名的解析流程

当输入一个域名的时候，如何解析？

1. 查看浏览器缓存
2. C:\Windows\System32\drivers\etc路径下的本机hosts文件
3. 查找本地域名服务器
4. 如果上面的几种方法都没有查找到，那么就进入了第四步，向其他的几个域名服务器进行查找
   此时有两种查询方法，递归查询和迭代查询

**递归查询**：逐层深入，从根域名服务器查找，如果没有，则继续从根域名服务器向顶级服务器开始一级一级向下查找
找到则返回，找不到则逐级向下查找。

![image-20231120194845646](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120194845646.png)

**迭代查找**：逐个访问各级域名服务器
首先查找根域名服务器，如果没有则继续由本地域名服务器向顶级域名服务器发起查找，其此类推

![image-20231120194855070](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120194855070.png)

# ICMP

ICMP协议是一个网络层协议 。（与IP同层）**Internet Control Message Protocol** (互联网控制协议)

一个新搭建好的网络, 往往需要先进行一个简单的测试, 来验证网络是否畅通; 但是**IP协议并不提供可靠传输. 如果丢包了, IP协议并不能通知传输层是否丢包以及丢包的原因**，所以此时就需要用到**ICMP来进行网络诊断。**

ICMP的消息一般分为两类：**一类是通知出错原因，一类是用于诊断查询。**

![image-20231120194940923](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120194940923.png)

- 确认IP包是否成功到达目标地址.
- 通知在发送过程中IP包被丢弃的原因.
- ICMP也是基于IP协议工作的. 但是它并不是传输层的功能,因此人们仍然把它归结为网络层协议;
- ICMP只能搭配IPv4使用. 如果是IPv6的情况下, 需要是用ICMPv6;

## ping

通常我们想测试网络连通性时所使用的ping命令，就是基于ICMP协议。

![image-20231120203036619](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120203036619.png)

- 注意, 此处 ping 的是域名, 而不是URL，一个域名可以通过DNS解析成IP地址.
- ping命令不光能验证网络的连通性, 同时也会统计响应时间和TTL(IP包中的Time To Live, 生存周期).
- ping命令会先发送一个 ICMP Echo Request给对端;
- 对端接收到之后, 会返回一个ICMP Echo Reply;

而网上常见的一个问题，ping命令属于什么端口？这个问题其实就是一个陷阱，因为ping基于ICMP协议，而ICMP又处于网络层，网络层并不关注端口号这一信息，端口号是属于传输层的内容。

## 补充

**虽然 ICMP 是网络层协议**，但是它不像 IP 协议和 ARP 协议一样直接将报文传递给数据链路层，而**是先封装成 IP 数据报然后再传递给数据链路层。**所以在 IP 数据包中如果协议类型字段的值是 1 的话，就表示 IP 数据是 ICMP 报文。IP 数据报就是靠这个协议类型字段来区分不同的数据包的。如下图 WireShark 抓包所示：

![image-20231120203308291](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120203308291.png)

**在 IP 通信中如果某个包因为未知原因没有到达目的地址，那么这个具体的原因就是由 ICMP 负责告知。**而 ICMP 协议的类型定义中就清楚的描述了各种报文的含义。

# 代理服务

> VPN英文全称是“Virtual Private Network”，即“虚拟专用网络”。

想必这个词大家一定非常熟悉，在我们访问一些网站，或者玩游戏的时候，通常都会使用加速器和代理服务，使网络更加流畅，那么他是如何做到的呢？

**其实代理服务器就是个人网络与互联网服务商之间的中间代理机构。**

打个比方，比如我们直接访问服务器，就相当于我们直接去生产厂家买东西，此时可能因为路途遥远，或者手续繁琐导致不方便。而通过代理服务器请求服务，就相当于我们在代销商这里买东西，他把货从遥远的地方进过来，手续他也处理好了，我们只需要付钱拿货就行。

例如我们访问一些国外的网站时，直接访问是不行的，而通过代理服务器请求服务，让他来获取数据，再将数据转发给我们，此时我们也就能获取到了这个网站的数据。

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120210417705.png" alt="image-20231120210417705" style="zoom: 50%;" />

当使用代理服务器访问同一个目标服务器的人多了之后，为了提高效率，不用每次都去转发来获取数据，代理服务器可能会提前将目标服务器的内容缓存下来，有人访问时直接将缓存的数据发送回去，这就是反向代理的技术，而前面所说的转发的那种是正向代理。

代理服务是一个应用层服务，可以部署在任意设备上，工作在应用层，我们个人向代理服务器发起请求，然后他再去访问目标服务器。
而NAT服务是一个网络层服务，部署在网关设备上，工作在网络层，我们请求的是目标服务器

> - 从应用上讲, NAT设备是网络基础设备之一, 解决的是IP不足的问题. 代理服务器则是更贴近具体应用, 比如通过代理服务器进行外网访问, 另外像迅游这样的加速器, 也是使用代理服务器.
> - 从底层实现上讲, NAT是工作在网络层, 直接对IP地址进行替换. 代理服务器往往工作在应用层.
> - 从使用范围上讲, **NAT一般在局域网的出口部署(WAN口IP为公网IP)**, 代理服务器可以在局域网做, 也可以在广域网做
> - 从部署位置上看, NAT一般集成在防火墙, 路由器等**硬件设备**上, 代理服务器则是一个软件程序, **需要部署在服务器上**

# DHCP

DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）

如果每到一个新地方，每连接一台新设备，都需要设置一次IP地址，这无疑是一件非常麻烦的事情。(对于我们自己而言)

所以**为了实现自动设置IP地址，统一管理IP地址分配，就产生了DHCP**

**路由器又可以看作是一个DHCP服务器**

![image-20231120210947201](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120210947201.png)

当主机连接到网关设备时，就会广播DHCP请求，获取到网关的IP地址，以及DHCP服务自动分配给自己的IP地址，生成自己的路由表。

![image-20231120211148645](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231120211148645.png)

# 通信流程

浏览器中输入url后，发生的事情

通过DNS将URL中的域名解析成为IP地址。
根据HTTP协议组织请求数据 - HTTP请求报文
搭建TCP客户端与服务器建立连接，发送HTTP请求
等待HTTP响应，得到HTML响应后进行页面解析渲染

通过QQ发送一条消息时，发生的事情

应用层：将数据组织成qq应用层协议格式
传输层：搭建tcp客户端，与qq服务器建立连接，并且封装传输层信息（通过端口描述哪两个进程在进行通信）。
网络层：传输层将TCP报文数据交给网络层进一步封装（通过ip地址描述哪两台主机在进行通信）
数据链路层：通过arp请求获取指定ip地址的相邻设备的mac地址，将网络层传来的IP报文进行局域网转发。
物理层：通过光电信号发送

当内网设备与外网通信时，发生的事情

主机连接到网关设备（家里的路由器）时，会广播DHCP请求，获取到网关的ip地址，以及给自己动态分配的ip地址，生成自己的路由表。
将数据发送到网关设备上，路由器会根据自己的路由表和目的地址，逐个匹配构建出一条路径，决定数据应该发送给哪一个相连网络。
发送时将内网地址通过NAT技术映射成为统一的外网地址，并且通过NAPT技术进行映射(指定回复时应该转交给内网哪个主机)
路由器连接指定相邻网络的网卡，将数据发送到那个网络。
数据经过这样层层转发之后到达目的网络

> 其实这里就是之前的 应用层 传输层 网络层 数据链路层学习之后的一个总结, 因为网络中传输数据就是这样一个过程