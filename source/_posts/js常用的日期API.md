title: js常用的日期API
date: 2014-05-06 11:20
categories: javascript
---
一些常用的javascript日期API
<!--more-->

```
var now = new Date();// 2014-5-6

console.log(now.getYear());// 114
console.log(now.getFullYear());// 2014
console.log(now.getMonth());// 4
console.log(now.getDay());// 2
console.log(now.getDate());// 6
```

主要就是getMonth()函数返回值是从0开始，比如5月返回的是4，所以为了在页面上正确展示，需要把返回值+1

下面是一个技巧，可以得到某年某月的最大天数，会自动计算闰年，很方便：

```
var temp = new Date(2014, 2, 0);
console.log(temp.getDate());// 28
```