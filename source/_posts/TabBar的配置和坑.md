title: TabBar的配置和坑
date: 2014-06-19 23:42
categories: iOS
---
iOS6中对TabBar的配置比较自由，但是到iOS7里多了很多限制。因此有些效果在iOS7里实现起来比较麻烦，本文简单总结一下
<!--more-->

容易实现的：
```
[[UITabBar appearance] setBarTintColor:color];// 背景色
[[UITabBar appearance] setSelectedImageTintColor:[UIColor whiteColor]];// 选中图标颜色
```
TabBar默认的背景色是灰色的，但是可以调；被选中的TabBarItem的颜色默认是蓝色的，这个也可以调

下面是一些默认的风格，而且比较难通过代码自定义，建议就遵循apple的规范好了，否则会很费力

1、未选中的TabBarItem的图标颜色，默认是灰色的

2、选中TabBarItem之后的背景颜色，就是BarTintColor。但是如果希望选中之后，有一个特殊的颜色，很难做到

3、TabBarItem，图标的尺寸。自己设置大小很难，虽然可以设置Insets，但是会有一个重复点击收缩的BUG，所以不如让美工提供一个合适尺寸的图标

4、TabBarItem，默认上面是image，下面是title，都预留了位置。如果想去掉title，把空间留给上面的image，也比较困难

总的来说，相对于NavigationBar来说，TabBar的自定义要麻烦很多，尽量不要找麻烦