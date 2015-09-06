title: javascript几个知识点
date: 2015-08-20 11:31:18
categories: javascript
---
本文总结一下javascript几个比较重要的知识点，包括scope chain，this，和函数的一些高级特性
<!--more-->

# scope chain

scope chain是javascript函数调用里最核心的概念，尤其是要理解闭包的概念的话，必须先了解scope chain的原理

## 函数在scope chain上查找变量

function执行时，会在scope chain自底向上地查找变量。scope chain的第一个对象是自己的调用对象（activation object），然后是外层的function的调用对象，然后是更外层的调用对象，直到global object

如这段代码：

```
function wrapper(){

    var name = "kyfxbl";

    function inner(){
        console.log(name);
    }

    inner();
}
```
inner执行时，自己的调用对象上没有变量name，于是在wrapper的调用对象上查找。如果wrapper的调用对象上也没有，就去全局变量里找

## scope chain在函数定义时确定

javascript的作用域规则是词法作用域，也就是说，函数的scope chain是在定义时决定的，而不是调用时才决定

比如这段代码：

```
function wrapper(){

    var scope = "local";

    return function(){
        console.log(scope);
    }

}

var f = wrapper();

var scope = "global";

f();
```

返回的那个匿名函数，其作用域在wrapper()执行时就确定了：self call object -> wrapper call object -> global object。而不是当执行f()的时候才决定

## scope chain上的调用对象可修改

虽然一个函数在定义的时候，scope chain就已经确定了，但是只是确定了scope chain上包含哪些对象。这些对象的属性可以改变，比如这段代码：

```
function wrapper(){

    var name = "original";

    function setName(new_name){
        name = new_name;
    }

    function getName(){
        console.log(name);
    }

    return [setName, getName];
}

var funcs = wrapper();

funcs[1]();// original
funcs[0]("new_name");
funcs[1]();// new_name
```
wrapper()被调用时，定义了2个函数。这2个函数的scope chain上，共享着wrapper的call object。所以调用setName()改变name的值，会对getName()也产生影响。scope chain是确定的，但是其中对象的属性却是可以修改的

## 闭包（closure）如何形成

简单而言，当嵌套函数做为返回值，被外部的变量引用，或者作为外部对象的属性时，闭包就形成了。形成闭包之后，scope chain上的调用对象，都能继续使用，而不会立刻被垃圾回收。犀牛书上有更详细的描述：

a function is executed in the scope in which it was defined. When a function is invoked, a call object is created for it and placed on the scope chain. When the function exits, the call object is removed from the scope chain. When no nested functions are involved, the scope chain is the only reference to the call object. When the object is removed from the chain, there are no more references to it, and it ends up being garbage collected. But nested functions change the picture. If a nested function is created, the definition of that function refers to the call objects because that call object is the top of the scope chain in which the function was defined. If the nested function is used only within the outer function, however, the only reference to the nested function is in the call object. When the outer function returns, the nested function refers to the call object, and the call object refers to nested function, but there are no other references to either one, and so both objects become available for garbage collection. Things are different if you save a reference to the nested function in the global scope. You do so by using the nested function as the return value of the outer function or by storing the nested function as the property of some other object. In this case, there is an external reference to the nested function, and the nested function retains its reference to the call object of the outer function. The upshot is that the call object for that one particular invocation of the outer function continues to live, and the names and values of the function arguments and local variables persist in this object. JavaScript code cannot directly access the call object in any way, but the properties it defines are part of the scope chain for any invocation of the nested function.

## 所有变量都是某个对象的属性

1. 在function内部，通过var声明的变量，是临时变量，本质上是调用对象的属性
2. 在function内部，用function声明的函数，也是临时变量
3. 在function内部，不通过var声明的变量，是全局变量，本质上是全局对象的属性。如果使用strict模式，这种写法是被禁止的，解释器会直接报错
4. 不在任何function内部，声明的变量，是全局变量

以下示例代码可以说明：
```
(function(){

    var a = function(){
        console.log("a");
    };

    b = function(){
        console.log("b");
    };

    function c(){
        console.log("c");
    }

})();

a();// error
b();// ok
c();// error
```

node和浏览器环境，在表现上稍有区别。浏览器环境下，在function外部声明的变量，是全局对象window的属性；而在node环境，不会直接成为global的属性，因为其实node每个文件都是被一个匿名的function包裹着

# this指向哪个对象

javascript和java里的this，完全不同。java中的this含义非常明确，即当前实例。而javascript中的this要复杂得多。比如以下代码：
```
function doSomething(){
    console.log(this.name);
}
```

如果只看这段代码，无法确定在运行时this指向哪个对象。要确定javascript中的this指向哪个对象，必须依赖上下文。总共有5种情况：

## 作为某对象的方法被调用

```
var obj = {
    name: "kyfxbl"
};

obj.sayHi = function(){
    console.log(this.name);
};

obj.sayHi();
```

当function做为某对象的方法被调用时，this指向该对象

## 直接调用

```
global.name = "kyfxbl";

function sayHi(){
    console.log(this.name);
}

sayHi();
```

```
global.name = "kyfxbl";

function wrapper(){

    (function(){
        console.log(this.name);
    })();
}

wrapper();
```

直接调用时，this指向全局对象，浏览器环境下是window，node环境是global

## 作为构造函数被调用

```
function Person(name){
    this.name = name;
}

var person = new Person("kyfxbl");

console.log(person.name);
```

当function被作为构造函数调用时，this指向新创建的那个对象。在OO编程里，这是一种常见的形式

## 通过call和apply调用

```
function sayHi(){
    console.log(this.name);
}

var obj = {
    name: "kyfxbl"
};

sayHi.call(obj);
```

这种情况下，this就是call的第一个参数

## 在顶层代码中的行为

在顶层环境，即不在任何function内部的this，在浏览器环境和node环境不同

在node环境，以下代码的输出结果可能有点出人意料：
```
console.log(this === exports);// true
console.log(this === global);// false

(function(){
    console.log(this === exports);// false
    console.log(this === global);// true
})();
```
在顶层环境，this指向module.exports，而非global

而在浏览器环境，顶层的this指向window
```
alert(this === window);// true
```

这是浏览器环境和node环境的差异

# function的高级属性

function有些高级属性，平时可能用得比较少，但是在写框架和库的时候，有时必须要用到

## apply和call函数

通过apply和call，可以实现函数动态调用

这2个函数很接近，唯一的区别是，apply是传参数数组，而call是传变参

## arguments和this

函数被调用时，会自动创建arguments对象，它的行为类似数组，但又不是数组。通过arguments可以动态判断参数的情况

另一个关键字是this，前面介绍过了。

因为每一个function内部都有这2个关键字，所以如果存在函数嵌套的情况，内部函数的this和arguments就会覆盖外层函数的this和arguments。因此如果需要在内层函数里访问外层函数的这2个属性，通常的做法是：
```
function wrapper(age){

    var that = this;

    var thatArguments = arguments;

    function inner(){
        console.log(that.name);
        console.log(thatArguments);
    }

    inner();
}

var obj = {
    name: "kyfxbl"
};

wrapper.call(obj, 23);// kyfxbl, {'0': 23}
```

## length属性

length表示该function形参的个数：
```
function sayHi(name, words){
    // logic
}

console.log(sayHi.length);// 2
```

## caller

caller指向当前调用此函数的外层函数，如果本身就是最外层的函数，则返回null
```
function wrapper(){

    function inner(){
        console.log(inner.caller.toString());
    }

    inner();
}

wrapper();
```

另外，如果在node环境下，最外层的function，其caller并不是null，而是某一个函数。这也说明了node里的每一个文件，其实都被一个function包裹着：
```
function test(){
    console.log(test.caller.toString());
}

test();
```

输出是：
```
function (exports, require, module, __filename, __dirname) { 

    function test(){
        console.log(test.caller.toString());
    }

    test();
}
```

## callee

callee是arguments上的一个属性，指向自己。当需要获取到function自身的引用时，会用到这个属性。比如实现匿名函数的递归调用
```
function test(){
    console.log(arguments.callee === test);
}

test();// true
```