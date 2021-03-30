---
title: Go笔记-数组与切片
date: 2021-03-18 11:35:36
tags: go
---

Go与其他语言一样，使用```[]```来表示数组，不过数组在Go中不够灵活，几乎不怎么使用，而是使用切片```slice```来代替。切片```slice```在构建数组上非常便捷，而且功能强大。

<!-- more -->

#### 数组

Go语言中数组是一种具有相同唯一类型的一组已经编号且长度固定的数据向序列。

##### 声明

```go
// 声明一个长度为5，int类型的数组
var arr1 [5]int
```

上面的数组声明后，会被初始化默认值```0```。

##### 数组的值

获取数组的值:

```go
arr[index] // index为数组下标，范围0到len(arr) - 1
```

数组是可变的，可以被赋予新的值：

```go
arr[i] = value
```

Go语言中的数组是一种值类型的，也可以通过```new```来创建数组。

```go
var arr1 = new([5]arr)
```

区别于：```var arr2 [5]int```的是：

```arr1```的类型是```*[5]int```，```arr2```的类型是```[5]int```。

当我们把数组赋值给另一个数组时，会做一次数组内存的拷贝:

```go
arr2 := *arr1
arr2[2] = 1
```

重新赋值后```arr2```不会改变`arr1`的值。

在函数传参中传入数组时`func1(arr2)`，实际上就产生了一次内存拷贝，函数`func1`不会改变原始`arr2`的值。

如果想要改变原始数组`arr2`，那么`arr2`必须通过操作符`&`来通过引用方式传入`func1(&arr2)`。

eg:

```go
package main

import "fmt"

func main() {
  var arr [3]int
  f(arr)
  fp(&arr)
}

func f(a [3]int) {
  fmt.Println(a) // [0 0 0]
}

func fp(a *[3]int) {
  fmt.Println(a) // &[0 0 0]
}
```

##### 多维数组

```go
[3][5]int
```

#### 切片

切片是一种*引用*类型的结构。和数组一样，切片是可以索引的，与数组不同，切片的长度是可变的，切片是一个长度可变的数组。

##### 声明切片

```go
var identifier []type
```

##### 初始化切片

```go
var slice1 []type = arr1[start:end]
// eg
s := []int{1,2,3}
```

获取切片长度使用`len(slice1)`，获取容量`cap(slice1)`

> 不能将指针指向`slice`，因为它本身就是一个引用类型（指针）。

##### 切片作为参数传递

```go
package main

import "fmt"

func main() {
  a := []int{1,2,3,4,5}
  s := sum(a)
  fmt.Printf("Sum is: %v", s)
}

func sum(a []int) int {
  s := 0
  for i := 0; i < len(a); i++ {
    s += a[i]
  }
  
  return s
}
```

##### 使用`make()`创建切片

```go
make([]int, len)

var slice1 []type = make([]type, len)
// 简写
slice1 := make([]type, len)
```

eg:

```go
package main

import "fmt"

func main() {
  var slice1 []int = make([]int, 10)
  for i := 0; i < len(slcie1); i++ {
    slice1[i] = 5 * i
  }
  
  for i := 0; i < len(slice1); i++ {
    fmt.Printf("Slice at %d is %d\n", i, slice1[i])
  }
}
```

##### make()和new()的区别

- `new(T)`为每个新的类型`T`分配一片内存，初始化为`0`，返回新类型为`*T`的内存地址，返回一个指向类型为`T`，值为`0`的地址的指针，适用于值类型为*数组*和*结构体*
- `make(T)`返回的是一个类型为`T`的初始值，适用于2种内建的引用类型：*切片*、`map`和`channel`

`new`用来*分配内存*，`make`用来*初始化*。

##### bytes包

`bytes`包类似于字符串。`[]bytes`类型的切片在`Go`语言中十分常见。

```go
b := make([]bytes, len)
```

`bytes`中的`Buffer`类型十分有用，如常见的字符串追加，我们通常会使用`+=`，有了`Buffer`后可以使用如下：

```go
import (
	"bytes"
  "fmt"
)

var buffer bytes.Buffer

for {
  if s, ok := getNextString(); ok {
    buffer.WriteString(s)
  } else {
    break
  }
}

fmt.Print(buffer.String(), "\n")
```

这种方法比`+=`更节省内存和`CPU`

##### 切片的复制和追加

有时我们需要增加切片的容量，此时我们需要做的就是创建一个新的更大的切片，然后将原切片拷贝出来。此时就需要用到切片的拷贝`copy`和追加`append`。

```go
package main

import "fmt"

func main() {
  slForm := []int{1,2,3}
  slTo := make([]int, 10)
  n := copy(slTo, slForm)
  fmt.Println(slTo) // [1 2 3 0 0 0 0 0 0 0]
  fmt.Printf("Copied %d elements\n", n) // Copied 3 elements
  
  sl3 := []int{1,2,3}
  sl3 = append(sl3, 4, 5, 6)
  fmt.Println(sl3) // [1 2 3 4 5 6]
}
```

##### 字符串、数组、切片

- 修改字符串中的某个字符

Go中的字符串是不可变的，如果想要修改字符串中的某个字符，必须先将字符串转换成字节数组，然后通过修改数组中的元素来达到修改字符串的目的。

```go
// 将hello修改为cello
s := "hello"
c := []byte(s)
c[0] = "c"
s2 := string(c) // cello
```

- 搜索、排序切片和数组

Go官方标准库```sort```提供了常见的搜索和排序操作。如：

>- `func Ints(a []int)`实现对`int`类型的切片排序：`sort.Ints(arri)`
>- `func Strings(s []string)`实现对字符串的排序：`sort.Strings(s)`
>- `func SearchInt(a []int, n int) int`实现搜索`int`类型的切片，返回的是索引值
>- `func SearchStrings(s []string, str string) int`实现对字符串类型切片的查找

还有其他的方法可查看官方文档。

##### 切片和垃圾回收

切片的底层是一个数组，数组的容量可能大于实际定义是的容量，只有当切片没有任何指向的时候，底层的数组才会被释放。

```go
a := make([]int, 32)
b := a[1:16]
a = append(a, 1) // a重新分配内存
a[2] = 42 // a,b不再共享内存，a变化不会影响到b
```

`append`函数在`cap`不够用的时候，会重新分配内存，扩大容量，如果够用就不会重新分配。

