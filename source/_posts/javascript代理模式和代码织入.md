title: javascript代理模式和代码织入
date: 2014-02-23 21:01
categories: javascript 
---
我们的产品有多个客户端，包括web客户端和终端客户端（ios，android），源码都是一段js，其中涉及在数据库中查找记录，然后展示。但是web客户端和终端的逻辑有些不同。终端只要直接从本地sqlite数据库中查询，而web需要先到server查询，然后插入web sql，最后从本地web sql中取出。昨天看到同事写的一段代码很巧妙
<!--more-->

通常来说，上层的业务代码，需要判断当前的环境，require不同的dao代码，类似：

```
if(isMobile()){
    var dao = require("local-data-interface");
}else{
    var dao = require("web-data-interface");
}
```
然后，调用dao上的方法

```
dao.queryUsers();
dao.insertUser();
dao.updateUser();
```

前提是local-data-interface.js和web-data-interface.js导出的接口是一致的，这样客户端才能不加修改地调用dao上的各方法

但是有个问题，由于js模块通过seajs导入，而seajs扫描模块是静态扫描，所以前面的if判断是无效的

于是我的同事就十分机智地想到一个办法，客户端统一require web-data-interface.js，但是将web-data-interface作为local-data-interface的代理：

```
var localDataInterface = require("./local-data-interface");

function queryUsers(){
    if(isMobile()){
        localDataInterface.queryUsers();
        return;
    }
    // web端的逻辑
}
```
前面已经保证了web端和终端的接口是一样的，所以这个方法才能奏效。类似于java中的代理模式，定义了一个DataInterface的接口，然后LocalDataInterface和WebDataInterface都实现此接口，同时WebDataInterface作为LocalDataInterface的代理

但是，还有另一个问题：所有的方法，都需要先判断当前的环境，如果是移动环境，调用代理方法；web环境才调用自己的逻辑。如果有一个方法忘记判断，那么终端就会使用web端的逻辑，产生BUG

一个办法是把所有的方法都检查一遍，但是方法多了难免就会出错，而且也有大量的冗余代码。不要紧，这位同事的机智还没有用完，利用js动态语言的特性，用很少的代码就实现了代码织入（function weave）

```
function buildExports(){

        var methods = [initService, queryServiceCount, queryServiceBelongRecordCate];

        // method weave
        _.each(methods, function(method){
            exports[method.name] = function(){
                if(utils.isMobileEnv()){
                    featureDataI[method.name].apply(featureDataI, arguments);
                    return;
                }
                method.apply(exports,arguments);   
            }
        });
    }
```
methods数组里声明所有的接口方法，然后迭代织入代码，动态生成方法并导出。代码很好理解，就不解释了

这种method weave我以前用java也做过，方法是基于spring的AOP，比起这段代码来要繁琐很多。这也体现了js作为动态语言的优势