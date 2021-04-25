---
title: Go笔记-读写数据
date: 2021-04-16 16:53:20
tags: go
---

读写数据是编程中经常用到的操作，比如命令行读取用户输入、文件读取等。

<!-- more -->

#### 读取用户输入

- 读取命令行输入

```go
package main

import "fmt"

var firstName, lastName string

func main() {
	fmt.Println("Please input your name:")
	fmt.Scan(&firstName, &lastName)
	fmt.Printf("Hi, %s %s", firstName, lastName)
}
```

使用标准库`fmt`的`Scan`实现用户输入。

也可以使用`bufio`提供的缓冲读取数据：

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

var inputReader *bufio.Reader

func main() {
	inputReader = bufio.NewReader(os.Stdin)
	fmt.Println("Please input something:")
  input, err := inputReader.ReadString('\n')
	if err == nil {
		fmt.Printf("The input is: %s\n", input)
	}
}
```

#### 文件读写

文件句柄使用标准库`os.File`

- 读文件

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	inputFile, inputError := os.Open("./test.txt")
	if inputError != nil {
		return
	}
	defer inputFile.Close()

	inputReader := bufio.NewReader(inputFile)
	for {
		inputString, readerErr := inputReader.ReadString('\n')
		fmt.Printf("The input was: %s", inputString)
		if readerErr == io.EOF {
			return
		}
	}
}
```

- 写文件

`buffo`的`Writer`实现文件写入：

```go
package main

import (
	"bufio"
	"os"
)

func main() {
	outputFile, outputError := os.OpenFile("output.txt", os.O_WRONLY|os.O_CREATE, 0666) // O_WONLY只写 O_CREATE创建
	if outputError != nil {
		return
	}
	defer outputFile.Close()

	outputWriter := bufio.NewWriter(outputFile) // 创建缓冲区
	outputString := "hello world!\n"

	for i := 0; i < 10; i++ {
		outputWriter.WriteString(outputString) // 写入缓冲区
	}
	outputWriter.Flush() // 从缓冲区写入文件
}
```

如果写的东西很简单，也可以使用`fmt`的`Fprintf`直接将内容写入文件，`fmt`里面`F`开头的`Print`函数可以直接写入任何`io.Writer`。

#### 文件拷贝

使用`io`包可以将一个文件拷贝到另一个文件。

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	CopyFile("./target.txt", "./source.txt")
	fmt.Printf("Copy done!")
}

func CopyFile(dstName, srcName string) (written int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		fmt.Errorf("open src file err")
		return
	}
	defer src.Close()

	dst, err := os.Create(dstName)
	if err != nil {
		fmt.Errorf("open dst file err")
		return
	}
	defer dst.Close()

	return io.Copy(dst, src)
}
```

#### 从命令行读取参数

- os包

`os`包中`string`类型的切片变量`os.Args`用来处理一些基本的命令行参数，可以在程序启动后读取命令行输入的参数：

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	who := "Alice"
	if len(os.Args) > 1 {
		who += strings.Join(os.Args[1:], " ")
	}
	fmt.Println("Good Moring", who)
}
```

还有其他的如`flag`包，现代化命令行`cobra`包。