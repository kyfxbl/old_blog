title: 用inline-block实现两列布局兼容移动端
date: 2014-11-10 18:23
categories: web 
---
用inline-block实现两列布局是一种常见做法，不过需要处理手机浏览器的兼容性问题
<!--more-->

下面这个DOM结构

```
<div>
    <div>div1</div>
    <div>div2</div>
</div>
```

使用inline-block的方式实现2列布局：

```
div {
    font-size: 0;
}

div > div {
    display: inline-block;
    width: 50%;
    font-size: 14px;
}
```

虽然在PC上可以解决1px间隙的问题，但是在很多手机浏览器上（android 4.2以下），会有兼容性问题。右边的div会掉到下面

所以更好的办法是：

```
div {
    overflow: hidden;
}

div > div {
    float: left;
    width: 50%;
}
```