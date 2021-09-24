---
title: Elasticsearch 设置密码访问
date: 2021-09-17 18:18:38
categories:
- elasticsearch
tags:
- password
---

## 前言

默认安装的 `elasticsearch` 是没有设置访问密码的，也就是只要是知道 `elasticsearch` 地址的，都可以向内部写入数据，这样缺少安全管理，因此这里需要对其设置访问密码

## 开启 xpack 安全选项

默认 `elasticsearch` 的 `7.*` 版本已经内置了 `XPack` 插件，但是这个插件选项是没有打开的，所以需要对其进行设置
```shell
$ sudo vi /etc/elasticsearch/elasticsearch.yml
```
在最后追加 `xpack.security.enabled: true`

## 设置密码

```shell
$ sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

出现如下提示

```shell
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]

```

也就是默认存在 `elastic`, `apm_system`, `kibana`, `kibana_system`, `logstash_system`, `beats_system`, `remote_monitoring_user` 这些用户，只需要对这些用户设置相应的密码即可。设置的时候注意保存各个用户对应的密码。可以看到这些用户基本涵盖了能和 `elastcisearch` 服务打交道的其他服务端。

## 重启 Elasticsearch

```shell
$ sudo systemctl restart elasticsearch.service
```

## 验证 

当再次执行 `curl http://localhost:9200` 时，得到如下提示
```shell
{"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}},"status":401}
```
可以看到密码已经生效了，再使用 curl 访问时，需要带上用户名和密码

```
$ curl -u elastic http://localhost:9200

Enter host password for user 'elastic':

```
根据提示输入密码即可得到

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