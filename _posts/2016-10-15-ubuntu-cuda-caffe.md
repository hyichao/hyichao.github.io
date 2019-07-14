---
layout: post
title:  "Ubuntu 14.04 安装Cuda／安装驱动／安装caffe"
tags: [Deep Learning]
---

本文实际上是旧文翻新，不小心弄坏了自己的cuda环境，折腾了一天左右重新搞好，然后加深了一点点对整个安装过程的理解，以及更新一点相关问题的解决方案。

## Motivation
在近年的深度学习热潮里面，研究者都离不开使用一些流行的深度学习框架，较早期是caffe，后面各大公司各大团队纷纷开源出新的深度框架，各有特色，呈百家争鸣之势，如tensorflow,mxnet等。关于这些框架的优劣比较，可以参考[这篇文章](https://github.com/zer0n/deepframeworks)。当然，如题所示，本文的重点不在于这些框架，而是在于如何安装使用这些框架。实际上，网络上充斥着各种教程，大部分都是知其然不知其所以然，即使是stackoverflow或者是askubuntu，也遇到大量无用答案，遇到问题搜索起来还是比较痛苦。本文算是为自己做一点笔记，也希望有一天能帮助到有需要的人。

## Must Know
无论我们最终选择使用哪个深度学习框架，都必须使用Nvidia的显卡和cuda（Nvidia是这次深度大潮的真正大赢家）。那么在我们入坑的第一天，毫无疑问，需要做的事情就是装cuda环境。接下俩我们一一分析一些常见教程里面的每一个步骤究竟做了什么，在什么情况下会出问题，又该如何修复。在开始之前，首先问自己以下几个问题：

1. 你的机器是什么**系统**？（本文暂时支持Ubuntu14.04 64bit，16.04有特殊的打开方式..）
2. 你的**显卡**是什么**型号**？（本人使用的是TitanX，1000系列有特殊的打开方式..）
3. 你的系统现在处于什么状态？（**新装／装过驱动／装过cuda**）
4. 你使用**独显渲染GUI**吗？还是使用**集显输出GUI而独显仅作计算**？（非常重要！本人使用集显输出显示独显仅做计算，好处是训练的时候不会卡..）

对于第一个问题和第二个问题，主要是考虑一些驱动的兼容性问题。一般而言14.04相对比较稳定，大多数显卡都能够在14.04使用。然而比较新的型号入NVidia 1070或者1080这个系列仅支持Ubuntu16.04，然后就引起一些编译器版本的兼容性问题，据说是16.04的gcc版本太高,cuda-8.0无法编译，也是比较搞笑。如果显卡不是这么高端的话，且还可以选择系统的话，建议14.04。

对于第三个问题，主要是考虑驱动残留的问题。有时候，小白会根据网上一些教程胡乱安装导致系统残留各种奇怪的程序。大部分情况下，其实都没有什么问题，但是一旦涉及到驱动，很容易就出现无法进入图形界面，登陆循环等问题。一般而言，比较理想的安装环境就是没有任何nvidia驱动，而为了达到这个目的，我们可以通过卸载所有nvidia驱动实现。

```shell
# the command uninstall all driver remain in system.
sudo apt-get remove --purge nvidia-*

# if this cause GUI problem, try the following
sudo apt-get install ubuntu-desktop
```

如果是安装过cuda但是想卸载掉现有的cuda重新安装更新的版本，比较简单的做法自然是进入cuda的安装路径，直接删除，包括一个cuda-x.x和一个软连接。

```shell
cd /usr/local
sudo rm -rf cuda*
```

OK,该卸的卸该删的删，既然决定了要重新安装，那就说明你和现有的这个cuda没有感情了，当断则断，否则剪不断理还乱。然后，我们开始选择一个合适的方式去安装。

## Installation Guide

为了使用GPU跑深度框架，我们需要完成以下几项安装流程：

### 1. 必要的基本依赖
参考caffe的[instruction](http://caffe.berkeleyvision.org/install_apt.html)。这里面所安装的有些是caffe的依赖如protobuf/leveldb/opencv，有些是c++环境如build-essential，有些是计算库如blas。其中opencv基本上是视觉领域的标配，protobuf和leveldb是caffe为了加速io选择的技术，build-essential包含了如编译器gcc等内容，etc. 那么因为这些依赖基本对系统核心不影响，全部装上即使不用也无所谓，可以放心安装。
### 2. Nvidia驱动的安装
从此处开始，我们要小心翼翼的选择我们的安装方式，安装驱动这件事情坑实在太多，一不小心就把自己陷入麻烦的漩涡。首先这里我们提出两种安装NvidiaDriver的方式.

1. 方案一：通过[NVIDIA](http://www.nvidia.com/Download/index.aspx?lang=en)官网下载驱动的`.run`文件，赋予执行权之后，运行`.run`文件安装
2. 方案二：安装cuda toolkit顺便把驱动也装了

两种安装方式都可行，如使用方案一，那么在安装cuda的时候可以选择安装驱动；否则，在cuda安装的同时把驱动也装上其实也是可以的。

***！！有一个地方一定要注意！！***

假如**<u>使用独显仅作计算而使用集显输出GUI</u>**，那么我们需要在安装驱动的时候加上一个高级选项 `--no-opengl-libs`。原因是，NVidia的驱动默认会安装openGL，而实际上ubuntu内核本身也有openGL而且和GUI显示息息相关，那么一旦NVidia的驱动覆写了opengl，在GUI需要动态链接opengl库的时候就引起问题。本人遇到的就是登陆界面死循环，英语一般称为“login loop”或者"stuck in login"。所以，如果是使用`.run`的方式安装，在运行`.run`的时候，就带上刚刚提到的高级选项。完整流程如下：

```shell
# switch to tty by control+alt+f1
sudo service stop lightdm
cd /path/to/your/runfile/
# could use command ll to check if it is executable
chmod +x NVIDIA-Linux-x86_64-367.57.run
sudo ./NVIDIA-Linux-x86_64-367.57.run -no-opengl-libs
```

###### 可能遇到的问题以及解决方案
上述要注意的这个地方，万一一个不小心手抖了添加no-opengl的选项，那么如无意外会遇到循环登陆（login loop）的问题。不要慌，首先把刚刚安装的NVIDIA驱动全部卸载干净。

```
sudo apt-get remove --purge nvidia-*

# reboot computer..
sudo reboot now
```

google搜到一些其他解决方案在这个情况下一般都没用。

1. 删除Xauthority。无用，此处问题的关键在于OpenGL。
2. **重装lightdm**或者**安装gdm**。无用，因为lightdm或者gdm均为显示管理，处于驱动的上层，驱动层出的问题无论重装其上层多少次都不会改变现状。
3. 安装NVidia-current。可能有用可能没用。实际上通过命令`sudo apt-get install nvidia-current`所得到的驱动是nvidia-304。不一定是当前机器真正需要用的，而且这个做法也可能覆写了一些其他内容，不推荐这样做。
4. 重装一个系统图形界面。有用但不好。有一些solution提出的是**弃用ubuntu改为使用lubuntu**，因为这个安装可以认为是系统层面的，那么驱动也会被覆写称为lubuntu的驱动，那么就可以继续使用了。然而，这个方案最大的缺点是，**界面丑**...强迫症患者请不要选择这个方案，否则最终会因为无法忍受界面丑陋而重装或者再次折腾。

当彻底卸载NVIDIA的驱动之后，实际上相当于回到的新装系统的状态（刚装机的时候就是没有任何NVIDIA驱动的，全凭系统本身的集成显卡驱动去输出界面。若本身在BIOS选择用独显输出的话，卸载驱动之后理论上应该就是完全无法显示图形界面了）

同样的，利用上述的方案二，实际上可以理解为CUDA Toolkit里面包含了驱动的`.run`包。那么也是注意根据独显和集显的情况，添加`--no-opengl-libs`高级选项。

### 3. CUDA安装

cuda的安装实际上可以大致上分为两种方式：

1. 通过`.run`包直接执行然后根据提示一步步往下安装。
2. 通过`.deb`包配合apt-get进行自动的安装以及配置。

实际上在cuda的[下载页面](https://developer.nvidia.com/cuda-downloads)有官方的指引，此处做一下搬运。

若以方式1安装，则

```
# You may need to use chmod +x for the run file
sudo sh cuda_8.0.44_linux.run
# Follow the command-line prompts
```

若以方式2安装，则

```
sudo dpkg -i cuda-repo-ubuntu1404-8-0-local_8.0.44-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```
此处稍微分析一下第一句command。我们可以尝试理解为`.deb`包是一个安装程序。我们通过dpkg把安装程序解压成为一个可执行的安装程序。然后在`sudo apt-get install cuda`的时候真正发起安装。

***依然，有一个地方要注意...***

通过deb包安装的方式，默认情况下，是不会禁用nouveau，也不会禁止openGL覆写。所以对于集显输出GUI的同学乃至部分独显输出GUI的同学，通过这个方式安装容易引起login loop的问题。那么，不慌，解决方案同上，卸载赶紧所有NVIDIA的驱动然后重启是可以恢复的。