---
layout: post
title:  "MongoDB和Redis的CRUD"
tags: [Server]
---

简单了解和试用一下MongoDB和Redis，学习一下初级CRUD

## Have a Try
安装好MongoDB后， 可以其中的javascript shell来尝试运行一下
在Mac下，从安装到能够运行mongo的shell，步骤如下：
1. 找到MongoDB的官网。下载合适的安装包。如Mac下面的dmg
2. 解压安装。
3. 配置环境变量。
详情就不展开，因为另外一篇文章已经提及。

MongoDB的CRUD，是很基础的数据库内容，在图灵系列的MongoDB里面第一章便是CRUD。
打开MongoDB的脚本，也就是直接键入mongo，就可以开始测试下面的语句。

Create：

```
> post = {“title” : “C”, “Content” : “Create”}
> db.blog.insert(post)
```

先建立一个“信号”，然后插入信号到数据库

Read：

```
> db.blog.find()
> db.blog.findOne()
> db.blog.fing().pretty()
```

`.pretty()`是能在显示的时候令格式好看。。

Update：

```
> post.comment = []
> db.blog.update( {title: “C”}, post)
```

先更新post这个信号，然后把post发送到数据库并找到应该要更新的那一条

Delete：

```
db.blog.remove({title: “C"})
```

另一个要掌握的数据库是Redis，Redis的网站提供了一个很有意思的tutorial，是交互式的，一边敲，一边教。
需要掌握的东西不多

一个set一个get

```
> set genius charlie
> get genius
```

一些数据结构，list, set, sorted set, hash。
每一种都有一些基础的cmd，懒得记，查api就好。

