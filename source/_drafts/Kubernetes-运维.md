---
title: Kubernetes 运维
categories:
- k8s
tags:
---

## 集群管理

### Node 管理

1. 创建 `.yaml` 配置文件，定义 Node
```yaml
apiVersion: v1
kind: Node
...
```
2. 生成 Node

```shell
$ kubectl create -f xxx.yaml
```

3. 使用配置文件更新 Node
```shell
$ kubectl replace -f xxx.yaml
```

4. 查看 Node

```shell
$ kubectl get nodes
```

5. 不使用配置文件，使用命令更新 Node 状态，这里是让 Node 脱离集群的调度
```shell
$ kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'
```

6. kubectl 子命令 cordon 和 uncordon 来对 Node 进行隔离和恢复

```shell
$ kubectl cordon k8s-node-1

node/k8s-node-1 cordoned

$ kubectl uncordon k8s-node-1

node/k8s-node-1 uncordoned
```

### 更新资源对象的 Label

```shell
# 给 xxx Pod 添加 role=backend 的 标签
$ kubectl label pod xxx role=backend

# 删除标签
$ kubectl label pod xxx role-

# 修改标签
$ kubectl label pod xxx role=master --overwrite
```


### Namespace

1. 创建 namespace

*namespace-development.yaml*:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```
```shell
$ kubectl create -f namespace-development.yaml

namespace/development created
```
