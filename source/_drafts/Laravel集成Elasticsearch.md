---
title: Laravel集成Elasticsearch
categories:
tags:
---

## 前言

我们知道 `Elasticsearch` 是一个开源的分布式搜索和分析引擎。既然是搜索引擎，那么它搜索些数据，如何搜呢？主要遵循如下两点

1. 按照 `Elasticsearch` 要求的规则创建可搜索的索引
2. 按照 `Elastcisearch` 要求的规则搜索相关数据

本篇将展示 `Laravel` 结合 `Elasticsearch` 实现上面的两点。

## 安装 Elasticsearch

参考博客中 `macOS安装Elasticsearch`

## 安装 Laravel Elasticsearch 扩展

```shell
$ composer require elasticsearch/elasticsearch
```

