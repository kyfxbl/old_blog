title: 不依赖storyboard实现ViewController跳转和传参
date: 2013-12-27 16:03
categories: iOS 
---
产品里原来的页面都是通过storyboard实现的，这2天开发新功能，顺便尝试了一下纯代码的实现，感觉更喜欢用代码实现的方式，本文就简单总结一下
<!--more-->

首先还是需要创建ViewController的实例，然后进行初始化，再把UIView挂到ViewController的view属性上

这个过程无论是否使用storyboard都是需要的，只是storyboard会自动完成这一步。用代码来实现的话，类似于：

```
YLSStepThreeViewController *doRegisterViewController = [[YLSStepThreeViewController alloc] initWithNibName:nil bundle:[NSBundle mainBundle]];    
YLSStepThreeView *stepThreeView = [[YLSStepThreeView alloc] initWithFrame:CGRectMake(0, 0, 540, 720)];
doRegisterViewController.view = stepThreeView;
```

创建好了ViewController，接下来就可以跳转了：

```
doRegisterViewController.modalPresentationStyle = UIModalPresentationFormSheet;
doRegisterViewController.modalTransitionStyle = UIModalTransitionStyleCoverVertical;

[self presentViewController:doRegisterViewController animated:YES completion:^(void){
    NSLog(@"done...");
}];
```
前面2行是设置弹出模态窗口的效果，最后一行是真正进行跳转

弹出页面之后，稍后关闭页面：

```
[self dismissViewControllerAnimated:YES completion:^(void){
    NSLog(@"done...");
}];
```
这个dismiss方法有点讲究，关键是每个ViewController的实例，都有presentingViewController和presentedViewController属性，搞清楚这2个属性很重要

比如ViewControllerA负责弹出了ViewControllerB，那么A的presentedViewController就指向B，而B的presentingViewController指向A：

![](http://img.blog.csdn.net/20131227155024343)

如果B的工作完成了，想关闭B回到A，应该在A上调用dismiss方法，而不是在B上调用。关于这一点，xcode的文档里说得很清楚：

Dismisses the view controller that was presented by the receiver.

The presenting view controller is responsible for dismissing the view controller it presented. If you call this method on the presented view controller itself, it automatically forwards the message to the presenting view controller.

此方法将会关闭dismiss消息接收者的presentedViewController，所以在A上调用dismiss方法，则会关闭B，因为A的presentedViewController指向了B

文档里也补充了，如果在B上调用此方法（出于错误？），则B会自动调用A上的dismiss方法，所以最后的结果是一样的，可能这个行为也引起了一些误解

但是，一个很常见的场景是，A打开了B，B打开了C，那么这时候在B上调用dismiss，会发生什么行为呢？试验了一下，只会关闭C，B本身并不会被关闭。也就是说，文档里提到的dismiss消息传递行为并没有发生。我猜想，或许是如果调用dismiss的ViewController的presentedViewController属性为nil，则会向上传递；如果不为空，那么就不传递了

总的来说，对于一个ViewController的序列（A打开B，B打开C，C打开D），___想要回到哪个Controller，就在哪个Controller上调用dismiss就行了___

最后，涉及到在ViewController里传参的问题，本来用storyboard处理页面跳转，传参的事情由prepareForSegue方法来搞定就行了，现在没用segue了，传参的方法也很多。比如用delegate模式，或者暴露出@property直接设值，都是可以的