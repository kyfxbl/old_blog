title: 奇怪的js回调混乱
date: 2014-12-11 01:13
categories: javascript 
---
今天发现一段代码，发生了奇怪的回调混乱现象
<!--more-->

调用的API是这样的：

```
api.method = function(sql, condition, successCallback, failureCallback){
    // logic
}
```

我们自己的业务代码调用了这个函数：

```
var sql = "insert into xxxx";

var condition = {};

api.method(sql, condition, function(result){
    // callback when success
}, function(err){
    // callback when error
});
```

原来的理解，应该是调用这个函数之后，如果结果正确，第一个回调函数会被调用；否则第二个回调函数被调用。实际上发现，确实调用了第一个回调函数，但是没执行完，又突然跳到第二个回调函数里

最后读api.method的源码，发现其内部是类似这样处理的：

```
api.method = function(sql, condition, successCallback, failureCallback){

    var result = {};

    // do some logic
    if(err){
        failureCallback(err);
        return;
    }

    try{
        successCallback(result);
    }catch(err){
        failureCallback(err);
    }
}
```

如上，successCallback是被try...catch包裹住的，如果执行successCallback的过程中，同步方法抛出了异常，就会中止successCallback，转而执行failureCallback。于是接下来检查我们的successCallback，果然有一行代码抛出了异常

感想：

1、我们的代码里，try...catch用得比较少，其实代码还是比较脆弱的，像这种错误就很难定位，甚至有时候默默地出错了，很长时间都发现不了。所以在容易出错的地方，主动地加上try...catch可能会好一点。在这一点上，JAVA就比较好，虽然CheckedException，UncheckedException遭到很多人诟病，但是其实在异常的捕获和定位方面，还是很有帮助的

2、上述API的设计，我觉得并非完全没有道理，在成功回调错误的情况下，可以跳转到错误回调，还是挺机智的。但是仅对同步方法有效，如果我的成功回调里，包含异步函数，它照样无法捕获到异步函数内部的错误，所以也不是很可靠。另外，中断一个回调，跳转到另一个回调，这是明显的潜规则，绝对会令调用者误解，所以如果加上注释和日志，就会好很多