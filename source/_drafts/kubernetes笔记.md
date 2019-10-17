---
title: kubernetes笔记
categories:
tags:
---

## 定义

容器的编排工具，手动管理转为程序执行。

## 产品

* docker compose

docker集成的，面向单机

* docker swarm

docker集成的集群工具，依赖 docker machine（将主机转为可以加入 swarm 中的 docker 主机）

* mesos，marathon

* kubernetes

k8s，神器，垄断，其他的可以忽略了

## 特性

* 自动装箱，自动修复，水平扩展，服务发现和负载均衡，自动部署和回滚
* 密钥和配置管理，存储编排，批量处理执行
* 

## 结构

* 集群
* Master/Node
    * Master: API Server, Scheduler, Controller-Manager
    * Node: kubelet, docker，kube-proxy
* Pod, Label, Label Selector
    * Label: key=value
    * Label Selector

* Pod:
    * 自主式（人工控制）
    * 控制器管理的 Pod (kuberlete 自动控制)
        * ReplicationController
        * ReplicaSet
        * Deployment (管理无状态应用)
        * StatefulSet (管理有状态应用)
        * DaemonSet
        * Job，Ctonjob (运行一次即结束)
    * HPA 高可用控制
        * HorizontalPodAutoscaler
    * Service 内部Pod组合服务
    * AddOns: 附件
        * DNS
    * 同一个Pod内的多个容器间： lo
    * 各Pod之间的通信（一般不直接通信）
        * 物理桥桥接，地址资源有限
        * Overlay Network（叠加网络），即新增一层网络封装
        * 结点分配不冲突的IP，Pod地址也不会冲突，可以直接通过隧道通信
    * Pod与Service之间的通信
        * Pod，Service各自都有自己的网络
        * kube-proxy协调地址

* etcd 共享存储
    * Api Server 通过etcd配置来获取集群的相关配置，实现共享
    * etcd 集群


* CNI 网络管理
    * 网络名称空间隔离Pod分组
    * flannel: 网络配置
    * calico: 网络配置， 网络策略
        * canel



## 安装

* kubeadm: 管理master内部各个组件（etcd, API Server, Scheduler, Controller-Manager），即集成安装包
    * 安装步骤：
        1. master, nodes: 安装kubelet, kubeadm, docker, kubectl命令行客户端
        2. master: kubeadm init 初始化
        3. nodes: kubeadm join 节点加入
* flannel
    * 网络组件，运行在各个结点之上


```shell
# master 安装
$ yum install docker-ce kubelet kubeadmm kubectl
# node 安装
$ yum install docker-ce kubelet kubeadmm

# master初始化
$ kubeadmm init 

# kubectl 使用 
$ kubectl get nodes # 获取结点列表

# 安装 flannel 组件，使用 kubectl 部署
$ kubectl apply -f https://xxxx/kube-flannel.yml

$ kubectl get pods # 获取节点，需要名称空间

# node，安装相关程序，之后 join 到 master
$ kubeadmn join xxx



```

> pause: 基础架构容器，网络，加载券，等


## 资源

* Restful api 控制
* 资源对象：
    * workload: 负载型资源，如 Pod，ReplicaSet, Deployment, StatefulSet, DaemonSet, Job, Cronjob, ...
    * 服务发现和服务均衡： Service, Ingress, ...
    * 配置与存储： Volume, CSI(容器扩展接口)
        * ConfigMap, Secret
        * DownwardAPI
    * 集群级资源
        * 名称空间，Node, Role, ClusterRole, RoleBinding, ClusterRolebinding
    * 元数据型资源
        * HPA(高可用)， PodTemplate, LimitRange
    * 

> apiserver 仅接收JSON格式的资源定义;
> yml 格式提供配置清单，apiserver 可自动将其转为json格式，而后再提交；
下面是yaml中定义的大部分资源清单：
    * apiVersion:
        $ kubectl api-versions
    * kind: 资源类别
    * metadata: 元数据
        * name
        * namespace: kubernetes级别
        * labels
        * annotations: 资源注解
    * spec: 期望状态
        * containers
            - name <string>
              image <string>
              imagePullPolicy <string> Always, Never, IfNotPresent
              ports <array> 说明性配置，没有实际端口暴露意义
                name
                containerPort
              livenessProbe
              readinessProbe
              lifecycle
        * nodeSelector <map[string]string> : 节点选择器
            
        * nodeName
        * restartPolicy: 重启策略
            Always, OnFailure, Never, Default to Always.
        * ExecAction: exec
        * TCPSoketAction: tcpsocket
        * HTTPGetAction: httpget
    * status: 当前状态，由kubernetes集群维护，用户不能自定义


    * 标签
        * 标签选择器
            等值关系：=，==，!=
            集合关系：
                KEY in (...)
                KEY notin (...)
                KEY
                !KEY


    Pod的生命周期：
        状态： Pending, Running, Failed, Succeeded, Unknown

        创建Pod:
        Pod生命周期中的重要行为：
            初始化容器
            容器探测： 
                liveness
                readiness
    

    探针类型三种：
        ExecAction, TCPSocketAction, HTTPGetAction


### 通过yaml清单创建pod步骤

```shell
# 1. 定义yaml清单
$ vi demo.yaml
# 2. 创建 pod
$ kubectl create -f demo.yaml
# 3. 产看 pod 运行情况
$ kubectl get pods -w
# 4. 删除 pod
$ kubectl delete pods demo 或者 kubectl delete -f demo.yaml

```


Pod 控制器：
    ReplicaSet: 副本集，表示运行多少个副本Pod
        1. 
    Deployment: 构建于 ReplicaSet之上的，以ReplicaSet形式部署Pod
    DaemonSet: 表示每个节点始终保持只运行一个指定的Pod
    Job:
    Cronjob:
    StatefulSet:
        1. 稳定且唯一的网络标识
        2. 稳定且持久的存储
        3. 有序，平滑的部署和扩展
        4. 有序，平滑的删除和终止
        5. 有序的滚动更新

        三个组件：headless service, StatefulSet, volumeClaimTemplate



Helm: 头盔
    


### Service

网络管理，CoreDNS, kube-dns
node network, pod network, cluster network 虚拟地址

kube-proxy 始终监听 ApiService 上资源变动状态，然后更新节点上关于集群的网络配置 ipvs 规则

工作模式： 
    userspace: 1.1-
    iptables: 1.10-
    ipvs: 1.11+

类型：
    ExternalName, ClusterIP, NodePort, LoadBalancer

    ClusterIP: 
    NodePort: client -> NodeIP:NodePort->ClusterIP:ServicePort->PodIP:containerPort
    LoadBalancer: 负载均衡类型
    
    No ClusterIP: Headless Service
        ServiceName -> PodIP

资源记录
    SVC_NAME.NS_NAME.DOMAIN.LTD
    其中 SVC_NAME，即清单创建的Service的name
    NS_NAME，即Service对应的名称空间
    示例
    redis.default.svc.cluster.local


### ingress & ingress controller

ingress 服务来分组Pod

ingress controller 代理外部请求，将请求转发到ingress 分组的Pod


## 挂载卷

外部存储类型

gitRepo
hostpath

pod 与 pvc 服务通信，pvc 再与外部存储 pv 通信

存储类
pvc 不直接和 pv 打交道，而是通过存储类打交道，这样pvc就不用关系pv

configMap: 配置中心作用
secret: 非明文配置中心

配置容器化应用的方式：
    1. 自定义命令行参数
    2. 把配置文件直接拷进镜像
    3. 环境变量
        1. Clound Native的应用程序一般可直接通过环境变量加载配置
        2. 通过entrypoint脚本来预处理变量为配置文件的配置信息
    4. 存储卷
    5. 

## 认证 && serviceaccount

客户端 -> API server
    user： username, uid
    group: 分组
    extra: 额外信息
    API：特定的api资源
        Request path: 标识 api 资源

serviceaccount: k8s资源，专用账号，该账号可以 通过RBAC来进行管理
 

## RBAC

## Dashboard

管理界面
1. 部署：
    kubectl apply -f https://xxxx./.yml

2. 将Service改成NodePort
    kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kube-system
3. 认证：
    认证时的账号必须为ServiceAccount: 被dashboard pod 拿来由kubernetes进行认证；

    token：
        1. 创建ServiceAccount,根据其管理目标，使用rolebingding或clusterrolebinding绑定至合理的role或clusterrole;
        2. 获取到此ServiceAccount的secret,查看secret的详细信息，其中就有token；
    


    kubeconfig：把ServiceAccount的token封装为kubeconfig文件
        1. 创建ServiceAccount,根据其管理目标，使用rolebingding或clusterrolebinding绑定至合理的role或clusterrole;
        2. kubectl get sercret | awk '/^ServiceAccount/{print $1}'
        3. 生成config

## 网络

docker:
    * bridge
    * joined
    * open
    * none 

kubernetes网络通信
    1. 容器间通信：同一个Pod内的多个容器的通信，lo
    2. Pod通信： Pod IP <--> Pod IP
    3. Pod与Service通信： PodIP <--> ClusterIP

CNI:
    flannel
    calico
    canel
    kube-router

    解决方案：
        虚拟网桥（纯软件的方式）
        多路复用： MacVLAN
        硬件交换：SR-IOV

    flannel:
        支持多种后端：
            VxLAN
                (1) vxlan
                (2) Directrouting
            host-gw: Host Gateway
            UDP: 
        
## 调度器

调度器选定Node步骤
    1. 预选：排除不合适的
    2. 优选：根据规则得分选取
    3. 选定：根据得分选定

也有根据预设值进行选定，在预选之后

污点容忍，污点驱离

## 监控系统指标

设置Pod，占用的CPU，内存等，同时监控这些消耗，从而进行管理

HeapSter：指标数据收集工具，可存入数据库，同时也可以使用Garafana,来展示数据库中存储额数据

## 资源指标API & 自定义指标API


资源指标：metrics-server
自定义指标：prometheus, k8s-prometheus-adapter

crd: 自定义资源


## helm

软件包管理

核心术语：
    Chart: 一个helm程序包
    Repository: Charts仓库，https/http服务器
    Release: 特定的Chart部署于目标集群上的一个实例

    Chart -> Config -> Release

程序架构
    helm: 客户端，管理本地chart仓库，管理Chart, 与tiller服务器交互，发送Chart,实例安装、查询、卸载等操作。chart是helm包，相当于软件清单，例如 brew的包，
    tiller: 服务端，接收helm发来的charts与Config,合并成release

    helm将请求发送tiller, tiller再转交给 api server，


## Paas

kubernetes 应用场景，DevOps中扮演重要的角色，将极大的简化DevOps实施，同时更容易促使其落地。