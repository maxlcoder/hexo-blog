---
title: GO RPC
date: 2021-08-29 10:18:36
categories:
- Go
- RPC
tags:
---

## 前言

最近整理了一下 `Go` `RPC` 相关的实践内容，下面将我的目前涉及到的内容作如下梳理。主要说明 `RPC` 和 `gRPC` 在 `Go` 中的具体实现例子，以及可能遇到的问题。


## 什么是 rpc

> Remote procedure call

`RPC`: 远程过程调用。简单点说可以理解为不同对象通过网络进行远端的函数调用。这里就不对 `RPC` 具体的实现做说明，主要强调这是一种设计，通常可以理解为一种协议，只要是能按照这种设计作出的实现，都可以称之实现了 `RPC`。最早的设计请参考 [RPC论文](http://www.cs.cmu.edu/~dga/15-712/F07/papers/birrell842.pdf)。

## RPC 结构

![rpc](https://pic4.zhimg.com/45366c44f775abfd0ac3b43bccc1abc3_b.jpg)

通过结构图，可以比较直观的看到 `RPC` 工作的过程。

## 什么是 gRPC

> A high performance, open source universal RPC framework

这里可以看到 `gRPC` 是 `RPC` 的通用实现框架。其中 `g` 代表的是 `Google` 而不是 `Golang`，表示这个框架是由 `Google` 开发的。这个要知道。关于 `gRPC` 背后的故事，关心的可以自行搜索一下。

## gRPC 结构
![grpc](https://www.grpc.io/img/landing-2.svg)


初步了解 `RPC` 与 `gRPC` 的概念之后，我们可以来看看 `RPC` 的具体实现了。前者是基础，后者是在前者的基础之上做了很多系统性的封装处理，也就是更通用，也更好用。关于具体的 `RPC` 的原理

