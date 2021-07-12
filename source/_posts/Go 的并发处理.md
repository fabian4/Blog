---
title: Go 的并发处理
date: 2020/12/13
description: Go 的并发处理
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/u9vgyrxMZbcAEG2.png'
categories:
  - go
tags:
  - go
  - 语法
abbrlink: 20817
---

# Go 的并发处理

## 一、Goroutine

在了解Goroutine并发之前，我们需要先了解下进程和线程,并发与并行 (Concurrency and Parallelism)的概念。

* **进程**：进程是操作系统中进行保护和资源分配的基本单位，操作系统分配资源以进程为基本单位。

  cpu在切换程序的时候，如果不保存上一个程序的状态（也就是我们常说的context--上下文），直接切换下一个程序，就会丢失上一个程序的一系列状态，于是引入了进程这个概念，用以划分好程序运行时所需要的资源。因此进程就是一个程序运行时候的所需要的基本资源单位（也可以说是程序运行的一个实体）。

* **线程**：线程是进程的组成部分，它代表了一条顺序的执行流。

  cpu切换多个进程的时候，会花费不少的时间，因为切换进程需要切换到内核态，而每次调度需要内核态都需要读取用户态的数据，进程一旦多起来，cpu调度会消耗一大堆资源，因此引入了线程的概念，线程本身几乎不占有资源，他们共享进程里的资源，内核调度起来不会那么像进程切换那么耗费资源。

  > 进程是从操作系统获得基本的内存空间，所有的线程共享着进程的内存地址空间。此外，每个线程也会拥有自己私有的内存地址范围，其他线程不能访问。然而由于所有的线程共享进程的内存地址空间，所以线程间的通信就非常容易，通过共享进程级全局变量就可以实现线程间的通信。

* **协程**：协程拥有自己的寄存器上下文和栈。
  协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此，协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。线程和进程的操作是由程序触发系统接口，最后的执行者是系统；协程的操作执行者则是用户自身程序，goroutine也是协程。

* **并发**：并发是指程序的逻辑结构,交替做不同事的能力,在这里通常是不同程序交替执行的性能。

* **并行**：并行是指程序的运行状态,同时做不同事的能力,在这里通常是指不同程序同时执行的性能。

如果某个系统支持两个或者多个动作（Action）同时存在，那么这个系统就是一个**并发系统**。如果某个系统支持两个或者多个动作同时执行，那么这个系统就是一个**并行系统**。

并发系统与并行系统这两个定义之间的关键差异在于“存在”这个词。在并发程序中可以同时拥有两个或者多个线程。这意味着，如果程序在单核处理器上运行，那么这两个线程将交替地换入或者换出内存。这些线程是同时“存在”的——每个线程都处于执行过程中的某个状态。如果程序能够并行执行，那么就一定是运行在多核处理器上。

此时，程序中的每个线程都将分配到一个独立的处理器核上，因此可以同时运行。这里相信你已经能够得出结论——“并行”概念是“并发”概念的一个子集。也就是说，你可以编写一个拥有多个线程或者进程的并发程序，但如果没有多核处理器来执行这个程序，那么就不能以并行方式来运行代码。因此，凡是在求解单个问题时涉及多个执行流程的编程模式或者执行行为，都属于并发编程的范畴。

Goroutine 的概念类似于线程，但 Goroutine 由 Go 程序运行时的调度和管理。Go 程序会将Goroutine 中的任务合理地分配给每个 CPU。Go 程序从 main 包的 main() 函数开始，在程序启动时，Go 程序就会为 main() 函数创建一个默认的 Goroutine。

Goroutine 是 Go语言原生支持并发的具体实现，在Go中的代码都是运行在Goroutine中的。Goroutine占用的资源非常小(Go 1.4将每个Goroutine stack的size默认设置为2k)，goroutine调度的切换也不用陷入(trap)操作系统内核层完成，代价很低。因此，一个Go程序中可以创建成千上万个并发的goroutine。

所有的Go代码都在goroutine中执行，即使是go的runtime也不例外。我们可以启动成千上万的goroutine，但是Go的runtime负责对goroutine进行调度。这里的调度就是决定何时哪个goroutine将获得资源开始执行、哪个goroutine应该停止执行让出资源、哪个goroutine应该被唤醒恢复执行等.

### Go的调度器

**Go的调度器内部有三个重要的结构：G P M**

- G: 表示goroutine，存储了goroutine的执行stack信息、goroutine状态以及goroutine的任务函数等；另外G对象是可以重用的。

  ~~~go
  struct G {
      uintptr    stackguard;  // 分段栈的可用空间下界
      uintptr    stackbase;   // 分段栈的栈基址
      Gobuf      sched;       //进程切换时，利用sched域来保存上下文
      uintptr    stack0;
      FuncVal*   fnstart;     // goroutine运行的函数
      void*      param;       // 用于传递参数，睡眠时其它goroutine设置param，唤醒时此goroutine可以获取
      int16      status;      // 状态Gidle,Grunnable,Grunning,Gsyscall,Gwaiting,Gdead
      int64      goid;        // goroutine的id号
      G*         schedlink;
      M*         m;           // for debuggers, but offset not hard-coded
      M*         lockedm;     // G被锁定只能在这个m上运行
      uintptr    gopc;        // 创建这个goroutine的go表达式的pc
      ...
  }
  ~~~

  结构体G中的部分域如上所示。其中包含了栈信息stackbase和stackguard，还有运行的函数信息fnstart。这样就可以成为一个可执行的单元了，只要得到CPU就可以运行。goroutine切换时，上下文信息保存在结构体的sched域中。goroutine是轻量级的线程或者称为协程，切换时并不必陷入到操作系统内核中，所以保存过程很轻量。

  而G中的Gobuf，只保存了当前栈指针，程序计数器，以及goroutine自身。

  ~~~go
  struct Gobuf {
      // The offsets of these fields are known to (hard-coded in) libmach.
      uintptr    sp;
      byte*      pc;
      G*         g;
      ...
  }
  ~~~

  这里g是为了恢复当前goroutine的结构体G指针，运行时库中使用了一个常驻的寄存器extern register G* g，这个是当前goroutine的结构体G的指针。这样做是为了快速地访问goroutine中的信息.

- P:  表示逻辑processor 代表cpu，P的数量决定了系统内最大可并行的G的数量（系统的物理cpu核数>=P的数量）；P的最大作用还是其拥有的各种G对象队列、链表、一些cache和状态。

  在代码中结构体P的加入是为了提高Go程序的并发度，实现更好的调度。M代表OS线程。P代表Go代码执行时需要的资源。当M执行Go代码时，它需要关联一个P，当M为idle或者在系统调用中时，它也需要P。有刚好GOMAXPROCS个P。所有的P被组织为一个数组，在P上实现了工作流窃取的调度器。

  ~~~go
  struct P {
      Lock;
      uint32    status;       // Pidle或Prunning等
      P*        link;
      uint32    schedtick;    // 每次调度时将它加一
      M*        m;           // 链接到它关联的M (nil if idle)
      MCache*   mcache;
  
      G*        runq[256];
      int32     runqhead;
      int32     runqtail;
  
      // Available G's (status == Gdead)
      G*        gfree;
      int32     gfreecnt;
      byte      pad[64];
  }
  ~~~

  在P中有一个Grunnable的goroutine队列，这是一个P的局部队列。当P执行Go代码时，它会优先从自己的这个局部队列中取，这时可以不用加锁，提高了并发度。如果发现这个队列空了，则去其它P的队列中拿一半过来，这样实现工作流窃取的调度。这种情况下是需要给调用器加锁的。

- M: M代表着执行计算资源。在绑定有效的p后，进入schedule循环；而schedule循环的机制大致是从各种队列、p的本地队列中获取G，切换到G的执行栈上并执行G的函数，调用goexit做清理工作并回到m，如此反复。M并不保留G状态，这是G可以跨M调度的基础。

  M是machine的缩写，是对机器的抽象，每个m都是对应到一条操作系统的物理线程。M必须关联了P才可以执行Go代码，但是当它处理阻塞或者系统调用中时，可以不需要关联P。

  ~~~go
  struct M {
      G*       g0;                 // 带有调度栈的goroutine
      G*       gsignal;            // signal-handling G 处理信号的goroutine
      void     (*mstartfn)(void);
      G*       curg;               // M中当前运行的goroutine
      P*       p;                  // 关联P以执行Go代码 (如果没有执行Go代码则P为nil)
      P*       nextp;
      int32    id;
      int32    mallocing;         //状态
      int32    throwing;
      int32    gcing;
      int32    locks;
      int32    helpgc;            //不为0表示此m在做帮忙gc。helpgc等于n只是一个编号
      bool     blockingsyscall;
      bool     spinning;
      Note     park;
      M*       alllink;          // 这个域用于链接allm
      M*       schedlink;
      MCache   *mcache;
      G*       lockedg;
      M*       nextwaitm;       // next M waiting for lock
      GCStats  gcstats;
      ...
  }
  ~~~

  和G类似，M中也有alllink域将所有的M放在allm链表中。lockedg是某些情况下，G锁定在这个M中运行而不会切换到其它M中去。M中还有一个MCache，是当前M的内存的缓存。M也和G一样有一个常驻寄存器变量，代表当前的M。同时存在多个M，表示同时存在多个物理线程。

  结构体M中有两个G是需要关注一下的，一个是curg，代表结构体M当前绑定的结构体G。另一个是g0，是带有调度栈的goroutine，这是一个比较特殊的goroutine。普通的goroutine的栈是在堆上分配的可增长的栈，而g0的栈是M对应的线程的栈。所有调度相关的代码，会先切换到该goroutine的栈中再执行。

G 代表 goroutine，M 可以看做真实的资源（OS Threads）。P是 G-M 的中间层，P是一个“逻辑Proccessor”，组织多个Goroutine跑在同一个 OS Thread 上。

对于G来说，P就是运行它的“CPU”，可以说：G的眼里只有P。但从Go scheduler视角来看，真正的“CPU”是M，只有将P和M绑定才能让P的runq中G得以真实运行起来。这样的P与M的关系，就好比Linux操作系统调度层面用户线程(user thread)与核心线程(kernel thread)的对应关系那样(N x M)。


一个 P上会挂着多个G，当一个G执行结束时，P会选择下一个 Goroutine 继续执行。而当一个Goroutine执行太久没有结束，这样就需要调度给后面的 Goroutine 运行的机会。所以，Go scheduler 除了在一个 Goroutine 执行结束时会调度后面的 Goroutine 执行，还会在正在被执行的 Goroutine 发生以下情况时让出当前 goroutine 的执行权，并调度后面的 Goroutine 执行：IO 操作,Channel 阻塞,system call,运行较长时间.

对于运行时间较长的Goroutine，scheduler会在其 G对象上打上一个标志（ preempt），当这个 goroutine 内部发生函数调用的时候，会先主动检查这个标志，如果为 true ,就需要主动调用Gosched()来让出CPU。

然而如果G被阻塞在某个channel操作或network I/O操作上时，G会被放置到某个wait队列中，而M会尝试运行下一个runnable的G；如果此时没有runnable的G供m运行，那么m将解绑P，并进入sleep状态。当I/O available或channel操作完成，在wait队列中的G会被唤醒，标记为runnable，放入到某P的队列中，绑定一个M继续执行。

如果G被阻塞在某个system call操作上，那么不光G会阻塞，执行该G的M也会解绑P，与G一起进入sleep状态。如果此时有idle的M，则P与其绑定继续执行其他G；如果没有idle M，但仍然有其他G要去执行，那么就会创建一个新M。

当阻塞在syscall上的G完成syscall调用后，G会去尝试获取一个可用的P，如果没有可用的P，那么G会被标记为runnable，之前的那个sleep的M将再次进入sleep。

### 调度器状态的查看方法

~~~go
> GODEBUG=schedtrace=1000 godoc -http=:6060

SCHED 0ms: gomaxprocs=4 idleprocs=2 threads=5 spinningthreads=1 idlethreads=0 runqueue=0 [0 0 0 0]
SCHED 1007ms: gomaxprocs=4 idleprocs=4 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 2016ms: gomaxprocs=4 idleprocs=3 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 3017ms: gomaxprocs=4 idleprocs=3 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 4018ms: gomaxprocs=4 idleprocs=1 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 5018ms: gomaxprocs=4 idleprocs=1 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 6018ms: gomaxprocs=4 idleprocs=2 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 7018ms: gomaxprocs=4 idleprocs=3 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 8019ms: gomaxprocs=4 idleprocs=3 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
SCHED 9020ms: gomaxprocs=4 idleprocs=0 threads=27 spinningthreads=1 idlethreads=4 runqueue=0 [0 0 0 0]
SCHED 10020ms: gomaxprocs=4 idleprocs=3 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]
~~~

GODEBUG这个Go运行时环境变量非常有用，我们通过给其传入不同的key=value,...组合，可以查看Go的runtime会输出不同的调试信息，比如在这里我们给GODEBUG传入了”schedtrace=1000″，其含义就是每1000ms，打印输出一次goroutine scheduler的状态，每次一行.其他的状态如下:

~~~go
SCHED 1007ms: gomaxprocs=4 idleprocs=4 threads=27 spinningthreads=0 idlethreads=5 runqueue=0 [0 0 0 0]

SCHED：     调试信息输出标志字符串，代表本行是goroutine scheduler的输出.
1007ms：    即从程序启动到输出这行日志的时间.
gomaxprocs: P的数量.
idleprocs:  处于idle状态的P的数量；通过gomaxprocs和idleprocs的差值，我们就可知道执行go代码的P的数量.
threads:    os threads的数量，包含scheduler使用的m数量，加上runtime自用的类似sysmon这样的thread的数量.
spinningthreads: 处于自旋状态的os thread数量.
idlethread: 处于idle状态的os thread的数量.
runqueue=0： go scheduler全局队列中G的数量.
[0 0 0 0]:   分别为0个P的local queue中的G的数量.
~~~

### Goroutine的闭包函数

闭包函数可以直接引用外层代码定义的变量，需要注意的是，在闭包函数里面引用的是变量的地址，当goroutine被调度时，改地址的值才会被传递给goroutine 函数。

使用匿名函数或闭包创建 goroutine 时，除了将函数定义部分写在 go 的后面之外，还需要加上匿名函数的调用参数，格式如下：

```go
go func( 参数列表 ){
    函数体
}( 调用参数列表 )
```

其中：

* 参数列表：函数体内的参数变量列表。
* 函数体：匿名函数的代码。
* 调用参数列表：启动 goroutine 时，需要向匿名函数传递的调用参数。

### Goroutine的使用

设置goroutine运行的CPU数量，最新版本的go已经默认已经设置了。

```go
num := runtime.NumCPU()    //获取主机的逻辑CPU个数
runtime.GOMAXPROCS(num)    //设置可同时执行的最大CPU数
```

应用示例:

```go
package main

import (
    "fmt"
    "time"
)

func count(a int , b int )  {
    c := a+b
    fmt.Printf("%d + %d = %d\n",a,b,c)
}

func main() {
    for i :=0 ; i<10 ;i++{
        go count(i,i+1)  //启动10个goroutine 来计算
    }
    time.Sleep(time.Second * 3) // sleep作用是为了等待所有任务完成
} 
```

由于goroutine是异步执行的，那很有可能出现主程序退出时还有goroutine没有执行完，此时goroutine也会跟着退出。此时如果想等到所有goroutine任务执行完毕才退出，go提供了sync包和channel来解决同步问题，当然如果你能预测每个goroutine执行的时间，你还可以通过time.Sleep方式等待所有的groutine执行完成以后在退出程序。

### 通过sync实现goroutine之间的同步

WaitGroup 等待一组goroutinue执行完毕. 主程序调用 Add 添加等待的goroutinue数量. 每个goroutinue在执行结束时调用 Done ，此时等待队列数量减1.，主程序通过Wait阻塞，直到等待队列为0.

~~~go
package main

import (
	"fmt"
	"sync"
)

func count(a ,b int,n *sync.WaitGroup){
	c := a +b
	fmt.Printf("The Result of %d + %d=%d\n",a,b,c)
	defer n.Done() //goroutinue完成后, WaitGroup的计数-1
}


func main(){
	var wg sync.WaitGroup
	for i:=0;i<10 ;i++{
		wg.Add(1) // WaitGroup的计数加1
		go count(i,i+1,&wg)
	}
	wg.Wait()  //等待所有goroutine执行完毕
}

The Result of 9 + 10=19
The Result of 7 + 8=15
The Result of 8 + 9=17
The Result of 5 + 6=11
The Result of 0 + 1=1
The Result of 1 + 2=3
The Result of 2 + 3=5
The Result of 3 + 4=7
The Result of 6 + 7=13
The Result of 4 + 5=9
~~~

### 通过channel实现goroutine之间的同步

通过channel能在多个groutine之间通讯，当一个goroutine完成时候向channel发送退出信号,等所有goroutine退出时候，利用for循环channe去channel中的信号，若取不到数据会阻塞原理，等待所有goroutine执行完毕，使用该方法有个前提是你已经知道了你启动了多少个goroutine。

~~~go
package main

import (
	"fmt"
	"time"
)

func count(a int , b int ,exitChan chan bool)  {
	c := a+b
    fmt.Printf("The Result of %d + %d = %d\n",a,b,c)
	time.Sleep(time.Second*2)
	exitChan <- true
}

func main() {

	exitChan := make(chan bool,10)  //声明并分配管道内存
	for i :=0 ; i<10 ;i++{
		go count(i,i+1,exitChan)
	}
	for j :=0; j<10; j++{
		<- exitChan  //取信号数据，如果取不到则会阻塞e
	}
	close(exitChan) // 关闭管道
}

The Result of 9 + 10 = 19
The Result of 0 + 1 = 1
The Result of 7 + 8 = 15
The Result of 6 + 7 = 13
The Result of 3 + 4 = 7
The Result of 4 + 5 = 9
The Result of 1 + 2 = 3
The Result of 2 + 3 = 5
The Result of 8 + 9 = 17
The Result of 5 + 6 = 11
~~~



## 二、Channel

### Channel 管道

Channel是Go中的一个核心类型，你可以把它看成一个管道，通过它并发核心单元就可以发送或者接收数据进行通讯(communication).

Channel用于数据传递或数据共享，其本质上是一个先进先出的队列，使用Goroutine 和channel进行数据通讯简单高效，同时也线程安全，多个Goroutine可同时修改一个Channel，不需要加锁。

管道是一系列由channel联通的状态（stage），而每个状态是一组运行相同函数的Goroutine。在每个状态的Goroutine上:

* 通过流入（inbound）channel接收上游的数值.
* 运行一些函数来处理接收的数据，一般会产生新的数值.
* 通过流出（outbound）channel将数值发给下游.

每个语态都会有任意个流入或者流出channel，除了第一个状态（只有流出channel）和最后一个状态（只有流入channel）。第一个状态有时被称作源或者生产者；最后一个状态有时被称作槽（sink）或者消费者。

#### 创建 Channel

我们先从简单开始使用内置的make函数,创建一个Channel:

```go
ch := make(chan int)
````

然而每个channel都有一个特殊的类型,也就是channels可发送数据的类型。一个可以发送int类型数据的channel一般写为chan int，一个channel有发送和接受两个主要操作,都是通信行为。一个发送语句将一个值从一个goroutine通过channel发送到另一个执行接收操作的goroutine。发送和接收两个操作都是用<-运算符。

```go
ch <- p    // 发送值p到Channel ch中
p := <-ch  // 从Channel ch中接收数据，并将数据赋值给p
```

注意：在channel中箭头的指向是数据的流向．

和map类似,channel也一个对应make创建的底层数据结构的引用。当我们复制一个channel或用于函数参数传递时,我们只是拷贝了一个channel引用,因此调用者何被调用者将引用同一个channel对象。和其它的引用类型一样,channel的零值也是nil。两个相同类型的channel可以使用==运算符比较。如果两个channel引用的是相通的对象,那么比较的结果为真。一个channel也可以和nil进行比较。

Channel类型的定义格式：

```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) Type .
```

它包括三种类型的定义。可选的<-代表channel的方向。如果没有指定方向，那么Channel就是双向的，既可以接收数据，也可以发送数据。

```go
chan p            // 可以接收和发送类型为p的数据
chan<- float64   // 只可以用来发送 float64 类型的数据
<-chan int       // 只可以用来接收 int 类型的数据
```

这里需要注意下：<-总是优先和最左边的类型结合。

使用make初始化Channel,我们还可以设置channel的容量,容量(capacity)代表Channel容纳的最多的元素的数量，代表Channel的缓存的大小。

```go
ch = make(chan int)    // 无缓冲 channel
ch = make(chan int, 0) // 无缓冲 channel
ch = make(chan int, 3) // 缓冲 channel容量是3
```

如果没有设置容量，或者容量设置为0, 说明Channel没有缓存，只有sender和receiver都准备好了后它们的通讯(communication)才会发生(Blocking)。如果设置了缓存，就有可能不发生阻塞， 只有buffer满了后 send才会阻塞， 而只有缓存空了后receive才会阻塞。一个nil channel不会通信。

#### 无缓冲的Channels

无缓冲：发送和接收动作是同时发生的Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。如果没有 goroutine 读取 channel （<- channel），则发送者 (channel <-) 会一直阻塞。


#### 缓冲的Channels

缓冲：缓冲 channel 类似一个有容量的队列。当队列满的时候发送者会阻塞；当队列空的时候接收者会阻塞。


此外在使用channel之后可以进行关闭，关闭channel后,对该channel的任何发送操作都将导致panic异常。对一个已经被close过的channel之行接收操作依然可以接受到之前已经成功发送的数据;如果channel中已经没有数据的话讲产生一个零值的数据。

使用内置的close函数就可以关闭一个channel:

```go
close(ch)
```

但是关于关闭channel 有几点需要注意:

* 重复关闭 channel 会导致 panic.
* 向关闭的 channel 发送数据会 panic.
* 从关闭的 channel 读数据不会 panic，读出channel中已有的数据之后再读就是channel类似的默认值，比如 chan int 类型的channel关闭之后读取到的值为 0.

这里我们需要区分一下第三种channel 中的值是默认值还是channel 关闭了。可以使用 ok-idiom 方式，这种方式在 map 中比较常用.

~~~go
ch := make(chan int, 10)
...
close(ch)

// ok-idiom 
val, ok := <-ch
if ok == false {
    // channel closed
}
~~~

### Channel的典型用法

#### goroutine 使用channel通信

~~~go
func main() {
    x := make(chan int)
    go func() {
        x <- 1
    }()
    <-x
}
~~~

#### select

select语句选择一组可能的send操作和receive操作去处理。它类似switch,但是只是用来处理通讯(communication)操作。
它的case可以是send语句，也可以是receive语句，亦或者default。

receive语句可以将值赋值给一个或者两个变量。它必须是一个receive操作。

select在一定程度上可以类比于linux中的 IO 多路复用中的 select。后者相当于提供了对多个 IO 事件的统一管理，而 Golang 中的 select 相当于提供了对多个 channel 的统一管理。当然这只是 select 在 channel 上的一种使用方法。

~~~go
select {
    case e, ok := <-ch1:
        ...
    case e, ok := <-ch2:
        ...
    default:  
}
~~~

这里需要注意的是 select 中的 break 只能跳到 select 这一层。select 使用的时候一般需要配合 for 循环使用，因为正常 select 里面的流程也就执行一遍。这么来看 select 中的 break 就稍显鸡肋了。所以使用 break 的时候一般配置 label 使用，label 定义在 for循环这一层。

~~~go
for {
    select {
        ...
    }
}
~~~

#### range channel

使用range channel 我们可以直接取到 channel 中的值。当我们使用 range 来操作 channel 的时候，一旦 channel 关闭，channel 内部数据读完之后循环自动结束。

~~~go
func consumer(ch chan int) {
    for x := range ch {
        fmt.Println(x)
        ...
    }
}

func producer(ch chan int) {
  for _, v := range values {
      ch <- v
  }  
}
~~~

#### 超时控制

select有很重要的一个应用就是超时处理。由于如果没有case需要处理，select语句就会一直阻塞着。这时候我们可能就需要一个超时操作，用来处理超时的情况通常在很多操作情况下都需要超时控制，我们可以利用 select 实现超时控制:

~~~go
func main() {
    c1 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c1 <- "result 1"
    }()
    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(time.Second * 1):
        fmt.Println("timeout 1")
    }
}
~~~

这里利用的是time.After方法，它返回一个类型为<-chan Time的单向的channel，在指定的时间发送一个当前时间给返回的channel中。

#### channel同步

channel可以用在goroutine之间的同步。
下面的例子main中goroutine通过done channel等待 mission完成任务。 mission做完任务后只需往channel发送一个数据就可以通知main goroutine任务完成。

~~~go
func mission(done chan bool) {
	time.Sleep(time.Second)
	// 通知任务已完成
	done <- true
}
func main() {
	done := make(chan bool, 1)
	go mission(done)
	// 等待任务完成
	<-done
}
~~~

在Golang中的channel 将goroutine 隔离开，并发编程的时候可以将注意力放在 channel 上。在一定程度上，这个和消息队列的解耦功能还是挺像的。

#### 扇出和扇入

通常情况下多个函数可以同时从一个channel接收数据，直到channel关闭，这种情况被称作扇出。这是一种将工作分布给一组工作者的方法，目的是并行使用CPU和I/O。

如果一个函数同时接收并处理多个channel输入并转化为一个输出channel，直到所有的输入channel都关闭后，关闭输出channel，这种情况就被称作扇入。

但是main可以容易的通过关闭done　channel来释放所有的发送者。关闭是个高效的发送给所有发送者的信号。我们扩展channel管道里的每个函数，让其以参数方式接收done，并通过defer语句在函数退出时执行关闭操作，这样main里所有的退出路径都会触发管道里的所有状态退出。

~~~go
func main() {
   // 构建done channel，整个管道里分享done，并在管道退出时关闭这个channel
    // 以此通知所有Goroutine该推出了。
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // 发布sq的工作到两个都从in里读取数据的Goroutine
    c1 := sq(done, in)
    c2 := sq(done, in)

    // 处理来自output的第一个数值
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 或者 9

    // done会通过defer调用而关闭
}

func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n*n
        }
        close(out)
    }()
    return out
}
~~~

merge对每个流入channel启动一个Goroutine，并将流入的数值复制到流出channel，由此将一组channel转换到一个channel。一旦启动了所有的output  Goroutine，merge函数会多启动一个Goroutine，这个Goroutine在所有的输入channel输入完毕后，关闭流出channel。sq函数是把上一个函数的chan最为参数，下一个输出的chan作为返回值。

但是往一个已经关闭的channel输出会产生异常（panic），所以一定要保证所有数据发送完成后再执行关闭。

所以发送Goroutine将发送操作替换为一个select语句，要么把数据发送给out，要么处理来自done的数值。done的类型是个空结构，因为具体数值并不重要：接收事件本身就指明了应当放弃继续发送给out的动作。而output Goroutine会继续循环处理流入的channel，c,而不会阻塞上游状态.

~~~go
//gen函数启动一个Goroutine，将整数数列发送给channel，如果所有数都发送完成，关闭这个channel
func gen(nums ...int) <-chan int {
 out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}

// 从一个channel接收整数，并求整数的平方，发送给另一个channel.
// mission的循环中退出，因为我们知道如果done已经被关闭了，也会关闭上游的gen状态.
// mission通过defer语句，保证不管从哪个返回路径，它的out channel都会被关闭.


func mission(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}


func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // 为每个cs中的输入channel启动一个output Goroutine。outpu从c里复制数值直到c被关闭
    // 或者从done里接收到数值，之后output调用wg.Done
    output := func(c <-chan int) {
        for n := range c {
            select {
            	case out <- n:
            	case <-done:
            }
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // 启动一个Goroutine，当所有output Goroutine都工作完后（wg.Done），关闭out，
    // 保证只关闭一次。这个Goroutine必须在wg.Add之后启动
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
~~~

在channel模式中有个模式:

* 状态会在所有发送操作做完后，关闭它们的流出channel

* 状态会持续接收从流入channel输入的数值，直到channel关闭

这个模式使得每个接收状态可以写为一个range循环，并保证所有的Goroutine在将所有的数值发送成功给下游后立刻退出。

所以在构建channel的时候:

* 状态会在所有发送操作做完后，关闭它们的流出channel.
* 状态会持续接收从流入channel输入的数值，直到channel关闭或者其发送者被释放.

因而管道要么保证足够能存下所有发送数据的缓冲区，要么接收来自接收者明确的要放弃channel的信号，来保证释放发送者。

## 三、Sync.WaitGroup解析

Golang中的同步可以通过Sync.WaitGroup来实现的．WaitGroup的功能，它实现了一个类似队列的结构，可以一直向队列中添加任务，当任务完成后便从队列中删除，如果队列中的任务没有完全完成，可以通过Wait()函数来出发阻塞，防止程序继续进行，直到所有的队列任务都完成为止．

WaitGroup的特点是,Wait()方法可以用来阻塞直到队列中的所有任务都完成时才解除阻塞，而不需要sleep一个固定的时间来等待．但是其缺点是无法指定固定的Goroutine数目,我们可以通过使用channel解决这个问题。

Sync.WaitGroup中有3个方法，Add()，Done()，Wait()。其中Done()是Add(-1)的别名。

Sync.WaitGroup中三个方法的作用是：

* Add:添加或者减少等待goroutine的数量
* Done:相当于Add(-1),减掉一个goroutine计数，计数不为0
* Wait:执行阻塞，直到所有的WaitGroup数量变成0

这里需要注意下，Sync中的Add()方法和Done方法。即在运行main函数的goroutine里运行Add()方法，在其他的goroutine里面运行Done()函数。

Add()方法添加将可能为负的增量添加到WaitGroup计数器。如果计数器变为零，则释放在等待时阻止的所有goroutine。如果计数器变为负数，则panic。
请注意，在计数器为零时发生的具有正增量的调用时必须在Wait()方法之前。 具有负增量的调用或具有在计数器大于零时开始的正增量的调用可以在任何时间发生。
通常，这意味着对Add的调用应该在语句之前执行,创建要等待的goroutine或其他事件。如果重新使用WaitGroup等待几个独立的事件集，新的Add()调用必须在所有先前的Wait()调用返回后发生。

~~~go
func (wg *WaitGroup) Add(delta int) {
	statep, semap := wg.state()
	if race.Enabled {
		_ = *statep // trigger nil deref early
		if delta < 0 {
			// Synchronize decrements with Wait.
			race.ReleaseMerge(unsafe.Pointer(wg))
		}
		race.Disable()
		defer race.Enable()
	}
	state := atomic.AddUint64(statep, uint64(delta)<<32)
	v := int32(state >> 32)
	w := uint32(state)
	if race.Enabled && delta > 0 && v == int32(delta) {
		// The first increment must be synchronized with Wait.
		// Need to model this as a read, because there can be
		// several concurrent wg.counter transitions from 0.
		race.Read(unsafe.Pointer(semap))
	}
	if v < 0 {
		panic("sync: negative WaitGroup counter")
	}
	if w != 0 && delta > 0 && v == int32(delta) {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	if v > 0 || w == 0 {
		return
	}
	// This goroutine has set counter to 0 when waiters > 0.
	// Now there can't be concurrent mutations of state:
	// - Adds must not happen concurrently with Wait,
	// - Wait does not increment waiters if it sees counter == 0.
	// Still do a cheap sanity check to detect WaitGroup misuse.
	if *statep != state {
		panic("sync: WaitGroup misuse: Add called concurrently with Wait")
	}
	// Reset waiters count to 0.
	*statep = 0
	for ; w != 0; w-- {
		runtime_Semrelease(semap, false)
	}
}
~~~

Done()方法将WaitGroup计数器减1

~~~go
func (wg *WaitGroup) Done() {
	wg.Add(-1)
}
~~~

Wait()直到WaitGroup计数器为零

~~~go
func (wg *WaitGroup) Wait() {
	statep, semap := wg.state()
	if race.Enabled {
		_ = *statep // trigger nil deref early
		race.Disable()
	}
	for {
		state := atomic.LoadUint64(statep)
		v := int32(state >> 32)
		w := uint32(state)
		if v == 0 {
			// Counter is 0, no need to wait.
			if race.Enabled {
				race.Enable()
				race.Acquire(unsafe.Pointer(wg))
			}
			return
		}
		// Increment waiters count.
		if atomic.CompareAndSwapUint64(statep, state, state+1) {
			if race.Enabled && w == 0 {
				// Wait must be synchronized with the first Add.
				// Need to model this is as a write to race with the read in Add.
				// As a consequence, can do the write only for the first waiter,
				// otherwise concurrent Waits will race with each other.
				race.Write(unsafe.Pointer(semap))
			}
			runtime_Semacquire(semap)
			if *statep != 0 {
				panic("sync: WaitGroup is reused before previous Wait has returned")
			}
			if race.Enabled {
				race.Enable()
				race.Acquire(unsafe.Pointer(wg))
			}
			return
		}
	}
}
~~~

应用示例

~~~go
func main(){
	var wg sync.WaitGroup

	for i:=0;i<5;i=i+1{
		wg.Add(1)
		go func(n int) {
			//defer wg.Done(),注意这个Done的位置，是另一个函数
			defer wg.Add(-1)
			EchoNumber(n)
		}(i)
	}
	wg.Wait()
}

func EchoNumber(i int){
	time.Sleep(time.Millisecond *2000)
	fmt.Println(i)

}

3
4
2
0
1
~~~

这个应用示例很简单，是将每次循环的数量过3秒钟输出。那么，这个程序如果不用WaitGroup，那么将看不见输出结果。因为Goroutine还没执行完，主线程已经执行完毕。注释的defer wg.Done()和defer wg.Add(-1)作用一样。

## 四、Sync.Map解析

在Go1.9之前，Go自带的Map不是并发安全的,因此我们需要自己再封装一层，给Map加上把读写锁,例如:

~~~go
type MapWithLock struct {
    sync.RWMutex
    M map[string]Kline
}
~~~

用MapWithLock的读写锁去控制map的并发安全。

但是到了Go1.9发布，它有了一个新的特性，那就是sync.map，它是原生支持并发安全的map，不过它的用法和以前我们熟悉的map完全不一样，主要还是因为sync.map封装了更为复杂的数据结构，用来实现比之前加锁map更优秀的性能。

空间换时间。 通过冗余的两个数据结构(read、dirty),实现加锁对性能的影响。使用只读数据(read)，避免读写冲突。动态调整，miss次数多了之后，将dirty数据提升为read。double-checking。
延迟删除。 删除一个键值只是打标记，只有在提升dirty的时候才清理删除的数据。
优先从read读取、更新、删除，因为对read的读取不需要锁。

### 数据结构

~~~go
type Map struct {
    // 当涉及到dirty数据的操作的时候，需要使用这个锁
    mu Mutex
    // 一个只读的数据结构，因为只读，所以不会有读写冲突。
    // 所以从这个数据中读取总是安全的。
    // 实际上，实际也会更新这个数据的entries,如果entry是未删除的(unexpunged), 并不需要加锁。如果entry已经被删除了，需要加锁，以便更新dirty数据。
    read atomic.Value // readOnly
    // dirty数据包含当前的map包含的entries,它包含最新的entries(包括read中未删除的数据,虽有冗余，但是提升dirty字段为read的时候非常快，不用一个一个的复制，而是直接将这个数据结构作为read字段的一部分),有些数据还可能没有移动到read字段中。
    // 对于dirty的操作需要加锁，因为对它的操作可能会有读写竞争。
    // 当dirty为空的时候， 比如初始化或者刚提升完，下一次的写操作会复制read字段中未删除的数据到这个数据中。
    dirty map[interface{}]*entry
    // 当从Map中读取entry的时候，如果read中不包含这个entry,会尝试从dirty中读取，这个时候会将misses加一，
    // 当misses累积到 dirty的长度的时候， 就会将dirty提升为read,避免从dirty中miss太多次。因为操作dirty需要加锁。
    misses int
}
~~~

它的数据结构很简单，值包含四个字段：read、mu、dirty、misses。

readOnly.m和Map.dirty存储的值类型是*entry,它包含一个指针p, 指向用户存储的value值。

~~~go
type entry struct {
    p unsafe.Pointer // *interface{}
}
~~~

p通常有三种类型的值:

* nil: entry已被删除了，并且m.dirty为nil
* expunged: entry已被删除了，并且m.dirty不为nil，而且这个entry不存在于m.dirty中
* 其它： entry是一个正常的值

它使用了冗余的数据结构read、dirty。dirty中会包含read中为删除的entries，新增加的entries会加入到dirty中。
read的数据结构是：

~~~go
type readOnly struct {
    m       map[interface{}]*entry
    amended bool // 如果Map.dirty有些数据不在中的时候，这个值为true
}
~~~

amended指明Map.dirty中有readOnly.m未包含的数据，所以如果从Map.read找不到数据的话，还要进一步到Map.dirty中查找。

对Map.read的修改是通过原子操作进行的。

虽然read和dirty有冗余数据，但这些数据是通过指针指向同一个数据，所以尽管Map的value会很大，但是冗余的空间占用还是有限的。

sync.Map主要有五个方法:

1、Load   取key对应的value

2、Store   存 key,value

3、Delete   删除key,及其value

4、Range   遍历所有的key,value

### Load方法

Load方法，提供一个键key,查找对应的值value,如果不存在，通过ok反映：

~~~go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // 1.首先从m.read中得到只读readOnly,从它的map中查找，不需要加锁
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    // 2. 如果没找到，并且m.dirty中有新数据，需要从m.dirty查找，这个时候需要加锁
    if !ok && read.amended {
        m.mu.Lock()
        // 双检查，避免加锁的时候m.dirty提升为m.read,这个时候m.read可能被替换了。
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        // 如果m.read中还是不存在，并且m.dirty中有新数据
        if !ok && read.amended {
            // 从m.dirty查找
            e, ok = m.dirty[key]
            // 不管m.dirty中存不存在，都将misses计数加一
            // missLocked()中满足条件后就会提升m.dirty
            m.missLocked()
        }
        m.mu.Unlock()
    }
    if !ok {
        return nil, false
    }
    return e.load()
}
~~~

这里会先从m.read中加载，不存在的情况下，并且m.dirty中有新数据，加锁，然后从m.dirty中加载。其次是这里使用了双检查的处理，因为在下面的两个语句中，这两行语句并不是一个原子操作。

```go
if !ok && read.amended {
        m.mu.Lock()
```

当第一句执行的时候条件满足，但是在加锁之前，m.dirty可能被提升为m.read,所以加锁后还得再检查m.read，后续的方法中都使用了这个方法。

如果我们查询的键值正好存在于m.read中，无须加锁，直接返回，理论上性能优异。即使不存在于m.read中，经过miss几次之后，m.dirty会被提升为m.read，又会从m.read中查找。所以对于更新／增加较少，加载存在的key很多的case,性能基本和无锁的map类似。
接着我们看下如何m.dirty是如何被提升的。 missLocked方法中可能会将m.dirty提升。

~~~go
func (m *Map) missLocked() {
    m.misses++
    if m.misses < len(m.dirty) {
        return
    }
    m.read.Store(readOnly{m: m.dirty})
    m.dirty = nil
    m.misses = 0
}
~~~

上面的最后三行代码就是提升m.dirty的，很简单的将m.dirty作为readOnly的m字段，原子更新m.read。提升后m.dirty、m.misses重置， 并且m.read.amended为false。

### Store方法

* Store方法是更新或者新增一个entry。

```go
func (m *Map) Store(key, value interface{}) {
    // 如果m.read存在这个键，并且这个entry没有被标记删除，尝试直接存储。
    // 因为m.dirty也指向这个entry,所以m.dirty也保持最新的entry。
    read, _ := m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok && e.tryStore(&value) {
        return
    }
    // 如果`m.read`不存在或者已经被标记删除
    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok {
        if e.unexpungeLocked() { //标记成未被删除
            m.dirty[key] = e //m.dirty中不存在这个键，所以加入m.dirty
        }
        e.storeLocked(&value) //更新
    } else if e, ok := m.dirty[key]; ok { // m.dirty存在这个键，更新
        e.storeLocked(&value)
    } else { //新键值
        if !read.amended { //m.dirty中没有新的数据，往m.dirty中增加第一个新键
            m.dirtyLocked() //从m.read中复制未删除的数据
            m.read.Store(readOnly{m: read.m, amended: true})
        }
        m.dirty[key] = newEntry(value) //将这个entry加入到m.dirty中
    }
    m.mu.Unlock()
}
func (m *Map) dirtyLocked() {
    if m.dirty != nil {
        return
    }
    read, _ := m.read.Load().(readOnly)
    m.dirty = make(map[interface{}]*entry, len(read.m))
    for k, e := range read.m {
        if !e.tryExpungeLocked() {
            m.dirty[k] = e
        }
    }
}
func (e *entry) tryExpungeLocked() (isExpunged bool) {
    p := atomic.LoadPointer(&e.p)
    for p == nil {
        // 将已经删除标记为nil的数据标记为expunged
        if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
    }
    return p == expunged
}
```

通常是先从操作m.read开始的，如果不满足条件再加锁，然后操作m.dirty。Store 方法可能会在某种情况下(初始化或者m.dirty刚被提升后)从m.read中复制数据，如果这个时候m.read中数据量非常大，可能会影响性能。

### Delete方法

Delete方法用来删除一个键值。

~~~go
func (m *Map) Delete(key interface{}) {
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    if !ok && read.amended {
        m.mu.Lock()
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        if !ok && read.amended {
            delete(m.dirty, key)
        }
        m.mu.Unlock()
    }
    if ok {
        e.delete()
    }
}
~~~

这里的删除操作还是从m.read中开始， 如果这个entry不存在于m.read中，并且m.dirty中有新数据，则加锁尝试从m.dirty中删除。

此外需要,双检查的。 从m.dirty中直接删除即可，就当它没存在过，但是如果是从m.read中删除，并不会直接删除，而是打标记：

~~~go
func (e *entry) delete() (hadValue bool) {
    for {
        p := atomic.LoadPointer(&e.p)
        // 已标记为删除
        if p == nil || p == expunged {
            return false
        }
        // 原子操作，e.p标记为nil
        if atomic.CompareAndSwapPointer(&e.p, p, nil) {
            return true
        }
    }
}
~~~

### Range方法

因为for ... range map是内建的语言特性，所以没有办法使用for range遍历sync.Map, 但是可以使用它的Range方法，通过回调的方式遍历

~~~go
func (m *Map) Range(f func(key, value interface{}) bool) {
    read, _ := m.read.Load().(readOnly)
    // 如果m.dirty中有新数据，则提升m.dirty,然后在遍历
    if read.amended {
        //提升m.dirty
        m.mu.Lock()
        read, _ = m.read.Load().(readOnly) //双检查
        if read.amended {
            read = readOnly{m: m.dirty}
            m.read.Store(read)
            m.dirty = nil
            m.misses = 0
        }
        m.mu.Unlock()
    }
    // 遍历, for range是安全的
    for k, e := range read.m {
        v, ok := e.load()
        if !ok {
            continue
        }
        if !f(k, v) {
            break
        }
    }
}
~~~

Range方法调用前可能会做一个m.dirty的提升，不过提升m.dirty不是一个耗时的操作。