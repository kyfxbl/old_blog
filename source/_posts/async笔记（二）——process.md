title: async笔记（二）——process
date: 2014-01-30 16:55
categories: javascript 
---
本篇是流程控制相关的API
<!--more-->

async库在：[async github](https://github.com/caolan/async)

示例代码在：[Async Example](https://github.com/kyfxbl/AsyncExample)

# series

一组函数顺序执行，这个API很常用。当一个步骤完成时，调用callback()，不传递参数；如果其中一个步骤出错，则调用callback(err)，后续的步骤就不会继续执行。每个步骤执行的结果，会汇总到最终callback的results参数中

```
async.series([function(callback){

    console.log("first task");
    callback(null, 1);

}, function(callback){

    console.log("second task");
    callback({message:"some error"}, 2);// callback with a error parameter

}, function(callback){

    console.log("third task");// not called
    callback(null, 3);

}],function(error, results){

    if(error){
        console.error("error happend: " + error);
    }
    console.log(results);// [1, 2]
});
```

# parallel

基本上和series一样，包括API的参数，区别在于，series是顺序执行，而parallel是同时执行

```
async.parallel([function(callback){

    callback(null, 1);

}, function(callback){

    callback({message:"error"}, null);

}, function(callback){

    callback(null, 3);

}], function(error, results){

    if(error){
        console.error(error.message);
    }

    console.log(results);

});
```

# whilst和until

重复执行一个函数，直到test function的值为false或true。类似的还有doWhilst和doUntil

```
var count = 0;

async.whilst(
    function () { return count < 5; },
    function (callback) {
        console.log("call once");
        count++;
        setTimeout(callback, 1000);
    },
    function (err) {
        if(err){
            console.log("error: " + err);
        }
        console.log("whilst done");
    }
);
```
```
var count3 = 0;

async.until(function(){

    return (count3 > 5);

}, function(callback){

    count3 ++;
    setTimeout(callback, 1000);

}, function(error){

    console.log("until done");
    count3 = 0;

});
```

# forever

循环执行一个函数，直到抛出错误

```
async.forever(function(callback){

    setTimeout(function(){
        console.log("forever...");
        console.log(flag);
        flag ++;
        if(flag > 5){
            callback({message:"hehe"});
        }else{
            callback();
        }
    }, 1000);

}, function(err){

    // once this callback be called, the "forever" stops
    if(err){
        console.error("there is an error");
    }

});
```

# waterfall

这个可能是async库中最重要的一个方法，可以解决callback嵌套的问题。上一个流程的执行结果，会传给下一个流程的参数。如果其中一个流程出错，则会中止后续流程的执行，直接调用最终的callback。否则最后一个流程的结果，会传递给最终callback

```
async.waterfall([function(callback){

    callback(null, "kyfxbl", 29);

}, function(name, age, callback){

    console.log(name);// kyfxbl
    console.log(age);// 29
    callback(null, 10000, 200);

}, function(salary, bonus, callback){

    console.log(salary);// 10000
    console.log(bonus);// 200
    callback(null, [1, 2, 3]);

}], function(err, results){

    console.log(results);// [1, 2, 3]

});
```

# compose

可以将几个function组合成function，这个方法不是很常用

```
// f(g(h(n)))
async.compose(function(n, callback){
    setTimeout(function(){
        callback(null, n + 1);
    }, 10);
}, function(n, callback){
    setTimeout(function(){
        callback(null, n * 3);
    },10);
})(4, function(err, result){
    console.log("compose result: " + result);// 13
});
```

# queue

创建一个执行任务的队列，类似oc中的NSOperationQueue，好像用得不多

```
var q = async.queue(function(task, callback){

    console.log("hello " + task.name);
    callback(null);

}, 3);

q.push({name:"kyfxbl"}, function(err){
    console.log("done");
});

q.push({name:"liting"}, function(err){
    console.log("finish");
})
```

# nextTick

node里其实已经有nextTick方法，这个方法是为了统一node和浏览器环境的行为，在浏览器环境，实际调用的是setTimeout(func, 0)函数

```
var order = [];

async.nextTick(function(){
    order.push(222);
});

order.push(111);

process.nextTick(function(){
    console.log(order);// [111, 222]
});
```

# times

重复执行函数n次，并收集最终结果

```
async.times(5, function(n, next){

    console.log("n: " + n);
    next(null, n * 2);

}, function(err, results){

    console.log(results);// [0, 2, 4, 6, 8]
});
```