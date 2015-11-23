title: 松鼠书读书笔记（二）——HTTP报文
date: 2013-09-24 11:56
categories: web  
---
松鼠书读书笔记，关于http message
<!--more-->

# 报文的一些术语 

在规范里有这样2组术语，需要知道它们的意思，才能理解后面的内容 

一组是流入/流出，即inbound和outbound。“流入”总是指http message从client agent发往server；“流出”总是指http message从server发往client agent 

另一组术语是上游/下游，即upstream和downstream，http message总是从上游发往下游。两个http节点A和B，如果对http request来说，A是B的上游；那么对http response来说，A就是B的下游 

# http message的结构 

无论是http request还是http response，http message都由3部分组成，分别是starting line、header、entity-body 

entity-body翻译为“实体的主体部分”，很拗口，而且实际上并没有“实体的不是主体的部分”，还不如就翻译成body，或者entity好点 

用apache的http client组件的话，API里有一个getEntity()方法，得到的就是http message的entity-body部分 

# 状态码 

http response的起始行里，会返回一个状态码： 

1xx：信息提示 
2xx：成功 
3xx：重定向 
4xx：客户端请求错误 
5xx：服务端错误 

一般特别常见的就是200、304、404、500等 

# header 

首部就是一行一行的键值对，用冒号分隔，最后用一个空行表示结束。首部对于http message是至关重要的，很多功能都是依靠首部来完成的