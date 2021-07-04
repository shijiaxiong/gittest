## Context

>  context 包是Go 1.7中引入的标准库，context用于在 goroutine 之间传递
>
>  取消信号（Context.Done）、
>
>  超时时间（context.WithTimeout）、
>
>  截止时间（context.WithDeadline）
>
>  共享的值（context.WithValue ）等。

[深度解密GO语言-context](https://qcrao.com/2019/06/12/dive-into-go-context/)

[Context 应用示例](https://app.yinxiang.com/shard/s43/nl/13675070/bdca16fd-9de7-48ed-9682-42954c91198b)



## Q:sync.Mutex

- mutex是互斥锁，不区分读写，无论是读写都是互斥的。

- 两种工作模式：

  - 正常模式：锁的等待者会按照先进先出的顺序获取锁。刚唤醒的锁要与新来的goroutine竞争。

  - 饥饿模式：当前的goroutine将mutex所有权移直接交给等待队列最前端的goroutine。

    

一般情况下互斥锁比atomic慢的原因：互斥锁会有goroutine的上下文切换。

```go
var x int64
var wg sync.WaitGroup
var lock sync.Mutex

func add() {
    for i := 0; i < 5000; i++ {
        lock.Lock() // 加锁
        x = x + 1
        lock.Unlock() // 解锁
    }
    wg.Done()
}
func main() {
    wg.Add(2)
    go add()
    go add()
    wg.Wait()
    fmt.Println(x)
}
```



## Q:RWMutex 读写互斥锁

互斥锁是完全互斥的，但是有很多实际的场景下是读多写少的，当我们并发的去读取一个资源不涉及资源修改的时候是没有必要加锁的，这种场景下使用读写锁是更好的一种选择。读写锁在Go语言中使用sync包中的RWMutex类型。

读写锁分为两种：读锁和写锁。当一个goroutine获取读锁之后，其他的goroutine如果是获取读锁会继续获得锁，如果是获取写锁就会等待；当一个goroutine获取写锁之后，其他的goroutine无论是获取读锁还是写锁都会等待。



## sync.atomic

- `sync/atomic`包将底层硬件提供的原子操作封装成了 Go 的函数。

- 但这些操作只支持几种基本数据类型，因此为了扩大原子操作的适用范围，Go 语言在 1.4 版本的时候向`sync/atomic`包中添加了一个新的类型`Value`。此类型的值相当于一个容器，可以被用来“原子地"存储（Store）和加载（Load）**任意类型**的值。
- 原子操作由**底层硬件**支持，而锁则由操作系统的**调度器**实现。锁应当用来保护一段逻辑，对于一个变量更新的保护，原子操作通常会更有效率，并且更能利用计算机多核的优势，如果要更新的是一个复合对象，则应当使用`atomic.Value`封装好的实现。

##### CAS(乐观锁)

CAS实现原理：CAS有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做并返回false。

atomic.CompareAndSwapInt32()

**CAS缺陷**

ABA问题

循环时间长开销大

只能保证一个共享变量的原子操作



[atomic.value 的前世今生](https://app.yinxiang.com/shard/s43/nl/13675070/f3680664-5c7f-4f21-93ff-1bbbeae1aaa5)



## sync.Map

```go
// sync/map.go
type Map struct {
   // 当写read map 或读写dirty map时 需要上锁
   mu Mutex

   // read map的 k v(entry) 是不变的，删除只是打标记，插入新key会加锁写到dirty中
   // 因此对read map的读取无需加锁
   read atomic.Value // 保存readOnly结构体

   // dirty map 对dirty map的操作需要持有mu锁
   dirty map[interface{}]*entry

   // 当Load操作在read map中未找到，尝试从dirty中进行加载时(不管是否存在)，misses+1
   // 当misses达到diry map len时，dirty被提升为read 并且重新分配dirty
   misses int
}

// read map数据结构
type readOnly struct {
   m       map[interface{}]*entry
   // 为true时代表dirty map中含有m中没有的元素
   amended bool
}

type entry struct {
   // 指向实际的interface{}
   // p有三种状态:
   // p == nil: 键值已经被删除，此时，m.dirty==nil 或 m.dirty[k]指向该entry
   // p == expunged: 键值已经被删除， 此时, m.dirty!=nil 且 m.dirty不存在该键值
   // 其它情况代表实际interface{}地址 如果m.dirty!=nil 则 m.read[key] 和 m.dirty[key] 指向同一个entry
   // 当删除key时，并不实际删除，先CAS entry.p为nil 等到每次dirty map创建时(dirty提升后的第一次新建Key)，会将entry.p由nil CAS为expunged
   p unsafe.Pointer // *interface{}
}
```

- 适用于key比较固定，读多写少的场景。
- 读`Map.read`使用`atomic.value`，无需加锁。写

[go sync.map的实现细节](https://app.yinxiang.com/shard/s43/nl/13675070/24f64918-c715-4f87-a419-5a212b9e323b)

[sync.map ](https://blog.csdn.net/u010853261/article/details/103848666)



## WaitGroup

##### 32 与 64位的内存对齐

> 为了提升性能不加锁，其实是把 `counter` 和 `waiter` 看成一个 64 位整数进行处理。

- 64位架构中一次事务处理的长度是8bytes,如果state1后边两个元素表示一个字段的话，CPU需要读取内存两次，不能保证原子性。
- 32位架构中想要原子性的操作8bytes,需要由调用方保证其数据地址是64位对齐。state1的第一个元素做padding，用state1的后两个元素合并成unit64来表示statep。

[Go中由WaitGroup引发对内存对齐思考](https://www.luozhiyun.com/archives/429)

[Golang WaitGroup 原理深度剖析](https://www.cyhone.com/articles/golang-waitgroup/)



## Once

保证在 Go 程序运行期间的某段代码只会执行一次,例如只加载一次配置文件、只关闭一次通道等。

并发的场景下，某一项操作(初始化并赋值)全部完成后才能让其他的goroutine操作。



## Cond

暂缺



## Pool

保存和复用临时对象，减少内存分配，降低GC的压力。

> Get 返回 Pool 中的任意一个对象。如果 Pool 为空， 则调用 New 返回一个新创建的对象。
>
> 放进 Pool 中的对象，会在说不准什么时候被回收 掉。所以如果事先 Put 进去 100 个对象，下次 Get 的 时候发现 Pool 是空也是有可能的。不过这个特性的 一个好处就在于不用担心 Pool 会一直增长，因为 Go 已经帮你在 Pool 中做了回收机制。
>
> 这个清理过程是在每次垃圾回收之前做的。之前每次 GC 时都会清空 pool，而在1.13版本中引入了 victim cache，会将 pool 内数据拷贝一份，避免 GC 将其清 空，即使没有引用的内容也可以保留最多两轮 GC。



## ErrGroup

- [`golang/sync/errgroup.Group`](https://draveness.me/golang/tree/golang/sync/errgroup.Group) 在出现错误或者等待结束后会调用 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的 `cancel` 方法同步取消信号；
- 只有第一个出现的错误才会被返回，剩余的错误会被直接丢弃



## I/O 模型

go语言中使用了多模块网络轮训器，会根据目标平台选择树中特定的分支进行编译。Linux环境下netpoll->epoll

## 定时器 

#### Timer

时间到了执行一次

如果timer定时器要每隔间隔的时间执行，实现ticker的效果，使用 func (t *Timer) Reset(d Duration) bool

#### Tricker

时间到了执行多次



未读References

编写可维护代码-并发
https://dave.cheney.net/practical-go/presentations/qcon-china.html#_concurrency 
共享内存通信
https://blog.golang.org/codelab-share

#### 如果对齐内存的写入是原子性的，为什么我们还需要sync/atomic包

https://dave.cheney.net/2018/01/06/if-aligned-memory-writes-are-atomic-why-do-we-need-the-sync-atomic-package

中文地址：https://www.jianshu.com/p/92fab32580f8

CPU的运行速度始终快于主内存。为了隐藏主内存的等待时间，在CPU和主内存之间还存在多级缓存。为了抚平这种差异所以需要atomic包。

GO数据竞争检查
http://blog.golang.org/race-detector

冰激凌制造商和数据竞争
https://dave.cheney.net/2014/06/27/ice-cream-makers-and-data-races
冰激凌制造商和数据竞争2
https://www.ardanlabs.com/blog/2014/06/ice-cream-makers-and-data-races-part-ii.html

### atomic包与mutex包的比较

https://medium.com/a-journey-with-go/go-how-to-reduce-lock-contention-with-the-atomic-package-ba3b2664b549

一般情况下atomic性能优于metux，从benchmark的pprof运行结果可以看出，metux有较多的上下文切换

trace包
https://medium.com/a-journey-with-go/go-discovery-of-the-trace-package-e5a821743c3c

#### 互斥锁和饥饿

https://medium.com/a-journey-with-go/go-mutex-and-starvation-3f4f4e75ad50

中文地址： https://xie.infoq.cn/article/ab2a8a779cd7ef510adce957e

##### Go1.8 barging

当锁被释放时，它将唤醒第一个等待者，并将锁交给第一个传入请求的goroutine或此已唤醒的goroutine

##### handoff

释放后，互斥锁将保持锁，直到第一个等待goroutine准备好获取它。这将降低吞吐量，因为即使有其他goroutine请求也无法获取到锁

##### spinning

当等待的队列为空或应用程序大量使用互斥锁时，自旋很有用。 停放和唤醒的成本很高，可能比仅自旋等待下一个锁的获取要慢

##### Go1.9饥饿

所有等待锁定超过一毫秒的goroutine，也称为有界等待，将被标记为饥饿。 当标记为饥饿时，解锁方法现在将把锁直接移交给第一位等待着

## Q:常见的并发模型

- channel：关注数据流向

- WaitGroup:等待同时返回结果
- context

