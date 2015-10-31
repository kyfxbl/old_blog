title: node.js的global variable，和module.exports
date: 2013-10-10 23:28
categories: javascript 
---
node.js中的global和module.exports
<!--more-->

global是node.js中的全局对象，对应浏览器环境的window对象。而module.exports则是node.js特有的概念

# global

javascript的语言特性决定了，一定会有一个顶层对象（top object），但是顶层对象由执行环境决定

比如在浏览器执行环境中，顶层对象是window。而在node里，顶层对象是global

global里定义了一些全局的对象或函数，在node的任何一个模块里，都可以直接使用，比如console，setTimeout()，require()等，完整的global object document见：[node.js global objects](http://nodejs.org/api/globals.html)

如果想在不同的模块（文件）之间共享变量，有一个可行但是很糟糕的做法，就是借助global object，在global上定义的属性和函数，在任何模块里都可以访问到

```
global.name = "Tony";
```

然后在a.js里

```
require("./b");// 执行b.js里的代码

console.log(name);// Tony
```

另外一种更糟糕的做法，是不用var关键字声明变量，也会成为global的属性

```
name = "Tony";
```

但是，如果加上了var关键字，声明的变量就只局限在module（当前文件）的作用域中，所以强烈推荐显式地用var来声明变量，以免不小心挂到了global上

```
var name = "Tony";
```

为什么依赖global是不好的实践呢？因为所有的模块都可以不受限制地使用global，而且缺少命名空间的约束，非常容易引起冲突，从而引发潜在的BUG。而且这种BUG一旦发生，要定位是极其困难的，不知道是在哪里改变了全局变量而引发的问题

所以javascript的最佳实践，是<span style="color:#ff0000">不要修改global object，只使用global上预定义的属性和函数</span>

# module

在模块里用var声明的变量，只在当前module里可见。众所周知，javascript只有全局作用域和函数作用域，那么node.js是如何实现module级别的作用域的呢？

实际上，每一个module都被包裹在一个匿名的function中，下面的代码可以清楚地说明这一点：
```
var a = "abc";
console.log(arguments.callee.toString());
```

执行结果：
```
function (exports, require, module, __filename, __dirname) { var a = "abc";
console.log(arguments.callee.toString());

}
```

所以临时变量a在模块外部是不可见的

```
global.name = "Tony";
```
```
require("./b");// 执行b.js里的代码

var name = "kyfxbl";

console.log(name);// kyfxbl
```

以上代码会输出"kyfxbl"，而不是"Tony"，因为module的name比global的name更优先

# module.exports vs exports

如果想不借助global，在不同模块之间共享代码，就需要用到exports属性。令人有些迷惑的是，在node.js里，还有另外一个属性，是module.exports。一般情况下，这2个属性的作用是一致的，但是如果对exports或者module.exports赋值的话，又会呈现出令人奇怪的结果

网上关于这个话题的讨论很多，流传最广的是这个帖子：[exports vs module.exports](http://www.hacksparrow.com/node-js-exports-vs-module-exports.html)，但是这篇帖子里有些说法明显是错误的，却没有人指出来。下面说一下我的理解

首先，exports和module.exports都是某个对象的引用（reference），初始情况下，它们指向同一个object，如果不修改module.exports的引用的话，这个object稍后会被导出

![](http://img.blog.csdn.net/20131010224155750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所以不管用exports还是module.exports，给这个object添加属性或函数，都是完全等效的

```
exports.name = "Tony";
module.exports.age = 33;
```

```
var b = require("./b");
console.log(b);// {name:"Tony", age:33}
```

所以如果只是给对象添加属性，不改变exports和module.exports的引用目标的话，是完全没有问题的

但是有时候，希望导出的是一个构造函数，那么一般会这么写：

```
module.exports = function (name, age) {
    this.name = name;
    this.age = age;
}

exports.sex = "male";
```

```
var Person = require("./b");
var person = new Person("Tony", 33);
console.log(person);// {name:"Tony", age:33}
console.log(Person.sex);// undefined
```
这个sex属性不会导出，因为引用关系已经改变：

![](http://img.blog.csdn.net/20131010230949562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

而node导出的，永远是module.exports指向的对象，在这里就是function。所以exports指向的那个object，现在已经不会被导出了，为其增加的属性当然也就没用了

如果希望把sex属性也导出，就需要这样写：

```
exports = module.exports = function (name, age) {
    this.name = name;
    this.age = age;
}

exports.sex = "male";
```
事实上，查看很多module的源代码，会发现就是这么写的，这时的引用关系：

![](http://img.blog.csdn.net/20131010231244734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所以我感觉exports根本是多余的，最终只会导出一个object，却设计了2个引用，很多时候反而会造成迷惑。exports的唯一好处就是可以少敲几个字，还不如只保留module.exports就好了

# exports的缓存

执行require之后，目标模块的代码会被完整地执行一次，然后module.exports对象被返回

需要注意的是，这个过程只会发生一次，后面重复的require，只会拿到同一个对象

```
exports.name = "Tony";
exports.age = 33;
```

```
var b1 = require("./b");
var b2 = require("./b");
console.log(b1 === b2); // true

console.log(b2.age);// 33
b1.age++;
console.log(b2.age);// 34

var b3=require("./b");
console.log(b3.age);// 34
```

这个时候的引用关系是这样的：

![](http://img.blog.csdn.net/20131010232057359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)