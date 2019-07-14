---
layout: post
title:  "直接在iOS设备上运用theos开发Tweak"
tags: [iOS]
---

## Motivation
学习iOS逆向工程，入门阶段需要掌握许许多多工具。本文重点纪录我在使用theos初次写一个tweak所踩过的一些坑。

## Theos
Theos是iOS逆向开发的一个非常简洁有力的工具框架，能够快速构建app，dylib，tweak等模版。

> Theos is a cross-platform suite of development tools for managing, developing, and deploying iOS software without the use of Xcode. It is an important tool for people building extensions (tweaks) for jailbroken iOS; most extension developers use Theos.

引自 <http://iphonedevwiki.net/index.php/Theos>

利用theos进行开发，大部分情况下落在命令行环境中。一般有两种做法，一是在macOS上安装theos以及其它编译所需的工具和环境，编写程序之后进行编译打包安装，得到想要的可执行文件（Application的话就是.app文件，tweak的话就是.dylib文件，等等）；二是直接在iOS平台上安装theos，安装必须的编译环境以及进行配置，最后同样是进行编译打包安装。

## iOS上直接进行开发
接下来简单介绍如何在iOS上直接开发一个tweak。

参考了一个youtube视频
<https://www.youtube.com/watch?v=VwFCebaEnng>

基本思路如下：

* 若没有越狱设备，首先要进行越狱，这个非常简单，点赞盘古团队
* Cydia需要安装一些软件，这些软件可能需要一些特定的源，例如theBigBoss，CoolStar等。视频上所提到的coolstar源，实际上是为了后面的perl安装使用的。尽量准备好几个比较常见的源，如 BigBoss (本文必须) && CoolStar (本文必须)
* Cydia安装BigBoss Recommand Tool。根据搜索的资料，这个东西似乎只是一些常用工具的集合。那我猜测实际上也可以分别安装几个重要的工具，openSSH,MTerminal等。
* Cydia安装Perl。目前我还不清楚安装perl的作用，但是如果不安装或者版本过旧的话，在后面编译打包的时候会报错。必须安装CoolStar源的那个Perl5.22版本。
* Cydia安装iOS toolchain。这里面包括了clang等编译器，实际上等同于把编译环境安装到iOS平台上，是核心工具之一。若不安装这个，命令行直接找不到clang，也就是说编译不了。实际上后面有一个步骤是关于ldid配置的，也会报错。
* 打开MTerminal（关于如何打开iDevice的命令行，除了在真机上操作，也可以通过电脑ssh到设备上），从命令行安装 `installtheos3`。安装theos需要一段相对较长的时间，网速慢的同学请先去吃个饭。安装完成后，在`/var/`路径下能够找到`/theos`文件夹。
* 配置perl。此处配置perl是因为perl的默认安装路径改变了，从`/usr/bin`变为了`/usr/local/bin`。为了theos能够找到perl，就创建一个link到原来的路径即可。
* 配置64位。由于新的苹果设备均为64位系统，theos里面一些内容需要稍微修改了一下名字以支持64位的对应命名。需要修改的内容有：

```
file: /path/to/theos/bin/boostrap.sh
...
if [[ "$(uname -s)" == "Darwin" && "$(uname -p)" != "arm" ]]; then
		echo " Compiling native CydiaSubstrate stub..."
		make CydiaSubstrate target=native > /dev/null
fi
...
改为
...
if [[ "$(uname -s)" == "Darwin" && "$(uname -p)" != "arm64" ]]; then
		echo " Compiling native CydiaSubstrate stub..."
		make CydiaSubstrate target=native > /dev/null
fi
...
```
以及

```
/path/to/theos/makefiles/targets/Darwin-arm
重命名或者创建一个link
/path/to/theos/makefiles/targets/Darwin-arm64
```
以及
```
/path/to/theos/makefiles/platform/Darwin-arm.mk
重命名或者创建一个link
/path/to/theos/makefiles/platform/Darwin-arm64.mk
```

* 下载iOS SDK。若在正向工程里面，这个SDK就是XCode包揽了的任务，不过现在就只能我们手动下载了。网址为 <https://jbdevs.org/sdks/> 。找到最适合目前手头上的iDevice的版本，下载并解压到 `/path/to/theos/sdks/`即可。
* 配置ldid。ldid是iOS签名工具，没有签名App是无法发布运行的。这里的配置只是简单的两个

```
cd /usr/bin
ldid -s clang
ldid -s clang++
```
ldid是随着前面步骤的toolchain下载下来的，这两句相当于绑定一下签名和编译器。

* 使用theos里面的nic.pl创建模版。nic全称为new instance creator。运行`/path/to/theos/bin/nic.pl`,然后根据实际情况填入信息即可。
* 创建好了工程，cd进入后可以直接make package install。不过由于没有什么内容，也看不出什么东西。不过能够正常运行到这一步的话，说明前面的配置基本上都是好的。

### 《iOS应用逆向工程》的例子
配置好所有必备的工具后，就可以尝试仿照书本的例子，hook住SpringBoard的applicationDidFinishLaunching:来显示一个UIAlertView。

1. nic创建tweak实例。
2. 修改Tweak.xm

```
%hook SpringBoard

-(void) applicationDidFinishLaunching:(id)application
{

%orig;

UIAlertView* alert = [[UIAlertView alloc] initWithTitle:@"Hello World" message:nil delegate:self cancelButtonTitle:@"OK" otherButtonTitles:nil];
[alert show];

[alert release];
}

%end
```

3. 修改Makefile内容

```
ARCHS = armv7 arm64

include theos/makefiles/common.mk

TWEAK_NAME = regreeting
regreeting_FILES = Tweak.xm
regreeting_FRAMEWORKS = UIKit

include $(THEOS_MAKE_PATH)/tweak.mk

after-install::
	install.exec "killall -9 SpringBoard"
```

主要是添加了两句，一句是ARCHS指定为arm64，一句是应用UIKit框架。

4. 大功告成，接下来就是见证奇迹的时刻

```
make package install
```
睁大眼睛，看着你的越狱设备，突然桌面进程被关掉，然后出现小苹果，然后，出现了一个UIAlertView写着Hello World!


## Ref
<http://iphonedevwiki.net/index.php/Theos/Setup>
<https://www.youtube.com/watch?v=VwFCebaEnng>



