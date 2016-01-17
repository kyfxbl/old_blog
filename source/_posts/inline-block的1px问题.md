title: inline-block的1px问题
date: 2014-10-25 14:16
categories: web
---
浏览器上臭名昭著的1px blank问题
<!--more-->

下面这个dom结构：
```
<div>
    <div id="div1"></div>
    <div id="div2"></div>
</div>
```
通过inline-block的方法让2个子div水平排列：
```
div > div {
    display: inline-block;
    width: 50%;
}
```

实践发现，在chrome下，div2会掉到下面去。调试以后发现，多了1px神秘的间隙，然后2个div的width都是50%，所以总的width变成100% + 1，超过了父div的width，结果第二个div就掉下去了

最简单的改法，可以把html写到同一行：

```
<div>
    <div id="div1"></div><div id="div2"></div>
</div>
```
但是这种方法显然不好，不仅html的格式很糟，而且代码格式化，平台差异，自己忘记了等等因素都会使bug重现。比较好的方法，是设置font-size为0，然后在具体的div里再覆盖font-size

```
#parent {
    font-size: 0;
}

.son {
    display: inline-block;
    width: 50%;
    font-size: 1rem;
}
```