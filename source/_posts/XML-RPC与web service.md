title: XML-RPC与web service
date: 2013-09-24 11:23
categories: java 
---
最近其中一项工作，是分析本行业欧洲项目的IT系统集成状况，所以稍微了解了一下XML-RPC，有一些想法记录一下
<!--more--> 

# 原理

只要熟悉web service，要掌握XML-RPC就非常简单，因为它们本质上是一样的。无论以何种方式实现RPC，都需要解决2个问题，第一是传输问题，即系统间必须有通讯链路，消息才能传输；第二是数据格式问题，系统间的数据，必须能够互相解析 

XML-PRC解决这2个问题的方法是：用HTTP作为传输层，用XML传递数据格式 

![](http://dl2.iteye.com/upload/attachment/0085/4304/0e35d6f7-d74b-3f94-875d-8de41f31d3bb.png)

以client的流程举例（server端类似） 

发送XML-RPC请求，会经过以下几个步骤 

1、首先将请求的格式（方法名、方法参数等）封装成XML格式的数据（XML格式是由XML-RPC规范规定的） 
2、然后将XML数据放到HTTP请求体中 

接收XML-RPC响应，会经过以下几个步骤 

1、从HTTP响应中取出XML 
2、将XML解析为本地数据 

# 解决的问题，和没有解决的问题 

前面说过，XML-RPC可以解决2个问题，第一个就是RPC的能力，另一个由于XML-RPC规范中定义的数据结构是通用的，所以也解决了异构系统集成的问题，比如
```
<int>23</int>
```

这个参数，在JAVA平台可以处理为int类型，在.NET平台也可以解析为对应的数据类型 

但是，XML-RPC规范中没有规定安全的问题如何解决，还有如何公开发布一个XML-RPC服务，在规范中也没有明确的说明。但是其实我觉得这也是有利有弊，XML-RPC的规范是非常简单的，上个厕所的时间就可以读完： [XML-RPC规范](http://xmlrpc.scripting.com/spec.html) 

因此，XML-RPC也是相当轻量级的，在各个平台也有成熟的实现（JAVA平台常用的实现是Apache XML-RPC）。所以，在简单的集成场景下，放弃笨重的web service，选择轻量级的XML-RPC，其实也是不错的选择 

# JAVA平台的实现

理解了XML-RPC的原理，实现起来并不困难 

对于client，首先用一个组件将调用请求包装为XML数据，然后用HTTP组件（比如commons-http），封装HTTP消息并发送；收到的响应，也先用HTTP组件处理得到XML，再从XML中还原得到本地数据，继续进行业务处理 

对于server，有一种方式是通过ServerSocket侦听HTTP请求，然后进行处理，但是这样显然太麻烦。所以更好的选择是基于JAVA平台的Servlet规范来实现，将HTTP处理的部分交给servlet容器来完成，只需要关注怎么实现method的路由，以及XML的转换即可 

这里也有2种选择，第一种是为每个XML-RPC服务开发一个单独的Servlet，这样的好处是省略了方法路由的处理；缺点是如果XML-RPC服务很多的话，会有很多的Servlet，并且也不利于动态扩展。第二种办法就是采用“front-end controller”的设计模式，只开发唯一的一个Servlet，在里面进行方法路由，根据client请求的方法名，映射到不同的业务实现类。阅读cxf的源码可以发现，cxf正是采用这种思路来实现 

# 前景 

前面说过，XML-RPC是相当轻量级的异构系统RPC规范，我个人很喜欢轻量级的技术。但是XML-RPC目前基本上被web service取代了，这里面可能有XML-RPC先天不足的原因，另外商业的推广可能是更重要的原因。（web service在前十年得到了很多IT巨头厂商的大力推广）以IBM为典型代表的IT龙头老大很擅长这套，先发明一个BUZZ-WORD，然后围绕着出一系列解决方案，然后赚一波钱，接着再发明下一个BUZZ-WORD，其实回头看看，很多时候不是什么新东西，只是包装得很好 

另一方面，前段时间突然又重新流行起来的RESTful，也是直接批判了XML-RPC，当然说的也不无道理。但是无论如何，我个人的看法是，技术没有好坏之分，只有适用场景。对于简单的集成场景，很多时候XML-RPC可以满足需求，又很简单，那么也可以作为一个选择 

# web service 

web service本质上和XML-RPC完全是一样的，也是遵循本地数据-XML-HTTP这个模型。但是web service在XML这层，采用的是SOAP协议（其实就是更复杂的XML-RPC），原理上毫无区别 

web service有明显的“第二版效应”，竭力想弥补第一个版本的缺失，因而在功能上非常的“丰富”和“复杂”，其中有些是很好的，有些是用不上并且相当笨重的 

web service相对于XML-RPC，在服务发布上有比较明显的优势。通过WSDL，可以清晰地描述一个服务，然后在client，就可以根据这个WSDL，很容易地生成客户端的代码 

另外的一些特性，比如ws-security、wsdd等，给我的感觉是过于笨重了