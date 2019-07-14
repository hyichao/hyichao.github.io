---
layout: post
title:  "Java后台开发环境搭建"
tags: [Server]
---

服务器端的开发，主要依赖Java以及其相关类库，管理工具等等。再者就是一些数据库的知识。
从零开始，我首先需要搞懂一些基本概念。然后要把环境配起来。配置环境在mac上还是挺简单的，都是下载安装，顶多配置一下环境变量。

-------------------------------------------
## 1.下载安装 Intellij IDEA
 一个java的集成开发环境。
## 2.下载安装 Java的JDK
这个是java必须的开发包。安装完毕之后需要配置环境变量。这里先放一放，等后面的其他步骤完成后一并添加。
## 3.下载安装 maven
一个管理工具或者说项目构建工具，能协调管理java工程的第三方库，角色类似于iOS里面的cocoaPod，不过据说比pod复杂一百倍。。Anyway，还没有直接感受，先装好。这个管理工具也需要配置环境变量，也是稍后再配。
## 4.下载安装 MongoDB
一个NoSql数据库（非关系型，类似于protobuf那种感觉），面向文档型的数据库。关于关系型和非关系型，需要再深入了解一下。而所谓面向文档，这里的文档不是传统意义的文档，其实就是类似于关系型数据库的“行”的概念，只是“文档”并不要求固定数据结构。 Mongo需要配置环境变量。
## 5. 下载安装Redis
一个k－v数据库，据说是作为某些其他kv数据库不足的补充，也可以作为某些关系型数据库的补充。根据其官网的说法，既可以当cache，也可以当storage。（有点类似于之前用leveldb做缓存的感觉）。安装编译很简单，根据 http://redis.io/download 提示即可。make完在src里面有可执行文件。
## 6. 配置环境变量
打开终端，到home下

```
cd
vim .bash_profile
```

然后键入

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home
export M2_HOME=/Users/Charlie/tools/apache-maven-3.3.3
export M2=$M2_HOME/bin
export MAVEN_OPTS="-Xms256m -Xmx512m"
export MONGODB_HOME=/Users/Charlie/db/mongodb-osx-x86_64-3.0.2
export MONGODB=$MONGODB_HOME/bin
export PATH=\$M2:\$JAVA_HOME:\$MONGODB:\$PATH
```
其中M2_HOME和MONGODB_HOME取决于maven和mongo安装的路径在哪里。

重启终端，使用以下命令可以查看状态

```
echo $PATH
```
可以查看现在的环境变量，看看是否正常。

```
java -version
```
查看java的jdk是否正常

```
mvn -version
```
查看maven是否正常

```
mongod
```
打开mongdb，如果路径正确，应该是能看到一堆文字，然后停留在某一个地方不动。看看终端里面文字说了什么，就可以查看是否正常安装mongodb

```
mongo
```
打开mongodb的shell，能够用命令行的形式对数据库进行操作。具体的CRUD，见另一篇文章。