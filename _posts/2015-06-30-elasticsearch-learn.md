---
layout: post
title:  "初窥ElasticSearch"
tags: [Server]
---

**粗略看起来，其实Elastic Search (w.r.t. es)和其他DB没什么大区别，只是在搜索上有很多强大功能，所以很适合用在需要搜索的项目。貌似用curl发送一个JSON格式的数据（实际上是命令）到es就可以做CRUD**

* elasticsearch权威指南，一本书，也许有帮助，在gitbook上正在翻译。下载下来看，翻译的其实挺不错。虽说看英文版原汁原味，但是看中文还是快上很多。。下载下来叫LearnElasticSearch.pdf

<http://es.xiaoleilu.com>

* 一个简单的es介绍，中文的看起来快

<http://www.elasticsearch.cn>

* 一个入门型的指导，是gradle的

<http://java.dzone.com/articles/first-step-spring-boot-and>

* 来自spring io的文档，简单讲了一下springboot里面可以直接应用elastic search 

<http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-nosql.html>

---------
####**花了一晚上看了不少资料，才堪堪搞明白这个东西是啥玩意。。资质驽钝。。。ES本质上其实真的和Mongodb之类的NoSql数据库没有特别大的区别，估计最大的优点是模糊搜索，范围搜索之类的功能，所以名字就很强调search。我一开始以为这个是用来搜索的算法库，，结果最终还是个数据库。。希望我现在的理解是肤浅的，希望它功能远远不止这些。。。根据LearnElasticSearch.pdf，可以初步学习ES在命令行下面的一些基本知识，其实就是数据库的CRUD之类的，还有更高级一点的就是ES的搜索功能（重点）。**

---------

C: 

```
curl -XPUT localhost:9200/megacorp/employee/1 -d ‘{“name”:”charlie"}'
```

R:

```
curl -XGET localhost:9200/megacorp/employee/1
```

U:

```
curl -XPOST localhost:9200/megacorp/employee/1 -d ‘{“last_name”:”peng"}'
```

D:

```
curl -DELETE localhost:9200/megacorp/employee/1
```

在基础的CRUD之后，ElasticSearch提供了强大的搜索功能。
搜索的时候，其实是发送一个json数据到一个url，然后这个json数据包里面包含了搜索条件。

```
curl -XGET localhost:9200/magacorp/employee/_search?pretty -d ‘{“query”:{“match”:{“last_name”:”huang"}}}'
```

这里的json称谓DSL，就是所谓的Domain Specific Language

更复杂的DSL如下，

全文搜索：

```
{
     "query" : {
     "filtered" : {
          "filter" : {
               "range" : {
               "age" : { "gt" : 30 }
          }
     },
    "query" : {
          "match" : {
          "last_name" : "Smith"
         }
 }
     }
}
```

这个会根据搜索评分按顺序给出反馈

```
{
   "query" : {
         "match" : { "last_name" : “John Smith"}
     }
}
```
这个会精确搜过该phrase

```
{
   "query" : {
         "match_phrase" : { "last_name" : “John Smith"}
     }
}
```



