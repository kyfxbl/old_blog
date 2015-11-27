title: RPC简介，及与web service的对比
date: 2013-09-24 11:14
categories: java 
---
最近分析的这个系统，逻辑架构中有一层是RPC interface。之前对RPC不熟悉，就上网搜索了一下资料，在此总结一下
<!--more--> 
 
# RPC原理
 
RPC是Remote Procedure Calling，远程过程调用的缩写。RPC总的来说是一个Client/Server的结构，提供服务的一方称为Server，消费服务的一方称为Client 

下图是本地过程调用，所有的过程都在本地服务器上，依次调用即可 

![](http://dl.iteye.com/upload/attachment/0077/7327/c22da7f0-828b-3cf8-a59c-02f6d82aff55.png)

下图则是所谓的远程过程调用，需要在Client和Server中交互 

![](http://dl.iteye.com/upload/attachment/0077/7329/208b92ad-c7ea-3026-a4f5-6ebae0c4dbd0.png)

因此，两种调用方式，会产生什么区别呢？ 

1、网络传输的开销，和编程的额外复杂性 
2、本地过程调用中，过程在同一块物理内存中，因此就可以传递指针了。而远程过程调用则不能，因为远程过程与调用者运行在完全不同的地址空间中 
3、远程过程不能共享调用者的环境，所以它就无法直接访问调用者的I/O和操作系统API 

简单来说，就是远程过程调用会比本地过程调用复杂。除了性能的额外开销之外，编程也复杂得多。可以想到，交互双方需要能够封装数据结构，理解协议，处理连接等等，确实是很麻烦的。可能一个很简单的调用，却需要做很多的编程工作。所以，为了简化RPC调用的编程，就提出了一个RPC的标准模型 

下面是RPC的原理草图 

![](http://dl.iteye.com/upload/attachment/0077/7331/05bd0743-6d8e-35ae-b51f-612aa3a84b1f.png)

可以看到，该模型中多了一个stub的组件，这个是约定的接口，也就是server提供的服务。注意这里的“接口”，不是指JAVA中的interface，因为RPC是跨平台跨语言的，用JAVA写的客户端，应该能够调用用C语言提供的过程 

对客户端来说，有了这个stub，RPC调用过程对client code来说就变成透明的了，客户端代码不需要关心沟通的协议是什么，网络连接是怎么建立的。对客户端来说，它甚至不知道自己调用的是一个远程过程，还是一个本地过程 

然后，前面说的理解协议，处理连接的工作，总是要有人做的，这个工作就是在下面的RPC Interface里完成的 

# 与web service的对比

最近几年，遇到这种场景（需要调用远程机器上的服务），往往会考虑用web service来完成，其实我认为web service和RPC是非常相像的，下面是web service的原理草图 

![](http://dl.iteye.com/upload/attachment/0077/7333/1dc09dba-474f-3740-8310-013a959a30cd.png)

对比一下RPC草图，就会发现非常的接近。在组件层次，和交互时序上完全没有差别，只是方框内的字不一样，但是实际上承担的职责却是完全对应的 

web service接口就是RPC中的stub组件，规定了server能够提供的服务（web service），这在server和client上是一致的，但是也是跨语言跨平台的。同时，由于web service规范中的WSDL文件的存在，现在各平台的web service框架，都可以基于WSDL文件，自动生成web service接口 

下面的web service框架，根据所选的平台有所不同，比如在JAVA平台中，现在最流行的是apache的cxf框架。它做的事情也和RPC Interface是一样的，负责解析协议（SOAP协议），负责处理连接（建立HTTP连接） 

因此RPC和web service非常得接近，只是RPC的传输层协议，以及应用层协议，可以自行实现，所以选择的余地更大一点。和web service有很多成熟框架可供选择一样，RPC也有很多现成的框架可供选择，比如在JAVA平台上有nfs-rpc等 

总结来说，要实现远程过程调用，需要有3要素： 

1、server必须发布服务 
2、在client和server两端都需要有模块来处理协议和连接 
3、server发布的服务，需要将接口给到client 

当然，应用协议是什么样的，怎么连接，服务接口怎么给到client，是可以自行实现的，选择余地很大，但是RPC协议提供了一种标准的建议

# 该系统的实现

最后回到我最近正在分析的系统上来说。本文一开始就提到，它的架构中有一个RPC Interface，它将server服务发布给client的方式，是向client提供了API，对JAVA平台的程序员来说，就是一个xxx.jar 

这个jar包里，有2部分内容： 

1、client stub，包括接口和封装过的数据结构。即ServerService，和XXXForm、XXXFilter等。对于client程序员来说，就只需要调用ServerService.xxxx()的方法，并组装XXXForm对象作为参数即可，类似

```
public interface ServerService{

    public XXXFilter giveMeTheFilter(XXXForm form);

}
```

程序员只需要关心怎么封装合适的XXXForm，以及什么时候调用giveMeTheFilter()方法即可，底层的协议，server端的实现，对client程序员来说都是透明的 

2、RPC Interface层的实现。这部分就做了协议解析、连接处理、异常处理等，但这部分类，是不对client程序员开放的