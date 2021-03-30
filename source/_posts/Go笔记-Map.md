---
title: Go笔记-Map
date: 2021-03-25 13:57:32
tags: go
---

Go语言中`Map`是一种元素对的集合`(key=>value)`，一个`key`对应的值为`value`，可以根据`key`快速的定位到`value`，也称之为关联数组或者字典。

<!-- more -->

#### 声明和初始化Map

Map是一种引用结构，声明如下：

```go
var map1 make[keytype]valuetype
// 如
var map1 make[string]int
```

声明时不需要指定`map`的长度，`map`会根据内容动态增长。未初始化的`map`值为`nil`

```go
package main

import "fmt"

func main() {
	var mapLit map[string]int
	var mapAssigned map[string]int

	mapLit = map[string]int{"one": 1, "two": 2}
	mapAssigned = mapLit
	mapAssigned["two"] = 3

	fmt.Printf("mapLit at \"two\" is: %d\n", mapLit["two"]) // 3
}
```

不要使用`new`来构造`map`，而是使用`make`。

##### Map的容量

Map会根据新增的`key-value`动态的进行伸缩，所以`Map`没有固定长度和最大限制，对于比较大的或者会快速扩张的`Map`，最好是能标明其容量。

##### 使用切片作为Map的值

`Map`中一个`key`对应一个`value`值，`value`是一个原始类型，如果我们需要一个`key`对应到多个`value`值时，此时我们就可以使用切片作为`key`的值来解决这个问题。

```go
map1 := make(map[int][]int)
map2 := make(map[int]*[]int)
```

#### 判断Map中键值对是否存在及删除元素操作

##### 判断键值对是否存在

```go
map1 := make(map[string]int)

map1["key1"] = 1
```

我们可以通过`map1["key1"]`来获取`map1`中`key1`的值，但如果`key1`不存在，`val1`就是一个值类型的空值。这种情况没有办法区分`key1`是否存在，还是说`val1`本来就是一个空值。

此时我们就需要判断`key1`是否存在`map1`中：

```go
val1, isPresent = map1["key1"]
```

`isPresent`是一个`bool`值，`true`说明存在，相反说明`map1`中没有`key1`。

```go
if _, ok := map1[key1]; ok {
  // TODO
}
```

##### 从Map中删除元素

使用`delete(map1, key1)`就可以删除`map1`中键为`key1`的元素。就算`key1`不存在，也不会报错。

```go
package main

import "fmt"

func main() {
	var mapLit map[string]int
	var mapAssigned map[string]int

	mapLit = map[string]int{"one": 1, "two": 2}
	mapAssigned = mapLit
	mapAssigned["two"] = 3

	fmt.Printf("mapLit at \"two\" is: %d\n", mapLit["two"]) // 3

	delete(mapLit, "one")

  fmt.Println(mapLit) // map[two:3]
}
```

##### Map排序

map默认是无序的，如果想要对map进行排序，需要将map的key或者value拷贝到一个切片中，再对切片食用`sort`进行排序。

```go
package main

import (
	"fmt"
  "sort"
)

var barVal = map[string]int{"alpha":35,"bravo":56,"charlie":23,"delta":87,"echo":56}

func main() {
  fmt.Println("before sort:")
  for k, v := range barVal {
    fmt.Printf("Key: %v, Value: %v /", k, v)
  }
  keys := make([]string, len(barVal))
  i := 0
  for k, _ := range barVal {
    keys[i] = k
    i++
  }
  
  sort.Strings(keys)
  fmt.Println()
  fmt.Println("after sort:")
  for _, k := range keys {
    fmt.Printf("Key: %v, Value: %v /", k, barVal[k])
  }
}
```

```
// out
before sort:
Key: echo, Value: 56 /Key: alpha, Value: 35 /Key: bravo, Value: 56 /Key: charlie, Value: 23 /Key: delta, Value: 87 /
after sort:
Key: alpha, Value: 35 /Key: bravo, Value: 56 /Key: charlie, Value: 23 /Key: delta, Value: 87 /Key: echo, Value: 56 /
```

