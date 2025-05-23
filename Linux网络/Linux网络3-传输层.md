# 计算机网络的层状结构

![image-20231118195839175](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118195839175.png)

网络传输时，向下要封装报头，向上要解包，也就是去掉报头。发送方需要将应用层数据包逐层向下，添加每层的报头进行封装。接收方的数据链路层通过物理层接收到对方传来的报文时，需要逐层向上去掉报头，进行解包，提取出最终的有效载荷。也就是网络发送方真正要传输的数据。

几乎任何协议，都要首先解决两个问题：1. 如何分离（将报头和有效载荷拆分开，接收方网络协议栈的每一层都需要做的）和如何封装（发送方做的，添加报头）2. 如何向上交付（有效载荷拆分出来之后，交付给上一层）

套接字 = IP + 端口号。IP是网络层IP协议报头包含的字段，标识着网络传输时数据从哪个主机传输给哪个主机。端口号是传输层协议报头包含的字段，对应着传输层报文中的有效载荷（应用层数据包）应该交付给该主机上的哪个进程。这样对应进程收到传输层的有效载荷之后，就可以根据应用层协议，将应用层报文中的有效载荷提取出来。

# 传输层

传输层是整个网络体系结构中的关键层次之一，主要负责两个主机中**进程之间的数据传输。**
在传输层中常见的协议就是TCP协议（传输控制协议），UDP协议（用户数据报协议）

# UDP协议

**UDP协议，即用户数据报协议**

## UDP协议格式

![image-20231118200415984](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118200415984.png)

- **16位源端口号 / 目的端口号**：标明了此UDP报文是从对应主机上的哪个进程发出的，发送给哪个进程。

- **16位数据报长度**：标志UDP首部与发送数据的长度之和，大小为2^16，即64K，65535。

- **16位校验和**：用于接收端检验接收的数据与发送的数据是否一致，不一致则丢弃。

  > 校验方法：二进制反码求和，即对报文从头开始的每个字节进行取反相加，高出16位则截断高位，与低16位相加，得到校验和。

- 如何解包（分离）：UDP采用固定长度报头，接收方将报文前8字节提取出，剩下的就是有效载荷。

- 如何向上交付：接收方的OS的传输层收到UDP报文之后，16位目的端口号标明了对应进程。

  > 该进程bind了端口号，在内核中，存储诸如port : PCB指针这样的KV类型，就可以通过端口号找到对应的进程
  > 这也是为什么在应用层编写UDP代码时，定义端口号时，要定义为uint16_t，正是因为传输层UDP协议使用的端口号为16位的

- UDP如何提取到整个完整报文：16位UDP长度字段

> **理解UDP/TCP报文的本质**
>
> ![image-20231118200744855](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118200744855.png)
>
>
> 1. **UDP/TCP报头在操作系统中本质就是一个位段类型。**
> 2. OS中会有很多UDP报文，TCP报文，那么，OS需要管理这些报文，即先描述，再组织。所以报文在内核中并非仅位段 + 有效载荷。还会有其他属性字段。

## UDP的特点

UDP传输过程类似于寄信。

**无连接，不可靠，面向数据报**

- **无连接**: **即不需要建立连接也可以发送数据。**只需要知道对端的端口号和ip地址就可以直接传输。就比如寄快递，只需要知道人家的地址就可以送过去，不需要与别人提前建立联系。

- **不可靠**: **即不能保证数据安全, 有序的到达对端**。
  因为UDP不存在重传机制与确认机制，所以并不能保证数据能够到达对端，可能会在途中出现丢包，即使丢包了也不会报错和重传。
  UDP的报头中不存在序号，并且是无连接的，所以他没法保证数据到达后的顺序，需要我们自己在应用层进行包序管理。UDP协议层也不会给应用层返回任何错误信息，更不会重传等等;

- **面向数据报: 不能够灵活的控制读写数据的次数和数据的长度**
  应用层交给UDP多长的报文, UDP原样发送, 既不会拆分, 也不会合并;   用UDP传输100个字节的数据: 如果发送端调用一次sendto, 发送100个字节, 那么接收端也必须调用对应的一次recvfrom, 接收100个字节; 而不能循环调用10次recvfrom, 每次接收10个字节;  发送端的sendto和接收端的recvfrom次数是一样的。

**UDP数据报长度字段只有16位，报头要占8个字节，所以数据报的长度不能大于(64K- 8) 。**
我们注意到, UDP协议首部中有一个16位的最大长度. 也就是说一个UDP能传输的数据最大长度是64K(包含UDP首部8字节). 然而64K在当今的互联网环境下, 是一个非常小的数字. **如果我们需要传输的数据超过64K, 就需要在应用层手动的分包, 多次发送, 并在接收端手动拼装;**

UDP给应用层传下来的报文添加首部后就会直接转交给网络层，所以对于较大的数据，<u>需要我们自己在应用层进行分包和包序管理进行多次发送。</u>

**UDP是全双工的**

UDP没有发送缓冲区，有接收缓冲区，数据在网络中的发送和接收互不影响，可以同时进行，因此为全双工的。UDP的socket既能读，也能同时写。

**如何让UDP实现可靠传输**

如果光光依靠UDP本身是无法实现可靠传输的，因为其无法保证数据安全且有序到达，因此应当参考TCP的可靠性机制, 在应用层实现类似的逻辑。如

- 引入序列号，保证数据有序。
- 引入确认应答，确保对端收到了数据。
- 引入超时重传，如果一段时间内没有收到应答，就重发数据，保证数据不会丢失。

**UDP的缓冲区**

UDP没有真正意义上的发送缓冲区，调用sendto会直接交给内核, 由内核将数据进一步传给网络层协议进行后续的传输动作;

UDP具有接收缓冲区，但是因为UDP不可靠，没有任何传输控制行为。故这个接收缓冲区无法保证接收到的UDP报的顺序和发送UDP报的顺序一致。如果缓冲区满了，再到达的UDP数据就会被丢弃。

> **sendto/recvfrom/send/recv/write/read IO类接口**
>
> ![image-20231118201942760](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118201942760.png)
>
> 我们调用UDP的sendto/recvfrom和TCP的recv/send时，表面上是网络发送和网络接收函数，实质上，**它们只是拷贝函数，将应用层缓冲区的数据拷贝到发送缓冲区，将接收缓冲区中的数据拷贝到应用层缓冲区中（注意，这个过程中需要进行解包，看下方）。**（注意，UDP没有发送缓冲区，所以为虚线，若TCP则为实线。）
>
> 将数据拷贝到发送缓冲区之后，什么时候进行网络发送，发多少，出错了怎么办，这些都是由传输层协议决定的。缓冲区也是传输层提供的。（针对TCP）

## UDP报文解包（去报头）发生在哪个阶段

**UDP的接收缓冲区中存储的是完整的UDP报文，而不是去报头之后的有效载荷！！！**

UDP报文的解包（去掉UDP报头）通常发生在传输层协议栈内核处理接收到的数据时。这个过程通常包括以下几个阶段：

1. **数据接收：** 当操作系统内核的UDP接收缓冲区接收到UDP数据报时，操作系统会将完整的UDP数据报（包括UDP报头和数据部分）存储在接收缓冲区中。
2. **协议栈处理：** 接下来，操作系统的协议栈会介入处理这个接收到的UDP数据报。<u>协议栈是操作系统内置的网络协议处理程序</u>，负责在不同层次上处理网络数据。在传输层，UDP协议栈会负责处理UDP数据。
3. **解析报头：** 在协议栈中，UDP协议会解析UDP数据报的报头部分。这包括解析源端口、目标端口、长度和校验和等信息。操作系统可能会校验校验和以确保数据的完整性。
4. **去掉报头：** 一旦UDP协议栈解析了UDP报头并完成校验，它会将UDP报头从数据中去掉，只留下数据部分。<u>这个阶段实际上就是你所描述的解包过程。</u>解包后的数据将会被传递给操作系统内核的上层应用程序(根据目的端口字段)，这样应用程序就能够获取到不包含UDP报头的原始数据。
5. **传递给应用层：** 解包后的数据被传递给应用层，通常通过调用类似于 `recvfrom` 函数来获取。应用层可以进一步处理这些数据，根据需要解析、处理或者将其显示给用户。

总之，解包UDP报文的过程发生在操作系统协议栈的传输层处理阶段，它涉及解析UDP报头以及去掉报头部分，使应用程序能够获取到不包含UDP报头的原始数据。

是的，你理解得很准确。**在进程没有调用 `recvfrom` 函数的情况下，接收到的UDP报文会存储在操作系统内核的UDP接收缓冲区中等待处理。当你调用 `recvfrom` 函数时，操作系统的协议栈会触发解析UDP数据报的过程，然后将解析后的数据传递给应用程序。**这个过程包括解析UDP报头、去掉报头部分以及将数据交付给应用层，使得应用程序能够获取到不包含UDP报头的原始数据。**这样的机制允许应用程序根据需要控制数据的接收时机，并在需要时处理和使用接收到的数据。**

## 基于UDP的知名应用层协议

- NFS: 网络文件系统
- TFTP: 简单文件传输协议
- DHCP: 动态主机配置协议
- BOOTP: 启动协议(用于无盘设备启动)
- DNS: 域名解析协议

DNS DHCP

**UDP协议实现简单聊天室（client+server）**

[zzz](https://github.com/DaysOfExperience/Linux_Network_Programming/tree/main/UDP)

> 上方是使用UDP协议实现的一个简单的聊天室的server和client。
>
> 服务端：socket创建套接字，bind绑定ip和端口，然后开始recvfrom，业务处理，sendto（因为UDP无连接，不可靠） 这里的业务处理很简单，存在一个std::unordered_map<std::string, struct sockaddr_in> _map; // [ip:port] | struct，记录当前聊天室的所有客户端，value是这个客户端对应的sockaddr_in，将来用于sendto。将每个客户端发来的数据和它的ip:port字符串进行拼接，发送给这个unordered_map中的所有客户端。在客户端第一次发信息过来时，会自动加入聊天室，也就是进入unordered_map中
>
> 客户端：socket创建套接字，不需要调用bind，创建两个线程，一个收取网络数据，另一个获取用户输入，发送网络数据。接收线程只需要套接字，发送线程需要套接字，server_ip，server_port（用于sendto）
>
> 因为接收线程和发送线程的打印信息存在混乱的情况。所以将接收用户输入的线程的打印信息设为cerr，也就是往标准错误中输出。而接收服务端发送的网络数据的线程的输出信息定为cout。再通过mkfifo创建管道文件，将客户端的标准输出重定向到管道文件中。解决输出信息混乱的问题。

# TCP协议

TCP全称为 "传输控制协议(Transmission Control Protocol")

## TCP协议格式

![image-20231118203227323](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118203227323.png)

- **16位源/目的端口号：表示数据从哪个进程来，到哪个进程去。**

- **16位校验和**：用于检验接收的数据与发送的数据是否一致，不一致则丢弃。

- **16位紧急指针**：标识哪部分数据是紧急数据, 这段数据的优先级更高，会提前传输

- **选项**：暂不考虑

- **数据**：就是传输层报文携带的有效载荷，在TCP/IP网络通信过程中，一般为应用层报文

- **16位窗口大小**: 用于实现滑动窗口机制，来进行流量控制

- **4位首部长度字段**：表示该TCP报头有多少个32bit（4字节），TCP报头不是定长的，而是变长的，因为选项内容不定。所以，TCP头部最大为15\*4=60字节。即20~60字节。

  > TCP报文如何进行解包（分离）：解包的大致过程为：提取20字节，获取4位首部长度（在20字节中的位置固定），x\*4-20为选项的长度。提取出x*4-20字节后，剩下的就是有效载荷（也就是应该交给应用层的数据）

- **保留位（6bit位）**：该字段的位设置为零，这些位保留，供以后扩用。

- **6位标志位**：用于描述报文的性质/属性, TCP报文有多种类型（属性），这6个标志位是用于标记报文类型的，比如ACK标志位若为1，则代表这个报文有ACK属性，即表示确认序号有效（见下方确认应答（ACK）机制）（多个标志位可叠加，一个报文可有多种属性类型）
  **URG: 紧急指针是否有效**（TCP因为有序号，故数据是按序到达的，URG配合16位紧急指针可以实现将有效载荷中的某紧急数据提前向上交付）
  **ACK: 确认序号是否有效**（凡是该报文具有应答特征，该标志位都会被设置为1。大部分网络报文ACK都是被设置为1的，因为TCP有捎带应答机制。但是第一个TCP连接请求报文的ACK标志位不为1）
  **PSH**: 提示接收端应用程序立刻从TCP缓冲区把数据读走（联系流量控制和滑动窗口机制）
  **RST**: 对方要求重新建立连接; 我们把携带RST标识的称为**复位报文段**（比如一端网线断了，则两端对于连接建立认知不一致）
  **SYN**: 请求建立连接; 我们把携带SYN标识的称为**同步报文段**
  **FIN**: 通知对方, 本端要关闭了, 我们称携带FIN标识的为**结束报文段**

## TCP的特点

其主要特点为：面向连接，可靠，面向字节流。

- **面向连接**：通信必须在建立连接之后，通过连接管理机制实现
- **可靠传输**:  保证数据安全, 有序的到达对端
- **面向字节流**：以字节流的方式传输

**TCP全双工，具有发送缓冲区和接收缓冲区**

![image-20231118204507829](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118204507829.png)

**TCP全双工，client和server都有发送缓冲区与接收缓冲区。**

我们编写TCP代码时，使用的send，recv方法，其实只是拷贝函数，而不是网络收发函数。send的作用是把应用层的缓冲区数据拷贝到发送缓冲区，recv的作用是把接收缓冲区中的数据拷贝到应用层缓冲区。

而拷贝到发送缓冲区之后，什么时候发，发多少，丢包出错了怎么办，这都是TCP协议需要管的，所以称之为传输控制协议！（好兄弟，TCP是面向字节流的）

这两对缓冲区之间的数据流动互不影响，所以TCP是全双工的！

## 连接管理机制

**TCP通过首部的标志位来实现连接的管理，完成通信。  : SYN FIN ACK**

TCP是面向连接的, 数据传输之前要经过三次握手建立连接，四次挥手断开连接。

> **如何理解连接**
>
> 对于一个server来说，可能会有很多client连接server，所以server端一定会存在大量的连接。**OS需要管理这些连接！则需要先描述，再组织。**所以所谓的连接，本质就是操作系统内核中的一种数据结构类型。当建立连接成功时，就是在内存中创建对应的连接对象。再对多个连接对象进行某种数据结构的组织，方便管理。

### TCP建立连接的三次握手

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231118204729273.png" alt="image-20231118204729273" style="zoom:67%;" />


1. TCP三次握手的过程中，并非是传输SYN，ACK...而是传输TCP报文，这个报文的SYN，ACK标志位为1

2. 可以将上图的纵轴理解为时间线，TCP三次握手的过程需要时间消耗，每次报文的传输也需要时间消耗。

3. 三次握手并不是一定成功，只是较大概率成功，**TCP的可靠性是在三次握手建立连接完成之后才能保证的。**主要原因是第三次握手时，客户端发来的ACK报文可能丢包。

4. 客户端和服务端在三次握手的进行过程中，会有状态变化。比较值得关注的是，客户端在发送完第三次握手的ACK之后，进入ESTABLISED状态，而服务端接收到ACK才会认为连接建立完成。具有滞后性。

5. > 简单点来说：
   > 第一次握手：客户端：在吗，我是XX，你听到我说话了吗？
   > 第二次握手：服务端：我听到你说话了，你听到我说话了吗？
   > 第三次握手：客户端：我也听到你说话了，既然我们互相都能听到，就开始通信吧。

### TCP断开连接的四次挥手

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119135043375.png" alt="image-20231119135043375" style="zoom:67%;" />


1. 主动断开连接的一方先发送FIN报文：FIN，ACK，FIN，ACK

2. 被动断开连接的一方可能将第二次挥手的ACK和第三次挥手的FIN合并为一个报文。**四次挥手可能为三次挥手。**

3. 四次挥手不一定顺利完成，比如第二次ACK丢失，最后一次ACK丢失。但是因为TCP有超时重传机制，所以整体来说不影响。

4. > 简单来说
   > 第一次挥手：客户端：我要说的话已经说完了。(从此，客户端关闭写操作，不再发数据，但是可以收数据)
   > 第二次挥手：服务端：你要说的我都听到了，但是我的话还没说完。（从此，服务端关闭读操作）
   > 第三次挥手：服务端：我要说的话也说完了。（第三次挥手前，服务端要发的数据都发完了，关闭写操作，进行第三次挥手）
   > 第四次挥手：客户端：既然我们都说完了，那就结束通话吧。

### 保活机制

在TCP通信中，如果两端长时间没有数据往来（默认7200秒），则每隔一段时间（默认75秒），服务端就会向客户端发送一个保活探测数据报，让客户端进行回复，如果多次（默认9次）没有收到响应，则代表连接已经断开。

**通过保活机制来确保如果有一端断开，另一端能够及时处理。**

### 三握四挥的状态变化

#### 服务端状态转化

三次握手:

[CLOSED -> **LISTEN**] 服务器端调用listen后进入LISTEN状态, 等待客户端连接;
[**LISTEN** -> **SYN_RCVD**] 一旦监听到连接请求(同步报文段SYN), 就将该连接放入内核等待队列中, 并向客户端发送SYN确认报文.
[**SYN_RCVD** -> ESTABLISHED] 服务端一旦收到客户端的确认报文, 就进入ESTABLISHED状态, 可以进行读写数据了.

四次挥手:

[ESTABLISHED -> **CLOSE_WAIT**] 当客户端主动关闭连接(调用close), 服务器会收到结束报文段, 服务器返回确认报文段并进入CLOSE_WAIT;
[**CLOSE_WAIT** -> **LAST_ACK**] 进入CLOSE_WAIT后说明服务器准备关闭连接(需要处理完之前的数据); 当服务器真正调用close关闭连接时, 会向客户端发送FIN, 此时服务器进入LAST_ACK状态, 等待最后一个 ACK到来(这个ACK是客户端确认收到了FIN)
[**LAST_ACK** -> CLOSED] 服务器收到了对FIN的ACK, 彻底关闭连接.

#### 客户端状态转化:

三次握手:

[CLOSED -> **SYN_SENT**] 客户端调用connect, 发送同步报文段;SYN
[**SYN_SENT** -> ESTABLISHED] connect调用成功（返回）, 则进入ESTABLISHED状态, 开始读写数据; 

四次挥手:

[ESTABLISHED -> **FIN_WAIT_1**] 客户端主动调用close时, 向服务器发送结束报文段, 同时进入 FIN_WAIT_1;
[**FIN_WAIT_1** -> **FIN_WAIT_2**] 客户端收到服务器对结束报文段的确认, 则进入FIN_WAIT_2, 开始等待服务器的结束报文段FIN;
[**FIN_WAIT_2** -> **TIME_WAIT**] 客户端收到服务器发来的结束报文段, 进入TIME_WAIT, 并发出LAST_ACK;
[**TIME_WAIT** -> CLOSED] 客户端要等待一个2MSL(Max Segment Life, 报文最大生存时间)的时间, 才会进入CLOSED状态.

其中比较关键的两个状态是：

1. TCP被动关闭方收到对方的FIN，并发出ACK之后，在发出第三次挥手的FIN之前，会进入CLOSE_WAIT状态。

2. 主动关闭方在收到对方第三次挥手的FIN，并发出ACK之后，会进入一段TIME_WAIT状态，而不是直接进入CLOSED状态。

### 问题补充

**TCP握手为什么三次，能不能两次或者四次**

两次不安全，四次没必要。SYN的目的是为了确认对方是否具有收发数据的能力，当得到ACK回复后，则证明对方收到数据并且当前在线。如果要建立连接，就必须确保双方都具有收发数据的能力并且当前处于在线状态。

**TCP三次握手为什么不能两次？**
如果只有两次就能建立连接，那就代表着客户端发起SYN连接，服务端确认后回复ACK+SYN就直接建立连接。

1. **如果客户端连续发送多次请求，服务端就会建立多次连接，严重的浪费资源。**
2. 如果客户端发起SYN请求后就断开，或者是因为延迟很久才发到，等服务端收到时客户端已经断开连接，这时这个连接是失败的，但是服务端还是创建了一个毫无意义的套接字，严重浪费了资源。

3. 三次握手可以验证全双工，验证通信信道。

**TCP三次握手为什么不能四次？**
四次握手完全没有必要，建立连接的SYN和确认回复的ACK报文是可以一起发送的，没有必要分开来增加操作。

**TCP挥手为什么要四次，三次可以吗？**

首先，因为断开连接是建立TCP连接双方的事情，需要双方都关闭此方到彼方的传输信道，**故每一方都需要发送一次FIN报文。**

那么，第二次和第三次挥手可以合并吗？

**不行，发送FIN包只能代表着主动关闭方不再发送数据，但不代表着不再接收数据，所以被动关闭方在回复ACK后还是有可能继续发送数据，等到被动关闭方所有数据发送完后才会发送FIN，主动关闭方收到后回复ACK才会断开连接。**（如果被动关闭方不需要发送数据，则此时可以三次挥手）

**TCP三次握手失败时，服务端如何处理？**
1.如果服务端没有收到SYN，则什么都不做（因为压根没有建立起来连接，出现情况可能是SYN丢失）
2.服务端发送了SYN和ACK后，没有收到客户端的ACK，此时则说明客户端可能不在线，此时发送一个RST重置连接，并且释放已有资源。

**四次挥手的CLOSE_WAIT状态**

大多情况下，TCP连接的断开，都是由客户端发起的。则若服务端在收到客户端的FIN并发出ACK之后，不发出FIN，则客户端会一直进入CLOSE_WAIT状态。**而发出第三次挥手的FIN，就需要服务端主动close关闭文件描述符。**

故，服务端需要注意，当TCP通信结束后，需要close对应的文件描述符。**否则，这个TCP连接将不会释放，时间长了，越来越多的TCP连接占用内存资源，会引发问题。同时，服务进程的文件描述符也会越来越少。**

所以，若发现服务器有大量的close_wait状态的连接存在时，原因是什么呢？即应用层服务器写的有bug，忘记关闭对应的连接sockfd，导致四次挥手没有正确完成。

**四次挥手的TIME_WAIT状态**

主动断开连接的一方，收到对方第三次挥手的FIN，且发出ACK之后，会进入TIME_WAIT状态，而不是直接进入CLOSED状态。经验证，确实是！

**在此状态下，虽然应用程序终止了，但TCP协议层的TCP连接并没有完全断开，地址信息ip, port依旧是被占用的。**
故，有一个现象：当服务端直接ctrl+c之后，因为文件描述符表的生命周期随进程！故，进程终止，相当于所有文件描述符被关闭，相当于服务端主动进行四次挥手断开连接。故TCP协议层的连接此时会进入TIME_WAIT状态，故此时ip, port是被占用的，故此时若立刻再次启动服务进程，会显示bind失败。

![image-20231119140807011](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119140807011.png)

1. TCP协议规定,主动关闭连接的一方要处于TIME_ WAIT状态, 等待两个MSL(maximum segment lifetime) 的时间后才能进入到CLOSED状态;
2. 我们使用Ctrl-C终止了server, 所以server是主动关闭连接的一方, 在TIME_WAIT期间仍然不能再次监听同样的server端口;
3. MSL在RFC1122中规定为两分钟,但是各操作系统的实现不同, 在Centos7上默认配置的值是60s;

**为什么要有TIME_WAIT状态？（且还是2MSL）**

1. **MSL是TCP报文的最大生存时间, 因此TIME_WAIT持续存在2MSL的话, 就能保证在两个传输方向上的尚未被接收或迟到的报文段都已经消失**(否则服务器立刻重启, 可能会收到来自上一个进程的迟到的数据, 但是这种数据很可能是错误的);
2. **保证最后一个ACK成功传输给对端。**（若传输失败，则对端可能会进行超时重传，重新传来一个FIN，这时虽然客户端不存在了，但是TCP连接还在，仍然可以重发LAST_ACK;

**解决TIME_WAIT状态引起的bind失败的方法**

在server的TCP连接没有完全断开之前不允许重新监听, 某些情况下可能是不合理的。

> 服务器需要处理非常大量的客户端的连接(每个连接的生存时间可能很短, 但是每秒都有很大数量的客户端来请求).  
>
> 这个时候如果由服务器端主动关闭连接(比如某些客户端不活跃, 就需要被服务器端主动清理掉), 就会产生大量TIME_WAIT连接. 
>
> 由于我们的请求量很大, 就可能导致TIME_WAIT的连接数很多, 每个连接都会占用一个通信五元组(源ip,  源端口, 目的ip, 目的端口, 协议). 其中服务器的ip和端口和协议是固定的. 如果新来的客户端连接的ip和端口号和TIME_WAIT占用的链接重复了,就会出现问题。

int opt = 1; setsockopt(listensock, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt));  进行setsockopt即可让端口号和ip在TIME_WAIT期间，依旧被服务器再次绑定。

## 可靠传输

**TCP与UDP不一样，他能够确保数据安全, 有序的到达对端。**
通过以下方式

1. 面向连接
2. 确认应答机制（确保对端是否收到数据，并且根据序号使数据有序）
3. 超时重传机制（确保数据丢失后能够重传）
4. 流量控制机制
5. 拥塞控制机制
6. TCP首部报文中的序号和确认序号（确保数据有序）
7. TCP首部报文中的校验和(确保收发数据一致，不一致则要求重传)

### 确认应答（ACK）机制

> TCP和UDP一个最大的不同就是，TCP具有可靠性，而可靠性就需要TCP采取一些机制和策略措施来实现，其中最重要的就是确认应答（ACK）机制。

> 为什么网络传输存在不可靠性呢？（这里的不可靠，比如数据在传输过程中丢失了，丢包了。）其实，**单纯只是因为传输距离变长了**，

**TCP协议的确认应答机制：当A向B发送一条消息，我们无法保证，最新传输的这条消息的可靠性，但是如果B就这条消息，给A一个ACK（一个确认应答），则A一定可以保证B收到了这条消息。**

TCP在首部中给每一个发送的TCP报文都设置了序号和确认序号。
通过序号和确认序号，发送方告诉了接收方我发送的数据的序号，以及数据的长度。而接收方则回复发送方，我已经收到了哪些数据，你下一次应该从哪一个位置开始发送。

**seq**: **本条数据的起始序号。为上一条数据的ack**
**ack**: 对对方发送数据的确认序号，**告诉对方这个位置之前的所有数据已收到**。确认序号为本条数据的起始序号加上数据长度。也就是上一条的seq + len
**len**: **本条数据的长度。**

![image-20231119143019105](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119143019105.png)

![image-20231119143025282](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119143025282.png)

如果在传输过程中，前面的某一个数据包丢失，即使这里的1025-2048和2049-3072已经到达，但是这些数据依旧无法确认回复, **因为确认回复必须要保证确认序号ack前的所有数据都要到达**。**这么做的目的是为了防止因为确认回复ACK的丢失导致的重传。（见下方的作用）**

![image-20231119143246382](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119143246382.png)

即使是因为丢包或者延迟而导致前面的数据晚到或重传，等到接收到重传的数据后，再一个个对之前发送的数据进行确认应答，并通过序号在接受缓冲区中进行排序。

---

确认序号的含义是：表示确认序号对应的数字，之前的所有数据报文已经全部都收到了。告知发送方下次从确认序号指明的序号开始发送。
比如，接收方发送ACK为1001，2001，3001三个ACK报文，此时若1001和2001ACK丢失了，3001ACK接收方接收到了，这种情况并不影响，因为根据确认序号的含义，表示之前的包含1-3000的数据的三个报文，接收方都收到了。

**序号和确认序号的作用**

1. 将请求和应答一一对应。（接受方知道哪个ACK对应哪个之前发送过的数据报文）
2. 允许部分ACK丢失或者不给应答。因为确认序号的含义是确认序号之前的所有数据都收到了。
3. 接收方可以根据序号，将接收到的报文进行排序，解决报文乱序问题。(确保数据有序)
4. 接收方可以根据TCP协议报头中的序列号，很容易进行去重。（超时重传ACK丢失）

一个报文中设置序号和确认序号两个字段是因为，TCP是全双工的，任何一方在发送ACK确认的时候，也可能同时想向对方发送数据消息。既可以收，也可以发。（只需要填写确认序号字段，且将ACK标志位置为1即可）

### 超时重传

> 超时重传机制是TCP协议保证可靠性的一个关键因素。个人认为，确保TCP协议可靠性最关键的就是两个，一个是确认应答ACK机制，另一个就是超时重传机制。有了这两个机制，几乎就可以保证数据可靠地传输给对方。

**在通信时，某一主机在给另一个主机发送数据后，可能会因为延迟或者丢包等网络原因，导致数据不能到达，为了确保数据能够到达对端，TCP加入了超时重传机制，当发送端在特定的时间内没有收到来自接收端的确认应答时，就会重新发送**

> 在确认应答机制的前提下，当A向B发送报文，收到对应的ACK后，可以确保报文传达给了B。**而当A在一定时间内没有收到B的ACK时，则判定为出问题了，则A重新给B发送报文。**

可能的情况：

1. 主机A发送数据给B之后, 可能因为网络拥堵等原因, 数据报无法到达主机B，丢包了。
2. 报文没有丢包，B发送的ACK丢失了。

则不论情况一还是情况二，A都需要在一定时间没有收到ACK之后，重新发送报文。但如果是情况二，则主机B会收到很多重复数据（报文），则TCP协议就需要能够识别出哪些包是重复的包，并将重复的丢弃掉。**接收方可以根据TCP协议报头中的序列号，很容易进行去重。**

**超时的时间如何确定？**

这个时间长短，随着网络环境的不同，是有差异的。网络环境好，则时间应相对短一些，网络环境差，则时间应相对长一些。
如果时间设得太长，会影响整体的重传效率。如果时间设的太短，有可能会频繁发送重复的包(误判)。也会影响整体传输效率。

**TCP为了保证无论在任何环境下都能比较高性能地通信, 因此会动态计算这个最大超时时间**. 

Linux中(BSD Unix和Windows也是如此), 超时以500ms为一个单位进行控制, 每次判定超时重发的超时时间都是500ms的整数倍. 如果重发一次之后, 仍然得不到应答, 等待 2\*500ms 后再进行重传. 如果仍然得不到应答, 等待 4*500ms 进行重传. 依次类推, 以指数形式递增. 累计到一定的重传次数, TCP认为网络或者对端主机出现异常, 强制关闭连接.

> **避免丢包重传**  👇  也可以理解为可靠性吧~

### 流量控制（16位窗口大小）

接收端处理数据的速度是有限的. 如果发送端发的太快, 导致接收端的缓冲区被打满, 这个时候如果发送端继续发送, 就会造成丢包, 继而引起丢包重传等等一系列连锁反应.**因此TCP支持根据接收端的处理能力（其实就是接收端的接收缓冲区的剩余空间大小）, 来决定发送端的发送速度. 这个机制就叫做流量控制(Flow Control);**

![image-20231119144142506](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119144142506.png)

> 上图中 主机B的返回的第一个数值是确认序号, 第二个数字就是16位窗口大小

1. **在TCP头部的窗口字段中存储的是接收方接收缓冲区的剩余空间大小，而不是滑动窗口的大小。**这个字段被称为“窗口大小”或“接收窗口”（Window Size or Receive Window），它表示接收方当前可用于接收数据的缓冲区大小。接收端可以通过ACK报文通知发送端
2. 窗口大小字段越大, 说明网络的吞吐量越高;
3. 接收端一旦发现自己的缓冲区快满了, 就会将窗口大小设置成一个更小的值通知给发送端; 发送端接受到这个更小的窗口大小值之后, 就会减慢自己的发送速度;
4. 如果接收端缓冲区满了, 就会将窗口置为0; 这时发送方不再发送数据, 但是需要定期发送一个窗口探测数据段（可以理解为不携带有效载荷的报文）, 使接收端把窗口大小告诉发送端。
5. TCP首部中的16位窗口大小字段，就是存放了窗口大小信息。16位最大表示65535字节。实际上，TCP首部40字节选项（最大40字节）中还包含了一个窗口扩大因子M, 实际窗口大小是窗口字段的值左移 M 位;
6. 当接收端的ACK报文窗口大小总是很小或总是为0时，发送端发送的报文中可以将PSH标志位置为1，表示督促对方尽快将接收缓冲区中的数据向上交付。

7. 流量控制是TCP连接双方的，因为TCP协议的每一方都有一个发送缓冲区和一个接收缓冲区。
8. 在TCP连接双方第一次进行携带有有效载荷的报文进行通信时，如何得知对方的窗口大小？其实，第一次数据通信 不等于 第一次交换报文。在TCP三次握手时，需要传输报文，这时就可以填写窗口大小字段，交换双方的接收缓冲区剩余空间大小。

### 拥塞控制

> 丢包问题可能分为两种，1. 少量丢包，则超时重传 / 快重传 2. **大量丢包，则可能是因为此时网络拥塞，则此时不该采取重传策略，而是先给网络缓冲的机会。**（注意，网络中的TCP/IP连接是很多的）

**对于这种避免网络拥塞，以及网络拥塞之后的应对策略，称为拥塞控制。**

**拥塞控制：以一种慢启动，快增长的传输方式，根据网络状态调整发送速度的机制**

- 拥塞窗口：单机主机一次向网络中发送大量的数据时，可能会引发网络拥塞的上限值。

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119144806000.png" alt="image-20231119144806000" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119144754670.png" alt="image-20231119144754670" style="zoom:67%;" />

**慢启动**：设置拥塞窗口大小为1，之后每次收到一个确认回复ACK，就将拥塞窗口的大小*2, 拥塞窗口大小指数增长。当达到慢启动的阈值时，就会使用拥塞避免算法。

**拥塞避免**：当达到慢启动阈值的时候，拥塞窗口会由指数增长转换为线性增长，即每次收到一个确认回复ACK，就让窗口大小+1, 这个操作称为拥塞避免。
当又出现网络拥塞的情况时(大量丢包/超时)，调整拥塞窗口阈值为当前拥塞窗口大小的一半，接着将当前窗口大小设置为1，重新进入慢启动。

**快重传**：当连续收到三次重复确认应答时，他此时就会对确认序号ack位置的数据进行重传，因为发送三次确认回复的时间比起等待超时要少了很多，所以这种机制也被称为快速重传机制。

**快恢复**：当出现了快重传的情况时，就说明当前网络状况出现了问题，但是又由于我们能够连续三次收到确认应答，就说明了当前的问题并不是很严重，没有必要重新进行慢启动。
所以当前会将拥塞窗口的阈值设置为当前窗口大小的一半，并直接将窗口大小设置为阈值，直接开始线性增长进行拥塞避免。由于跳过了慢启动阶段直接进行拥塞避免，因此被称为快恢复

> 上面介绍了TCP拥塞控制的几种情况, 包括慢启动, 拥塞避免, 快重传, 快重传之下的快恢复

在TCP连接刚建立好时，如果在刚开始阶段就发送大量的数据, 仍然可能引发问题，因为此时网络状况并不清楚，因此TCP引入 慢启动 机制，先发少量数据，探探路，摸清当先的网络拥堵状况，再逐渐增大数据传输速度。**慢启动机制除了发生在每一次发生网络拥塞之后, 还会在TCP刚建立好时使用**

- "慢启动" 只是指初始时慢(开始时拥塞窗口大小为1), 但是增长速度非常快, 因为是指数增长

- 不能一直指数增长, 故引入一个拥塞窗口的阈值，超过阈值之后线性增长, 这个称之为拥塞避免

- 为什么网络拥塞之后，前期是指数增长？指数：前期较慢，后期增长较快。前期给网络一个缓冲的机会 - 慢, 中后期，网络恢复之后，需要尽快恢复通信的效率 - 快。 即**慢启动 + 快恢复** (快恢复的第二层含义)

拥塞控制, 归根结底是TCP协议想尽可能快的把数据传输给对方, 但是又要避免给网络造成太大压力的折中方案.

## 挽救传输性能

### 滑动窗口

因为有了确认应答机制，于是对于每一个发出的数据报文，都需要一个ACK确认应答。收到ACK之后再发送下一个数据段。但这样做会导致TCP网络通信性能较差，尤其是数据往返的时间较长时。   (滑动窗口的引出) **既然这样一收一发的效率过慢，那么一次性发送多条，就可以解决这种问题，但是如果发送的数量过多，就可能会导致缓冲区满而出现丢包，为了控制发送数据的规模，于是TCP就引入了滑动窗口机制。**

滑动窗口用来指定发送方能够发送的字节数量，**从而影响流量控制和拥塞控制。**

> 滑动窗口策略就是用来提高TCP传输效率的。

![image-20231119155705787](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119155705787.png)

![image-20231119155725209](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119155725209.png)

- 滑动窗口的窗口大小指的是无需等待确认应答就可以继续发送数据的最大值.上图中的窗口大小即为4000字节。（TCP面向字节流，这4000字节被分为几个报文进行发送不确定，上图中为4个）发送前四个段时，不需要等待任何ACK，可以直接发送。收到第一个ACK后，滑动窗口向右滑动。继续发送第五个段的数据；以此类推。

- **窗口越大，表示网络的吞吐率越高（传输效率越高）**
- 操作系统内核为了维护这个滑动窗口, 需要开辟 发送缓冲区 来记录当前还有哪些数据没有应答; 只有确认应答过的数据, 才能从缓冲区删掉;

> 如图一所示，发送缓冲区中的数据可以大致分为三个部分，已发送并收到ACK确认的数据，已发送但未收到ACK确认的数据，未发送的数据。其中，窗口包括第二部分，可能包括未发送数据的某一部分，也就是，已发送但未收到ACK的数据，收到ACK之后，窗口会右移（后移），此时窗口可能包括未发送的数据的一部分，也就是不需要等待前方报文的ACK就可以立即发送了，但尚未发送。
>
> 滑动窗口的发送策略，其实并不是完全按照一批一批发送的，也就是，图二中，若收到了1001的ACK，则窗口会右移，此时，4001-5000的数据报文就可以立即发送了。

- 滑动窗口在发送缓冲区内，属于发送方的发送缓冲区的一部分，滑动窗口的本质：发送方，可以一次性向对方发送数据的最大值（滑动窗口，提高效率的一个策略）。滑动窗口的大小 = 接收方接收缓冲区的剩余空间大小（16位窗口大小字段） 和 拥塞窗口的较小值。

- 滑动窗口模型理解：可以将发送缓冲区理解为一个字节数组，每一个字节都有下标。滑动窗口有两个下标:win_start, win_end。win_end = win_start + 滑动窗口大小，若收到ACK，则win_start = ACK的确认序号，win_end = win_start + 滑动窗口大小。**故，滑动窗口的本质就是接收缓冲区中的指针或者下标。**

- 滑动窗口不一定必须向右移动，比如收到ACK且min(x, y)减小，则可能不会右移。
  滑动窗口可能为0，比如对方窗口大小 == 0 或者 拥塞窗口大小 == 0。
  中间发送的某些报文的ACK丢失，可能，但是不影响。因为ACK的确认序号的含义是该序号前的数据都收到了。
  滑动窗口一直右移，可能越界吗？不可能，因为发送缓冲区的物理上是线性的，逻辑上是环形的。

### 快重传

![image-20231119175558232](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119175558232.png)

之前说了丢包问题的一种可能，即中间或者开头的报文的ACK丢失，但是根据确认序号的含义，所以不影响。

另一种可能为开头的报文丢失，则后面收到的所有ACK的确认序号都会为1001。就像是在提醒发送端 "我想要的是 1001"  一样;
如果发送端主机连续三次收到了同样一个 "1001" 这样的应答, 就会将对应的数据 1001 - 2000 重新发送;
这个时候接收端收到了 1001 之后, 再次返回的ACK就是7001了(因为2001 - 7000)接收端其实之前就已经收到了, 被放到了接收端操作系统内核的接收缓冲区中;

这样的方法比之前的超时重传要快，因为发送三次ACK的时间比起等待超时要少了很多，所以这种机制也被称为快速重传机制。

这种机制被称为 "高速重发控制"(也叫 "快重传").    （快重传与超时重传是协作关系）

> 这个在拥塞控制中也用到了, 快重传之后, 会配合拥塞控制的快恢复机制

为什么是三次?

三次提供了一个缓冲的时间，可以适当避免因为网络延迟而导致的数据延迟到达。

三次其实也是相当于一个确认，如果在第一次或者第二次后收到了该数据，则就不会再发送后面几条，发送端也不需要重传。

> 其实这两点都一样, 核心都是为了确定确实丢失了

### 延迟应答

![image-20231119180205541](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119180205541.png)

有了滑动窗口之后，接收方接收缓冲区剩余空间大小，也就是接收能力会直接影响滑动窗口大小, 进而影响网络吞吐量/TCP传输效率。那么当接收方接收到数据后，若立刻进行ACK，则此时窗口大小是比较小的。若接收方可以延迟一些时间再应答，给应用层一定时间，将接收缓冲区中的数据读取走，则会给发送方一个较大的窗口大小。窗口越大，网络吞吐量越大，传输效率越高。

**作用是: 在网络不拥塞的情况下尽量提高传输效率。**

> 假设接收端缓冲区为1M. 一次收到了500K的数据; 如果立刻应答, 返回的窗口就是500K;  但实际上可能处理端处理的速度很快, 10ms之内就把500K数据从缓冲区消费掉了; 在这种情况下, 接收端处理还远没有达到自己的极限, 即使窗口再放大一些, 也能处理过来; 如果接收端稍微等一会再应答, 比如等待200ms再应答, 那么这个时候返回的窗口大小就是1M;

也不是所有的包都可以延迟应答，比如，数量限制：每隔N个包就应答一次。时间限制：超过最大延迟时间就应答一次。具体的数量和超时时间, 依操作系统不同也有差异; 一般N取2, 超时时间取200ms;（肯定也不能太长，否则会触发超时重传）

### 捎带应答

![image-20231119180812076](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119180812076.png)

一种提高TCP传输效率的机制

**在即将要发送的数据头部中加上上一条接收到的数据的ACK**。

往往并不是只有一方向另一方发送数据。而是双方互发数据。所以有时候，ACK可以和"接收方"想向"发送方"发送的数据报文合并，一起回给发送方。

## TCP面向字节流

对于TCP来说，每当创建一个socket的时候，就会同时在内核中创建一个接收缓冲区和发送缓冲区。

> 注意, 下方的数据指的是应用层报文数据, 也就是应用层调用send发送的数据

**接收**：对于接收到的数据，并非像UDP一样一条一条往上交付，而是先将接收到的数据放入缓冲区，再根据上层所需要的长度，从接收缓冲区中取出相应的数据交付。
**发送**：对于发送的数据并不会直接发送，而是先存入发送缓冲区。如果数据过大，则会被拆分成多个TCP数据包发送。而如果数据过小，则会在发送缓冲区中等待，直到大小合适后再从发送缓冲区中取出数据发送。

**优点**：**灵活**
这种传输方式比较灵活，对于多个小数据，会合并为一条大的数据一次性发送过去，**这样就大大的减少了IO的次数**。接收方也更加灵活，他可以任意取出想要的数据，不会像UDP一样必须交付一条完整的报文(应用层报文)。
**缺点**：**粘包问题**
因为数据可能会在缓冲区中进行合并或者拆分，这就导致了数据直接的边界无法控制，所以TCP交付的这条数据可能并非一条完整的数据(应用层报文数据)，而是半条或者多条数据，所以可能会导致上层会将多条数据按照一条来处理。这也就是TCP粘包问题。

## TCP粘包问题

即TCP可能将多条数据按照一条处理

**解决方案**：**需要我们自己进行边界的管理。**

1. 每条数据之间以特殊字符进行间隔（如果数据中有该字符可能要转义处理）
2. 数据定长传输，不够则补位（数据如果过短，则会传递大量无用的补位数据）
3. 应用层协议头部定义数据长度（这里可以参考http协议和udp协议的做法）

http: 头部以\r\n\r\n表示结束（空行间隔头部和正文），并且在头部的Content-Length确定正文长度。
udp: 传输层就解决了，在头部中就已经定义数据报长度。

针对TCP面向字节流-粘包问题的自定义应用层协议的网络版本计算器server&&client
http://t.csdn.cn/uMEuP

> 也就是使用UDP协议时, 调用一次recvfrom就对应一次sendto, 对应一次发送方发送的应用层报文
>
> 而TCP, 我们需要接收若干字节的数据, 却不能保证这是一整条应用层报文, 所以需要不断的接收, 然后进行应用层报文解析

![image-20231119183459914](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119183459914.png)

神. ChatGPT

## TCP异常情况

进程终止: 进程终止会释放文件描述符, TCP连接仍然可以发送FIN. 和正常关闭没有什么区别. 

机器重启: 和进程终止的情况相同. 

机器掉电/网线断开: 接收端认为连接还在, 一旦接收端有写入操作, 接收端发现连接已经不在了, 就会进行reset. 即使没有写入操作, TCP自己也内置了一个保活定时器, 会定期询问对方是否还在. 如果对方不在, 也会把连接释放. 

另外, 应用层的某些协议, 也有一些这样的检测机制. 例如HTTP长连接中, 也会定期检测对方的状态. 例如QQ, 在QQ断线之后, 也会定期尝试重新连接.

## 基于TCP应用层协议

HTTP HTTPS SSH Telnet FTP SMTP

当然, 也包括你自己写TCP程序时自定义的应用层协议;

## TCP/UDP对比

之前说过（好像说过），**TCP的可靠和UDP的不可靠是一个中性词**，没有优劣之分，这只是它们的特点。因为可靠，一定意味着复杂，不可靠就意味着简单。

而也不是说TCP的传输效率就一定低于UDP，因为TCP有很多提高传输效率的策略和机制，比如，滑动窗口，快重传，延迟应答，捎带应答等。

总的来说

- **TCP**用于**可靠传输**的情况, 应用于文件传输, 重要状态更新等场景。

- **UDP**用于对**高速传输和实时性要求较高**(因为UDP不用建立连接, 直接传)的通信领域, 例如, 早期的QQ, 视频传输等. 另外**UDP可以用于广播**。

具体TCP和UDP什么时机用，怎么用，还要根据具体的需求场景去判定。

## listen的第二个参数

```C++
#include <iostream>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include "Sock.hpp"

int main()
{
    Sock sock;
    int listen_sock = sock.Socket();
    sock.Bind(listen_sock, 8080);
    sock.Listen(listen_sock, 3);

    while (true)
    {
        sleep(5);
        std::string ip;
        uint16_t port;
        sock.Accept(listen_sock, &port, &ip);
    }
    return 0;

}
```

Linux内核协议栈为一个tcp连接管理维护两个队列：

- **半连接队列，被称为SYN队列**（用来保存处于SYN_SENT和SYN_RECV状态的请求）
- **全连接队列，被称为accept队列**（用来保存处于established状态，但是应用层没有调用accept取走的请求）

![image-20231119184018797](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119184018797.png)

**在服务端进行listen调用后，此时这个服务端就已经可以和客户端建立TCP连接了，即三次握手的过程。**
三次握手并非在服务端调用accept才能完成，**服务端调用accept只是获取已经建立好的TCP连接**，这些连接在OS内核TCP层维护的全连接队列中。

#### TCP全连接队列的意义

而如果同一时刻或者较短时间内大量客户端发起连接，此时服务端可能来不及提供服务（如创建新线程提供服务），此时就需要先进行连接，然后将此连接保存起来，之后服务端再提供对应服务。（联想海底捞门口的等待队列）

**全连接队列的长度 = listen第二个参数+1**

1、客户端发送SYN包，并进入SYN_SENT状态
2、服务端接收到数据包将相关信息放入半连接队列（SYN 队列）,并返回SYC+ACK包给客户端。
3、客户端返回ACK进行第三次握手，服务端接收客户端ACK数据包，这时如果全连接队列（accept 队列）没满，就会从半连接队列里面将对应连接数据取出来放入全连接队列，等待accept，当队列已满就会跟据tcp_abort_on_overflow配置执行策略。

![image-20231119184203250](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119184203250.png)

上图为，当listen的backlog(第二个参数)为3时，建立了四个TCP连接（三次握手完成，ESTABLISHED状态），此时再发起一个连接，会处于SYN_RECV状态，不会完成三次握手连接。 且这个连接一段时间后就会被自动终止

![image-20231119184250787](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119184250787.png)

**经过实验证实，服务端调用accept，会获取存储在全连接队列中的TCP连接，并将其移除全连接队列。**上图为当backlog == 3时，建立了6个TCP连接。

## 使用 wireshark 分析 TCP 通信流程

## 各种版本的基于TCP协议的cs

client的短连接版对应server的线程池版。client的长连接版对应server的多进程，多线程版。 

![image-20231119184335108](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119184335108.png)

上图为多进程2.0版本中，若server创建子进程之后，不进行close文件描述符，则会有大量CLOSE_WAIT状态连接存在。

![image-20231119184352418](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231119184352418.png)

上图为线程池版本中，每一次客户端都是以短连接的方式与服务端进行TCP连接。大概就是，先建立TCP连接，客户端send，服务端recv，进行业务处理，send结果，close文件描述符，客户端recv，打印输出，close文件描述符。但是因为往往是服务端先进行close，所以会有大量TIME_WAIT连接存在。而因为客户端端口号是由OS随机分配的，所以图中Foreign Address中的端口号都不一样。 

线程池版本中，若服务端每次不关闭短连接的文件描述符，则会发现，存在很多CLOSE_WAIT状态连接，而因为server和client都是在一个主机上运行的，所以同时会发现还有很多FIN_WAIT2状态的TCP连接，就是因为主动断开TCP连接的一方没有收到对方的第三次FIN挥手。