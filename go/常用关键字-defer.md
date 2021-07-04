## 数据结构

- defer结构体是延时调用链表上的一个元素，所有结构体都通过link字段串联成链表。
- 只要获取到defer结构体就会最加到Groutine 的 _defer 链表最前面。

```go
type _defer struct {
	siz       int32
	started   bool
	openDefer bool
	sp        uintptr
	pc        uintptr
	fn        *funcval
	_panic    *_panic
	link      *_defer
}
```



## defer的三种设计和实现原理

### 1、堆上分配

- 编译期间会将defer关键字转换成`runtime.deferproc`函数，这个函数会生成`runtime._defer`结构体，追加到goroutine的`_defer` 链表的最前面。
- 在有defer关键字的末尾插入`runtime.deferreturn` 函数，这个函数会从前往后执行goroutine的`_defer`链表。

### 2、栈上分配(Go 1.13版本新增)

- defer关键字在函数中最多执行一次时会转换为在栈上执行。

### 3、 开放编码(Go 1.14版本新增)

- 编译期间通过defer关键字、return语句个数确定是否开启开放编码优化。
- 通过`deferBits`存储defer关键字信息。
- 如果defer关键字的执行可以在编译期间确定，会在函数返回前直接插入相应的代码，否则由运行时的`runtime.deferreturn`处理。



#### 开放编码的使用条件

- 函数的 `defer` 数量少于或者等于 8 个；
- 函数的 `defer` 关键字不能在循环中执行；
- 函数的 `return` 语句与 `defer` 语句的乘积小于或者等于 15 个；



### defer使用传值的方式传递参数时会进行预计算原因

- 调用 `runtime.deferproc` 函数创建新的延迟调用时就会立刻拷贝函数中引用的外部参数，函数的参数不会等到真正执行时计算。

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}

执行结果：0
```



### defer表达式中可以修改函数中的命名返回值

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}			
执行结果：2

func f() (r int) {
     t := 5
     defer func() {
       t = t + 5
     }()
     return t
}
执行结果：5

return xxx
等价
返回值 = xxx
调用defer函数
空的return
```



### Reference

[defer 示例](https://app.yinxiang.com/shard/s43/nl/13675070/e701678c-4d52-41aa-a21c-b17ff8a4b6e3)

