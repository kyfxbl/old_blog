title: iOS6下用Cordova弹出图片选择页面，造成应用crash
date: 2014-01-20 00:16
categories: iOS 
---
iOS6下cordova导致的crash问题
<!--more-->

我们的应用设置为只支持横屏（Landscape），在iOS6下，当通过Cordova插件弹出图片选择页面时，应用直接崩溃，错误信息：

Supported orientations has no common orientation with the application, and shouldAutorotate is returning YES'

研究了一下，本来是想覆盖这个方法：

```
- (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation
```
不过在文档里看到，这个方法在ios6和7里已经被deprecated，应该用下面2个方法来替代：

```
-(NSUInteger)supportedInterfaceOrientations
{
    return UIInterfaceOrientationMaskLandscape;
}

- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation
{
    return UIInterfaceOrientationLandscapeRight;
}
```

第一个方法是返回该页面支持的模式（横屏或竖屏），也可以用|进行或运算：

```
-(NSUInteger)supportedInterfaceOrientations
{
    return UIInterfaceOrientationMaskLandscapeLeft | UIInterfaceOrientationMaskLandscapeRight;
}
```

第二个方法则是返回优先的模式。注意，这个方法不支持用|进行或运算，只能返回唯一确定值，而且enum名是没有MASK的，虽然和第一个方法里的enum名看起来很像

另外，这2个方法并不是只覆盖一次就可以了，貌似在涉及到的ViewController里都需要写。我们的应用在MainViewController，CDVCamera插件，和一个第三方的图片裁剪组件里都覆盖了，否则都会造成应用crash