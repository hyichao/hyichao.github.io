---
layout: post
title:  "iOS从零开始使用protobuf"
tags: [iOS]
---

让我们一起打开下面这个链接

<https://github.com/alexeyxo/protobuf-objc>

在github上有protobuf-objc，其中的readme可以教会我们安装proto到咱们电脑里面。然后利用protoc，也就是protobuf的编译器可以编译.proto文件，生成一些.h和.m文件。
在移动App中，使用protobuffer可以做储存，可以做网络传输，可以干很多和数据打交道的事情。
最简单的，加入做一个APP，你要记录用户数据对吧？用户账号是？密码是？性别是？有没有女朋友？
为了记录这些数据到服务器，就需要合适的数据结构。有人说，为什么一定要用protobuf？为什么不用其他的如json？如xml？关于这个问题，请到stackoverflow，csdn等格调甚高的地方去寻找，去发现。。我要用的原因，就是我需要用，不用就会落后，就会挨打。。


* 首先是怎么安装protobuf这个工程。

(摘抄一段来自<https://github.com/alexeyxo/protobuf-objc>的文档)

>How To Install Protobuf

>Building the Objective-C Protobuf compiler
>
>Check if you have Homebrew

>`brew -v`

>If you don't already have Homebrew, then install it

>`ruby -e "$(curl -fsSLhttps://raw.githubusercontent.com/Homebrew/`install/master/install)"

>Install the main Protobuf compiler and required tools
>
```
brew install automake`
brew install libtool
brew install protobuf
```
(optional) Create a symlink to your Protobuf compiler.
`ln -s /usr/local/Cellar/protobuf/2.6.1/bin/protoc /usr/local/bin`
Clone this repository.
`git clonehttps://github.com/alexeyxo/protobuf-objc.git`
Build it!
`./build.sh`

什么？看不懂？没关系，本爷就是为了翻译才贴上的

首先，打开终端！

`brew -v`

：查看你的mac里面有没有装brew。brew是mac os里面，类似于ubuntu的apt-get的功能，都可以直接在终端输入命令然后安装程序。－v自然就是版本version的意思

`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

这一句半懂不懂，，大概就是利用curl工具访问那个url，然后在ruby环境下载安装brew

```
brew install automake
brew install libtool
brew install protobuf
```

就是利用brew下载安装了。protobuf就是我们想要的，另外两个是依赖库

`git clone https://github.com/alexeyxo/protobuf-objc.git
./build.sh`

从github下载protobuf－objc这个工程，build脚本里面做的是编译

注意：编译工程过程中，有可能会出现错误。别慌！看编译错误的提示。一般错误只是因为环境变量和路径没有配置好，少了一些东西，例如少了编译protobuf这个工程的依赖库，按照提示添加路径即可

* 有了工程以后，我们就可以开始测试一下怎么用protobuf了

打开Xcode！新建一个工程！
然后有两个方法把protobuf添加到你的工程里面，一个是直接添加，一个是利用cocoapod
强烈推荐后者，因为cocoapods能够很方便的管理第三方类库，以后人家的工程升级了，你只需要一行

`pod update`

就ok～

关于安装和使用cocoapod，属于另一个话题，看另一个博文
在Podfile添加下面这个句子

`platform :ios , 6.0`

`pod "ProtocolBuffers", "~> 1.9.7" `


在保存之后，到终端，cd到工程里面，
`pod install`

等一会，cocoapod就会帮我们添加好，以后我们就应该打开
project的workspace，因为添加了pod作为子工程。

还没结束
在你的工程里面，新建一个文件夹，命名假如叫Protobuf
在这个文件夹里面新建一个proto文件。例如要在本地储存用户信息，那么就新建一个user.proto
里面内容可以如下

```
package csdnblog;

message PBUser {

required string userId = 1;                       // 用户ID
optional string nick = 2;                         // 用户昵称
optional string avatar = 3;                       // 用户头像

optional string password = 7;
optional string email = 8;
optional string mobile = 9;                       // 手机号码
optional string qqOpenId = 10;                    // QQ ID
optional string sinaId = 11;                      // SINA UserID
optional string weixinId = 12;                    // WeChat UserID
}
```

上面这个例子包括了几个要素。
一个是包名。包的概念在object c里面没有，java里面有，c++里面namespace也是差不多意思。
顺便提一下，oc里面一般在库名前面添加两个字母，起的作用差不多就是包的作用，作为类的上一层组织结构。
例如官方的NS，例如AFNetworking这种第三方类库的AF。

回到正题。编写pb文件，第二个要素是message
一个message就是一个整体，里面有哪些必要的内容，哪些可选的内容。详细的proto语法随便一找一大把，就不啰嗦了。

写好了proto，接下来就是编译这个proto文件，protobuf－objc这个类库会编译声称一些源码，是读写proto数据的接口API。
编译命令如下：
先打开工程，建议单独新建一个文件夹作为输出路径，例如工程下新建文件夹Gen，用来放generate出来的.pb.h文件&&.pb.m文件
打开终端
cd到工程路径下
protoc --plugin=/usr/local/bin/protoc-gen-objc person.proto --objc_out="./Gen"

大功告成！

