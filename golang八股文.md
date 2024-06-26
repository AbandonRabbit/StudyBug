# golang八股文整理（持续搬运）

## 进程、线程、协程

进程：资源分配和CPU调度的基本单位

线程：CPU调度的基本单位，线程除了有一些自己的必要的堆栈空间之外，其它的资源都是共享的线程中的，共享的资源包括：

1. 所有线程共享相同的虚拟地址空间，即它们可以访问同样的代码段、数据段和堆栈段。

2. 文件描述符：进程打开的文件描述符是进程级别的资源，所以同一个进程中的线程可以共享打开的文件描述符，这意味着它们可以同时读写同一个文件。
3. 全局变量：全局变量是进程级别的变量，因此可以被同一个进程中的所有线程访问和修改。
4. 静态变量：静态变量也是进程级别的变量，在同一个进程中的线程之间共享内存空间。
5. 进程ID、进程组ID

独占的资源：

1. 线程ID

2. 寄存器组的值
3. 线程堆栈
4. 错误返回码
5. 信号屏蔽码
6. 线程的优先级

协程：用户态的线程，可以通过用户程序创建、删除。协程切换时不需要切换内核态。

协程与线程的区别：

1.线程是操作系统的概念，而协程是程序级的概念。线程由操作系统调度执行，每个线程都有自己的执行上下文，包
括程序计数器、寄存器等。而协程由程序自身控制。
2.多个线程之间通过切换执行的方式实现并发。线程切换时需要保存和恢复上下文，涉及到上下文切换的开销。而协
程切换时不需要操作系统的介入，只需要保存和恢复自身的上下文，切换开销较小。
3.线程是抢占式的并发，即操作系统可以随时剥夺一个线程的执行权。而协程是合作式的并发，协程的执行权由程序
自身决定，只有当协程主动让出执行权时，其他协程才会得到执行机会。

线程的优点

1. 创建一个新线程的代价要比创建一个新进程小的多

2. 线程之间的切换相较于进程之间的切换需要操作系统做的工作很少
3. 线程占用的资源要比进程少很多
4. 能充分利用多处理器的可并行数量
5. 等待慢速 IO操作结束以后，程序可以执行其他的计算任务

缺点：

1. 性能损失（ 一个计算密集型线程是很少被外部事件阻塞的，无法和其他线程共享同一个处理器，当计算密集型的线程的数量比可用的处理器多，那么就有可能有很大的性能损失，这里的性能损失是指增加了额外的同步和调度开销，二可用资源不变。）

2. 健壮性降低（线程之间是缺乏保护性的。在一个多线程程序里，因为时间上分配的细微差距或者是共享了一些不应该共享的变量而造成不良影响的可能影响是很大的。）
3. 缺乏访问控制（ 因为进程是访问控制的基本粒度，在一个线程中调用某些OS函数会对整个进程造成影响 。）
4. 编程难度提高（编写和 调试一个多线程程序比单线程困难的多。）


有栈协程和无栈协程

有栈协程：把局部变量放入到新开的空间上，golang的实现，类似于内核态线程的实现，不同协程间切换还是要切换对应的栈上下文，只是不用陷入
内核
无栈协程：直接把局部变量放入系统栈上，js、c++、rust那种await、async实现，主要原理就是闭包+异步，换句话说，其实就是协程的上下文都放
到公共内存中，协程切换时，使用状态机来切换，就不用切换对应的上下文了，因为都在堆里的。比有栈协程都要轻量许多。


## Go语言——垃圾回收

![在这里插入图片描述](https://img-blog.csdnimg.cn/15ce47bbccba4cabad1432ec85c89b0a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
Go V1.3之前的标记-清除：
1.暂停业务逻辑，找到不可达的对象，和可达对象
2.开始标记，程序找出它所有可达的对象，并做上标记
3.标记完了之后，然后开始清除未标记的对象。
4.停止暂停，让程序继续跑。然后循环重复这个过程，直到process程序生命周期结束

标记-清除的缺点：
STW（stop the world）：让程序暂停，程序出现卡顿
标记需要扫描整个heap
清除数据会产生heap碎片

为了减少STW的时间，后来对上述的第三步和第四步进行了替换。

Go V1.5 三色标记法
1.把新创建的对象，默认的颜色都标记为“白色”
![在这里插入图片描述](https://img-blog.csdnimg.cn/9dbca38cd1e5431887f6464e0296cde2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)

2.每次GC回收开始，然后从根节点开始遍历所有对象，把遍历到的对象从白色集合放入“灰色”集合
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e134376eb4544b39729389acac1a344.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)

3.遍历灰色集合，将灰色对象引用的对象从白色集合放入到灰色集合，之后将此灰色对象放入到黑色集合
![在这里插入图片描述](https://img-blog.csdnimg.cn/303b3faf82154d27b76a666223043b96.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
4.重复第三步，直到灰色中无任何对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0513b94817a456d9b873cb3deb931f8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/39715ce8c513415c9e22fa189017c2d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
5.回收所有的白色标记的对象，也就是回收垃圾

三色标记法在不采用STW保护时会出现：
1.一个白色对象被黑色对象引用
2.灰色对象与它之间的可达关系的白色对象遭到破坏

这两种情况同时满足，会出现对象丢失

解决方案：
1.强三色不变式：强制性的不允许黑色对象引用白色对象（破坏1）
2.弱三色不变式：黑色对象可以引用白色对象，白色对象存在其他灰色对象对它的引用，或者可达它的链路上游存在灰色对象（破坏2）

屏障：
1.插入屏障：在A对象引用B对象的时候，B对象被标记为灰色（满足强三色不变式，黑色引用的白色对象会被强制转换为灰色）。只有堆上的对象触发插入屏障，栈上的对象不触发插入屏障。在准备回收白色前，重新遍历扫描一次栈空间。此时加STW暂停保护栈，防止外界干扰。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e46e72416d7b495ca585d57498179429.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
不足：结束时需要使用STW来重新扫描栈

2.删除屏障：被删除的对象，如果自身为灰色或者白色，那么被标记为灰色（满足弱三色不变式）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd3eea36d3934783b5d784da910be5df.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_20,color_FFFFFF,t_70,g_se,x_16)
删除屏障的不足：回收精度低，一个对象即使被删除了最后一个指向它的指针也依旧可以活过这一轮，在下一轮GC中被清理掉。

Go V1.8的三色标记法+混合写屏障机制
具体操作：
1.GC开始将栈上的可达对象全部扫描并标记为黑色（之后不再进行第二次重复扫描，无需STW）
2.GC期间，任何在栈上创建的新对象，均为黑色
3.堆上被删除对象标记为灰色
4.堆上被添加的对象标记为灰色
满足：变形的弱三色不变式（结合了插入、删除写屏障的优点）

![在这里插入图片描述](https://img-blog.csdnimg.cn/98ecbe5030a54f6aa7c92be6c89fa90f.png)
对于堆上的对象，采用三色标记法+写屏障保护

GC垃圾收集的多个阶段：

1.标记准备阶段；

```
启动后台标记任务
暂停程序（STW），所有的处理器在这时会进入安全点（Safe point）；
如果当前垃圾收集循环是强制触发的，我们还需要处理还未被清理的内存管理单元；
将根对象入队
开启写屏障
```

2.标记阶段；

```
恢复用户协程
使用三色标记法开始标记，此时用户协程和标记协程并发执行
```

3.标记终止阶段；

```
暂停用户协程
计算下一次触发GC时需要达到的堆目标
唤醒后台清扫协程
```

4.清理阶段；

```
关闭写屏障
恢复用户协程
异步清理回收
```

什么是根对象？
根对象（root object）是指那些能够从全局可达的地方访问到的对象。垃圾回收器会从根对象开始，通过遍历根对象的引用关系，逐步追踪并标记所有可达的对象。任何未被标记的对象都会被认为是垃圾，最终被回收释放。

```
1.全局变量：全局变量可以被程序中的任何位置引用到，因此是根对象。
2.当前正在执行的函数的局部变量：当一个函数正在执行时，其局部变量可以被当前函数中的代码访问到，因此也是根对象。
3.当前正在执行的 goroutine 的栈中的变量：goroutine 是 Go语言并发编程中的轻量级线程，每个 goroutine 都有一块独立的栈空间，其中的变量可以被当前 goroutine 访问到，也是根对象。
4.其他和运行时系统相关的数据结构和变量。
```

三色标记法的缺点：

```
1.暂停时间：在进行垃圾回收时，必须停止程序执行，这会导致应用程序暂停。引入写屏障保护可以减少暂停时间，
但仍然可能导致性能下降。
2.内存开销：三色标记法需要为每个对象维护额外的状态信息，以记录其标记状态。这会增加内存开销，并可能对内
存资源造成负担。
3.频繁的垃圾回收：三色标记法需要频繁地迭代标记和清除对象，如果要回收的垃圾对象很多，可能会导致回收过程
变得非常耗时。
4.碎片化：垃圾回收过程中，如果频繁进行对象的移动和重新分配内存，可能会导致内存碎片化，降低内存的利用
率。
```

### GC的触发条件

1.主动触发(手动触发)，通过调用 runtime.GC 来触发GC，此调用阻塞式地等待当前GC运行完毕。
2.被动触发，分为两种方式：

2.1.使用步调（Pacing）算法，其核心思想是控制内存增长的比例,每次内存分配时检查当前内存分配量是否已达到阈值（环境变量GOGC）：默认100%，即当内存扩大一倍时启用GC。
2.2.使用系统监控，当超过两分钟没有产生任何GC时，强制触发 GC。

### GC调优

1.控制内存分配的速度，限制Goroutine的数量，提高赋值器mutator的CPU利用率（降低GC的CPU利用率）
2.少量使用+连接string
3.slice提前分配足够的内存来降低扩容带来的拷贝
4.避免map key对象过多，导致扫描时间增加
5.变量复用，减少对象分配，例如使用sync.Pool来复用需要频繁创建临时对象、使用全局变量等
6.增大GOGC的值，降低GC的运行频率

### GMP调度和CSP模型

CSP模型是“以通信的方式来共享内存”，不同于传统的多线程通过共享内存来通信。用于描述两个独立的并发实体通过共享的通讯channel来进行通信的并发模型。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b50c1c625a894442a02089b3cf19ea3c.png)
GMP分别是什么，分别有多少数量？
G：goroutine，go的协程，每个go关键字都会创建一个协程
M：machine，工作线程，在Go中称为Machine，数量对应真实的CPU数
P：process，包含运行Go代码所需要的必要资源，用来调度G和M之间的关联关系，其数量可以通过GOMAXPROCS来设置，默认为核心数

![在这里插入图片描述](https://img-blog.csdnimg.cn/62ae4b7d85824284acdcb436d4d833dd.png)

```go
// src/ runtime/ runtime2.go
type g struct {
goid int64//唯一的goroutine的ID
sched gobuf //goroutine切换时，用于保存g的上下文
stack stack //栈
gopc		// pc of go statement that crlated this goroutine
startpc uintptr //pc of goroutine function
...
}

type p struct {
lock mutex
id int32
status uint32 // one of pidle/ prunning / . ..
// Queue of runnable goroutines. Accessed without lock.
runqhead uint32 //本地队列队头
runqtail uint32 //本地队列队尾
runq	[256]guintptr //本地队列，大小256的数组，数组往往会被都读入到缓存中，对缓存友好，效率较高
runnext guintptr // 下一个优先执行的goroutine (一定是最后生产出来的)，为了实现局部性原理,runnext中的G永远会被最先调
...

type m struct{
g0 *g 
// 每个M都有一个自己的G0，不指向任何可执行的函数，在调度或系统调用时，M会切换到G0，使用G0的栈空间来调度
curg *g
//当前正在执行的G
...
}

type schedt struct{
...
runq gQueue //全局队列，链表（长度无限制）
runqsize int32 //全局队列长度
...
}

type gobuf struct{
 //保存CPU的rsp寄存器的值
 sp uintptr
 //保存CPU的rip寄存器的值
 pc uintptr
 //记录当前这个gobuf对象属于哪个Goroutine
 g guintptr
 //保存系统调用的返回值
 ret sys.Uintreg
 //保存CPU的rbp寄存器的值
 ...
}
```

线程想运行任务就得获取 P，从 P 的本地队列获取 G，当 P 的本地队列为空时，M 也会尝试从全局队列或其他 P 的本地队列获取 G。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

Goroutine调度策略
1.队列轮转：P会周期性的将G调度到M中执行，执行一段时间后，保存上下文，将G放到队列尾部，然后从队列中再取出一个G进行调度，P还会周期性的查看全局队列是否有G等待调度到M中执行
2.系统调用：当G0即将进入系统调用时，M0将释放P，进而某个空闲的M1获取P，继续执行P队列中剩下的G。M1的来源有可能是M的缓存池，也可能是新建的。
3.当G0系统调用结束后，如果有空闲的P，则获取一个P，继续执行G0。如果没有，则将G0放入全局队列，等待被其他的P调度。然后M0将进入缓存池睡眠。
![在这里插入图片描述](https://img-blog.csdnimg.cn/843ed96c28c9441e8eb7565d1028546c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_19,color_FFFFFF,t_70,g_se,x_16)

### Groutine的切换时机

1.select操作阻塞时
2.io阻塞
3.阻塞在channel
4.程序员显示编码操作
5.等待锁
6.程序调用

### Goroutine调度原理

调度循环：每个p都有一个协程g0，调度时首先切换到协程g0，切换到接下来将要运行的协程g，再从协程g切换到协程g0，开始新一轮调度。
从协程g0调度到协程g，经历了从schedule函数到execute函数再到gogo函数的过程。
schedule函数处理具体的调度策略，选择下一个要执行的协程。
execute函数执行一些具体的状态转移、协程g与结构体m之间的绑定等操作。
gogo函数是与操作系统有关的函数，用于完成栈的切换及CPU寄存器的恢复。

从协程g切换回协程g0时，mcall函数用于保存当前协程的执行现场。

goroutine调度的本质就是将**Goroutine(G)**按照一定算法放到CPU上去执行。

CPU感知不到Goroutine，只知道内核线程，所以需要Go调度器将协程调度到内核线程上面去，然后操作系统调度器将内核线程放到CPU上去执行

M是对内核级线程的封装，所以Go调度器的工作就是将G分配到M

Go调度器的实现不是一蹴而就的，它的调度模型与算法也是几经演化，从最初的GM模型、到GMP模型，从不支持抢占，到支持协作式抢占，再到支持基于信号的异步抢占，经历了不断地优化与打磨。

被调度的对象：

```
G的来源
P的runnext(只有1个G，局部性原理，永远会被最先调度执行)
P的本地队列（数组，最多256个G)
全局G队列(链表，无限制)
网络轮询器network poller (存放网络调用被阻塞的G)

P的来源
全局P队列(数组，GOMAXPROCS个P)

M的来源
休眠线程队列(未绑定P，长时间休眠会等待GC回收销毁)
运行线程（绑定P，指向P中的G)
自旋线程（(绑定P，指向M的GO)
```

运行线程数+自旋线程数<=P的数量（GOMAXPROCS），M个数>=P的个数

G的生命周期：G从创建、保存、被获取、调度和执行、阻塞、销毁，步骤如下:

```
步骤1:创建G，关键字go func( )创建G

步骤2∶保存G，创建的G优先保存到本地队列P，如果P满了，则会平衡部分P到全局队列中
保存G的详细流程如下:
执行go func的时候，主线程M0会调用newproc()生成一个G结构体，这里会先选定当前M0上的P结构
每个协程G都会被尝试先放到P中的runnext，若runnext 为空则放到runnext中，生产结束
若runnext满，则将原来runnext中的G踢到本地队列中，将当前G放到runnext中，生产结束
若本地队列也满了，则将本地队列中的G拿出一半，放到全局队列中，生产结束。

步骤3∶唤醒或者新建M执行任务，进入调度循环（步骤4,5,6)

步骤4∶M获取G，M首先从P的本地队列获取G，如果P为空，则从全局队列获取G，如果全局队列也为
空，则从另一个本地队列偷取一半数量的G(负载均衡)，这种从其它P愉的方式称之为work stealing

步骤5: M调度和执行G，M调用G.func()函数执行G
如果M在执行G的过程发生系统调用阻塞（同步)，会阻塞G和M(操作系统限制)，此时P会和当前M解绑，
并寻找新的M，如果没有空闲的M就会新建一个M，接管正在阻塞G所属的P，接着继续执行P中其余的
G，这种阻塞后释放P的方式称之为hand off。当系统调用结束后，这个G会尝试获取一个空闲的P执行，
优先获取之前绑定的P，并放入到这个P的本地队列，如果获取不到P，那么这个线程M变成休眠状态，加
入到空闲线程中，然后这个G会被放入到全局队列中。
如果M在执行G的过程发生网络IO等操作阻塞时(异步)，阻塞G，不会阻塞M。M会寻找P中其它可执行的	
G继续执行，G会被网络轮询器network poller接手，当阻塞的G恢复后，G1从network poller被移回到P的
LRQ中，重新进入可执行状态。异步情况下，通过调度，Go scheduler成功地将I/O的任务转变成了
CPU任务，或者说将内核级别的线程切换转变成了用户级别的goroutine切换，大大提高了效率。

步骤6∶M执行完G后清理现场，重新进入调度循环（将M上运行的goroutine切换为G0，G0负责调度时	
协程的切换)
```

调度策略：

```
抢占式调度
	sysmon检测到协程运行过久(比如sleep，死循环)
	切换到g0，进入调度循环
主动调度
	新起一个协程和协程执行完毕触发调度循环
	主动调用runtime.Gosched()切换到g0，进入调度循环。
	垃圾回收之后。stw之后，会重新选择g开始执行
被动调度
	系统调用(比如文件IO)阻塞(同步)
		阻塞G和M，P与M分离，将P交给其它M绑定，其它M执行P的剩余G。
	网络IO调用阻塞(异步)
		阻塞G，G移动到NetPoller，M执行P的剩余G 
	atomic/mutex/channel等阻塞(异步)
		阻塞G，G移动到channel的等待队列中，M执行P的剩余G
```

使用什么策略来挑选下一个goroutine执行?

由于P中的G分布在runnext、本地队列、全局队列、网络轮询器中，则需要挨个判断是否有可执行的G，大体逻辑如下:

```
每执行61次调度循环，从全局队列获取G，若有则直接返回
从P上的runnext看一下是否有G，若有则直接返回
从P上的本地队列看一下是否有G，若有则直接返回
上面都没查找到时，则去全局队列、网络轮询器查找或者从其他Р中窃取,t一直阻塞直到获取到一个可用
的G为止
```

netpoller中拿到的G是_Gwaiting状态（存放的是因为网络IO被阻塞的G)，从其它地方拿到的是_Grunnable状态

### Goroutine的抢占式调度

非协作式:就是由runtime来决定一个goroutine运行多长时间，如果你不主动让出，对不起，我有手段可以抢占你，把你踢出去，让后面的goroutine进来运行。

基于协作的抢占式调度流程:

```
1.编译器会在调用函数前插入runtime.morestack，让运行时有机会在这段代码中检查是否需要执行抢占
调度
2.Go语言运行时会在垃圾回收暂停程序、系统监控发现Goroutine运行超过10ms，那么会在这个协程设置	
一个抢占标记
3.当发生函数调用时，可能会执行编译器插入的runtime.morestack，它调用的runtime.newstack会检查抢
占标记，如果有抢占标记就会触发抢占让出cpu，切到调度主协程里
```

基于信号的抢占式调度
真正的抢占式调度是基于信号完成的，所以也称为“异步抢占"。不管协程有没有意愿主动让出cpu运行权，只要某个协程执行时间过长，就会发送信号强行夺取cpu运行权。

```
M注册一个SIGURG信号的处理函数:sighandler
sysmon启动后会间隔性的进行监控，最长间隔10ms，最短间隔20us。如果发现某协程独占
P超过10ms，会给M发送抢占信号
M收到信号后，内核执行sighandler函数把当前协程的状态从_Grunning正在执行改成_Grunnable可执	
行，把抢占的协程放到全局队列里，M继续寻找其他goroutine来运行
被抢占的G再次调度过来执行时，会继续原来的执行流
```

### Context结构原理

Context（上下文）是Golang应用开发常用的并发控制技术 ，它可以控制一组呈树状结构的goroutine，每个goroutine拥有相同的上下文。Context 是并发安全的，主要是用于控制多个协程之间的协作、取消操作。

type Context interface {
Deadline() (deadline time.Time, ok bool)
Done() <-chan struct{}
Err() error
Value(key interface{}) interface{}
}

「Deadline」 方法：可以获取设置的截止时间，返回值 deadline 是截止时间，到了这个时间，Context 会自动发起取消请求，返回值 ok 表示是否设置了截止时间。
「Done」 方法：返回一个只读的 channel ，类型为 struct{}。如果这个 chan 可以读取，说明已经发出了取消信号，可以做清理操作，然后退出协程，释放资源。
「Err」 方法：返回Context 被取消的原因。
「Value」 方法：获取 Context 上绑定的值，是一个键值对，通过 key 来获取对应的值。

几个实现context接口的对象：

context.Background()和context.TODO()相似，返回的context一般作为根对象存在，其不可以退出，也不能携带值。要具体地使用context的功能，需要派生出新的context。
context.WithCancel()函数返回一个子context并且有cancel退出方法。子context在两种情况下会退出，一种情况是调用cancel，另一种情况是当参数中的父context退出时，该context及其关联的context都退出。
context.WithTimeout函数指定超时时间，当超时发生后，子context将退出。因此子context的退出有3种时机，一种是父context退出；一种是超时退出；一种是主动调用cancel函数退出。
context.WithDeadline()与context.WithTimeout()返回的函数类似，不过其参数指定的是最后到期的时间。
context.WithValue()函数返回待key-value的子context

具体context内容请看：[context参考](https://blog.csdn.net/qq_43716830/article/details/124141576?spm=1001.2014.3001.5501)

### Context原理

context在很大程度上利用了通道在close时会通知所有监听它的协程这一特性来实现。每一个派生出的子协程都会创建一个新的退出通道，组织好context之间的关系即可实现继承链上退出的传递。

context使用场景：

```
1.RPC调用
2.PipeLine
3.超时请求
4.HTTP服务器的request互相传递数据
```

### Golang内存分配机制

Go语言内置运行时（就是runtime)，抛弃了传统的内存分配方式，改为自主管理。这样可以自主地实现更好的内存使用模式，比如内存池、预分配等等。这样，不会每次内存分配都需要进行系统调用。

设计思想：

```
内存分配算法采用Google的 TCMalloc算法，每个线程都会自行维护一个独立的内存池，进行内存分配时
优先从该内存池中分配，当内存池不足时才会向加锁向全局内存池申请，减少系统调用并且避免不同线程
对全局内存池的锁竞争
把内存切分的非常的细小，分为多级管理，以降低锁的粒度
回收对象内存时，并没有将其真正释放掉，只是放回预先分配的大块内存中，以便复用。只有内存闲置过
多的时候，才会尝试归还部分内存给操作系统，降低整体开销
```

Golang的内存管理组件主要有：mspan、mcache、mcentral和mheap

内存管理单元：mspan

mspan是内存管理的基本单元，该结构体中包含next和 prev两个字段，它们分别指向了前一个和后一个mspan，每个mspan都管理npages个大小为8KB的页，一个span是由多个page组成的，这里的页不是操作系统中的内存页，它们是操作系统内存页的整数倍。

```go
type mspan struct {
next *mspan //后指针
prev *mspan //前指针
startAddr uintptr //管理页的起始地址，指向page
npages uintptr //页数
spanclass spanClass //规格，字节数
...
}
type spanclass uint8
```

mspan的种类

```bash
// class  bytes/obj  bytes/span  objects     tail waste  max waste
//
//     1          8        8192     1024           0     87.50%
//     2         16        8192      512           0     43.75%
//     3         24        8192      341           8     29.24%
//     4         32        8192      256           0     21.88%
//     5         48        8192      170          32     31.52%
//     6         64        8192      128           0     23.44%
// 略...
//    62      20480       40960        2           0      6.87%
//    63      21760       65536        3         256      6.25%
//    64      24576       24576        1           0     11.45%
//    65      27264       81920        3         128     10.00%
//    66      28672       57344        2           0      4.91%
//    67      32768       32768        1           0     12.50%
```

线程缓存：mcache

mache管理线程在本地缓存的mspan，每个goroutine绑定的P都有一个mcache字段

```go
type mcache struct{
	alloc [numSpanClasses]*mspan
}

_NumSizeClasses=68
numSpanClassed=_NumSizeClassed<<1
```

mcache用Span classes作为索引管理多个用于分配的mspan ，它包含所有规格的mspan。它是_NumSizeClasses 的2倍，也就是68X2=136，其中*2是将spanClass分成了有指针和没有指针两种，方便与垃圾回收。对于每种规格，有2个mspan，一个mspan不包含指针，另一个mspan则包含指针。对于无指针对象的mspan在进行垃圾回收的时候无需进一步扫描它是否引用了其他活跃的对象。
mcache在初始化的时候是没有任何mspan资源的，在使用过程中会动态地从mcentral申请,
之后会缓存下来。当对象小于等于32KB大小时，使用mcache 的相应规格的mspan进行分配。

mcache拥有一个allocCache字段，作为一个位图，来标记span中的元素是否被分配。
当请求的mcache中对应的span没有可以使用的元素时，需要从mcentral中加锁查找，分别遍历mcentral中有空闲元素的nonempty链表和没有空闲元素的empty链表。没有空闲元素的empty链表可能会有被垃圾回收标记为空闲的还未来得及清理的元素，这些元素也是可用的

中心缓存：mcentral

mcentral管理全局的mspan供所有线程使用，全局mheap变量包含central字段，每个mcentral结构都维护在mheap结构内

```go
type mcentral struct{
	spanclass spanClass //指当前规格大小
	partial [2]spanSet //有空闲object的mspan列表
	full [2]spanSet //没有空闲object的mspan列表
}
```

每个mcentral管理一种spanClass的mspan，并将有空闲空间和没有空闲空间的mspan分开管理。partial和full的数据类型为spanSet，表示mspans集，可以通过pop、push来获得mspans

```go
type spanSet struct {
spineLock mutex
spineunsafe.Pointer //指向[]span的指针
spineLen uintptr  //Spine array length,accessed atomically
spineCap uintptr  // Spine array cap,accessed under lock
index headTailIndex//前32位是头指针，后32位是尾指针
```

简单说下mcache 从 mcentral获取和归还mspan的流程:

```
获取:加锁，从 partial链表找到一个可用的mspan；并将其从 partial链表删除；将取出的mspan加入到full
链表；将mspan返回给工作线程，解锁。
归还:加锁，将mspan从full链表删除；将mspan 加入到partial链表，解锁。
```

页堆：mheap

mheap管理Go的所有动态分配内存，可以认为是Go程序持有的整个堆空间，全局唯一

```go
type mheap struct {
lock mutex //全局锁
pages pageAlloc //页面分配的数据结构
allspans []*mspan //所有通过mheap_申请的mspans
	//堆
arenas [1 << arenaL1Bits]*[1 << arenaL2Bits]*heapArena
//所有中心缓存mcentral
central [numSpanClasses]struct {
mcentral mcentral
pad [cpu.CacheLinePadSize - unsafe.Sizeof(mcentral(})%cpu.CacheLinePadSize]byte
}
...
}
```

所有mcentral的集合则是存放于mheap中的。mheap里的arena区域是堆内存的抽象，运行时会将 8KB 看做一页，这些内存页中存储了所有在堆上初始化的对象。运行时使用二维的runtime.heapArena数组管理所有的内存，每个runtime.heapArena都会管理64MB的内存。

当申请内存时，依次经过mcache和mcentral都没有可用合适规格的大小内存，这时候会向mheap申请一块内存。然后按指定规格划分为一些列表，并将其添加到相同规格大小的 mcentral 的非空闲列表后面

分配流程：

```
首先通过计算使用的大小规格
然后使用mcache 中对应大小规格的块分配。
如果mcentral中没有可用的块，则向mheap申请，并根据算法找到最合适的mspan 。
如果申请到的mspan超出申请大小，将会根据需求进行切分，以返回用户所需的页数。剩余的页构成一个		
新的mspan 放回mheap 的空闲列表。
如果mheap中没有可用span，则向操作系统申请一系列新的页(最小1MB)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8c39769122114eaa97d6323f8c902384.png)

### 竞态、内存逃逸

1.资源竞争，就是在程序中，同一块内存同时被多个 goroutine 访问。我们使用 go build、go run、go test 命令时，添加 -race 标识可以检查代码中是否存在资源竞争。

解决这个问题，我们可以给资源进行加锁，让其在同一时刻只能被一个协程来操作。

sync.Mutex
sync.RWMutex

2.一般来说，局部变量会在函数返回后被销毁，因此被返回的引用就成为了"无所指""的引用，程序会进入未知状态。

但这在Go中是安全的，Go编译器将会对每个局部变量进行逃逸分析。如果发现局部变量的作用域超出该函数，则不会将内存分配在栈上，而是分配在堆上，因为他们不在栈区，即使释放函数，其内容也不会受影响。

```go
func add(x, y int) *int {
	res := x + y
	return &res
}
```

这个例子中，函数add局部变量res发生了逃逸。res作为返回值，在main函数中继续使用，因此res指向的内存不能够分配在栈上，随着函数结束而回收，只能分配在堆上。

编译时可以借助选项 -gcflags=-m，查看变量逃逸的情况

逃逸场景：

指针逃逸
栈空间不足逃逸
变量大小不确定

```go
package main

func escape(){
	number:=3
	s:=make([]int,number)
}
```

动态类型逃逸

动态类型就是编译期间不确定参数的类型、参数的长度也不确定的情况下就会发生逃逸
空接口interface(可以表示任意的类型，如果函数参数为interface)，编译期间很难确定其参数的具体类型，也会发生逃逸。

```go
func escape(a interface{}) {
	fmt.Println(a)
}

func escape2() {
	fmt.Println(111)
}


func main() {
	escape(1)
}
```

闭包引用对象逃逸

### golang内存对齐机制

为了能让CPU可以更快的存取到各个字段，Go编译器会帮你把struct结构体做数据的对齐。所谓的数据对齐，是指内存地址是所存储数据大小（按字节为单位）的整数倍，以便CPU可以一次将该数据从内存中读取出来。编译器通过在结构体的各个字段之间填充一些空白已达到对齐的目的。

不同硬件平台占用的大小和对齐值都可能是不一样的，每个特定平台上的编译器都有自己的默认“对齐系数”，32位系统对齐系数是4，64位系统对齐系数是8
不同类型的对齐系数也可能不一样，使用Go 语言中的unsafe.Alignof函数可以返回相应类型的对齐系数，对齐系数都符合2^n这个规律，最大也不会超过8

对齐原则：

```
1.结构体变量中成员的偏移量必须是成员变量大小和成员对齐系数两者最小值的整数倍
2.整个结构体的地址必须是最大字节和编译器默认对齐系数两者最小值的整数倍（结构体的内存占用是1/4/8/16 byte...）
3.struct{}放在结构体中间不进行对齐，放在结构体最后一个字段则要根据最大字节和编译器默认对齐系数两者最小值
来进行字段对齐
type T2 struct{
	i8 int8
	i64 int64
	i32 int32
}

type T3 struct{
	i8 int8
	i32 int32
	i64 int64
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/31b7ef13828a409283e095e056d9f12e.png)

```go
type C struct {
	a struct{}
	b int64
	c int64
}

type D struct {
	a int64
	b struct{}
	c int64
}

type E struct {
	a int64
	b int64
	c struct{}
}

type F struct {
	a int32
	b int32
	c struct{}
}

func main() {
	fmt.Println(unsafe.Sizeof(C{})) // 16
	fmt.Println(unsafe.Sizeof(D{})) // 16
	fmt.Println(unsafe.Sizeof(E{})) // 24
  	fmt.Println(unsafe.Sizeof(F{})) // 12
}
```

### golang中new和make的区别？

var声明值类型的变量时，系统会默认为他分配内存空间，并赋该类型的零值
如果是指针类型或者引用类型的变量，系统不会为它分配内存，默认是nil。

1.make 仅用来分配及初始化类型为 slice、map、chan 的数据。
2.new 可分配任意类型的数据，根据传入的类型申请一块内存，返回指向这块内存的指针，即类型 *Type。
3.make 返回引用，即 Type，new 分配的空间被清零， make 分配空间后，会进行初始。
4.make函数返回的是slice、map、chan类型本身
5.new函数返回一个指向该类型内存地址的指针

### Golang的slice的实现原理

slice不是线程安全的
切片是基于数组实现的，底层是数组，可以理解为对底层数组的抽象

```go
type slice struct{
	array unsafe.Pointer
	len int
	cap int
}
```

slice占24个字节
array：指向底层数组的指针，占用8个字节
len:切片的长度，占用8个字节
cap：切片的容量，cap总是大于等于len，占用8个字节

初始化slice调用的是runtime.makeslice，makeslice函数的工作主要就是计算slice所需内存大小，然后调用mallocgc进行内存的分配

所需内存的大小=切片中元素大小*切片的容量

### Golang中，array和slice的区别

1)数组长度不同

```
数组初始化必须指定长度，并且长度就是固定的
切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大
```

2)函数传参不同

```
数组是值类型，将一个数组赋值给另一个数组时，传递的是一份深拷贝，函数传参操作	
都会复制整个数组数据，会占用额外的内存，函数内对数组元素值的修改，不会修改原
数组内容。

切片是引用类型，将一个切片赋值给另一个切片时，传递的是一份浅拷贝，函数传参操作不会拷贝整个切
片，只会复制len和cap，底层共用同一个数组，不会占用额外的内存，函数内对数组元素值的修改，会修
改原数组内容。
```

3)计算数组长度方式不同

```
数组需要遍历计算数组长度，时间复杂度为O(n)
切片底层包含len字段，可以通过len)计算切片长度，时间复杂度为O(1)
```

### Golang的map实现原理

Go中的map是一个指针，占用8个字节，指向hmap结构体，map底层是基于哈希表+链地址法存储的。

map的特点：

```
1.键不能重复
2.键必须可哈希（目前我们已学的数据类型中，可哈希的有：int/bool/float/string/array）
3.无序
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d1ddf7d3ef9c4e08810ab8c9dcdd1132.png)

源码包中src/runtime/map.go定义了hmap的数据结构
hmap包含若干个结构为bmap的数组，每个bmap底层都采用链表结构，bmap通常叫其bucket
![在这里插入图片描述](https://img-blog.csdnimg.cn/17568bc248a54a54915ee1d828465dfb.png)

```go
// A header for a Go map.
type hmap struct {
count int
//代表哈希表中的元素个数，调用len(map)时，返回的就是该字段值。
flags uint8
//状态标志是否处于正在写入的状态等
B uint8
//buckets(桶)的对数
//如果B=5，则buckets数组的长度=2^B=32，意味着有32个桶
noverflow uint16
//溢出桶的数量
hash0 uint32 
//生成hash的随机数种子
buckets unsafe.Pointer
//指向buckets数组的指针，数组大小为2^B，如果元素个数为0，它为nil.
oldbuckets unsafe.Pointer
//如果发生扩容，oldbuckets是指向老的buckets数组的指针，老的buckets数组大小是新的buckets的1/2;非扩容状态下，它为ni1.
nevacuate uintptr
//表示扩容进度。小于此地址的buckets代表已搬迁完成。
extra *mapextra
//存储溢出桶，这个字段是为了优化GC扫描面设计的
```

bmap 就是我们常说的”桶"，一个桶里面会最多装8个key，这些key之所以会落入同一个桶，是因为它们经过哈希计算后，哈希结果的低B位是相同的，关于key的定位我们在map的查询中详细说明。在桶内，又会根据 key 计算出来的hash值的高8位来决定key到底落入桶内的哪个位置（一个桶内最多有8个位置)。

```go
//A bucket for a Go map.
type bmap struct {
tophash [bucketCnt]uint8 
//len为8的数组
//用来快速定位key是否在这个bmap中
//一个桶最多8个槽位，如果key所在的tophash值在tophash中，则代表该key在这个桶中
}
```

上面bmap结构是静态结构，在编译过程中runtime.bmap会拓展成以下结构体:

```go
type bmap struct{
tophash [8]uint8
keys [8]keytype
// keytype由编译器编译时候确定
values [8]elemtype
// elemtype由编译器编译时候确定
overflow uintptr
//overflowi的下一个bmap，overflow是uintptr而不是*bmap类型，保证bmap完全不含指针，是为了减少gc，溢出桶存储到extra字段中
}
```

tophash就是用于实现快速定位key的位置，在实现过程中会使用key的hash值的高8位作为tophash值，存放在bmap的tophash字段中
tophash字段不仅存储key哈希值的高8位，还会存储一些状态值，用来表明当前桶单元状态，这些状态值都是小于minTopHash的

```go
emptyRest= 0 //表明此桶单元为空，且更高索引的单元也是空
emptyOne=1 //表明此桶单元为空
evacuatedX= 2 //用于表示扩容迁移到新桶前半段区间
evacuatedY= 3 //用于表示扩容迁移到新桶后半段区间
evacuatedEmpty = 4 //用于表示此单元已迁移
minTopHash= 5 // key的tophash值与桶状态值分割线值，小于此值的一定代表着桶单元的状态，大于此值的一定是key对应的tophash值

func tophash(hash uintptr) uint8 {
top := uint8(hash >> (goarch.PtrSize*8 - 8))
if top < minTopHash {
top += minTopHash
}
return top
}
```

为了避免key哈希值的高8位值和这些状态值相等，产生混淆情况，所以当key给哈希值高8位若小于minTopHash时候，自动将其值加上minTopHash作为该key的tophash。

注意到key 和value是各自放在一起的，并不是 key/value/key/valuer …这样的形式，当key和alue类型不一样的时候，key和value占用掌节大小不一样，使用keylvalue这种形式可能会因为内存对齐导致内存空间浪费，所以Go采用key和value分开存储的设计，更节省内存空间

map的初始化过程：

```
1.创建一个hmap结构体对象
2.生成一个哈希因子hash0并赋值到hmap对象中（用于后续为key创建哈希值）
3.根据hint=10，并根据算法规则来创建B，此时的B为1
4.根据B去创建桶(bmap对象)并存放在bucket数组中。当前的Bmap的数量为2
	B<4时，根据B创建桶的个数的规则为：2^B（标准桶）
	B>=4时，根据B创建桶的个数的规则为：2^B+2^(B-4) （标准桶+溢出桶）
```

golang map的渐进式扩容：

```
Go map扩容，数据迁移不是一次性迁移，而是等到访问到具体某个bucket时才将数据从旧bucket中迁移到新bucket
中

1.一次性迁移会涉及到cpu资源和内存资源的占用，在数据量较大时，会有较大的延时，影响正常业务逻辑。因此Go
采用渐进式的数据迁移，每次最多迁移两个bucket的数据到新的buckets中（一个是当前访问key所在的bucket，然后
再多迁移一个bucket）
2.cpu资源：扩容时需要迁移map中oldbuckets的元素，其中的rehash操作会消耗cpu的计算资源，有可能会影响到用
户协程的调度
```

### Golang的map为什么是无序的？

使用range多次遍历map时输出的key和vabue 的顺序可能不同。这是Go语言的设计者们有意为之，旨在提示开发者们，Go底层实现并不保证map遍历顺序稳定，请大家不要依赖range遍历结果顺序

主要原因有2点:

```
1.map在遍历时，并不是从固定的0号bucket开始遍历的，每次遍历，都会从一个随机值序号的bucket，
再从其中随机的cell开始遍历
2.map遍历时，是按序遍历bucket，同时按需遍历bucket中和其overflow bucket中的cell。但是map在扩
容后，会发生key的搬迁，这造成原来落在一个buket中的Key,搬迁后，有可能会落到其他bucket中了，
从这个角度看，遍历map的结果就不可能是按照原来的顺序了
```

map本身是无序的，且遍历时顺序还会被随机化，如果想顺序遍历map，需要对 map key先排序，再按照key 的顺序遍历map。

### Golang的map如何查找？

map的查找通过生成汇编码可以知道，根据key的不同类型/返回参数，编译器会将查找函数用更具体的函数替换，以优化效率：

![在这里插入图片描述](https://img-blog.csdnimg.cn/729a32be99a14158a7cbc127dd1a323a.png)

1.写保护检测

```
函数首先会检查map的标志位flags。如果flags的写标志位此时被置1了，说明有其他协程在执行‘写’操	
作，进而导致程序panic，这也说明了map不是线程安全的
if h.flags&&hashWriting!=0{
	throw("concurrent map read and map write")
}
```

2.计算hash值

```go
hash:=t.hasher(key,uintptr(h.hash0))
key经过哈希函数计算后，得到的哈希值如下，不同类型的key会有不同的hash函数
```

3.找到hash对应的bucket

```
bucket定位：哈希值的低B个bit位，用来定位key所存放的bucket
如果当前正在扩容中，并且定位到的旧bucket数据还未完成迁移，则用旧的bucket（扩容前的bucket）
hash:=t.hasher(key, uintprt(h.hash0))
m=bucketMask(h.B)
b:=(*bmap)(add(h.buckets,(hash&m)*uintptr(t.bucketsize))
if c:=h.oldbucket;c!=nil{
	if !h.sameSizeGrow(){
	m>>=1
}
oldb:=(*bmap)(add(c,(hash&m)*uintptr(t.bucketsize)))
if !evacuated(oldb){
	b=oldb
}
}
```

4.遍历bucket查找

tophash值定位︰哈希值的高8个bit位，用来快速判断key是否已在当前bucket中(如果不在的话，需要去bucket的overflow中查找）
用步骤2中的hash值，得到高8个bit位，也就是10010111，转化为十进制，也就是151

```go
top := tophash(hash)
func tophash(hash uintptr) uint8 {
top := uint8(hash >> (goarch.PtrSize*8 - 8))
if top < minTopHash {
top += minTopHash)
}
return top
}
上面函数中hash是64位的，sys.PtrSize值是8，所以 top:= uint8(hash >> (sys.Ptrsize*8 - 8))等效top = 
uint8(hash >> 56)，最后top取出来的值就是hash的高8位值

在bucket 及bucket的overflow中寻找tophash值(HOB hash)为151*的槽位，即为key所在位置，找到了空
槽位或者2号槽位，这样整个查找过程就结束了，其中找到空槽位代表没找到。
for ;b != nil; b = b.overflow(t) {
for i := uintptr(0); i < bucketCnt; i++ {
if b.tophash[i] != top {
//未被使用的槽位，插入
if b.tophash[i] == emptyRest {
	break bucketloop
}
	continue
)
//找到tophash值对应的的key
k := add(unsafe.Pointer(b),dataOffset+i*uintptr(t.keysize))
if t.key.equal(key , k){
e:=add(unsafe.Pointer(b),dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
return e
	}
  }
}	
```

5.返回key对应的指针

如果通过上面的步骤找到了key对应的槽位下标i，我们再详细分析下key/value值是如何获取的：

```go
dataOffset=unsafe.Offsetof(struct{
	b bmap
	v int64
}{},v)

bucketCnt=8

//key定位公式
k:=add(unsafe.Pointer(b),dataOffset+i*uintptr(t.keysize))

//value定位公式
v:=add(unsafe.Pointer(b),dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
```

### 为什么Golang的Map的负载因子是6.5？

什么是负载因子?

```
负载因子(load factor)，用于衡量当前哈希表中空间占用率的核心指标，也就是每个bucket桶存储的平均	
元素个数。
负载因子=哈希表存储的元素个数/桶个数
```

另外负载因子与扩容、迁移等重新散列(rehash)行为有直接关系:

```
在程序运行时,会不断地进行插入、删除等，会导致 bucket不均，内存利用率低，需要迁移。
在程序运行时，出现负载因子过大，需要做扩容，解决 bucket过大的问题。
```

负载因子是哈希表中的一个重要指标，在各种版本的哈希表实现中都有类似的东西，主要目的是为了平衡buckets 的存储空间大小和查找元素时的性能高低，在接触各种哈希表时都可以关注一下，做不同的对比，看看各家的考量。

Go官方发现:装载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。反之，装载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数
根据这份测试结果和讨论，Go官方取了一个相对适中的值，把Go中的 map的负载因子硬编码为6.5，这就是6.5的选择缘由。
这意味着在Go语言中，当map存储的元素个数大于或等于6.5*桶个数时，就会触发扩容行为。

### Golang map如何扩容

扩容时机:

在向map插入新key的时候，会进行条件检测，符合下面这2个条件，就会触发扩容

```go
if !h.growing() &&(overLoadFactor(h.count+1,h.B)||tooManyOverflowBuckets(h.noverflow， h.8)){
hashGrow(t, h)
goto again // Growing the table invalidates everything, so try again
}

//判断是否在扩容
func (h *hmap) growing( ) bool {
return h.oldbuckets != nil
}
```

扩容条件：

```
条件1：超过负载 map元素个数>6.5*桶个数
func overLoadFactor(count int, B uint8) bool{
	return count > bucketCnt && uintptr(count)>loadFactor*bucketShift(B)
}
其中

bucketCnt=8，一个桶可以装的最大元素个数
loadFactor=6.5，负载因子，平均每个桶的元素个数
bucketShift(8), 桶的个数

条件2：溢出桶太多
当桶总数<2^15时，如果溢出桶总数>=桶总数，则认为溢出桶过多
当桶总数>=2^15时，直接与2^15比较，当溢出桶总数>=2^15时，即认为溢出桶太多了
```

对于条件2，其实算是对条件1的补充。因为在负载因子比较小的情况下，有可能map的查找和插入效率也很低，而第1点识别不出来这种情况。

表面现象就是负载因子比较小比较小，即 map里元素总数少，但是桶数量多（真实分配的桶数量多，包括大量的溢出桶）。比如不断的增删，这样会造成overflow的bucket数量增多，但负载因子又不高，达不到第1点的临界值，就不能触发扩容来缓解这种情况。这样会造成桶的使用率不高，值存储得比较稀疏，查找插入效率会变得非常低，因此有了第2扩容条件。

扩容机制：

```
1.双倍扩容：针对条件1，新建一个buckets数组，新的buckets大小是原来的2倍，然后旧的buckets数据
搬迁到新的buckets
2.等量扩容：针对条件2，并不扩大容量，buckets数量维持不变，重新做一遍类似双倍扩容的搬迁操作，
把松散的键值对重新排列一次，使得同一个bucket中的key排列地更紧密，节省空间，提高buckets利用
率，进而保证更快的存取。该方法我们称之为等量扩容。
```

### Golang的sync.Map

Go语言的 sync.Map支持并发读写，采取了“空间换时间”的机制，冗余了两个数据结构，分别是: read和dirty

```go
type Map struct {
mu Mutex  //互斥锁
read atomic.Value  //无锁化读，包含两个字段：m map[interface}l*entry数据和amended bool标识只读map是否缺失数据
dirty map[interface}*entry //存在锁的读写
misses int //无锁化读的缺失次数
}
type entry struct{
	p unsafe.Pointer
}
```

kv中的value，统一采用unsafe.Pointer的形式进行存储，通过entry.p的指针进行链接。
entry.p指向分为三种情况：

```
1.存活态：正常指向元素
2.软删除态：指向nil
3.硬删除态：指向固定的全局变量expunged
```

存活态很好理解，即key-entry对仍未删除;
nil态表示软删除，read map和dirty map底层的 map结构仍存在 key-entry对，但在逻辑上该key-entry对已经被删除，因此无法被用户查询到;
expunged态表示硬删除，dirty map 中已不存在该key-entry 对.

无锁化读的m是dirty的子集，amended标识是true时代表缺失数据，此时再去读dirty，并把misses加1，当misses到一定阈值之后，同步dirty到read的m中

range会同步dirty到read的m中

读：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e607b529516043d4a2d18aa56e9a585d.png)

对比原始map:

```
和原始map+RWLock的实现并发的方式相比，减少了加锁对性能的影响。它做了一些优化:可以无锁访问
read map，而且会优先操作read map，倘若只操作read map就可以满足要求，那就不用去操作write 
map(dirty)，所以在某些特定场景中它发生锁竞争的频率会远远小于map+RWLock的实现方式
```

优点:

```
适合读多写少的场景
```

缺点∶

```
写多的场景，会导致read map缓存失效，需要加锁，冲突变多，性能急剧下降
```

### Golang中对nil的Slice和空Slice的处理是一致的吗?

首先Go的JSON 标准库对 nil slice 和 空 slice 的处理是不一致。
1.slice := make([]int,0）：slice不为nil，但是slice没有值，slice的底层的空间是空的。
2.slice := []int{} ：slice的值是nil，可用于需要返回slice的函数，当函数出现异常的时候，保证函数依然会有nil的返回值。

### Golang的内存模型中为什么小对象多了会造成GC压力？

通常小对象过多会导致GC三色法消耗过多的GPU。优化思路是，减少对象分配。

### Channel的实现原理

概念:

```
Go中的channel是一个队列，遵循先进先出的原则，负责协程之间的通信(Go语言提倡不要通过共享内存
来通信，而要通过通信来实现内存共享，CSP(CommunicatingSequential Process)并发模型，就是通过
goroutine和channel来实现的)
```

使用场景;

```
1.停止信号监听
2.定时任务
3.生产方和消费方解耦
4.控制并发数
```

底层数据结构:

```
通过var声明或者make函数创建的channel变量是一个存储在函数栈帧上的指针，占用8个字节，指向堆上	
的hchan结构体，源码包中src/ runtime/ chan.go定义了hchan的数据结构:
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b63477d1ef954b9db50237d5cf2b899c.png)
hchan结构体:

```go
type hchan struct i
	closed uint32 // channel是否关闭的标志
	elemtype *_typel// channel中的元素类型
 
 // channel分为无缓冲和有缓冲两种。
 //对于有缓冲的channel存储数据，使用了ring buffer(环形缓冲区）来缓存写入的数据，本质是循环数组
 //为啥是循环数组?普通数组不行吗，普通数组容量圄定更适合指定的空间，弹出元素时，普通数组需要全部都前移
 //当下标超过数组容量后会回到第一个位置，所以需要有两个字段记录当前读和写的下标位置
buf unsafe.Pointer //指向底层循环数组的指针(环形缓冲区)
qcount uint //循环数组中的元素数量
dataqsiz uint //循环数组的长度
elemsize uint16 //元素的大小
sendx uint //下一次写下标的位置
recvx uint //下一次读下标的位置

//尝试读取channel或向channel写入数据面被阻塞的goroutine
recvq waitq //读等待队列
sendq waitq //写等特队列
lock mutex //互斥锁
}
```

等待队列：
双向链表，包含一个头节点和一个尾节点
每个节点是一个sudog结构体变量，记录哪个协程在等待，等待的是哪个channel，等待发送/接收的数据在哪里

```go
type waitq struct{
	first *sudog
	last *sudog
}

type sudog struct{
	g *g //哪个协程在等待
	next *sudog
	prev *sudog
	elem unsafe.Pointer //等待发送/接收的数据在哪里
	c *hchan //等待的是哪个channel
	...
}
```

操作：

创建
使用make(chan T, cap)来创建channel，make语法会在编译时，转换为makechan64和makechan

```go
func makechan64(t *chantype, size int64) *hchan {
if int64(int(size)) != size {
panic(plainError("makechan: size out of range" ))
)
return makechan(t,int(size))
)
```

make创建channel时会做一些检查：

```
1.元素大小不能超过64k
2.元素的对齐大小不能超过maxAlign也就是8字节
3.计算出来的内存是否超过限制
```

创建时的策略：

```
1.如果是无缓冲的channel，会直接给hchan分配内存
2.如果是有缓冲的channel，并且元素不包含指针，那么会为hchan和底层数组分配一段连续的地址
3.如果是有缓冲的channel，并且元素包含指针，那么会为hchan和底层数组分别分配地址
```

发送：

```
发送操作，编译时转换为runtime.chansend函数
func chansend(c *hchan,ep unsafe.Pointer,block bool,callerpc uintptr) bool
```

阻塞式：

```
调用chansend函数，并且block=true
ch<-10
```

非阻塞式：

```
调用chansend函数，并且block=false
select{
	case ch <- 10:
	...
	default
}
```

向channel中发送数据时大概分为两大块：检查和数据发送，数据发送流程如下：

```
1.如果channel的读等待队列存在接收者goroutine
	将数据直接发送给第一个等待的goroutine，唤醒接收的goroutine
2.如果channel的读等待队列不存在接收者goroutine
	如果循环数组buf未满，那么将会把数据发送到循环数组buf的队尾
	如果循环数组buf已满，这个时候就会走阻塞发送的流程，将当前goroutine加入写等待队列，并挂起等	
	待唤醒
```

接收：

```
发送操作，编译时转换为runtime.chanrecv函数
func chanrecv(c *hchan,ep unsafe.Pointer,block bool)(selected, received bool)
```

也包含阻塞和非阻塞式

向channel中接收数据时大概分为两大块，检查和数据发送，而数据接收流程如下:

```
如果channel的写等待队列存在发送者goroutine
	如果是无缓冲channel，直接从第一个发送者goroutine那里把数据拷贝给接收变量，唤醒发送的
	goroutine
	如果是有缓冲channel(已满)，将循环数组buf的队首元素拷贝给接收变量，将第一个发送者goroutine的	
	数据拷贝到buf循环数组队尾，唤醒发送的goroutine

如果channel的写等待队列不存在发送者goroutine
	如果循环数组buf非空，将循环数组buf的队首元素拷贝给接收变量
	如果循环数组buf为空，这个时候就会走阻塞接收的流程，将当前goroutine 加入读等待队列，并挂起等
	待唤醒
```

当通道的元素类型包含指针时，确实需要单独分配内存空间的原因主要有以下两个方面：

```
1.避免竞争条件：在多个Goroutine之间共享内存时，需要确保内存的安全和一致性。当通道的元素类型为指针时，不同的Goroutine可
能会同时修改或访问同一个指针指向的内存区域，造成竞争条件。为了避免这种情况，通道会对每个元素都单独分配内存空间，确保
每个Goroutine都持有独立的内存副本，避免竞争条件的发生。

2.管理生命周期：通道作为一种线程安全的数据传输机制，其内部会负责管理元素的生命周期。当通道的元素类型为指针时，通道会确	
保在元素被发送或接收完成后，正确释放对应的内存空间。通过单独分配内存空间，通道可以更好地管理元素的生命周期，防止内存
泄漏等问题的发生。
```

channel是并发安全的：

```
通道的发送和接收操作是原子的，即一个完整的发送或接收操作是一个原子操作，不会被其他goroutine中断。

当一个goroutine向channel发送数据时，如果channel已满，则发送操作会被阻塞，直到有其他goroutine从该channel中接收数据后释放
空间，发送操作才能继续执行。在这种情况下，channel内部会获取一个锁，保证只有一个goroutine能够往其中写入数据。

同样地，当一个goroutine从channel中接收数据时，如果channel为空，则接收操作会被阻塞，直到有其他goroutine向该channel中发送
数据后才能继续执行。在这种情况下，channel内部也会获取一个锁，保证只有一个goroutine能够从其中读取数据。
```

### Channel是同步的还是异步的？

Channel是异步进行的, channel存在3种状态：

1.nil，未初始化的状态，只进行了声明，或者手动赋值为nil
2.active，正常的channel，可读或者可写
3.closed，已关闭，千万不要误认为关闭channel后，channel的值是nil，对已关闭channel读写都会panic
![在这里插入图片描述](https://img-blog.csdnimg.cn/1fc3bad2b2b84efe97b35aabc04fa039.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCP5byg5ZCM5a2m6K-l5Yqq5Yqb5LqG,size_13,color_FFFFFF,t_70,g_se,x_16)

### Channel死锁场景

1.非缓存channel只写不读
2.非缓存channel读在写后面
3.缓存channel写入超过缓冲区数量
4.空读
5.多个协程相互等待

1.非缓存channel只写不读

```go
func deadlock1( ) {
ch:=make(chan int)
ch<-3 //这里会发生一直阻塞的情况，执行不到下一句
}
```

2.非缓存channel读在写后面

```go
func deadlock2() {
ch:=make(chan int)
ch<-3 //这里会发生一直阻塞的情况，执行不到下一句
num:=<-ch
fmt.Println("num=",num)
}
```

3.缓存channel写入超过缓冲区数量

```go
func deadlock3() {
ch:=make(chan int,3)
ch<-3
ch<-4
ch<-5
ch<-6  //这里会发生一直阻塞的情况
}
```

4.空读

```go
func deadlock4() {
ch:=make(chan int)
fmt.Println(<-ch)
}
```

5.多个协程相互等待

```go
func deadlock5() {
ch1 := make(chan int)
ch2 := make(chan int)//互相等对方造成死锁
go func() {
for {
select {
case num := <-ch1:
fmt.Println("num=" , num)
ch2 <- 100
}
}
)()
for {
select {
case num := <-ch2:
fmt.Println ("num=", num)
ch1 <- 300
}
)
}
```

### Golang互斥锁的实现原理

互斥锁对应的底层结构是sync.Mutex结构体，位于src/sync/mutex.go中

```go
type Mutex struct{
	state int32
	sema uint32
}
```

state表示锁的状态，有锁定、被唤醒、饥饿模式等，并且是用state的二进制来标识的，不同模式下会有不同的处理方式
![在这里插入图片描述](https://img-blog.csdnimg.cn/0fcf54bb4744426782dfb5df5bb85c7d.png)
state字段表示当前互斥锁的状态信息，它是int32类型，其低三位的二进制位均有相应的状态含义。

```
mutexLocked是state中的低1位，用二进制表示为0001(为了方便，这里只描述后4位)，它代表该互斥锁
是否被加锁。
mutexwoken是低2位，用二进制表示为0010，它代表互斥锁上是否有被唤醒的	
goroutine
mutexstarving是低3位，用二进制表示为0100，它代表当前互斥锁是否处于饥饿模式。
state剩下的29位用于统计在互斥锁上的等待队列中goroutine数目(waiter)。
```

sema表示信号量，mutex阻塞队列的定位是通过这个变量来实现的，从而实现goroutine的阻塞和唤醒
sema 锁是信号量/信号锁。
核心是一个 uint32 值，含义是同时可并发的数量。
每一个 sema 锁都对应一个 SemaRoot 结构体，其中有一个平衡二叉树用于协程队列：
获取 sema 锁，其实就是将一个 uint32 的值减一，如果这个操作成功，便获取到锁。
释放 sema 锁，将 unit32 加一 atomic.Xadd(addr, 1)，如果这个操作成功，便获释放锁。
获取锁的时候，如果 uint32 值一开始就为 0，或减到了 0，则协程休眠: goparkunlock()，进入堆树等待：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3cbfdbbe4ff04541b6f232d53ad1c455.png)
加锁过程：

```
1.通过原子操作CAS获取锁
2.获取失败则判断是否满足自旋条件，满足则进入自旋状态
3.自旋超过一定次数之后获取sema，获取sema成功后将其记录在平衡树上，并将WaiterShift值置为1，进
入阻塞状态，等待被唤醒
4.锁被释放后会唤醒进入阻塞态的goroutine，被唤醒的goroutine重试之前的操作
5.请求锁的等待时间过长，则进入饥饿模式，不用抢直接拿到锁
```

解锁过程：

```
1.通过原子操作add解锁
2.如果仍有goroutine在等待，唤醒等待的goroutine
3.如果是饥饿模式下，让其直接获取到锁
```

### Goroutine的枪锁模式

1.正常模式(非公平锁)

```
1.在刚开始的时候，是处于正常模式(Barging)，也就是，当一个G1持有着一个锁的时候，G2会自旋地去	
尝试获取这个锁
2.当自旋超过4次还没有能获取到锁的时候，这个G2就会被加入到获取锁的等待队列里面，并阻塞等待
唤醒goroutine 竞争锁，新请求锁的 goroutine具有优势：它正在CPU上执行，而且可能有好几个，所以刚
唤醒的 goroutine有很大可能在锁竞争中失败，长时间获取不到锁，就会切换  到饥饿模式
```

2.饥饿模式(公平锁)

当一个goroutine等待锁时间超过1毫秒时，它可能会遇到饥饿问题。在版本1.9中，这种场景下Go Mutex切换到饥饿模式(handoff)，解决饥饿问题。

```
starving = runtime_nanotime( )-waitStartTime > 1e6

队列中排在第一位的goroutine(队头)，同时饥饿模式下，新进来的goroutine不会参与抢锁也不会进入自旋
状态，会直接进入等待队列的尾部,这样很好的解决了老的goroutine一直抢不到锁的场景
```

那么也不可能说永远的保持一个饥饿的状态，总归会有吃饱的时候，也就是总有那么一刻Mutex会回归到正常模式，那么回归正常模式必须具备的条件有以下几种:

```
1.G的等待的时间是否小于1ms
2.等待队列已经全部清空了
```

当满足上述两个条件的任意一个的时候，Mutex会切换回正常模式，而Go的抢锁的过程，就是在这个正常模式和饥饿模式中来回切换进行的.

```go
delta := int32(mutexLocked - 1<<mutexwaiterShift)
if !starving || old>>mutexwaiterShift == 1 {
delta -= mutexStarving
}
atomic.AddInt32(&m.state, delta)
```

### 读写锁原理

RWMutex的5个方法：
Lock/Unlock——写操作时调用的方法
RLock/RUnlock——读操作时调用的方法
RLocker——读操作时返回的Locker接口的对象——它的Lock方法会调用RLock，Unlock方法会调用RUnlock方法

```go
type RWMutex struct{
	w Mutex			  //互斥锁
	writerSem uint32  //信号量，写锁等待读取完成
	readerSem uint32  //信号量，读锁等待写入完成
	readerCount int32 //当前正在执行的读操作的数量
	readerWait int32  //写操作被阻塞时等待的读操作数量
}
```

读锁操作：

```
1.先通过原子操作将readerCount加1
2.如果readerCount>=0就直接返回，所以如果只有获取读取锁的操作，那么其成本只有一个原子操作
3.当readerCount<0时，说明当前有写锁，当前协程将借助信号量陷入等待状态，如果获取到信号量则直		
接退出，没有获取到信号量时的逻辑与互斥锁的逻辑相似
4.读锁解锁时，如果当前没有写锁，则其成本只有一个原子操作并直接退出
5.如果当前有写锁正在等待，则调用rUnlockSlow判断当前是否为最后一个被释放的读锁，如果是则需要
增加信号量并唤醒写锁
```

写锁操作：

```
1.写锁申请时必须先获取互斥锁，因为它复用了互斥锁的功能。接着readerCount减去
rwmutexMaxReaders阻止后续读操作
2.获取到互斥锁并不一定能直接获取写锁，如果当前已经有其它Goroutine持有互斥锁的读锁，那么当前
协程会加入全局等待队列并进入休眠状态，当最后一个读锁被释放时，会唤醒该协程
3.解锁时，调用Unlock方法，将readerCount加上rwmutexMaxReader，表示不会阻塞后序的读锁，依次
唤醒所有等待中的读锁，当所有的读锁唤醒完毕后会释放互斥锁
```

### Golang的原子操作有哪些？

Go atomic包是最轻量级的锁（也称无锁结构），可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作，不过这个包只支持int32/Int64/uint32/uint64/uintptr这几种数据类型的一些基础操作（增减、交换、载入、存储等）

概念∶

```
原子操作仅会由一个独立的CPU指令代表和完成。原子操作是无锁的，常常直接通过CPU指令直接实
现。事实上，其它同步技术的实现常常依赖于原子操作。
```

使用场景:

```
当我们想要对某个变量并发安全的修改，除了使用官方提供的 mutex，还可以使用sync/atomic包的原子	
操作，它能够保证对变量的读取或修改期间不被其他的协程所影响。
atomic 包提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作，善用atomic能够避免程
序中出现大量的锁操作。
```

常见操作：

```
增减Add
载入Load
比较并交换CompareAndSwap
交换Swap
存储Store
```

atomic操作的对象是一个地址，你需要把可寻址的变量的地址作为参数传递给方法，而不是把变量的值传递给方法

下面将分别介绍这些操作：

增减操作：
此类操作的前缀为Add

```go
func AddInt32(addr *int32，delta int32)(new int32)
func AddInt64(addr *int64, delta int64) 《new int64)
func AddUint32(addr *uint32, delta uint32)(new uint32)
func AddUint64(addr suint64,delta uint64)(new uint64)
func AddUintptr(addr *uintptr，delta uintptr) (new uintptr)
func add(addr *int64， delta int64) {
atomic.AddInt64(addr, delta)//加操作
fmt.Println("add opts: ",*addr)
}
```

载入操作：
此类操作的前缀为Load

```go
func LoadInt32( addr *int32)(val int32)
func LoadInt64(addr *int64)(val int64)
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
func LoadUint32( addr *uint32)(val uint32)
func LoadUint64(addr *uint64) (val uint64)
func LoadUintptr(addr *uintptr) (val uintptr)
//特殊类型:Value类型，常用于配置变更
func (v *Value) Load() (x interface{}){} 
```

比较并交换：

此类操作的前缀为CompareAndSwap，该操作简称CAS，可以用来实现乐观锁

```go
func CompareAndSwapInt32(addr *int32,old,new int32)(swapped bool)
func CompareAndSwapInt64(addr *int64,old,new int64) (swapped bool)
func CompareAndSwapPointer(addr *unsafe.Pointer，old,new unsafe.Pointer) (swapped bool)
func CompareAndSwapUint32(addr *uint32,old,new uint32)(swapped bool)
func CompareAndSwapUint64(addr *uint64,old,new uint64) (swapped bool)
func CompareAndSwapUintptr(addr *uintptr,old,new uintptr) (swapped bool)
```

该操作在进行交换前首先确保变量的值未被更改，即仍然保持参数old 所记录的值，满足此前提下才进行交换操作。CAS的做法类似操作数据库时常见的乐观锁机制。
需要注意的是，当有大量的goroutine对变量进行读写操作时，可能导致CAS操作无法成功，这时可以利用for循环多次尝试。

### 原子操作和锁的区别

1.原子操作由底层硬件支持，而锁是基于原子操作+信号量完成的。若实现相同的功能，前者通常会更有效率
2.原子操作是单个指令的互斥操作；互斥锁/读写锁是一种数据结构，可以完成临界区（多个指令）的互斥操作，扩大原子操作的范围
3.原子操作是无锁操作，属于乐观锁；说起锁的时候，一般属于悲观锁
4.原子操作存在于各个指令/语言层级，比如*机器指令层级的原子操作"，““汇编指令层级的原子操作”，“Go语言层级的原子操作”等。
5.锁也存在于各个指令/语言层级中，比如“机器指令层级的锁”，“汇编指令层级的锁“Go语言层级的锁“等

### Goroutine的实现原理

Goroutine可以理解为一种Go语言的协程（轻量级线程），是Go支持高并发的基础，属于用户态的线程，由Goruntime管理而不是操作系统。

底层数据结构：

```go
type g struct {
 goid int64 //唯一的goroutine的ID
 sched gobuf //goroutine切换时，用于保存g的上下文
 stack stack //栈
gopc //pc of go statement that created this goroutine
 startpc uintptr  //pc of goroutine function
 ...
}
type gobuf struct {
 sp uintptr //栈指针位置
 pc uintptr //运行到的程序位置
 g guintptr //指向goroutine
 ret uintptr //保存系统调用的返回值
 ...
}
type stack struct {
lo uintptr //栈的下界内存地址
hi uintptr //栈的上界内存地址
}
```

goroutine的状态流转：
![在这里插入图片描述](https://img-blog.csdnimg.cn/6566c8588461467cbf2836bd9b99ac7e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e257e183034442985cabdf0f7449588.png)
1.创建：go关键字会调用底层函数runtime.newproc()创建一个goroutine，调用该函数之后，goroutine会被设置成runnable状态

```
创建好的这个goroutine会新建一个自己的栈空间，同时在G的sched中维护栈地址与程序计数器这些信
息。
每个G在被创建之后，都会被优先放入到本地队列中，如果本地队列已经满了，就会被放入到全局队列
中。
```

2.运行：goroutine本身只是一个数据结构，真正让goroutine运行起来的是调度器。Go实现了一个用户态的调度器（GMP模型），这个调度器充分利用现代计算机的多核特性，同时让多个goroutine运行，同时goroutine设计的很轻量级，调度和上下文切换的代价都比较小。
![ ](https://img-blog.csdnimg.cn/ee0d002f0b77439ab0dd3407dee3644a.png)
调度时机:
1.新起一个协程和协程执行完毕
2.会阻塞的系统调用，比如文件io、网络io
3.channel、mutex等阻塞操作
4.time.sleep
5.垃圾回收之后
6.主动调用runtime.Gosched()
7.运行过久或系统调用过久等等

1.每个M开始执行P的本地队列中的G时，goroutine会被设置成running 状态。

2.如果某个M把本地队列中的G都执行完成之后，然后就会去全局队列中拿G，这里需要注意，每次去全局队列拿G的时候，都需要上锁，避免同样的任务被多次拿。
从全局队列取的G数量: N= min(len(GRQ)/GOMAXPROCS 1,len(GRQ/2)) (根据GOMAXPROCS负载均衡)

如果全局队列都被拿完了，而当前M也没有更多的G可以执行的时候，它就会去其他Р的本地队列中拿任务，这个机制被称之为work stealing机制，每次会拿走一半的任务，向下取整，比如另一个P中有3个任务，那一半就是一个任务。
从其它P本地队列窃取的G数量: N=len(LRQ)/2 (平分)

当全局队列为空，M也没办法从其他的Р中拿任务的时候，就会让自身进入自旋状态，等待有新的G进来。最多只会有GOMAXPROCS个M在自旋状态，过多M的自旋会浪费CPU 资源。

3.阻塞：channel的读写操作、等待锁、等待网络数据、系统调用等都有可能发生阻塞，会调用底层函数runtime. gopark()，会让出CPU时间片，让调度器安排其它等待的任务运行，并在下次某个时候从该位置恢复执行。
当调用该函数之后，goroutine会被设置成waiting状态。

4.唤醒：处于waiting状态的goroutine，在调用runtime.goready()函数之后会被唤醒，唤醒的goroutine会被重新放到M对应的上下文P对应的runqueue中，等待被调度。
当调用该函数之后，goroutine会被设置成runnable状态

5.退出：当goroutine执行完成后，会调用底层函数runtime.Goexit()，当调用该函数之后，goroutine会被设置成 dead 状态

### Groutine的泄露

泄露原因

```
Goroutine内进行channel/mutex等读写操作被一直阻塞。
Goroutine内的业务逻辑进入死循环，资源一直无法释放。
Goroutine内的业务逻辑进入长时间等待，有不断新增的Goroutine进入等待
```

### 怎么查看Goroutine的数量？怎么限制Goroutine的数量？

在开发过程中，如果不对goroutine加以控制而进行滥用的话，可能会导致服务整体崩溃。比如耗尽系统资源导致程序崩溃,或者CPU使用率过高导致系统忙不过来。

1.在Golang中,GOMAXPROCS中控制的是未被阻塞的所有Goroutine,可以被 Multiplex 到多少个线程上运行,通过GOMAXPROCS可以查看Goroutine的数量。
2.使用通道。每次执行的go之前向通道写入值，直到通道满的时候就阻塞了

### Goroutine和线程的区别？

1.一个线程可以有多个协程
2.线程、进程都是同步机制，而协程是异步
3.协程可以保留上一次调用时的状态，当过程重入时，相当于进入了上一次的调用状态
4.协程是需要线程来承载运行的，所以协程并不能取代线程，「线程是被分割的CPU资源，协程是组织好的代码流程」

### Go的Struct能不能比较？

1.相同struct类型的可以比较
2.不同struct类型的不可以比较,编译都不过，类型不匹配

### Go的Slice如何扩容？

go1.18之前：
1.首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
2.否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍。
3.否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的1.25倍，直到新最终容量大于等于新申请的容量
4.如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。

go1.18之后：
低于256，每次扩容两倍。超过256，不再是每次扩容1/4，而是每次增加（旧容量+3*256）/4即1.25倍+192

### 在Go函数中为什么会发生内存泄露？发生了泄漏如何检测？

Goroutine 需要维护执行用户代码的上下文信息，在运行过程中需要消耗一定的内存来保存这类信息，如果一个程序持续不断地产生新的 goroutine，且不结束已经创建的 goroutine 并复用这部分内存，就会造成内存泄漏的现象。
可以通过Go自带的工具pprof或者使用Gops去检测诊断当前在系统上运行的Go进程的占用的资源。

### Go中两个Nil可能不相等吗？

Go中两个Nil可能不相等。

接口(interface) 是对非接口值(例如指针，struct等)的封装，内部实现包含 2 个字段，类型 T 和 值 V。一个接口等于 nil，当且仅当 T 和 V 处于 unset 状态（T=nil，V is unset）。

两个接口值比较时，会先比较 T，再比较 V。接口值与非接口值比较时，会先将非接口值尝试转换为接口值，再比较

```go
func main() {
 var p *int = nil
 var i interface{} = p
 fmt.Println(i == p) // true
 fmt.Println(p == nil) // true
 fmt.Println(i == nil) // false
}
```

### Go语言中的内存对齐

CPU 并不会以一个一个字节去读取和写入内存。相反 CPU 读取内存是一块一块读取的，块的大小可以为 2、4、6、8、16 字节等大小。块大小我们称其为内存访问粒度，内存访问粒度跟机器字长有关。

对齐规则：
1.结构体的成员变量，第一个成员变量的偏移量为 0。往后的每个成员变量的对齐值必须为编译器默认对齐长度或当前成员变量类型的长度，取最小值作为当前类型的对齐值。其偏移量必须为对齐值的整数倍
2.结构体本身，对齐值必须为编译器默认对齐长度，或结构体的所有成员变量类型中的最大长度，取最大数的最小整数倍作为对齐值
3.结合以上两点，可得知若编译器默认对齐长度，超过结构体内成员变量的类型最大长度时，默认对齐长度是没有任何意义的

### interface底层原理

```go
type iface struct {
tab *itab	//存储了接口的类型
data unsafe.Pointer //存储了接口中动态类型的数据指针
}
type itab struct {
inter *interfacetype //类型的静态类型信息，比如io.Reader
_type *_type  //接口存储的动态类型
hash uint32 //接口动态类型的唯一标识，_type中hash的副本
_ [4]byte  //用于内存对齐
fun [1]uintptr //动态类型的函数指针列表
type _type struct {
    size       uintptr  // 类型大小
    ptrdata    uintptr	//偏移量
    hash       uint32   // 哈希
    tflag      tflag 	//标志
    align      uint8    // 对齐方式
    fieldAlign uint8
    kind       uint8    // 类别
    equal      func(unsafe.Pointer, unsafe.Pointer) bool
    gcdata     *byte
    str        nameOff
    ptrToThis  typeOff
}
type interfacetype struct{
	typ _type	
	pkgpath name //接口所在的包名
	mhdr []imethod  //接口中暴漏的方法在最终可执行文件中的名字和类型的偏移量
}
```

从iface或itab都可以看出，接口interface包含有两种类型：

```
一种是接口自身的类型，称为接口的静态类型，比如io.Reader等，用于确定接口类型，直接存储在itab结
构体中
一种是接口所持有数据的类型，称为接口的动态类型，用于在反射或者接口转换时确认所持有数据的实际
类型，存储在运行时runtime/_type结构体中。
```

hash是一个uint32类型的值，实际上可以看作是类型的校验码，当将接口转换成具体的类型的时候，会通过比较二者的hash值确定是否相等，只有hash相等 才能进行转换。

fun最终指向的是接口的方法集，即存储了接口所有方法的函数的指针。通过比较接口的方法集和类型的方法集，可以用来判断该类型是否实现了该接口。 把fun指向的方法集看作是一个虚函数表，也是很贴切的。

空接口：

```go
type eface struct{ // 两个指针，16byte
    _type *_type             // 指向一个内部表
    data unsafe.Pointer   // 指向所持有的数据
}
```

1.空接口是单独的唯一的一种接口类型，因此自然不需要itab中的接口类型字段了
2.空接口也没有任何的方法，因此自然也不存在itab中的方法集了

### 两个 interface 可以比较吗？

1.判断类型是否一样
reflect.TypeOf(a).Kind() == reflect.TypeOf(b).Kind()

2.判断两个interface{}是否相等
reflect.DeepEqual(a, b)

3.将一个interface{}赋值给另一个interface{}
reflect.ValueOf(&a).Elem().Set(reflect.ValueOf(b))

### go 打印时 %v %+v %#v 的区别？

%v 只输出所有的值；
%+v 先输出字段名字，再输出该字段的值；
%#v 先输出结构体名字值，再输出结构体（字段名字+字段的值）；

```go
package main
import "fmt"

type student struct {
 id   int32
 name string
}

func main() {
 a := &student{id: 1, name: "微客鸟窝"}

 fmt.Printf("a=%v \n", a) // a=&{1 微客鸟窝} 
 fmt.Printf("a=%+v \n", a) // a=&{id:1 name:微客鸟窝} 
 fmt.Printf("a=%#v \n", a) // a=&main.student{id:1, name:"微客鸟窝"}
}
```

### 什么是 rune 类型？

Go语言的字符有以下两种：

1.uint8 类型，或者叫 byte 型，代表了 ASCII 码的一个字符。
2.rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。rune 类型等价于 int32 类型。

### 空 struct{} 占用空间么？用途是什么？

空结构体 struct{} 实例不占据任何的内存空间。

用途：
1.将 map 作为集合(Set)使用时，可以将值类型定义为空结构体，仅作为占位符使用即可。
2.不发送数据的信道(channel)
使用 channel 不需要发送任何的数据，只用来通知子协程(goroutine)执行任务，或只用来控制协程并发度。
3.结构体只包含方法，不包含任何的字段

### golang值接收者和指针接收者的区别

golang函数与方法的区别是，方法有一个接收者。

如果方法的接收者是指针类型，无论调用者是对象还是对象指针，修改的都是对象本身，会影响调用者

如果方法的接收者是值类型，无论调用者是对象还是对象指针，修改的都是对象的副本，不影响调用者

```go
package main

import "fmt"

type Person struct {
	age int
}

func (p *Person) IncrAge1() {
	p.age += 1
}

func (p Person) IncrAge2() {
	p.age += 1
}

func (p Person) GetAge() int {
	return p.age
}

func main() {
	p := Person{
		22,
	}
	p.IncrAge1()
	fmt.Println(p.GetAge()) //23
	p.IncrAge2()
	fmt.Println(p.GetAge()) //23
	p2 := &Person{
		age: 22,
	}
	p2.IncrAge1()
	fmt.Println(p2.GetAge()) //23
	p2.IncrAge2()
	fmt.Println(p2.GetAge()) //23
}
```

通常我们使用指针类型作为方法的接收者的理由：

```
1.使用指针类型能够修改调用者的值
2.使用指针类型可以避免在每次调用方法时复制该值，在值的类型为大型结构体时，这样做更加高效
```

### 引用传递和值传递

什么是引用传递?
将实参的地址传递给形参，函数内对形参值内容的修改，将会影响实参的值内容。Go语言是没有引用传递的，在C++中，函数参数的传递方式有引用传递。
Go的值类型(int、struct等）、引用类型（指针、slice、map、 channel)

int类型：形参和实际参数内存地址不一样，证明是值传递﹔参数是值类型，所以函数内对形参的修改，不会修改原内容数据

指针类型：形参和实际参数内存地址不一样，证明是值传递，由于形参和实参是指针，指向同一个变量。函数内对指针指向变量的修改，会修改原内容数据

```go
func add(x *int) {
	fmt.Printf("函数里指针地址：%p\n", &x)
	fmt.Printf("函数里指针指向变量的地址：%p\n", x)
	*x += 2
}

func main() {
	a := 2
	p := &a
	fmt.Printf("原始指针地址：%p\n", &p)
	fmt.Printf("原始指针指向变量的地址：%p\n", p)
	add(p)
	fmt.Println(a)
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1aa1c5ed0d2142e39dbb140616bf5998.png)
这说明函数调用时，传递的依然是指针的副本，依然是值传递，但指针指向的是同一个地址，所以能修改值

slice：slice 是一个结构体，他的第一个元素是一个指针类型，这个指针指向的是底层数组的第一个元素。当参数是slice类型的时候，frmt printf通过%p打印的slice变量的地址其实就是内部存储数组元素的地址，所以打印出来形参和实参内存地址一样。

```go
func add(x []int) {
	fmt.Printf("函数里指针地址:%p\n", &x)
	fmt.Printf("函数里指针指向变量的地址:%p\n", x)
}

func main() {
	a := []int{1, 2, 3}
	fmt.Printf("原始指针地址:%p\n", &a)
	fmt.Printf("原始指针指向变量的地址:%p\n", a)
	add(a)
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea321b9455aa45769e7fbd4f3d82f5bf.png)
map和channel也是同理的

### defer关键字的实现原理

Go14中编译器会将deler函数直接插入到函数的尾部，无需链表和栈上参数拷贝，性能大幅提升。把deler函数在当前函数内展开并直接调用，这种方式被称为open code ddefer

1.函数退出前，按照先进后出的顺序，执行defer函数
2.panic后的defer不会被执行（遇到panic，如果没有捕获错误，函数会立刻终止）
3.panic没有被recover时，抛出的panic到当前goroutine最上层函数时，最上层程序直接异常终止

```go
package main

import "fmt"

func F() {
	defer func() {
		fmt.Println("b")
	}()
	panic("a")
}

func main() {
	defer func() {
		fmt.Println("c")
	}()
	//子函数抛出的panic没有recover时，上层函数时，程序直接异常终止
	F()
	fmt.Println("继续执行")
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5167fc7458a4f3a8762ccdce068d4cc.png)
4.panic有被recover时，当前goroutine最上层函数正常执行

```go
package main

import "fmt"

func F() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("捕获异常", err)
		}
		fmt.Println("b")
	}()
	panic("a")
}

func main() {
	defer func() {
		fmt.Println("c")
	}()
	//子函数抛出的panic没有recover时，上层函数时，程序直接异常终止
	F()
	fmt.Println("继续执行")
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/62dc3d8557e441b292978316c7110329.png)
panic发生后，会自动依次往上层去执行defer函数，并且panic之后的函数不会被执行，即使是被recover

大概的流程就是，如果遇见panic关键字的话，那么go执行器就会进入代码gopanic函数中，进入之后会拿到表示当前协程g的指针，然后通过该指针拿到当前协程的defer链表，通过for循环来进行执行defer，如果在defer中又遇见了panic的话，则会释放这个defer，通过continue去执行下一个defer，然后就是一个一个的执行defer了，如果在defer中遇见recover，那么将会通过mcall(recovery)去执行panic.

### Select底层原理

select的每一个case在运行时都是一个scase结构体，存放了通道和通道中的元素类型等信息。

```go
type scase struct{
	c *hchan
	elem unsafe.Pointer
	kind uint16
	...
}
```

scase结构体中的kind代表scase的类型。scase一共有4种具体的类型，分别为caseNil、caseRecv、caseSend及caseDefault。caseNil代表当前分支中的通道为nil，caseRecv代表当前分支从通道接收消息，caseSend代表当前分支发送消息到通道，caseDefault代表当前分支为default分支。pollorder通过引入随机数的方式给序列带来了随机性。

select运行时会调用核心函数selectgo，selectgo有两个关键的序列，分别是pollorder和lockorder。
pollorder代表乱序后的scase序列，pollorder通过引入随机数的方式给序列带来了随机性。
lockorder是按照大小对通道地址排序，对所有的scase按照其通道在堆区的地址大小，使用大根堆排序算法进行排序。selectgo会按照该序列依次对select中所有的通道加锁。而按照地址排序的目的是避免多个协程并发加锁时带来的死锁问题。

当对所有的scase中的通道加锁完毕后，selectgo开始了一轮对于所有scase的循环。循环的目的是找到当前准备好的通道。
1.如果scase的类型为caseNil，则会被忽略。
2.如果scase的类型为caseRecv，则和普通的通道接收一样，先判断是否有正在等待写入当前通道的协程，如果有则直接跳转到对应的recv分支。接着判断缓冲区是否有元素，如果有则直接跳转到bufrecv分支执行。
3.如果scase的类型为caseSend，则和普通的通道发送一样，先判断是否有正在等待读取当前通道的协程，如果有，则跳转到send分支执行，接着判断缓冲区是否空余，如果有，则跳转到bufsend分支执行。
4.如果scase的类型为caseDefault，则会记录下来，并在循环完毕发现没有已经准备好的通道后，判断是否存在caseDefault类型，如果有，则跳转到retc分支执行。
5.当任何一个case都不满足时，当前协程进入休眠状态，不管是读取通道还是写入通道都需要创建一个新的sudog并放入指定通道的等待队列，之后当前协程将进入休眠状态。当select case中的任意一个通道不再阻塞时，当前协程将被唤醒。要注意的是，最后需要将sudog结构体在其他通道的等待队列中出栈，因为当前协程已经能够正常运行，不再需要被其他通道唤醒。

### gRPC

gRPC是基于go的远程过程调用。RPC 框架的目标就是让远程服务调用更加简单、透明，RPC 框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节。服务调用者可以像调用本地接口一样调用远程的服务提供者，而不需要关心底层通信细节和调用过程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2084bf5b95b41b8812c3715c9f3aaa9.png)

### 服务发现是怎么做的？

主要有两种服务发现机制：客户端发现和服务端发现

客户端发现：是指客户端应用程序主动扫描、获取和管理可用的服务实例列表。客户端通过向服务注册中心发送请求来获取服务实例列表，然后根据负载均衡算法选择其中一台实例进行服务调用。客户端负责维护可用实例列表，包括实例的添加、删除和更新等操作。客户端发现需要在客户端应用程序中集成相应的服务发现客户端，例如Netflix的Eureka客户端。
![在这里插入图片描述](https://img-blog.csdnimg.cn/316a344634ae4a8aa3d05a853a57df5e.png)
服务端发现：是指服务注册中心负责主动维护和管理可用的服务实例列表。服务实例在启动时向服务注册中心进行注册，注册中心负责记录和维护服务实例的信息，包括实例的网络地址、健康状态、负载情况等。当客户端需要调用服务时，向服务注册中心发送请求，注册中心根据负载均衡算法选择合适的实例返回给客户端。服务端发现通常需要使用专门的服务注册中心，例如Consul、ZooKeeper等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/164cf773f91a42c3962e8cdd88ceaae9.png)
区别：

```
1.负载均衡：客户端发现将负载均衡逻辑放在客户端，客户端根据自己的负载均衡策略选择服务实例；服	
务端发现将负载均衡逻辑放在服务注册中心，客户端只需向注册中心请求可用实例。
2.可用性：客户端发现存在单点故障风险，如果注册中心宕机，客户端无法获取服务实例列表；服务端发
现通过多个注册中心节点实现高可用性，即使其中一个节点宕机，仍可保证服务可用性。
3.数据传输：客户端发现需要在每次调用服务时与注册中心交互，增加了网络传输开销；服务端发现将服	
务实例列表缓存在注册中心，客户端只需要定期或根据需要从注册中心获取实例列表，减少了网络传输开
销。
4.系统依赖性：客户端发现对服务端没有依赖，只需和注册中心交互；服务端发现服务实例和注册中心存	
在依赖关系。 选择使用客户端发现还是服务端发现，取决于具体的系统需求和设计考虑。
```

### 设计模式

设计模式分为三类：
1.创建型模式：用于控制对象的创建过程
2.结构型模式：用于处理类和对象之间的关系
3.行为型模式：用于描述对象之间的通信和协作

1.工厂模式（Factory Pattern）是一种创建型模式，它提供了一种统一的接口来创建对象，但是具体的对象创建过程则由子类来实现。工厂模式可以将对象的创建和使用解耦，使得代码更加灵活和可扩展。常见的工厂模式有简单工厂模式、工厂方法模式和抽象工厂模式。

2.代理模式（Proxy Pattern）是一种结构型模式，它为其他对象提供一种代理，以控制对这个对象的访问。代理模式可以在不改变原始对象的情况下，增加一些额外的功能或限制对原始对象的访问。常见的代理模式有静态代理和动态代理。

3.建造者模式（Builder Pattern）是一种创建型模式，它将一个复杂对象的创建过程分解成多个简单的步骤，从而可以灵活地组合这些步骤来构建不同的对象。建造者模式可以避免使用多个参数构造函数的情况，使得代码更加清晰和可读。同时，建造者模式还可以通过链式调用来简化对象的构建过程。

### 反射

反射是指计算机程序可以访问、检测和修改它本身状态或行为的一种能力。程序在编译时，变量被转换为内存地址，变量名不会被编译器写入到可执行部分。在运行程序时，程序无法获取自身的信息。
支持反射的语言可以在程序编译期将变量的反射信息，如字段名称、类型信息、结构体信息等整合到可执行文件中，并给程序提供接口访问反射信息，这样就可以在程序运行期间获取类型的反射信息，并且有能力修改它们。

反射的两个弊端：

```
1.代码不易阅读，不易维护，容易发生线上panic
2.性能很差，比正常代码慢一到两个数量级
```

golang当中反射最重要的两个概念是Type和Value，Type用于获取类型相关的信息（比如Slice的长度，struct的成员，函数的参数个数），Value用于获取和修改原始数据的值（比如修改slice和map中的元素，修改struct的成员变量）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c59a33644d694e0aac22d56aa7ee868e.png)

### Golang字符串拼接对比

+号拼接：因为golang的字符串是静态的，所以每次+都会重新分配一个内存空间存相加的两个字符串
fmt.Sprintf：主要是使用到了反射
Strings.Builder：

```go
type Builder struct {
    addr *Builder // of receiver, to detect copies by value
    buf  []byte // 1
}
```

addr字段主要是做copycheck，buf字段是一个byte类型的切片，这个就是用来存放字符串内容的，提供的writeString()方法就是像切片buf中追加数据。

```go
func (b *Builder) WriteString(s string) (int, error) {
 b.copyCheck()
 b.buf = append(b.buf, s...)
 return len(s), nil
}
```

[]byte的申请是成倍的，例如，初始大小为 0，当第一次写入大小为 10 byte 的字符串时，则会申请大小为 16 byte 的内存（恰好大于 10 byte 的 2 的指数），第二次写入 10 byte 时，内存不够，则申请 32 byte 的内存，第三次写入内存足够，则不申请新的，以此类推。
提供的String方法就是将[]byte转换为string类型，这里为了避免内存拷贝的问题，使用了强制转换来避免内存拷贝。

```go
func (b *Builder) String() string {
 return *(*string)(unsafe.Pointer(&b.buf))
}
```

bytes.Buffer：strings.Builder 和 bytes.Buffer 底层都是 []byte 数组，但 strings.Builder 性能比 bytes.Buffer 略快约 10% 。一个比较重要的区别在于，bytes.Buffer 转化为字符串时重新申请了一块空间，存放生成的字符串变量，而 strings.Builder 直接将底层的 []byte 转换成了字符串类型返回了回来。

### 常见字符集

ASCII：采用了一个字节存储一个字符，首位是0，总共能能表示128个字符（英文、符号等）
UTF-8：是Unicode字符集的一种编码方案，采取可变长编码方案，总共分为四个长度区：1个字节、2个字节、3个字节、4个字节。英文字符、数字等只占1个字节（兼容标准ASCII编码），汉字字符占用3个字节。
![在这里插入图片描述](https://img-blog.csdnimg.cn/659a38dd22d9455783f156ac62ad2672.png)
UTF-16：编码时以2个字节为基本单位
UTF-32：4个字节表示一个字符，定长编码

### String和[]byte的区别

string类型本质也是一个结构体，定义如下：

```go
type stringStruct struct {
    str unsafe.Pointer
    len int
}
```

string类型底层是一个byte类型的数组，stringStruct和slice还是很相似的，str指针指向的是byte数组的首地址，len代表的就是数组长度。

string和byte的区别：

string类型为什么还要在数组的基础上再进行一次封装呢？
这是因为在Go语言中string类型被设计为不可变的，不仅是在Go语言，其他语言中string类型也是被设计为不可变的，这样的好处就是：在并发场景下，我们可以在不加锁的控制下，多次使用同一字符串，在保证高效共享的情况下而不用担心安全问题。

string类型虽然是不能更改的，但是可以被替换，因为stringStruct中的str指针是可以改变的，只是指针指向的内容是不可以改变的。

### HTTP和RPC对比

RPC（Remote Produce Call）：远程过程调用，
HTTP：网络传输协议

相同点：

```
1.都是基于TCP协议的应用层协议
2.都可以实现远程调用，服务调用服务
```

不同点：

```
1.RPC主要用于在不同的进程或计算机之间进行函数调用和数据交换。HTTP主要用于数据传输和通信。
2.RPC协议通常采用二进制协议和高效的序列化方式，而HTTP通常采用文本协议和基于ASCII码的编码
方式，数据传输效率较低
3.RPC通常需要使用专门的IDL文件来定义服务和消息类型，生成服务端和客户端的代码。而HTTP没有	
这个限制，可以使用套接字进行通信
```

### gRPC和RPC对比

gRPC是一种高性能，通用的远程过程调用（RPC）框架，采用基于HTTP/2的二进制传输协议实现可以实现双向流、头部压缩和多路复用等，使用Protocol Buffers作为默认的序列化协议

GRPC和RPC区别：

```
1.通信协议不同：gRPC是基于HTTP/2协议进行数据传输，而传统的RPC框架通常使用TCP和UDP等传
输层协议
2.序列化方式不同：gRPC使用Protocol Buffers作为默认的序列化协议，而传统的RPC框架使用JSON、
XML等格式。
3.支持多种语言：gRPC支持多种编程语言，包括C++、Java、Python、Go、Ruby等，而传统的RPC框
架通常只支持少数几种语言
4.高性能：由于采用了HTTP/2协议和Protocol Buffers序列化协议，gRPC具有更高的性能和效率
5.自动生成代码：gRPC可以根据服务定义文件自动生成客户端和服务端的代码，大大简化了开发过程。
6.安全性：gRPC提供了TLS加密和认证等安全机制，保障通信的安全性。
```

### Sync.Pool的使用

sync.Pool 本质用途是增加临时对象的重用率，减少 GC 负担；

sync.Pool 中保存的元素有如下特征：

```
1.Pool 池里的元素随时可能释放掉，释放策略完全由 runtime 内部管理；
2.Get 获取到的元素对象可能是刚创建的，也可能是之前创建好 cache 的，使用者无法区分；
3.Pool 池里面的元素个数你无法知道；
```

sync.Pool的数据结构：

```go
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() any
}
```

申请对象Get
释放对象Put

初始化Pool实例：

```go
package main

import (
        "fmt"
        "sync"
        "sync/atomic"
        "time"
)

var createNum int32

func createBuffer() interface{} {
        atomic.AddInt32(&createNum, 1)
        buffer := make([]byte, 1024)
        return buffer
}

func main() {
        bufferPool := &sync.Pool{New: createBuffer,}

        workerPool := 1024 * 1024
        var wg sync.WaitGroup
        wg.Add(workerPool)

        for i := 0; i < workerPool; i++ {
                go func(){
                        defer wg.Done()
                        buffer := bufferPool.Get()
                        _ = buffer.([]byte)
                        defer bufferPool.Put(buffer)
                        //buffer := createBuffer()
                        //_ = buffer.([]byte)
                }()
        }
        wg.Wait()
        fmt.Printf(" %d buffer objects were create.\n",createNum)
        time.Sleep(5 * time.Second)
}
```

### JWT

jwt结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/89c04873d6814dbba95c5081dd43ac67.png)
jwt认证授权过程：

```
1.客户端发送账号密码等一些登陆信息给服务端
2.服务端验证通过之后，通过上述jwt的字段信息，返回给客户端一个token
3.之后的客户端每次请求都把这个token放到头部
4.服务端需要授权的接口会先拿到token进行解析来检验token并且获取用户的信息
```

### Gin框架

[参考网站](https://zhuanlan.zhihu.com/p/611116090)

选择gin框架的原因：

```
1.支持中间件操作（handlersChain机制）
2.更方便的使用（gin.Context）
3.更强大的路由解析能力（radix tree路由树）
```

gin是Golang HTTP标准库net/http基础之上的再封装

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/27ee40203c234281ae0b2a86bedfcf3a.png)
初始化Engine：

```
1.创建一个gin.Engine实例
2.创建Enging的首个RouterGroup，对应的处理函数链Handlers为nil，基础路径basePath为“/”，root标识为true
3.构造了9棵方法路由数，对应9种http方法
4.创建gin.Context的对象池
```

注册middleware：

```
注册的middleware会添加到RouterGroup.Handlers中，后续 RouterGroup 下新注册的 handler 都会在前缀中拼上这
部分 group 公共的 handlers.
```

注册handler：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0a0bc1f1fa0049d0b9f2955aaca1abc6.png)

1.完整路径拼接：根据RouterGroup中的basePath和注册时传入的relativePath，组成absolutePath
2.完整handlers生成：深拷贝RouterGroup中handlers和注册传入的handlers，生成新的handlers数组并返回
3.注册handler到路由树：获取http method对应的methodTree，将absolutePath和对应的handlers注册到methodTree中

启动服务：

```
一键启动 Engine.Run 方法后，底层会将 gin.Engine 本身作为 net/http 包下 Handler interface 的实现类，并调用 
http.ListenAndServe 方法启动服务。，ListenerAndServe 方法本身会基于主动轮询 + IO 多路复用的方式运行，因此
程序在正常运行时，会始终阻塞于 Engine.Run 方法，不会返回.
```

处理请求：

```
在服务端接收到 http 请求时，会通过 Handler.ServeHTTP 方法进行处理. 而此处的 Handler 正是 gin.Engine
1.对于每次http请求，会为其分配一个gin.Context，在handlers链路中持续往下传递
2.调用Engine.handleHTTPRequest方法，从路由树中获取handlers链，然后遍历调用
3.处理完http请求后，会将gin.Context进行回收，整个回收复用的流程基于对象池管理
```

gin的路由树（压缩前缀树radix tree）

```
压缩前缀树相比与前缀树的优化在于：若某个子节点是其父节点的唯一孩子，则与父节点进行合并
```

补偿策略：

```
在组装路由树时，会将注册路由句柄数量更多的child node摆放在children数组更靠前的位置，这是因为某个链路注册
的 handlers 句柄数量越多，一次匹配操作所需要花费的时间就越长，被匹配命中的概率就越大，因此应该被优先处
理.
```

gin.Context

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/6ed44aafde4a44df8e16a43a66a6af96.png)
gin.Context 的定位是对应于一次 http 请求，贯穿于整条 handlersChain 调用链路的上下文，其中包含了如下核心字段：

```
Request/Writer：http 请求和响应的 reader、writer 入口
handlers：本次 http 请求对应的处理函数链
index：当前的处理进度，即处理链路处于函数链的索引位置
engine：Engine 的指针
mu：用于保护 map 的读写互斥锁
Keys：缓存 handlers 链上共享数据的 map
```

gin.Context 作为处理 http 请求的通用数据结构，不可避免地会被频繁创建和销毁. 为了缓解 GC 压力，gin 中采用对象池 sync.Pool 进行 Context 的缓存复用，处理流程如下，只有在新请求来获取到context时，才会将其变为空白

### 协程池

协程已经很轻量了，为什么还要有协程池？

```
1.限制协程的数量，不让协程无限制地增长
2.减少GC和协程创建的开销
```

### math/rand

math/rand是协程安全的

为什么 math/rand 需要加锁呢?

大家都知道 math/rand 是伪随机的, 但是在设置完 seed 后, rng.vec 数组的值基本上就确定下来了, 这明显就不是随机了, 为了增加随机性, 通过 Uint64() 获取到随机数后, 还会重新去设置 rng.vec. 由于存在并发获取随机数的需求, 也就有了并发设置 rng.vec 的值, 所以需要对 rng.vec 加锁保护.

随机数的种子设置有什么注意的地方？

随机数种子设置的一样，则每次获取的随机值也是一样的。

随机种子计算随机数的计算方法：

```
算法1：平方取中法。
算法2：线性同余法
```

### golang中指针的作用

1.传递大对象
2.修改函数外部变量
3.动态分配内存
4.函数返回指针