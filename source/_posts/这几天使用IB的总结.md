title: 这几天使用IB的总结
date: 2015-12-11 00:21:10
categories: iOS
---
这几天尝试了使用Interface Builder，跟以前用纯代码开发还是有比较明显的区别，本文总结一下这几天的感受
<!--more-->

# 原理

总的来说，在IB里的操作，会在编译时由xcode生成代码，本质上和纯代码是一样的。但是通过IB的方式，可以用更少的代码实现同样的功能

比如说segue，实际上还是会生成pushViewController或者presentViewController的调用。以及各种组件的实例化，也都会生成init的调用等

以下总结一下IB里的各种操作，与代码的对应关系

# 实例化

在IB里拖出一个UITableViewController，那么xcode会自动创建TableView的实例，并调用相应的init方法。类似的，将一个ViewController嵌套在NavigationController里，xcode也会自动创建这2个实例，并将ViewController设置成rootViewController

纯代码实现：
```
HomeIndexController *vc = [[HomeIndexController alloc] init];
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:vc];
```

不仅如此，在ViewController的生命周期里，各种loadView，以及View的实例化，也都可以在IB里完成大部分

# 通过IBOutlet获取引用

IBOutlet解决的是获取引用的问题

比如ViewController需要拿到UILabel和UIButton的引用，纯代码的方式可能是：
```
@property UILabel *label;

MyView *view = (MyView*)self.view;
UILabel* theLabel = view.label;
```

而在IB里，只需要从storyboard里拖出一个IBOutlet到ViewController里就可以了

# 设置各种属性

比如要设置一个UIView可点击，纯代码的方式是：
```
view.userInteractionEnabled = YES;
```

或者其他的属性，如backgroundColor等都是类似的，在IB里可以直接设置
![](http://pic.kyfxbl.com/ib.jpg)

# 设置delegate和dataSource

也可以在IB里直接完成：
![](http://pic.kyfxbl.com/ib2.jpg)

# 通过IBAction设置交互行为

设置target-action也是非常常见的情况，纯代码方式：
```
[button addTarget:controller action:@selector(backButtonPress) forControlEvents:UIControlEventTouchUpInside];
```

另外一种是设置手势响应，纯代码方式：
```
[view addGestureRecognizer:[[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(onTap)]];
```

这2种情况都可以通过拉出一个IBAction来替代

# controller跳转

代码的方式一般是：
```
[navigationController pushViewController:controller animated: YES];
```
或者
```
[controller presentViewController:photoSlider animated:YES completion:nil];
```

而返回上一个controller，纯代码实现是：
```
[navigationController popViewControllerAnimated: YES];
```
或者
```
[self dismissViewControllerAnimated:NO completion:nil];
```

这种跳转的场景可以通过segue和unwind segue来替代

# 缩小IB中controller的面积

controller在IB里会特别大，如果用外接显示器还好，如果是用macbook开发，编辑storyboard的时候会特别吃力

其实可以双击空白区域，会把controller缩小，可视范围就大多了

# 单元测试特别方便

我手头的版本是xcode7.1.1，这个版本单元测试非常方便，只要继承XCTestCase就可以，点击左侧的小菱形就可以直接跑单元测试，比以前的版本方便很多，可以多写点单元测试了

# 关于布局和约束的问题

以前我一般是用masonry来写约束，还是比较熟练的。代码类似：
```
[phoneLabel mas_makeConstraints:^(MASConstraintMaker *make) {
            make.left.equalTo(@10);
            make.top.equalTo(@0);
            make.width.equalTo(@80);
            make.height.equalTo(@50);
        }];
        
        [_phoneInput mas_makeConstraints:^(MASConstraintMaker *make) {
            make.left.equalTo(phoneLabel.mas_right);
            make.right.equalTo(@-10);
            make.top.equalTo(phoneLabel);
            make.height.equalTo(phoneLabel);
        }];
```

基本上就是各种init和addSubview之后，开始用masonry来写约束

而在IB里，也是通过拖动的方式来做的，一种做法是在Pin面板里设置，另一种做法是直接在组件上control + drag，还可以在size inspector里编辑。反正挺麻烦的，我现在还不太熟练，基本上还是用masonry的思路在做

# IB的缺点

IB最大的优势当然就是节省代码，这是毋庸置疑的。但是IB也有劣势

首先是多人协作开发的时候，虽然xcode已经多次优化过，conflict问题始终不能很完美地解决。毕竟merge代码很容易，要merge XML文件实在是有点困难

其次是我仍然认为代码更加直观，比如一个控件的属性等，在代码里比较集中，而在IB里就要分散到各个地方去找了。有时候出现一些很诡异的问题，比较难定位。比如下面这个问题

# UIImageView约束失效的诡异问题

今天我拖出了一个UITableViewController，然后在自定义的cell里放了一个UIImageView，并且设置好了约束。结果我发现这个ImageView老是变形，检查了好几遍约束，始终没发现问题

最后在connections inspector里发现，我不知道什么时候不小心把这个ImageView拖了2个IBOutlet到自定义的Cell类里，于是xcode生成代码的时候就出错了。最后把2个Outlet都删掉，重新拖了一次就好了

像这种问题，真心难发现，如果是纯写代码，就不会发生这种事了