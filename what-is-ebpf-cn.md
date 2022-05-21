# what is ebpf - 个人翻译

原作者： Liz Rice

## Versions
|Version|Desc|Date|
|:-|:-|:-|
|0.1|File Created|2022/05/19|
|0.9|First Draft of Translation|2022/05/22|

---

## 正文

### Chapter 1 Introduction
#### P1. Intruduction
在过去的几年里， eBPF 已经从一个相对朦胧的东西，转变成为现代计算基础设施中最火热的技术领域之一。就我个人而言，自从我看了 Thomas Graf 在 DockerCon 17  Black Belt session 里的演讲后，我就对 eBPF 带来的可能性感到兴奋。在云原生计算基金会（Cloud Native Computing Foundation, CNCF），技术监督(?)委员会（Technical Oversight Committe） 中的同事们将 eBPF 看作是在 2021 年中预测的众多有可能发展的技术之中很值得关注的领域之一。超过 2500 人报名参加了那一年的 eBPF 峰会线上会议（eBPF Summit virtual conference），同时世界上最著名的几家软件公司也一起创办了 eBPF 基金会 （eBPF Foundation）。很显然，这项技术肯定是有很多乐趣在其中。

在这个简短的报告中（short report），我希望能给你一些我的看法，告诉你为什么人们会对 eBPF 以及 eBPF 为在现代计算环境设计工具的能力（capabilities it offers for tooling in modern compute environments），而感到这么兴奋。（在这篇文章中）你会得到 eBPF 和他为什么这么强大的一个思维模型。这里也会有一些代码实例来帮助你具体地理解（但是如果你愿意的话也可以跳过这些）。你会理解到，在构建 eBPF 工具（eBPF-enabled tools）的过程中，哪些东西参与其中，同时也能理解为什么 eBPF 在这么短时间内变得看起来这么无处不在。

很显然，这么短的报告是没办法塞下所有细节的，但是如果你愿意深入研究，我会给出更多信息的链接。

#### P2. Extended Berkeley Packet Filter
我们先不细究这个缩写： eBPF 指的是 Extended Berkeley Packet Filter。从名字上你就能看得出，他最开始是用来过滤网络数据包（its roots lay in filtering network packets），同时最初的论文写于伯克利实验室 （Berkeley Lab, Lawrence Berkeley National Laboratory）。不过（在Liz Rice 看来），这个名字（eBPF）对传达他真正的能力不是特别有帮助，因为这个 “扩展” 的版本提供的能力已经远远多于数据包过滤。现在， eBPF 已经作为一个专用名词来使用，他的含义已经远多于这个缩写所能表示的了。

所以，如果他不是仅用来做包过滤，那 eBPF 是什么？ eBPF 是一个框架，可以让用户在系统内核中（within the kernel of the operating system），加载并运行自定义的程序 (load and run custom programs)。这说明， eBPF 可以扩展甚至是修改内核的行为。

当一个 eBPF 程序被加载到内核中时，校验器 （verifier） 首先会确保这个程序是能够安全地运行的（is safe to run），如果不是的话就会拒绝运行（rejects it if not）。一旦被成功加载， eBPF 程序需要被附加到一个事件上（be attached to an event），在这之后无论何时，只要这个事件发生，这个程序就会被触发。

eBPF 最开始是为 Linux 开发设计的，Linux 也是这个报告中重点关注的操作系统。但是我要指出的是，在我（Liz Rice）写这篇报告的时候，微软正在开发 Windows 实现的 eBPF。

由于现在广泛使用的 Linux 内核都支持了这个 “扩展” 的部分，所以现在 eBPF 和 BPF 很大程度上也可以互换使用了。

#### P2. eBPF-Based Tools
之后你在这个报告中会看到，能够动态地改变内核行为的这个能力是非常有用的。传统做法上，如果我们想要观测（observe）我们的应用的行为，我们需要在这些程序中增加代码，来生成日志和追踪（generate logs and traces）。 eBPF 让我们可以收集程序是如何运行的自定义信息（collect customized information about how an app is behaving），同时又不需要以任何方式修改这个程序。我们可以基于这种观测能力，创建 eBPF 安全工具，从内核处检测甚至是防御恶意活动。同时，我们也可以借助 eBPF 来构建强大的、高性能的网络功能，在内核层面处理网络数据包，同时避免在用户空间中进行开销巨大的转换（avoiding costly transitions to and from user space）。

从内核的视角来观测应用程序，这个并不是完全新的概念，（以前）是基于老的 Linux 特性来做的，比如说 perf ，也是可以在不对正在进行观测的应用进行修改的情况下，从内核中收集行为和性能的信息（collects behavior and performance information）。但是这些工具对可以收集的数据的范围，以及可以收集的数据的格式都有定义（限制）。借助 eBPF ，我们有了更多的灵活性，因为我们可以完全定制程序，允许我们构建各种各样实现不同目的的工具。

eBPF 编程是非常强大的，但同时也很复杂。对我们中的大多数来说，利用 eBPF 的方式不是我们自己编写程序，而是使用其他人编写的工具。已经有越来越多的，基于 eBPF 平台的项目和供应商（projects and vendors）在创造新一代的工具，覆盖了包括观测、安全、网络等各种方面。

在这篇报告的后面，我会进一步讨论一些这样的更高级的工具，但是如果你比较习惯 Linux 命令行，又迫不及待地想要上手 eBPF ，BCC 项目是一个很好的起点。他包括了非常多的追踪工具（a huge collection of tracing tools）。仅仅是瞥一眼这个列表应该就能让你明白，我们用 eBPF 可以操作的范围到底有多大，这其中包括了文件操作、内存使用、CPU 状态，甚至是观测在系统中任何地方进入的 bash 命令（observing any bash command entered anywhere in the system）。

在下一章里，我们会了解到，为什么改变内核的行为这么有用，为什么相较于直接编写内核代码， eBPF 让这件事做起来变得这么容易。

---

### Chapter 2 Changing the Kernel Is Hard
既然 eBPF 允许我们在 Linux 内核中运行自定义的代码，我们要先确保自己明白内核都做什么。然后我们就能说说为什么在讲到修改内核行为的时候， eBPF 带来了这么多的变化（why eBPF changes the game）。

#### P5. The Linux Kernel
Linux 内核是夹在你的应用程序和应用程序运行所在的硬件之间的一个软件层（software layer）。应用程序运行在一个没有权限的层（unprivileged layer），叫做用户空间（user space），这里是不能直接访问硬件的。取而代之的，应用程序通过使用系统调用接口（system call interface, syscall interface）来发送请求，请求内核代表应用程序来执行操作（request the kernel to act on its behalf）。这种硬件访问包括了：读写文件、接收和发送网络流量，甚至是访问内存。内核同时也负责协调并发进程，允许同一时间有多个应用程序同时运行。

作为应用程序开发者，我们通常不会直接使用系统调用接口，因为编程语言给我们提供了高级抽象（high-level abstractions）和标准库（standard libraries），对编程而言这些是更容易使用的接口。于是，许多人完全没有意识到内核在我们的程序运行时到底做了多少事情。如果你想知道内核介入的频率有多高，你可以用 strace 工具来显示一个应用程序执行了多少次系统调用。下面是一个例子，我们用 cat 从一个文件中读取一个单词 hello 然后把它写到屏幕上，这中间调用了超过 100 次系统调用：

```text
liz@liz-ebpf-demo-1:~$ strace -c cat liz.txt
hello
% time seconds usecs/call calls errors syscall
------ ----------- ----------- --------- --------- -------------
0.00 0.000000 0 5 read
0.00 0.000000 0 1 write
0.00 0.000000 0 21 close
0.00 0.000000 0 20 fstat
0.00 0.000000 0 23 mmap
0.00 0.000000 0 4 mprotect
0.00 0.000000 0 2 munmap
0.00 0.000000 0 3 brk
0.00 0.000000 0 4 pread64
0.00 0.000000 0 1 1 access
0.00 0.000000 0 1 execve
0.00 0.000000 0 2 1 arch_prctl
0.00 0.000000 0 1 fadvise64
0.00 0.000000 0 19 openat
------ ----------- ----------- --------- --------- -------------
100.00 0.000000 107 2 total
```

因为应用程序非常依赖于内核，这意味着，如果我们可以观测应用程序和内核的交互，我们就可以从中学习到很多关于应用是怎么运行的知识（about how an application behaves）。举个例子，如果你能够拦截打开文件的系统调用（intercept the system call for opening files），你就看到看到任何一个应用程序到底在访问哪些文件。但是你怎么去做这种拦截呢？让我们来想想，如果我们想要修改内核，增加新的代码，在系统调用的时候来增加一些输出，有哪些东西是我们需要做得。

#### P6. Adding New Function to the Kernel
Linux 内核是很复杂的，现在里面有大概三千万行的代码。对任何一块代码做修改，需要你对现有的代码有一定的了解，所以除非你已经是一位内核开发者，不然这就是一项挑战了。

但是你面临的挑战也不单纯一个技术难题。 Linux 是一个通用操作系统，会被用在各种各样的环境之中。这意味着，如果你想要对内核做一些改动，并不是说你写的代码可以运行就简单完事了。这些代码必须要被社区接受（更具体地说是被 Linus Torvalds 接受，他是 Linux 的创造者和主要开发者），因为你的改动需要给所有人都带来更大的好处。这（代码被社区和 Linus 接受）并不是理所当然的，因为提交的内核补丁里面只有三分之一被接受了。

我们假设你已经找到了一个很好的技术方法，可以拦截打开文件的系统调用。经过几个月的讨论和艰难的开发工作，假如你提交的代码被合并到了内核中。很好，但是这之后又要花多久（你提交的代码的功能）才能安装到所有人的机器上呢？

现在每两个月或者三个月，Linux 内核就会有一个新的 release ，但是即便一项改动已经成功加入到其中一个 release 中，距离这项改动在大多数人的生产环境中可以获取到（being available in most people's production environments），也仍然还有很长的时间。这是因为大多数人不是直接使用 Linux 内核的，我们用的是像是 Debian， Red Hat， Alpine， Ubuntu 之类的 Linux发行版，这些发行版包装了某一个版本的 Linux 内核，连带着还有其他的组件。你会发现你偏好的某个发行版正在使用好几年前的内核版本。

比如说，许多的企业用户正在用红帽的企业版 Linux （RHEL）。写这篇报告的时候，当前的 RHEL 的版本是 8.5 ，2021 年 11 月发布的。他基于的内核版本是 4.18 。这个内核版本是 2018 年 8 月发布的。

正如图 2-1 里的动画展示的那样，一个新功能从一个想法，一直到进入 Linux 内核生产环境，中间要花上好多年。

![](./Pasted%20image%2020220520014559.png)

#### P8. Kernel Modules
如果你不想等好多年才能让你（提交）的修改成功进入内核中，你其实还有其他的选择。 Linux 内核设计上是能接受内核模块的，这个是可以按需加载和卸载的（be loaded and unloaded on demand）。如果你想要修改或者扩展内核行为，编写模块肯定是一种办法。在我们这个监控打开文件的系统调用（instrumenting the system call for opening files）的例子中，你可以写一个内核模块来实现这个。

这里最大的挑战就是，（编写内核模块）还是一个完完全全的内核编程（full-on kernel programming）。因为一个很简单的理由，（Linux 的）使用者一直都对使用内核模块这件事非常得谨慎：如果内核代码崩溃了，那整个机器以及所有运行在机器上的东西就都挂了。作为一个使用者，他怎么才能确保这个内核模块能够安全地运行（be confident that a kernel module is safe to run）？

“安全地运行”并不仅仅意味着不会崩溃，使用者想知道的是这个内核模块从安全的角度（from a security perspective）来说是安全的。（这个内核模块）有没有漏洞会被攻击者利用？我们能相信（这个内核模块的）作者没有把恶意代码放到里面嘛？因为内核是特权代码（privileged code），它能够访问机器上的任何东西，也包括所有的数据，所以内核中有恶意代码是一件很严重的事情。对内核模块而言也是如此。

内核的安全性是 Linux 发行版要花费这么长时间来合并一个新的发布（release）的一个重要原因。如果其他人已经在各种各样的环境下运行一个内核版本好几个月或者好多年，那么（安全问题）应该差不多都被解决了（this should have flushed out issues）。这个发行版的维护者会有一定的信心确定他们的提供给使用者和用户的内核是被加固过的——就是指是能安全地运行的。

eBPF 提供了多种方式来确保安全性，包括了 eBPF 验证器（eBPF verifier），这个东西确保了 eBPF 程序只会在他能够安全运行的前提下才被加载（eBPF program is only loaded if it's safe to run）。

#### P9. eBPF Verification and Security
因为 eBPF 允许我们在内核中运行任意代码（run arbitrary code in the kernel），所以这就需要有一种机制来确保（eBPF 程序）能够安全得运行，不会把用户的机器搞崩溃，也不会损坏他们的数据（compromise their data，应该是指对数据做了不该做的修改）。这种机制就是 eBPF 验证器（eBPF verifier）。

这个验证器会分析 eBPF 程序，来确保无论输入是什么，（eBPF 程序）总是会安全的停止（always termiunate safely），而且其中值包含有限数量的指令（within a bounded number of instructions）。举个例子，如果一个程序解引用了一个指针（derefences a pointer，`*ptr` 这种操作），这个验证器就会要求程序要先检查这个指针来确保它的值不是 null 。解引用一个指针的意思是查看这个地址上的值（looking up the value at this address），null 和 0 值不是一个能查看值的合法的地址（is not a valid address to look up）。如果你在一个应用程序中解引用一个 null 指针，这个程序就会崩溃；同时如果你在内核中解引用一个 null 指针那整个机器就会崩溃，所以避免这种情况是非常重要的。

验证（Verification）这件事情同时也确保 eBPF 程序只能访问他们允许访问的内存（only access memory they are supposed to access）。举个例子，比如说一个 eBPF 程序在网络栈中被触发了（is triggered in the networking stack），同时传递了内核中包含了被传输的数据的 socket buffer（passed the kernel's socket buffer that includes the data being transferred）。有一些特殊的辅助函数（helper functions），比如说 `bpf_skb_load_bytes()` ，这个 eBPF 程序可以调用它来从 socket buffer 中读取到字节数据（read bytes of data from the socket buffer）。又比如说，另外一个 eBPF 程序因为某个系统调用而被触发了，但是现在没有可用的 socket buffer （no socket buffer available），那这个时候它（这个 eBPF 程序）就不允许使用这个辅助函数。验证器同时也会确保这个程序只能读取 socket buffer 里面的数据——也就是说他不允许访问任意内存（it's not allowed to access arbitrary memory）。这样做的目的是为了确保 eBPF 程序从安全的角度看是安全的（make sure that eBPF programs are safe from a security perspective）。

当然，我们还是有可能可以写出恶意的 eBPF 程序。如果你有办法用正当理由来观测数据，那你同样也能用不正当的理由来观测这些数据（If you can observe data for legitimate reasons, you can also observe it for illegitimate ones）。要小心我们只能从我们能够验证的来源那里，加载我们信任的 eBPF 程序，并且只能够向拥有 root 权限的人中，你所信任的那些人提供管理 eBPF 工具的权限（Be careful to load only trusted eBPF programs from verifiable sources, and only grant permissions to manage eBPF tools to people that you would trust with root access）。


#### P10. Dynamic Loading of eBPF Programs， eBPF 程序的动态加载
eBPF 程序可以动态地加载进内核，或者从内核中移除（loaded into and removed from the kernel dynamically）。一旦他们被附加到一个事件上（attached to an event），无论具体是因为什么导致这个事件出现，他们（eBPF 程序）都会因为这个事件而被触发。举个例子，如果你在打开文件的系统调用上附加一个（eBPF）程序，那么无论是哪一个进程在尝试打开一个文件，这个程序都会被触发。这（触发 eBPF 程序这件事）和 eBPF 程序被加载的时候，这些（尝试打开文件的）进程是不是已经在运行了，两者之间并没有什么关系。

这就是使用 eBPF 来做观测或者开发安全工具（observability or security tooling）的优势之一——它立刻就能提供一种视野，让你看到在这个机器上面发生的任何事情。

另外，就像图 2-2 里画的那样，人们可以利用 eBPF 非常快速地创造新的内核功能（create new kernel functionality），同时又不需要每一个 Linux 用户都接受这些改变（without requiring every other Linux user to accept the same changes）。

![](./Pasted%20image%2020220521140750.png)

现在你已经看到了，eBPF 是怎么样允许对内核进行动态的、自定义的修改（how eBPF allows dynamic, custom changes to the kernel），那让我们看看如果你想要写一个 eBPF 程序，有哪些东西是需要你知道的。

---

### Chapter 3 eBPF Programs
在这个章节中，我们会讲在编写 eBPF 代码时需要知道哪些东西。我们需要考虑运行在内核中的 eBPF 程序本身，同时也要考虑那些之后会和 eBPF 程序交互的、处于用户空间的代码（user space code that will interact with it）。

#### P13. Kernel and User Space Code， 内核空间和用户空间的代码
首先，我们可以用哪些语言来写 eBPF 程序？

内核接受的是字节码形式的 eBPF 程序（eBPF programs in bytecode form）。你还是有办法手写这种字节码的，很大程度上和用汇编语言手写应用程序代码差不多——但是一般来说，对人类而言还是使用这种之后可以被编译（自动翻译）成字节码的高级语言更加实在一些。

eBPF 程序因为一些原因，没办法用任何一种高级语言来编写。首先，这种语言的编译器需要能支持生成(?)内核所期望的 eBPF 字节码格式（have support for emitting the eBPF bytecode format that the kernel expects）。第二，许多编译型语言都有的运行时这个特征（many compiled languages have runtime features），比如说 Go 语言的内存管理和垃圾回收，就让他们不太适合写 eBPF 程序。在（Liz Rice）写这个报告的时候，编写 eBPF 程序唯一的选择就是用 C 语言（要用 clang/llvm 来编译）以及最近的 Rust 语言。绝大部分目前已经发布的 eBPF 程序都是用 C 语言编写的，这其实也很合理因为 Linux 内核就是用 C 语言写的。

不管怎么样，用户空间里面最起码要有什么东西，是要用来把 eBPF 程序加载进内核中，同时要把它附加到正确的事件上（load the program into the kernel and attach it to the right event）。像是 bpftool 这种工具可以帮我们做这些，但是这些都是底层工具（low-level tools，靠近系统底层的工具），他们都假设了（使用者）对 eBPF 程序有一定的知识（assume detailed knowledge of eBPF），而且相较于普通用户，这些工具更像是提供给 eBPF 专家使用的。大多数基于 eBPF 的工具，都有一个处于用户空间的应用程序（a user space application），来帮助我们把 eBPF 程序加载进内核，把任何配置参数传递进去，同时把 eBPF 程序收集到的信息用用户友好的方式展示出来。

至少在理论上，这些用户空间部分的 eBPF 工具，可以用任何语言来编写，尽管实践过程中这些库（libraries）也只支持一小部分语言，里面包括 C、Go、Rust、和 Python。选择语言其实还要更复杂一些，因为也不是所有的语言都有支持 libbpf 的库，而如果你要让你的 eBPF 程序在不同版本的内核上都能运行， libbpf 库目前是比较流行（常见）的选择了（... has become a popular option for making eBPF programs portable across different versions of the kernel）。（我们会在第四章讨论 libbpf）

#### P14. Custom Programs Attached to Events， 附加到事件上的自定义程序
eBPF 程序本身通常使用 C 或者 Rust 编写的，会被编译成目标文件（compiled into an object file）。这是一种标准的 ELF （Executable and Linkable Format）文件，你可以用像是 readelf 这种工具来检查这个文件类型，这里面通常包含了程序的字节码，还有（使用到的）映射的定义（contains both the program bytecode and the definition of any maps）。就像图 3-1 展示的那样，如果（前几章中提到的）验证器验证通过了，用户空间的程序会读取这个文件（指目标文件），然后把它加载进内核中。

![](./Pasted%20image%2020220521144147.png)

一旦你把一个 eBPF 程序加载到内核中，他就必须被附加到一个事件上。无论何时这个事件发生了，与之关联的 eBPF 程序就会运行。我们有非常多的事件可以把程序附加上去，这里不会涉及到所有的事件，但是下面会提到一些相对来说比较常用的选择。

####  P15. Entry to/Exit from Functions， 进入和退出函数
你可以附加一个 eBPF 程序，好让他在进入或者退出一个内核函数的时候被触发。现在的很多 eBPF 例子都是用的 kprobes （挂载到内核函数的入口，entry point） 和 kretprobes （函数返回点，function exit） 这两种机制来实现的。在比较新的内核版本中，又有了一种效率更高的替代方法，叫做 fentry / fexit。

需要注意的是，你其实没办法保证定义在某一个内核版本中的所有函数，在未来的那些版本中都一定还存在，除非他们是类似于 syscall 系统调用接口这一类稳定的 API （unless they are part of a stable API such as the syscall interface）。

你也可以通过 uprobes 和 uretprobes 来把 eBPF 程序附加在用户空间的函数中。

#### P15. Tracepoints， 跟踪点
你也可以把 eBPF 程序附加在那些被定义在内核中的 tracepoints 跟踪点里面。可以在 `/sys/kernel/debug/tracing/events` 目录下面找到你机器上的这些事件（events，这里指的是可以使用的跟踪点 tracepoints）。

#### P15. Perf Events， Perf 事件
Perf 是用来收集性能数据的一个子系统（a subsystem for collecting performance data）。你可以把 eBPF 程序通过 hook 方法挂载到所有 perf 数据被收集的地方（hook eBPF programs to all places where perf data is collected），可以在机器上通过 `perf list` 来查看（哪些地方收集了 perf 数据）。

#### P15. Linux Security Module Interface
LSM 接口（Linux Security Module Interface）在内核允许做某些操作之前，先检查安全策略（allows for security policies to be checked before the kernel allows certain operations）。你有可能已经在 AppArmor 或者 SELinux 里面用过这个接口。借助 eBPF ，你可以把自定义的程序附加到检查点上（the same checkpoints，这里的 the same 应该是指，和 LSM 接口检查安全策略的位置一样），来检查更灵活的、动态的安全策略，同时也提供了新的方法来编写运行时安全工具（new approaches to runtime security tooling）。

#### P16. Network Interfaces - eXpress Data Path， 网络接口 - XDP 高速数据路径
XDP 允许我们把 eBPF 程序附加到网络接口上，一旦接收到一个数据包（package），这个 eBPF 程序就会被触发。他可以拦截甚至是修改数据包，而且程序的返回值（exit code）可以告诉内核下一步对这个数据包做什么：（继续）传递、丢弃，或者是转发。这些都是（实现）一些非常高效的网络功能的基础。

#### P16. Sockets and Other Networking Hooks， 套接字和其他网络 Hook
你可以附加一个 eBPF 程序，让他在其他应用程序打开网络套接字或者是对网络套接字进行其他操作（open or perform other operations on a network socket）的时候运行这个 eBPF 程序，或者是在发送或接收信息（messages）的时候（运行这个 eBPF 程序）。在内核网络栈中，我们也有些叫做网络流量控制（traffic control， tc）的 hook 点， eBPF 程序可以在初始的网络数据包处理之后运行（There are also hooks called traffic control or tc within the kernel’s network stack where eBPF programs can run after initial packet processing #？ ）。

有些特殊的功能可以只借助 eBPF 程序来实现，但是大多数情况下，我们是想用 eBPF 代码来从用户空间的程序那里接收信息，或者把信息发到那边去（receive information from, or pass data to, a user space application）。能够帮助我们把数据在 eBPF 程序和用户空间之间，或者是不同 eBPF 程序之间互相传递的机制，叫做 maps 。

#### P16. eBPF Maps
maps 的发展（应该是指相较于 cBPF）就是一种重要的不同（significant differences），对于 eBPF 这个缩写来说，也刚好能够证明其中的 “e” 表示的是 “extended” 。

maps 是会和 eBPF 程序一起被定义的一种数据结构。我们有很多种不同类型的 maps ，但是他们基本上都是以 key-value 这种形式存储的。eBPF 程序可以对他们（maps）进行读写，用户空间的代码也可以做这些事。maps 的常用方式包括：
- 一个 eBPF 程序可以写入关于某一个事件的数据信息（writing metrics）和其他数据，以供后续的用户空间的代码来获取
- 用户空间的代码可以写入配置信息（writing configuraiton information），让某个 eBPF 程序来读取配置信息，并且根据这些配置信息来做出对应的动作（read and behave accordingly）
- 一个 eBPF 程序可以把数据写入到一个 map 中，以供后续另外一个 eBPF 程序来获取，从而允许跨多个内核事件协调信息（allowing the coordination of information across multiple kernel events）

如果内核和用户空间的代码都会访问同一个 map ，那么他们就都需要了解存储在这个 map 中的数据结构。可以通过在内核和用户空间的代码中都加载包含了这些数据结构的头文件来实现这点，但是如果他们（指内核和用户空间的代码）不是用的同一种编程语言的话，那就需要作者小心地创建字节兼容的结构定义（create structure definitions that are byte-for-byte compatible，应该是指同一个结构定义，在内核和用户空间的代码对应的语言中，在字节层面是对齐的）。

我们已经讨论了 eBPF 工具的主要组成部分：在内核中运行的 eBPF 程序，用来加载和交互这些 eBPF 程序的用户空间代码，以及用来在程序之间共享数据的 maps 。为了巩固这些知识，我们来看一些例子。

#### P17. Opensnoop Example
在这个 eBPF 程序例子中，我（Liz Rice）选择了 `opensnoop` ，他能展示给你看哪个进程打开了哪些文件。这个工具最开始的那个版本，是 Brendan Gregg 最初为 BCC 项目（BCC Project）编写的许多 BPF 工具的其中一个，这个项目你可以在 GitHub 上面找到。后来这个工具被用 libbpf 重写了（下个章节我们会讲这个），这里我用的是新版本的工具，在 `libbpf-tools` 这个目录下面可以找到。

当你运行 `opensnoop` 的时候，你能看到的输出的结果很大程度上取决于当时在虚拟机上到底发生了什么，不过这个结果应该和下面的差不多：

```text
PID 	COMM 	FD 	ERR PATH
93965 	cat 	3 	0 	/etc/ld.so.cache
93965 	cat 	3 	0 	/lib/x86_64-linux-gnu/libc.so.6
93965 	cat 	3 	0 	/usr/lib/locale/locale-archive
93965 	cat 	3 	0 	/usr/share/locale/locale.alias
...
```

输出中的每一行都表示，有一个进程打开（或者尝试打开）一个文件。里面的行记录的是进程ID、运行的命令、文件描述符、任何错误代码的标识，以及被打开的文件的路径。

`opensnoop` 通过在 `open()` 和 `openat()` 这两个系统调用上面附加 eBPF 程序来实现的，任何应用都必须要向内核请求这两个系统调用才能打开一个文件。让我们继续看看这个是怎么实现的。为了简单起见，我们不会逐行去看这些代码，但是我希望这些能足够让你了解这是怎么工作的（如果你对这些细节没有什么兴趣，也可以跳到下个章节）。

#### P18. Opensnoop eBPF Code
这个 eBPF 代码是用 C 编写的，代码在 `opensnoop.bpf.c` 。在靠近这个文件最开始的地方，你能看到两个 eBPF maps 的定义，`start` 和 `events`：

```c
struct {
	__uint(type, BPF_MAP_TYPE_HASH);
	__uint(max_entries, 10240);
	__type(key, u32);
	__type(value, struct args_t);
} start SEC(".maps");

struct {
	__uint(type, BPF_MAP_TYPE_PERF_EVENT_ARRAY);
	__uint(key_size, sizeof(u32));
	__uint(value_size, sizeof(u32));
} events SEC(".maps");
```

当这个 ELB 目标文件生成的时候，里面有了一个段（section），其中包含了所有会被加载到内核中的 map 和程序，代码中的宏 `SEC()` 就用来定义这些段。

当我们接着看这个程序时，你马上就会看到，`start` 这个 map 被用来暂时存储 syscall 的参数，其中包括了当这个 syscall 正在被调用的时候，即将被打开的文件名字。 `events` 这个 map 是用来把事件的信息从内核中的 eBPF 代码传递到用户空间的可执行程序那边去。图 3-2 展示了整个过程。

![](./Pasted%20image%2020220521181919.png)

`opensnoop.bpf.c` 这个文件的后面，你还能找到两个非常相似的函数：

```c
SEC("tracepoint/syscalls/sys_enter_open")
int tracepoint__syscalls__sys_enter_open(
	struct trace_event_raw_sys_enter* ctx
)
```

和

```c
SEC("tracepoint/syscalls/sys_enter_openat") 
int tracepoint__syscalls__sys_enter_openat(
	struct trace_event_raw_sys_enter* ctx
)
```

有两个不同的系统调用可以用来打开文件：`openat()` 和 `open()` 。他们差不多是完全一样的，除了 `openat()` 有一个额外的参数用来表示目录文件描述符，同时即将被打开的文件的路径名是相对于这个目录的相对路径。类似的，`opensnoop` 中的两个函数也几乎是一样的，除了处理这些不同参数的部分之外。

正如你所看到的，这两个函数都接收一个参数，这个参数是一个指向一个叫做 `trace_event_raw_sys_enter` 的结构体的指针。你可以在你正在运行的特定内核生成的 `vmlinux` 头文件中找到这个结构体的定义。编写 eBPF 程序的方法包括了确定每个程序会接收到的上下文的结构体，以及如果才能访问到结构体中信息。

这两个函数使用了一个 BPF 辅助函数来获取调用这个系统调用的进程的 ID：

```c
u64 id = bpf_get_current_pid_tgid();
```

（下面）这段代码可以获取到文件名，以及任何传入到这个 syscall 中的 flags ，然后把这些都存到一个叫做 `args` 的结构体中:

```c
args.fname = (const char *)ctx->args[0];
args.flags = (int)ctx->args[1];
```

这个结构体是写在 `start` 这个 map 中的，（这个 map）用当前进程的 ID 作为 key：

```c
bpf_map_update_elem(&start, &pid, &args, 0);
```

这就是这个 eBPF 程序在这个有人调用（进入）这个 syscall 时做的所有的事情了（do on entry to the syscall）。但是 `opensnoop.bpf.c` 里面还有另外一对 eBPF 程序，他们会在这个 syscall 退出的时候被触发：

```c
SEC("tracepoint/syscalls/sys_exit_open")
int tracepoint__syscalls__sys_exit_open
```

这个程序（指上面的 `tracepoint__syscalls__sys_exit_open`）还有 `openat()` 对应的函数 （指实际代码中的 `tracepoint__syscalls__sys_enter_openat`）共用了函数 `trace_exit()` 中同样的代码。你有没有注意到， eBPF 程序调用的所有的函数，都有一个 `static __always_inline` 的前缀？这会迫使编译器把这些函数中的代码指令内联（forces the compiler to put the instructions for these functions inline），因为在比较老的内核中， BPF 程序时不允许跳到其他的函数中去的（not allowed to jump to a separate function）。比较新的内核，还有比较新版本的 LLVM 是支持非内联的函数调用，不过（内联编译）那种做法肯定是能确保 BPF 验证器满意（a safe way to ensure the BPF verifier stays happy，可能是指 BPF 验证器不会报错或者肯定能通过验证的意思）。（现在还有一种叫做 BPF 尾调用（BPF tail call）的概念，可以让执行过程从一个 BPF 程序跳到另一个 BPF 程序。你可以在 eBPF 文档中学到更多关于 BPF 函数调用和尾调用的信息）。

`trace_exit()` 这个函数创建了一个空的事件结构体（an empty event structure）：

```c
struct event event = {};
```

这里面会被填充那些有关 `open/openat` 系统调用的信息，这些信息之后会被总结，然后通过 `events` 这个 map 发到用户空间。

在 `start` 这个 hash map 会有一个条目，对应这当前进程的 ID：

```c
ap = bpf_map_lookup_elem(&start, &pid);
```

这（条目， `ap`）里面存着之前 `sys_enter_open(at)` 系统调用过程中记录的文件名（filename）和标志位（flags）这些信息。 标志位（flags）是一个整型数据，直接存在这个结构体里面，所以直接从结构体里面读取数据是没问题的：

```c
event.flags = ap->flags;
```

然而，文件名（filename）是写在用户空间内存中的一些字节数据，验证器需要确保对于 eBPF 程序来说，从这个位置的内存中读取这些字节数据是安全的。这是通过另一个辅助函数 `bpf_probe_read_user_str()` 来实现的：

```c
bpf_probe_read_user_str(&event.fname, sizeof(event.fname), ap->fname);
```

当前命令的名字（就是说，调用了 `open(at)` 系统调用的可执行程序的名字）同样也被复制进了事件结构体（event structure）里面，用的是另外一个 BPF 辅助函数：

```c
bpf_get_current_comm(&event.comm, sizeof(event.comm));
```

这个事件结构体会被写到 `events` 这个 perf buffer map 里面：

```c
bpf_perf_event_output(ctx, &events, BPF_F_CURRENT_CPU, &event, sizeof(event));
```

用户空间的代码可以从这个 map 里面读取到事件信息。在我们讲这一步之前，让我们先简单看一下这个 Makefile 。

#### P21. libbpf-tools Makefile
当你想要编译 eBPF 代码的时候，你会得到一个包含了这个 eBPF 程序还有映射 maps 的二进制定义的目标文件（an object file containing the binary definitions of the eBPF programs and maps）。同时你还需要一个额外的用户空间的可执行程序，用来把这些 eBPF 程序和 maps 加载进内核里，然后作为用户可以使用的一个接口。让我们看看用来编译 `opensnoop` 的 Makefile 文件，看下他是怎么构建 eBPF 目标文件还有这个可执行程序的。

Makefiles 包含了一组规则，这些规则的语法看起来会有些不太清晰，所以如果你不是很熟悉 Makefiles 然后你又不是特别关注这些细节，你也可以直接跳过这一部分！

我们刚才看的 `opensnoop` 这个例子是一个很大的实例工具集（a large set of example tools）的其中一个，这些都是用一个 Makefile 来编译的，你可以在 libbpf-tools 的目录下面找到这个文件。这个文件里面不是所有的内容都特别重要，不过有一些规则我还是要重点标注一下。第一个规则是把用 clang 编译器把 `bpf.c` 文件编译成 BPF 目标文件（create a BPF target object file）：

```Makefile
$(OUTPUT)/%.bpf.o: %.bpf.c $(LIBBPF_OBJ) $(wildcard %.h) $(AR..
	$(call msg,BPF,$@)
	$(Q)$(CLANG) $(CFLAGS) -target bpf -D__TARGET_ARCH_$(ARCH) \
		-I$(ARCH)/ $(INCLUDES) -c $(filter %.c,$^) -o $@ && \
	$(LLVM_STRIP) -g $@
```

在这里面，`opensnoop.bpf.c` 被编译成了 `$(OUTPUT)/opensnoop.bpf.o` 。这个目标文件包含了之后会被加载到内核中的 eBPF 程序还有 maps 信息。

另一个规则是 `bpftool gen skeleton` ，这是用来根据目标文件 `bpf.o` 中包含的 map 和程序定义，创建一个骨架头文件（skeleton header file）：

```Makefile
$(OUTPUT)/%.skel.h: $(OUTPUT)/%.bpf.o | $(OUTPUT)
	$(call msg,GEN-SKEL,$@)
	$(Q)$(BPFTOOL) gen skeleton $< > $@
```

`opensnoop.c` 这个用户空间的代码里面，会包含 `opensnoop.skel.h` 这个头文件，用来拿到他（用户空间代码）与内核中的 eBPF 程序共享的 maps 的相关定义。这就让用户空间代码和内核代码都知道了存储在这些 maps 里面的那些数据结构的布局（layout of the data structures）。

接下来的一个规则会把用户空间的代码从 `opensnoop.c` 编译成叫做 `$(OUTPUT)/opensnoop.o` 这个二进制文件：

```Makefile
$(OUTPUT)/%.o: %.c $(wildcard %.h) $(LIBBPF_OBJ) | $(OUTPUT)
	$(call msg,CC,$@)
	$(Q)$(CC) $(CFLAGS) $(INCLUDES) -c $(filter %.c,$^) -o $@
```

最后，还有一个规则用 `CC` 把用户空间的应用程序文件（在这个例子里就是 `opensnoop.o`）和一些可执行程序链接到一起：

```Makefile
$(APPS): %: $(OUTPUT)/%.o $(LIBBPF_OBJ) $(COMMON_OBJ) | $(OUT...
	$(call msg,BINARY,$@)
	$(Q)$(CC) $(CFLAGS) $^ $(LDFLAGS) -lelf -lz -o $@
```

现在既然你已经看过 eBPF 和用户空间程序是怎么分别生成的，那接下来让我们看看用户空间的代码。

#### P23. Opensnoop User Space Code
我之前提到过，会和 eBPF 代码交互的用户空间代码，可以用几乎任何编程语言来写。在这一部分我们举的例子里面使用 C 来写得，但是如果你感兴趣的话，你可以和最初那个用 Python 写的 BCC 版本比较一下，那个版本你可以在 `bcc/tools` 里面找到。

用户空间的代码在 `opensnoop.c` 文件里面。最开始的一部分有 `#include` 指令（其中一个是之前自动生成的 `opensnoop.skel.h` 文件），不同的一些定义，以及用来处理不同命令行选项的代码，这一部分我们在这里不会涉及。我们先忽略例如 `print_event()` 这种把事件信息输出到屏幕上的这种函数。从 eBPF 的角度出发，所有有趣的代码都在 `main()`  函数中。

你会看到像是 `opensnoop_bpf__open()` 、 `opensnoop_bpf__load()` 和 `opensnoop_bpf__attach()` 这种函数。这些都被定义在了 `bpftool gen skeleton`指令自动生成的代码里面。这些自动生成的代码处理了 eBPF 程序、 maps ，以及定义在 eBPF 目标文件中的附加点（attachment points）。

一旦运行 `opensnoop` ，他的工作就是监听 `events` 这个 perf buffer ，然后把每一个事件中的信息写到屏幕上。首先，他会打开和 perf buffer 关联的文件描述符，然后把 `handle_event()` 设置成当一个新事件出现时会被调用的函数：

```c
pb = perf_buffer__new(
	bpf_map__fd(obj->maps.events),
	PERF_BUFFER_PAGES, handle_event, handle_lost_events, 
	NULL, NULL
);
```

然后，他会轮询这个缓冲区事件（buffer events），直到达到了时间限制，或者是用户中断了这个程序：

```c
while (!exiting) {
	err = perf_buffer__poll(pb, PERF_POLL_TIMEOUT_MS);
	...
}
```

传到 `handle_event()` 的数据参数（data parameter）指的是 eBPF 程序为了当前这个事件，写入到 map 里面的事件结构体（points to the event structure that the eBPF program wrote into the map for this event）。用户空间的代码可以获取这些信息，对它们进行组合，然后打印出来展示给用户看。

跟你看到的一样， `opensnoop` 注册了一个 eBPF 程序，每一次任何应用程序调用了 `open()` 或者 `openat()` 的时候，这个 eBPF 程序就会被调用。这些运行在内核中的 eBPF 程序在收集这两个系统调用的上下文信息（information about the context of that system call）——可执行程序的名字和进程 ID ——以及被打开的文件的信息。这些信息被写进了一个 map 里面，用户空间的代码可以从里面读到数据并且展示给用户。

你可以在 libbpf-tools 目录下面找到几十个像这样的 eBPF 工具的例子，每一个例子通常都包含了一个系统调用，或者是一系列像是 `open()` 和 `openat()` 这样的系统调用。

系统调用是一种稳定的内核接口，他们提供了一种非常强大的途径让你可以观察（虚拟）机器上正在发生什么。但是你不要误以为 eBPF 编程只能拦截系统调用。我们还有很多其他稳定的接口，包括 LSM （Linux Security Module）以及网络栈中的不同点（various points in the networking stack）， eBPF 也可以附加到这些接口上面。如果你愿意接受风险，或者愿意在不同内核版本之间的做修改，那么你能够附加 eBPF 程序的范围是非常非常大的。

---

### Chapter 4 eBPF Complexity
你已经看到过一个 eBPF 编程的例子，让你了解了一下它是怎么工作的。虽然基础的例子让 eBPF 看起来好像相对来说是比较简单的，但是还是有一些复杂的地方，让编写 eBPF 变得很有挑战性。

历来让编写和发布 eBPF 程序变得比较困难的其中一个方面，就是内核兼容性。

#### P25. Portability Across Kernels
eBPF 程序可以访问内核的数据结构，这些数据结构可能会随着不同内核的版本而发生改变。这些结构体本身是定义在头文件里面，这些头文件又是 Linux 源代码的一部分。在过去，你必须要使用与你打算运行这些程序的内核相兼容的，正确的头文件集来编译你的 eBPF 程序。

#### P25. BCC Approach to Portability
为了解决跨内核的可移植性， BCC （BPF Compiler Collection）项目采取的是在目标机器上，在运行时就地编译 eBPF 代码这种方式。这意味着，编译工具链（compilation toolchain）需要在每一个你打算运行代码的目标机器上面都安装好，然后你还需要等待编译结束之后才能使用你的工具。同时你还要祈祷文件系统里面有内核头文件（有的时候不一定有）。用 BOF CO-RE 吧。

#### P26. CO-RE
CO-RE（compile once, run everywhere） —— 编译一次，到处运行 —— 这种方法包括了下面这些：
- BTF （BPF Type Format）
	- 这是表示数据结构和函数签名布局（layout of data structures and function signatures）的一种格式。现代 Linux 内核是支持 BTF 的，所以你可以在正在运行的一个系统上面，生成一个叫做 `vmlinux.h` 的头文件，这里面包含了 BPF 程序有可能需要的，关于这个内核的所有数据结构的信息。

- libbpf ，BPF 库
	- 另一方面，libbpf 提供了用来把 eBPF 程序和映射 maps 加载到内核中所需的函数。但是他同样在可移植性上有重要的作用：它依赖于 BTF 信息来调整 eBPF 的代码，以此来补偿这些在编译时产生的数据结构，和在目标机器上面的数据结构之前的任何差异。

- Compiler support 编译器支持
	- `clang` 编译器被专门加强过，所以当你用它来编译 eBPF 程序时，他会把 BTF 重定位（BTF relocations）也包含进去，这是 `libbpf` 用来知道它在把 BPF 程序和 maps 加载到内核中的过程中需要调整什么

- a BPF skeleton，可选的
	- 骨架（skeleton）可以从一个用 `bpftool gen skeleton` 命令编译好的 BPF 目标文件中自动生成得到，这里面包含了那些用户空间的代码可以调用，用来管理 BPF 程序生命周期的有用的函数 —— 把他们（BPF 程序）加载进内核中，把他们附加到事件上，以及其他的一些操作。这些函数是更高阶的抽象，相比于直接使用 libbpf，可以帮助开发者更方便地进行开发。

你可以读 Andrii Nakruiko 这篇很好的文章来了解 CO-RE 更详细的解释。

`vmlinux` 文件中的 BTF 信息都被包含在了从 Linux 内核 5.4 版本开始的版本里面，但是 libbpf 可以使用的原始的 BTF 数据（raw BTF data），同样也可以生成出来，给老版本的内核使用。放在 BTF Hub 上面，有教你如何生成 BTF 文件的信息，还有给不同 Linux 发行版本使用的存档文件。

BPF CO-RE 这个方法让 eBPF 程序员相比过去，能够更加容易地让他们的代码可以运行在任何 Linux 发行版上面 —— 至少能运行在那些足够新的，能够支持他们的程序所需要的任何一组 eBPF 功能的 Linux 发行版上面（on any Linux distribution new enough to have support for whatever set of eBPF capabilities their program uses）。但是这其实也没让 eBPF 编程变得有多容易，他本质上其实还是内核变成。

#### P27. Linux Kernel Knowledge
很明显，为了能编写更高级的工具，你需要一些关于 Linux 内核这个领域的知识。你需要了解你能访问的数据结构，这个取决于调用你的 eBPF 代码的上下文（depend on the context in which your eBPF code is called）。并不是所有的应用开发者都有处理网络流量包、访问 socket buffers 、或者处理系统调用的参数的经验的。

内核会对你写的 eBPF 代码做出什么样的反应？正如你在第二章里面学习的那样，内核里包含了上百万行代码。他们的文档是比较少的，所以你到后来可能会发现你还是需要去阅读内核源码，才能搞清楚某些部分到底是怎么运作的。

同样你也需要搞清楚，你的 eBPF 代码到底要附加到哪些事件上面（what events your eBPF code should attach to）。现在你有能力在整个内核的任何函数入口点上面附加 kprobe ，对你来说可能并没有那么好选。有的时候做这种选择是非常简单的 —— 比如说，如果你想要访问即将进来的网络数据包，那么很显然你应该选择在合适的网络接口上用 XDP hook （the XDP hook on the appropriate network interface）。如果你想要对特定的内核事件进行观测，那么在对应的内核代码中找到合适的附加点（find the appropriate point within the kernel code）估计也不算是特别难的事情。

但是在其他情况下，这种选择可能就没那么明显了。比如说，有些工具会简单地用 kprobes 来 hook 调用那些用来组成内核系统调用接口的函数（functions that make up the kernel's syscall interface），这种做法可能会受到一种叫做 *time-of-check to time-of-use* （TOCTTOU）的安全漏洞的攻击。攻击者有一定几率可以在 eBPF 代码读取系统调用的参数之后、但是还没有被复制到内核内存之前（after the eBPF code has read them, but before they have been copied into kernel memory），修改系统调用的参数。 Rex Guo 和 Junyuan Zeng 在 DEF CON 29 会议上有一次非常精彩的演讲。有一些现在非常常用的 eBPF 工具其实是用非常简单地方法（in quite a naive way）编写的，这种编写方法其实是会收到这种攻击的。这种攻击其实也不是很容易，同时我们也有很多办法来缓解这些攻击（mitigate these attacks），但是如果你正在保护非常非常敏感的数据，不受那些非常有经验的、有动力的攻击者的攻击的话，请你一定要深入地了解你正在用的工具是不是会收到这种攻击（TOCTTOU）的影响。

你已经看到 BPF CO-RE 是如何让 eBPF 程序能够在不同内核版本上运行的了，但是这只考虑了数据结构布局方面的不同（only takes into account the changes in data structure layout），而没有考虑到内核行为这方面更多的不同。比如说，如果你想把一个 eBPF 程序附加在内核的一个特定的函数或者跟踪点上面，你其实需要一个 Plan B 来确定，如果说这个函数或者跟踪点在不同的内核版本上不存在的话，这个 eBPF 程序该怎么做。

#### P28. Coordinating Multiple eBPF Programs
现在我们能用的许多的基于 eBPF 的工具提供了很多的观测能力，都是通过在许多内核事件上面用 eBPF 程序进行 hook 来实现的。这其中大部分工具都是 Brendan Gregg 和其他人在 BCC 项目和 `bpftrace` 工具中所做的工作开创的。现在的工具（一般都是商用工具）可能会提供更加美观的图像和 UI ，但是他们使用的 eBPF 程序基本上都是高度基于那些原始的工具（指 Brendan Gregg 他们开发的那些工具）。

当你想要用写代码的方式，来协调不同种类的事件之间的交互时（coordinates interactions between different types of events），这件事会变得非常非常复杂。举个例子来说，Cilium 会通过内核网络栈上的许多点来查看网络流量包（sees network packets at a variety of points through the kernel's networking stack），然后根据 Kubernets CNI （container network interface，容器网络接口）提供的关于 Kubernets pods 的信息，对网络流量进行修改。想要构建这样的一个系统，需要 Cilium 的开发者对内核如何处理网络流量有一个非常深的理解，还要对用户空间中的 “pods” 和 “containers” 这些概念是怎么对应到内核中像是 “cgroups” 和 “namespaces” 这一类的概念，也要有很深的理解。实际上，有些 Cilium 维护者同时也是内核的开发者，他们致力于对 eBPF 和网络的支持，因此他们都具备这方面的知识。

这里的底线是，尽管 eBPF 提供了一个非常高效和强大的平台，来帮助你 hook 内核，但是对于没有多少内核开发经验的普通开发者而言，这也不简单。如果你有兴趣上手尝试 eBPF 编程的话，我强烈建议你把它当作是一种学习练习：在这个领域里面积累经验是非常有价值的，因为在未来的几年之内，这肯定是一项很受欢迎的专业技能。但是说实话，大多数的组织其实都不太可能自主研发出多少定制的 eBPF 工具（unlikely to build much bespoke eBPF tooling in-house），取而代之的是他们会利用由专业的 eBPF 社区所开发的项目和产品。

我们接下去来想想为什么这些基于 eBPF 的项目和产品在云原生环境里（in a cloud native environment）这么强大。

### Chapter 5 eBPF in Cloud Native Environments
这几年，云原生计算的发展突飞猛进。在这一章里，我们会讨论为什么 eBPF 这么适合开发适用于云原生环境的工具。为了让说明更加具体，我们会专门讲 Kubernetes （k8s），但是这些概念其实也适用于其他使用容器的平台。

#### P31. One Kernel per Host
为了理解为什么 eBPF 在云原生世界里面这么强大，你需要非常清楚这么一个概念：每一台机器（或者是虚拟机）都只有一个内核，所有运行在这个机器上面的容器都在共享这一个内核，就像图 5-1 展示的那样。运行在任何一个指定的主机上面的所有应用程序代码，用的都是同一个内核，这个内核同样也能感知到所有的这些应用程序代码（The same kernel is involved with and aware of all the application code running on any given host machine）。

![](./Pasted%20image%2020220521230936.png)

通过观测这个内核，就像我们用 eBPF 的时候做得那样，我们可以同时观测运行在那台机器上的所有应用程序代码。当我们在内核里加载一个 eBPF 程序，并把它附加到一个事件上的时候，不管是哪一个进程涉及到了这个事件，这个 eBPF 程序都会被触发。

#### P32. eBPF Versus the Sidecar Model
在 eBPF 之前，k8s 用的最多的观测和安全工具，都是用的 sidecar 模型。这个模型允许你在应用的那个 pod 里面，部署一个单独的容器用来进行观测。当这个方法被发明出来的时候，这算是一种很大的进步了，因为这意味着你不必再往 app 里面直接添加用来进行观测的代码了。简单地部署一个 sidecar ，你就可以对同一个 pod 中的其他容器进行观测了。注入 sidecars 的过程（the process of injecting sidecars）通常都是自动化的，所以这也提供了一种机制可以让你所有的 app 都能被观测到。

每一个 sidecar 容器都会消耗资源，而且这种消耗会根据注入 sidecar 的 pod 的数量而呈倍数增长。这种增长是非常明显的 —— 比如说，如果每一个 sidecar 他自己都需要复制一份路由信息或者是策略规则（needs its own copy of routing information, or policy rules），这其实是很浪费的。（你可以看看 Thomas Graf 写得这篇 “comparison of sidecars with eBPF for service mesh” 来了解更多细节）。

用 sidecars 的另一个问题是，你没办法保证这个机器上的所有应用都被正确地观测到。想象一下，假设有个攻击者成功拿下了你手里的一台主机，然后新建了一个单独的 pod 来运行一个加密矿机。他们基本上不可能很“礼貌地”让你用你的 sidecar 的观测和安全工具来观测他们的矿机 pod 。你需要一个单独的系统来发现这种活动（need a separate system to be aware of this activity）。

但是这个加密矿机和运行在这台主机上的其他合法的 pods 都在共享同一个内核。如果你用基于 eBPF 的观测工具，那这个矿机实际上是自动地就被发现了的，就像在图 5-2 里面展示的那样。

![](./Pasted%20image%2020220521232556.png)

#### P33. eBPF and Process Isolation
除了用这种每一个 pod 里都要安装一个 sidecars ，我主张的是把这些功能（应该是指 observability and security tools  观测和安全的能力）都整合进一个安装在单个 node 节点中的，基于 eBPF 的代理 agent 里面（a single per-node, eBPF-based agent）。如果这个代理能够访问运行在这个机器上的所有 pods ，这会不会是一种安全风险？我们有没有丢失应用之间的那种，用来防止他们互相干扰的隔离呢？

对那些已经花了很多时间在容器安全方面工作的人来说，我可以理解他们的这些担忧，但是深入到底层机制来真正地理解为什么这么做其实并不像最初看起来的那样是错的，这还是非常有必要的。

你要记住的很重要的一件事情是，所有的容器都共用同一个内核，但是这个内核对这些 pods 或者 containers 其实没有一种天生的理解（innate understanding of pods or containers，指的是内核并不知道什么是 pod 和容器，这些概念都对应到内核的 cgroups 和 namespaces 一类的概念）。取而代之的，内核操作的是进程，用的是 cgroups 和 namespaces 来把各个进程相互隔离。这些结构阻止用户空间的进程能够相互查看或者相互干扰，这都是由内核来控制的。一旦数据在内核里面被处理的时候（比如说数据是从磁盘里读取的，或者数据被发往网络去），你其实是依赖于内核现在是正常运行的（relying on the kernel behaving correctly）。只有内核自己的代码才能说，比如说，内核我需要遵守文件权限。没有其他额外的充满魔法的权限层（no extra magic authoritative layer）来阻止内核无视文件权限，进而读取任何他想要访问的文件中的数据 —— 仅仅只是因为内核他本身代码就是这么写的，不让他这么做而已。

Linux 系统里的安全控制假设了内核本身是可以信任的。这些安全控制的存在是为了防止用户空间里的代码产生不好的行为（protect against bad behavior from code running in user space）。

我们在第二章里面看到， eBPF 验证器确保了任何 eBPF 程序只能够去访问他应该访问的内存。这个验证器会检查程序没有办法离开他所能访问的范围（can't possibly go beyond its remit），包括确保这块内存（指 eBPF 程序想要访问的内存）是被当前进程所拥有的，或者这块内存是属于当前网络数据包中的一部分。这意味着， eBPF 代码相比于附近的内核代码，其实是受制于更加严格的控制之中，附近的内核代码其实是不经过任何类型的验证步骤的。

如果一个攻击者从容器化的应用中成功逃逸到 node 节点上，然后又能够进行提权（escalate privileges），那这个攻击者就能拿下同一个节点上的其他应用程序。因为这种逃逸是未知的（since those escapes are not unknown），作为容器安全专家，我其实是不建议在共享的机器上面，和那些没办法信任的应用程序，或者是没有安全额外安全工具的用户一块儿，运行比较敏感的应用程序。对于那些更加敏感的数据，你甚至是最好都别把你的应用和那些无法信任的用户一起运行在同一个物理机上的同一个虚拟机里面。不过如果你准备好把你的应用并行运行（run applications side-by-side）在同样的虚拟机里面（对很多应用来说，这其实是合理的，因为它们也不是非常敏感），那么 eBPF 相比于共用同一个内核，其实是不会增加其他的风险的。

当然了，一个恶意的 eBPF 程序会造成各种各样的灾难，而且写一个做坏事的 eBPF 程序也是很容易的 —— 比如说，把所有的网络数据包都拷贝一份然后发给一个窃听者。默认情况下，非 root 用户是没有权限加载 eBPF 程序的，你应该只在你信任某个用户或者软件系统的情况下，给他们这个权限，就跟你给 root 权限其实是一样的。所以，你必须要小心你要运行的代码的出处（现在也有一项倡议是要支持 eBPF 程序的签名检查，来避免上述的问题）。你也可以用一个 eBPF 程序来监控另外的 eBPF 程序！

既然你已经大致了解了为什么 eBPF 对于云原生观测来说是这么强大的基础，下一章会给你看下云原生系统中的一些具体地 eBPF 工具的例子。

---

### Chapter 6 eBPF Tools
既然你已经学到了 eBPF 是什么，也了解了 eBPF 程序是怎么工作的，那让我们来看些用这项技术构建的具体的工具，这些工具说不定你今天就会在产品部署过程中用到。我们会讲到一些开源的基于 eBPF 的项目，他们都提供了三个重要领域中的功能：网络、观测和安全。

#### P35. Networking
eBPF 程序可以被附加到网络接口上，以及内核网络栈上的许多附加点上面。在每一个附加点上， eBPF 程序可以丢弃数据包、把他们发往不同的目的地，甚至是修改其中的内容。这能够做到许多强大的事情。让我们看一些现在常用 eBPF 实现的网络功能。

#### P35. Load Balancing
如果你对于 eBPF 在网络方面的扩展性有任何质疑的话，你要知道 eBPF 在 Facebook 已经大规模使用了。他们是 BPF 的早期使用者，在 2018 年他们引入了 Katran ，这是一个开源的四层负载均衡。

另外一个大规模使用负载均衡的例子是 Cloudflare 的 Unimog edge 负载均衡器。运行在内核中的 eBPF 程序可以修改网络数据包，并把他们发往合适的目的地，就不需要每一个数据包都要经过网络数据栈然后再发往用户空间。

Cilium 项目更广为人知的名字是 eBPF Kubernetes 网络插件（我稍后会讨论这个），但是他也被用在大型电信以及在内部作为独立的负载均衡使用。同样的，能够在早期阶段处理数据包，而不需要把他们传到用户空间的这个能力，让处理网络包变得非常高效。

#### P36. Kubernetes Networking
CNCF 项目 Cilium 最早是一个基于 eBPF 的 CNI 容器网络接口的实现。他最早是由一组致力于 eBPF 的内核维护者发起的，他们意识到了 eBPF 在云原生网络方面的潜能。现在 Cilium 已经是作为 Google Kubernetes Enginer ， Amazon EKS Anywhere ， 和 Alibaba Cloud 的默认数据平面（the default data plane）来使用的了。

在云原生世界里， pods 一直在不停的停止和启动，每一个 pod 都会得到一个 IP 地址。在使用开启 eBPF 的网络（eBPF-enabled networking）以前，每一个节点都必须要为这些变化（指 pods 不停关闭和启动）一直更新一组 iptables 规则，为的是能够在这些 pods 之间进行路由，当规模变大时，管理这些 iptables 规则就会变得很笨重。就像图 6-1 里展示的那样， Cilium 极大地简化了路由，把路由这个东西本质上变成了在 eBPF 中的一张查找表（a simple lookup table in eBPF），带来了显著的性能改进（measurable performance improvements）。

![](./Pasted%20image%2020220522004426.png)

![](./Pasted%20image%2020220522004433.png)

另一个除了传统的 iptables 版本之外，还提供了 eBPF 实现的 Kubernetes CNI 是 Calico 项目。

#### P38. Service Mesh，服务网格
eBPF 作为基础，为 Service Mesh 提供的更高效的数据平面（a more efficient data plane）也很有意义。许多 service mesh 的特性是操作在七层应用层上，并且使用例如 Envoy 的代理组件来代表应用进行操作。在 Kubernetes 里面，这种代理通常是用 sidecar 模型部署的，每一个 pod 里面有一个代理容器，这样这个代理就能够访问 pod 所在的网络命名空间（network namespace）。就像你在第五章里看到的， eBPF 相比 sidecar 模型是一种更高效的方式。因为内核可以访问所有 pod 的 namespaces ，我们可以用 eBPF 在 pods 中的应用程序和主机上的单个代理之间创建连接，就像图 6-2 展示的那样。

![](./Pasted%20image%2020220522005414.png)

我有另外一篇文章，讲的是用 eBPF 来做更高效的 service mesh 数据平面，叫做 Solo.io 。在我写这篇的时候， Cilium Service Mesh 已经在 beta 测试了，并且在性能表现上已经超出了超过传统 sidecar 代理方式。

#### P39. Observability
就像你在这篇报告的前面看到的， eBPF 程序其实可以看到发生在这台机器上的任何事情。通过收集事件的数据并把他们传到用户空间， eBPF 支持一系列强大的观测工具，这些工具能向你展示你的应用程序的现在的表现是怎么样的，而且这些都不需要你对应用程序做任何的改动。 eBPF 同样也能对整个系统而不是仅仅只能对单一应用进行观测，所以你就能够了解你的主机的表现了。

在这篇报告前面你已经了解到 BCC 项目，那过去几年里面 Brendan Gregg 在 Netflix 完成了开创性的工作，像我们展示了 eBPF 工具是怎么样在大规模的情况下，能够高效地观测到几乎是任何你感兴趣的数据。

Kinvolk 的 Inspektor Gadget 把一些源于 BCC 项目中的工具引入到 Kubernets 中，让你可以很容易地通过命令行来观测特定的工作负载（observe specific workloads）。

新一代的项目和工具在此基础上进行构建，从而提供基于 GUI 的观测能力（就是有 GUI 了）。 CNCF 的 Pixie 项目让你能够运行事先写好的，或者是自定义的脚本，通过一个强大又好看的 UI 界面来查看数据和日志。因为他是基于 eBPF 的，这意味着你可以自动地观测你所有的应用程序，获取到性能数据，又不需要修改任何代码或者配置。图 6-3 展示了 Pixie 提供的许多可视化中的一个例子。

![](./Pasted%20image%2020220522011127.png)

另外一个叫做 Parca 的观测项目主要专注于持续性分析，使用 eBPF 来高效的对例如 CPU 使用率等数据进行采样，这样你就能够使用这些数据来检测性能瓶颈。

Cilium 的 Hubble 组件是一个同时提供命令行接口和 UI 的观测工具（图 6-4 展示的那样），他关注的主要是你的 Kubernetes 集群中的网络流。

![](./Pasted%20image%2020220522011457.png)

在云原生环境中， IP 地址都是持续不断地被动态地分配和重新分配（continually being dynamically assigned and reassigned），传统的基于 IP 地址的网络观测工具使用起来是非常受限制的。作为一个 CNI ， Cilium 能够访问到工作负载的身份信息（workload identity information），这意味着 Hubble 能够展示由 Kubernetes pods 、 services 服务 ，以及 namespaces 一起来标识的服务地图和流量数据（service maps and flow data）。这对于诊断网络问题而言是非常宝贵的。

如果你能够给观测活动（observe activity，应该是指网络活动等），这其实是安全工具的基础，安全工具能够把当前正在发生的现象，与策略或者规则进行比较，从而了解当前这个活动是预期之内的还是可疑的（expected or suspicious）。让我们看看一些使用了 eBPF 来提供云原生安全能力的工具。

#### P41. Security
我们有很多强大的云原生工具，他们通过 eBPF 来检测甚至是防止恶意活动，以此来强化安全性。我会把他们分成两类来讨论：一类是保护网络活动（securing network activity），另一类是在运行时保护应用程序按照预期来运行（securing the expected behavior of applications at runtime）。

#### P41. Network Security
因为 eBPF 允许检查和修改网络数据包，他在网络安全中有很多的作用。最基本的原则是，如果一个网络数据包因为他不满足一些安全检测标准而被认为是恶意的或者是有问题的，那他就可以直接被丢掉（can simply be dropped）。 eBPF 能够很高效地实现这点，因为他可以 hook 内核网络栈中相关的部分，或者是直接 hook 网卡（network interface card，指的应该就是使用 XDP 这种方式）。这意味着策略外的（out-of-policy，应该是指不满足策略的）或者是恶意的数据包可以在传入网络栈并且传递到用户空间，从而产生处理开销之前就被丢弃掉。

早期在生产环境中大规模使用的 eBPF 用法之一，就是 Cloudflare 做得 DDoS 防御。DDoS 攻击会向目标机器发送大量的网络信息，以此希望这个目标机器没办法足够快地处理这些信息，并且由于机器忙于处理这些信息而无法正常地工作。 Cloudflare 的工程师利用 eBPF 程序，在网络数据包到达的那一刻进行检查，快速地判断这个数据包是不是这种 DDoS 攻击的一部分，如果是的话就丢弃他。数据包没必要传到内核网络栈里面，所以他处理起来就会消耗更加少的资源，目标机器也能够给应对速率更高的恶意流量（cope with a much higher rate of malicious traffic）。

eBPF 程序也被用来作为一种针对 “packet of death” 的内核漏洞的动态缓解方式（a dynamic mitigation against "packet of death" kernel vulnerabilities）。攻击者会构造一个网络数据包，构造的方式是利用了一种内核的 bug ，他会组织内核正确地处理数据包。不必等到内核发布对应的补丁，我们可以通过加载 eBPF 程序的方式，在流量中寻找这种特殊构造的数据包并且丢弃掉，以此来缓解这种攻击。这种方式最棒的地方在于， eBPF 程序可以动态地加载，而不需要改变机器上的任何东西。

在 Kubernetes 里面，网络策略（network policy）是一等的资源，但是又需要网络插件来执行网络策略（left to the networking plug-in to enforce it，就是说得让插件来处理策略）。有一些 CNI ，包括 Cilium 和 Calico ，都通过扩展的网络策略能力（extended network policy capabilities）提供更加强大的规则，比如说根据完全限定的域名（by fully qualified domain name），而不是仅通过 IP 地址，允许或者拒绝发往某一个目的地的流量。就像图 6-5 展示的那样，在 app.networkpolicy.io 那里有一个很好的工具，可以用来设计网络规则，同时还能看到这些规则是什么效果。

![](./Pasted%20image%2020220522015132.png)

标准的 Kubernetes 网络策略规则会被应用到进出应用 pod 的流量上面，但是因为 eBPF 拥有能够给查看所有网络流量的能力，eBPF 还可以用来当作主机防火墙（used for host firewall capabilities），用来限制进出主机（虚拟机）的流量。

eBPF 还可以被用来提供透明加密（transparent encryption），不管是通过 WireGuard 还是 IPsec 。这里的透明（transparent）指的是应用程序不需要进行任何修改 —— 事实上应用程序可以压根意识不到他的网络流量被加密了。

#### P43. Runtime Security
eBPF 也被用来构建一些工具，可以用来检测应用程序是什么时候开始用非预期的，或者是恶意的方式运行，同时这里面的有些工具也可以用来防止这一类不良行为。可疑行为的一些例子包括了，预料外的文件访问、运行可执行程序，或者是尝试获取额外权限。

事实上，你很可能已经以 seccomp 的形式，使用了基于 BPF 的安全强制措施（BPF-based security enforcement），这是一个 Linux 特性，用来限制任何应用程序可以调用哪些系统调用（limiting the set of syscalls that any application can call）。

Falco 这个 CNCF 项目扩展了这种限制应用程序可以执行的 syscall 的概念。 Falco 的规则定义是用 YAML 形式定义的，相较于 seccomp 的 profiles 文件来说， YAML 文件对人而言更容易阅读和理解。默认的 Falco 驱动是一个内核模块，不过也有一种 eBPF probe 驱动可以附加在 “raw syscall” 事件上面。他不会阻止这些 syscalls 执行完毕，但是他会生成日志或者是提示（generate logs or other notifications）来提醒操作者有一个潜在的恶意活动发生了。

就像我在第三章里面看到的那样， eBPF 程序可以被附加在 LSM 接口上面，用来阻止恶意行为或者缓解已知漏洞（prevent malicious behaviors or to mitigate known vulneraibilities）。举个例子， Denis Efremov 写了一个 eBPF 程序来阻止 `exec*()` 系统调用在没有任何参数传递进去的情况下运行，这是为了缓解 PwnKit 这个高危漏洞。 eBPF 同样还可以缓解 “Spectre” 这个预测执行方面的漏洞攻击（mitigate against speculative execution "Spectre" attacks）。

Tracee 是另外一个使用了 eBPF 的运行时安全开源项目。和基于 syscall 的检查一样， Tracee 也使用了 LSM 接口。这能够帮助你避免 TOCTTOU 条件竞争漏洞（TOCTTOU race conditions），这个当我们只检查 syscall 的时候还是有可能会发生的。Tracee 支持使用 Open Policy Agent's Rego 语言定义的规则，当然也允许使用 Go 定义的插件。

Cilium 的 Tetragon 组件提供了另外一种强大的能力，可以使用 eBPF 来监控容器安全观测的四项黄金信号（four golden signals fo container security observability）：进程执行，网络 sockets，文件访问，以及七层网络身份（process execution, network sockets, file access, and layer 7 network identity）。这允许操作者能够详细的查看到，到底是什么造成了这个恶意或者可疑的活动，包括了特定 pod 中的可执行程序名称以及用户身份。举个例子，如果你收到了加密矿机的攻击，你就能够给清楚地看到，是哪个可执行程序打开了连到矿池的网络连接，从哪个 pod 里发起的，什么时候发起的。这些取证信息对于理解主机是如何被拿下的是非常宝贵的，也能让你更加容易地设计安全策略，在未来可以防御类似的攻击。

如果你还想更加深入地了解使用 eBPF 进行安全观测这个话题，可以去看一下 Natália Ivánkó 和 Jed Salazar 的这篇报告。你可以密切关注云原生 eBPF 这个话题，因为不久之后我们就能看到利用 BPF LSM 还有其他 eBPF 定制化来实现的安全以及观测的工具。

我们看了一圈在网络、观测和安全方面的云原生工具。相比于之前一代的工具，使用 eBPF 给他们带来了两大优势：
- 从内核方面的优势来看， eBPF 程序对所有的进程都有可见性
- 通过避免内核和用户空间程序之间的传递， eBPF 程序提供了一种极其高效的方式来收集事件信息，或者是处理网络数据包

这并不意味着我们就应该用 eBPF 来做任何事情！用 eBPF 来写那些业务相关的应用程序就没什么意义，就像是我们把应用程序写得跟内核模块那样（没什么意义）。这条规则也有可能会有一些例外，也许对那些像是高频交易一类的，对性能要求极其高的系统（可能有点意义）。正如本篇中所讲的， eBPF 主要是用来编写工具，来观测其他应用程序的，就像我们在这一章里面看到的那些（工具）一样。

### Chapter 7 Conclusion
我希望这篇简短的报告可以给你一些关于 eBPF 的知识和理解，告诉你为什么 eBPF 这么强大。我所希望的是你现在就准备好想要去自己尝试一些基于 eBPF 的工具！

如果你想要在技术层面更加深入的话，ebpf.io 是一个好地方，在那里你可以找到关于更多的技术方面还有 eBPF 基础相关的信息。对于示例代码，我（Liz Rice）在 ebpf-beginners 这个 GitHub 仓库中有一些资源。

想要学习其他人是怎么更好地使用 eBPF 工具的话，你可以加入像是 eBPF Summit 还有 Cloud Native eBPF Day 这样的活动，在那里大家会分享他们的成功经验和知识。这里还有一个很活跃的 Slack 频道 ebpf.io/slack 。我希望能够在那里看到你！
