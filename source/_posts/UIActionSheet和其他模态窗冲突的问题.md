title: UIActionSheet和其他模态窗冲突的问题
date: 2015-05-01 17:16
categories: iOS 
---
虽然iOS8引进了UIActionController，但是由于目前还需要兼容iOS7版本，所以还不能完全放弃UIActionSheet。而ActionSheet和其他模态窗有冲突，本文介绍解决的办法
<!--more-->

我们有一个界面用到了自定义的模态对话框。当用户点击某个按钮时，会弹出ActionSheet，然后选择ActionSheet的一项，会弹出一个模态的对话框。基本的思路是用一个透明的view直接add到UIWindow上，类似：

```
-(void) show
{
    [[UIApplication sharedApplication].keyWindow addSubview:self];
    timer = [NSTimer scheduledTimerWithTimeInterval:fadeDelay target:self selector:@selector(fade) userInfo:nil repeats:NO];
}

-(void) fade
{
    [self removeFromSuperview];
    [timer invalidate];
}
```

但测试发现，当ActionSheet自动dismiss的时候，会附带把模态对话框也关闭了，解决的方法是，不使用keyWindow属性，而是使用以下的代码：

```
[[[[UIApplication sharedApplication] windows] firstObject] addSubview:self];// 如果使用keyWindow，此模态框会跟UIActionSheet冲突
```