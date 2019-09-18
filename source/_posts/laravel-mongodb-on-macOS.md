---
title: laravel mongodb on macOS
date: 2019-09-17 01:20:24
tags:
---

> 前提假设 `macOS` 上没有安装任何 `MongoDB` 相关的内容

1. 采用 `brew` 安装 `MongoDB`，由于 `MongoDB` 开源许可的问题，已经不再 `brew` 的官方源中，需要单独的加载源，`MongoDB` 有不同的版本可根据需求选择，演示采用 `MongoDB Community Edition`，安装参考链接 https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/

```
brew tap mongodb/brew

brew install mongodb-community@4.2

brew services start mongodb-community@4.2 // 启动 mongodb 服务

```

2. 这边需要测试 `Laravel` 中使用 `MongoDB`，所以还行 `PHP` 安装 `MongoDB` 扩展 

```
sudo pecl install mongodb

# 如果遇到一个 mkdir 报错 则手动执行  
mkdir -p /usr/local/lib/php/pecl

# 查看是否安装成功
pecl list 

# 加载 extension
touch /usr/local/etc/php/5.6/conf.d/ext-mongodb.ini
# 在 ext-mongodb.ini 中添加
extension="/usr/local/Cellar/php@5.6/5.6.35/pecl/20131226/mongodb.so"
```

3. 为 `Laravel` 框架安装 `MongoDB` 扩展包

```
composer require jenssegers/mongodb
```

4. 具体的使用请参考 https://github.com/jenssegers/laravel-mongodb