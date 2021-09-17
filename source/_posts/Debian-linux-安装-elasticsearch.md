---
title: Debian linux 安装 elasticsearch
categories:
  - elasticsearch
tags:
  - linux
  - debian
date: 2021-09-17 17:32:41
---


## 前言

本文记录如何根据官方文档提示在 `Debian Linux` 系统中使用 `APT` 安装 `Elasticsearch`

### 导入 Elasticsearch PGP Key

默认的系统可能不包含对一个的密钥，因此需要下载一个

```shell
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

之后执行

```shell
$ sudo apt-key list
```
查看密钥是否成功获取，我这里得到如下信息，表示成功获取

```shell
# ...

pub   rsa2048 2013-09-16 [SC]
      4609 5ACC 8548 582C 1A26  99A9 D27D 666C D88E 42B4
uid           [ unknown] Elasticsearch (Elasticsearch Signing Key) <dev_ops@elasticsearch.org>
sub   rsa2048 2013-09-16 [E]

#...

```

### 从 APT 仓库开始安装

安装之前可能需要先安装 `apt-transport-https` 包，主要是让 `apt` 命令支持 `https` 开头地址包的获取
```shell
$ sudo apt-get install apt-transport-https
```

再获取对应的 `elasticsearch` 的 APT 仓库的地址
```shell
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```
对应的地址保存在 `/etc/apt/sources.list.d/elastic-7.x.list` 中，如果发现错误了，可以直接删除，重新寻址正确的可安装的包源

```shell
$ sudo rm -rf /etc/apt/sources.list.d/elastic-7.x.list
```

准备工作做好之后，安装 `elasticsearch`
```shell
$ sudo apt-get update && sudo apt-get install elasticsearch
```
安装完成则得到如下信息

```shell
# ...
### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
 sudo systemctl daemon-reload
 sudo systemctl enable elasticsearch.service
### You can start elasticsearch service by executing
 sudo systemctl start elasticsearch.service
Created elasticsearch keystore in /etc/elasticsearch/elasticsearch.keystore
Processing triggers for systemd (241-7~deb10u8) ...
```

到这里可以看到安装成功了，再本机执行 `curl http://localhost:9200`，得到如下内容

```
{
  "name" : "xxx-elasticsearch",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "v8h7lDeJSdKdlSS0qxD0bg",
  "version" : {
    "number" : "7.14.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "66b55ebfa59c92c15db3f69a335d500018b3331e",
    "build_date" : "2021-08-26T09:01:05.390870785Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

