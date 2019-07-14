---
layout: post
title:  "Elastic Search 相关度计算"
tags: [Server]
---

为了搞清楚elastic search背后是如何计算文档搜索时候的相关度，我决定自己做实验去探索
这篇博客讲得还不错

<http://blog.csdn.net/dm_vincent/article/details/42099063>

而博客本身也只是翻译了官方文档

<https://www.elastic.co/guide/en/elasticsearch/guide/current/scoring-theory.html>

我准备验证一下

在进行文档的搜索时，应用了以下几个基础算法的组合。名字听起来很唬人，实际上也的确很唬人，基本上是基于统计理论里面的测度论进行的一些计算。
_____

### 1. 布尔模型(Bool Model)
假如现在搜索一个词组”hunter plus java"（利用terms可以做到）首先会应用一个bool模型，也就是先判断文档里面是否存在这三个term之一或者更多，只有存在关键词的文档才可以进入下一轮的竞争排序。这个bool模型很大程度保证了计算的实时性和有效性。什么？为什么要先排除不带有关键词的？连关键词都没有，凑合什么呢！

### 2. 词条频度/倒排文档频度(TF/IDF)
假如现在有两个文档如下

```
{"name":"charlie","description”:”hunterplus java"}
{"name”:"charles","description”:”hunterplus web"}
```
后面的三个计算都以这两个简单文档为例子

#### 词条频度 TF
词条频度，全称Term Frequency，简称TF，是对bool模型残留下来的所有文档进行词条的统计，得出了一个频数的平方根
这个频数是指残留文档中出现了关键词的文档数量除以总残留文档数量

* 公式为 TF ＝ sqrt(frequency) 
* 其中frequency为某个词在其文档出现的频数

例如hunterplus这个词条在description这个字段出现的频率就是1, 则
TF=sqrt(1)=1 

#### 倒排文档频度 IDF
倒排文档频度，全称Inverse Document Frequency， 简称IDF，是对某个词条在全数据库中所有文档出现的频率作的计算。比如说一篇文章里面，and和or，或者说中文里面的“的”字，都是经常出现的，那么如果这类型的词条当作关键词，其权重应该是非常低的，因为太频繁出现了，没有区分度。

* 公式为 IDF ＝ 1+log ( total/ (fre+1) )  
* 其中total是总文档数（过滤后的），fre是某个搜索词出现的频数


fre后面加一是为了分母不为零，前面的加一并不对函数变化产生影响
依然拿上面的例子来讲，对于hunterplus这个词条，就是
IDF=1+log(2/3) = 0.82

### 3. 字段长度归约(Field-Length Norm)
字段长度归约是为了让内容较短的字段发挥更大的作用，而内容较长的字段权重相对降低

* 公式为 NORM＝1/sqrt(numTerms+1)
* 其中numTerms是该文档里面关键词所在的字段的词条数量

依然取hunterplus为例子，
Norm = 1/sqrt(3) = 0.55

### 验证
1.2.3所描述的三个因子都是文档在index的时候就马上计算出来的，会占用一定的存储空间
我们可以通过explain查看具体的计算结果来验证

`curl localhost:9200/candidate/_search?pretty=true&&explain=true -d '{"query":{"term":{"description":"java"}}}'`

很可惜，看到的结果和理论值并不一致。。囧。。应该是es内部的实现不是理论这么简单，有其他方面的计算
不过基本思想是一样的，依然是通过计算词条的统计学量值来分析相关度。

-----
上面描述了对一个词条的相关度计算，那么假如是多个词条的话，就是多个词条对应的相关度构成一个一维向量，然后计算不同文档之间的向量距离，elastic采取的是余弦距离
例如上述例子，搜索两个term，一个是hunterplus一个是java，那么两个document都会出现，但是第一个doc会计算出比较大的相关度向量，假设是(5,2)，另一个(0,2)，那么以大的为最准，求出(0,2)与(5,2)的余弦距离，然后排序即可。