title: NSOperationQueue简介
date: 2014-01-08 20:21
categories: iOS 
---
iOS开发涉及到多线程时，一般都习惯用GCD。本文介绍稍微不那么常用的NSOperationQueue
<!--more-->

iOS上主要有3种多线程开发的方式：

1、NSThread以及基于它的performSelector方法

2、NSOperation和NSOperationQueue

3、GCD

我们的项目最早使用的是NSThread，后来全部换成了GCD，但是NSOperation就从来没试过，今天也稍微看了下

感觉还比较简单，一共只有2个类，分别是NSOperation和NSOperationQueue

NSOperation类似于java中的Runnable接口，需要实例化才能用，不过cocoa已经提供了2个现成的子类，NSInvocationOperation和NSBlockOperation，一般情况下，用这2种现成的就可以了，特殊情况才需要自定义。创建好NSOperation后，将其放到NSOperationQueue里就会运行了，示例代码：

```
NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(testMethod) object:nil];
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation];
```

指定的方法可以有一个参数，也可以有返回值，比performSelect稍微强点。另外利用addDependency方法，可以很方便地设置任务的依赖顺序：

```
NSInvocationOperation *operation1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(testMethod:) object:nil];
NSInvocationOperation *operation2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(testMethod2) object:nil];

[operation2 addDependency:operation1];// 这行代码保证了执行的先后顺序

NSOperationQueue *queue = [[NSOperationQueue alloc] init];
[queue addOperation:operation2];// 即使operation2先放入队列，也会在operation1之后执行
[queue addOperation:operation1];
```

感觉NSOperation的抽象层次比GCD更高，所以API调用更加简单

关于GCD和NSOperation的比较，下面2篇帖子都不错：

[GCD和NSOperation的比较](http://wiki.eoe.cn/page/iOS_pptl_artile_28129.html)
[使用NSOperation简化多线程开发](http://marshal.easymorse.com/archives/4519)