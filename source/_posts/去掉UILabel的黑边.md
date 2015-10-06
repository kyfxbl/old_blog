title: 去掉UILabel的黑边
date: 2014-07-21 20:33
categories: iOS 
---
当UILabel的size存在小数时，会出现黑边
<!--more-->

自定义了一个柱状图的组件，发现UILabel有一条黑边，检查代码发现：

```
CGFloat barLength = value * lengthPerValue;
UILabel *bar = [[UILabel alloc] initWithFrame:CGRectMake(80, 5, barLength, 30)];
```

关键在于barLength计算出来不是整数，结果可能是比如181.332这样的小数，所以graph hardware难以处理。解决的办法是用ceil或者round的函数，去掉小数部分就可以了

```
CGFloat barLength = ceil(value * lengthPerValue);
UILabel *bar = [[UILabel alloc] initWithFrame:CGRectMake(80, 5, barLength, 30)];
```