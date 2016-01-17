title: 总结View切换的三种方式
date: 2014-07-21 21:25
categories: iOS 
---
app的交互，归根结底是view的切换。我总结view的切换总共有3大类
<!--more-->

# 切换ViewController

第一种方式是切换了整个ViewController，通过presentViewController和pushViewController这些API。这种方式，适用于模块间的切换，比较难保持view的状态

# 替换挂载的View

这种方式，代码类似于这样：

```
View1 *v1;
self.view = v1;

// 做一些事

View2 *v2;
self.view = v2;
```
这种方式，view的切换都在同一个ViewController内部完成

# 操作同一个View

有些场景，用这个方式很合适。比如若干个子view，作为同一个UIScrollView的subview，然后视情况滚动。还有一种情况，给view设置datasource和reload方法，datasource读取元数据之后，调用view的reload方法刷新视图

这种方式的好处，是可以实现一些比较好的效果。同时由于没有多次创建view的实例，资源的损耗也比较小