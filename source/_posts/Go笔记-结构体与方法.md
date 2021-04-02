---
title: Go笔记-结构体与方法
date: 2021-03-30 15:29:33
tags: go
---

Go通过类型别名和结构体来支持用户自定义类型，需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。然后可以访问这些数据。有点像`php`、`java`中的`class`。

<!-- more -->

#### 结构体

##### 定义

组成结构体类型的数据称之为字段，每个字段都有一个字段名和类型，且字段名是唯一的。

定义如下：

```go
type identifier struct {
  field1 type1
  field2 type2
}
```

eg:

```go
type User struct {
  Name string
  Age int
}
```

结构体的字段可以使任何类型，甚至结构体本身，也可以是*函数*或者*接口*。

##### 使用new给一个新的结构体分配内存

```go
import "fmt"

type struct1 struct {
  i1 int
  i2 float32
  str string
}

func main() {
  ms := new(struct1)
  ms.i1 = 5
  ms.i2 = 10.0
  ms.str = "hello"
  
  fmt.Printf("i1 is: %d\n", ms.i1)
  fmt.Printf("i2 is: %f\n", ms.i2)
  fmt.Printf("str is: %ds\n", ms.str)
  fmt.Println(ms)
}
```

```
i1 is: 5
i2 is: 10.000000
str is: hello
&{5, 10.0, hello}
```

##### 工厂方法创建结构体实例

Go使用构造子工厂的方法实现构造函数。

```go
type File struct {
  fd int
  name string
}


func NewFile(fd int, name string) *File {
  if fd < 0 {
    return nil
  }
  return &File{fd, name}
}
```

调用

```go
f := NewFile(10, "./test.txt")
```

##### 强制使用工厂方法

```go
type matrix struct {
  ....
}

func NewMatrix(params) * matrix{
  m := new(matrix)
  return m
}
```

使用

```go
func main() {
  r := matrix.NewMatrix(params)
}
```

##### 匿名字段和内嵌结构体

结构体可以包含一个或者多个匿名字段，匿名字段没有显示名称，但是必须有字段类型，字段类型就是字段名称。匿名字段同时可以是一个结构体，当一个结构体中包含的一个匿名字段是一个结构体时相当于面向对象中的继承行为，在Go中是通过组合来实现的。

```go
package main

import "fmt"

type innerS struct {
  in1 string
  in2 int
}

type outerS struct {
  a int
  b string
  int
  innerS // 组合结构体
}

func main() {
  outers := new(outerS)
  outers.a = 1
  outers.b = "hello"
  outers.int = 2
  outers.in1 = "world"
  outers.in2 = 3
  
  // 使用结构体字面量
  outers2 := outerS{4, "kirintang", 5, innerS{"github", 6}}
}
```

> 组合结构体会出现一个问题，字段命名冲突，此时需要自行修改保证字段名不能一致。

#### 方法

##### 定义

方法的定义一般格式为：

```go
// recv方法的1接收者
func (recv receiver_type) methodName(parameter_list) (return_value_list) {
  ......
}
```

如果方法不需要使用`recv`的值，可以省略`_`：`receiver_type`必须和*方法名*在同样的包中声明。

```go
func (_ receiver_type) methodName(parameter_list) (return_value_list) {
  ......
}
```

`recv`相当于面向对象语言中的`this`或者`self`，`Go`中没有这两个关键字，所以可以写成`this`或者`self`来代替：

```go
func (this receiver_type) methodName(parameter_list) (return_value_list) {
  ......
}
// 或者
func (self receiver_type) methodName(parameter_list) (return_value_list) {
  ......
}
```

eg:

```go
import "fmt"

type TwoInt struct {
	a, b int
}

func main() {
	two1 := new(TwoInt)
	two1.a = 1
	two1.b = 2

	fmt.Printf("sum is: %d\n", two1.AddThem()) // 11
	fmt.Printf("sum of param: %d\n", two1.AddToParam(10)) // 21
}

func (tn *TwoInt) AddThem() int {
  tn.b = 10
	return tn.a + tn.b
}

func (tn *TwoInt) AddToParam(param int) int {
	return tn.a + tn.b + param
}
```

##### 函数和方法的区别

函数将变量作为参数：`Function1(recv)`

方法在变量上被调用：`recv.Method1()`

##### 内嵌类型的方法和继承

当一个匿名类型被嵌套在结构体中时，匿名类型的可见方法也会被嵌套进去，相当于外层类型继承了这些可见方法：子类型继承父类型的公共方法 (有面向对象内味儿了)。

一个`Engine`的接口类型，一个`Car`的结构类型包含`Engine`类型的匿名字段：

```go
type Engine interface {
  Start()
  Stop()
}

type Car struct {
  Engine
}

func (c *Car) GotoWork() {
  c.Start()
  c.Stop()
}
```

