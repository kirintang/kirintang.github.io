---
title: Go笔记-基础结构和数据类型
date: 2021-03-13 10:12:56
tags: go
---



Go语言中基本的结构和数据类型：关键字与标识符、```Go```程序的基本结构和要素、常量、变量、```strings```和```strconv```包、时间和日期、指针。

<!-- more -->

#### 一，关键字和标识符

Go语言区分大小写，有效的标识符必须以字母开头，后面紧跟0或多个字符或数字。

Go语言中使用到的关键字和保留字。

| break    | default     | func   | interface | select |
| -------- | ----------- | ------ | --------- | ------ |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

#### 二，```Go```程序的基本结构和要素

- 包

使用关键字```import```导入第三方或者官方标准库。

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello World")
}
```

当一个包中的标识符(包括常量、变量、函数名、类型、结构体字段)以大写字母开头，这对外是可见的，否则外部导入不可使用。

- 函数

> 函数的定义:

```go
func 函数名() {}
func 函数名(参数 类型,参数 类型) 返回值/类型 {}
```

括号中可以传递一个或者多个参数，参数后必须有参数类型。

> 实例

```go
// 相当于private
func sum(a, b int) int {
  return a + b
}

// 相当于public
func Add(a, b int) int() {
  return a + b
}
```

- 注释

```go
package main

import "fmt" // Package implementing fromatted I/O

/*
多行注释
*/
func main() {
  fmt.Println("Hello World!") // 单行注释
}
```

- 类型

基本类型：```int```、```float```、```bool```、```string```

复合类型：```struct```、```array```、```slice```、```map```、```channel```

- 类型转换

go语言中一个类型的值可以被转换成另一种类型的值，go语言没有隐式转换、所有的转换必须是显示说明。

```
valueTypeB = typeB(valueTypeA)
```

#### 三，常量

go语言中常量使用```const```关键字定义，用于存储不会变化的数据。

存储在1常量中的数据类型只能是：布尔类型、数字型、浮点型。

```go
const Pi = 3.1415926
const beef, two, c = "drink", 2, "c"
const (
	Unknown = 0
  Female = 1
  Male = 2
)
```

#### 四，变量

声明变量的一般形式是使用```var```关键字：```var identifier type```

```go
var a int
var b string
var c, d int
```

变量声明后未赋值时，系统会自动赋予该变量类型的值：``` int```为```0```、```string```为空字符串、```float```为```0.0```、指针为```nil```

变量命名规则遵循驼峰，首个字符单词小写、每个新单词首字母大写：```getNum```

变量可以不指定类型，系统会根据变量的值自动推断出其类型。

- 简短形式

常量赋值可以不用```var```关键字，使用```:=```操作符。

```go
a := 5
```

#### 五，strings和strconv包

- 前缀和后缀

HasPrefix判断字符串```s```是否以```prefix```开头：

```go
string.HashPrefix(s, prefix string) bool
```

​	  HasSuffix判断字符串```s```是否以```suffix```结尾:

```go
string.HasSuffix(s, suffix string) bool
```

- 字符串包含

  Contains判断字符串```s```是否包含```substr```:

```go
strings.Contains(s, substr string) bool
```

- 判断子字符串或者字符在父字符串中出现的位置

Index返回字符串```str```在字符串```s```中的索引，```-1```表示字符串```s```不包含字符串```str```:

```go
strings.Index(s, str string) int
```

- 字符串替换

Replace用于将字符串```str```中的前```n```个字符串```old```替换为字符串```new```，并返回新的字符串，如果```n=-1```折替换所有字符串```old```为```new```:

  ```go
  strings.Replace(str, old, new string, n int) string
  ```

- 统计字符串出现的次数

Count用于统计字符串```str```在字符串```s```中出现的次数：

```go
strings.Count(s, str string) int
```

- 重复字符串

Repeat用于重复```count```词字符串```s```并返回一个新的字符串

```go
strings.Repeat(s string, count int) string
```

- 修改字符串大小写

```go
strings.ToLower(s) string // 小写
strings.ToUpper(s) string // 大写
```

- 剔除字符串

```go
strings.TrimSpace(s) // 剔除空白符
strings.TrimLeft(s)
strings.TrimRight(s)
strings.Trim(s, "cunt") // 剔除其他的字符串
```

- 分割字符串

```go
strings.Fields(s) // 利用一个或多个空白1符号来作为动态长度的分隔符将字符串分割成若干小块，返回切片
strings.Split(s, sep) // 用于自定义分隔符号来对字符串进行分割，返回切片
```

- 拼接```slice```到字符串

Join用于将元素类型为```string```的```slice```使用分割富豪来进行拼接成一个字符串

```go
strings.Join(s1 []string, sep string)
```

- 从字符串中读取内容

函数```strings.Reader(str)```用于生成一个```Reader```并读取字符串中的内容，然后返回指向该```Reader```的指针:

  ```go
strings.Reader(str)
strings.Read(str) // 从[]byte中读取内容
strings.ReadByte(str) // 从字符串中读取下一个byte或者rune
strings.ReadRune() 同上
  ```

#### 六，时间和日期

时间和日期标准库```time```

```go
package main

import(
	"fmt"
  "time"
)

func main() {
  t := time.Now()
  t := time.Now.UTC()
  week := 60 * 60 * 24 * 7 * 1e9
  week_from_now := t.Add(time.Duration(week))
}
```

#### 七，指针

Go语言取址符为```&```，变量前使用```&```可以返回相应变量的内存地址。

```go
var i1 = 5
fmt.Printf("An integer: %d, it's location in memory: %p\n", i1, &i1)
```

定一个指针，指向上面的```i1```

```go
package main

import "fmt"

func main() {
  var i1 = 5
  var intP *int
  intP = &i1
  fmt.Println("The Value of intP is", *intP) // 5
  
  s := "good bye"
  var p *string = &s
  *p = "ciao"
  fmt.Printf("Here is the pointer p: %p\n", p) // Here is the pointer p: 0xc00010a220
  fmt.Printf("Here is the string *p %s\n", *p) // Here is the string *p ciao
  fmt.Printf("Here is the string s: %s\n", s) // Here is the string s: ciao
}
```

