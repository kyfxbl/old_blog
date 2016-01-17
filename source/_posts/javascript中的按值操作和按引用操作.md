title: javascript中的按值操作和按引用操作
date: 2013-09-24 10:28
categories: javascript 
---
犀牛书第5版第3章最后一节，谈的是by value versus by reference，这一节总结得很好
<!--more-->

In javascript, as in all programming languages, you can manipulate a data value in three important ways. First, you can copy it. For example, you might assign it to a new variable. Second, you can pass it as an argument to a function or method. Third, you can compare it with another value to see whether the two values are equal. To understand any programming language, you must understand how these three operations are performed in that language. 

# 规则1

number和boolean是按值使用，Object(包括Array、Function)是按引用使用

# 规则2

但是引用本身是按值传递(references themselves are passed by value)，上个例子，不然太抽象了

```
function test() {
	var person = {name:"michael"};
	alert(person.name);
	notChangeObject(person);
	alert(person.name);
}

function changeObject(obj){
	obj.name = "john";// obj是引用，因此会实际修改name的值
} 

function notChangeObject(obj){
	var temp = {name:"john"};
	obj = temp;// obj本身是按值传递，因此person依然指向原来的对象
}
```

# 规则3

string比较特殊，可以认为是按引用复制，按引用传递的。但是实际上没有意义，因为javascript中的string是不可变的。string是按值比较 

# 总结

boolean和number: 值复制，值传递，值比较 
string: 复制不可变，传递不可变，值比较 
object(包括array和function): 引用复制，引用传递，引用比较。但是引用本身是值传递。