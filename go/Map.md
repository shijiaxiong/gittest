## Q：MAP的内存模型

- 数据结构包含`hmap`,`bmap`,`mapextra`
- `bmap`的buckets值(指针)指向了`bmap`，存储具体的key和value
- `bmap`中的key和value各自单独存放。这样可以在某些情况下省略掉padding字段，节省内存空间。
- 当哈希表中存储的数据过多，单个桶已经装满时就会使用 `extra.nextOverflow` 中桶存储溢出的数据。溢出桶和正常桶在内存中是连续存储的。

例如：

```go
map[int64]int8 // 由于内存对齐，如果k-v格式存放会额外padding7个字节
```

## Q: MAP是如何初始化的

- 字面量创建：编译器优化。
- 运行时创建:
- 无论字面量还是运行时`map`的创建调用了`runtime.makemap`函数，函数返回一个`*hmap`指针
- `map`作为函数参数时，在函数参数对map的操作会影响`map`本身。(GO语言中的函数传参都是值传递!!!)

## Q:哈希函数与冲突解决

- `runtime/alg.go`会检测处理器是否支持aes,如果支持则使用aes hash,否则使用memhash。
- 用拉链法(链表)解决hash冲突

## Q:key定位过程

### 定位桶

- 64位机器下，key经过哈希计算后得到64个bit位的哈希值，取低五位(假设hmap.B = 5)计算桶的位置。

### 定位key

- 用哈希的高8位，找到key在bucket中的位置。

```
10010111 | 000011110110110010001111001010100010010110010101010 │ 01010

01010 - 第10号桶
10010111 - 151位tophash值
```

## Q:map的两种get操作

- 带comma和不带comma。带comma回有布尔型的返回值。
- 根据key的不同类型，编译器还会将查找、插入、删除的函数用更具体的函数替换，优化效率。

## Q:如何进行扩容

- 一个bucket装载8个key
- 装载因子`loadFactor := count / (2^B)`
- 

### 触发条件

- 装载因子超过阈值，6.5
- `overflow bucket`数量过多。

### 扩容步骤

- 扩容的原因是溢出桶过多，那么此次扩容就是等量扩容。等量扩容创建的桶数量和之前一致。
- `hashGrow`分配新的buckets，并将老的buckets挂载到oldbuckets字段。
- 真正搬迁的动作在`growWork()`中，会在插入、修改、删除的时候搬迁buckets。

## Q:map的遍历

1、找到初始化随机的bucket值和cell值
2、找到后判断是否发生过搬迁 b.tophash
3、如果搬迁完成就遍历当前的bucket 和overflow。否则回到之前的oldbucket找到会迁移到当前bucket的值

## Q：map的赋值

1、检查hmap的flags标志位是否被置为4，如果有其他协程在写会导致panic
2、key定位到某个bucket后需要确保这个bucket对应的老bucket完成迁移。
3、bucket中可以存新key，key会安置在第一次发现的空位（tophash的empty）
4、bucket中不可以存新key，挂在bucket的overflow bucket上。
5、mapassign函数最终返回了key所对应value的位置指针，进行赋值。

## Q:map可以边遍历边删除吗

不可以。同时读写一个map，如果被测到，会直接panic

## Q:map的key可以是浮点型吗

- 从语法上看除了slice、map、functions这几个类型，其他都可以做map的key。
- 布尔值、数字、字符串、指针、通道、接口类型、结构体、只包含上述类型的数组。这些类型的共同特征是支持 == 和 != 操作符。
- 使用float做key要慎重。 math.Nan()做key的时候哈希函数会生成一个随机数。

# Reference

[深度解密Go语言之map](https://www.qcrao.com/2019/05/22/dive-into-go-map/)

[理解Golang哈希表Map的原理](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/)

