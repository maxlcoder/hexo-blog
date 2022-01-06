---
title: Dockerfile 详解
date: 2021-09-10 14:16:20
categories:
- docker
tags:
- docker
---

## 前言

`Dockerfile` 是 `docker` 镜像的构成文件，直白点就是 `docker` 根据 `Dockerfile` 文件来制作用户需要的镜像，通过该镜像来生成容器。
本文主要介绍 `Dockerfile` 的详细使用以及使用中需要注意的地方

## 实例

```docker
FROM nginx:mainline-alpine AS local-nginx
EXPOSE 80/tcp
COPY test.php /hello/test.php
RUN mkdir /testdir
RUN echo "Hello Wolrd" > /testdir/greeting
VOLUME /testdir

```

## 格式

`Dockerfile` 由一行行 `指令` 和 `参数` 组成，这些一行行的指令和参数组成了 docker 镜像的层，因此我们在构建 Dockerfile 时尽量使文件的行数越少越好

```docker
# 注释
指令[FROM,RUN,...] 参数[nginx-alpine,...]
```

### 注释

注释是被忽略的，这里的忽略是，你可以理解为注释在文件中是可以被忽视的，当作不存在。在命令换行中插入注释都是被允许的。如下面描述中 `# 注释` 是被允许的，同时实际上是不被镜像包含的。
```docker
RUN echo hello \
# 注释
world
```

### 解析器指令

Dockerfile 中有一种解析器指令也是以 `#` 开头的，但是格式如下
```docker
# directive=value
```
形如上面的注释格式，docker 会把它当作解析器指令处理，解析器指令不允许换行，同时不允许重复，下面的情况是不被允许的
```docker
# direc \
tive=value

# directive=value1
# directive=value2

FROM ImageName
```

同时如下几种情况解析器指令是被当作注释处理的
* 在构建指令之后
```docker
FROM ImageName
# directive=value
```
* 在注释之后
```docker
# About my dockerfile
# directive=value
FROM ImageName
```

然而 Dockerfile 中目前仅支持两种解析器指令
1. syntax 

此功能仅在使用BuildKit后端时可用，可以看作当前操作下做一些与处理来完成当前Dockerfile 的处理。可以达到如下目的：

* 在不更新 Docker 守护进程的情况下，自动获取 bug 修复补丁 
* 让每个使用当前 Dockerfile 来完成构建的人，构建环境相同，这个就相当于，解析的环境配置
* 在不更新 Docker 守护进程的情况下，，使用最新的功能
* 在不更新 Docker 守护进程的情况下，可以提前集成新功能或者第三方功能
* 使用自定义设置 

2. escape 

定义转义字符，例如 `\`


### 环境变量

`${variable_name}` or `$variable_name`: 以实际边变量值为准
`${variable_name:-word}`: 如果变量值设置了则是设置的变量值，如果没有设置，则 word 作为默认值
`${variable_name:+word}`: 如果变量值设置了，则 word 作为对应的值，这里表现就有点奇怪了。如果变量值没有设置，则是空字符串

### .dockerignore 文件
主要是定义了构建镜像需要忽略的文件。这个和 git 的 .gitignore 表现是类似的。


## 指令

### FROM

```docker
FROM [--platform=<platform>] <image> [AS <name>]

# or
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]

# or
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]

```

`platform`: `linux/amd64`，`linux/arm64`，`windows/amd64`，指定那中宿主机能够使用当前镜像，如果不指定的话，则表示镜像是全平台适用。

### RUN

* shell 模式: `RUN <command> `
* exec 模式: `RUN ["executable", "praram1", "praram2"]` 
官方默认推荐 exec 模式，RUN 指令是构建镜像过程执行的，也就在对应的镜像层执行一个命令，这个要和后面的 CMD 指令区分开。举例:
```docker
RUN yum install -y gcc

RUN ["yum", "install", "-y", "gcc"]  # 推荐用法
```
exec 模式在 windows 系统下标记路径时要求转义反斜杠，如果参数中含有环境变量，是不会替换的，所以要做 shell 处理: `RUN ["sh", "-c", "echo $HOME"]`，这样 `$HOME` 变量才会正常替换。

### CMD

* shell 模式: `CMD <command> `
* exec 模式: `CMD ["executable", "praram1", "praram2"]` 
* ENTRYPOINT 模式: `CMD ["praram1", "praram2"]` 
CMD 指令是为一个已经存在容器提供默认值，也就是完成容器的初始化的工作，这个是 RUN 允许的时机是不一样的。在构建镜像的时候 CMD 是进行的。
CMD 指令只运行一个，当存在多个时，后面的会覆盖前面的。
ENTRYPOINT 模式相当于为 ENTRYPOINT 传参。也就是 CMD 没有命令只有参数。

### LABEL

LABEL 指令为镜像提供元数据。以键值对的形式存在
```docker
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

### EXPOSE

```docker
EXPOSE <port> [<port>/<protocol>...]
```
TCP 或 UDP 服务时暴露端口号。
```docker
EXPOSE 80/tcp
EXPOSE 80/udp
```
上面的指令时暴露容器的端口，也就是将融的 80 端口对外暴露（向 docker 暴露）。配合下面的命令可以实现容器对外建立通信。
```shell
$ docker run -p 80:80/tcp -p 80:80/udp ...
```
`-p [宿主机器端口]:[容器端口]` 将容器的某个端口和宿主机的某个端口进行绑定。当用户访问宿主机端口时，就相当于访问了容器的端口。这用关系由 docker 的内部网络实现。

Dockerfile
```docker
FROM nginx:mainline-alpine AS local-nginx
EXPOSE 80/tcp
```
执行
```shell
$ docker run -p 8082:80/tcp docker-nginx-start
```
表示已经将宿主机的 `8082`端口和容器的 `80` 端口进行了绑定，我们访问宿主机的 `8082` 端口就相当于访问了容器的 `80` 端口，所以我们可以通过 `localhost:8082` 访问容器的 nginx 服务。

### ENV 
```docker
ENV <key>=<value> ...
```
设置构建阶段的环境变量。一直持续到容器被创建。在创建容器的时，可以通过指定 `--env` 来对环境变量进行替换
```shell
$ docker run --env <key>=<value>
```
如果不是全局的环境变量，可以在单行指令中指定环境变零如
```docker
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y 
```
表示 `DEBIAN_FRONTEND` 进行在当前指令执行时有效，其他时候没有使用到。

### ADD

```docker
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
将主机或者构建环境的 `源` 添加到 `目的地`，可以是文件，也可以是目录，或者是远端的源文件，一般是将 Dockerfile 文件所在目录对应位置的 `源` 添加到 `镜像` 对应的 `目的地` 位置 

### COPY

```docker
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
简化版 `ADD`，直接将 `源` 复制到 `容器` 的 `目的地`。

### ENTRYPOINT

* shell 模式: `ENTRYPOINT <command> `
* exec 模式: `ENTRYPOINT ["executable", "praram1", "praram2"]` 

和 CMD 类似，但是 ENTRYPOINT 的参数是不会被覆盖，而 CMD 参数是可以被覆盖的，如果 CMD 仅含参数时，相当于给 ENTRYPOINT 追加参数。
ENTRYPOINT 指令只运行一个，当存在多个时，后面的会覆盖前面的。

### VOLUME

```docker
VOLUME ["/data"]
```
指定存储挂载卷

### USER
```docker
USER <user>[:<group>]
# or
USER <UID>[:<GID>]
```
执行执行 RUN，CMD，ENTRYPOINT 的用户组

### WORKDIR

```docker
WORKDIR /path/to/workdir
```
指定运行 RUN，CMD，ENTRYPOINT 的运行目录

### ARG

```docker
ARG <name>[=<default value>]
```

定义变量，可以构建镜像时通过 `--build-arg <varname>=<value>` 来这是变量，同时这些变量在上面介绍的指令中都是可以使用的。就相当于给 Dockerfile 增加了人工定制的入口，很容易通过参数生成不同的镜像，增加镜像生成的灵活性。`ENV 变量可以覆盖 ARG 同名的变量`。


### ONBUILD

```docker
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```
构建触发器，当一个镜像被用于下一个镜像构建时，当前镜像定义的这个触发器就会在稍后的一个时间执行。

### STOPSIGNAL

```docker
STOPSIGNAL signal
```
定义退出容器的系统信号

### HEALTHCHECK

```docker
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
定义容器健康状态检查操作


### SHELL

```docker
SHELL ["executable", "parameters"]
```

定义执行 shell 格式指令时，使用的 shell，例如 linux: `["/bin/sh", "-c"]`，windows: `["cmd", "/S", "/C"]` 