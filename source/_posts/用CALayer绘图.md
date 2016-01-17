title: 用CALayer绘图
date: 2015-12-27 15:00:49
categories: iOS
---
在iOS中绘图，可以使用UIView，也可以使用CALayer。实际上，UIView也是由底层的CALayer完成绘制的工作
<!--more-->

# UIView和CALayer的关系

每个UIView内部都有一个CALayer对象，由它来完成绘制的工作。和view一样，layer也是一个树形的结构

当不需要自定义组件的时候，用UIView的API就足以胜任，把需要的子view通过addSubview()方法放到view的层次里即可；但是如果需要自己绘制一些图形，就需要在UIView的drawRect()方法或是CALayer的相关方法中，调用CoreGraphics的API来画图

跟几个朋友也讨论过这个问题，我认为用layer来画是更好的办法，因为相对于view，layer是更轻量级的组件，可以节省系统资源。同时layer是动画的基本单元，加动画特效也更容易。并且view负责响应手势等，把绘制的代码都放在layer里，逻辑上也更加清晰

但是需要注意，layer不能直接响应触摸事件，所以手势识别还是需要通过view来完成

# 在UIView中绘图

在UIView中绘图非常简单，当调用
```
self.setNeedsDisplay()
```

iOS系统会自动调用view上的drawRect()方法，可以在drawRect()方法中绘制图形

# 在CALayer中绘图

在layer中绘图，生命周期比view复杂一些

首先也是调用layer上的setNeedsDisplay()触发的

## display

首先会进入layer的display()方法，在这里可以把CGImage赋给layer的contents，那么会直接把该CGImage作为此layer的样式，不会进入后续的方法
```
// 绘图方法
override func display() {
        
    if let img = getFrameImage(wheelStyle) {
        contents = img.CGImage
    }        
}
```

## displayLayer

如果没有实现display()方法，或者调用了super.display()，并且设置了layer的delegate，那么iOS系统会调用delegate的displayLayer()方法

```
let myLayer : MyLayer = MyLayer()
myLayer.delegate = self;
myLayer.frame = bounds;
```

```
override func displayLayer(layer: CALayer) {

    if let img = getFrameImage(wheelStyle) {
        contents = img.CGImage
    }
}
```

## drawInContext

如果没有设置delegate，或者delegate没有实现displayLayer()方法，那么接下来会调用layer的drawInContext方法

```
override func drawInContext(ctx: CGContext) {
        
    CGContextSetLineWidth(ctx, 1);
    CGContextMoveToPoint(ctx, 80, 40);
    CGContextAddLineToPoint(ctx, 80, 140);
    CGContextStrokePath(ctx);
}
```

## drawLayerInContext

如果layer没有实现drawInContext方法，那么接下来就会调用delegate的drawLayerInContext方法

```
override func drawLayer(layer: CALayer, inContext ctx: CGContext) {
    CGContextSetLineWidth(ctx, 1);
    CGContextMoveToPoint(ctx, 80, 40);
    CGContextAddLineToPoint(ctx, 80, 140);
    CGContextStrokePath(ctx);
}
```

## 总结

所以，一般来说，可以在layer的display()或者drawInContext()方法中来绘制

在display()中绘制的话，可以直接给contents属性赋值一个CGImage，在drawInContext()里就是各种调用CoreGraphics的API

假如绘制的逻辑特别复杂，希望能从layer中剥离出来，那么可以给layer设置delegate，把相关的绘制代码写在delegate的displayLayer()和drawLayerInContext()方法。这2个方法与display()和drawInContext()是分别一一对应的