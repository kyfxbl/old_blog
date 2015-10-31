title: prototype chain
date: 2014-02-19 11:06
categories: javascript
---
javascript是基于原型的继承，本文介绍prototype chain的概念
<!--more-->

# 属性查找规则

每个对象都有一个原型对象，原型对象又有自己的原型对象，这样组成一条“原型链”，直到Object.prototype为止，Object.prototype的原型对象指向null

当在一个对象上查找属性时，先在这个对象自己的属性里查找，称为own property，如果没有找到，就到它的原型对象上查找，直到Object.prototype为止

# 对象的原型

通过对象字面量创建的对象：

```
{name:"kyfxbl"}
```
它的原型对象直接就是Object.prototype

通过构造函数创建的对象：

```
function Person(){
}

Person.prototype.sayHello = function(){
}

var person = new Person();
```
原型对象是Person.prototype，Person.prototype的原型对象是Object.prototype

函数在javascript中也是对象，本质上也是通过构造函数创建的，构造函数是Function，所以每个函数的原型对象都是Funtion.prototype，Function.prototype的原型对象也是Object.prototype

```
function Person(){

}

var person = new Person();

console.log(Person.__proto__ == Function.prototype);// true
console.log(person.__proto__ == Person.prototype);// true
```

# 隐藏属性

原型对象是javascript引擎查找对象属性的时候隐式使用的，一般不会直接在代码中引用一个对象的原型对象，不过chrome，safari，firefox的实现，提供了一个隐藏属性，指向对象的原型对象：
```
__proto__
```

```
function Person(){

}

Person.prototype.sayHello = function(){
    console.log("hello world");
}

var person = new Person();

console.log(person.__proto__ === Person.prototype);// true
```

利用这个属性，可以指定一个对象的原型对象，不过这不是好的实践，也不能保证可移植性，因为并非所有的javascript实现都支持这个属性：

```
function Person(){

}

Person.prototype.sayHello = function(){
    console.log("hello world");
}

var o = {};

console.log(o.__proto__ === Object.prototype);// true，此时o的原型对象是Object.prototype

o.__proto__ = Person.prototype;

o.sayHello();// hello world
```

```
var obj = {name:"kyfxbl"};

var o = {};

o.__proto__ = obj;

console.log(o.name);// kyfxbl
```