## Q:什么是堆栈？

>  在计算机中堆栈的概念分为：数据结构的堆栈和内存分配中堆栈

### 数据结构中的堆栈

**堆**：可以看成一棵树。

**栈**：一种先进后出的数据结构。

###内存分配中的堆栈

**堆**：一般由程序员分配释放,若程序员不释放，程序结束时可能由OS回收，分配方式倒是类似于链表。

**栈**： 由操作系统自动分配释放,存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。

###堆栈的缓存方式

**堆**：二级缓存，生命周期由虚拟机的垃圾回收算法来决定（并不是一旦成为孤儿对象就能被回收）。所以调用这些对象的速度要相对来得低一些。

**栈**： 一级缓存，他们通常都是被调用时处于存储空间中，调用完毕立即释放。

> 一级缓存可分为一级指令缓存和一级数据缓存。一级指令缓存用于暂时存储并向CPU递送各类运算指令；一级数据缓存用于暂时存储并向CPU递送运算所需数据。
>
> 二级缓存就是一级缓存的缓冲器，存储那些CPU处理时需要用到、一级缓存又无法存储的数据。

## Q:Go中变量是分配在堆上还是栈上

编译器在编译阶段决定变量分配在堆还是栈上。



## Q:go逃逸分析

### 检测方式

- `go build -gcflags '-m -l' main.go`(-m打印逃逸分析信息，-l禁止内联编译)
- `go tool compile -S main.go | grep runtime.newobject`（汇编代码中搜runtime.newobject指令，该指令用于生成堆对象）

### 函数传值使用值传递好，还是指针传递好？





## Q:栈的初始化

- 栈空间在运行的时候包含两个重要的全局变量`runtime.stackpoll`和`runtime.stackLarge`，这两个变量分别表示全局的栈缓存和大栈缓存。前者可以分配小于 32KB 的内存，后者用来分配大于 32KB 的栈空间。

- `runtime.stackpoll`和`runtime.stackLarge` 用于分配空间的全局变量都与内存管理单元 `runtime.mspan` 有关，我们可以认为 Go 语言的栈内存都是分配在堆上的，运行时初始化会调用 `runtime.stackinit` 初始化这些全局变量。
- 在每一个`runtime.mcache`中都加入了栈缓存减少锁竞争。

## Q:goroutine的栈

### goroutine stack多大呢？

linux系统下每个goroutine（g0除外，g0分配64k）在初始化时stack大小都为2KB, 运行过程中会根据不同的场景做动态的调整。

### 栈扩容

- 运行时会检查当前goroutine的栈内存空间是否充足。
- 检测到当前栈大小不够用(SP大于可用内存空间)，调用morestack进行动态扩容。
- `runtime.morestack`函数的主要功能是保存当前的栈的一些信息，然后转换成调度器的栈调用`runtime.newstack`
- `runtime.newstack`分配一个2倍于当前大小的新栈。
- 旧栈中的内容拷贝到新栈中，函数在新栈中运行(栈扩容后一个变量的内存地址会发生变化)。
- 释放旧栈。

### 栈缩容

- 函数返回时处理栈缩小的问题。
- 栈的收缩是垃圾回收的过程中实现的。当检测到栈只使用了不到1/4时，栈缩小为原来的1/2，如果新栈的大小低于程序的最低限制 2KB，那么缩容的过程就会停止。

### 对服务有什么影响吗？如何排查栈扩容缩容带来的问题呢?

栈的扩容会有”中断“并栈拷贝。缩容会存在栈拷贝和写屏障。对内存占用和延时敏感的服务中，可能面临内存占用高、服务不稳定的状况。

## Reference

- [为何说Goroutine的栈空间可以无限大](http://blog.xiayf.cn/2014/01/17/goroutine-stack-infinite/)
- [Goroutine stack-扩缩容对服务的影响](https://studygolang.com/articles/10597)
- [go语言连续栈](https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.5.html)
- [go栈内存管理](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-stack-management/)
- [解密Go协程的栈内存管理](https://juejin.cn/post/6871550379432574990)
- [go堆栈内存管理-带图](https://studygolang.com/articles/25547)
- 

