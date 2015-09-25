title: CSS实现平铺滚动效果
date: 2014-11-13 15:12
categories: web
---
用css实现平铺滚动的效果
<!--more-->

效果：
![](http://img.blog.csdn.net/20141113150541343)

思路是在父容器上设置水平方向滚动，并禁止换行。然后子元素用inline-block实现平铺。关键CSS如下：

```
<ul>
    <li></li>
    <li></li>
    <li></li>
</ul>
```

```
ul {
    width: 100%;
    overflow: scroll;
    white-space: nowrap;
}

li {
    display: inline-block;
    width: 50%;
    vertical-align: top;
}
```