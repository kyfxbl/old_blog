title: 自定义UITextField
date: 2014-06-24 17:15
categories: iOS
---
自定义UITextField
<!--more-->

目的是实现如下的效果：

![](http://img.blog.csdn.net/20140624164556687)

UITextField的leftView是自定义的UIView，其中：

1、包含一个居中显示的icon，并且上，左，下各有1px的间隙

2、左上和左下是圆角，右边没有圆角

# 居中展示icon

关键是leftView不是UIImageView，而是把UIImageView设置为subview

```
CGFloat height = frame.size.height;

UIView *leftView = [[UIView alloc] initWithFrame:CGRectMake(0, 1, height - 1, height - 2)];

UIImageView *icon = [[UIImageView alloc] initWithImage:[UIImage imageNamed:iconName]];
icon.frame = CGRectMake(height / 4, height / 4, height / 2, height / 2);
[leftView addSubview:icon];
```
上面的代码，使icon在leftView居中显示

# 制造1px间隙

top和bottom的间隙很容易做，只要令leftView的height比UITextField的height少2，然后y设置为1

左侧的间隙比较麻烦，首先将width设置为比UITextField的width少1。但是直接设置x为1无效，需要override以下方法：

```
-(CGRect) leftViewRectForBounds:(CGRect)bounds {
    CGRect iconRect = [super leftViewRectForBounds:bounds];
    iconRect.origin.x += 1;
    return iconRect;
}
```

# 制造部分圆角

制造所有的圆角比较简单，只需要：

```
self.layer.cornerRadius = 5;
```

但是要制造部分圆角，就比较麻烦：
```
UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:leftView.bounds byRoundingCorners:UIRectCornerTopLeft | UIRectCornerBottomLeft cornerRadii:CGSizeMake(5, 5)];
CAShapeLayer *maskLayer = [[CAShapeLayer alloc] init];
maskLayer.frame = leftView.bounds;
maskLayer.path = maskPath.CGPath;
leftView.layer.mask = maskLayer;
```

# 创建阴影

```
self.layer.shadowColor = [UIColor colorWithRed:132/255.0f green:214/255.0f blue:219/255.0f alpha:1.0f].CGColor;
self.layer.shadowOpacity = 0.5;
self.layer.shadowOffset = CGSizeMake(3, 3);
```

# 完整的代码

```
@implementation LosTextField

-(id)initWithFrame:(CGRect)frame Icon:(NSString*)iconName
{
    self = [super initWithFrame:frame];
    if (self) {

        CGFloat height = frame.size.height;

        UIView *leftView = [[UIView alloc] initWithFrame:CGRectMake(0, 1, height - 1, height - 2)];
        leftView.backgroundColor = [UIColor colorWithRed:132/255.0f green:214/255.0f blue:219/255.0f alpha:1.0f];

        UIImageView *icon = [[UIImageView alloc] initWithImage:[UIImage imageNamed:iconName]];
        icon.frame = CGRectMake(height / 4, height / 4, height / 2, height / 2);
        [leftView addSubview:icon];

        UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:leftView.bounds byRoundingCorners:UIRectCornerTopLeft | UIRectCornerBottomLeft cornerRadii:CGSizeMake(5, 5)];
        CAShapeLayer *maskLayer = [[CAShapeLayer alloc] init];
        maskLayer.frame = leftView.bounds;
        maskLayer.path = maskPath.CGPath;
        leftView.layer.mask = maskLayer;

        self.leftView = leftView;
        self.leftViewMode = UITextFieldViewModeAlways;
    }
    return self;
}

-(CGRect) leftViewRectForBounds:(CGRect)bounds {
    CGRect iconRect = [super leftViewRectForBounds:bounds];
    iconRect.origin.x += 1;
    return iconRect;
}

@end
```

```
self.username = [[LosTextField alloc] initWithFrame:CGRectMake(20, 150, 280, 40) Icon:@"login_username"];
```