

# 切片

### 底层实现

```go
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```



## 切片声明

| 序号 | 方式               | 代码示例                                             |
| ---- | ------------------ | ---------------------------------------------------- |
| 1    | 直接声明           | `var slice []int`                                    |
| 2    | new                | `slice := *new([]int)`                               |
| 3    | 字面量             | `slice := []int{1,2,3,4,5}`                          |
| 4    | make               | `slice := make([]int, 5, 10)`                        |
| 5    | 从切片或数组“截取” | `slice := array[1:5]` 或 `slice := sourceSlice[1:5]` |

- 但是所有的空切片的数据指针都指向同一个地址 `0xc42003bda0`(zerobase)。

| 创建方式      | nil切片              | 空切片                  |
| ------------- | -------------------- | ----------------------- |
| 方式一        | var s1 []int         | var s2 = []int{}        |
| 方式二        | var s4 = *new([]int) | var s3 = make([]int, 0) |
| 长度          | 0                    | 0                       |
| 容量          | 0                    | 0                       |
| 和 `nil` 比较 | `true`               | `false`                 |

## 截取

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
slice := data[2:4:6] // data[low, high, max]
```

- low是索引的开始位置，hign和max是开区间。high-1是最后元素的位置，max-1是最大容量位置。
- high==low的时候，slice为空。
- high和max必须在原数组或者原切片的容量(cap)范围内。



## 切片赋值或函数传递

- 对切片本身赋值或参数传递时，和数组指针的操作方式类似，只是赋值切片头信息(reflect.SliceHeader),并不会复制底层数组。
- 切片结构体中第一个元素是指针，指向数组中`slice` 指定的开始位置。切片是引用类型,所以当引用改变其中元素的值时，其他所有引用都会改变该值。
- 接上一条。如果在函数内部切片发生了内存地址的重新分配上述说法将不生效。



## Q:切片和数组的区别？

- slice的底层数据是数组，slice是对数组的封装，它描述一个数组的片段。两者都可以通过下标来访问单个元素。

- 数组是定长的，长度定义好之后，不能更改。长度是类型的一部分，比如`[3]int`和`[4]int` 就是不同的类型。 

- 切片比较灵活，可以动态扩容。切片的类型和长度无关。



## 切片append

- 使用 append 可以向 slice 追加元素，实际上是往底层数组添加元素。
- 切片的append操作在容量不足的情况下，会导致内存重新分配和复制数据。
- 通常情况下切片容量小于1024的时候，扩容后新切片是老切片的2倍。由于存在内存对齐，当容量大于等于1024的时候扩展的容量不止1.25倍。
- 当append连续扩容的场景

```go
	s := []int{1,2}
	s = append(s,4, 5, 6)
	//s = append(s,4)
	//s = append(s,5)
	//s = append(s,6)
	fmt.Printf("len=%d, cap=%d",len(s),cap(s))
```

输出

```
len=5, cap=6
```

原因：

切片扩容在调用`growslice`函数的时候第二个参数为oldcap为2，第三个参数cap传5。满足

```go
// et 元素类型
// old 原slice
// cap 新slice要求的最小容量
func growslice(et *_type, old slice, cap int) slice {
    // ……
    newcap := old.cap
	  doublecap := newcap + newcap
	  if cap > doublecap {
			newcap = cap
		} else {
			// ……
		}
		// ……
	
		capmem = roundupsize(uintptr(newcap) * ptrSize)
		newcap = int(capmem / ptrSize)
}
```



