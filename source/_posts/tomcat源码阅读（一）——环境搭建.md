title: tomcat源码阅读（一）——环境搭建
date: 2013-09-24 11:23
categories: 源码阅读 
---
本系列是阅读tomcat源码的总结。本文介绍如何搭建阅读源码的环境
<!--more-->

# 工具准备 

需要SVN、Maven、JDK、Eclipse、M2Eclipse 

# 下载源码及发布包 

源码在：[http://svn.apache.org/repos/asf/tomcat/tc7.0.x/tags/TOMCAT_7_0_27/](http://svn.apache.org/repos/asf/tomcat/tc7.0.x/tags/TOMCAT_7_0_27/) 

发布包在：[http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.27/bin/](http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.27/bin/) 

说明：下载发布包这个步骤是可选的，好处是免得从源码再自行构建，节省时间；另外发布包里的配置文件等，接下来可以直接拿来用，很方便 

# 整理目录 

前面下载得到了源码和发布包，现在要把它们放到同一个目录里，再整理一下，方便后面把它转化成eclipse工程，毕竟后续读源码，以及调试，都要在eclipse里完成 

新建一个单独的目录，叫tomcat7.0.27，然后把刚才下载的源码和发布包都放进去。源码目录重命名为code；发布包重命名为launch 得到的目录结构见下图： 

![](http://dl2.iteye.com/upload/attachment/0086/2206/1e53dfcc-3c1d-3b43-bafc-3907c3c7bff6.jpg)

一会就会把这个目录导入eclipse，变成可运行，可调试的eclipse工程 

# 转换成maven工程 

将附件中的pom.xml放入目录，与code、launch目录平行 得到的目录结构见下图： 

![](http://dl2.iteye.com/upload/attachment/0086/2208/829253aa-44df-338c-a0e5-c3b08a020632.jpg)

说明：这也不是必须的，只是为了方便 

# 导入eclipse 

![](http://dl2.iteye.com/upload/attachment/0086/2210/34685a7e-f014-3926-a263-2bbfa957b7b5.png)

![](http://dl2.iteye.com/upload/attachment/0086/2212/a8f0ef25-73ce-3a4c-87e7-ba5ea1e470fc.png)

导入成功以后，eclipse里的工程目录结构如下图： 

![](http://dl2.iteye.com/upload/attachment/0086/2214/ccffc2cd-aa38-3889-bcad-4d8dd63404c5.png)

接下来就可以在eclipse里运行和调试tomcat了，也可以随意修改源代码，或者自己添加测试用例 

# 启动tomcat 

tomcat启动入口类是：org.apache.catalina.startup.Bootstrap 

平时我们用发布包启动tomcat一般是用脚本startup.bat或者startup.sh，其实就是在脚本中先处理启动参数和系统变量，然后调用这个入口类的main()方法 

所以在eclipse里启动，我们也是直接执行这个类的main()方法，只是模拟脚本，设置一下启动参数和系统变量 

方法1： 在VM arguments中，拷贝以下参数 

```
-Dcatalina.home=launch -Dcatalina.base=launch -Djava.endorsed.dirs=launch/endorsed -Djava.io.tmpdir=launch/temp -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.util.logging.config.file=launch/conf/logging.properties
```

如图： 

![](http://dl2.iteye.com/upload/attachment/0086/2216/4b70caed-8d89-326c-bf80-9146d4f7064a.png)

方法2： 将附件中的启动脚本，拷贝到工程目录下，结构如下图： 

![](http://dl2.iteye.com/upload/attachment/0086/2218/36473b2f-cb20-3294-9faf-b2aee423971c.png)

然后直接在start-tomcat7.launch上右键点击，run就可以 

启动效果如下图： 

![](http://dl2.iteye.com/upload/attachment/0086/2220/d8950116-8435-3228-86c8-6dbbb4fcadce.png)

眼熟，和普通的脚本启动，以及启动嵌入式tomcat的信息都是一样的 

最后用浏览器访问：[http://localhost:8080/examples/](http://localhost:8080/examples/) 

# tomcat7核心架构

![](http://dl2.iteye.com/upload/attachment/0086/2222/26ed3ea9-c6c0-3300-843d-fadc0522a588.png)

| *包名* | *作用* | 
| javax.* | 各种JSR的API，如jsp、servlet、el等 |
| org.apache.catalina | tomcat自身架构 |
| org.apache.coyote | http、ajp协议实现 |
| org.apache.el | EL规范实现 |
| org.apache.jasper | JSP规范实现 |
| org.apche.juli | 日志 |
| org.apache.naming | JNDI实现 |
| org.apache.tomcat | 工具包、XML解析器等 |