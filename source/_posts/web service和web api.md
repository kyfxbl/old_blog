title: web service和web api
date: 2013-09-24 10:59
categories: java
---
曾经对web service和servlet的理解，现在回头看，比较幼稚。实际上，servlet是java平台的规范之一，可以用于实现web api。而web service是web api的一种规范，甚至可以说，是一种过时的规范
<!--more-->

# 相同点

有以下相同点

## 流程相似

客户端（可以是浏览器或者应用程序）访问指定的地址，服务端（web service或者servlet）接受参数，在内部进行逻辑处理，然后返回一个结果给客户端。这个过程是相同的，所以上面说了，用web service可以做到的事情，用servlet一样可以做到 

## 应用协议基本相同

servlet是基于HTTP协议，web service是基于HTTP协议之上的SOAP协议
 
# 不同点

有以下不同点

## 跨平台

servlet在服务端是只能用java实现的（本身就是JavaEE规范的一部分），而web service的服务端可以用任何语言来实现。相应的，servlet只能部署在Servlet容器内，而web service则无此限制 

## 数据格式

servlet返回的是纯文本，而web service返回的是语义更加丰富的XML，而且可以是有类型的 

## 调用方式

servlet的地址，如果不公开声明，客户端是不知道的。比如我们这个项目，提供servlet的子系统，对终端声明了servlet的地址，终端才知道这个URL，才知道往哪里调。而web service，可以通过WSDL，对外公开发布自己的地址 

## 通用规范

web service的请求和响应，都有一套XML规范（标准、协议），所以只要遵循web service规范，任何人或者说任何程序都能读懂web service的请求和响应。而如果用servlet的话，当然也可以自己定义一套输入和输出的XML格式，但是这个格式除了你自己，或者组织内部，是没有人懂的。所以把这个servlet放到网上，根本没有用，因为别人不知道怎么按照你的要求传递参数给你，也不知道怎么解析你的返回值。从这个角度说，web service是通用的规范 

## 实现

servlet是JavaEE规范的一部分，定义了一组API，其实现依赖各应用服务器厂商，比如jboss、tomcat、WAS等，但无论其如何实现，都是基于JAVA平台的。而web service是一种定义了“网络服务”如何提供和使用的规范，没有规定其实现的平台，所以具有跨平台的特点 

# 总结

总结来说，servlet和web service不是一个层面的东西，虽然有很多共同点，但是并不容易放在一起来比较。 如果有跨平台的需求，或者需要开放给网络上的其他组件调用，那么选择web service是不错的，因为其没有绑定JAVA平台，更重要的是，web service已经有了输入输出的规范，就节省了定义和推广的成本 

但是，如果只是在一个有限的系统内要实现互相调用，每个子系统都是基于JAVA平台来实现，这个服务也没有打算发布到互联网的话，那么用servlet或者RMI就更合适，因为更简单，也不需要额外的成本，并且性能也更好