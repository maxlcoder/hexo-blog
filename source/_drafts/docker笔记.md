---
title: docker笔记
categories:
tags:
---

## 容器创建

```shell
$ docker run --name t1 -it --rm busybox:latest
# 指定网络，其实就是默认的配置
$ docker run --name t1 -it --network bridge --rm busybox:latest 
# -h / --hostname 指定主机名，不指定默认是和容器id一样 会自动更新在容器 /etc/hosts 中生成指向本地的hosts
$ docker run --name t1 -it --network bridge -h local.com --rm busybox:latest 
# --dns 指定 dns 解析
docker run --name t1 -it --network bridge -h local.com --dns "114.114.114.114" --rm busybox:latest 
```


## 网络

设备从属网络名称空间（网络隔离）

docker 网路方式：

1. 创建容器网络名称空间，但是不创建网络设备，也就是容器内封闭，对外没有交流 
```shell
$ docker run --name t1 -it --network none --rm busybox:latest # none 不创建网络设备
```
2. 创建容器网络名称空间，一半连接容器，一半连接 docker 虚拟桥
3. 创建容器网络名称空间，同时创建另外一个容器网络名称空间，这个容器网络加入到前一个网络
4. 创建容器网络名称空间，共享宿主机的网络名称空间

## Volumes(挂载卷)

docker 有两种设置 volumes 方式
1. 手动绑定宿主机和容器的 volumes 目录
```shell
$ docker run --name infracon -it -v /Users/woody/workspace/docker/volumes/data/:/data/web/html busybox
```

2. docker 来管理宿主机和容器的 volumes 目录
```shell
$ docker run --name infracon -it -v /data/web/html busybox
```

当多个容器共享 volumes 目录时，通常可以指定一个容器的 volumes 目录，其他的容器来共享这个容器的 volumes 目录，做法：

```shell
# --volumes-from [container name]
$ docker run --name infracon -it -v /Users/woody/workspace/docker/volumes/data/:/data/web/html busybox
$ docker run --name nginx --network container:infracon --volumes-from infracon -it busybox
```

## Dockerfile

#### .dockerignore

