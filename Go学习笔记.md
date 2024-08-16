# 序言

内存管理主要包含两个动作：分配和释放。逃逸分析就是服务于内存分配的，而内存的释放有GC负责

## 逃逸分析

> 基本数据类型一般来说分配在栈区，编译器存在一个逃逸分析
>
> 引用数据类型一般分配到堆区

用来标识变量内存应该被分配到栈区还是堆区，分配时遵守一下两种规则

1. 指向栈上对象的指针不能被存储到堆中 —— 堆中的变量通常生命周期更长
2. 指向栈上对象的指针不能超过该栈对象的生命周期

## init 函数

每个源文件都可以有一个 init 函数，执行顺序，先执行变量定义的函数，最后执行main函数

先执行引入文件的 init 函数在执行当前文件的 init 函数

## 指针

函数传递参数时，大对象是否要传递指针，要进行分析，因为在内存分配机制中，这个大对象的内存会被分配到堆上，增加系统开销

## 类型转换

~~~go
目标类型(V)
~~~

注意：

1. 数据类型可以大转小，也可以小转大
2. 被转换的是变量存储的数据(值)，变量本身的数据类型并没有发生变化
3. 如果将高精度转换为低精度，编译时不会报错，只是转换的结果是按溢出处理
4. 数据类型转换必须显示转换

### 基本数据类型转 string

~~~go
//方式一
fmt.Sprintf(format_String,interface{})  //返回string

//方式二
使用strconv包的函数
~~~



## 错误处理机制

错误处理方式：defer、panic、recover

可以抛出一个 panic 的异常，然后在 defer 中通过 recover 捕获这个异常，然后正常处理

~~~go
defer fun (){
    err := recover()	//recover()内置函数，可以捕获异常
    if err != nil{		//说明捕获到错误
        fmt.Println("err=",err)
        //错误处理
    }
}()
~~~

### 自定义错误

~~~go
//使用 errors.New 和 panic 内置函数
errors.New("错误说明") //返回一个error类型的值，表示一个错误
panic //内置函数
~~~

## goroutine

### 协程和主线程

**go协程的特点**

1. 有独立的栈空间
2. 共享程序堆空间
3. 调度由用户控制
4. 协程是轻量级的线程

### MPG模式

- M：操作系统的主线程（物理线程）
- P：协程执行需要的上下文
- G：协程

### 设置运行 cpu 数目

> 1.8之前需要设置
>
> 1.8之后不需要设置，默认运行在多核上

~~~Go
func main(){
    cpuNum := runtime.NumCPU()
    fmt.Println("cpuNum",cpuNum)
    
    runtime.GOMAXPROCS(cpuNum - 1)
    fmt.Println("ok")
}
~~~

### 调度器的设计策略

1. 复用线程

   1. work stealing 机制：将线程a上等待的协程换到空闲的线程上面执行
   2. hand off 机制：当线程a有多个协程，并当前运行的协程被阻塞或等待，则新建一个线程，将线程a上面的协程放到新建的线程上，然后将线程a睡眠，等到阻塞的协程运行完毕，根据情况将这个协程销毁或放在其他队列，并将所在的线程销毁

2. 利用并行

   通过 GOMAXPROCS 限制 P 的个数，

3. 抢占

   每个 G 最多可以使用 CPU 10ms 如果没执行完毕，则被新的 G 抢占

4. 全局 G 队列

   对 work stealing 机制的补充，当所有 P 的本地队列都为空时，从全局队列拿取（当 G 被阻塞或睡眠时放入全局队列）

## 管道channel

管道一定不能被读取者close()

管道不能只有写或只有读，必须两者都有，可以频率不一致，否则会触发deadlock

一个数据结构 - 队列
先进先出的
线程安全，多goroutine访问时，不需要加锁，不会发生资源竞争问题
有类型，一个string的channel只能放string类型数据

~~~go
//定义/声明
//可以声明为只读或只写，在 chan 前面或后面加上 <-
var 变量名 chan 数据类型

//初始化
变量名 = make(chan 数据类型，长度)

//向管道写入数据
变量名<- value

//取值
value<- 变量名

//注意
空接口类型，在读取的时候要进行类型断言
~~~

说明：

1. channel 是引用类型
2. channel必须初始化才能写入数据，即make后才能使用
3. 管道是有类型的，intChan只能写入整数 int
4. 数据放满就不能再放，数据取完就不能再取

### 遍历和关闭

#### 关闭

当channel关闭后，不能再写入数据，只能读数据

~~~go
//使用内置函数close可以关闭channel
close(channelName)
~~~

#### 遍历

使用for-range遍历

1. 再遍历时，如果channel没有关闭，会出现deadlock的错误
2. 再遍历时，如果channel已经关闭，则会正常遍历

~~~go
for v:=range channel{
    //循环体
}
~~~

go run  : 编译并运行

go build ：编译程序，并生成一个可运行的exe文件

**buindin 内置函数**



## 单元测试



# 正文

**变量声明注意事项**

变量使用` := `声明只能在函数体内使用

多变量不同类型声明方式

~~~go
var(
	k int
    t 
)
~~~

**函数的多返回值**

返回值是匿名的，返回值跟在 return 后面

返回值有名称的，在函数最后赋值，只写 return 就好

**导包**

- `.`：表示引入这个包的所有变量和方法，在使用的时候可以直接使用这个包里面的方法和变量，类似将这个包里面的代码插入到当前包
- `_`：只调用这个包的 init 方法
- 别名：在前面直接加上别名，默认为是包名

**切片细节**

对切片再进行切片，两个切片会指向同一个内存地址，

使用copy操作切片，会创建新的切片，

**append操作原理**

先创建一个新的数组，将旧的数组内容 copy 过去，然后将新添加的内容追加过去，再将旧的数组删除，最后将指针指向新的数组

## 泛型

在1.18版本加入了对泛型的支持，泛型是为了解决执行逻辑与类型无关的问题，

### 泛型方法

~~~go
func Sum[T int | float64](a, b T) T {
   return a + b
}
~~~

> **类型形参**：T就是一个类型形参，形参具体是什么类型取决于传进来什么类型
>
> **类型约束**：`int | float64`构成了一个类型约束，这个类型约束内规定了哪些类型是允许的，约束了类型形参的类型范围
>
> **类型实参**：`Sum[int](1,2)`，手动指定了`int`类型，`int`就是类型实参。

### 泛型结构

~~~go
//切片
type GenericSlice[T int | int32 | int64] []T
GenericSlice[int]{1, 2, 3} //使用时就不能省略掉类型实参
~~~





## 反射reflect

Valueof 和 Typeof

~~~go
str := "hello world!"
//获取变量的完整类型信息_会指出是那个包里面的某个类型
reflectType := reflect.TypeOf(str)
//获取变量的基本类型信息
reflectType := reflect.TypeOf(str).Kind()
//获取数据结构所存储的元素类型，必须是指针，切片，数组，通道，映射表其中之一
reflectType := reflect.TypeOf(str).Elem()
//获取对应类型所占的字节大小
reflect.TypeOf(0).Size()

//获取值
reflectValue := reflect.ValueOf(str)
//获取反射值原有的值
func (v Value) Interface() (i any)
//设置值
func (v Value) Set(x Value)


~~~

### 函数反射和调用

~~~go
func Max(a, b int) int {
   if a > b {
      return a
   }
   return b
}

func main() {
   rType := reflect.TypeOf(Max)
   // 输出函数名称,字面量函数的类型没有名称
   fmt.Println(rType.Name())
   // 输出参数，返回值的数量
   fmt.Println(rType.NumIn(), rType.NumOut())
   rParamType := rType.In(0)
   // 输出第一个参数的类型
   fmt.Println(rParamType.Kind())
   rResType := rType.Out(0)
   // 输出第一个返回值的类型
   fmt.Println(rResType.Kind())
    
    
     // 获取函数的反射值
   rType := reflect.ValueOf(Max)
   // 传入参数数组
   rResValue := rType.Call([]reflect.Value{reflect.ValueOf(18), reflect.ValueOf(50)})
   for _, value := range rResValue {
      fmt.Println(value.Interface())
   }
}
~~~





## 文件操作

go1.16之后 ioutil 包被迁移到了 io 和 os 包中

Go文件操作的基础数据类型支持是 []byte

### 打开

常见的两种打开文件的方式是使用`os`包提供的两个函数，`Open`函数返回值一个文件指针和一个错误，

```go
func Open(文件名 string) (*File, error)
```

后者`OpenFile`能够提供更加细粒度的控制，实际上`Open`函数就是对`OpenFile`函数的一个简单封装。

```go
func OpenFile(name string, flag int, perm FileMode) (*File, error)
```

先来介绍第一种使用方法，直接提供对应的文件名即可，代码如下



# 网络编程

**端口分类**

- 0：保留端口号

- 1-1024：固定端口

  > 22：SSH远程登录协议
  > 23：telnet使用
  > 24：ftp使用
  > 25：smtp服务使用
  > 80：iis使用
  > 7：echo使用

## TCP编程

**TCP协议： **传输控制协议/网际协议，是一种面向连接（连接导向）的、可靠的、基于字节流的传输层通信协议。一个TCP服务端可以同时连接很多个客户端

TCP服务端程序的处理流程：

> 1. 监听端口
> 2. 接收客户端请求建立链接
> 3. 创建goroutine处理链接。 

~~~go
//设置协议和监听端口
listen, err := net.Listen("tcp", "127.0.0.1:20000")

//建立链接
conn, err := listen.Accept()

//发送数据
conn.Write([]byte(str))

//关闭链接
conn.Close()

~~~

TCP客户端进行TCP通信的流程

> 1. 建立与服务端的链接
> 2. 进行数据收发
> 3. 关闭链接

~~~go
//创建链接  返回一个Conn接口对象  对
conn, err := net.Dial("tcp", "127.0.0.1:20000")
~~~

## UDP编程

**UDP协议：**用户数据报协议，一种无连接的传输层协议，不需要建立连接就能直接进行数据发送和接收，属于不可靠的、没有时序的通信，但是UDP协议的实时性比较好，通常用于视频直播相关领域。

UDP服务端

~~~go
//创建链接
listen, err := net.ListenUDP("udp", &net.UDPAddr{
	IP:   net.IPv4(0, 0, 0, 0),
	Port: 30000,
})
~~~

`"udp"` ：指定网络类型，可以是`"udp"`, `"udp4"`, 或 `"udp6"`，

`net.UDPAddr`结构体包含以下字段：

- **`IP`**: 监听的本地IP地址。在这里，`net.IPv4(0, 0, 0, 0)`表示`0.0.0.0`，即监听所有可用的网络接口（可以接收来自任何本地IP地址的数据）。
- **`Port`**: 监听的端口号，这里指定为`30000`。
- **`Zone`**: 适用于IPv6的作用域标识符，这里没有使用，可以忽略。

~~~go
//接收数据
n, addr, err := listen.ReadFromUDP(data[:])
// 发送数据
_, err = listen.WriteToUDP(data[:n], addr) 
//关闭链接
listen.Close()
~~~

UDP客户端

~~~go
//创建一个UDP链接
socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
	IP:   net.IPv4(0, 0, 0, 0),
	Port: 30000,
})
~~~

**`net.DialUDP("udp", nil, &net.UDPAddr{...})`**:

- `net.DialUDP`是`net`包中的一个函数，用于创建并返回一个UDP连接（`*net.UDPConn`）。
- 第一个参数`"udp"`指定了网络类型，可以是`"udp"`, `"udp4"`, 或 `"udp6"`，这里指定的是`"udp"`，表示使用UDP协议。

**`nil`**:

- 第二个参数是本地地址（`laddr`），可以指定本地IP地址和端口。如果传递`nil`，表示让系统自动选择一个本地地址和端口。

**`&net.UDPAddr{...}`**:

- 第三个参数是远程地址（`raddr`），是一个指向`net.UDPAddr`结构体的指针，用于指定远程服务器的IP地址和端口。

- `net.UDPAddr`
  
  结构体包含三个字段：

  - `IP`: 远程服务器的IP地址。在这里使用`net.IPv4(0, 0, 0, 0)`表示任意IP地址，即所有可用的网络接口。
  - `Port`: 远程服务器的端口号，这里指定为`30000`。
  - `Zone`: 用于IPv6地址的作用域标识符，这里没有使用，可以忽略。

~~~go
//发送数据
_, err = socket.Write(sendData)
//接受数据
n, remoteAddr, err := socket.ReadFromUDP(data)
~~~

## TCP粘包

客户端不间断分10次发送的数据，在服务端并没有成功的输出10次，而是多条数据“粘”到了一起。

## 为什么出现粘包

主要原因就是tcp数据传递模式是流模式，在保持长连接的时候可以进行多次的收和发。“粘包”可发生在发送端也可发生在接收端：

> 1. 由Nagle算法造成的发送端的粘包：Nagle算法是一种改善网络传输效率的算法。简单来说就是当我们提交一段数据给TCP发送时，TCP并不立刻发送此段数据，而是等待一小段时间看看在等待期间是否还有要发送的数据，若有则会一次把这两段数据发送出去。
> 2. 接收端接收不及时造成的接收端粘包：TCP会把接收到的数据存在自己的缓冲区中，然后通知应用层取数据。当应用层由于某些原因不能及时的把TCP的数据取出来，就会造成TCP缓冲区中存放了几段数据。 

## 解决办法

可以自己定义一个协议，比如数据包的前4个字节为包头，里面存储的是发送的数据的长度。



# 常用标准库

## fmt

### Print  打印输出

~~~go
func Print(a ...interface{}) (n int, err error)	//直接输出
func Printf(format string, a ...interface{}) (n int, err error)//格式化输出
func Println(a ...interface{}) (n int, err error)//输出一行
~~~

### Fprint  向 io.Writer 中输出

~~~go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error) 
~~~

### Sprint  返回字符串

~~~go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
~~~

### Errorf  返回包含格式化字符串的错误

```go
func Errorf(format string, a ...interface{}) error 
```

### Scan  将输入数据读入到参数列表中，返回成功扫描的数据个数和遇到的任何错误。

~~~go
func Scan(a ...interface{}) (n int, err error)  //用空格分开
func Scanf(format string, a ...interface{}) (n int, err error)//格式化读入
func Scanln(a ...interface{}) (n int, err error) //遇到换行时才停止扫描
~~~

### Fscan  从io.Reader中读取数据。

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

### Sscan  从指定字符串中读取数据

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

## time

### 时间操作

~~~go
func (t Time) Add(d Duration) Time //时间+时间间隔
func (t Time) Sub(u Time) Duration //求两个时间之间的差值
func (t Time) Equal(u Time) bool   //判断两个时间是否相同，会考虑时区的影响
func (t Time) Before(u Time) bool  //t代表的时间点在u之前，返回真
func (t Time) After(u Time) bool   //t代表的时间点在u之后，返回真
~~~

### 定时器

~~~go
Time.Tick(时间间隔)  //定时器 返回一个channel，这个channel每隔一段时间会被读出
~~~

时间按格式化

~~~go
2006年1月2号15点04分
    fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
    // 12小时制
    fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan"))
    fmt.Println(now.Format("2006/01/02 15:04"))
    fmt.Println(now.Format("15:04 2006/01/02"))
    fmt.Println(now.Format("2006/01/02"))
~~~

## IO操作

### 终端操作底层原理

终端其实是一个文件，相关实例如下：

- `os.Stdin`：标准输入的文件实例，类型为`*File`
- `os.Stdout`：标准输出的文件实例，类型为`*File`
- `os.Stderr`：标准错误输出的文件实例，类型为`*File`

```go
//以文件的方式操作终端
func main() {
    var buf [16]byte
    os.Stdin.Read(buf[:])
    os.Stdin.WriteString(string(buf[:]))
}
```

### 文件操作

- ` func Create(name string) (file *File, err Error)  ` 
  - 根据提供的文件名创建新的文件，返回一个文件对象，默认权限是0666
  
- `func NewFile(fd uintptr, name string) *File` 
  - 根据文件描述符创建相应的文件，返回一个文件对象
  
- `func Open(name string) (file *File, err Error)  ` 
  - 只读方式打开一个名称为name的文件
  
- `func OpenFile(name string, flag int, perm uint32) (file *File, err Error)  `
  - 打开名称为name的文件，flag是打开的方式，只读、读写等，perm是权限
  
- ` func (file *File) Write(b []byte) (n int, err Error) ` 
  - 写入byte类型的信息到文件
  
- ` func (file *File) WriteAt(b []byte, off int64) (n int, err Error) ` 
  - 在指定位置开始写入byte类型的信息
  
- `  func (file *File) WriteString(s string) (ret int, err Error)  `

  - 写入string信息到文件

- `  func (file *File) Read(b []byte) (n int, err Error)  `

  - 读取数据到b中

- `func (file *File) ReadAt(b []byte, off int64) (n int, err Error)  `

  - 从off开始读取数据到b中

- `  func Remove(name string) Error  `

  - 删除文件名为name的文件
