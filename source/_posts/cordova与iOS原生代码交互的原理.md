title: cordova与iOS原生代码交互的原理
date: 2014-08-06 18:00
categories: iOS
---
本文简要介绍cordova实现javascript代码和iOS原生代码交互的原理
<!--more-->

# js调用native

下面是API客户端的调用代码片段：
```
datePicker.show(options, function (date) {
    var month = date.getMonth() + 1;
    callback(null, date.getFullYear() + "-" + month + "-" + date.getDate());
});
```
cordova插件最终暴露出来的都是普通js接口，调用者完全不知道自己在调用一个cordova插件，或是一个普通的js函数

但是在任何cordova js方法内部，最后一定会调用cordova.exec函数：

```
cordova.exec(successCallback, errorCallback, "DatePicker", "show", []);
```

然后就进入了关键的cordova.exec函数，这是cordova框架的js端的最后一环，就是由它完成对ios native的调用

在exec函数里，首先会判断平台，可能是android，ios或者wp，其他平台本文省略，如果是ios平台，cordova会采用以下2种方式的一种，实现调用iOS原生代码

## 通过iframe

cordova.exec往当前的html中插入一个不可见的iframe，从而请求UIWebView加载一个特殊的URL，这个URL里当然就包含了要调用的native plugin的类名，方法名，参数，回调函数等信息

接下来，由于被请求加载URL，于是UIWebViewDelegate的这个方法被调用：

```
- (BOOL)webView:(UIWebView*)theWebView shouldStartLoadWithRequest:(NSURLRequest*)request navigationType:(UIWebViewNavigationType)navigationType
```
这里就进入了native侧，从request里可以拿到了js端传过来的信息，然后调用到native plugin

## 通过XHR

另一种方式，cordova.exec里直接发起一个XHR请求，被native侧的NSURLProtocol拦截，于是调用到这个native方法：

```
+ (BOOL)canInitWithRequest:(NSURLRequest*)theRequest
```
也进入了native侧，然后以同样的方式调用到native plugin

在2种方式中，cordova会优先选择XHR方式，只有当XHR方式不可用时，才会使用iframe的方式。这2种方法都为从js到native打开了一条通道，剩下的就是传递参数和路由的问题了

# native调用js

另一条通道就简单的多，因为iOS提供了原生支持，所以不需要想特别的办法。即通过UIWebView的这个方法：

```
- (NSString *)stringByEvaluatingJavaScriptFromString:(NSString *)script;
```
看一下cordova框架native侧的代码，我去掉了注释和无关代码：
```
- (void)evalJsHelper:(NSString*)js
{
    if (![NSThread isMainThread] || !_commandQueue.currentlyExecuting) {
        [self performSelectorOnMainThread:@selector(evalJsHelper2:) withObject:js waitUntilDone:NO];
    } else {
        [self evalJsHelper2:js];
    }
}
```
```
- (void)evalJsHelper2:(NSString*)js
{
    NSString* commandsJSON = [_viewController.webView stringByEvaluatingJavaScriptFromString:js];
}
```
可以看到，正是通过UIWebView提供的这个方法完成的，一定执行在main thread

# 同步和异步的问题

从上面的分析可以发现，从js调用native，2种方式都必定是异步的。而从native回到js，却是一个同步的方法，而且是跑在主线程里

调用cordova插件的代码，对返回值的处理一定要放在回调函数里，因为结果是异步返回的。同时，回调函数的执行时间不能太长，否则会阻塞native主线程

# 参考

本文参考了以下2篇文章，都写得很好：

[iOS版PhoneGap原理分析](http://sjpsega.com/blog/2014/06/01/phonegap-ios/)

[浅析Cordova for iOS](http://zhenby.com/blog/2013/05/16/cordova-for-ios/)