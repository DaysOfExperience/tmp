# 关于这个项目

仿muduo库   One Thread One Loop式   主从Reactor模型    实现⾼并发服务器

通过咱们实现的⾼并发服务器组件，可以简洁快速的完成⼀个⾼性能的服务器搭建。 并且，通过组件内提供的不同应⽤层协议⽀持，也可以快速完成⼀个⾼性能应⽤服务器的搭建（当前 为了便于项⽬的演⽰，项⽬中提供HTTP协议的⽀持）。

在这⾥，要明确的是咱们要实现的是⼀个⾼并发服务器组件，因此当前的项⽬中并不包含实际的业务内容。

## HTTP服务器

HTTP（Hyper Text Transfer Protocol），超⽂本传输协议是应⽤层协议，是⼀种简单的请求-响应协议（客⼾端根据⾃⼰的需要向服务器发送请求，服务器针对请求提供服务，完毕后通信结束）。

需要注意的是HTTP协议是⼀个运⾏在TCP协议之上的应⽤层协议，这⼀点本质上是告诉我们，HTTP服务器其实就是个TCP服务器，只不过在应⽤层  基于HTTP协议格式   进⾏数据的组织和解析   来明确客⼾端的请求并完成业务处理。

因此实现HTTP服务器，简单理解只需要以下⼏步即可

1. 搭建⼀个TCP服务器，接收客户端请求。（传输层）
2. 以HTTP协议格式进⾏解析请求数据，明确客⼾端⽬的。（应用层接收客户端的请求
3. 明确客⼾端请求⽬的后提供对应服务。（进行业务处理，获取响应）
4. 将服务结果用HTTP协议格式进⾏组织，发送给客⼾端（响应结果也是HTTP格式）

## Reactor模型

实现⼀个HTTP服务器很简单，但是实现⼀个⾼性能的服务器并不简单，这个单元中将讲解基于Reactor模式的⾼性能服务器实现。

### Reactor概念

Reactor 模式，是指通过⼀个或多个输⼊同时传递给服务器进⾏请求处理时的事件驱动处理模式。

服务端程序处理传⼊多路请求，并将它们同步分派给请求对应的处理线程，Reactor 模式也叫 Dispatcher 模式。

**简单理解就是使⽤ I/O多路复用统⼀监听事件，收到事件后分发给处理进程或线程**，是编写⾼性能⽹络服务器的必备技术之⼀。

### 分类

> 单Reactor单线程: 所有都在一个线程中完成, 包括listen套接字以及网络套接字的监听, 网络IO, 业务处理
>
> 单Reactor多线程: 多个子线程仅进行业务处理, 所有listen套接字的事件监听以及其余网络套接字的事件监听以及网络IO都是在主线程的epoll中完成的(注定了这个模式的缺点: 可能不及时处理新连接请求)
>
> 多Reactor多线程: 弥补上方的缺点, 所以. 其实多Reactor, 就像它的名字一样, 此时不再是一个epoll多路复用, 而是多个
> 一个主Reactor主线程只监听listen套接字, 获取新连接, 若干从Reactor从线程监听网络套接字, 其实就是从主线程那里获取的, 而网络套接字的IO事件监听以及业务处理都是在某从Reactor线程中完成的
>
> 这里面的单Reactor和多Reactor, 其实就是有一个epoll还是若个epoll= =

**单Reactor单线程：单I/O多路复⽤+业务处理**

1. 通过IO多路复⽤模型进⾏客⼾端请求监控

2. 触发事件后，进⾏事件处理

   a. 如果是新建连接请求，则获取新建连接，并添加⾄多路复⽤模型进⾏事件监控。

   b. 如果是数据通信请求，则进⾏对应数据处理（接收数据，处理数据，发送响应）。
   
   (也就是说, 这个线程中的这个多路复用会监听所有套接字的所有事件(包括listen套接字和常规TCP连接套接字))

优点：所有操作均在同⼀线程中完成，**思想流程较为简单**，
			**不涉及进程/线程间通信及资源争抢问题。**

缺点：**⽆法有效利⽤CPU多核资源，性能上限较低**（性能不高）

适⽤场景：适⽤于客⼾端数量较少，且业务处理速度较为快速的场景。（业务处理较复杂或活跃连接较多时，会导致单线程串⾏处理的情况下，后处理的连接⻓时间⽆法得到响应）。
简单来说就是，如果请求少，且业务逻辑简单，对性能要求不高，可以单Reactor单线程。如果请求连接多或者业务处理慢，则会有性能缺陷

**单Reactor多线程：单I/O多路复用+线程池（业务处理）**

1. Reactor线程通过I/O多路复⽤模型进⾏客⼾端请求监控

2. 触发事件后，进⾏事件处理

   a. 如果是新建连接请求，则Reactor线程进行accept获取新建连接，并添加⾄多路复⽤模型进行事件监控。

   b. 如果是数据通信请求，则Reactor线程读取数据后分发给Worker线程池进⾏业务处理。

   c. ⼯作线程处理完毕后，将响应交给Reactor线程进⾏数据响应

（worker线程池内的线程仅进行业务处理，新建连接请求以及网络通信时数据的接收和发送都是Reactor线程进行的）

优点：充分利⽤CPU多核资源

缺点：多线程间的数据共享访问控制较为复杂，**单个Reactor承担所有事件的监听和响应，在单线程中运行，高并发场景下容易成为性能瓶颈。**

![image-20230910131824343](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230910131824343.png)

![image-20230910131836485](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230910131836485.png)

（注：listen套接字描述符的事件也是由Reactor线程进行监控的，且listen套接字的事件处理：accept获取新连接也是Reactor进行的，上图没有标注）

为什么说**单Reactor多线程在高并发场景下容易造成性能瓶颈**呢？

答：单Reactor线程对所有连接（listen套接字以及所有TCP连接）的所有事件进行监控。listen套接字读事件触发就accept，其他连接读事件触发就读取数据，交给worker线程池内线程进行业务处理，然后由Reactor线程进行响应嘛。

这样一来，一个Reactor线程就要监控所有事件，要处理新连接请求还要处理其他连接的IO。高并发：一个时刻内或短暂时间内有大量连接请求。而单Reactor还要负责很多网络IO，**所以很可能来不及处理新的客户端的连接**。(其余套接字的事件处理完, 可能listen套接字已经等候多时了)

<u>解决方案：将新连接的获取和其余网络连接的IO处理分离开。也就是listen套接字的监听和其余套接字的监听这两个工作不能让一个Reactor线程来完成。这样就可以提高性能，更好地应对高并发场景。</u> 将新连接的处理单独分离出来。

**多Reactor多线程：多I/O多路复用+线程池（业务处理）**

基于单Reactor多线程的缺点：一个Reactor线程既进行新连接处理，又进行其余网络连接的IO。所以IO多的时候，新连接的获取/处理可能就不及时。

因此让一个**主Reactor线程**专门进行**新连接的监控**（监控listen套接字的读事件），有新连接就accept获取新连接。若干个**从Reactor线程**分摊其余全部TCP网络连接的IO监控，比如1000个新连接，3个从Reactor线程，就一个从Reactor线程监控300+个。仅**进行TCP网络连接的IO事件监控**，有新数据，就读取。

worker线程池进行业务处理，接收从Reactor线程传来的数据，业务处理，再交给从Reactor线程进行数据响应。

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230910133915820.png" alt="image-20230910133915820" style="zoom:67%;" />

主从Reactor模式

![image-20230910134255983](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20230910134255983.png)

这里的合理分配指的是：<u>业务处理耗时多就多分配几个业务线程。业务处理较为简单，就多分配几个从Reactor线程。</u>

1. 在主Reactor中处理新连接请求事件，有新连接到来则accept然后分发到从Reactor中进行后续的事件监控
2. 在⼦Reactor中进⾏客⼾端通信监控，有事件触发，则读取数据，将请求分发给Worker线程池进行业务处理
3. Worker线程池分配独⽴的线程进⾏具体的业务处理。
4. ⼯作线程处理完毕后，将响应交给⼦Reactor线程进⾏数据响应。

**优点：充分利⽤CPU多核资源，主从Reactor各司其职**

但是，因为线程也不是越多越好，一方面线程切换影响效率，另一方面还要加锁考虑线程安全。**所以主从Reactor的另一种模式是：将连接的IO事件监控和业务处理都放在从属Reactor线程中进行。**

### 目标定位：One Thread One Loop主从Reactor模型高并发服务器

咱们要实现的是**主从Reactor模型服务器**，也就是**主Reactor线程仅仅监听listen套接字，获取新建连接**，<u>保证获取新连接的⾼效性</u>，提⾼服务器的并发性能。

**主Reactor获取到新连接后分发给⼦Reactor进⾏通信事件监控。⽽⼦Reactor线程监控各⾃的描述符的读写事件进⾏数据读写以及业务处理。**

One Thread One Loop的思想就是把所有的操作都放到⼀个线程中进⾏，<u>⼀个线程对应⼀个事件处理的循环。</u>

**当前实现中，因为并不确定组件使用者的使用意向，因此并不提供业务层worker线程池的实现，只实现主从Reactor。**

### 功能模块划分

基于以上的理解，我们要实现的是⼀个带有协议⽀持的Reactor模型⾼性能服务器，因此将整个项⽬的实现划分为两个⼤的模块：

SERVER模块：实现Reactor模型的TCP服务器；

协议模块：对当前的Reactor模型服务器提供应⽤层协议⽀持。

# Server模块

实现Reactor模型的TCP服务器。

## Buffer模块/类

每一个客户端服务端建立的TCP连接的应用层缓冲区：接收缓冲区 / 发送缓冲区。因为TCP面向字节流，有粘包问题，我们得先把接收缓冲区中数据读到应用层再进行具体处理。

## Socket模块/类

一个数据成员：sockfd，然后提供了很多socket常见的相关方法的封装。

## Channel模块/类

事件管理类，包括：该文件描述符关心哪些事件、当前有哪些事件就绪、其中每个事件就绪时对应的回调函数。还有一个比较关键的HandleEvent方法，也就是在epoll_wait之后，该文件描述符某些事件就绪了，外层可以直接调用这个HandleEvent方法进行事件处理：很简单，就是看revents数据成员，然后调用对应的回调方法即可。当然，在外层epoll_wait之后，也会对其中的revents进行设定。使用的接口就是SetRevents。

每一个需要被epoll监控的文件描述符都有一个配对的Channel，不仅是listen套接字和TCP连接套接字的文件描述符。还有其他的，比如eventfd还有timerfd（详情看下方）



还有: 怎么启动读事件/写事件的关心呢? Channel类代表一个套接字, 他一定是和某一个EventLoop绑定的, 而_events是该套接字所关心的事件, 但是如何在对应的epoll中实际开启事件的关心呢? 调用方法, 而这个方法是在类外定义的, 在EventLoop那里再说吧

## Poller模块/类

就是对于epoll模型的一个最直接的封装，每一个Poller都是一个epoll模型。作为一个epoll模型，最需要，最基础的功能就是添加描述符事件监控，修改监控，删除描述符事件监控等等..当然这里的参数都是Channel对象作为参数，因为channel内部包含关心的事件，并且epoll在epollwait之后，也很方便的对channel中的就绪事件revents数据成员进行修改。（SetRevents）

除了增/删/改描述符的事件监控，作为一个epoll模型，少不了的就是epoll_wait，Poll成员方法就是epoll_wait的封装，并且会将当前有事件发生的Channel作为参数返回回去:std::vector<Channel *> * actives

然后上层可以直接对这个vector中的有事件待处理的连接（channel）调用其HandleEvent方法进行事件处理，就很清晰明了了。



这里面维护了 文件描述符 : Channel*的一个unordered_map 这样在epoll_wait之后才可以获取到fd之后找到对应的Channel

## TimerTask模块/类

最直接的描述：一个**定时任务。**

数据成员：

需要执行的任务的方法（一个方法void()），定时时长，是否被取消。
还有另一个回调方法，是因为定时任务对象一定是在时间轮里面管理起来的，**时间轮会用一个unordered_map管理这些定时对象。**
调用这个方法，从时间轮里面删除这个定时任务（和定时任务的工作机制有关）。
而因为被上层时间轮管理，所以才有这个定时任务ID。

最核心的：
**定时任务如何执行的逻辑**:  当定时任务对象析构时，析构函数中，会调用任务方法， 执行任务。而上层的时间轮就可以在时间到了之后，直接释放shared_ptr\<TimerTask>，如果引用计数为1，变为0，就会调用TimerTask的析构函数，然后执行定时任务。

**定时任务取消功能**：可以取消定时任务，很简单，只要把bool成员变量变为false，然后析构函数执行时就不会执行这个任务了。

**定时任务的延时功能**：这里的做法是：再在TimerWheel中构造一个shared_ptr\<TimerTask>这样引用计数就不是1了，这样到时候第一个智能指针被free的时候，就不会调用TimerTask的析构函数了，任务就得以延时了~

## TimerWheel模块/类

**时间轮**：管理定时任务

时间轮：一个vector，vector的每一个元素都是一个std::vector\<std::shared_ptr\<TimerTask>>，然后时间到了，秒针(一个变量)向前走，到一个std::vector\<std::shared_ptr\<TimerTask>>中，释放这里的全部智能指针，如果之前没有延时，那么此时肯定就是引用计数为1，然后直接调用TimerTask的析构即可执行任务喽。

因为涉及延时功能，所以用一个std::unordered_map<uint64_t, weak_ptr\<TimerTask>>，可以通过weak_ptr获取对应的shread_ptr，再创建一个shared_ptr并添加到时间轮中，就可以延时了。（uint64_t是定时任务ID）

**时间轮最基本的计时功能如何实现： 利用Linux的<u>timerfd</u>相关API实现定时。**

`timerfd_create`返回一个timerfd的文件描述符，可以设定如何定时，也就是每1s往描述符对应文件中写入数据，比如定时每1s，就写入一个1。

再通过一个channel管理这个timerfd，也就是设定读回调，当定时的时间到了，往文件写入数据，调用读回调方法，读取超时次数（一般是1，除非上层阻塞了很久超时多次一直每有处理这个读事件），然后让秒针根据超时次数：一般是1前移，然后到了时间轮的轮子就析构这里的智能指针。

**所以最基本的定时还是通过一个timerfd相关接口完成的，具体不是很清楚，模式就是超时了就写文件。而我们定好读事件回调即可。**



除此之外，**timerfd的读事件还是要由Poller的epoll模型监控的，所以，TimerWheel肯定是要绑定一个EventLoop的**，让这个EventLoop来监管这里的timerfd的读事件。超时了，读事件就绪就调用timerfd绑定的channel里的handleevent。handleevent检测到读事件就绪，调用读事件回调方法，其实读事件回调就是Ontime：Ontime逻辑也很简单: 读取timerfd的文件内的超时次数，然后秒针前移，释放智能指针则执行任务。

其实就是想说，时间轮肯定是要绑定一个EventLoop -> Poller -> epoll -> 来监控timerfd的读事件的，不然超时了，文件写入了，你怎么知道读事件就绪/触发了呢？    这本来是就是一个主从Reactor模式的TCP服务器呀。



有关一些接口，逻辑都简单，明白它的功能和实现目的即可。

对外接口: 添加定时任务, 刷新定时任务, 删除定时任务, 这里和多线程安全有关

主要是这里的InLoop我又有点不太懂了，可能需要再回顾回顾。

## EventLoop模块/类

> 注释：下方用EL代替EventLoop

EventLoop类，正如其名所示，事件循环类。

**<u>EventLoop数据成员解析：</u>**

- 包含一个Poller，也就是一个epoll，进行事件监控。

- 并且因为是one thread one loop，所以这个EventLoop要绑定一个线程，所以有一个线程id。（但是线程并不是在这里创建的，还要在上层封装中创建，只是EL要绑定一个线程。）

- 任务池：此eventloop除了要监管tcp连接的事件，并调用事件处理的回调方法外，还有一个任务池，每次loop时都要执行一下任务池里的任务。不过任务池里面都可能有哪些任务需要eventloop去执行可以帮助我们更好的理解EventLoop的任务池。
  任务池里面的可能的任务：
  1. 添加定时任务
  2. 取消定时任务
  3. 延时定时任务

- 一个锁：任务池作为一个共享资源，可能同时被主线程和eventloop所绑定的线程并发访问（具体场景代码中的注释有），所以需要一个锁保护这个任务池，哪里加锁，哪里就是可能并发访问的地方了呗，也就是临界区！！！！
- **一个<u>eventfd</u>**，其实就是Linux的一套有关eventfd的API(类似于timerfd)，目的：因为EL中不仅处理各种连接的事件，还要处理任务池的任务。所以当外部线程往任务池中投放任务，但是可能EL正在epoll_wait中阻塞等待事件，所以可能无法及时地处理这个任务，所以用eventfd来解决这个问题。
  解决方法也很简单，这个eventfd在初始的时候就已经被EL监管读事件了，所以往eventfd写数据，读事件触发，就可以让EL线程从epoll_wait中返回。哈啊哈哈哈哈哈，从而处理eventfd的读事件，然后执行任务池的任务！！！！！读事件处理方法也很简单，读一下eventfd对应文件中的数据即可。只是因为目的就是为了唤醒epoll_wait中阻塞的线程，所以内容也不重要。（这里是为了实现定时删除功能，因为任务池中的任务很多都是定时任务相关的操作，比如添加/删除/更新。你想想，每次TCP连接建立，要创建定时任务，有新数据/新事件，要延时/刷新定时任务。这些都是很频繁的）
  目前我还没有确定除了定时任务相关的之外，这个任务池还有可能有什么任务，需要后面再看。
- 当然还要包含一个时间轮TimerWheel才能实现定时任务啊，每一个EventLoop都绑定一个线程，一个TimerWheel），在TimerWheel构造的时候，就已经把TimerWheel中的那个timerfd的读事件关心添加到这个EL中了，而对应的读事件处理其实就是Ontime。这不就前后联系起来了吗？

ok了，所有的数据成员解读结束。

<u>**EventLoop成员方法解析**</u>

Start进行事件循环呀，还有RunInLoop用于外部线程（主从Reactor，所以其实就是主Reactor线程）给EventLoop放任务啊。

还有就是，epoll - Poller相关 ：添加/更新/删除文件描述符的监控

事件轮TimerWheel相关：添加/更新/删除定时任务

EL的定时任务接口调用TimerWheel的接口，TimerWheel调用RunInLoop，而具体的往任务池中放的任务(不是定时任务)其实就是TimerWheel实现的功能了，比如定时任务的增加，删除，延时等等。

事件监控相关：Channel的添加/删除事件关心，调用EL的接口。EL又作为Poller的封装，所以肯定是要调用Poller的接口。这里面并不涉及RunInLoop, 因为可以确定事件关心等都是这个EventLoop所绑定的线程进行的!!!!
而定时任务的方法需要RunInLoop主要是因为, 定时任务的添加/删除等可能是由主Reactor线程来进行的

**这两者的区别（RunInLoop）和定时事件和事件关心的使用场景有关。**



okokok很清晰了。~~fuckyou~~

## LoopThread模块/类

EventLoop和一个线程绑定，且是在EventLoop构造时，直接将构造它的线程作为绑定线程。**所以LoopThread实际上就是：创建从属Reactor线程，并且在从Reactor线程内部去构造EventLoop对象。然后在这个线程内部去调用EventLoop的Start进行事件循环。**

外部可能会获取这个EventLoop*（实际上外部指的就是主Reactor线程，**<u>不然还能是谁</u>**），但是可能获取时（调用GetLoop），EventLoop还没构造出来，所以要有一个锁和条件变量来解决这个问题。

其实不用想的那么复杂， 外部主Reactor线程可能获取这个EventLoop\*，从Reactor线程可能会修改EventLoop*，所以这就是多线程并发访问。所以加锁+条件变量。

## LoopThreadPool模块/类

上方的LoopThread其实就是一个从属Reactor线程。并且自动开始进行事件循环EventLoop。

**而从属Reactor线程可能有多个，所以，这里的LoopTheadPool就是对从属Reactor线程池的封装。**

并且最重要的，如果确实是主从Reactor模式，则主Reactor线程在获取新连接之后，需要把后续新连接的监控及事件处理交给从属Reactor线程池内的某一个线程。并且要负载均衡，所以这里面的GetLoopBalanced方法就是这个功能，**其实采用的就是轮询的方式进行的。每次获取一个从属Reactor线程。**

## Any模块/类

## Connection模块/类

Connection模块是对Buffer模块，Socket模块，Channel模块的⼀个整体封装，**实现了对⼀个通信套接字的整体的管理，每⼀个进行数据通信的套接字（也就是accept获取到的新连接）都会对应一个Connection对象**

## Acceptor模块/类

Acceptor模块是对Socket模块，Channel模块的⼀个整体封装，**实现了对⼀个监听套接字的整体的管理。**

监听套接字的管理嘛，肯定要有EventLoop*也就是主Reactor的EventLoop。

还要有socket，肯定没有Buffer，还要有Channel，emmmm，大概就这些吧。

事实上，还有一个读事件回调函数，其实就是获取新连接的逻辑呗。





emm，Acceptor是对listen套接字的管理，创建这个Acceptor时，会把listen套接字的Channel绑定到主Reactor线程上，然后设定读事件回调，这里有两部。第一步是Acceptor的成员函数，也就是最直接的回调，调用Socket的accept获取新的通信套接字。但是此时主Reactor线程需要将这个新连接分配到从Reactor线程池中，但是Acceptor并没有LoopTHreadPool数据成员，也就无法进行分配，所以具体的后续工作，其实是由TcpServer模块进行的， 也就是通过回调的方式。Acceptor这里的逻辑只是accept获取一下新连接，然后调用回调。（TcpServer设定的新连接建立与分配逻辑）

其实挺简单的。

并且在Acceptor的构造中并没有开启主Reactor（其实就是一个EventLoop）的读事件关心。只是设定了读事件回调。

开启读事件关心还设定了一个方法。



也就是说其实在构造这个对象时，就开始了listen，开始进行三次握手，会把建立好的连接放在全连接队列中。但是还没有开启读事件关心也就没有真正获取新连接。

## TcpServer模块/类

最终的整合模块

作为一个主从Reactor模式的TcpServer，免不了有一个主Reactor线程，也就是EventLoop _base_loop

还要有一个LoopThreadPool，其实就是若干个LoopThead，也就是若干个从属Reactor线程，本质上还是若干个进行EventLoop的EventLoop（可以设定，如果这个里面从属Reactor线程数量为0，则整体就是一个单Reactor单线程模型了）

Acceptor对象，配对这个_base_loop。

因为是用Connection来描述每一个连接嘛，所以有一个conn_id，也就是每一个Connection一个conn_id。

使用一个unordered_map来组织所有的Connection。

端口号

timeout超时时间（超时自动销毁连接功能）  是否开启超时自动销毁（bool类型）



注意，Connection的读时间，写事件，异常等事件回调是在Connection里面的。而这里的若干个方法其实是，

业务处理回调，

连接建立时的回调，

连接关闭时的回调，

任意事件回调，

它们分别有作用，后面再看。



构造函数：

比较简单，函数体内进行的是：设定新连接建立的回调方法。然后开启主Reactor对于读事件的关心（base_loop）

并且LoopThreadPool _pool;   // 从属Reactor线程池在构造函数的初始化列表中就已经完成构造了，也就是若干个从属Reactor线程已经开始EventLoop了。

几个回调函数的设定方法。（就是上面那几个）

Start方法：主Reactor只是对事件进行了关心，但是主Reactor也就是baseloop这个EventLoop还没开始进行Start，进行epoll_wait，这里的Start就是让主Reactor线程开始进行事件循环，处理可能已经就绪的listen套接字的读事件。



**比较核心的：  void NewConnection(int fd) {**

其实就是主Reactor在accept获取新连接之后会执行这个。也还好吧



其实Connection的SvrCloseCallback这个回调只是，删除一下TcpServer用unordered_map组织的Connection罢了。

**但是，这个工作不能直接由从线程完成，因为这样就可能多线程并发访问这个unordered_map了，就涉及线程安全了，所以这个回调的处理方式是给主Reactor线程，也就是baseloop这个EventLoop的任务池中添加一个任务，任务就是TcpServer的另一个方法，也就是从unordered_map中删除对应的Connection。**
