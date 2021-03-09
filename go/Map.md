## Q：MAP的内存模型

- 数据结构包含`hmap`,`bmap`,`mapextra`
- `bmap`的buckets值(指针)指向了`bmap`，存储具体的key和value
- `bmap`中的key和value各自单独存放。这样可以在某些情况下省略掉padding字段，节省内存空间。

例如：

```go
map[int64]int8 // 由于内存对齐，如果k-v格式存放会额外padding7个字节
```

## Q: MAP是如何初始化的

- `map`的创建调用了`runtime.makemap`函数，函数返回一个`*hmap`指针
- `map`作为函数参数时，在函数参数对map的操作会影响`map`本身。(GO语言中的函数传参都是值传递!!!)

## Q:哈希函数与冲突解决

- `runtime/alg.go`会检测处理器是否支持aes,如果支持则使用aes hash,否则使用memhash。
- 用拉链法(链表)解决hash冲突



