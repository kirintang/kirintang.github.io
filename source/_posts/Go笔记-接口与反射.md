---
title: Go笔记-接口与反射
date: 2021-04-02 14:30:33
tags: go
---

Go语言中没有继承和类的概念，但是有接口，通过接口可以实现面向对象中的很多特性。接口提供了一组方式来说明其对象的行为。

<!-- more -->

#### 接口

##### 定义

接口定义了一组方法，这些方法不包含其具体的实现，也不包含变量。如下：

```go
type Foo interface {
  Method1(params) return_type
  Method2(params) return_type
  ......
}
```

按照接口约定，只包含一个方法的接口名由[e]r后缀组成，如：`Printer`、`Reader`，`Writer`等，一些不常用的方法以`able`或者`I`结尾，如：`Recoverable`。

##### 接口嵌套接口

一个接口可以包含另外的一个或者多个其它接口。如：

```go
type ReadWrite interface {
  Read(b Buffer) bool
  Write(b Buffer) bool
}

type Lock interface {
  Lock()
  Unlock()
}

type File interface {
  ReadWite
  Lock
  Close()
}
```

##### 类型断言

类型断言用来检测接口变量的类型和转换接口变量，通常一个接口类型的变量`varI`可以包含任何类型的值，需要有一种方法来检测它的类型。

```go
v := varI(T)
```

安全的方式：

```go
if _, ok := varI.(T); ok {
  // TODO
}
```

eg:

```go
package main

import (
	"fmt"
  "math"
)

type Square struct {
  side float32
}

type Circle struct {
  radius float32
}

type Shaper interface {
  Area() float32
}

func (s *Square) Area() float32 {
  return s.side * s.side
}

func (c *Circle) Area() float32 {
  return math.Pi * c.radius * c.radius
}

func main() {
  var areaIntf Shaper
  sq1 := new(Square)
  sq1.side = 5
  
  areaIntf = sq1
  
  if t, ok := areaIntf.(Square); ok {
    fmt.Printf("the type of areaIntf is: %T\n", t)
  }
}
```

##### 接口类型判断

接口类型同样可以使用`type-switch`来判断：

```go
switch areaIntf.(type) {
  case *Square:
  	//
  case *Circle:
  	//
  case nil:
  	//
	default:
  	//
}
```

eg:

```go
func classifier(items ...interface{}) {
  for i, x := range items {
    switch x.(type) {
      case bool:
      fmt.Println("bool")
    	case float32:
      fmt.Println("float32")
      ......
    }
  }
}
```

##### 测试一个值是否实现了某个接口

假定一个值`v`，判断它是否实现了`Stringer`接口：

```go
type Stringer interface {
  String() string
}

if _, ok := v.(Stringer); ok {
  // ok
}
```

