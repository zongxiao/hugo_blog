---
title: '操作系统进程描述'
date: 2018-03-20 15:25:53
tags: [进程,并发,读书笔记,后端]
published: true
hideInList: false
feature: 
isTop: false
---

> 这是`操作系统进程`系列文章第二篇-操作系统进程描述

## 进程

### 什么是进程

在给进程下定义前，先考虑以下几个概念：

1. 一个计算机平台包括一组硬件资源：比如处理器、内存、I/O 模块、定时器和磁盘驱动器等。
2. 计算机程序是为执行某些任务而开发的。典型情况下，它们接受外来的输入，做一些处理后，输出结果。
3. 直接根据给定的硬件平台写应用程序效率是低下的
4. 开发操作系统是为了给应用程序提供一个方便、安全和一直的接口。操作系统是计算机硬件和应用程序直接的一层软件，对应用程序和工具提供了支持。
5. 可以把操作系统想象为资源的统一抽象表示，可以被应用程序请求和访问。资源包括内存、网络接口和文件系统等。

有了上述概念，现在就可以讨论操作系统怎样以一个有序的方式管理应用程序的执行，以达到以下目的：

* 资源对多个应用程序是可用的
* 物理处理器在多个应用程序间切换以保证所有程序都在执行中
* 处理器和 I/O 设备能得到充分的利用

现代操作系统采用的方法都是`依据对应于一个或多个进程存在的应用程序执行的一种模型`。

关于进程有很多定义：

- 一个正在执行的程序
- 计算机中正在运行的程序的一个实例
- 可以分配给处理器并由处理器执行的一个实体
- 由单一的顺序的执行线程、一个当前状态和一组相关的系统资源所描述的活动单元

### 进程状态

一个被执行的程序，操作系统会为该程序创建一个进程或任务，并且控制进程的执行。

简单来说，程序只有两种状态：`运行态`、`未运行态`。

![两状态进程模型](<http://media.gusibi.mobi/OskbbEAvy2Pml-DAIPGNsq73iTnGmEoG61GC1jFK2S7grZpYiIZxN5rYXw1NvA_3>)

* 当操作系统创建一个新进程时，它将该进程以未运行态加入到系统中，操作系统知道进程的存在，并等待执行机会。
* 当前运行的进程不时中断，操作系统的分派器将选择一个新进程运行。
* 前一个进程从运行态转换到未运行态，另一个从未运行态转换到运行态。

同时，未运行的进程需保持在某种类型的队列中，并等待它们的执行时机。

上图中的排队图可以描述分派器的行为：被中断的进程转移到等待进程队列中，或者，如果进程以及结束或取消，则被销毁。在任何一种情况下，分派器均从队列中选择一个进程来执行。

通过这个模型，可以看出操作系统需要用某种方式来表示每个进程，使得操作系统能够跟踪它，也就是说需要有一些与进程相关的信息，包括进程在内存中的状态和位置，即`进程控制块`。

### 进程控制块

进程在任意时间都可以唯一地被表征为以下元素：

![简化的进程控制块](<http://media.gusibi.mobi/Nra2ykMZsSS2HqGQjASJUXZ7krpqAhCZPSVKc7_G6CUH_kusdOcZpLL2BddET7oJ>)

* 标识符：存储在进程控制块中的数字标识符，包括（次进程的标识符-进程 ID，父进程标识符，用户标识符-用户 ID）
* 状态：进程状态（如运行态，就绪态，等待态等）
* 优先级：用于描述进程调度优先级的一个或多个域。
* 程序计数器：程序中即将被执行的下一条指令的地址
* 内存指针：包括程序代码和进程相关数据的指针，还有和其他进程共享内存块的指针
* 上下文数据：进程执行时处理器的寄存器的数据
* I/O 状态信息：包括显示的 I/O 请求、分配给进程的 I/O 设备和被进程使用的文件列表等
* 记账信息：可能包括处理器时间总和、使用的时钟数总和、时间限制、记账号等。

这些信息被存放在一个叫`进程控制块`的数据结构中，它由操作系统创建和管理。进程控制块是进程存在的唯一标志，也就是说任何一个进程只要进程创建了它就一定有一个跟它相对应的进程控制块，进程结束了进程控制块就会被操作系统回收，进程在执行的过程对进程的所有操作都是通过进程控制块来实现的。

#### 进程创建和终止

进程除运行和未运行外，在进程的生命周期中，创建和终止都是不可避免的。

##### 进程创建

通常有4个事件会导致创建一个进程：

1. 新的批量作业
2. 交互登录。终端用户登录到系统
3. 操作系统因为提供一项服务而创建。操作系统可以创建一个进程，代表用户程序执行一个功能，使用户无需等待。
4. 由现有进程派生。基于模块化的考虑，或者为了开发并行性，用户程序可以指示创建多个进程。

>  当一个进程派生另一个进程时，前一个称为父进程，被派生的被称为子进程。

一旦操作系统决定创建一个新进程，它就会按以下步骤进行：

1. 给新进程分配一个唯一的进程标识符。
2. 给进程分配空间。
3. 初始化进程控制块。
4. 设置正确的连接。（例如，如果操作系统把每个调度队列都保存成链表，则新进程必须放置在就绪或就绪/挂起链表中）。
5. 创建或扩充其他数据结构。

##### 进程终止

有很多事件可以导致进程终止，比如：

1. 进程完成
2. 进程超时。进程运行时间超过规定的时限
3. 无可用内存
4. I/O 失败
5. 算术错误
6. 无效指令
7. 父进程终止
8. 父进程请求
   。。。

#### 五状态模型

系统中还存在着一些处于非运行状态但已经就绪等待执行的进程，而且还存在另一些处于阻塞状态等待 I/O 操作结束的进程。

这时，`就绪态(ready)`和`阻塞态(blocked)`出现了，两状态模型升级为了5状态模型，5个状态如下：

![五状态进程模型](<http://media.gusibi.mobi/TaAMUjZ0MfiJF65CMJc6Z-q5vVlGvMYOAz5UBsfIOTF_HplbJuYKEc2pyfZj9wwJ>)


* 运行态：该进程正在执行
* 就绪态：进程做好了准备，等待处理器调度
* 阻塞/等待态：进程在某些事件发生前不能执行，比如 I/O 操作完成
* 新建态：刚刚创建的进程，操作系统还没有把它加入到可执行进程组中。通常是进程控制块已经创建但还没有被加载到内存中。
* 退出态：操作系统从可执行进程组中释放出的进程，或者是因为它自身停止了，或者是因为某种原因被取消。


> `新建-就绪`: 操作系统准备好再接纳一个进程时，把一个进程从新建态转换到就绪态。大多数系统基于心有的进程数或分配给现有进程的虚拟内存数量设置一些限制，以确保不会因为活跃进程数量过多而导致系统的性能下降。
>
> `就绪-退出`: 在某些系统中，父进程可以在任何时候终止一个子进程。如果一个父进程终止，与该父进程相关的所有子进程都将被终止。

#### 挂起

> `就绪态、运行态和阻塞态`提供了一种为进程行为建立模型的系统方法，但有个问题需要考虑：每个被执行的进程必须完全载入内存，当一个进程在等待 I/O 操作时，处理器可以转移到另一个进程，但 I/O 活动比CPU 计算速度慢很多，因此大多数情况下处理器在多数时候都是空闲的。但是如果内存中都是阻塞态的进程怎么办呢？

* 一种办法就是扩充内存已适应更多的进程
* 另一种方案是把进程中的某个内存的一部分或者全部移到磁盘中。当内存中没有处于就绪态的进程时，操作系统就把被阻塞的进程换出到磁盘中的`挂起队列`，这是暂时保存从内存中被驱逐出的进程队列，或者说是被挂起的进程队列。操作系统在此之后取出挂起队列中的另一个进程，或者接受一个新进程的请求，将其纳入内存运行。


![有挂起态的进程状态转换图](<http://media.gusibi.mobi/RERwAhivDCla0-LTO6wylcXfaA0WFJTFS1oDvMIh9EkzsNrTbRVqu9rWuS9fUmMf>)


> 这里有两个独立的概念：进程是否在等待一个事件（阻塞与否）以及进程是否已经被换出内存（挂起与否）。这里需要4个状态：

* `就绪态`：进程在内存中并可以执行
* `阻塞态`：进程在内存中并等待一个事件
* `阻塞/挂起态`：进程在外存中并等待一个事件
* `就绪/挂起态`：进程在外存中，但是只要被载入内存就可以执行

现在状态转换如下：

> `阻塞`-`阻塞/挂起`：如果没有就绪进程，则至少一个阻塞进程被换出，为另一个没有阻塞的进程让出空间

> `阻塞/挂起`-`就绪/挂起`：如果等待事件发生了，比如 I/O 不再阻塞，则处于阻塞/挂起 状态的进程可以转换到 就绪/挂起状态。

> `阻塞/挂起`-`阻塞`：比如一个进程终止了，释放了一些内存空间，阻塞/挂起队列中有一个进程比 就绪/挂起队列中的任何任何进程的优先级都要高，并且操作系统有理由相信阻塞进程的时间很快就会发生，这时，把阻塞进程而不是就绪进程调入内存是合理的。

### 进程控制

大多数处理器至少支持两种执行模式，某些指令只能在特权态下运行，包括读取或改变诸如程序状态之类控制寄存器的指令，原始 I/O 指令和与内存管理相关的指令。另外有部分内存区域仅在特权态下可以被访问到。

> `特权态`：特权态可称做系统态、控制态或内核态，内核态指的是操作系统的内核。
> `用户态`：用户程序常在该模式下运行

两种模式可以保护操作系统和重要的操作系统表不受用户程序的干涉。

操作系统内核的典型功能：

![操作系统内核的典型功能](http://media.gusibi.mobi/icSKmpD5WmGdRg1YaqotICAvgAbhxbK7fAzcXE1nhVSzua6LhvbeZJGpQdXiKqX-)

#### 进程切换

从表面看，进程切换非常简单。在某一时刻，操作系统中断正在运行的进程，然后指定另一个进程为运行态，并把控制权交给这个进程。但是现在会有几个问题：

* 什么事件触发进程切换
* 模式切换和进程切换的区别
* 进程切换时，操作系统要做哪些工作

`何时切换进程`？

进程切换可以在操作系统从当前正在运行的进程中获得控制权的任何时刻发生。以下是可能把控制权交给操作系统的事件：

![进程执行的中断机制](http://media.gusibi.mobi/Jb4HJnGGvdlYgSmF1goSBFTo2sHE8atDcPaO3yw54AHs39xw8PxXiKrpnKZ7frtn)

 系统中断通常分为两种，一种是`中断`，另一种是`陷阱`。
`中断`与当前正在运行的进程无关的某种类型的外部事件相关，比如 I/O 操作；`陷阱`与当前正在运行的进程锁产生的错误或异常条件相关，比如非法的文件访问。

以下是一些常见的中断事件：

> * `时钟中断`：操作系统确认当前正在运行的进程的执行时间已经超过了最大允许时间段（时间片：即进程在被中断前可以执行的最大时间段），进程必须切换到就绪态，调入另一个进程。
> * `I/O 中断`：进程等待 I/O 活动。
> * `内存失效`：处理器访问一个虚拟内存地址，且次地址单元不在内存中，操作系统必须从外存中把包含这个引用的内存块调入内存中。在发出调入内存块的 I/O 请求之后，操作系统可能会执行一个进程切换，以恢复另一个进程的执行，发生内存失效的进程被置为阻塞态，当前的块调入内存中时，该进程被置为就绪态。

对于`陷阱`,操作系统首先确认错误或者异常是否是致命的。如果是，当前进程被转换到退出态；如果不是，操作系统的动作取决于错误的种类和操作系统的设计（有可能是视图恢复或通知用户）。
操作系统也可能被来自正在执行的程序的`系统调用`激活，比如打开文件，通常，使用系统调用会导致把当前进程置为阻塞态

##### 系统调用

Unix 系统是由用户空间（userland）和内核组成。Unix 内核位于计算机硬件之上，是与摇篮吗交互的中介。这些交互包括通过问卷系统进程读/写、在网络上发送数据、分配内存，以及通过扬声器播放音频。这些都是用户应用程序所不能涉及的，只能通过系统调用来完成。

> `系统调用`为内核和用户空间搭建了桥梁。规定了程序和计算机硬件直接所允许发生的一切交互。

模式切换和进程切换是不同的。发生模式切换可以不改变正处于运行态的进程的状态，而进程被转换到另一个状态操作系统必须使其环境产生实质性的变化。

进程切换步骤如下：
1. 保存处理器上下文环境，包括程序计数器和其他寄存器
2. 更新当前处于运行态进程的进程控制块
3. 将进程的进程控制块移到相应的队列（就绪、挂起等）
4. 选择另一个进程执行
5. 更新所选择进程的进程控制块，包括将进程的状态变为运行态
6. 更新内存管理的数据结构
7. 恢复处理器在被选择的进程最近一次切换出运行态时的上下文环境。

下一篇将介绍 Unix 进程

## 参考

《操作系统-精髓与设计原理》

------


**最后，感谢女朋友支持和包容，比❤️**

想了解以下内容可以在公号输入相应关键字获取历史文章： `公号&小程序` | `设计模式` | `并发&协程`

| 关注 |赞赏 |
|---|---|
|![](http://media.gusibi.mobi/kel2L88yf9YXZYecLIn0LPZPSXc7zJfHyGUz5biWsZrGh7xF2JONZT93dgClGdMn)|![](http://media.gusibi.mobi/VFjjmZ7cgkIkpieAFHYXcLVBB8f9snm2vAzc0GyLjSmCzok8mL3vqLNMzYVvrDha)|