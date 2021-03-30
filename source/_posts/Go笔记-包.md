---
title: Go笔记-包
date: 2021-03-29 15:16:18
tags: go
---

Go语言中的内置包(package)多达150多个，像`fmt`、`os`等，go语言的内置包称之为标准库，对应的列表可以查询[这里](https://gowalker.org/search?q=gorepos)。

<!-- more -->

#### regexp包

regexp用来进行正则表达式匹配的标准库。

```go
ok, _ := regexp.Match(pat, []byte(searchIn))
```

`ok`返回`boolean`值`true/false`。

```go
package main

import (
	"fmt"
  "regexp"
)

func main() {
  //目标字符串
	searchIn := "John: 2578.34 William: 4567.23 Steve: 5632.18"
	pat := "[0-9]+.[0-9]+" //正则

	f := func(s string) string{
    	v, _ := strconv.ParseFloat(s, 32)
    	return strconv.FormatFloat(v * 2, 'f', 2, 32)
	}

	if ok, _ := regexp.Match(pat, []byte(searchIn)); ok {
    fmt.Println("匹配成功!")
	}

	re, _ := regexp.Compile(pat)
	//将匹配到的部分替换为"##.#"
	str := re.ReplaceAllString(searchIn, "##.#")
	fmt.Println(str) // John: ##.# William: ##.# Steve: ##.#
	//参数为函数时
	str2 := re.ReplaceAllStringFunc(searchIn, f)
	fmt.Println(str2) // John: 5156.68 William: 9134.46 Steve: 11264.36
}
```

#### 锁和sync

Go中锁是通过`sync`包中的`Mutex`来实现的。

`sync.Mutex`是一个互斥锁，它可以保证在临界区入口同一个时间只能有一个线程进入临界区，线程将有序的对同一变量进行访问。

eg:

```go
import "sync"

type Info struct {
  mu sync.Mutex
  Str string
}

func Update(info *Info) {
  info.mu.Lock()
  info.Str = "new string"
  info.mu.Unlock()
}
```

有时我们允许在同一时间有多个线程对变量进行访问，但是又不允许同时对其修改，我们需要使用`sync`包中的`RWMutex`锁可以通过`Rlock()`来允许同一时间多个线程对变量进行读操作，但是同一时间只能有一个线程进行写操作。

