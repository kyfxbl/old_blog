title: iOS7应用隐藏状态栏
date: 2014-01-24 16:53
categories: iOS 
---
iOS7中隐藏状态栏的方法也有变化
<!--more-->

在iOS7之前，在AppDelegate里用这行代码就可以隐藏状态栏：

```
[[UIApplication sharedApplication] setStatusBarHidden:YES];
```

但是在iOS7下，这行代码不生效，需要先在项目的plist文件里增加一个配置：
```
<key>UIViewControllerBasedStatusBarAppearance</key>
<false/>
```
或者用图形化页面添加

View controller-based status bar appearance，设置为NO，效果是一样的

配置了这个选项之后，上面那行代码就可以隐藏status bar了

但是，如果应用里用到了UIImagePickerController，在弹出照片选择界面的时候，状态栏又会跑出来，解决的办法是：

先声明Controller实现UINavigationControllerDelegate协议，然后设置为ImagePickerController的delegate

```
UIImagePickerController *imagePicker = [[UIImagePickerController alloc] init];
imagePicker.delegate = self;
```
然后实现此方法：

```
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    [[UIApplication sharedApplication] setStatusBarHidden:YES];
}
```

因为ImagePickerController是继承自UINavigationController的