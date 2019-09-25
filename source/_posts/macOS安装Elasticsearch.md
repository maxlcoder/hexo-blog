---
title: macOS安装Elasticsearch
date: 2019-09-20 13:47:54
categories:
tags:
---


## Elasticsearch

Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

## 安装

```shell
$ brew install elasticsearch // 注意目前 Homebrew 上的 elasticsearch 还是 6.* 版本，官网已经是 7.* 版本，所以如果想要安装最新的版本，使用下面的方式安装

$ brew tap elastic/tap
$ brew install elastic/tap/elasticsearch-full

```

## 安装目录

| 类型 | 说明 | 安装位置 |  
| --- | --- | --- |
| home | 主目录 $ES_HOME | /usr/local/var/homebrew/linked/elasticsearch-full |
| bin | 工具目录 | /usr/local/var/homebrew/linked/elasticsearch-full/bin |
| conf | 配置目录 | /usr/local/etc/elasticsearch |
| data | 数据目录 | /usr/local/var/lib/elasticsearch |
| logs | 日志目录 | /usr/local/var/log/elasticsearch |
| plugins | 插件目录 | /usr/local/var/homebrew/linked/elasticsearch/plugins |

## 启动

```shell
$ brew services start elasticsearch-full // 启动服务
```
启动服务之后，访问 `http://localhost:9200/` ，返回如下内容结构，则说明服务启动成功

```json
{
  "name" : "woodydeMac-mini.local",
  "cluster_name" : "elasticsearch_woody",
  "cluster_uuid" : "BzXnf9EvQtikruIe9R3P8g",
  "version" : {
    "number" : "7.3.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "4749ba6",
    "build_date" : "2019-08-19T20:19:25.651794Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

