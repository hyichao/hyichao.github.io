---
layout: post
title:  "Qt在Linux环境下如何进行动态链接 i.e. Caffe+Qt"
tags: [Qt]
---

## Motivation
利用Qt进行c++的GUI开发，在我看来是所有GUI方案中，最有吸引力的一个。
一方面，Qt的API封装的特别好，很接近其他“先进”的UI框架，就算是进行大规模的程序开发，Qt也能够胜任。
另一方面i，Qt Creator在构建工程方面，原来很是方便。我一开始不太了解链接库方面的只是，于是拼命找轻量级的GUI框架。用了一下FLTK，和Qt一比简直就是渣渣。下面简单记录一下Qt如何配置工程的动态链接库。

## Install
安装Qt的流程随便google一下就可以找到。简单说下。

1. google Qt
2. download Qtxxxxxx.run
3. go to terminal, sudo chmod -x Qtxxxxx.run （+x意思是增加execute权限）
4. follow install UI

## Detail
安装完成之后，新建一个工程，找到.pro文件。
假设：
现在有一个动态链接库名为  libXYZ.so， 路径为/path/to/the/lib，
该动态链接库的头文件为  XYZ .h，路径为/path/to/the/include
则应该添加如下内容到.pro文件

```
INCLUDEPATH += /path/to/the/include
LIBS +=  -L/path/to/the/lib
LIBS += -lXYZ
```
### 理解：

1. 符号 -L 与 符号 -l 区别： 大写 L 代表着动态库（.so文件）所在的路径；小写 l 代表动态库的名字
2. 动态库的命名： 在Linux下面，动态库命名规则是 libXXXX，即lib是必须的前缀。则在添加LIBS项的时候，可以省略, 即 -lXYZ实际上就等于 libXYZ
3. Qt的qmake实际上实现的是把这个.pro文件编译成为Makefile文件。假如打开工程文件夹并找到Makefile，可以看到我们在.pro文件添加的内容已经添加到Makefile里面的相应位置。
4. .pro文件要求使用绝对路径，故不能像shell脚本一样使用一些变量。但是有一个小trick，可以利用PWD获得.pro文件所在的路径，也可以某种程度上实现相对路径。例如：

```
INCLUDEPATH += $$PWD/include
```

## Example
下面举一个例子。这个例子也许有很多搞机器学习的同学有同样需求
使用CAFFE深度学习框架进行某种detection的试验，然后想直接利用caffe的代码进行前向，并接入后续的工程，构建一个UI界面，该如何做呢？
这个例子除了要把caffe生成的.so文件链接到Qt工程，还需要把CAFFE的依赖添加到工程，否则caffe也无法执行
.pro文件如下


```
#-------------------------------------------------
#
# Project created by QtCreator 2015-11-11T22:18:07
#
#-------------------------------------------------

QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

TARGET = xxxxxx
TEMPLATE = app


SOURCES += main.cpp\
        mainwindow.cpp \
    argmanager.cpp \
    imageprocessor.cpp \
    statuscontroll.cpp

HEADERS  += mainwindow.h \
    argmanager.h \
    common.h \
    imageprocessor.h \
    statuscontroll.h

FORMS    += mainwindow.ui


# adding dynamic links of CAFFE and its dependencies

# caffe
INCLUDEPATH += /home/somebody/caffe/include  /home/somebody/caffe/build/src
LIBS += -L/home/somebody/caffe/build/lib
LIBS += -lcaffe

# cuda
INCLUDEPATH += /usr/local/cuda/include
LIBS += -L/usr/local/cuda/lib64
LIBS += -lcudart -lcublas -lcurand

# opencv
LIBS += -lopencv_core -lopencv_imgproc -lopencv_highgui

# other dependencies
LIBS += -lglog -lgflags -lprotobuf -lboost_system -lboost_thread -llmdb -lleveldb -lstdc++ -lcudnn -lcblas -latlas

# 注意路径不能照搬，要根据具体情况具体分析哦！
```