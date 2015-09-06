title: 两种常见的exports写法
date: 2014-02-18 15:15
categories: javascript
---
要在node中导出一个模块，一种写法是直接导出一个object，function作为exports的属性；另一种是导出一个function作为构造函数，真正想导出的function写在prototype上。本文介绍2种写法的区别
<!--more-->

# 导出对象

第一种写法是作为对象导出：
```
exports.sayHello = sayHello;

function sayHello(){
    console.log("hello world");
}
```

然后调用的时候：
```
var obj = require("./method1");

obj.sayHello();
```

# 导出构造函数

第二种写法是作为构造函数导出：
```
exports = module.exports = Person;

function Person(){

}

Person.prototype.sayHello = function(){
    console.log("hello world");
}
```

调用的时候：
```
var Person = require("./method2");

var someone = new Person();

someone.sayHello();
```

# 两种方式比较

第一种方式更常见一点，不过也有很多开源框架用的是第二种方式（比如express）。第二种方法更OO，在需要保存和访问实例变量时，应该用第二种方法。如果导出的方法要共享变量，用第一种方法更方便

另外用第二种方法导出，在webstorm里可以直接链接到prototype上的function定义，读代码比较方便

还见过一种写法，是混合使用2种方式：
```
exports = module.exports = Person;

function Person(){

}

Person.prototype.sayHello = function(){
    console.log("hello world");
}

Person.sayName = function(){
    console.log("my name is kyfxbl");
}
```

调用者：
```
var Person = require("./method2");

Person.sayName();

var someone = new Person();
someone.sayHello();
```
因为在javascript里，function也是object，也可以有自己的属性，所以可以把别的function挂在这个构造函数上面

# 另一种不推荐的做法

其实还有一个更简单的办法，在默认情况下，this，exports，module.exports指向同一个object，所以直接用

```
this.sayHello = sayHello;

function sayHello(){
    console.log("hello world");
}
```
也可以导出sayHello函数，但是这样不直观，而且IDE也识别不了sayHello方法，所以不推荐使用