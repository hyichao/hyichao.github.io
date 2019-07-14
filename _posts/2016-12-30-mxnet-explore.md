---
layout: post
title:  "MXNet试用体验"
tags: [Deep Learning]
---

[**mxnet**](https://github.com/dmlc/mxnet) 是深度学习领域的主流框架之一，近段时间还成为了Amazon的AWS默认深度学习引擎（知乎上还说是Amazon的宫斗，然而吃瓜群众只关心实用性不关心背后的故事）。由于本人以前一直使用的都是caffe，故而本文或多或少会对[**caffe**](https://github.com/BVLC/caffe)和[**mxnet**](https://github.com/dmlc/mxnet)进行一定程度的比较。

## 安装
mxnet常常被吐槽的是文档欠缺和社区稍弱。但是在安装这一项上，mxnet对于新手是非常友好的，仅需要进行

```
git clone https://github.com/dmlc/mxnet.git --recursive
cd mxnet/setup-utils
bash install-mxnet-ubuntu-python.sh
```
官方文档还加了几个步骤，是修改编译选项的操作，主要是添加cuda编译和cudnn编译。install的安装脚本里面还有几个依赖项的安装，例如`jupyter notebook`用于画网络结构图（吐槽一下，这个图挺丑的，根本不实用..）

一般情况下，我们都只需要使用python接口，所以第一次编译项目的话，直接使用`install-mxnet-ubuntu-python.sh`或者根据里面的内容自己敲进命令行都可以。如果后续还需要重新编译项目，那就直接make就可以了。编译的过程相对来说比较漫长，即使用上了8线程也等了颇为漫长的一段时间，所以如果没有什么异常情况的话，就不要`make clean`了

### mnist实验
编译完成之后，我们一般就会跑一个mnist实验。在深度学习里面，跑一个mnist就像写一个hello world一样。

mxnet根目录下有很多个文件夹，其中有一个叫example，里面有多个不同的CV或者NLP任务的例子。进入其中的image-classification子目录，然后找到train_mnist.py，执行

```
python train_mnist.py -h
```
在mxnet中如果对于高层API的选项有疑问，可以使用`-h`来查看。example里面的`train_*.py`类脚本都可以如此使用。如果不在脚本后加入arg，训练过程会默认使用cpu，不存模型，仅进行默认的数据增强(data augmentation)。一般做实验我们需要用到的选项是如下两个：

```
python train_something.py \
	--gpus 0	\
	--model-prefix snapshot/something		\
```
分别是指定使用GPU进行计算，还有定时模型存档。懒得敲就写个脚本来使用。更高级一点的功能就是数据增强的部分。caffe原生只支持mirror操作和crop操作，mxnet提供了更加丰富的功能，包括：

1. `--random-crop` 常规的切图
2. `--random-mirror` 常规的镜像
3. `--max-random-h` 图像H通道的抖动，默认开启，抖动范围36
4. `--max-random-s` 图像S通道的抖动，默认开启，抖动范围50
5. `--max-random-aspect-ratio` 长宽比抖动
6. `--max-random-rotate-angle` 旋转
7. `--max-random-shear-ratio` （待查明，暂时理解为切掉图像的一部分）
8. `--max-random-scale`和`--min-random-scale`两个值配合使用控制图像尺寸的抖动，但是由于CNN网络一旦有全连接层，则要求图像输入一致，所以一般只能固定为1

通过观察系统资源调度，发现mxnet的augmentation是多线程的CPU操作，效率比较高。以前在caffe中想要实现类似的训练实时数据增强，一般都只能在data_layer中修改代码，不太容易实现多线程，估计是我太弱[摊手]。所以在mxnet数据增强的部分，加一秒，啊不，加一分。

回到正题，本节内容主要是想讲述如何跑起来一个mnist实验。好，找到train_mnist.py脚本，运行，讲完...

脚本里面有下载数据的代码，example里面有几个候选网络，一切都准备好了，完全就是傻瓜式操作...我不甘心！怎么能把本少爷当傻瓜！接下来，我要用自己的数据跑一个实验！

## 数据准备

为了完整跑出来一个实验，我决定用本人的研究课题“第一视角手势交互”的数据进行一个简单的手势分类实验。

首先是数据准备的流程。在caffe里面做分类任务，一般需要准备两个装图片的文件夹和两个文本用来标明图片对应的类别编号，如

```
img1.jpg 0
img2.jpg 1
img3.jpg 0
img4.jpg 2
```

通过调用convert_imageset可执行文件生成lmdb。而mxnet则是类似的调用一个工具，生成后缀为`.rec`的文件。

caffe源码里面对于读入label文本的部分逻辑比较生硬，例如直接用boost库的split来分割一个空格' '，例如仅读如一个int型的label，对multi-label的任务而言，使用者必须修改代码，这一点也是比较坑。mxnet读如文本的部分支持multi-label，数据格式大概是

```
id label label label ... label path/to/image
```
例如单label的话就是

```
3123 0 img1.jpg
4322 1 img2.jpg
5123 2 img3.jpg
```
第一个id我暂时不清楚用途，后面就是直接是label和图片路径。

#####更妙的是
对于简单的分类任务，mxnet提供了工具能够直接遍历一个文件夹，然后生成label文本。所以，使用者只需要按照类别把图片存在不同的子目录下面就可以了。

```
python im2rec.py -h
```
工具`im2rec.py`存与`/mxnet/tools/`下，同样可以查看选项。

```
python im2rec.py prefixname /path/to/your/data --list prefixname.lst --recursive --train-ratio 0.8 --test-ratio 0.2
```
上述的命令意思是递归地查看/path/to/your/data目录下面的所有子目录和图片，每个字目录分别给予一个label的编码，然后按照一定的比例划分，生成一个*.lst的列表。这个列表就可以用来生成我们的需要的`.rec`文件用于训练。

```
python im2rec.py prefixname /path/to/your/data --resize 224 --num-thread 8
```
第二个命令依然是im2rec.py，只是在这个时刻，`.lst`文件也已经生成好了，所以高级选项不同了。通过resize可以控制数据的短边尺寸（会保持长宽比的），numthread选项可以多线程读图加快rec文件的生成速度。这一点，还是mxnet加一秒。

## 网络训练

进行CNN训练需要两个部分，一个是定义训练过程的脚本`train_*.py`，一个是定义网络的symbol脚本`some_network_name.py`。思想上和caffe里面的solver.prototxt结合train_val.prototxt一致，只是换了语言...

随手抓一个`train_cifar10.py`，对代码做一定的修改。

```python
import os
import argparse
import logging
logging.basicConfig(level=logging.DEBUG)
from common import find_mxnet, data, fit
from common.util import download_file
import mxnet as mx

def use_dataset():
    data_dir="data"
    fnames = (os.path.join(data_dir, "train.rec"),
              os.path.join(data_dir, "test.rec"))
    return fnames

if __name__ == '__main__':

    (train_fname, val_fname) = use_dataset()

    # parse args
    parser = argparse.ArgumentParser(description="train",
                                     formatter_class=argparse.ArgumentDefaultsHelpFormatter)
    fit.add_fit_args(parser)
    data.add_data_args(parser)
    data.add_data_aug_args(parser)
    data.set_data_aug_level(parser, 2)
    parser.set_defaults(
        # network
        network        = 'tinynet',
        # data
        data_train     = train_fname,
        data_val       = val_fname,
        num_classes    = 12,
        num_examples   = 40675,
        image_shape    = '3,160,160',
        random_crop    = 0,
        # train
        batch_size     = 256,
        num_epochs     = 100,
        lr             = .01,
        lr_factor      = .1,
        lr_step_epochs = '50',
        # display
        disp_batches   = 50,
    )
    args = parser.parse_args()

    # load network
    from importlib import import_module
    net = import_module('symbol.'+args.network)
    print(net)

    sym = net.get_symbol(**vars(args))

    # train
    fit.fit(args, sym, data.get_rec_iter)
```
实际上需要修改的只有dataset的名字（或者是路径，如果不在data/下的话），以及parser.set_defaults里面的内容。学习率依然是一个比较难设定的参数。一开始做实验以为是哪里代码不对，一个简单的二分类问题都不收敛，后来发现是**默认学习率太大**。如果是caffe的话，可以看到loss值非常大，甚至会出现nan或者inf，但是在mxnet中只打印使用者设置的一个衡量指标，例如accuracy，而不会计算loss值，所以只能看到accuracy一直不变，无从判断发生了什么。接下来看看修改代码打印一下loss值会更加便于使用。这个点上，caffe加一秒。

定义网络的py脚本放在每个example的symbol文件夹里面，非常简单。假如需要自己定义一个网络，可以随手扒一个alex或者vgg然后修改修改,例如：

``` python
import mxnet as mx

def get_symbol(num_classes, **kwargs):
    input_data = mx.symbol.Variable(name="data")
    input_data = mx.symbol.BatchNorm(data=input_data, fix_gamma=True, eps=2e-5, momentum=0.9, name='bn_data')
    # stage 1
    conv1 = mx.symbol.Convolution(data=input_data, kernel=(3, 3), stride=(1, 1), pad=(1, 1), num_filter=32)
    relu1 = mx.symbol.Activation(data=conv1, act_type="relu")
    pool1 = mx.symbol.Pooling(data=relu1, pool_type="max", kernel=(2, 2), stride=(2,2))
    # stage 2
    conv2 = mx.symbol.Convolution(data=pool1, kernel=(3, 3), pad=(1, 1), num_filter=32)
    relu2 = mx.symbol.Activation(data=conv2, act_type="relu")
    pool2 = mx.symbol.Pooling(data=relu2, kernel=(2, 2), stride=(2, 2), pool_type="max")
    # stage 3
    conv3 = mx.symbol.Convolution(data=pool2, kernel=(3, 3), pad=(1, 1), num_filter=64)
    relu3 = mx.symbol.Activation(data=conv3, act_type="relu")
    pool3 = mx.symbol.Pooling(data=relu3, kernel=(2, 2), stride=(2, 2), pool_type="max") 
    # stage 4
    conv4 = mx.symbol.Convolution(data=pool3, kernel=(3, 3), pad=(1, 1), num_filter=64)
    relu4 = mx.symbol.Activation(data=conv4, act_type="relu")
    pool4 = mx.symbol.Pooling(data=relu4, kernel=(2, 2), stride=(2, 2), pool_type="max")        
    # stage 5 & 6
    conv5 = mx.symbol.Convolution(data=pool4, kernel=(3, 3), pad=(1, 1), num_filter=96)
    relu5 = mx.symbol.Activation(data=conv5, act_type="relu")
    conv6 = mx.symbol.Convolution(data=relu5, kernel=(3, 3), pad=(1, 1), num_filter=96)
    relu6 = mx.symbol.Activation(data=conv6, act_type="relu")
    # stage 7
    flatten = mx.symbol.Flatten(data=relu6)
    fc1 = mx.symbol.FullyConnected(data=flatten, num_hidden=256)
    relu7 = mx.symbol.Activation(data=fc1, act_type="relu")
    dropout1 = mx.symbol.Dropout(data=relu7, p=0.5)
    # stage 8
    fc2 = mx.symbol.FullyConnected(data=dropout1, num_hidden=32)
    relu8 = mx.symbol.Activation(data=fc2, act_type="relu")
    dropout2 = mx.symbol.Dropout(data=relu8, p=0.5)
    # stage 9
    fc3 = mx.symbol.FullyConnected(data=dropout2, num_hidden=num_classes)
    softmax = mx.symbol.SoftmaxOutput(data=fc3, name='softmax')

    return softmax
```

细心的读者会可能会发现，这个网络在data后面加了BatchNorm。关于[**BatchNorm**](http://jmlr.org/proceedings/papers/v37/ioffe15.pdf)，我一开始没有太多关注，仅知道大概是把输入属于变换为均值为0的高斯分布，从而减少数值溢出的风险。

***仔细思考了一下整个数值计算过程***
> 在大多数CNN设计中，输入都直接是图片像素值的范围(0,255)。网络初始化的时候，一般权重也是一个均值为0的高斯分布。这样的情况下，卷积过程中，某几个权重和某几个对应像素值相乘之和可能比较大，然后后面的卷积层池化层等等有可能“滚雪球”一样在前向过程中越滚越大（因为池化一般取max-pooling求最大值，激活一般取relu正数为线性，所以均不存在数值上界；假如存在起到归一化作用的层会停止这个“滚雪球”，例如激活取sigmoid，但是sigmoid激活会引起梯度弥散问题）。假如必须要在这种输入条件下进行训练，只能通过设置一个非常小的学习率来保护反向传播的梯度。这样整个收敛速度都非常非常慢。

那么，为了解决这个“数值爆炸”的问题，处理手段有好些。在caffe我一般的做法是在data层加上mean参数和scale参数，让输入的数值范围从(0,255)变换到(-1,+1)。假设输入数值本身就是一个均值为128的高斯分布，那么（没理解错的话）加上mean和scale就等价于BatchNorm起到的作用，能够保证数值处于合适的范围。mxnet中不支持对数据进行scale操作，假如通过修改代码实现的话，需要同时对data_augmentation的代码进行修改，否则随机噪声会成为巨大的错误。所以，在data后面加上BatchNorm是一个相对简单的实现。

关于BatchNorm除了阅读论文以外，还可以简单看一下[**这篇博客**](http://shuokay.com/2016/05/28/batch-norm/)，文风比较优雅。有一些博客提到BN一般使用在每个convolution层或fc层后面，我目前没有验证是否有这个必要性。也许大数据集上或者极深网络中需要吧。在小规模网络中我个人认为只放一个在data层后面基本足够了。

## 网络预测

预测过程就是一个单纯的前向过程。caffe进行前向就是在**extract_feature.cpp**的代码中扒出前向的部分，然后补上适合自己的输入和输出。根据经验，mxnet一定也存在类似的代码。简单搜索后发现这个代码名为`mxnet_predict_example.py`，放在/mxnet/python/mxnet目录下。修改输入图像的尺寸，修改数据的预处理部分，修改输出，指定定义网络的`.json`文件和储存权重的`.param`文件，基本上就可以使用了。特别提一下，mxnet储存网络结构的是一个json，储存权重的是一个二进制文件后缀是param,都放在开始训练的时候指定的路径中。

## 小结

花了一天左右时间基本上摸了一遍mxnet做分类任务的整个流程。接下来再摸索一下其他example，就基本上覆盖公司需要的所有业务了...由于对mxnet的模型储存还不够熟悉，暂时没法做相对高难度的工作（压缩什么的），但是目前看来对于做研究而言，其实mxnet还真的不错。再考虑一段时间要不要让师弟师妹转投mxnet...