title: css笔记
date: 2014-04-24 21:23
categories: web 
---
css的一些学习笔记
<!--more-->

# 文档流和盒子模型

1、html文档流，默认从上往下，从左往右地解析

2、DOM由各种盒子组成，大致上分为block盒子和span盒子

block默认会把宽度撑满自己的父元素；span盒子正好相反，会尽量把宽度缩紧到刚好够包裹内容

盒子的width由4个部分组成：content，padding，border，margin

3、css选择器，主要有3种，基于文档结构，基于id和class，基于property

# 集中常见的模式

列举了一些常见的模式

## 文本垂直居中

对于行内元素，height会自动收缩到包裹住文本的高度，所以不存在这个问题。但是对于block和inline-block等盒子元素，如果设置了height属性，则文本默认会在上方显示。如果希望文本在垂直方向上居中，可以设置line-height属性等于height属性，或者大于height属性

```
<div>
    hello world
</div>
```

```
div {
    height: 200px;
    line-height: 200px;
}
```

## 文本水平居中，图标分列左右两侧

效果是左侧一个小箭头，右侧一个小箭头，日期显示在中间

```
<div id="wrapper">
    <span><</span>
    <span>2014年5月11日</span>
    <span>></span>
</div>
```

```
#wrapper{
    text-align: center;
    position: relative;
    width: 200px;
}

#wrapper > span:first-child{
    float: left;
}

#wrapper > span:last-child{
    float: right;
}
```

如果给2个float元素设置width（比如为了增大点击区域的目的），则width应该设置成一样，否则会破坏日期水平居中的效果

## 为float元素设置width

一般来说，行内元素（如span），会自动收缩以适应文本宽度，为其设置width属性是无效的。但是当span元素被float了之后，就可以为其设置width属性了

## 盒子有多大

对于块元素，如block和inline-block，如果不设置其width，则会自动扩展到父容器的宽度。此时设置它的padding和border，不会改变盒子的大小（还是和父容器一样宽），但是会缩小content的宽度

```
<div id="parent">
    <div id="son">abc</div>
</div>
```

```
#parent {
    widht: 200px;
}

#son {
    padding: 1px;
    border: 1px solid black;
}
```

son的width自动扩展到200px，然后由于padding和border占了4px，内容区域就是196px

如果设置了son的width，实际上设置的是content的width，加上border和padding后，实际的宽度会超过父容器

```
#son {
    width: 200px;
    padding: 1px;
    border: 1px solid black;
}
```
content的宽度是200px，加上border和padding的4px，最后盒子的宽度是204px

但是，如果设置box-sizing为border-box，则设置的width就变成了整个盒子的宽度，此时再设置border和padding，又会缩小content的宽度了

```
#son {
    box-sizing: border-box;
    width: 200px;
    padding: 1px;
    border: 1px solid black;
}
```
实践中，在全局设置box-sizing为border-box挺不错的，这样设置padding和border时，就不用担心造成盒子宽度的变化了

## 水平排放li

```
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
    <li>5</li>
</ul>
```

```
ul{
    font-size: 0;
}

li{
    font-size: 1rem;
    display: inline-block;
    width: 20%;
}
```

这里的一个技巧是设置ul的font-size为0，因为</li>和<li>之间有空隙，在大部分浏览器上都会造成1px的间隙，造成一行无法容纳5个li，于是最后一个li元素就会掉到下一行。因此把ul的font-size设置成0，在li中再恢复，可以避免此1px问题。用inline-block方式实现水平布局，大多都会有这个问题

## N列固定比例水平布局

有点类似上面的li平铺

```
<div>
    <nav></nav>
    <div></div>
    <aside></aside>
</div>
```

```
div {
    font-size: 0;
}

div > * {
    display: inline-block;
    font-size: 1rem;
    vertical-align: top;
}

div > nav {
    width: 30%;
}

div > div {
    width: 50%;
}

div > aside {
    width: 20%;
}
```
基本上和li平铺的模式完全一样，区别是设置所有的子元素vertical-align: top。否则当各子元素的高度不一致时，看起来似乎没有在同一行。其实用dev tools可以看出来，3个子元素还是在同一行，只是在垂直方向上没有对齐，设置vertical-align可以解决此问题

## 某些列宽度固定，其他列宽度比例固定的水平布局

```
<div>
    <nav></nav>
    <div></div>
    <aside></aside>
</div>
```

```
div {
    display: -webkit-box;
}

div > nav {
    width: 200px;
}

div > div {
    -webkit-box-flex: 1;
}

div > aside {
    -webkit-box-flex: 2;
}
```

优点是不会出现1px问题，也不需要设置子元素为inline-block，很方便。另外，设置display: box之后，子元素会自动变成等高。这在某些场景下很方便，但是也不适用于另外一些场景。最后display: box，在老的浏览器上支持不好，所以这种方式没有老技巧那么通用。但是随着浏览器日趋标准，我觉得慢慢就不是问题了