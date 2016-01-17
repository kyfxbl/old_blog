title: iPhone dismiss keyboard
date: 2014-06-12 22:41
categories: iOS 
---
在iOS7.1环境下，UITextField的键盘不会自动收起来，最后在stackoverflow上找到办法
<!--more-->

首先要有View或者ViewController实现UITextFieldDelegate

```
@interface LoginView : UIView<UITextFieldDelegate>
```

# 按下键盘的return键，收起键盘

```
-(BOOL) textFieldShouldReturn: (UITextField *) textField
{
    [textField resignFirstResponder];
    return YES;
}
```

# 触摸屏幕其他位置，收起键盘

```
{
    id currentResponder;
}
```

```
-(void) textFieldDidBeginEditing:(UITextField *)textField
{
    currentResponder = textField;
}
```

```
UITapGestureRecognizer *singleTap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(resignOnTap)];
[singleTap setNumberOfTapsRequired:1];// 触摸一次
[singleTap setNumberOfTouchesRequired:1];// 单指触摸
[self addGestureRecognizer:singleTap];
```

```
-(void) resignOnTap
{
    [currentResponder resignFirstResponder];
}
```
# 原理

关键就是resignFirstResponder方法，但是触发时机不同

第一个场景，触摸键盘的return键时，会调用delegate的方法，然后调用resignFirstResponder

第二个场景，在整个view上注册了手势识别，当触摸到屏幕的其他位置时，调用resignFirstResponder