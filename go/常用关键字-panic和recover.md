## Q:panic触发条件

- go语言中panic关键字用于主动抛出异常。



## Q:panic影响范围

- `panic` 只会触发**当前** Goroutine 的 `defer`。
- `panic` 能够改变程序的控制流，调用 `panic` 后会立刻停止执行当前函数的剩余代码，并在当前 Goroutine 中递归执行调用方的 `defer`。
- `panic` 允许在 `defer` 中嵌套多次调用。



## Q:为什么可以捕获panic，捕获后底层是怎么处理的

> panic->defer->recover，如果recover中再次有异常，会退出程序。
>
> 如果没有捕获异常，panic退出时会打印调用栈，最终调用exit(-2)退出整个进程。

- 编译器会在编译阶段，将panic关键字转换成`runtime.gopanic`，执行过程中的panic其实是在调用`runtime.gopanic` 函数。
- 在运行过程中遇到 `runtime.gopanic` 方法时，会从 Goroutine 的链表依次取出 `runtime._defer` 结构体并执行。
- 如果defer中调用`runtime.gorecover`函数，就会将`_panic.recovered`标记为true并返回。
- 然后调用`runtime.recover`函数，调用之后函数会根据传入的 `pc` 和 `sp` 跳转回defer。`runtime.deferproc`。



## Q:recover的作用和生效条件

- `recover` 可以中止 `panic` 造成的程序崩溃。它是一个只能在 `defer` 中发挥作用的函数，在其他作用域中调用不会发挥作用；
- 在其他的作用域中，由于当前Groutine并没有调用panic，所以`runtime.gorecover`函数会直接返回nil。



## Q: panic后为什么还可以执行的defer

- panic的撤退比较有秩序，他会先处理完当前goroutine已经defer挂上去的任务，执行完毕后再退出整个程序。



## Reference

[panic 和 recover源码分析](https://app.yinxiang.com/shard/s43/nl/13675070/f423470d-c753-4d0c-b31a-c6276930de60)

[印象笔记panic 和 recover原理](https://app.yinxiang.com/shard/s43/nl/13675070/a578c175-26fb-44a6-846c-031cbe066636)

[GO 设计与实现 panic  recover](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-panic-recover/#544-%E5%B4%A9%E6%BA%83%E6%81%A2%E5%A4%8D)

[阿波张  - 函数栈的调用过程](https://app.yinxiang.com/shard/s43/nl/13675070/d8710725-d75d-4859-9508-cac794f7d787)

