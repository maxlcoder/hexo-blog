---
title: gRPC protoc 命令行
date: 2021-08-27 11:43:55
categories:
tags:
---

`--go_opt=paths=source_relative` 设置生成文件在相对目录，可以避免是用 `go_package` 指定的包路径，[详见](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

```
protoc -I . google/api/*.proto --go_out=plugins=grpc:. --go_opt=paths=source_relative
```