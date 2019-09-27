---
title: docker
categories:
  - docker
tags:
  - docker
date: 2019-09-24 21:42:03
---


## 简介

Docker的英文翻译是“搬运工”的意思，他搬运的东西就是我们常说的集装箱Container，Container 里面装的是任意类型的 App，我们的开发人员可以通过 Docker 将App 变成一种标准化的、可移植的、自管理的组件，我们可以在任何主流的操作系统中开发、调试和运行。

从概念上来看 Docker 和我们传统的虚拟机比较类似，只是更加轻量级，更加方便使，Docker 和虚拟机最主要的区别有一下几点：

* 虚拟化技术依赖的是物理CPU和内存，是硬件级别的；而我们的 Docker 是构建在操作系统层面的，利用操作系统的容器化技术，所以 Docker 同样的可以运行在虚拟机上面。
* 我们知道虚拟机中的系统就是我们常说的操作系统镜像，比较复杂；而 Docker 比较轻量级，我们的可以用 Docker 部署一个独立的 Redis，就类似于在虚拟机当中安装一个 Redis 应用，但是我们用 Docker 部署的应用是完全隔离的。
* 我们都知道传统的虚拟化技术是通过快照来保存状态的；而 Docker 引入了类似于源码管理的机制，将容器的快照历史版本一一记录下来，切换成本非常之低。
* 传统虚拟化技术在构建系统的时候非常复杂；而 Docker 可以通过一个简单的 Dockerfile 文件来构建整个容器，更重要的是 Dockerfile 可以手动编写，这样应用程序开发人员可以通过发布 Dockerfile 来定义应用的环境和依赖，这样对于持续交付非常有利。

```shell

## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

```

## 容器

演示容器从定义，到运行，再到镜像创建与发布的一整套过程。

1. 容器定义

> 新建 `Dockerfile`

```shell
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

2. 应用内容

> requirements.txt

```
Flask
Redis
```

> app.py

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)

```

3. 应用生成镜像

```shell
$ docker build --tag=friendlyhello . # --tag 指定镜像名称

# 查看生成的镜像（本地镜像）
$ docker image ls
```

4. 启动容器

```shell
$ docker run -p 4000:80 friendlyhello # -p [本地端口]:[容器内部暴露的接口] 指定镜像的名称

$ # 浏览器防伪 http://localhost:4000 即可查看运行情况
```

5. 分享镜像

```shell
$ docker login # 登陆 hub.docker.com 的账号

$ docker tag image username/repository:tag # 对镜像打标签，image: 镜像名，username：写自己hub上的实际用户名，repository: 仓库名，tag: 标签名
# 下面是个示例
$ docker tag friendlyhello xxx/yyy:zz

$ docker push username/repository:tag # 发布镜像，参数说明同上

# 下次使用这个远端镜像的时候直接进行如下操作，docker 会去 pull 相关镜像，并运行

$ docker run -p 4000:80 username/repository:tag 

```


## 服务组合

服务：运行一个或者多个相同的容器，可以指定容器的相关配置，如运行数量，暴露的端口等等。
服务组合：是将多种服务组成一个完整的应用或者功能，各服务相互配合。

依靠 `docker compose` 可以完成这些操作

1. 定义 docker-compose.yml

```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

2. 初始化

```
$ docker swarm init
```

3. 启动全家桶

```shell
$ docker stack deploy -c docker-compose.yml [name]

$ docker service ls # 查服务信息
$ docker stack services [name] # 这里这个name 是启动的name加上配置中service项

# 查看服务中运行的 task，在服务中运行的每一个容器成为 task
$ docker service ps [name] # 这里这个name 是启动的name加上配置中service项
```

### 服务调整

服务调整只需要修改 `docker-compose.yml` 文件，再重新运行

```shell
$ docker stack deploy -c docker-compose.yml [name]
```
相关配置将会自动生效，无需停止全家桶之后再启动。

关闭相关服务的命令如下

```shell
# 关闭 app
$ docker stack rm [name]

# 关闭 swarm
$ docker swarm leave --force
```


## Swarm

`docker` 集群管理工具，`swarm manager` 管理许多 `nodes` （节点），`swarm manager` 可以执行相关操作，其他 `worker` 机器是不能的，但是 `worker` 机器也可以转为 `swarm manager` 机器。


### 建立 Swarm

```shell
$ docker swarm init # 将当前机器编程 swarm manager

$ docker swarm join # 其他机器上运行，可以编程 worker
```

#### 创建集群

1. 首先创建多个服务器，当前使用 `VirtualBox` 在本地创建多个虚拟机模拟多个服务器

```shell
$ docker-machine create --driver virtualbox myvm1
$ docker-machine create --driver virtualbox myvm2

# mac 上下载奇慢无比，最后还是成功了。
# To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm1
```
2. 查看创建的机器

```shell
$ docker-machine ls
```

3. 将其中一个设置为 `manager`

```shell
# 将 myvm1 设置为 manager ，设置完成之后，将会返回对应的添加 worker 和 manager 的提示
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>" # 其中ip填写上一步对应的ip
```
4. 将其中一个设置为 `worker`

```shell
$ docker-machine ssh myvm2 "docker swarm join --token <token> <ip>:2377" # 其中 token 和 ip 上一步均有返回，默认 2377 端口为 swarm 管理的端口
```

5. 查看 swarm 节点（node）

```shell
$ docker-machine ssh myvm1 "docker node ls"
```

### 部署应用到 Swarm 集群

1. 设置当前 command 窗口为服务器的代理窗口, 因为 myvm1 为 manager ，所以设置窗口对应到这个服务器。这样每次执行 docker 命令就不用使用 `docker-machine ssh xxx` 再包装一层

```shell
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)

# 运行上面提示的
$ eval $(docker-machine env myvm1)

# 查看激活的窗口
$ docker-machine ls
```

2. 部署应用
```shell
$ docker stack deploy -c docker-compose.yml [name]

$ docker stack ps [name] # 此时可以看到应用已经被平均部署到两台主机上了
```

3. 访问应用

随机访问 myvm1 和 myvm2 的 ip:4000 ，可以看到都可以访问到应用。实际上 swarm 集群会随机访问这两台主机，

4. 应用调整

更新 `docker-compose.yml` , 然后 `docker stack deploy`，服务就重新自动调整了

5. 清除和重启

```shell
$ docker stack rm [name]

# 取消 docker-machine shell 变量
$ eval $(docker-machine env -u)

# 重启机器
$ docker-machine start <machine-name>

```

## 全家桶（Stack）

通过修改 `docker-compose.yml` ，来管理相关的的独立服务，示例如下

```
version: "3"
services:
  web:
    image: maxlcoder/friendlyhello:v1
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:

```

## 部署

安装 `Docker Enterprise` ，并且通过 UI 来控制部署 compose 文件