## Q: 底层数据结构

```go
// sync/mutex.go
type Mutex struct {
	state int32 // mutex当前的状态
	sema  uint32 // 信号量，用于唤醒goroutine
}

// state状态 常亮
const (
	mutexLocked = 1 << iota
	mutexWoken
	mutexStarving
	mutexWaiterShift = iota
	starvationThresholdNs = 1e6
)
```

- `mutexLocked`值为1。1表示已加锁，0表示未加锁。
- `mutexWoken`值为2(二进制：10)。表示mutex的唤醒状态，1已唤醒(有自旋的G)，0表示未唤醒。
- `mutexStarving`值为4(二进制：100)。表示mutex的工作模式。1表示饥饿模式，0表示正常模式。
- `metexWaiterShift`值为3。当前等待的goroutine数目。通过`mutex.state >> mutexWaiterShift`计算。
- `starvationThresholdNs` 1e6纳秒，也就是1毫秒。正常模式下当等待队列中队首的goroutine等待时间超过它，mutex进入饥饿模式。



## Q: 工作模式

### 正常模式

- 锁的等待者会按照先进先出的顺序获取锁。
- **切换到饥饿模式**：一个刚刚被唤醒的goroutine在和新来的goroutine竞争的时候，有时候并不能获取到mutex。(新来的goroutine已经在CPU上运行并且数量较多)。一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式。

### 饥饿模式

- 当前的goroutine将mutex所有权移直接交给等待队列最前端的goroutine。
- 饥饿模式下新来的goroutine直接在队列末尾等待。
- **切换到正常模式**：当一个等待者获取到mutex所有权，并且满足任意一种：1.是队列中的最后一个；2.等待时间少于1ms。



## Q:加锁的过程

- **无冲突**通过cas操作把当前状态设置为加锁状态
- **有冲突，开始自旋**等待锁释放，如果其他 goroutine 在这段时间内释放了该锁，直接获得该锁；如果没有释放，进入3
- **有冲突，且过了自旋阶段**  当前goroutine进入等待阶段。



## Q：Goroutine进入自旋的条件

- 锁已被占用，并且锁不处于饥饿模式。
- 积累的自旋次数小于最大自旋次数（active_spin=4），每次自旋30个时钟周期。
- cpu核数大于1；
- 有空闲的P；
- 当前goroutine所挂载的P下，本地待运行队列为空。

### 阻塞和唤醒机制

- go的阻塞和唤醒是semacquire和semrelease。虽然命名上是sema，但实际用途却是一套阻塞唤醒机制。



## Q：channel与mutex的比较



## Reference

[sync-Mutex源码分析](https://reading.hidevops.io/articles/sync/sync_mutex_source_code_analysis/)

[自旋 饥饿模式引入的原因](https://app.yinxiang.com/shard/s43/nl/13675070/44220bd7-07e0-4141-be64-fc18ed943a5f)

[mutex 的woken公平、使用锁带来的问题、阻塞唤醒机制](https://app.yinxiang.com/shard/s43/nl/13675070/af539023-483f-4999-ac6e-bcdf5295b122)

