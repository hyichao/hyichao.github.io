---
layout: post
title:  "Python在ubuntu上面的安装，遇到的问题，以及一些有用的python库安装"
tags: [Misc]
---

在linux环境下面，一般都配置了python环境。mac下面也是。

但是有时候用户会发现，系统自带的python版本比较旧，于是想要更新python版本。
例如我在自己的ubuntu下面有一个自带的python2.7。但是需要用到python3，于是我不得不重新安装一下python3。

如果从百度上面搜和python相关的东西，感觉真是错漏百出。原因就不说了。对于程序猿，有问题还是google好一点，免得火大。

```
sudo add-apt-repository ppa:fkrull/deadsnakes
＃添加一个源

sudo apt-get update
＃更新源列表，以获取最新的版本

sudo apt-get install python3
＃使用apt-get来安装
```

假如在系统中已经存在了python2，那么使用命令python的时候，自动会跳转到python2的版本。为什么呢？
其实是因为在/usr/bin目录里面，python这个“快捷方式”指向了python2。
例如我自己的电脑，就是本来自带一个python2.7，后来装了一个python3.4
如果不做下面的步骤，那么每次在terminal输入python，就会连接到2.7.
那么现在如果我要设置Python 3.4为python的默认命令，就

```
rm /usr/bin/python
ln -s /usr/bin/python3.4 /usr/local/bin/python
```

另外卸载Python 3.4命令：
`sudo apt-get remove python3.4`

安装了python，还不够。python有很多优秀的依赖库，如numpy, matplotlib, scipy，等等，都是学术研究以及其他领域中不可或缺的依赖。
例如matplotlib就提供了画图功能，很多曲线图可以通过编写一个python程序来实现。
为了安装这些东西，有很多人提供了优秀的工具。最有人气的就是pip。通过安装pip，可以方便的安装上述的依赖库

首先把python环境安装好。这个环境包括一些头文件，一些其他重要的基础的依赖文件。
`sudo apt-get install python3-dev`

下面安装pip

```
curl －o https://boostrap.pypa.io/get-pip.py
# 下载get-pip.py这个脚本
```

```
python get-pip.py
python3 get-pip.py （python3使用）
#通过这个脚本安装pip

#成功后就可以安装那些乱七八糟的库,如
pip install matplotlib
pip install scipy
```


* 注意几个地方！！！

首先，配置环境的时候，一定要严格搞清楚现在python使用的是哪一个版本。比如在使用caffe的python接口，貌似就一定要使用python2，不能使用python3，否则很多依赖库都用不了。

--------------------------------------------------------
今天因为python各种问题，一怒之下删除了python3，然后发现整个ubuntu都不好了
一查原因。看到原来ubuntu有很多software对python高度依赖，一旦卸载了python，相应的依赖python的software也都同时卸载掉。也幸好我只是删除了py3，py2还在，否则整个图形界面都用不了我就哭了。
恢复原来的ubuntu手段：

`sudo apt-get install ubuntu-minimal ubuntu-standard ubuntu-desktop`

这句命令重新安装了ubuntu初始状态下的software，然后把依赖库也一并装上（故py3也同时回来了。。）
目测这个命令也适合不小心删除了python2的情况，因为python2也会随着ubuntu-destop一起安装回来.
