---
title: Go搭建Restful HTTP API服务
date: 2021-09-28 17:01:38
tags: go
---

### 一，初始化

首先，我们在`Github`创建一个仓库，命名为`learn-go-by-example`，克隆到本地:

```bash
$ git clone https://github.com/kirintang/learn-go-by-examples.git
$ cd learn-go-by-examples
```
我们在仓库目录下定义我们的`api`服务目录`go-rest-api`:

<!-- more -->


```bash
$ mkdir go-rest-api
$ cd go-rest-api
```

初始化`Go`模块依赖管理:

```bash
$ go mod init github.com/kirintang/learn-go-by-examples/go-rest-api
```
在开始构建我们的`api`服务之前，作为一个优秀的实践项目，我们将创建一个简单的代码组织。

创建代码目录结构：

```bash
.
|---- README.md
|---- bin
|---- doc
|---- go.mod
|---- internal
|---- pkg
       |---- swagger
```

我们`Go`的`net/http`库来构建请求, `internal/main.go`：

```go
package main

import (
    "fmt"
    "html"
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
    })

    log.Println("Listening on localhost:8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

这个简单的示例启动一个 HTTP 服务器，侦听端口 8080 传入请求，在 / 上提供服务并返回“Hello”+路径。


现在，是时候运行我们的应用程序来测试它了：


```bash
$ go run internal/main.go
2021/09/27 18:10:47 Listening on localhost:8080
```
为了测试我们的 HTTP 服务器，我们可以在 localhost:8080 上 curl 或在浏览器中转到此端点：

```bash
$ curl localhost:8080
Hello, "/"%
```


### 二，`Makerfile` -> `Tashfile`

过去我使用`Makefile`来定义一组任务，但现在我使用`Taskfile`，一个`Makefile`的替代品。这里有一篇文章关于[Taskfile的介绍](https://dev.to/stack-labs/introduction-to-taskfile-a-makefile-alternative-h92)。

基于我们的应用程序，我们创建如下`Taskfile.yml`：

```yml
version: "3"

tasks:
    build:
        desc: Build the app
        cmds:
            - GOFLAGS=-mod=mod go build -o bin/go-rest-api internal/main.go

run:
    desc: Run the app
    cmds:
        - GOFLAGS=-mod=mod go run internal/main.go

swagger.gen:
    desc: Generate Go code
    cmds:
        - GOFLAGS=-mod=mod go generate github.com/kirintang/learn-go-by-example/go-rest-api/internal github.com/kirintang/learn-go-by-example/go-rest-api/pkg/swagger

swagger.validate:
    desc: Validate swagger
    cmds:
        - swagger validate pkg/swagger/swagger.yml

swagger.doc:
    desc: Doc for swagger
    cmds:
        - docker run -i yousan/swagger-yaml-to-html < pkg/swagger/swagger.yml > doc/index.html
```

安装`go-task`:

```bash
$ brew install go-task/tap/go-task
```

在我们的目录下执行`task --list`可以显示可用的任务列表：

```bash
$ task --list
task: Available tasks for this project:
* build: Build the app
* run: Run the app
* swagger.doc: Doc for swagger
* swagger.gen: Generate Go code
* swagger.validate: Validate swagger
```

### 三，`Swagger`

`Swagger`助力我们提供符合`OpenAPI`规范的标准化文档。借助`Swagger`应用程序，我们可以生成代码，向外部提供`HTML`格式的`API`文档。

#### `Swagger`安装

这里我们安装`go-swagger`工具，可参考[官方文档](https://github.com/go-swagger/go-swagger/blob/master/docs/install.md)。

```bash
$ brew tap go-swagger/go-swagger
$ brew install go-swagger
```

新建我们的`swagger.yml`规范文件，目录`pkg/swagger/swagger.yml`。

```yml
consumes:

- application/json

info:
    description: HTTP Server in Go with Swagger endpoints definition.
    title: go-rest-api
    version: 0.1.0

produces:
    - application/json

schemes:
    - http
  
swagger: "2.0"

paths:
    /healthz:
        get:
            operationId: checkHealth
            produces:
                - text/plain
            responses:
                "200":
                    description: OK message.
                    schema:
                        type: string
                        enum:
                            - OK

    /hello/{user}:
        get:
            description: Returns a greeting to the user!
            parameters:
                - name: user
                  in: path
                  type: string
                  required: true
                  description: The name of the user to greet.
            responses:
                200:
                    description: Returns the greeting.
                        schema:
                            type: string
                400:
                    description: Invalid characters in "user" were provided.
```

每次修改`swagger.yml`文件后，最好检查一下文件的有效性：


```bash
$ tash swagger.validate
task: [swagger.validate] swagger validate pkg/swagger/swagger.yml
2021/09/28 14:20:00
The swagger spec at "pkg/swagger/swagger.yml" is valid against swagger specification 2.0
```

校验成功后，我们将`swagger`定义生成可视化的`html`页面。

```bash
$ task swagger.doc
task: [swagger.doc] docker run -i yousan/swagger-yaml-to-html < pkg/swagger/swagger.yml > doc/index.html
```

此时，我们的`doc`目录下生成了一个`index.html`文件，打开文件后，可以看到一个`swagger`的可视化界面。


![](https://raw.githubusercontent.com/kirintang/imgLib/master/20210928150354.png)

在`pkg/swagger`下创建`gen.go`文件：

```go
package swagger

//go:generate rm -rf server
//go:generate mkdir -p server
//go:generate swagger generate server --quiet --target server --name hello-api --spec swagger.yml --exculde-main
```

我们根据`Swagger`规范生成`Go`文件:

```bash
$ tash swagger.gen
task: [swagger.gen] GOFLAGS=-mod=mod go generate github.com/kirintang/learn-go-by-example/go-rest-api/internal github.com/kirintang/learn-go-by-example/go-rest-api/pkg/swagger

```

运行后将在`pkg/swagger`下生成如下目录文件:

![](https://raw.githubusercontent.com/kirintang/imgLib/master/20210928150641.png)

这让我们构建`api`服务节省了很多时间。

### 四，正式开始编写我们的服务文件

修改`main.go`文件:

```go
package main


import (

"log"

"github.com/go-openapi/loads"

"github.com/go-openapi/runtime/middleware"

"github.com/kirintang/learn-go-by-examples/go-rest-api/pkg/swagger/server/restapi"

"github.com/kirintang/learn-go-by-examples/go-rest-api/pkg/swagger/server/restapi/operations"

)


func main() {
    // Initialize Swagger
    swaggerSpec, err := loads.Analyzed(restapi.SwaggerJSON, "")
    if err != nil {
        log.Fatalln(err)
    }

    api := operations.NewHelloAPIAPI(swaggerSpec)
    server := restapi.NewServer(api)
    defer func() {
        if err := server.Shutdown(); err != nil {
            log.Fatalln(err)
        }
    }()

    server.Port = 8080

    api.CheckHealthHandler = operations.CheckHealthHandlerFunc(Health)

    api.GetHelloUserHandler = operations.GetHelloUserHandlerFunc(GetHelloUser)
    
    if err := server.Serve(); err != nil {
        log.Fatalln(err)
    }

}

// Health route returns OK
func Health(operations.CheckHealthParams) middleware.Responder {
    return operations.NewCheckHealthOK().WithPayload("OK")
}

// GetHelloUser returns Hello + your name
func GetHelloUser(user operations.GetHelloUserParams) middleware.Responder {
    return operations.NewGetHelloUserOK().WithPayload("Hello " + user.User + "!")
}
```

这里我们定义了`2`个接口：

- 程序健康检查

- 获取`HelloUserHandler`


然后编译我们我们的程序：

```bash
$ go build -o bin/go-rest-api internal/main.go
```
当然，我们也可以使用`task`来编译：

```bash
$ tash build
task: [build] GOFLAGS=-mod=mod go build -o bin/go-rest-api internal/main.go
```

运行：

```bash
$ ./bin/go-rest-api
2021/09/28 15:20:27 Serving hello API at http://[::]:8080
```

`酷不酷！So Cool!!!`

另外，对以不同的操作系统，我们需要对应修改我们`Taskfile`编译命令。

- 对于`Windows`用户

```bash

# Windows 32 bits
$ GOOS=windows GOARCH=386 go build -o bin/go-rest-api-win-386 internal/main.go

# Windows 64 bits
$ GOOS=windows GOARCH=amd64 go build -o bin/go-rest-api-win-64 internal/main.go

```

- 对于`Linux`用户

```bash

# Linux 32 bits
$ GOOS=linux GOARCH=386 go build -o bin/go-rest-api-linux-386 internal/main.go

# Linux 64 bits
$ GOOS=linux GOARCH=amd64 go build -o bin/go-rest-api-linux-64 internal/main.go

```

- `MacOS`

```bash
# MacOS 32 bits
$ GOOS=darwin GOARCH=386 go build -o bin/go-rest-api-darwin-386 internal/main.go

# MacOS 64 bits
$ GOOS=darwin GOARCH=amd64 go build -o bin/go-rest-api-darwin-64 internal/main.go

# MacOS 64 bits for M1 chip
$ GOOS=darwin GOARCH=arm64 go build -o bin/go-rest-api-darwin-arm64 internal/main.go

```

接下来测试一下我们的程序：


```bash
$ curl http://localhost:8080
{"code":404,"message":"path / was not found"}

$ curl http://localhost:8080/healthz
OK

$ curl http://localhost:8080/hello/kirintang
"Hello kirintang!"
```