---
title: 将项目的订单搜索接入 Elasticsearch
categories:
  - elasticsearch
tags:
  - elasticsearch
date: 2021-09-22 17:15:54
---


## 前言

项目的订单数据量有 180w+ ，在订单管理列表中需要对这些订单进行搜索，而且搜索的条件比较多，传统的 Mysql 查询已经不能很好的支持查询了，只要是由于查询条件存在范围查询，模糊查询，基本上不能匹配上索引。全表查询速度很慢。目前当个订单列表的 API 处理时间维持在 5s+，急需对其进行优化。

## 思路

想到的思路有两条
1. 在不增加外部服务的情况下，对 Mysql 查询进行优化。调整表结构以提高查询效率。实际评估针对这些查询所要处理的表结构表复杂。在一个稳定的订单系统做这样的处理显然会比较繁琐，同时对历史数据处理起来也比较困难。
2. 增加搜索引擎，取代 Mysql 查询。目前想到的就是 Elasticsearch ，利用 Elasticsearch 的特点来解决 Mysql 查询慢的问题。


## 设计结构
![](订单系统.png)


## 步骤

> 有关 Elasticsearch 的介绍与安装参考其他博文

1. 将订单按照指定的结构同步到 Elasticsearch。

2. API 中对订单的搜索由 Mysql 查询替换为 Elasticsearch 的查询

3. 对查询结果进行数据填充，完成 API


## 注意点

### 订单同步

首先要弄清楚 Mysql 中订单表和 Elasticsearch 的中数据结构的关系。Elasticsearch 数据结构由 index 和 document 组成，分别对应了数据库中的表和行。查询数据库中某个表的行记录，可以对应为查询 Elasticsearch 中某个 index 的 document。

项目的订单数据查询，存在关联关系的处理，这点如果完全体现在 Elasticsearch 中将导致查询效率较低，需要针对