title: Masonry2个小技巧
date: 2015-07-26 22:27:45
categories: iOS
---
本文介绍使用Masonry时，怎么得到真实frame，以及如何实现动画
<!--more-->

# 获取真实frame

在不用Masonry时，frame在创建UIView时就已经确定，例如：
```
CGRect frame = CGRectMake(0, 0, 375, 200);
UIView *view = [[UIView alloc] initWithFrame:frame];
```
或者
```
UIView *view = [[UIView alloc] init];
view.frame = CGRectMake(0, 0, 375, 200);
```
但是使用Masonry时，坐标是根据约束计算得到的。在init方法里，view的frame是(0, 0, 0, 0)，这不是view的真实frame。有时候会遇到问题，比如希望给view加一个底部的border
```
[self addOneSidedBorderWithFrame:CGRectMake(0, self.frame.size.height-height, self.frame.size.width, height) andColor:color];
```
所以，需要能够获取到view的真实frame，正确的时机应该是在layoutSubviews方法里，因为这时候应该开始layout subviews，Masonry已经计算出了真实的frame
```
-(void) layoutSubviews
{
    [super layoutSubviews];
    [uploadArea addBottomBorderWithHeight:1.f andColor:[ColorUtility basicGrayLineColor]];
    [myUploadArea addBottomBorderWithHeight:1.f andColor:[ColorUtility basicGrayLineColor]];
}
```

# 实现动画效果

用以下的代码可以配合Masonry实现动画
```
[switchBar mas_remakeConstraints:^(MASConstraintMaker *make) {
    make.right.equalTo(@0);
    make.bottom.equalTo(@0);
    make.width.equalTo(toupiaoBtn);
    make.height.equalTo(@2);
}];
        
[UIView animateWithDuration:0.3 delay:0.0 options:UIViewAnimationOptionCurveLinear animations:^{
    [switchBar layoutIfNeeded];
} completion:nil];
```