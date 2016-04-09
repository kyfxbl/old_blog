title: iOS8，UILabel实现圆角
date: 2015-05-18 17:27
categories: iOS 
---
iOS8绘制圆角的方式和iOS7有所区别
<!--more-->

直接通过layer的cornerRadius属性，设置圆角，发现在iOS8下显示错误（故意设置了红色边框，看得清楚）

![](http://img.blog.csdn.net/20150518172457670)

虽然边框是圆角，但是UILabel本身还是直角

在iOS8下，需要这样写：

```
cancelLabel.layer.cornerRadius = 5.f;
cancelLabel.layer.borderColor = [UIColor whiteColor].CGColor;// 设置成UILabel的背景色
[cancelLabel.layer setMasksToBounds:YES];// 关键
```
效果：

![](http://img.blog.csdn.net/20150518172641125)