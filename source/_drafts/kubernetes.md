---
title: kubernetes
categories:
tags:
---

生产级别的容器编排系统 

Replication Controller: 声明 Pod

Replica Set: RC 的升级版，与 RC 的唯一差别就是其支持基于集合的标签选择，而 RC 只基于等式的标签选择

Deployment: 也是 RC 的升级，能够知道 Pod 部署过程中的状态

可见上面三个都是可以定义 Pod 和实现 Pod 的管理的功能的，只是后者可以看着是前者的升级。