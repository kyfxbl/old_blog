title: 纯CSS实现高度与宽度成比例的效果
date: 2014-11-06 21:21
categories: web 
---
今天学到一个很不错的CSS技巧：一个两列比例布局，比如左边栏80%，右边栏20%，所以右边栏的width是不知道的，但是无论右边栏的width是多少，都要求height和width保持1：1的比例。不需要js，用纯css也可以实现这个效果
<!--more-->

原贴：[Zihua Li](http://zihua.li/2013/12/keep-height-relevant-to-width-using-css/)

原理主要是2点：

1、当设置padding为百分比的时候，参照是父元素的width，这点和设置子元素的width行为是一致的。比如这个DOM：

```
<div style="width: 100px">
    <div id="son">abc</div>
</div>
```

```
#son {
    width: 20%;
    padding-bottom: 20%;
}
```

计算出son的width和padding-bottom都是相对于parent的20%，即都是20px，这样就可以实现高宽比的任意比例

2、overflow的计算包括了content和padding，所以只要padding够大，即使content是0，内容一样可以显示出来

所以，结合以上2点，就可以用纯CSS实现高度宽度固定比例的效果：

```
#son {
    width: 20%;
    padding-bottom: 20%;
    height: 0;
}
```