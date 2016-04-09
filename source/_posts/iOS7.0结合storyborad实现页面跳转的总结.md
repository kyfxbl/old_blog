title: iOS7.0结合storyborad实现页面跳转的总结
date: 2013-11-27 20:10
categories: iOS 
---
折腾了一整天，本文总结一下ios7.0页面跳转有关的内容
<!--more-->

# storyboard的潜规则

我接触ios很晚，环境已经是xcode5+ios7，所以对以前的IOS开发模式并不了解。在网上查阅了很多资料，发现以前的代码，很多都需要自己coding来创建ViewController，比如：

```
WTwoViewController *controller = [[WTwoViewController alloc]initWithNibName:@"WTwoViewController" bundle:nil];
[self presentViewController:controller animated:YES completion:nil];
```

但是用storyboard来管理view controller的话，storyboard会自动处理view controller的初始化，所以就不再需要自己coding来创建view controller的实例。在另外一篇博客里看到这句话：

用过xib的人我相信很多人都会经常用到-presentModalViewController:animated:以及-pushViewController:animated:这两个方法。这种代码在storyboard里将成为历史；取而代之的是Segue

# 基于控件的跳转

用storyboard做开发，经常需要拉线，本文不介绍，请看这篇官方文档：

[start developing iOS app today](https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOS/index.html#//apple_ref/doc/uid/TP40011343)

这种拉线，是从button拉到view controller：

![](http://img.blog.csdn.net/20131127180201046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这种方式只要点击了这个button，就会自动跳转，不需要写任何代码

# 直接从controller到controller

这种拉线是直接从View Controller到View Controller：

![](http://img.blog.csdn.net/20131127180652984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这种方式已经预先创建了segue，但是还需要手工编码，首先需要给segue设置一个identity

![](http://img.blog.csdn.net/20131127180923625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后写代码来跳转：

```
// 跳转到bootstrap
- (void) jumpToBootstrap{
    [self performSegueWithIdentifier:@"fromWelcomeToBootstrap" sender:self];
}
```
这段代码必须写在-viewDidAppear里，不能写在-viewDidLoad里，否则会报一个错误：whose view is not in window hierarchy

# 页面之间传值

以前用-presentModalViewController:animated:方法来跳转的时候，一般需要通过delegate等方式来传值，现在一律用segue API就能搞定

在segue发生之前，先会调用当前View Controller的-prepareForSegue:sender:方法，可以在里面做一些处理，比如：

```
BootstrapViewController* targetController = [segue destinationViewController];// 拿到目标view controller，然后要怎么样都可以了
```

不过这里要注意的是，似乎不能在prepareForSegue方法里设置destination view controller的view，因为这个时候view还没有被storyboard实例化。不过可以先传参，后面再设置

另外网上看到很多帖子，都说从B回到A的时候如果也需要传值，可以把A设置成B的delegate：

```
BViewController.delegate = self;
```
这里我不是很理解，当从B回到A的时候，也设置一个segue似乎就行了

# unwind segue

unwind segue比较特殊，是在目标View Controller里先设置一个action，然后在source View Controller里拖线到exit图标上。这种情况下，除了会调用source的prepareForSegue方法以外，target View Controller的那个action也会被调用。详见：[ios unwind](http://blog.csdn.net/kyfxbl/article/details/16987189)