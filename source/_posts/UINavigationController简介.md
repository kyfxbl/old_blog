title: UINavigationController简介
date: 2013-12-30 20:50
categories: iOS 
---
前几天做了几个模态页面，对通过presentViewController方法来实现页面跳转已经比较熟悉了，但是通过另一个常见方式NavigationController跳转还不了解，所以今天也稍微试了一下，以后肯定也用得到，本文总结一下
<!--more-->

# 两种跳转方式的主要区别

主要的区别体现在2点：

第1个区别，presentViewController方法，本质上是用一个模态ViewController遮住原来的ViewController，但是可以设置新模态窗口的尺寸，所以不一定会把旧的ViewController完全遮住（如果不设置，默认完全遮住）。而NavigationController，则是管理着一个ViewController栈，不是用模态窗口遮来遮去，而是执行进栈和出栈的操作，非常类似android中的Activity Stack。在storyboard拉线设置segue的时候，可以选择push和modal的方式，其实就是对应这2种跳转

第2个区别，则是在视觉效果上，通过presentViewController方法，看不出导航的感觉。而NavigationController，则在页面的最上方，创建一个NavigationBar，可以看出明显的导航的关系

# NavigationController本身也是ViewController

所以它可以直接赋值给window的rootViewController变量。但是，与一般的ViewController不同的是，它同时还是其它ViewController的容器。下面是创建NavigationController的代码：

```
YLSFirstViewController *first = [[YLSFirstViewController alloc] initWithNibName:nil bundle:[NSBundle mainBundle]];
first.view = [[YLSFirstView alloc] initWithFrame:CGRectNull];
first.title = @"the first view";

UINavigationController *navController = [[UINavigationController alloc] initWithRootViewController:first];

self.window.rootViewController = navController;
```
前面3行创建一个普通的ViewController，然后调用initWithRootViewController方法，创建了NavigationController。前面说过，NavigationController管理着controller栈，所以需要一个root controller，也就是栈顶元素。在创建时也可以不设置root controller，稍后再push也是一样的，总之就是对栈的操作。最后把NavigationController而不是普通的ViewController设置为window的rootViewController。虽然都叫root，不过意思完全不一样

# 常用的API

主要就是各种push和pop方法，比如以下这些方法：

```
[self.navController pushViewController:secondViewController animated:YES];
```
```
[self.navController popViewControllerAnimated:YES];
```
```
[self.navController popToRootViewControllerAnimated:YES];
```
大部分看名字就知道是什么意思了，具体的可以查看xcode的文档。NavigationController的视图，最明显的区别就是顶部的NavigationBar

![](http://img.blog.csdn.net/20131230203948218)

默认情况下（无需编码），中间会显示当前栈顶元素的title，左侧会生成一个按钮，点击会使当前ViewController出栈（即退回上一个页面），右侧无按钮

# 自定义NavigationBar的按钮

如果默认的NavigationBar不满足需求，那么可以自定义，覆盖默认行为。示例代码如下：

```
UIBarButtonItem *leftButton = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemAction target:self action:@selector(leftBarButtonPressed:)];
self.navigationItem.leftBarButtonItem = leftButton;

UIBarButtonItem *rightB1 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCancel target:self action:@selector(ohoh)];
UIBarButtonItem *rightB2 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCompose target:self action:@selector(ohoh)];
UIBarButtonItem *rightB3 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemEdit target:self action:@selector(ohoh)];
UIBarButtonItem *rightB4 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemDone target:self action:@selector(ohoh)];
NSArray *buttonArray = [[NSArray alloc] initWithObjects:rightB1, rightB2, rightB3, rightB4 ,nil];
self.navigationItem.rightBarButtonItems = buttonArray;
```
代码没调整过，不太规整，只是意思意思。。效果类似：

![](http://img.blog.csdn.net/20131230204814187)

NavigationController基础的用法就是这些，更细节的还没用到，暂时不研究。总的来说，就是一个可以作为容器的ViewController，管理着ViewController stack，最主要的作用，是在页面的上方展示导航栏