## Q:什么是CSP

- csp是通信顺序进程（Communicating Sequential Processes）
- Goroutine 和 Channel 分别对应 CSP 中的实体和传递信息的媒介，Goroutine 之间会通过 Channel 传递数据。
- Go中也可以使用共享内存加互斥锁的方式进行通信。

## Q：channel底层的数据结构是什么

- 缓冲型的channel底层是循环数组。
- 接收和等待的G队列是链表。

```go
// runtime/chan.go
type hchan struct {
	qcount   uint           // chan中元素的数量
	dataqsiz uint           // chan底层ring buffer的长度
	buf      unsafe.Pointer // 指向底层循环数组的指针，只针对有缓冲的channel
	elemsize uint16         // chan 中元素大小  是单个元素大小
	closed   uint32         // != 0 为关闭
	elemtype *_type         // chan中元素类型
	sendx    uint           // 已发送元素在ring buffer中的索引
	recvx    uint           // 已接收元素在ring buffer中的索引
	recvq    waitq          // 等待接收的goroutine队列，链表
	sendq    waitq          // 等待接收的goroutine队列

	lock     mutex          // 锁
}

// 双向链表
type waitq struct {
	first *sudog
	last  *sudog
}
```



## Q：向channel发送数据的过程是怎么样的

- channel为nil的情况处理。
- 非阻塞直接返回。
- 加锁
- channel关闭直接panic。给nil的channel发送数据也是在这一步panic的
- 接收队列里有goroutine(此时buffer中无数据)，直接将要发送的数据拷贝到接收goroutine(用到写屏障，减少了内存copy)。
- 缓冲型的channel有空间就放入缓冲空间。
- channel非阻塞直接返回。
- 当前的goroutine直接挂起。
- 唤起接收者(先阻塞先唤醒)

## Q:从channel接收数据的过程是什么

- channel为nil。非阻塞模式直接返回。阻塞模式下会gopark，因为nil的channel无法关闭。所以会造成泄漏。

- 非阻塞并且没有内容可接收直接返回。
- 加锁。
- channel关闭，仍然接收数据。
- 会调用recv函数，处理待发送队列里的goroutine。非缓冲型直接进行值拷贝，缓冲型拷走buf队首元素，发送者的数据拷贝到buf，唤醒发送的goroutine。
- 处理缓冲型buf中的数据。
- 无可以处理数据，构造待接收队列

## Q:关闭一个channel的过程是怎样的

- 关闭nil的channel会panic
- 上锁
- 如果channel已经关闭会panic
- c.closed = 1
- 处理接收者队列
- 处理发送者队列。如果发送者队列有数据会panic
- 

## Q:channel发送和接收元素的本质是什么

channel的发送和接收操作本质上都是值拷贝。

```go

type user struct {
	name string
	age int8
}

var u = user{name: "Ankur1", age: 25}
var g = &u

func modifyUser(pu *user) {
	fmt.Println("modifyUser Received Vaule", pu)
	pu.name = "Anand"
}

func printUser(u <-chan *user) {
	time.Sleep(2 * time.Second)
	fmt.Println("printUser goRoutine called", <-u)
}

func main() {
	c := make(chan *user, 5)
	c <- g
	fmt.Println(g)
	// modify g
	g = &user{name: "Ankur Anand", age: 100}

	// 1 后执行，打印值的值是最开始传入channel的值
	// channel会将 g的值做拷贝，不是拷贝指针
	go printUser(c)

	// 2 先执行，打印传入的结构体信息，并且修改name为Ankur
	go modifyUser(g)
	time.Sleep(5 * time.Second)

	// 打印的是修改后的值
	fmt.Println(g)
}
//&{Ankur1 25}
//modifyUser Received Vaule &{Ankur Anand 100}
//printUser goRoutine called &{Ankur1 25}
//&{Anand 100}
```



## Q:从一个关闭的channel中仍然能读出数据吗

可以从已经closed的channel中接收值：

### 带 buffer 的 channel

如果 channel 中有值，那么就从 channel 中取，如果没有值，那么会返回 channel 元素的 0 值。

区分是返回的零值还是 buffer 中的值可使用 comma, ok 语法：

```go
x, ok := <-ch
```

若 ok 为 false，表明 channel 已被关闭，所得的是无效的值。

### 不带buffer的channel

取到的值都是默认值

```go
ch := make(chan int)
close(ch)
x := <-ch // 0
x, ok := <-ch // 0 false
```



## Q:一个关闭的channel中能写入数据吗

被关闭的 channel 不能再向其中发送内容，否则会 panic

```go
ch := make(chan int)
close(ch)
ch <- 1 // panic: send on closed channel
```

注意，如果 close channel 时，有 sender goroutine 挂在 channel 的阻塞发送队列中，会导致 panic：

```go
func main() {
    ch := make(chan int)
    go func() { ch <- 1 }() // panic: send on closed channel
    time.Sleep(time.Second)
    go func() { close(ch) }()
    time.Sleep(time.Second)
    x, ok := <-ch
    fmt.Println(x, ok)
}
```

close 一个 channel 会唤醒所有等待在该 channel 上的 g，并使其进入 Grunnable 状态，这时这些 writer goroutine 会发现该 channel 已经是 closed 状态，就 panic了。

## Q:如何优雅关闭channel

如果确定不会有 goroutine 在通信过程中被阻塞(可能是发送者阻塞或者接收者阻塞)，关闭会造成panic，可以不关闭 channel，等待 GC 对其进行回收。

### 关闭channel的原则

- 不要在receiver侧关闭channel，也不要在有多个sender时，关闭channel。（不发送数据给或关闭一个关闭状态的通道。）

### 情景1：1个发送者，M个接受者

发送者去关闭channel

### 情景2：N个发送者，1个接收者

借助外部额外 channel 来做信号广播。接收者关闭信号channel下达关闭指令。发送者在监听到关系信号后，停止发送数据。

本质上并没有关闭原始channel

### 情景3：N个发送者，M个接收者

借助外部额外 channel 来做信号广播。

设置主持人角色？？？？

## Q:channel应用

- 停止信号
- 任务定时
- 解耦生产者消费者
- 控制并发

```go
var limit = make(chan int, 3)

func main() {
    // …………
    for _, w := range work {
        go func() {
            limit <- 1
            w()
            <-limit
        }()
    }
    // …………
}
// 启动了
```



## Q:资源泄漏

goroutine处于发送或接收阻塞状态，但一直未唤醒。垃圾回收器并不会收集此类资源，导致它们会在等待队列里长久休眠，形成资源泄漏。



## Reference

[码农桃花源-深度解密channel](https://www.qcrao.com/2019/07/22/dive-into-go-channel/)

[曹大-channel源码解读](https://github.com/cch123/golang-notes/blob/master/channel.md)

[如何优雅关闭channel](https://learnku.com/go/t/23459/how-to-close-the-channel-gracefully)

[go高级编程-常见的并发模型](https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-06-goroutine.html)

[channel特性](https://www.pengrl.com/p/21027/)

