title: 对web service和cxf的理解
date: 2013-09-24 11:00
categories: java
---
这个项目需要与一个老系统对接，交换业务数据。实际上用更加轻量级的servlet+json来做也是可以的，但是由于老系统要求使用web service，所以也只能接受。于是我就在想，web service和servlet的区别在哪里。本文整理一下思路
<!--more-->

如果一个servlet被互联网上很多应用所调用，那么也就可以认为它是一个广义上的web service了。而狭义上的web service，似乎可以理解成soap+wsdl 

# 主要区别

根据我的理解，web service和一般的servlet（或者其他平台的服务端技术）主要有2个区别： 

## 协议不同

普通的servlet，用的协议是自定义的格式（比如自己定义的plain text或者是xml），或者最近几年比较流行的json。而web service走的协议是soap。这就要求客户端需要自己对传输的内容进行解析，而web service的客户端则省去了这个步骤，因为各平台的web service实现框架（比如java平台有cxf），会处理实体数据和soap的转换 

## 客户端代码开发难度

普通的servlet，客户端代码需要自行开发。比如我知道某个地址提供了一个服务，那么我需要在了解接口协议之后，用httpclient+字符串解析等方式，来自行开发客户端的代码。而web service规范规定了wsdl文件，那么各平台的web service实现框架，可以提供一些工具，由wsdl生成平台相关的代码，省去了客户端自行开发的工作 

基于以上理解，我认为狭义的web service就是soap+wsdl，目的是简化客户端的工作（包括协议解析和客户端调用的代码编写）。这背后依赖的前提就是soap协议和wsdl格式是世界范围内通用的 

# web service的部署描述和运行时

我认为web service分为2个阶段，一个是部署时，一个是运行时。

部署时起作用的主要是wsdl，这个时候调用还没有发生，但是客户端可以根据此文件，生成平台相关的代码（无论是手写还是利用框架提供的工具）。实际运行的时候，没有wsdl都可以，也就是说WSDL是一个部署时描述符，不是运行时组件。相当于WSDL起到的是一个接口文档的作用，只不过这个接口文档的格式是全世界通用的，不是某2个特定系统自己协商的。平时项目开发时写的接口文档，就可以理解成一个不通用的WSDL 

运行时起作用的主要是soap，此时web service框架起到的作用是完成平台相关的代码和soap格式字符串之间转换，以及soap与http之间的转换
 
# cxf的作用

cxf是java平台的web service框架。如上文所说，它在部署时和运行时发挥的是不同的作用 

在部署时，cxf框架可以根据web service接口，生成wsdl文件；也可以根据wsdl文件，反向生成客户端的代码 

在运行时，cxf框架可以完成实体数据和soap格式字符串之间的转换，以及将soap包装成http数据流，或者从http数据流中取出soap 

上述的功能，没有框架同样是能做的。只要对wsdl足够熟悉，完全可以自己编写wsdl文件，或者根据wsdl文件手写代码；同样，只要对soap协议足够熟悉，这个转换也可以自行完成。cxf框架（或者xfire，axis等同类框架）只是提供了一个便利 

# web service最大的价值是跨平台

web service最大的意义还是在于跨平台。因为web service规范规定的数据类型不与具体平台的类型绑定，所以同样一个wsdl文件，各个平台的web service实现可以生成各自平台对应的代码 

如果不是出于跨平台的目的的话，web service我认为没有什么意义。比如说假设web service仅用于java平台，那我发布web service，既要提供wsdl文件，还要解析soap协议，未免就太麻烦了。我不如直接将客户端代码都写好，然后封装成一个jar包，公开提供给需要的系统调用就可以了 

```
public class MyServiceInvoker{

    public static String sayHi(String name){
        // 将name作为http post的内容
        // 将http的目标url设置成我提供服务的地址
        // 发起http请求
        // 将响应结果进行解析，反正格式是我规定的
        return 响应结果;
    }
}
```

然后我把这个类以及相关依赖的类打成一个invoker.jar，提供给需要的应用，那么应用在导入这个jar包后，客户端代码可以直接这么写：

```
String result = MyServiceInvoker.sayHi("kyfxbl");
```

岂不是比发布wsdl要简单很多，但是这样的话，.net平台就无法调用这个服务了。所以说，web service的根本意义就在于跨平台，否则就有更方便的方法来实现组件共享的目的