---
title: mongodb on macOs
date: 2019-09-17 01:20:24
tags:
---

1. 安装mongodb brew install 
2. mac php 安装 mongodb 扩展 
```
sudo pecl install mongodb

如果又一个 mkdir 报错 则需要  mkdir -p /usr/local/lib/php/pecl
pecl list 查看是否安装成功

$ touch /usr/local/etc/php/5.6/conf.d/ext-mongodb.ini

Finally add this line to the new file:

// example
extension="/usr/local/Cellar/php@5.6/5.6.35/pecl/20131226/mongodb.so"

```

3. laravel 框架为例 安装 mongodb 扩展包
