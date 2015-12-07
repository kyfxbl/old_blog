title: ActionContext、ValueStack、OGNL
date: 2013-09-24 11:11
categories: java 
---
本文简要介绍struts2中几个核心的组件ActionContext、ValueStack、OGNL表达式 
<!--more-->

# ActionContext 

当struts2接收到一个HTTP请求时，它立刻创建一个ActionContext、ValueStack、Action对象 

ActionContext里有6个对象，分别是valueStack、parameters、request、session、application、attr 

一个OGNL表达式，必须选择ActionContext中的一个对象作为根对象（root），默认情况下，是选择valueStack作为根对象，如果需要使用另外5个对象作为根对象，需要加上\#前缀

```
<s:property value="#session.xxx" />
```

如果不加\#前缀，则默认使用valueStack作为根对象，这也是最常见的情况，即\#valueStack.xxx，相当于xxx


# ValueStack（值栈） 

ValueStack中可以存储很多对象，它的一个特性是，它是一个虚拟对象，它可以将自己持有的对象的属性，当成是自己的属性 

比如说，ValueStack中有一个Action对象，而Action对象有一个name字段。那么当用OGNL表达式取name的值的时候，不需要${action.name}，而是可以直接${name} 

ValueStack是一个栈的数据结构（FILO），最后进入值栈的对象，总是在ValueStack的栈顶，这个结论很重要，因为栈顶的元素的值，会覆盖栈底的同名元素的值。比如说，ValueStack的栈底是一个Action对象，持有一个name字段；栈顶是一个Model对象，也持有一个name字段，那么用${name}，取出来的永远是Model对象的name字段，Action对象的name字段是不可见的 
 
# OGNL表达式 

这个可以分为2种场景

## s:标签

```
<s:property value="" />
```

在s:标签的属性里时，要看这个属性定义的类型是什么，如果是string类型，那么属性的值会被当做普通的string，如果不是string类型，那么属性的值会被直接当成OGNL的表达式 

比如说

```
<s:property value="" />
```

这个标签的value属性的类型是object，那么这个value的值，就会被直接作为OGNL表达式进行解析 

如果想在string类型的属性中使用OGNL表达式，就需要加上${}或者%{}

## jsp页面的其他地方 

在jsp页面的其他地方时（即不在s:标签内部），任何情况下都会当成string来处理，这时候如果想使用OGNL表达式，也需要加上${}或者%{} 