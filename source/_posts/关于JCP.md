title: 关于JCP
date: 2013-09-24 10:52
categories: java 
---
JCP的小知识
<!--more-->

今天上JCP去下载jdbc的规范，看到有2个链接： 

![](http://dl.iteye.com/upload/attachment/0079/6173/0fa60803-1fa9-3913-9f42-d77323c7d356.jpg)

记得以前下servlet规范时候也看到了，貌似JSR都有这2个，就google了一下，在stack overflow上看到一个完整的解答： 

The difference is in the license that you accept before downloading the specification. I'm surprised you didn't notice this as you careful studied each document! For the JSRs I've checked, the documents are identical—including the implementation license in-lined in the document. The evaluation link offers a "Limited Evaluation License" for evaluating the specification. I think this is aimed at JCP participants, public commentators, and application developers that want to understand the specification. The implementation link offers a license to implementers. They get a royalty-free license to distribute implementations under certain conditions. Paraphrasing loosely, these requirements include: that the implementation completely implements the specification, that the implementation doesn't modify the java package namespace beyond what the specification allows, and that the implementation passes the applicable TCK. 

总的来说，主要就是license上的区别，如果你是为了实现这个规范而去下载的话，就要承诺你的实现将要怎么怎么样

比如说，我要开发一个web app，需要了解servlet规范，我应该下for evaluation；如果我是要实现一个servlet容器，我就应该下for implementation 

JCP全称Java Community Process，是JAVA的一个社区组织，貌似是推动和管理JAVA规范的，java规范叫JSR 

JSR会有不同的阶段，到了最后的Final阶段，JCP还会提供一个RI和TCK 

RI是参考实现，Reference Implementation，也就是该规范实现的参考 

TCK是技术兼容包。当提供商声称他们的实现符合某个JSR，那么就要用TCK检测一下，是否真的符合规范。如果通过了TCK检测，再缴纳商标费，就通过J2EE认证（Authorized Java Licensees of J2EE）