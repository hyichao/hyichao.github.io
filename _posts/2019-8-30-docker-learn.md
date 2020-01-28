---
layout: post
title:  "机器学习工程师学Docker"
tags: [Misc]
---



## 前言

关于Docker，最好的介绍自然是跟随[官方网站](https://www.docker.com/)。

为什么要用Docker，个人认为官网的那句话足够表达。

> Debug your app, not your environment
>
> Securely build, share and run any application, anywhere



当我们在进行生产环境部署的时候，客户的机器千差万别，如果针对每一个不同的机器都做适配，工程量十分巨大，也不符合做软件工程的理念。自然而然地，工程师们需要一种方法，能够使得软件和对应的环境紧密绑定起来，使得代码能够在任何机器下进行部署，于是有了Docker应用，“把依赖环境和可执行程序打包在一起”。



本文目标读者是未接触或刚接触容器化技术的机器学习工程师，将介绍如何把现有的应用改成容器化应用，包括基于CUDA基础镜像的使用GPU的项目，包括常规C++项目，等等。

本文不涉及Docker基础概念以及Docker环境的安装，这部分内容可以参考 Docker 官网。



## 容器化

由于本文是从机器学习工程师的角度出发去看待Docker的使用，所以很自然地会从最简单的使用场景入手进行介绍。

#### C++工程

一般C++工程可以简单地从某个Linux系统出发去构建。想象一下，如果从一个“裸装”的机器，我们如何能够运行起来一个C++工程呢？我想大致上可以描述如下：

* 装系统（Ubuntu/CentOS/etc.）
* 装编译环境（cmake/make/etc）
* 装依赖（~~）
* 编译--运行



前言提到，Docker应用可以看作是“把依赖环境和可执行程序打包在一起”，所以除了”装系统“这个动作，其他步骤我们需要一个东西去打包成一个整体，于是引入[Dockerfile](https://docs.docker.com/engine/reference/builder/)的概念。



```dockerfile
FROM ubuntu:18.04

LABEL MAINTAINER="Somebody <somebody@company.net>"

# https://askubuntu.com/questions/909277/avoiding-user-interaction-with-tzdata-when-installing-certbot-in-a-docker-contai
ARG DEBIAN_FRONTEND=noninteractive

# replace with a mirror source
ADD sources.list.18.04 /etc/apt/sources.list

# install whatever you need like in real machine
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential wget tar git cmake \
        && apt-get clean && rm -rf /var/lib/apt/lists/*

COPY . /code
WORKDIR /code

RUN ./build.sh

ENTRYPOINT ["./run.sh"]
```



上述Dockerfile是一个简单的小例子，其中几个常见的关键词

如`FROM`，`ADD` ，`RUN`，就不多做介绍。

特别地，ubuntu下使用apt-get，或者centos下使用yum，常常遇到需要人工交换，但是Dockerfile在进行构建之时应该禁用交互，所以一般添加一句`ARG DEBIAN_FRONTEND=noninteractive`。另外在使用`apt-get install`之后，为了减少Docker的体积，会添加一句`apt-get clean && rm -rf /var/lib/apt/lists/*`，因为网络下载的东西用完删除，是不会被持久化的。



完成Dockerfile后，使用下面命令运行

```shell
docker run -it docker_image_name
```



如果应用是一个可执行程序，执行完毕就结束的话，docker的运行也是一样。

如果应用是一个服务的话，后台执行起来就是一个正儿八经的服务器了。



#### 小技巧

写Dockerfile是一个颇为挣扎的过程，特别是新手需要为一个规模稍大的工程添加Dockerfile，常常需要长时间的`apt-get update && apt-get install xxxxx`，也需要长时间的编译构建，出错重试周而复始。为了稍微减少这个过程，我们可以使用

```shell
docker run --entrypoint /bin/bash dokcer_image_name
```

进入容器的shell，然后一点一点增加apt语句和编译命令，完成后把history倒腾出来然后整理成为Dockerfile所需要的格式。Dockerfile里面的每一个命令，都是一个缓存层，把基本不变的部分放到前面，把变化较多的部分放到后面，也一定程度上减少试错时间。