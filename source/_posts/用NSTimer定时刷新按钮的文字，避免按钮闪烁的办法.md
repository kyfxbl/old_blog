title: 用NSTimer定时刷新按钮的文字，避免按钮闪烁的办法
date: 2013-12-27 21:03
categories: iOS
---
总的思路是在Button上罩一个Label，原来对Button样式的操作，都可以改为操作Label
<!--more-->

今天做一个功能，一开始将某按钮置灰，然后倒计时60秒。每秒钟都刷新按钮的文字，倒计时结束后，使按钮可用

很快就做好了，不过发现一个问题，就是按钮会闪烁，跟星星似的。我的代码是：

```
NSString *text = [NSString stringWithFormat:@"(%d)重发验证码", countDown];
[resendButton setTitle:text forState:UIControlStateDisabled];
```

在网上搜索了一番，说是用NSTimer刷新按钮文字就是会有这个现象

后来想了一个办法，就是在UIButton上罩一个同样大小的UILabel，然后每次刷新UILabel的文字，不刷新按钮，效果不错，不再闪烁了

```
self.resendCodeButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
self.resendCodeButton.frame = CGRectMake(280, 280, 150, 50);
self.resendCodeButton.enabled = NO;// 默认不可用
[self.resendCodeButton addTarget:self.controller action:@selector(resendCodeButtonPressed) forControlEvents:UIControlEventTouchUpInside];

self.resendCodeLabel = [[UILabel alloc] initWithFrame:CGRectMake(280, 280, 150, 50)];
self.resendCodeLabel.textAlignment = NSTextAlignmentCenter;
```

上面的代码，Label的frame和Button完全一样，后添加到view中，这样就完全盖在Button上面

```
NSString *buttonTitle = [NSString stringWithFormat:@"(%d)重发验证码",resendCountdown];
resendLabel.text = buttonTitle;
```