title: 对JAVAEE规范的理解
date: 2013-09-24 11:06
categories: java
---
本文总结对javaee，规范的理解。包括：什么是java ee，各种规范的接口的jar包（及源码）怎么获取，各种规范的实现的jar包（及源码）怎么获取
<!--more-->

# 什么是JAVA EE 

JAVA EE是由一系列规范组成的，规范是由JCP制定的，并且提供了参考实现。规范（Specification）是一系列接口，不包含具体实现 

有以下常见的JAVA EE实现，包括JBOSS、GLASSFISH等。而tomcat是一个servlet容器，实现了servlet规范、jsp规范。但是它并没有实现EJB、JMS、JPA等规范，所以tomcat不是一个完整的JAVA EE实现 

在oracle网站上，下载JAVA EE SDK时，会同时下载GLASSFISH，也就是同时下载了JAVA EE SDK，及一个JAVA EE的实现 

![](http://dl.iteye.com/upload/attachment/0071/9851/c46f71a0-4f9c-3b0b-b7cc-b4044f702023.png)

# 怎么获取某个规范的接口的jar包 

直觉上，我觉得既然规范是JCP制定的，那它当然也就应该负责提供接口的jar包及源码，比如jsr-914.jar，jsr-914_source.jar 

然后实现规范的产商，基于这个jar包开发各自的实现；而规范的使用者，也基于这个jar包调用。这样可以保证接口和实现的分离 

不过事实上，好像不是这样的。很多规范的接口jar包，我在www.jcp.org、www.java.net、www.oracle.com上，都找不到下载的链接

有人说是因为从sun把java卖给oracle之后，oracle关闭了很多项目，所以这些jar包都找不着了，我也不知道是不是这样 

总之，我感觉没有一个很方便的途径，可以获取到各种规范的“官方jar包” 

不过有2个办法，都可以做到 

第一个办法，是可以下载一个相关规范的实现，实现里肯定是有接口jar包的。还是拿jsr-914举例，我下载了2个实现，activemq和jboss 

在activemq安装目录的lib目录下，可以找到接口的jar 

![](http://dl.iteye.com/upload/attachment/0071/9859/74e91e81-5c6f-334f-8b42-f87979513c10.png)

在jboss安装目录的/modules/javax/jms/api/main目录下，也可以找到 

![](http://dl.iteye.com/upload/attachment/0071/9864/30946fba-c3a8-3ba2-b0c3-b8acc134d3f9.png)

第二个办法，好像更方便一点。eclipse有一个项目叫eclipse orbit，在这个项目里，可以找到大部分的规范接口jar包 

以下是官方对此项目的说明： 

This project will provide a repository of bundled versions of third party libraries that are approved for use in one or more Eclipse projects. The repository will maintain old versions of such libraries to facilitate rebuilding historical output. It will also clearly indicate the status of the library (i.e., the approved scope of use). The repository will be structured such that the contained bundles are easily obtained and added to a developer's workspace or target platform. 

下载后的jar包，放在eclipse安装目录的plugins目录下，名字看起来比较奇怪 

![](http://dl.iteye.com/upload/attachment/0071/9867/39b26bc3-f15b-3c5c-9393-4b975f05241d.png)

通过这2种方式，都可以得到规范的接口jar包，把它们加入到eclipse的工程里看一下： 

![](http://dl.iteye.com/upload/attachment/0071/9872/73db9653-6e1b-3cd6-8600-709294f2a869.png)

![](http://dl.iteye.com/upload/attachment/0071/9874/fc0335cc-c38a-3d29-b837-2d20d41a44a0.png)

![](http://dl.iteye.com/upload/attachment/0071/9876/6de8b4df-01fb-3537-86f9-b7fbceb98e80.png)

可以看到3个jar包中的内容基本是一样的，根据某些网友的说法，所有这些jar包，都是经过了JCP认证的，所以都可以直接使用。那我理解这就相当于JCP偷了懒，本来这个jar包应该是由它提供的，但是它没有这么做。而是由各实现提供商来提供这个jar包，JCP只负责认证 

# 怎么获取某个规范的接口的jar包的源码 

其实搞清楚了上面一个问题，这个问题就很简单了

orbit项目对每个接口jar包，都提供了相应的源码。所以如果是通过orbit得到jar包，那也就一定能够得到源码 

如果是通过下载实现的方式获取到接口jar包，那么如果这个实现是开源的，就能得到相应的源码；如果不是开源的实现，那么就得不到源码了 

比如说tomcat是比较方便的，可以直接下载并解压，得到apache-tomcat-7.0.23-src，其中的java目录，就是各种源码 

![](http://dl.iteye.com/upload/attachment/0071/9880/6f3597c0-42cf-32da-8472-260fd18214dc.png)

jboss也是开源的，不过没有tomcat那么方便，需要下载以后自己再跑脚本进行编译 

不过这里有一点要澄清一下，就是一般来说，开发者是不需要用到接口jar包的源码的 

[](http://java.net/jira/browse/GLASSFISH-11389)，这个帖子里说到： 

To assist developers to do what? The corresponding JARs are stripped (no bytecode for methods) and are meant to be used only for compilation. IDEs like NB or Eclipse bundle a zip for the javadoc. What other use case do you need to get the src? Debugging? In this case, these jars are not used for execution so also not for debugging. In the case of debugging, I think there is a complete GF src somewhere to step into the real code you execute from the real non stripped gf jars. 

接口jar包的源码是移除的，只是用于编译。一般的IDE都有zip文件来提供javadoc。对于需要debug的场景，需要源码的也应该是实现类，而不是接口 

[](http://stackoverflow.com/questions/7457810/how-to-get-the-source-code-for-the-javaxjavaee-api-6-0-jar)，这个帖子也表达了类似的意思： 

The purpose of the javaee-api module is to satisfy compile-time dependencies (That is why the Maven scope is set to provided). The module contains interface declarations (or contract) which must be satisfied by the J2EE container you plan to use. If you really need/want to see the source code, I'd suggest taking a look at one of the open source J2EE containers. 

接口jar包的主要目的是满足编译时的依赖 所以，一般来说，开发者是不需要用到接口jar包的源码的，无论是为了javadoc或者debug，都不需要。当然，你可以说我就是想看看，那可以用上面说的方法获取到 

# 怎么获取某个规范的实现的jar包/源码 

搞清楚了上面3个问题，这个问题也就解决了。比如要得到tomcat的servlet实现的jar包，就下载tomcat，在其中的lib目录下就有 

![](http://dl.iteye.com/upload/attachment/0071/9886/2c499218-f055-32d0-bc6e-faae73a29197.png)

然后，如果你选择的实现是开源的，那就再下载一下其源码发布包，就可以得到源码了 

# 题外话 

在eclipse开发JAVA EE应用，在编译时需要引入所需的jar包，这跟最终的目标容器是有关系的，比如我打算开发一个部署在jboss中的WEB应用，那么eclipse会自动导入jboss-library 

![](http://dl.iteye.com/upload/attachment/0071/9891/d82347c5-d840-3743-8adc-b6bfc01921e2.png)

如果是打算部署在tomcat里，那么eclipse会导入tomcat-library 

![](http://dl.iteye.com/upload/attachment/0071/9895/07c7e53d-a5af-36ac-abd6-f455142862f8.png)

可以看到，这2个库是不一样的。但是，这个时候导入的jar包，只是为了满足编译时的需要，这些jar包并不会被打入最后的.war包里（WEB-INF/lib下的才会）。这些Runtime Library，在运行时会从容器的classpath里被找到并加载 

理论上来说，开发一个servlet应用，就算开发时选择的是JBoss Runtime Library，但是只引用了接口，没有依赖jboss具体的实现类的话，那么打出的.war包，在运行时扔到tomcat里也是可以跑的。原因前面说过了，双方的servlet-api.jar，都是经过JCP认证的，所以能够保证一致性。但是如果编码时依赖了JBoss提供的实现类，那到tomcat里就绝对跑不了了 

但是，如果开发的是完整的JAVA EE应用，比如依赖了JMS、EJB的接口，那扔到tomcat里就不能跑了，还需要额外的JMS中间件的支持，原因上面也说过了，因为tomcat只是一个servlet容器，部分实现了JAVA EE规范，不是一个JAVA EE的完整实现