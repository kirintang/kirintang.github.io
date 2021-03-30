---
title: Go笔记-函数
date: 2021-03-14 12:53:12
tags: go
---

Go是编译型语言，所以函数编写的顺序是无关紧要的。编写多个函数的主要目的是将一个需要很多行代码的复杂问题分解为一系列简单的函数来解决，有助于代码复用。

<!-- more -->

Go里有三种类型的函数：

- 带有参数的普通函数
- 匿名函数
- 方法

除了```main()```、```init()```函数外，其它所有类型的函数都可以有参数与返回值。函数参数、返回值以及它们的类型被统称为函数签名。

函数被调用格式：

```go
包.函数名(参数1, 参数2, ...)
```

函数是一等公民，它们可以赋值给变量，```add := binOp```。

#### 函数参数与返回值

Go函数可以传递多个参数，并且可以有多个返回值，这是相对于C、C++、Java等其他语言的一大特性。

任何一个有返回值的函数，都必须以```return```或者```panic```结束。在函数块里，```return```之后的语句都不会被执行，如果一个函数需要返回值，这个函数里每一个分支都要有返回值。如下面这个函数不会被编译：

```go
func (st *Stack) Pop() int {
    v := 0
    for ix := len(st) - 1; ix >= 0; ix-- {
        if v = st[ix]; v != 0 {
            st[ix] = 0
            return v
        }
    }
}
```

正确的写法:

```go
func (st *Stack) Pop() int {
    v := 0
    for ix := len(st) - 1; ix >= 0; ix-- {
        if v = st[ix]; v != 0 {
            st[ix] = 0
            return v
        }
    }
  	return v
}
```

##### 函数传参

- 函数的参数传递有两种：按值传递、按引用传递。

  - Go默认使用按值传递来传递参数，函数在接收参数后被修改，不会影响原来参数的值。

    ```go
    Function(arg type){}
    ```

  - 如果希望函数可以直接修改参数的值，就要使用按引用传递来进行传参，此时传递给函数的是一个指针。

    ```go
    Function(a *A){}
    ```

    

##### 返回值

当函数需要返回多个非命名返回值时，需要使用```()```把他们括起来，如```(int, int)```

```go
func getX2AndX3(input int) (int, int) {
    return 2 * input, 3 * input
}
```

当函数需要返回命名返回值时，即使是一个返回值也需要括起来

```go
func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    return
}
```

##### 空白符

空白符```_```用来匹配一些使用不到的返回值，然后丢弃掉。

```go
package main

import "fmt"

func main() {
    var i1 int
    var f1 float32
    i1, _, f1 = ThreeValues()
    fmt.Printf("The int: %d, the float: %f \n", i1, f1)
}

func ThreeValues() (int, int, float32) {
    return 5, 6, 7.5
}
```

##### 改变外部变量

使用按值传递，不但可以节省内存，还可以改变外部变量的值，修改的值不需要再```return```回来。

```go
package main

import (
    "fmt"
)

func Multiply(a, b int, reply *int) {
    *reply = a * b
}

func main() {
    n := 0
    reply := &n
    Multiply(10, 5, reply)
    fmt.Println("Multiply:", *reply)
}
```

#### 传递变长参数

函数传参如```Function(...type)```的形式传递，说明函数处理的是一个变长的参数。

```go
func myFunc(a, b, c ...int) {}
```

```go
package main

import "fmt"

func main() {
	x := min(1, 3, 2, 0)
	fmt.Printf("The minimum is: %d\n", x)
	slice := []int{7,9,3,5,1}
	x = min(slice...)
	fmt.Printf("The minimum in the slice is: %d", x)
}

func min(s ...int) int {
	if len(s)==0 {
		return 0
	}
	min := s[0]
	for _, v := range s {
		if v < min {
			min = v
		}
	}
	return min
}
```

#### defer和追踪

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 `return` 语句之后）一刻才执行某个语句或函数

类似于其他语言如`Java`的`finally`

```go
package main
import "fmt"

func main() {
	function1()
}

func function1() {
	fmt.Printf("In function1 at the top\n")
	defer function2()
	fmt.Printf("In function1 at the bottom!\n")
}

func function2() {
	fmt.Printf("Function2: Deferred until the end of the calling function!")
}
```

输出

```
In function1 at the top
In function1 at the bottom!
Function2: Deferred until the end of the calling function!
```

defer常见的一些操作：

- 关闭文件流

```go
defer file.Close()
```

- 解锁一个加锁的资源

```go
mu.Lock()  
defer mu.Unlock() 
```

- 关闭数据库

```go
defer disconnectFromDB()
```

#### 闭包

不希望1给函数起名，可使用闭包:

```go
func(x, y int) int { return x + y }
```

匿名函数不能独立存在：

```go
fplus := func(x, y int) int { return x + y }
```

通过变量名对函数进行调用

```go
fplus(3,4)
```

也可以直接对匿名函数进行调用：

```go
func(x, y int) int { 
  return x + y 
}(3, 4)
```

#### 计算函数执行时间

```go
start := time.Now()
function()
end := time.Now()
delta := end.Sub(start)
fmt.Printf("function took this amount of time: %s\n", delta)
```

#### 通过内存缓存来提升性能

通过在内存中缓存和重复利用相同计算的结果，称之为内存缓存。

```go
package main

import (
	"fmt"
	"time"
)

const LIM = 41

var fibs [LIM]uint64

func main() {
	var result uint64 = 0
	start := time.Now()
	for i := 0; i < LIM; i++ {
		result = fibonacci(i)
		fmt.Printf("fibonacci(%d) is: %d\n", i, result)
	}
	end := time.Now()
	delta := end.Sub(start)
	fmt.Printf("func took this amount of time: %s\n", delta)
}
func fibonacci(n int) (res uint64) {
	// 见哈是否有缓存值
	if fibs[n] != 0 {
		res = fibs[n]
		return
	}
	if n <= 1 {
		res = 1
	} else {
		res = fibonacci(n-1) + fibonacci(n-2)
	}
	fibs[n] = res // 缓存值
	return
}
```

