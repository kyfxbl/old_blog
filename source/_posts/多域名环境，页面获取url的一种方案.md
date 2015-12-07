title: 多域名环境，页面获取url的一种方案
date: 2014-10-12 15:31
categories: 方案
---
由于系统是分布式部署的，并且有多个域名，所以经常涉及到获取url的问题。这是系统框架层面需要提供的能力，否则每个模块都需要自己去想办法获取ip，就会很混乱，上线也容易发生bug。本文总结一下思路
<!--more-->

# 要解决的问题

需要解决以下2个场景的需求

## 区分开发环境和生产环境

比如部署上线，url可能是
```
http://www.xxx.com/svc/hello
```

而在本地开发的时候应该是
```
http://127.0.0.1/svc/hello
```

不能写死，否则开发和部署就要换来换去，很麻烦

## 根据不同的服务区分URL

比如获取验证码的服务，应该调用
```
http://www.xxx.com/svc/getCode
```

而微信相关的服务，应该调用
```
http://wx.xxx.com/svc/xxx
```

# 配置文件

1、应用有对应的配置文件，里面说明了是以开发模式，还是以生产模式启动。并且将URL分离开，比如鉴权相关的URL，微信相关的URL，普通服务相关的URL等

2、同时，配置文件有多份，比如topo-dev.json，topo-production.json，topo-image.json等。这样就把不同的环境隔离开，如果是以开发模式启动，加载的就是topo-dev.json，其中配置的URL都是127.0.0.1这样的

3、启动的时候，加载此配置文件，并将关键信息放在global._g_env全局变量下面，运行时就能很方便地获取到环境和URL信息了

# 服务端获取URL

服务端的代码也是跑在node环境下，所以要获取URL就很简单，通过_g_env.url，就可以拿到配置文件里的路径了

# 前端页面获取URL

前端页面经常也需要发送ajax请求，所以也需要知道url。但是静态的js没有办法获取server的环境信息和URL等，所以需要从服务端获取到这些信息

首先服务端有一个服务，专门将这些信息下发：

```
function clientSettingScript(req, res, next){

    var script = "window.global = {_g_server:{}}; \n"+
        ";global[\"_g_server\"].staticurl=\"" +global["_g_topo"].clientAccess.staticurl + "\"\n"+
        ";global[\"_g_server\"].uploadurl=\"" +global["_g_topo"].clientAccess.uploadurl + "\"\n"+
        ";global[\"_g_server\"].authurl=\"" +global["_g_topo"].clientAccess.authurl + "\"\n"+
        ";global[\"_g_server\"].serviceurl=\"" +global["_g_topo"].clientAccess.serviceurl + "\"\n"+
        ";global[\"_g_server\"].wxserviceurl=\"" +global["_g_topo"].clientAccess.wxserviceurl + "\"\n"+
        ";global[\"_g_server\"].nail_pc_url=\"" +global["_g_topo"].connector.nail_pc_url + "\"\n"+
        ";global[\"_g_env\"] =\"" +global["_g_topo"].env+ "\";\n";
    res.end(script);

}
```

这是一个express的普通服务，但是其实是一段js脚本。在前端页面，用script标签来加载它

```
<script src="/svc/portal/setting"></script>
```

这样当浏览器拿到响应之后，就会将它作为一段js脚本来执行，在window上放了一个全局变量global，其中有环境信息和URL信息

同时，URL只包含域名，页面根据实际情况，组装完整的URL，比如：

```
security_code_url: global["_g_server"].serviceurl +  "/getCode/"
```

# 总结

这种做法的关键在于：

1、把URL和环境信息放到单独的配置文件中，而不是写死在代码里。同时根据开发环境、生产环境、镜像环境隔离不同的配置文件

2、server端专门写一个服务，把这些配置信息给到客户端页面，客户端页面也不用写死了