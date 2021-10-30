---
title: "Docker部署Go Web应用"
date: 2021-10-29T21:33:51+08:00
categories: ["Go","Docker"]
tags: ["Docker","Go"]
# description: "Desc Text."

# weight: 1

TocOpen: false
draft: false

hideSummary: false
hidemeta: false

disableHLJS: true # to disable highlightjs
disableShare: false

comments: true
canonicalURL: "https://www.niuwx.cn/"
cover:
    image: "/docker_go.jpg"
---

本文介绍了如何在Docker中部署Go Web 应用，包含了镜像构建、分段构建。

<!--more-->

### 简单示例

#### 应用代码
以这段简单的go web代码为例进行介绍。

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HandleHello)
	server := &http.Server{
		Addr: ":9090",
	}
  fmt.Println("Server startup...")
	if err := server.ListenAndServe(); err != nil {
		fmt.Printf("Server startup failed, err:%v\n", err)
	}
}

func HandleHello(w http.ResponseWriter, _ *http.Request) {
	w.Write([]byte("Hello World"))
}
```

#### 编写Dockerfile

```dockerfile
FROM golang:alpine

# 设置镜像环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 将本目录下代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件app
RUN go build -o app .

# 声明服务端口
EXPOSE 9090

# 启动容器时运行的命令
CMD ["/app"]
```

#### 构建镜像

使用命令构建镜像。

```go
docker build . -t goweb
```

#### 使用镜像

```
docker run -p 9090:9090 --name goweb-app goweb
```

使用`-p`来映射端口，这里容器中的应用需要在9090端口上运行，将其映射到主机的9090端口。也可以将其映射到其他端口，例如`-p 8080:9090`。

### 分段构建

在编译Go程序之后，我们得到了一个可执行的二进制文件，在最终的镜像中我们是不需要go编译器的，只需要一个可以运行二进制文件的容器即可。所以可以通过分段构建，第一步编译出二进制可执行文件，第二步将该可执行文件放进可以运行的环境即可。

```dockerfile
FROM golang:alpine AS builder

# 设置镜像环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 将本目录下代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件app
RUN go build -o app .

###################
# 最终的小镜像
###################
FROM scratch

# 从builder镜像中把/app 拷贝到当前目录
COPY --from=builder /build/app /

# 需要运行的命令
ENTRYPOINT ["/app"]
```

通过分段构建，我们就得到了一个体积非常小的镜像。

#### 静态资源的拷贝

如果需要部署的程序还需要用到静态资源，那么还需要将静态资源拷贝到镜像中。

```dockerfile
FROM golang:alpine AS builder

# 设置镜像环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# 移动到工作目录：/build
WORKDIR /build

# 将本目录下代码复制到容器中
COPY . .

# 将我们的代码编译成二进制可执行文件app
RUN go build -o app .

###################
# 最终的小镜像
###################
FROM scratch

COPY /templates /templates
COPY /static /static

# 从builder镜像中把/app 拷贝到当前目录
COPY --from=builder /build/app /

# 需要运行的命令
ENTRYPOINT ["/app"]
```

