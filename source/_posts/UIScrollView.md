title: UIScrollView
date: 2014-07-12 21:51
categories: iOS 
---
UIScrollView对滑动和缩放提供原生支持，API使用也非常方便，本文介绍一些相关知识
<!--more-->

# 最简单的用法

只要初始化UIScrollView，然后设置contentSize，再放入subview，就可以了。例：

```
UIScrollView *scroll = [[UIScrollView alloc] initWithFrame:rect];
scroll.contentSize = CGSizeMake(width, height);
[scroll addSubview: subview];
```

# 为什么内容无法滚动

在so和各种论坛上最常见的问题，就是为什么ScrollView无法滚动，一般都是因为没有设置contentSize，或者contentSize比UIScrollView自身的bound更小

基本上可以这么理解：UIScrollView是一个容器，其中放了subview。如果contentSize比UIScrollView的size还要小，那么不需要滚动就能一屏显示全，所以就不会产生滚动条

实际上，滚动的不是UIScrollView自身，而是它所容纳的subview

# 为什么drawRect中用CoreGraphics画的图形无法滚动

简单来说，因为滚动的并不是UIScrollView，而是它的content view也就是subview。如果CoreGraphics直接画在UIScrollView上就不能滚动，而是要画在subview上

下面是一个错误的例子片段：

```
@interface LosLineChart : UIScrollView

@end

@implementation

-(void) drawRect
{
    UILabel *label;
    [self addSubview:label];

    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextMoveToPoint(context, anchorPoint.x, anchorPoint.y);
    CGContextAddLineToPoint(context, anchorPoint.x, anchorPoint.y + maxHeight);
    CGContextStrokePath(context);
}

@end
```

上面的代码，LosLineChart自身就是ScrollView的实例，然后label是它的subview，因此label是可滚动的。而直线是直接画在LosLineChart上的，所以无法滚动

正确的做法应该是：

```
UIScrollView *scroll;
LosLineChart *chart;

scroll.contentSize = CGSizeMake(width, height);
[scroll addSubview:chart]; 
```

LosLineChart本身不是ScrollView，而是放进ScrollView里，这样用CG画的图形，也就可以滚动了。总之就是记住一句话：滚动的不是UIScrollView，而是它的subview

# paging

主页面是一个UIScrollView，左右滑动可以显示4个子页面

希望实现的效果是滑动一定距离，就显示完整的子页面。本来以为需要实现delegate方法，计算contentOffset的值再处理。但是那样逻辑比较麻烦，还需要判断滑动的方向

最后查阅文档，发现只需要设置paging的值就可以了

```
UIScrollView *dataArea = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 104, 320, 415)];
dataArea.contentSize = CGSizeMake(1280, 415);
dataArea.pagingEnabled = YES;
```

# 遍历subview的问题

我在UIScrollView里addSubview了几个自定义的view，然后在某些场景下遍历这些子view，然后调用reload方法。结果应用crash了，错误信息是：

```
unrecognized selector
```

原本觉得所有的subview都是我add的，应该不存在这个问题，后来debug发现，在for循环遍历scrollView的subviews时，某些对象类型是UIImageView，应该是UIScrollView默认就包含有一些subview

所以安全的代码应该增加类型检测：

```
for(UIView *subview in dataArea.subviews){

    if([subview conformsToProtocol:@protocol(ReportViewProtocol)]){
        [(id<ReportViewProtocol>)subview reload];
    }
}
```