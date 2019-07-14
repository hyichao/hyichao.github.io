---
layout: post
title:  "解决CocoaPods各种慢的方案（gem换源+pod repo换源）"
tags: [iOS]

---

**本文重点讲述如何对Cocoapods进行换源，解决由于github服务器慢带来的各种install慢update慢问题，亲测有效。**

## 1. 安装cocoapods
由于太多太多的教程讲述了如何安装cocoapods，这里就略过，简单提供几个大牛博客的链接，又或者指尖看官方文档即可。

* 唐巧的博客

<http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/>

* 来自code4app的

<http://code4app.com/article/cocoapods-install-usage>


## 2. gem换源
由于（你懂的）的原因，使用ruby的gem安装cocoapods需要换源，否则就会被（你懂的）

```
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```
对于taobao的这个源，请大家暗自说一声感谢老马哥

## 3. pod repo换源
cocoa pods如果使用命令

`pod repo`

会出现以下的字眼（这里我已经换源了，本来是来自github的源）

```
master
- Type: git (master)
- URL:  https://gitcafe.com/akuandev/Specs.git
- Path: /Users/Charlie/.cocoapods/repos/master

1 repo
```

这个repo记录着许许多多第三方库的地址，默认选择了github作为源，假如开发者需要升级repo，实际上就是从github上面把一个庞大无比的地址list克隆到本地一个叫什么 .CocoaPods之类的隐藏文件夹里面。

使用cocoa pods的开发者，自然体会过以下几个命令是如何的慢


```
pod install
pod update
pod repo update
```

有人提供了一个解决方案，如下

```
pod install --verbose --no-repo-update
pod update --verbose --no-repo-update
```

实际上这两条命令是取消了repo的更新，从而变快了pod的速度。但是，假如开发者本地的repo真的已经过时了（就是第三方的地址list有点旧旧的），则无法逃避repo的更新，所以使用还是要pod repo update，依然是慢的不能忍。

所以，此处需要对pod的source换源
有大神提供了几个镜像，使用如下方法换掉repo的源

```
pod repo remove master
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
```

目前搜集到的可选源有
https://gitcafe.com/akuandev/Specs.git
http://git.oschina.net/akuandev/Specs.git
https://git.coding.net/hging/Specs.git
（实际上就是几个常见的git托管站都有哈哈哈哈）
假如打开Podfile，我们可以看到这么一条

```
source 'https://github.com/CocoaPods/Specs.git'
```

将这个也换为刚刚repo使用的源，否则依然会从Github上面clone东西

最后，完成了上述两处地方更改之后，就可以直接使用

```
pod install 
pod update
```

祝大家使用cocoapods轻松加愉快！
