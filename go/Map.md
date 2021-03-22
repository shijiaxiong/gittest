## Q：MAP的内存模型

```go
// runtime/map.go
type hmap struct {
  // 元素个数，调用 len(map) 时，直接返回此值
	count     int
	flags     uint8
	// buckets 的对数 log_2，表示桶的个数
	B         uint8
	// overflow 的 bucket 近似数
	noverflow uint16
	// 哈希种子，计算 key 的哈希的时候会传入哈希函数
	hash0     uint32
  // 指向 buckets 数组，大小为 2^B
  // 如果元素个数为0，就为 nil
	buckets    unsafe.Pointer
	// 扩容的时候，buckets 长度会是 oldbuckets 的两倍
	oldbuckets unsafe.Pointer
	// 指示扩容进度，小于此地址的 buckets 迁移完成
	nevacuate  uintptr
	extra *mapextra // optional fields
}

type mapextra struct {
	overflow    *[]*bmap // 溢出桶
	oldoverflow *[]*bmap
	nextOverflow *bmap
}

// buckets指向的结构体
type bmap struct {
	tophash [bucketCnt]uint8
}
```

- `hash0`是哈希的种子，在初始化的时候调用fastrand()创建。
- `bmap`的buckets值(指针)指向了`bmap`，存储具体的key和value
- `bmap`中的key和value各自单独存放。这样可以在某些情况下省略掉padding字段，节省内存空间。
- 一个bmap只能存储8个键值对，当单个桶数据装满的时候就会使用溢出桶去存储数据。

例如：

```go
map[int64]int8 // 由于内存对齐，如果k-v格式存放会额外padding7个字节
```



## Q: MAP是如何初始化的

- 字面量创建(编译器会优化)、)运行时创建。
- 无论字面量还是运行时`map`的创建调用了`runtime.makemap`函数，函数返回一个`*hmap`指针
- `map`作为函数参数时，在函数参数对map的操作会影响`map`本身。(GO语言中的函数传参都是值传递!!!)
- 当桶的数量小于 2的4次方(B<4) 时，由于数据较少、使用溢出桶的可能性较低，会省略创建的过程以减少额外开销；
- 当桶的数量大于等于 2的4次方(B>=4))时，会额外创建 2的B−4次个溢出桶。溢出桶和正常桶是内存连续的。



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

```go
// src/runtime/hashmap.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
```

- 带comma和不带comma。带comma回有布尔型的返回值。
- 根据key的不同类型，编译器还会将查找、插入、删除的函数用更具体的函数替换，优化效率。



## Q:map的遍历

1、找到初始化随机的bucket值和cell值。
2、找到后判断是否发生过搬迁 b.tophash
3、如果搬迁完成就遍历当前的bucket 和overflow。否则回到之前的oldbucket找到会迁移到当前bucket的值



## Q：map的赋值

1、检查hmap的flags标志位是否被置为4，如果有其他协程在写会导致panic
2、key定位到某个bucket后需要确保这个bucket对应的老bucket完成迁移。
3、bucket中可以存新key，key会安置在第一次发现的空位（tophash的empty）
4、bucket中不可以存新key，挂在bucket的overflow bucket上。
5、mapassign函数最终返回了key所对应value的位置指针，进行赋值。





## Q:如何进行扩容

- 一个bucket装载8个key。当哈希表中存储的数据过多，单个桶已经装满时就会使用 `extra.nextOverflow` 中桶存储溢出的数据。溢出桶和正常桶在内存中是连续存储的。

### 触发条件

- 装载因子超过阈值，6.5。装载因子`loadFactor := count / (2^B)`，count是map元素的个数,2^B是桶的个数。
- `overflow bucket`数量过多。

### 扩容步骤

- 扩容的原因是溢出桶过多，那么此次扩容就是等量扩容。等量扩容创建的桶数量和之前一致。
- **分配**：`hashGrow`分配新的桶和预创建溢出桶，随后将原有的桶数组设置到oldbuckets上，并将新的空桶设置到buckets上，溢出桶也使用了相同的逻辑。
- **迁移**：扩容不是原子的，而是通过`growWork()`触发。在扩容期间访问哈希表时(插入、修改、删除)会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。当哈希表的容量翻倍时，每个旧桶的元素都会流到新创建的两个桶中。





## Q:map中key的删除

1、对bucket和cell的两层循环找到对应的元素。

2、 将key、value置为nil。

3、 将count值减一，对应位置的tophash置为empty。

## Q:map可以边遍历边删除吗

不可以。同时读写一个map，如果被测到，会直接panic。

**详解**：

- 写操作的方法`mapassign`和删除的方法`mapdelete`都会对h.flags标记为hashWriting。

- 读取map元素可能调用`mapaccess1`，`mapaccess2`，这两个方法都会检测读写冲突。当发现冲突后会panic。

```go
// src/runtime/hashmap.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)

// 读写冲突检测
if h.flags&hashWriting != 0 {
	throw("concurrent map read and map write")
}
```



## Q:map的key可以是浮点型吗

- 从语法上看除了slice、map、functions这几个类型，其他都可以做map的key。
- 布尔值、数字、字符串、指针、通道、接口类型、结构体、只包含上述类型的数组。这些类型的共同特征是支持 == 和 != 操作符。
- 使用float做key要慎重。 math.Nan()做key的时候哈希函数会生成一个随机数。

## Q:可以对map的元素取地址吗

无法对map的key或者value进行取地址。在编译阶段就会报错。

```go
package main
import "fmt"
func main() {
    m := make(map[string]int)
    fmt.Println(&m["hello world"])
}
```

如果通过其他 hack 的方式，例如 unsafe.Pointer 等获取到了 key 或 value 的地址，也不能长期持有，因为一旦发生扩容，key 和 value 的位置就会改变，之前保存的地址也就失效了。

# Reference

[深度解密Go语言之map](https://www.qcrao.com/2019/05/22/dive-into-go-map/)

[理解Golang哈希表Map的原理](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/)

