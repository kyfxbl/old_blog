title: core graphics path
date: 2014-07-13 11:57
categories: iOS 
---
当UIKit无法满足绘图需求的时候，就需要用到Core Graphics API，其中最普遍的就是path
<!--more-->

# graphics context

可以理解成canvas，在ios里对应CGContextRef类型，拿到它的方法是调用这个函数：

```
UIGraphicsGetCurrentContext()
```
graphics context有很多种，可以分别将图形绘制到bitmap，PDF，UIView里。最常见的当然就是往UIView里绘制，做法就是覆盖UIView的drawRect:方法，然后调用上面这个函数，就得到了针对UIView的graphics context。由于初始化的工作UIKit已经完成了，所以开发者可以立刻绘制图形，不需要额外配置

标准的quartz 2D，context的坐标系原点在左下角。但是UIKit已经自动转换了，原点移到了左上角，与UIView的坐标系保持一致

# graphics state

保存绘制参数，比如线条的粗细，颜色，样式等等，完整的参数列表可以看apple的官方文档。graphics state是一个stack数据结构，可以用以下函数执行push和pop的操作：

```
CGContextSaveGState()
CGContextRestoreGState()
```

如果不需要暂存state状态后面继续使用的话，可以直接调用CGContextSetXXX函数，即时设置状态，比如：

```
CGContextSetLineWidth()
CGContextSetStrokeColorWithColor()
```

# path的2段式绘制

绘制path分2个阶段，分别是path创建和path绘图

创建path用到的函数有很多，比如addLineToPoint，addRect，addArc等，这些函数只是创建了path和它的subpaths，并不会实际画到graphics context上

创建path之后，需要调用fill和stroke函数，把当前的path画出来，绘制的函数包括：

```
CGContextStrokePath()
CGContextFillPath()
CGContextDrawPath()
```
strokePath和fillPath都属于fluent function，如果需要同时stroke和fill，那么应该调用第三个函数，然后将绘制mode设置为既stroke又fill

初学者一个常见的问题是，创建了一段path之后，先调用strokePath()，再调用fillPath()，为什么fill没有生效。因为无论是fill还是stroke，调用之后都flash了缓冲区，之前已经绘制好的path就结束了，所以后调用的函数就不会生效。正确的方法是调用drawPath

# 一次只能绘制一个path

创建path一般是从调用这个函数开始：

```
CGContextBeginPath()
```
调用这个函数，标识着开始创建一个path。但是如果只有一个path，或者paint之后再次创建path，其实这个函数也不需要调用。一般这个函数是和CGPathRef配合使用的，只有需要暂存一个path，后续继续使用的场景下，才需要调用这个函数

但是需要了解“一次只能绘制一个path”这个概念，比如下面的代码：

```
CGContextMoveToPoint(context, 100, 100);
CGContextAddLineToPoint(context, 200, 100);

CGContextBeginPath();

CGContextMoveToPoint(context, 100, 200);
CGContextAddLineToPoint(context, 200, 200);

CGContextStrokePath(context);
```
先创建了一个path，然后又创建了第2个path，最后调用stroke方法。只有第2个path会被画出来，因为graphics context每次只会画出“当前的”path。上面的代码，第一个path永远也绘制不出来，等于是丢失了

# subpath

但是这并不意味着path不能绘制复杂的图形，因为一个path可以包含任意subpath。调用fillPath，strokePath，beginPath函数，都会开始一个新的path。但是在调用之前，可以添加任意个subpath，比如addLineToPoint，addRect等函数，都会添加subpath到当前的path中，下一次paint的时候，会把所有的subpath都画出来

# path闭合

graphics context会始终维护一个current point，创建path的第一步，就需要调用

```
CGContextMoveToPoint()
```
这样context就获得了第一个当前点，然后当调用addLineToPoint时，current point就会自动移动，从而绘制出连续的线条。如果想要画不连续的线条，就再次调用CGContextMoveToPoint，改变current point的位置。这个方法创建的是subpath，不会创建新的path

创建若干line之后，可以调用CGContextClosePath，创建出一个封闭的区域，对后续的stroke和fill都有效

# 抗锯齿

调用下面的2个函数，可以设置绘制的图形有抗锯齿效果：

```
CGContextSetShouldAntialias(context, YES);
CGContextSetAllowsAntialiasing(context, YES);
```