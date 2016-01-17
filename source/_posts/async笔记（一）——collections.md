title: async笔记（一）——collections
date: 2014-01-26 21:37
categories: javascript 
---
async在node和浏览器里都可以用，API基本没区别。本篇是集合操作相关的API
<!--more-->

node环境：
```
var async = require("async");
async.xxx();
```

浏览器环境：
```
<script type="text/javascript" src="async.js"></script>
<script type="text/javascript">

    async.xxx();

</script>
```

本文总结下各API调用的方法。本文的示例代码在：[AsyncExample](https://github.com/kyfxbl/AsyncExample)

# each

Array的迭代器方法，很常用

```
async.each(["abc", "def", "ghi"], function(item, callback){

    console.log(item);
    callback(null);// must call once

}, function(err){

    if(err){
        console.error("error");
    }
});
```

# eachSeries

和each基本一样，但是顺序执行，API接口和参数都一样

# eachLimit

也和each差不多，多出来的limit参数，是限制允许并发执行的任务数

```
async.eachLimit(["123", "456", "789"], 2, function(item, callback){

    console.log(item);
    callback();// 必须调用，才能触发下一个任务执行

}, function(error){

    if(error){
        console.error("error: " + error);
    }

});
```

# map

将一个Array中的元素，按照一定的规则转换，得到一个新的数组（元素个数不变），也比较常用

```
async.map([1,3,5], function(item, callback){

    var transformed = item + 1;
    callback(null, transformed);

}, function(err, results){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(results);// [2, 4, 6]

});
```

# mapSeries和mapLimit

基本差不多，就不多介绍了

# filter

用于过滤Array中的元素

```
async.filter([1, 5, 3, 7, 2], function(item, callback){

    callback(item > 3);

}, function(results){

    console.log(results);// [5, 7]

});
```

# filterSeries

类似

# reject和rejectSeries

和filter正好相反，filter是保留true的item，而reject是删除true的item

```
async.reject([4, 7, 88, 11, 36], function(item, callback){

    callback(item > 11);

}, function(results){

    console.log(results);// [4, 7, 11]

});
```

# reduce和reduceRight

将一个数组中的元素，归并成一个元素，看看就好了，用得不是很多，可以需要的时候再查

```
async.reduce([3, 2, 1], 0, function(memo, item, callback){

    callback(null, memo + item)

}, function(err, result){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(result);// 6
});
```

reduceRight差不多，只是Array中元素迭代的顺序是相反的：

```
async.reduceRight(["!", "ld", "wor"], "hello ", function(memo, item, callback){

    callback(null, memo + item)

}, function(err, result){

    if(err){
        console.error("error: " + err);
        return;
    }

    console.log(result);// hello world!
});
```

# detect和detectSeries

从数组中找出符合条件的元素

这个API很像filter，参数也都一样，但是只会返回一个结果

```
async.detect([13, 44, 23], function(item, callback){

    callback(item > 37);

}, function(result){

    console.log(result);// 44

});
```

# sortBy

数组元素排序

```
var person1 = {"name": "aaa", age:79};
var person2 = {"name": "bbb", age:23};
var person3 = {"name": "ccc", age:54};

async.sortBy([person1, person2, person3], function(person, callback){

    callback(null, person.age);

}, function(err, sorted){

    console.log(sorted);
});
```

# some

同名函数any。在数组中找至少一个元素，类似于filter和detect。区别在于，filter和detect是返回找到的元素，而some是返回bool

```
async.some([1, 5, 9], function(item, callback){

    callback(item > 10);

}, function(result){

    console.log(result);// false

});
```

# every

同名函数all。跟some相反，如果数组中所有元素都满足条件，则返回true，否则返回false

```
async.every([1, 21, 23], function(item, callback){

    callback(item > 10);

}, function(result){

    console.log(result);// false

});
```

# concat

对数组中的元素进行迭代操作，形成一个新数组

```
async.concat([1, 2, 3], function(item, callback){

    callback(null, [item+1, item+2]);

}, function(err, results){

    console.log(results);// [2, 3, 3, 4, 4, 5];

});
```