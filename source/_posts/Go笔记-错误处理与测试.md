---
title: Go笔记-错误处理与测试
date: 2021-04-26 10:22:13
tags: go
---

像`java`，`php`等其他语言使用`try-and-catch`来捕捉异常，`go`有自己的一套异常机制`defer-panic-recover`。`go`的异常捕获机制更轻量，只作为处理错误的最后手段。

<!-- more -->

#### 错误处理

Go中有一个预定义的`error`接口类型，错误值标识错误状态：

```go
type error interface {
  Error() string
}
```

##### 定义错误

当我们需要新定义一个错误类型时，都可以使用`error`包中的`error.New`函数接收错误信息来创建，如：

```go
err := error.New("error infos")
```

