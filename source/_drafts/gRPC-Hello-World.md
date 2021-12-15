---
title: gRPC Hello World
date: 2021-08-30 10:29:46
categories:
tags:
---

## 前言

参考官方文档整理梳理一下 `gRPC` 开发过程

## 安装

1. Go 环境（注意这里将 $GOPATH/bin 加入到系统的环境变量单中）
2. 安装 `Protocal buffer` 编译器，Google 的支持代码生成的编译器（不限于生成 Go 代码），这里展示 `macOS` 下安装方式，其他环境的可以参考 [安装](https://grpc.io/docs/protoc-installation/)
```
$ brew install protobuf
$ protoc --version  # Ensure compiler version is 3+
```
3. 安装 protocal 编译器的 Go 插件，来支持生成 Go 代码，如果是需要生成其他语言的，选择相应的插件即可（也可以不指定版本），这里参考网上的一些情况，这里存在新旧版本问题，一些安装介绍用的是 `github.com/golang/protobuf/protoc-gen-go`，注意这个是老版本插件，在后面的使用中可能会出现一些问题。
```
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

## 体验
这里可以根据官方文档体验官方的示例

1. 下载示例代码
```
$ git clone https://github.com/grpc/grpc-go
```
2. 进入包目录，由于后面要同时执行客户端和服务端，这里需要开多个 tap 窗口
```
$ cd grpc-go/examples/helloworld
```
3. 编译执行服务端代码
```
$ go run greeter_server/main.go
```
4. 编译执行客户端代码
```
$ go run greeter_client/main.go
2021/08/30 10:56:29 Greeting: Hello world 
```

## 自定义 `gRPC`

![grpc](https://grpc.io/img/landing-2.svg)

根据 `gRPC` 结构，我们知道需要构建服务端和客户端，前面介绍的 `protocal` 将根据 `.proto` 文件，为我们完成服务端和客户端 `Stub` 的创建，我们只需编写程序调用这些 `Stub` ，以及服务端的实现逻辑，来完成相关的需求。


```
# 注意这里有些教程是 --go_out=plugins=grpc:. 这种属于过时的操作了
protoc --go_out=. message.proto
protoc --go-grpc_out=. message.proto
```