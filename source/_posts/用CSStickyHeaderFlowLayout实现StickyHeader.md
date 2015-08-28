title: 用CSStickyHeaderFlowLayout实现StickyHeader
date: 2015-07-21 00:25:09
categories: iOS
---
最近流行的一种界面效果，是瀑布流的header固定，也叫sticky header或者parallax。对于UITableView，可以比较方便地让table header固定，但是对于UICollectionView，原生的iOS API比较难以实现。本文推荐一个开源组件，专门用于实现这种效果：[CSStickyHeaderFlowLayout](https://github.com/jamztang/CSStickyHeaderFlowLayout)
<!--more-->

# 整体效果

贴个整体示意图

![sticky header](http://kypic.oss-cn-hangzhou.aliyuncs.com/blog_pic_1.jpg)

# 配合autolayout使用

首先需要注意的是，这个组件必须配合autolayout来使用。比如整个header分为4个部分，我想固定的是其中下面的2个subview，一开始我的代码是写死这2个subview的坐标

```
UILabel *header1 = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 375, 50)];
header1.backgroundColor = [UIColor yellowColor];
header1.text = @"1111";

UILabel *header2 = [[UILabel alloc] initWithFrame:CGRectMake(0, 50, 375, 50)];
header2.backgroundColor = [UIColor blueColor];
header2.text = @"2222";
```

这样的话就无法获得预期的效果，因为虽然整个Supplementary的height缩小了，但是subview的坐标却是固定的，所以不会随着header被推到顶部。正确的做法是使用autolayout，我这里用的是masonry

```
UILabel *header1 = [UILabel new];
header1.backgroundColor = [UIColor yellowColor];
header1.text = @"1111";
        
UILabel *header2 = [UILabel new];
header2.backgroundColor = [UIColor blueColor];
header2.text = @"2222";

[self addSubview:header1];
[self addSubview:header2];

[header1 mas_makeConstraints:^(MASConstraintMaker *make) {
    make.bottom.equalTo(header2.mas_top);
    make.left.equalTo(@0);
    make.height.equalTo(@50);
    make.width.equalTo(@375);
}];
        
[header2 mas_makeConstraints:^(MASConstraintMaker *make) {
    make.bottom.equalTo(@0);
    make.left.equalTo(@0);
    make.height.equalTo(@50);
    make.width.equalTo(@375);
}];
```

# 切换cell类型时，注意处理offset

当切换“投票”和“排行榜”的时候，并没有替换Layout，但是切换了不同的cell实现，所以2边的contentOffset是不同的，有可能出现某一组cell已经向下滚动了很多，而另外一组cell在这个位置上无法正确的显示。处理的办法是，如果offset已经超过某个值，则滚动到顶部

```
// 如果已经超过顶部，则滚动到顶部
if(myView.contentOffset.y >= 70){
    [myView setContentOffset:CGPointMake(0, 70)];
}
```

# 与MJRefresh的冲突解决

MJRefresh是另一个流行的下拉刷新组件，当CSStickyHeaderFlowLayout和它一起使用的时候，在下拉刷新时会出现2次奇怪的动画效果。原因是MJRefresh的实现是修改了UICollectionView的contentInset，而CSStickyHeaderFlowLayout在0.2.7版本没有正确处理这种情况。作者已经在0.2.8修复了此问题。issue的具体现象和处理过程在[ghost image issue#85](https://github.com/jamztang/CSStickyHeaderFlowLayout/issues/85)