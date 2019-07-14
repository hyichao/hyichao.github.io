---
layout: post
title:  "iOS中的GCD，Block和dispatch queue"
tags: [iOS]
---

今天稍稍用到了ios里面的多线程，看了一下相关的知识，文章和代码，感觉真的相当有用。

当一个app正在使用，有时候需要碰到大运算量的任务。假如这个任务是在主线程进行，那么用户不得不等待该任务完成再进行下一个动作。这时候，用户第一个想做的实情，就是关掉app。。。

于是，一个叫做multithreading的技术不得不出生。。。（成语是 应运而生？）有一篇英文的文章，讲的相当通俗易懂，只要不是英语渣渣，看一下绝对有益。

<http://www.raywenderlich.com/4295/multithreading-and-grand-central-dispatch-on-ios-for-beginners-tutorial>

应用多线程的话，大计算量的任务就会离开主线程，在不影响用户的后台（某个其他线程）进行，然后用户就会很爽，开发者也很爽，大家都很爽。

贴一段代码：

```
dispatch_queue_t queue=dispatch_queue_create("Doing Sth ",NULL);

dispatch_async(queue, ^(void){   
	[SomeObject DoSomething];
	[SomeObject DoOtherSomething];
  });
```
  
这里面就浓缩了三个东西的用法，是最常见的多线程处理某些方法（函数）的代码块。
首先dispatch_queue_t声明了一个对象queue，利用dispatch_queue_create来初始化。对了，注意要添加头文件。。
然后就是dispatch_async()，在这个方法里面写需要实现的代码，注意是要serial的实现，就是最传统的那种面向过程的方法，毕竟是queue，讲究先进先出。。
最后有个叫做block的语法，就是那个 ^(void){}的块。。姑且理解为一块东西，可以当做一个整体使用（所以叫做block？）比如当做一个参数传入其他方法，等等。
