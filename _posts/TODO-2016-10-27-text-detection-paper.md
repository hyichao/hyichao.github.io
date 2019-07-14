---
layout: post
title:  "论文阅读：Text Detection, Tracking and Recognition in Video: A Comprehensive Survey"
date:   2016-10-27 20:00:00 +0800
categories: CV
---

## Motivation

为了快速了解text detection/tracking/recognition领域的相关工作，选择阅读一篇综述，来自殷绪成老师的TIP论文***[Text Detection, Tracking and Recognition in Video: A Comprehensive Survey](https://www.researchgate.net/publication/301316962_Text_Detection_Tracking_and_Recognition_in_Video_A_Comprehensive_Survey)***

## Core
文章里面一个很重要的图放在第二部分，描述了Detection/Tracking/Recognition之间的关系。

1. Detection: 定位文字的位置，用bbox勾勒
2. Tracking: 相邻帧维持bbox
3. Recognition: 基于检测的结果识别文字，故也称为Detection-based-Recognition
4. Tracking-with-Detection: 对于tracking问题，首帧如何获得以及丢帧的处理都是难题，故出现用detection来提供首帧以及丢帧复原
5. Tracking-based-Detection: 如果在视频流里面对于每一帧都仅做detection处理，则丢失了时序相关性。故出现在detection以外，配合tracking结果去进行纠正
6. Refinement-by-Recognition-for-Tracking: 利用文字识别的结果纠正Tracking结果，是一种负反馈的系统。例如文字识别模块认为该区域不是文字，则意味着Tracking结果不正确
7. Refinement-by-Recognition-for-Detection: 同样的，利用文字识别的结果，也可以负反馈到检测模块
8. Tracking-based-Recognition: 感觉这种手段比较少见，利用Tracking得到的带时许信息的特征导入识别模块，例如多个帧跟踪之后叠加成为多个通道或者铺开成为一个特征图然后进入文字识别

实际上，上述的Detection/Tracking/Recognition三者的关系不仅仅是视频文字领域，对于任意动态物体检测跟踪识别都有上述的几种思路。最有名的自然就是[TLD算法](http://kahlan.eps.surrey.ac.uk/featurespace/tld/Publications/2011_tpami)了。

在介绍完这些基本概念之后，论文就开始分点论述不同思路下的相关工作了。

#### Individual Frame

##### text detection 
> connected component (CC) based methods, and region based methods (also called sliding window based methods).

CC方法看引用可知是年代比较久远的方法，应该已经基本不适用了，例如颜色连通域，比较肯定的是鲁棒性较弱。至于region based method，与其说是一种方法，不如说是一种思路，把检测问题转化为分类问题（现在FRCNN不就是这个思路嘛～），取得region的方法，最最直观的就是滑动窗了，而分类器嘛，根据特征提取的优劣，选用合适的分类器。一般而言，特征可分类性越强，分类器就越弱，例如特征本身线性可分，那自然用线性分类器就足够了。

##### text recognition
实际上做recognition第一步总是先分割出文字区域，即滤除背景。
> text regions are first segmented from video frames and then fed into a state-of-theart OCR engine

毫无疑问，文字分割的好坏直接影响了文字识别的优劣。利用传统特征可以做到不错的分割，如gray-scale consistency constraint。然而CNN大行其道的今天，也自然有新的工作基于CNN结合领域知识做的一些突破。

假设现在有一个足够好的分割方法，那么接下来就需要识别文字。一般而言基于分割的文字识别会利用优化方法建立识别模型，如MarkovModel。另外还有一个叫做word spotting的做法，大概是，给出搜索样本，然后在文本池里面搜索配对的样本区域（有错请指正？），于是不需要进行背景滤除。近年有一些工作不再是基于背景滤除，例如CNN就可以在不滤出背景的情况下识别人字，然而这类型的工作对应需要的数据量都比较庞大。在数据不是大问题的大环境下，利用CNN做整词识别或者文本整行识别渐渐成为主流。

#### Multiple Frame

对于multiple frame的情形，实际上与single frame的区别就在于时序了，故实际上重点落在tracking方面。在multi frame状态下，tracking能够与detection进行互补或者加速。

> Text tracking is useful for verification, integration, enhancement and speedup in video text detection and recognition

##### text tracking
稍微分类一下tracking的方法的话，论文分成以下几个：

1. tracking with template matching. 这里的template matching一般指抽取过特征的template，例如提取的HOG/SIFT/SURF等等。然后，通过计算采样的样本和模版的相似度去判断搜索是否继续。
2. tracking with particle filter. 同样的需要经过特征提取，然后根据ObservationModel(注：暂时不理解这个)计算相似性，再通过当前帧计算下一帧可能出现的位置。
