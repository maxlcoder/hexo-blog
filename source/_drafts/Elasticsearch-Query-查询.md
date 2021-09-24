---
title: Elasticsearch Query 查询
categories:
- elasticsearch
tags:
- query
---

## Match Query 匹配查询

```json
{
    "query": {
        "match": {
            "testField": "abc"
        }
    }
}

# 对 text 类型 keyword 进行查询
{
    "query": {
        "match": {
            "testField.keyword": "abc"
        }
    }
}
```
这里要注意 `elasticsearch` 对字符串的存储默认是同时设置了 `text` 和 `keyword` 两种类型，属于一种动态类型。

这里有必要提到 `term` 和 `terms`， 与 `match` 的关系

*text*
* 会分词，然后进行索引，用于全文搜索。
* 支持模糊、精确查询
* 不支持聚合

*keyword*
* 不进行分词，直接索引，keyword用于关键词搜索
* 支持模糊、精确查询
* 支持聚合

通过匹配 `testField` 字段是否与查询内容匹配，如果 `testField` 不指定类型 ，这里默认是选择了`text`进行了分词查询的。
`match` 默认不指定默认查询标示时，都是进行的精确查询（存在分词时默认使用分词进行精确查询），所以如果要进行模糊查询需要根据情况请采用不同的匹配查询规则。

* 前缀查询: 查询条件中使用 `prefix`

```json
{
    "query": {
        "prefix": {
            "testField": "abc"
        }
    }
}
```
* 正则查询: 查询条件中使用 `regexp`

```json
{
    "query": {
        "prefix": {
            "testField": "abc"
        }
    }
}
```
* 通配符查询: 查询条件中使用 `wildcard`，查询值上使用 *（0或多个字符），？（单个字符）

```json
{
    "query": {
        "wildcard": {
            "testField": "abc*"
        }
    }
}
```



## Bool Queries 组合查询

```json
{
    "query" : {
        "bool" : {
            "should" : {
                "match" : { "my_other_field" : "xyz" }
            },
            "filter" : {
                "term" : { "my_field" : "abc" }
            }
        }
    }
}
```

`must`: 必须都匹配 AND
`must_not`: 必须都不匹配 AND NOT
`should`: 至少有一个匹配 OR