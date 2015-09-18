title: javascript闭包
date: 2014-02-20 11:36
categories: javascript
---
javascript中的function都关联到作用域链（scope chain），scope chain作为它的内部状态被保存下来。function的scope chain是其定义时就确定的作用域，而不是调用时的作用域。闭包指的是function，加上它的scope chain
<!--more-->

一般情况下，function定义和调用的作用域一样，这种情况比较好理解
```
function outter(){

    var text = "hello";

    function inner(){
        console.log(text);
    }

    inner();
}

outter();// hello
```

上面的例子中，inner函数的定义和调用所关联的作用域相同，都是outter的call object -> global scope，所以没什么特别的。但是当定义和调用的作用域不同时，情况就比较特殊。这通常发生在外围function将内部的function作为返回值时
```
var scope = "global scope";

function outter(){

    var scope = "local scope";

    function inner(){
        console.log(scope);
    }

    return inner;
}

outter()();// local scope
```

inner function的定义发生在outter()时，作用域是outter call object -> global scope。当调用时，作用域是global scope（其实函数调用时，scope chain的第一个对象是其自身的call object，这里不涉及）。但是真正发生作用的，是定义时的作用域

外围函数每调用一次，就产生一个新的scope chain（其中每次都包含新的call object），因此产生的每个闭包的scope chain都是私有的，不互相干扰。下面的例子是一个错误的写法：

```
var funcs = [];

function createClosure(){

    for(var i = 0; i < 5; i++){
        funcs.push(function(){
            return i;
        })
    }
}

createClosure();

for(var i = 0; i < 5; i++){
    console.log(funcs[i]());// all 5
}
```

createClosure函数，试图创建5个独立的闭包，但是createClosure只执行了一次。所以循环体内定义的5个函数，共享了同一个scope chain，变量i是所有闭包共享的

当真正执行时，i的值是5，所有的函数都输出5。正确的写法应该是这样：

```
var funcs = [];

function createClosure(i){

    funcs.push(function(){
        return i;
    })
}

for(var i = 0; i < 5; i++){
    createClosure(i);
}

for(var i = 0; i < 5; i++){
    console.log(funcs[i]());// 0, 1, 2, 3, 4
}
```

每次createClosure()执行时，都产生一个新的作用域，因此生成的闭包就具有一个独占的变量i

另外要注意的是，this和arguments比较特殊

this是关键字，并不是临时变量，所以不是保存在scope chain上，如果内部的函数要保留外部函数的this，需要把this赋值给一个临时变量

```
function outter(){

    var self = this;

    return function(){
        return self;
    }
}

outter()();// global
console.log(outter()() == global);// true
```

arguments也类似，因为内部嵌套的函数，自己也有arguments属性，所以会把外部函数的arguments属性覆盖掉，所以也需要把外部函数的arguments赋值给一个临时变量：
```
function outter(name){

    var outterArguments = arguments;

    return function(){
        return outterArguments[0];
    }

}

console.log(outter("kyfxbl")());// kyfxbl
```