---
title: Go搭建CLI应用程序
date: 2021-09-30 10:16:10
tags: Go
---

### 一，初始化

新建项目目录`learn-go-by-examples/go-cli`，初始化模块依赖`go mod init github.com/kirintang/learn-go-by-example/go-cli`.

<!-- more -->

代码目录结构：

```bash
.
├── README.md
├── bin
├── go.mod
└── go.sum
```

### 二，`Cobra`
[Cobra](https://github.com/spf13/cobra) 是一个用于构建强大的现代化的`CLI`应用程序库。

##### 安装

```bash
$ go get -u github.com/spf13/cobra/cobra
```
##### 初始化
使用`cobra`的初始化命令生成我们的`CLI`程序，该命令将生成具有正确文件结构和导入的应用程序：
```bash
$ cobra init --pkg-name github.com/kirintang/learn-go-by-examples/go-cli
```
初始化完成后我们的目录结构就变成如下结构：
```bash
.
├── LICENSE
├── README.md
├── bin
├── cmd
│   └── root.go
├── go.mod
├── go.sum
└── main.go
```
通常在构建一个`CLI`程序时，我们希望有以下提示能帮助使用者更好的理解如何使用它：
- 简短描述
- 完整描述
- 如何使用我们的程序
接下来我们修改我们的`cmd/root.go`文件：

```go
var rootCmd = &cobra.Command{
    Use: "go-cli",
    Short: "golang cli",
    Long: `cli application write by golang.`,
    // Uncomment the following line if your bare application
    // has an action associated with it:
    // Run: func(cmd *cobra.Command, args []string) { },
}

```
现在，我们要在`CLI`应用程序中添加一个`get`命令。 为此，我们将使用`cobra CLI`的`cobra add`命令：
```bash
$ cobra add get
get created at /Users/kirintang/develop/learn-go-by-examples/go-cli
```
此命令将会在`cmd`下新建一个`get.go`文件：

```bash
.
├── LICENSE
├── README.md
├── bin
├── cmd
│   ├── get.go
│   └── root.go
├── go.mod
├── go.sum
└── main.go
```
尝试运行我们的程序:
```bash
$ go run main.go
```
得到如下信息：
```bash
cli application write by golang.

Usage:
    go-cli [command]

Available Commands:
    completion generate the autocompletion script for the specified shell
    get A brief description of your command
    help Help about any command

Flags:
        --config string config file (default is $HOME/.go-cli.yaml)
    -h, --help help for go-cli
    -t, --toggle Help message for toggle

Use "go-cli [command] --help" for more information about a command.
```
运行我们`get`命令:
```bash
$ go run main.go get
get called
```
得到了`get called`响应信息。
接下来完善我们的`get`命令.
```go
var getCmd = &cobra.Command{
    Use: "get",
    Short: "get the name.",
    Long: `get name by input.`,
    
    Run: func(cmd *cobra.Command, args []string) {
        var inputName = "kirintang"
        if len(args) >= 1 && args[0] != "" {
            inputName = args[0]
        }
    fmt.Println("Hello " + inputName + ", Welcome to gopher world!")
    },
}

```

上述例子要求我们通过命令行传入的参数作为变量输出，运行下：

```bash
$ go run main.go get
Hello kirintang, Welcome to gopher world!

$ go run main.go get Bob
Hello Bob, Welcome to gopher world!

```
### 使用`Taskfile`构建
`Taskfile.yml`:
```yml

version: "3"

tasks:
    build:
        desc: Build the app
        cmds:
            - GOFLAGS=-mod=mod go build -o bin/go-cli main.go
    run:
        desc: Run the app
        cmds:
            - GOFLAGS=-mod=mod go run main.go
```

之前我们使用过`task`命令来构建运行我们的应用程序，相同的：

```bash
$ task --list
task: Available tasks for this project:
* build: Build the app
* run: Run the app

$ task build
task: [build] GOFLAGS=-mod=mod go build -o bin/go-cli main.go
```

构建完成的二进制文件可直接运行：
```bash
$ ./bin/go-cli get Bob
Hello Bob, Welcome to gopher world!
```
我们依然可以通过以上介绍，创建我们想要的任何`CLI`应用程序。