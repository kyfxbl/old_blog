title: seajs入门
date: 2013-10-14 23:21
categories: javascript 
---
seajs使用简介
<!--more-->

# seajs的价值

原生javascript的一个弱项，就是不支持模块化，说白了就是没有其他语言的import，include等语句。所以开发者就只有2个选择：把所有的东西写到一起，或者通过全局变量来交互

这造成以下几个问题：

1、污染全局变量，容易发生命名空间冲突，难以维护

2、无法按需加载

由于javascript官方迟迟未能解决这些问题，所以就有民间的社区提出标准，希望能自行解决，弥补语言的不足。主要有2种规范，一种是CommonJS提出的CMD规范，另一种是AMD规范，server端的node就是CMD的一种实现，seajs实现的也是CMD规范。所以熟悉node的用户会发现，seajs的API和node的API非常类似，这就是因为它们是同一种规范的不同实现

关于seajs的价值，这篇帖子说得更加详细：[why seajs](http://chaoskeh.com/blog/why-seajs.html)

# seajs的例子

官网上的5分钟入门例子也很简单，不过本文再简化一点，用一个更简单的例子进行说明：

目录结构简化后是这样的：

![](http://img.blog.csdn.net/20131014224034015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以下是代码：

```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello Sea.js</title>
</head>
<body>

<div id="thediv">
    <p>hello world</p>
</div>

<script src="../javascript/sea.js"></script>
<script>

    seajs.config({
        alias: {
            "jquery": "jquery-debug.js"
        }
    });

    seajs.use("../javascript/main");

</script>

</body>
</html>
```

首先引入了seajs，并且要放在第一行。然后这行代码是入口：

```
seajs.use("../javascript/main");
```

加载seajs以后，会引进一个唯一的全局变量seajs，这是无法避免的，除非是一次性执行的匿名函数，否则单全局变量模式已经是最好的结果。然后seajs上有一个use方法，是启动其他模块的入口方法，这里就是启动了main.js

```
define(function (require, exports, module) {

    var module1 = require("./module1");
    alert(module1.add(1, 2));

})
```
实际上use()和require()方法非常类似，不过官方推荐的最佳实践是，只把use()作为启动的入口方法，后续的模块导入都用require()来完成

熟悉node的用户会觉得非常眼熟，define方法是seajs自己的实现方式，require，exports，module都是CMD规范的设计，所以跟在node里是一样的。所有的seajs module，都应该写在define方法的factory function里

上面这段代码就引入了另一个模块module1，然后调用了其上的一个方法

```
define(function (require, exports, module) {
    exports.add = function (a, b) {
        return a + b;
    }

    var $ = require("jquery");
    $("#thediv").click(function () {
        alert("on click");
    });

})
```
这里又看到了熟悉的exports，导出了add()函数，并引用了jQuery框架，给DOM元素绑定了一个事件

这里有一点要注意，就是对jQuery进行了CMD改造（也就是导出了变量$），这里有个小坑，后面会说

factory function有3个参数，分别是require，exports，module，作用跟node里是完全一样的。但是还有另外一种写法：

```
define(function () {
    return {sayHello: function () {
        alert("hi there");
    }};
});
```
这个factory function没有任何参数，直接返回了导出的对象。这个叫做return语法，不过我用得比较少

# 与node module的比较

基本上是一致的，包括exports和module.exports的表现，还有require后立刻执行的行为，还有require的结果也同样会被cache起来

# 集成jQuery的一个坑

我下载了官网的例子，然后只是稍微移动了文件的路径，结果发现这行代码：

```
var $ = require("jquery");
```
$的值一直是null，百思不得其解。在官方的GitHub issue上，这个问题也有广泛的讨论。本质的原因在于seajs有一个路径和ID匹配的原则：[#930](https://github.com/seajs/seajs/issues/930)

seajs的设计思想是，路径即ID。一般在调用define()方法时，如果只传递一个factory function，那么这个模块就是个匿名模块；或者传递define(module id, dependency, factory)，这个模块就是个具名模块

如果一个文件就是一个模块，那么匿名模块就可以了。但是在生产环境中，往往会把多个模块放到一个文件里，但是路径只有一个，如何知道要加载哪个模块呢？这时候就需要给其中一个模块赋予module id，和path保持一致，seajs就知道应该加载这个ID和path匹配的模块了

如果具名模块的id和require的path参数不匹配，就会返回null，这就是我出现这个错误的原因：

```
var $ = require("jquery-debug");// path是"jquery-debug"
```
而jquery中的代码：

```
define("jquery/jquery/1.10.1/jquery-debug", [], function () { return jQuery; } );// module_id是"jquery/jquery/1.10.1/jquery-debug"
```

我移动了文件路径以后，path和module_id匹配不上，所以就失败了

把jQuery中的代码改成：

```
define("jquery-debug.js",[],function(){return jQuery});
```
或者
```
define(function(){return jQuery});
```
就行了，但是对于不熟悉的用户来说，这确实是一个坑

# 其他文档

本文只是seajs的入门贴。要详细了解，请看GitHub主页上的相关链接，精选几篇：

[前端模块化开发的价值](https://github.com/seajs/seajs/issues/547)

[前端模块化开发的历史](https://github.com/seajs/seajs/issues/588)

[ID和路径匹配原则](https://github.com/seajs/seajs/issues/930)

[与RequireJS的异同](https://github.com/seajs/seajs/issues/930)

[模块的加载启动](https://github.com/seajs/seajs/issues/260)

[模块标识](https://github.com/seajs/seajs/issues/258)