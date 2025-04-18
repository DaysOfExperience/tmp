# 7_进程信号

## 信号的基本认识

Linux信号机制：

**它是一种异步的通知机制，用来提醒进程一个事件已经发生。**

**信号是一个软中断。操作系统通过信号通知某个进程发生了某件事件，然后中断这个进程当前操作，让它优先去处理这个事件。**

> 我们在linux下常用的kill命令就是通过向进程发送一个信号来使进程中断，我们可以通过kill -l来查看信号的种类

![image-20231116215734535](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215734535.png)

Linux操作系统中，有编号为1~31的31个普通信号，编号为34~64的31个实时信号，共62个信号。日常中只会涉及和使用到普通信号。故下方对信号的学习仅对于1~31的普通信号。

> 每个信号都有一个编号和一个宏定义名称，本质上，这些都是通过#define的形式定义的。也就是用一个int型变量去代替某特定信号。（预处理之后，这些宏定义都会变为int整型）
>
> ![image-20231116215745501](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215745501.png)

## 信号产生：产生信号的若干种方式

### 1.通过键盘产生信号

ctrl + c：通过键盘组合键向前台进程发送2号SIGINT信号。

ctrl + \：通过键盘组合键向前台进程发送3号SIGQUIT信号。

> 有关前台进程与后台进程： Ctrl-C 产生的信号只能发给前台进程。一个命令后面加个&可以放到后台运行（如 ./mysignal &）,这样Shell不必等待进程 结束就可以接受新的命令,启动新的进程。Shell可以同时运行一个前台进程和任意多个后台进程,只有前台进程才能接到像 Ctrl-C 这种控制键产生的信号

前台进程在运行过程中用户随时可能按下 Ctrl-C 而产生一个信号,也就是说该进程的用户空间代码执行 到任何地方都有可能收到 SIGINT 信号而终止, 所以信号相对于进程的控制流程来说是**异步(Asynchronous)**的。

### 2.通过系统调用接口产生信号

`int kill(pid_t pid, int signo); `向指定进程发送指定信号

`int raise(int signo);` 向当前进程发送指定信号

`void abort(void);` 向当前进程发送6号SIGABRT信号

Linux还有一个kill命令，就是通过调用kill函数实现的。

```C++
// 通过系统调用发送信号
 
void handler(int signo)
{
    std::cout << "进程收到了一个" << signo << "号信号" << std::endl;
}
 
int main()
{
    signal(2, handler);  // 捕捉下方kill 和 raise发送的信号
    signal(SIGABRT, handler);  // 捕捉abort发送的6号SIGABRT信号
    kill(getpid(), 2);
    raise(2);
    abort();    // 向当前进程发送SIGABRT信号，使其异常终止（默认）
 
    while(true) sleep(1);
    return 0;
}
```

> ![](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215752723.png)
> 如上图，对6号SIGABRT信号明明捕捉了但是还是中止了。 这是一个很奇怪的现象，查了stack overflow：However, I cannot find any corroborating evidence of that behavior in the signal man page, which clearly states that The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored but makes no similar mention for SIGABRT. 也就是信号手册种明确说了9和19号信号不能被捕捉，阻塞，或者忽略，但是没有说SIGABRT信号不能被捕捉。其实这个是因为调用了abort，abort不仅发送了SIGABRT信号，还做了其他事情。所以如果把abort();换成kill(getpid(), SIGABRT); 就会捕捉SIGABRT且进程不会退出。


如何理解通过调用系统接口产生信号：其实很简单，系统调用接口通过提取参数，获取进程pid，信号编号，然后OS向进程PCB内写信号，也就是修改对应进程的pending位图的特定位，后续进程处理该信号。

### 3.由软件条件产生信号

如: 进程间通过管道通信时，因软件条件而产生SIGPIPE信号。

> 进程间通过管道通信时，不管是命名管道还是匿名管道，都会提供访问控制。也就是管道读写的四种情况。情况之一是：当管道的读端关闭，写端继续写，则OS会向写端进程发送SIGPIPE信号从而终止写端进程。（SIGPIPE信号的默认处理方式就是terminate process，终止进程。产生条件：Broken pipe：write to pipe with no readers）

```C++
// 通过软件条件发送信号，比如管道读端关闭，写端继续写，OS会终止写端进程
void handler(int signo)
{
    std::cout << "进程收到了" << signo << "号信号" << std::endl;
}
 
int main()
{
    int pipefd[2] = {0};
    pipe(pipefd);  // 创建匿名管道
 
    pid_t id = fork();
    if(id == 0)
    {
        // 子进程，读端，关闭写端
        close(pipefd[1]);
        sleep(3);
        close(pipefd[0]); // 三秒后关闭读端
    }
 
    // 父进程写端
    signal(SIGPIPE, handler);
    close(pipefd[0]); // 关闭读端
    while(true)
    {
        const char* message = "haha";
        write(pipefd[1], message, strlen(message));
        sleep(1); // 每隔一秒写一次
    }
    return 0;
}
```

如上，对SIGPIPE信号进行捕捉，当OS发送SIGPIPE信号时，会执行handler方法。下方打印1s进行一次。

![image-20231116215802730](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215802730.png)

SIGPIPE是一种由软件条件产生的信号

如: alarm函数，因软件条件产生SIGALRM信号。

> #include <unistd.h>
> unsigned int alarm(unsigned int seconds);
> 调用alarm函数可以设定一个闹钟,也就是告诉内核在seconds秒之后给当前进程发SIGALRM信号, 该信号的默认处理动作是终止当前进程。
> 这个函数的返回值是0或者是以前设定的闹钟时间还余下的秒数。（也就是一个进程在同一时刻只会有一个闹钟）

下方是一个基于alarm和SIGALRM信号的简单定时执行某任务的程序。

```C++
// 因软件条件产生信号：alarm函数，SIGALRM信号
 
typedef std::function<void()> func;
std::vector<func> callbacks;
 
uint64_t count = 0;
 
void showCount()
{
    std::cout << "current count is : " << count << std::endl;
}
 
void showLog()
{
    std::cout << "Log..." << std::endl;
}
 
void logUser()
{
    if(fork() == 0)
    {
        // 子进程
        execl("/usr/bin/who", "who", nullptr);
        exit(1);
    }
    // 父进程等待回收一下
    wait(nullptr);
}
 
void catchSIGALRM(int signo)
{
    std::cout << "进程收到了SIGALRM信号" << std::endl;
    // 对于SIGALRM的自定义捕捉
    for(auto& func : callbacks)
    {
        func();
    }
    alarm(1);   // 再定一个定时器
}
 
int main()
{
    callbacks.push_back(std::function<void()>(showCount));
    callbacks.push_back(std::function<void()>(showLog));
    callbacks.push_back(std::function<void()>(logUser));
 
    signal(SIGALRM, catchSIGALRM);  // 捕捉SIGALRM信号
    alarm(1); // 定一个1s的定时器
 
    while(true) ++count;
    return 0;
}
```

之前的SIGPIPE可以理解为是在管道IPC中，软件条件不满足从而OS发了信号。这里的SIGALRM可以理解为是闹钟软件条件满足从而发了信号。

这里的定时器闹钟，一定是OS管理的，因为整个程序只有一个执行流。OS中有很多进程，每个进程都有可能通过alarm设定定时器，所以，OS对于这些闹钟，一定是要管理的。所以需要先描述，再组织。就像struct file，task_struct一样。OS定期检查操作系统内所有的闹钟哪个到时间了，就向指定进程发送SIGALRM信号。

### 4.因硬件异常产生信号

例一：除零操作，触发硬件异常，OS发送8号SIGFPE信号

![image-20231116215810688](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215810688.png)

OS发送SIGFPE信号是当进程犯了一个floating point exception（浮点异常），本质上是由于硬件异常引起的，这种异常通常发生在进程进行浮点运算时出现了无效操作，如除以零。

例二：进程访问非法内存地址，触发MMU硬件异常，OS发送11号SIGSEGV信号

![image-20231116215820240](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215820240.png)

```C++
void handler(int signo)
{
    std::cout << "进程收到了一个" << signo << "号信号" << std::endl;
}
int main()
{
    signal(SIGFPE, handler);
    signal(SIGSEGV, handler);
    // int i = 1/0;   // 除零

    int* p = nullptr;
    *p = 10;
    return 0;
}
```

### 信号产生总结

上方所说多种产生信号的方式，不管方式是什么，最终一定是由OS向进程发送信号，而发送信号的本质为OS向进程PCB中的pending位图的对应比特位由0置1。最终都是由OS来执行，因为OS是进程的管理者！

产生信号  -> 在进程中注册信号

> **核心转储-Core Dump**
>
> ![image-20231116215828122](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215828122.png)
>
> <img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215834524.png" alt="image-20231116215834524" style="zoom:67%;" />
>
> **如上，Term表示这个信号的默认动作是终止这个进程，Core表示这个信号的默认动作是终止这个进程并核心转储（Core Dump）。它们的区别就是是否进行核心转储。**
>
> **当一个进程要异常终止时,可以选择把进程的用户空间内存数据全部 保存到磁 盘上,文件名通常是core,这叫做Core Dump。**
>
> 核心转储作用: 
>
> 进程异常终止通常是因为有Bug, 比如非法内存访问导致段错误, 事后可以用调试器检查core文件以查清错误原因,这叫做Post-mortem Debug（事后调试）。**核心转储的作用就是为了方便调试, 定位bug。**
>
> 注意
>
> 通常在云服务器这样的生产环境中核心转储功能是关闭的，也就是默认不允许产生core文件，因为core文件中可能包含用户密码等敏感信息, 不安全。其次，core文件体积较大，每次进程因Core信号而异常终止时如果都会进行核心转储生成core文件，时间长了是很耗费磁盘空间的。
>
> 可以通过ulimit -c 10240命令开启core dump功能。![img](https://img-blog.csdnimg.cn/img_convert/2aaa38ed39b7ef869ae6433533714b60.png)
>
>
> 使用core.pid文件：gdp调试->core-file core.pid即可。（配合gdb）

### 进程等待wait/waitpid的core dump标记位

![image-20231116215846226](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215846226.png)

之前在进程控制中的进程等待那里，可以传一个int* status的输出型参数获取子进程退出信息，当子进程被信号所杀时，status的第8个比特位就是core dump标记位（前7个比特位为终止信号的编号），若真的生成了core文件，则该标记位为1，否则为0。
注意：若被Action为Core的信号所杀，但是环境不允许生成core文件，则该标记位为0。也就是它标记的是当进程出现某种异常的时候，是否由OS将当前进程在内存中的相关核心数据，转存（dump）到磁盘中。

```C++
int main()
{
    if(fork() == 0)
    {
        sleep(3);
        int* p = nullptr;
        *p = 10; // SIGSEGV默认处理动作为Core 
    }
    int status = 0;
    waitpid(-1, &status, 0); // 阻塞式等待
    std::cout << "signal num : " << (status & 0x7f) << std::endl;
    std::cout << "core dump flag : " << ((status >> 7) & 1) << std::endl;
    return 0;
}
```

## 信号保存：进程如何保存收到的信号

为什么要进行信号保存?

在OS因某种原因给某进程发送了某信号之后，进程必须处理该信号，时机：进程是在合适的时候处理该信号的（由内核态转为用户态时，详见信号处理），所以，**进程处理信号可能不是立即的，故进程需要将收到的信号保存起来。**

### 信号相关概念

- 实际执行信号的处理动作称为**信号递达(Delivery)** （默认，忽略，自定义捕捉）

- 信号从产生到递达之间的状态,称为**信号未决(Pending)**

- 进程可以选择**阻塞（block）**某个信号。

- 被阻塞的信号产生时将保持在未决状态, 直到进程解除对此信号的阻塞, 才执行递达的动作。

- 注意,阻塞和忽略是不同的, 只要信号被阻塞就不会递达, 而忽略是信号递达的一种具体方式。

### 进程保存信号的方式

![image-20231116215853717](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215853717.png)

- **block信号集（阻塞信号集，信号屏蔽字）**：在进程PCB中本质是一个位图结构（sigset_t)。
  **每一个bit位用于表示对应信号是否被阻塞（屏蔽）**

- **pending信号集（未决信号集）** : 在进程PCB中本质是一个位图结构（sigset_t）
  **每一个bit位用于表示对应信号是否处于未决状态**（简单理解就是进程是否收到了这个信号，等处理状态，具体处理不处理还要看是否被block）

- **handler方法处理表**：**一个函数指针数组**。`void (* handlerArray[32]) (int);`
  信号处理三种方式：默认，忽略，自定义捕捉。这里存储的就是对应信号的具体处理方法，若存储的是SIG_DFL（本质是一个宏定义），就是该信号要执行默认处理逻辑，若SIG_IGN，就是忽略该信号，若自定义捕捉，则这里存的就是自定义捕捉方法的函数地址。（signal函数修改的就是handler方法处理表）

> 综上，也就是，进程收到信号之后，不一定会递达该信号，还要看该信号是否被该进程屏蔽（block）了。具体为什么要增加block信号这个功能呢？肯定是因为需要这个功能啊，有需求和使用场景...
>
> block信号集和pending信号集的结构完全一样，就是一个位图结构。只是具体比特位的意义不同，分别表示对应信号是否被阻塞和是否处于未决状态。故每个信号都有两个标志位分别表示阻塞(block)和未决(pending)
>
> pending信号集只能存储对应信号处于或不处于未决状态，因此若一个进程block某个信号，然后接收到了多次这个信号，实际上只会存储一次。解除block之后也只会递达一次。普通信号是这样的。而实时信号产生多次会以此放进一个队列中...不讨论

**sigset_t**

由上，我们已知pending信号集和block信号集本质就是一个位图结构，它们的类型其实就是sigset_t类型，sigset_t就是一个位图结构。

sigset_t是操作系统提供的自定义类型（语言会提供.h，.hpp以及语言的自定义类型，如int，double，OS也会提供.h和自定义类型，比如pid_t，sigset_t）

sigset_t类型对于每种信号会用一个bit表示“有效”或“无效”状态。具体这个有效和无效在阻塞信号集和pending信号集中含义不同，阻塞信号集中有效和无效指的是该信号是否被阻塞，在未决信号集中指的是该信号是否处于未决状态...

OS给我们提供了操作sigset_t（位图）的方法。也就是这个位图结构不建议我们去直接操作和修改，建议使用操作系统提供的接口。（因为linux内核是C语言写的，所以其实这个sigset_t位图就是一个struct，我们是可以获取到它的成员的）

**位图信号集sigset_t操作函数**

```C++
int sigemptyset(sigset_t *set);      // 初始化信号集，所有信号对应bit清零
int sigfillset(sigset_t *set);       // 初始化信号集，所有信号对应bit置位
int sigaddset (sigset_t *set, int signo); // 添加某种信号
int sigdelset(sigset_t *set, int signo);  // 删除某种信号
int sigismember（const sigset_t *set, int signo);  // 判断某信号在信号集中是否有效
```

**pending信号集，block信号集，handler方法处理表相关的函数接口**

```````c++
// 0. 读取或更改进程的信号屏蔽字
int sigprocmask(int how, const sigset_t *set, sigset_t *oset); // oset为输出型参数，用于获取旧的信号屏蔽字
// 有关sigprocmask的how参数：传入下方宏定义
// SIG_BLOCK ：set包含了我们希望添加到当前信号屏蔽字的信号，相当于mask = mask | set
// SIG_UNBLOCK : set包含了我们希望解除阻塞的信号，相当于 mask = mask & ~set
// SIG_SETMASK : 设置当前信号屏蔽字为set所指向的sigset_t，相当于mask = set
// 1. 读取当前调用进程的pending信号集（未决信号集）
int sigpending(sigset_t *set);
 
// 2. 自定义信号捕捉方法
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);  // handler是回调函数，通过回调的方式，修改对应的信号捕捉方法。
```````

signal系统调用：本质就是OS将调用进程的PCB中的handler方法表中signum对应函数指针修改为handler，即完成了signum**信号的自定义捕捉**，之后该进程进行signum信号递达时，就会调用handler方法（该方法在用户空间中，详见信号处理）

有block信号集的读写接口，有pending信号集的读接口，但是没有pending信号集的写接口。实际上，信号产生的方式就是写pending信号集的方式。kill -x pid

### 编码验证

1. 若进程将所有信号都捕捉，则进程无敌？

```C++
void handler(int sig)
{
    std::cout << "进程收到了一个" << sig << "号信号" << std::endl;
}
 
int main()
{
    std::cout << getpid() << std::endl;
    for(int sig = 1; sig <= 31; ++sig)
    {
        signal(sig, handler);
    }
    while(true) sleep(1);
    return 0;
}
```

> 然后给进程发送1~31号信号，结果：9和19号信号无法被捕捉，且19号SIGSTOP暂停进程之后，发送18号SIGCONT信号，18号信号被捕捉了，会调用18号信号的捕捉方法，同时也会continue process，也就是生效了...这些其实不是很重要，大概率是一些特殊设计。只需要注意并非所有的普通信号都可以被捕捉。

2. 验证将某信号block之后，向进程发送该信号，是否pending信号集中会有对应的1标记位出现

```c++
void showSigset(sigset_t* sigset)
{
    // 此处信号集可能为pending信号集可能是block信号集。
    for(int sig = 1; sig <= 31; ++sig)
    {
        if(sigismember(sigset, sig) == 1)   std::cout << "1";
        else    std::cout << "0";
    }
    std::cout << std::endl;
}
 
void handler(int sig)
{
    std::cout << "进程收到了一个" << sig << "号信号" << std::endl;
}
 
int main()
{
    for(int sig = 1; sig <= 31; ++sig)
    {
        signal(sig, handler);
    }
    sigset_t sigset;
    sigprocmask(SIG_BLOCK, nullptr, &sigset); // 获取进程初始时的block信号集
    std::cout << "初始时，进程的block信号集为 : ";
    showSigset(&sigset);
    sigpending(&sigset);
    std::cout << "初始时，进程的pending信号集为 : ";
    showSigset(&sigset);
 
    sigfillset(&sigset);
    sigprocmask(SIG_BLOCK, &sigset, nullptr); // 试图将全部信号进行block
 
    sigprocmask(SIG_BLOCK, nullptr, &sigset); // 获取新的block信号集
    std::cout << "试图将全部信号进行block之后，block信号集为 : ";
    showSigset(&sigset);
 
    sleep(5);
    int count = 0;
    while(true)
    {
        // 每秒打印一次pending信号集
        sigpending(&sigset);
        std::cout << "此时，pending信号集为 : ";
        showSigset(&sigset);
        sleep(1);
        count++;
 
        // if(count == 10)
        // {
        //     // 将全部信号解除block，看是否会递达
        //     sigemptyset(&sigset);
        //     sigprocmask(SIG_SETMASK, &sigset, nullptr); // 解除全部信号block
        // }
    }
    return 0;
}
```

<img src="https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215909254.png" alt="image-20231116215909254" style="zoom:67%;" />

上方，试图将1~31号信号都block，然后查看进程的信号屏蔽字，发现9号SIGKILL和19号SIGSTOP信号无法被阻塞，然后按编号顺序向进程每隔1s发送一个信号，不发9和19，结果如图，确实pending信号集中会存储因为被block而处于未决态的信号。

唯一超出预期的是18号信号，18号信号确实被block了，直接向进程发送18号信号，18号的pending信号集的比特位确实会变为1，但是当向进程发送19，20，21，22号时（这些信号的默认动作都是STOP），18号信号的比特位会变为0，也就是进行了信号递达...其实在上一个示例中就说过了SIGCONT的特殊，应该是特殊处理了，大致了解即可。

故，9号SIGKILL和19号SIGSTOP信号无法被block，无法被捕捉，无法被忽略。这也是为了防止，若恶意进程将所有信号block，则user和OS将无法杀死进程。

## 信号处理：进程处理信号的时机和流程

### 进程处理信号的三种方式

信号的接收方一定是进程，因为信号就是OS用来提醒进程某个事件已经发生。故，进程在接收到信号之后，一定要处理这个信号。

1. **默认处理方式：** 就是操作系统为每一种信号准备的对应的处理方式, 执行该信号的默认处理动作。
2. **忽略处理方式：** 忽略该信号。
3. **自定义处理方式：** 我们可以自己写一个回调函数(一个信号处理函数)来替换原来的处理方法，要求内核在处理该信号时切换到**用户态**执行这个处理函数，这种方式称为捕捉（Catch）一个信号。（就是程序员自定义进程对某信号的处理方法，该方法存储在用户代码中）

### 进程什么时候处理信号

OS向进程发送信号之后（pending信号集中某信号处于未决状态），那么，进程不是立即处理该信号的，而是在合适的时候（这也是为什么要把信号保存起来）。那么这个合适的时候是什么时候呢？

注意，有关信号的内核数据结构，如pending信号集，block信号集和handler方法处理表都在PCB中，这是内核数据，属于内核范畴。故，想要处理信号，就必须访问这些内核数据，则此时CPU的状态必须处于内核态（因为只有内核态才有权访问内核数据）。相对的，CPU运行用户程序，执行用户代码时，就处于用户态。

>  用户态和内核态的概念：内核态和用户态是操作系统内核与用户进程之间的两种不同状态。在这两种状态中，操作系统内核和用户进程**对于系统资源和硬件设备的访问权限是不同的**。
>
>  **内核态是 CPU 执行操作系统内核代码的状态。**在内核态中， CPU 拥有所有特权，可以执行所有底层操作，并具有访问所有内存地址和设备的权限。这种状态下，内核能够执行所有系统调用和硬件中断处理。**所有关于操作系统内核的操作都需要在内核态下执行。**
>
>   **用户态是 CPU 执行用户进程代码的状态。**在用户态中，进程只能访问被分配给它的内存，并且不能直接访问硬件和其他进程的内存。**当一个进程需要进行一些特殊操作，如请求系统调用、执行系统调用，则需要从用户态切换到内核态。**因此用户态下的进程不能进行一些底层的操作，对系统资源和硬件设备的访问也有限制，这些操作需要通过系统调用来完成。<u>这种方式可以保证系统安全。</u>
>
>  **划分内核态和用户态的主要目的是为了保护操作系统和其他进程免受用户进程的损害。** 在内核态下，操作系统内核拥有所有特权，可以执行所有底层操作和访问所有内存地址和设备，因此可以保证系统的稳定性和安全性。而在用户态下，用户进程只能访问被分配给它的内存，并且不能直接访问硬件和其他进程的内存，这样可以防止用户进程对其他进程和操作系统产生影响。 通过这种方式，可以将操作系统的核心代码和用户进程的代码分离开来，保护操作系统免受用户进程的损害。<u>当然这样也带来了一些性能的损失</u>，因为要在两种状态间不断地切换, 但相对于安全性来说是值得的
>
>  结合进程地址空间
>
>  ![image-20231116215919343](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215919343.png)
>
>  进程地址空间分为0~3G用户地址空间和3~4G内核地址空间（32位），用户空间通过用户级页表映射到物理内存中的用户数据和用户代码。内核空间通过内核级页表映射到物理内存中的内核数据和内核代码。
>
>  通常，用户级页表每个进程都有一个，因为它们映射的物理内存不同，实现了进程独立性。而操作系统在物理内存中只有一个，内核级页表也只需要一个即可，内核级页表映射到物理内存中的内核数据和内核代码，这个内核级页表是所有进程共享的。
>
>  结合，内核态就是CPU执行内核代码的状态，用户态就是CPU执行用户代码的状态。比如用户代码中调用了系统调用接口，就会从从用户态转为内核态（接口中有转换的指令），否则用户态无法执行内核代码。其实这个和调用动态库内的函数接口没什么本质区别，只是动态库代码在共享区，而系统接口代码在内核空间中。都是在进程地址空间内不断跳转完成的。用户代码可以直接跳转执行动态库中的代码，因为都属于用户空间的代码。
>
>  **哪些情况下会从用户态转为内核态？**
>
>  最典型的：用户代码调用系统接口，这个系统调用接口就属于内核代码，执行内核代码必须处于内核态，而系统调用接口编译之后形成的汇编指令中就有从用户态向内核态转变的指令，比如int 80（这里的int并非C语言的基本类型，而是一种汇编指令，意为interrupt）。这也就说明了，OS可以访问并管理硬件，而进程若想访问硬件，则必须调用系统调用接口通过OS访问，因为只有合法的系统调用接口里面才有向内核态的转换，否则用户态是无法直接执行内核代码或访问内核数据的，因为没有权限。这也是OS对硬件保护的体现。
>
>  其次：若某进程的调度时间到了，OS需要进行进程切换，此时执行的一定是内核代码
>  故并非一个进程一直执行用户代码就不会进入内核态，像这种进程调度切换是无法避免的。
>
>  还有：缺陷，中断，异常等的发生也会使CPU进入内核态执行OS内核代码。

> 关于内核态和用户态我的理解...
>
> 内存中有很多代码，都是二进制序列，CPU会不断地执行代码，代码分为用户代码和内核代码。CPU在执行代码时会记录此时执行的代码是用户的代码还是内核的代码，对应的就是用户态和内核态，**做此区分的主要目的就是为了限制用户态的权限**，也就是如果此时是用户的代码，则此代码不能访问那些用户没有权限访问的数据和代码（其实代码也属于数据的一种），比如内核数据，其他进程的数据。这其实就是为了保护OS，保护硬件，保护其他进程，提高稳定性。
>
> 而用户的代码若想完成某些功能，访问某些硬件或数据，则必须通过系统调用，系统调用属于内核代码，且其中就有用户态向内核态的转换指令

**进程进行信号处理的时机为：从内核态转回用户态时，进行信号检测和处理。**
也就是因某种原因进入了内核态，执行完内核代码之后，需要返回到用户代码中继续执行之前，进行信号检测和处理。

### 进程处理信号的流程

OS给进程发送信号之后（修改pending信号集）->进程因某原因陷入内核，进入内核态->返回用户态之前进行信号检测和处理->大致处理流程：检测pending信号集是否有信号处于未决状态->若有，看对应信号是否被阻塞->若阻塞，则信号处理完成（因为此时不可进行信号递达），返回用户态继续执行主控制流程的用户代码->若不阻塞，则看对应信号的handler方法处理表中函数指针

若信号处理方法为忽略（1强转为函数指针类型），则将pending信号集的对应比特位由1置0即可（忽略也是信号递达的一种方式）。信号处理结束，返回用户模式，从主控制流程中上次被中断的地方继续向下执行。

若信号处理方法为默认（0强转为函数指针类型），则执行信号的默认处理方法，如终止，暂停，继续，忽略（这里的忽略为默认处理方法）。这里因为具体处理行为不定，故要不要再返回用户态是不确定的，因为进程可能直接终止。但是，此时是处于内核态的，故，执行很多OS方法都有权限，可以直接执行。

若信号处理方法为用户自定义的捕捉方法，则此时的流程与默认，忽略不尽相同。

### 捕捉信号的流程

![image-20231116215928250](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215928250.png)

因某原因进入内核，未决，未阻塞...与默认和忽略的信号处理流程的不同之处在于

若信号的handler处理方法表中存储的是用户空间中自定义捕捉方法的函数地址，则此时需转为用户态去执行捕捉函数（上图的sighandler），注意，内核态是有权执行用户代码的，但这是不安全的，因为用户代码是否有非法操作是不确定的，故需要先转为用户态，再执行捕捉方法。执行完之后，通过系统调用sigreturn再次进入内核，做一些信号处理的收尾工作，如将pending信号集的对应信号比特位由1置0（可能这个操作并非此时进行），检测是否有新的信号产生...至此，信号捕捉完成，返回用户模式，从主控制流程中上次被中断的地方继续向下执行。（此处要恢复main函数的上下文）

![image-20231116215934934](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215934934.png)

如上图为记忆捕捉信号流程的示意图，一共进行了四次用户态和内核态之间的转变（蓝色标记），进行信号检测的时机是从内核态转回用户态之前（红色标记），上图仅适用于handler方法表中为自定义捕捉函数时。

### 信号捕捉方法2.0：sigaction

前面说了signal函数用于捕捉信号，设定信号的自定义捕捉方法。此处的sigaction也是这个功能，并有一些拓展功能。

```C++
#include <signal.h>
int sigaction(int signo, const struct sigaction *act, struct sigaction *oact);
 
// 系统提供的同名结构体类型sigaction
struct sigaction {
    void     (*sa_handler)(int);    // signo的自定义捕捉方法
    void     (*sa_sigaction)(int, siginfo_t *, void *);  // 有关实时信号，不关心
    sigset_t   sa_mask;    // 拓展功能，设定执行信号的处理函数时，block信号集的内容。
    int        sa_flags;   // 不关心，设为0即可
    void     (*sa_restorer)(void);  // 不关心
};
```

当某个信号的处理函数被调用时, 内核自动将当前信号加入进程的信号屏蔽字, 当信号处理函数返回时自动恢复原来的信号屏蔽字, 这样就保证了在处理某个信号时, 如果这种信号再次产生, 那么它会被阻塞到当前处理结束为止。处理结束后，会再次递达此信号。

```C++
// 验证：当信号捕捉函数处理时，OS是否会自动阻塞该信号
 
void showSigset(sigset_t * p)
{
    for(int i = 0; i <= 31; ++i)
    {
        if(sigismember(p, i) == 1)  std::cout << "1";
        else std::cout << "0";
    }
    std::cout << std::endl;
}
 
void handler2(int signo)
{
    std::cout << "进程收到了一个" << signo << "号信号" << std::endl;
    sigset_t sigset;
    sigprocmask(SIG_BLOCK, nullptr, &sigset);
    std::cout << "信号捕捉方法执行时，block信号集为 : ";
    showSigset(&sigset);
    sleep(10);
    std::cout << "2号信号捕捉方法执行结束" << std::endl;
}
 
void handler3(int signo)
{
    std::cout << "执行3号信号捕捉方法" << std::endl;
}
 
int main()
{
    sigset_t sigset;
    sigprocmask(SIG_BLOCK, nullptr, &sigset);
    std::cout << "初始时，block信号集为 : ";
    showSigset(&sigset);
    signal(2, handler2);
    signal(3, handler3);
 
    while(true) sleep(1);
    return 0;
}
```

![image-20231116215944294](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215944294.png)


如图，确实在信号捕捉方法执行时，会阻塞当前处理信号。但是其他信号不会被block，处理2号时，3号信号产生，会直接去处理三号。

如果在调用信号处理函数时, 除了当前信号被自动屏蔽之外, 还希望自动屏蔽另外一些信号, 则用sigaction的sa_mask字段说明这些需要额外屏蔽的信号,当信号处理函数返回时自动恢复原来的信号屏蔽字。

```C++
void showSigset(sigset_t * p)
{
    for(int i = 1; i <= 31; ++i)
    {
        if(sigismember(p, i) == 1)  std::cout << "1";
        else std::cout << "0";
    }
    std::cout << std::endl;
}
 
void handler2(int signo)
{
    std::cout << "进程收到了一个" << signo << "号信号" << std::endl;
    sigset_t sigset;
    sigprocmask(SIG_BLOCK, nullptr, &sigset);
    std::cout << "信号捕捉方法执行时，block信号集为 : ";
    showSigset(&sigset);
    sleep(10);
    std::cout << "2号信号捕捉方法执行结束" << std::endl;
}
 
void handler3(int signo)
{
    std::cout << "执行3号信号捕捉方法" << std::endl;
}
 
// 验证sigaction
 
int main()
{
    signal(3, handler3);
 
    struct sigaction sa;
    sa.sa_flags = 0;
    sigemptyset(&sa.sa_mask);
    sigaddset(&sa.sa_mask, 3);    // 2号处理时，将3号也屏蔽
    sa.sa_handler = handler2;   // 使用sigaction捕捉2号信号
    sigaction(2, &sa, nullptr);   // 使用sigaction捕捉2号信号
 
    while(true) sleep(1);
    return 0;
}
```

![image-20231116215950625](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215950625.png)


如上，在2号信号的捕捉方法执行时，发送3号信号，此时3号被block（sigaction的sa_mask字段），并没有立即递达，而是在2号捕捉方法执行完成后，恢复原来的信号屏蔽字，3号解除阻塞（此时2号也解除），递达3号。

这里也算是解释了OS设计block这个功能的本质原因（之一），也就是信号处理过程中，又来了同样的信号，怎么办？因为OS会自动阻塞该信号，故不会立即递达。处理完成后会恢复为原来的信号屏蔽字。

最后一句：信号捕捉并没有创建新的进程或线程...（突然的一句，和上方无直接关联）
这肯定是呀~.....

## SIGCHID

进程等待那里讲过wait和waitpid函数可用于避免僵尸进程和获取子进程退出信息。父进程可以阻塞等待子进程结束,也可以非阻塞地查询是否有子进程结束等待清理(也就是轮询的方式)。采用第一种方式,父进程阻塞了就不能处理自己的工作了;采用第二种方式,父进程在处理自己的工作的同时还要记得时不时地轮询一下,程序实现复杂。

其实,**子进程在终止时会给父进程发17号SIGCHLD信号，该信号的默认处理动作是忽略**（IGN，这里是默认方式的忽略，并非三种处理方式中的忽略，其实大多情况下没有区别）。

**利用子进程退出时向父进程发送SIGCHID信号，父进程可以捕捉此信号，父进程调用自定义处理函数去回收子进程或获取子进程退出信息。这样父进程就不必关心子进程了（避免阻塞或轮询方式处理子进程）。**

```C++
void handler(int signo)
{
    std::cout << "父进程收到了由子进程发送的" << signo << "号信号" << std::endl;
    pid_t id = 0;
    int status = 0;
    while((id = waitpid(-1, &status, WNOHANG)) > 0)
    {
        // 回收到了一个子进程
        if(WIFEXITED(status))
            std::cout << "子进程退出码为 : " << WEXITSTATUS(status) << std::endl;
        else
            std::cout << "子进程退出信号为 : " << (status & 0x7f) << "if core dump : " << (status >> 7 & 1) << std::endl;
    }
    std::cout << "子进程回收成功" << std::endl;
}
 
int main()
{
    if(fork() == 0)
    {
        // 子进程
        sleep(3);
        exit(1);   // 退出码为1
    }
 
    signal(SIGCHLD, handler);
 
    while(true)
    {
        printf("parent process is working\n");
        sleep(1);
    }
    return 0;
}
```

如上，可以通过捕捉SIGCHLD信号的方式回收子进程和获取子进程退出信息。

注意点：若父进程创建多个子进程，则在处理SIGCHLD时，此时具体退出的子进程数量不确定，故不能只调用一次waitpid（WNOHANG），因为SIGCHLD是普通信号，只能记录是否处于未决状态，信号数量不能保存。所以需要采用上方while循环方式不断回收，直到回收完全部的退出子进程。或者也可以把子进程pid放在一个全局vector中，每次都非阻塞式遍历等待子进程。（在只有一个子进程的情况下不需要）

上方while循环，若10个子进程中6个退出了，则会循环7次。第七次返回负数，条件不满足。

子进程并非只有终止时发送SIGCHLD给父进程，暂停时也会，故父进程在捕捉SIGCHLD的方法中必须非阻塞式等待，若阻塞式等待子进程可能会影响父进程的执行。

![image-20231116220014912](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116220014912.png)


若我们想更方便地回收子进程避免僵尸进程，且不关心子进程退出情况（也就是不想在处理函数中调用wait/waitpid），有什么更便捷的方式吗？

**事实上，由于UNIX 的历史原因，父进程调用signal或sigaction将SIGCHLD的处理方式设为SIG_IGN（忽略），这样，fork出的子进程在终止时会自动清理掉，不会产生僵尸进程。**

注意，系统默认处理方式的忽略和用户用sigaction函数设定的SIG_IGN忽略通常没有区别，但这是一个特例。此方法对于Linux可用,但不保证 在其它UNIX系统上都可用。

```C++
int main()
{
    if(fork() == 0)
    {
        std::cout << "child process pid : " << getpid() << std::endl;
        sleep(5);
        exit(0);
    }
    signal(SIGCHLD, SIG_IGN); // 忽略SIGCHLD
 
    while(true) ;
    return 0;
}
```

![image-20231116220020147](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116220020147.png)

5s后子进程自动终止且没有生成僵尸进程。

若不将SIGCHLD的处理方式设为SIG_IGNs

![image-20231116220034484](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116220034484.png)

则子进程会变为僵尸进程defunct

## 可重入函数vs不可重入函数

![image-20231116215958436](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116215958436.png)

main调用insert向链表head中头插一个结点node1，insert操作分为两步，执行完第一步之后，因硬件中断使进程切换到内核，返回用户态之前进行信号检测和处理，检测到信号之后调用捕捉方法：sighandler（用户空间），sighandler中也调用了insert方法向同一个链表中插入一个node2结点。insert(&node2);和sighandler都执行完之后，返回内核态，再次返回用户态就会从上次被中断的地方继续向下执行，即insert(&node1)的第二步操作。最后执行完insert(&node1)之后，main和sighandler先后向链表中插入两个结点，最后只有一个结点被真正插入链表中了。

**可重入函数(reentrant function)是指在一个函数调用过程中，如果它被其他函数调用，并且在第二次调用结束后能够正常返回到第一次调用，那么这个函数就是可重入函数。**

**相反，不可重入函数(non-reentrant function)就是不能在同一时间被多个函数调用的函数。**

重入一个函数就是在这个函数正在运行时再次调用它。如果这个函数是可重入的，那么它会正常处理这个重入请求；否则，可能会发生错误或者不可预料的结果。

如上，insert函数在同一时间被main和sighandler调用，因为insert访问一个全局链表，可能因为重入而造成错乱。像这样的函数就称为不可重入函数。如果一个函数只访问自己的局部变量和参数，则称为可重入函数。（想想为什么，函数栈帧！）

当一个函数访问了全局，static变量，则就是不可重入的，包括errno也是全局数据。再比如使用了malloc/free（因为malloc也是使用全局链表来管理堆的），调用了标准库IO函数（标准库IO函数很多都是以不可重入的方式使用全局数据）。则就称为不可重入函数（可重入和不可重入是函数的一种特征）

## volatile

之前在C++的地方就简单学过这个关键字。修改const变量那里？reinterpret_cast

```C++
// volatile
 
int num = 0;
 
void handler(int signo)
{
    std::cout << "num : " << num;
    num = 1;
    std::cout << "->" << num << std::endl;
}
 
int main()
{
    signal(2, handler);
 
    while(num == 0) ;
 
    std::cout << "while已退出，num != 0 num : " << num << std::endl;
    return 0;
}
```

![image-20231116220005293](https://cdn.jsdelivr.net/gh/DaysOfExperience/blogImage@main/img/image-20231116220005293.png)


1、不加volatile，默认优化程度。2、不加volatile，-O3优化程度。3、加volatile，-O3优化程度。

第二种-O3优化程度时(更高级别的优化)，编译器检测到main函数中对于这个num全局变量没有修改的语句，就进行优化行为：将num加载到CPU内寄存器中，如edx？之后while语句取num时，不再从内存中取，而是直接取寄存器中的num。这也就导致了handler将num改变之后，while仍旧没有推出。

当volatile int num = 0;之后，**用volatile声明num表示拒绝优化行为，每次取num都使用move指令去内存中取，保证了内存的可见性。**

注意，编译器的优化行为是在程序编译时进行的，也就是编译器进行的优化行为，而不是程序运行时，程序运行起来之后，程序的行为就已经确定了。
