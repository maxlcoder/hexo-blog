---
title: Hello Hexo
date: 2018-04-03 01:36:10
tags:
---

很早就想没事的时候写点东西，结果一直耽搁。自从决定开始写个人pages，困扰人的第一件事情就是使用当前市面上的哪款产品来写。Jekyll，Hexo，GitBook，Hugo等等，都是很不错的选择，非要说出选择Hexo的原因，那就只能说，感觉到了，就选它吧。

### Hexo

#### 使用

下面列举几个比较实用的命令

```shell
$ hexo new "My New Post" # 创建文章
$ hexo server # 开启hexo服务，可以本地预览
$ hexo generate # 生成静态文件，通常是生成到`public`文件夹
$ hexo deploy # 部署
```

#### 个人站点

这里介绍一下我利用hexo建个人站点的过程，基本上也用到了上面的几个命令，下面说下整个过程。

1. 准备一个一个域名

之前我已经购买了一个域名winhm.com，这里设置一个二级域名作为个人站点的域名

2. 准备一台服务器

3. 准备两个仓库

这里我准备了两个仓库，一个是hexo源码仓库，一个是发布内容的站点仓库。这里使用两个仓库的原因有两点：第一点是hexo源码仓库主要用来做CI，站点仓库主要是做CD，二者不会有太多的干扰；第二点就是hexo deploy可以很便捷的将hexo源码仓库中的内容部署站点仓库。

4. 服务器配置

配置nginx，将个人站点的二级域名指向，个人站点目录

```shell
server {
    listen 80;
    server_name xxx.winhm.com;
    root /var/www/xxx;

    index index.html;
}
```

5. 写文章

本地使用hexo new和hexo server就可以开始写文章了，实际上就是写markdown文件。

6. 部署

在站点仓库上建立runner，本地执行hexo d，将触发runner 部署文件到服务器的个人站点目录