title: iOS手势识别的几个问题
date: 2014-07-01 17:07
categories: iOS 
---
iOS手势识别的几个问题
<!--more-->

# 只让部分subview响应手势

比如一个view hierarchical下面，挂着3个subview，只想让其中的一个subview响应tap手势，有2种做法：

第一种方法，把UIGestureRecognizer挂到目标subview上

第二种方法，把UIGestureRecognizer挂到父view上，然后让另外2个subview不响应：

```
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer;
```

在这个delegate方法中return NO就可以了

# 让UILabel也能响应手势

默认情况下，UILabel和UIImage这些UIView无法响应手势，需要设置：

```
imageView.userInteractionEnabled = YES;
```

# 让手势和UITableViewCell点击共存

我在UITableView里挂了一个UIGestureRecognizer，结果发现手势识别覆盖了TableCell的触摸响应事件，需要设置“事件冒泡”

```
UITapGestureRecognizer *singleTap = [[UITapGestureRecognizer alloc] initWithTarget:controller action:@selector(hideSubViews:)];
[singleTap setNumberOfTapsRequired:1];
[singleTap setNumberOfTouchesRequired:1];
singleTap.cancelsTouchesInView = NO;
```