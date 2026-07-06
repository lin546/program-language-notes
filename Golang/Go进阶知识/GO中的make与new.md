# GO 中的 make 与 new

> Go 语言中 new 和 make 是两个内建函数，主要用来初始化类型，分配内存空间，但是它们之间有一些不同。
>

## 一、声明一个变量
```go
var a int
var b string
```

这里我们使用 var 关键字，声明两个变量，然后就可以在程序中使用。当我们不指定变量的默认值的时候呢，这些变量的默认值是它所属类型的零值。比如上面的 int 型它的零值为 0，string 的零值为 ""，引用类型的零值为 nil。

接下来尝试一下定义引用类型。

```go
package main

import "fmt"

func main() {
    var a *int
    *a = 10
    fmt.Println(*a)
}
```

<font style="color:rgb(99, 107, 111);">执行一下这个程序，它会发生 panic，原因是：</font>

```go
panic: runtime error: invalid memory address or nil pointer dereference
```

<font style="color:rgb(99, 107, 111);">从这我们可以看出，如果是一个引用类型，我们不仅要声明它，还要为它分配内存空间，否则我们赋值的 10 就无处安放。值类型的声明不需要我们分配内存空间，因为已经默认给我们分配好啦。</font>

## <font style="color:rgb(99, 107, 111);">二、new 关键字</font>
<font style="color:rgb(99, 107, 111);">我们使用 new 关键字来改写一下上面的 demo</font>

```go
package main

import "fmt"

func main() {
    var a *int
    a = new(int)
    *a = 10
    fmt.Println(*a)
}
```

<font style="color:rgb(99, 107, 111);">我们可以运行成功，并且成功打印出 10。下面贴出 new 内置函数的注释。</font>

```go
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly\
// allocated zero value of that type.
func new(Type) *Type
```

> 简单翻译： 此内置函数用于分配内存，第一个参数接收一个类型而不是一个值，函数**返回一个指向该类型内存地址的指针**，同时把分配的内存置为该类型的零值。
>

## 三、make 关键字
<font style="color:rgb(99, 107, 111);">make 也是用于分配内存，与 new 不同的是，它一般只用于 </font><font style="color:rgb(133, 128, 128);background-color:rgb(249, 250, 250);">chan</font><font style="color:rgb(99, 107, 111);">，</font><font style="color:rgb(133, 128, 128);background-color:rgb(249, 250, 250);">map</font><font style="color:rgb(99, 107, 111);">，</font><font style="color:rgb(133, 128, 128);background-color:rgb(249, 250, 250);">slice</font><font style="color:rgb(99, 107, 111);"> 的初始化，并且</font>**<font style="color:rgb(99, 107, 111);">直接返回这三种类型本身</font>**<font style="color:rgb(99, 107, 111);">。以下是 make 内置函数的注释。</font>

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
//  Slice: The size specifies the length. The capacity of the slice is
//  equal to its length. A second integer argument may be provided to
//  specify a different capacity; it must be no smaller than the
//  length. For example, make([]int, 0, 10) allocates an underlying array
//  of size 10 and returns a slice of length 0 and capacity 10 that is
//  backed by this underlying array.
//  Map: An empty map is allocated with enough space to hold the
//  specified number of elements. The size may be omitted, in which case
//  a small starting size is allocated.
//  Channel: The channel's buffer is initialized with the specified
//  buffer capacity. If zero, or the size is omitted, the channel is
//  unbuffered.
func make(t Type, size ...IntegerType) Type
```

<font style="color:rgb(99, 107, 111);">从声明也能看出，</font>**<font style="color:rgb(99, 107, 111);">返回的还是该类型</font>**<font style="color:rgb(99, 107, 111);">。</font>

## <font style="color:rgb(99, 107, 111);">四、两者异同</font>
+ <font style="color:rgb(125, 134, 136);">new 和 make 都是用于内存的分配。</font>
+ <font style="color:rgb(125, 134, 136);">make 只用于 chan，map，slice 的初始化。</font>
+ <font style="color:rgb(125, 134, 136);">new 用于给类型分配内存空间，并且置零。</font>
+ <font style="color:rgb(125, 134, 136);">make 返回类型本身，new 返回指向类型的指针。</font>

## <font style="color:rgb(125, 134, 136);">参考</font>
+ [https://learnku.com/articles/34137](https://learnku.com/articles/34137)



> 更新: 2023-07-15 19:28:06  
> 原文: <https://www.yuque.com/thinkspace/ovoe4b/uwbg67igbmvcvfpf>