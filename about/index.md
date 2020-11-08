---
layout: aboutme
title: About Me
tags: about
date: 2019-07-11
comments: false
--- 

> 看看论文，写写代码，调调参数，测测性能  
> 努力跟上时代的步伐

## Degree

#### Master's Degree
@ 华南理工大学，广州  
导师： 金连文教授

#### Bachelor's Degree
@ 中山大学，广州  

## Research
1. 计算机视觉与深度学习
2. 文字检测和文字识别（OCR）
3. 手势检测和识别

## Engineering

1. 深度学习常用 Caffe/PyTorch
2. 常规数字图像处理 OpenCV
3. 中型规模的c++工程，如产品级别的OCR工程
4. 小型规模的python工程，如内部研究的工具库等
5. 初级CUDA编程水平
6. iOS/Java不做好久了

## Publications

> Y. Huang, X. Liu, X. Zhang, and L. Jin, "A pointing gesture based interaction system: dataset, approach and application," IEEE Conference on Computer Vision and Pattern Recognition (CVPR) workshop, 2016

> Y. Huang, X. Liu, L. Jin and X. Zhang, "DeepFinger: A Cascaded Convolutional Neuron Network Approach to Finger Key Point Detection in Egocentric Vision with Mobile Camera," IEEE Conference on System, Man and Cybernetic (SMC), 2015

> X. Liu, Y. Huang, X. Zhang, and L. Jin, "Fingertip in the Eye: A cascaded CNN pipeline for the real-time fingertip detection in egocentric videos," arXiv preprint arXiv:1511.02282, 2015

## Work

一直投入OCR方向的底层研发，大概分为以下几个阶段。

#### 2017

参与研发以深度学习为基础的OCR引擎，主要负责整行文字识别部分。

+ **文字识别**

2017年的整行识别，整个行业基本都一样，只有一种靠谱的方案：

1. 模型方案：CNN+RNN+CTC，工作中做了大量实验找到比较关键的超参数；
2. 工程实现：Caffe+WarpCTC，工作中做好了模型训练的的代码质量，还有各种数据清洗；

#### 2018

持续研发以深度学习为基础的OCR引擎，除了整行识别部分以外，还涉及一些模型加速，并大量投入做OCR引擎的工程化。

+ **OCR引擎**

1. 重构OCR引擎进行模块化设计；
2. 兼容支持多个推理运算库保证一套代码多种部署如支持GPU和CPU；
3. 针对速度较慢但并行度高的环节添加CUDA支持；

+ **模型加速**

1. 利用张量分解和通道剪枝把文字识别模型加速到能够在CPU端使用；
2. 参考紧致模型设计的一些工作把文字检测模型也加速至CPU端可用；

#### 2019

为了拓展知识广度，深度参与了文字检测模型方面的研发，以及应用更多的工程技能到OCR引擎中。

+ **文字检测**

1. 探索和魔改Anchor+Link思路下进行任意方向文字检测的方案；
2. 探索和实践Segmentation思路进行任意方向文字检测的方案；
3. 稍微探索了一下End2End文字检测识别模型；
4. 最终确定任意方向通用文字检测的方案并优化细节直到上线支持各种业务；

+ **OCR引擎**

1. 初步支持任意方向的OCR引擎；
2. 进行了多项工程优化如流程优化和多线程等；
3. 为OCR引擎引入DevOps相关的内容；

+ **训练代码库**

1. 开发两套内部使用的模型训练库，能支持多种文字检测方案以及多种文字识别方案；
2. 完成训练和部署之间的自动化流程；

#### 2020

略


## Contact

* E-mail: *hyichao at foxmail.com*
* Github: *hyichao*