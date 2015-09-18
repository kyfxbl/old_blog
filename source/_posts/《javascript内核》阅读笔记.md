title: 《javascript内核》阅读笔记
date: 2013-09-24 10:31
categories: javascript
---
本文是阅读《javascript内核》的笔记总结。原系列文章在iteye上
<!--more-->

原文见[javascript core](http://abruzzi.iteye.com/)

# 数据类型
 
javascript中的数据类型分2种：基本类型和对象类型

其中对象类型包括：Object, Array, Function

基本类型包括：String, Number, boolean 

# 非空对象在boolean环境

所有非空对象，在boolean环境下，都会转换成true
```
if(""){
    alert("true");
}
```
这段代码不会alert true

```
if(new String("")){
    alert("true");
}
```
这段代码则会alert true 

经常看到这样的代码：
```
if(datamodel.item){
    // do something...
}else{
    datamodel.item = new Item();
}
```
datamodel.item是一个对象，而if需要一个boolean型的表达式，所以这里引擎自动将对象转换为boolean类型。如果该对象非空，则转换为true，否则为false 

# 变量的作用域

变量被定义的区域即为其作用域，全局变量具有全局作用域；在函数内部的变量则具有局部作用域，在函数的外部不能直接访问。javascript中没有块作用域 

# 取对象属性的操作符

用[]操作符和.操作符都可以取到对象中的属性，比如
```
var obj = {
    name : "zsd"
};
alert(obj.name);
alert(obj["name"]);
```
一般来说，点操作符比较方便，[]操作符常用于key也是变量的情况

# ==操作符

如果操作数具有相同的类型，则如果两个操作数的值相等，则返回true，否则返回false 

如果操作数的类型不同，分下列情况来判断： 

null和undefined相等

其中一个是数字，另一个是字符串，则将字符串转换为数字，再做比较 

其中一个是true，先转换成1（false则转换为0），再做比较 

如果一个值是对象，另一个是数字/字符串，则将对象转换为原始值（通过toString()或者valueOf()方法） 

其他情况，则直接返回false

对于===，比==更加严格，不允许隐式转型

# ===操作符 

如果操作数的类型不同，则不进行值的判断，直接返回false 

如果操作数的类型相同，分下列情况来判断： 

都是数字的情况，如果值相同，则两者等同，否则不等同 

都是字符串的情况，如果串的值不等，则不等同，否则等同 

都是布尔值，且值均为true/false，则等同，否则不等同 

如果两个操作数引用同一个对象（数组，函数），则两者完全等同，否则不等同 

如果两个操作数均为null/undefined，则等同，否则不等同 

# 顶级作用域中定义的变量

在顶级作用域中声明的变量将作为全局对象的属性被保存，从这一点上来看，变量其实就是属性。比如，在浏览器环境
```
var v = "global";
```

实际上相当于
```
window.v = "global";
```

在node环境，看起来并非如此：
```
var v = "global";
```

并不相当于
```
global.v = "global";
```
因为实际上node的每个模块，都被一个function包裹，所以实际上并不是在顶级作用域里声明的

# 原型链

javaScript本身是基于原型的，每个对象都有一个原型属性（属性名依赖于具体实现，比如在老版本的FF里，该属性被实现为\_\_proto\_\_）。这个prototype本身也是一个对象，因此它本身也可以有自己的原型，这样就构成了一个链结构

用构造函数创建出来的对象，其原型指向构造函数的prototype属性

```
function Person(){
}

Person.prototype.name = "kyfxbl";

var person = new Person();

console.log(person.name);// kyfxbl
```

访问一个属性的时候，解析器从下向上地遍历这个链结构，直到遇到该属性，则返回属性对应的值，或者遇到原型为null的对象（JavaScript的基对象Object的构造器的默认prototype有一个null原型），如果此对象仍没有该属性，则返回undefined
 
# 函数本身也是对象

函数也是对象，可以为其任意添加属性

```
function p(){
    alert("hello world");
}
p.id = "func";
p.type = "function";
```

# 调用函数的参数

javascript中的函数对参数的处理十分灵活，可以传递任意数量的参数给一个function
```
function sum() {
    var result = 0;
    for ( var i = 0; i < arguments.length; i++) {
        var current = arguments[i];
	    if (isNaN(current)) {
	        throw new Error("not a number exception");
	    } else {
	        result += current;
	    }
    }
    return result;
}
alert(sum(1, 2, 3, 4));
alert(sum(5, 6));
alert(sum(1, 2, "ky"));
```

# 扩展数组

```
Array.prototype.useless = function(){};
var arr = [1, 2, 3, 4, 5];
alert("length: " + arr.length);// 5

for(var prop in arr) {
    alert(prop + ": " + arr[prop]);// 会输出useless
}
for(var i = 0; i < arr.length; i++) {
    alert(arr[i]);// 不会输出useless
}
```
从这个例子可以看出，除非必要，尽量不要对全局对象进行扩展，因为对全局对象的扩展会造成所有继承链上都带上“烙印”，有时候会造成一些非常难以发现的BUG


# new操作符的本质

首先，创建一个空对象，然后调用函数的apply方法，将这个空对象传入作为apply的第一个参数
```
var triangle = new Shape("triangle", 23);
```
相当于
```
var triangle = {};
Shape.apply(triangle, ["triangle", 23]);
```

接下来把新对象的原型，指向Shape.prototype

# 函数柯里化 

柯里化就是预先将函数的某些参数传入，得到一个简单的函数，但是预先传入的参数被保存在闭包中
```
var adder = function(num) {
	return function(y) {
		return num + y;
	}
}
var inc = adder(1);
var dec = adder(-1);
alert(inc(99));// 100
alert(dec(101));// 100
alert(adder(100)(2));// 102
alert(adder(2)(100));// 102
```

# 原型对象prototype的实现

依赖javascript执行环境

```
var base = {
	name : "base",
	getInfo : function() {
		return this.name;
	}
}

var ext1 = {
	id : 0,
	__proto__ : base
}

var ext2 = {
	id : 9,
	__proto__ : base
}

alert(ext1.id);
alert(ext1.getInfo());
alert(ext2.id);
alert(ext2.getInfo());
```
以上代码在firefox和chrome下可以跑，在ie下则报错

# 构造器自动为新创建的对象设置原型对象

```
function Task(id) {
	this.id = id;
}

Task.prototype.status = "STOPPED";
Task.prototype.execute = function(args) {
	return "execute task_" + this.id + "[" + this.status + "]:" + args;
}

var task1 = new Task(1);
var task2 = new Task(2);

task1.status = "ACTIVE";
task2.status = "STARTING";

alert(task1.execute("task1"));
alert(task2.execute("task2"));
```

下图说明了此原型链的结构： 
![](http://dl.iteye.com/upload/attachment/473801/e034e3ab-0a37-31c6-92c1-498d353e74ae.png)

# scope chain

作用域链与原型链类似，也是一个对象组成的链，用以在上下文中查找标识符（变量，函数等）

查找时也与原型链类似，如果调用对象（更标准的名称是活动对象，Activation Object）本身具有该变量，则直接使用变量的值，否则向上层搜索，直到查找到或者返回undefined

作用域链的主要作用是查找自由变量，所谓自由变量是指，在函数中使用的，非函数内部局部变量，也非函数内部定义的函数名，也非形式参数的变量。这些变量通常来自于函数的“外层”或者全局作用域，比如在函数内部使用的window对象及其属性
```
var topone = "top-level";

(function outter() {

	var middle = "mid-level";

	(function inner() {
		var bottom = "bot-level";
		alert(topone + ">" + middle + ">" + bottom);
	})();

})();
```
![](http://dl.iteye.com/upload/attachment/473813/0192648f-37d0-37e7-bbf8-8d9860218c47.png)