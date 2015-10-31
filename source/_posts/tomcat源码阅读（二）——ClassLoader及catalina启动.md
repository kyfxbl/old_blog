title: tomcat源码阅读（二）——ClassLoader及catalina启动
date: 2013-09-24 11:24
categories: 源码阅读 
---
本系列是阅读tomcat源码的总结。本文介绍tomcat中的classloader体系
<!--more-->

# ClassLoader结构 

![](http://dl2.iteye.com/upload/attachment/0086/2693/d9e355b8-980d-3bb1-8964-dfed15889f9a.jpg)

tomcat的ClassLoader模型如上图，主要是为了满足servlet规范中类隔离的要求（见JSR154的Section9.4、9.6、9.7） 

## Bootstrap 

这个类加载器和普通的JAVA应用一样，都是由JVM启动的，加载%JAVA_HOME%/jre/lib下的JAR包，如rt.jar等 

通常情况下，Bootstrap和Extension是分开考虑的，但是在tomcat的ClassLoader体系里，没有将二者区分开。当谈到Bootstrap时，就包括了Bootstrap和Extension 

## System 

System类加载器，也叫App类加载器，一般就是启动应用程序的那个加载器，是根据classpath创建的 

但是在tomcat里，完全忽略了默认的classpath，而是根据指定的参数，直接创建System类加载器，默认情况下，会加载%CATALINA_HOME%/bin目录下的bootstrap.jar、tomcat-juli.jar、commons-daemon.jar 

这些jar包充当了启动入口的角色，但是tomcat真正的核心实现类，不是在这个ClassLoader里加载的，所以后面会提到，源码里调用核心实现类（Catalina等）的方法，必须指定ClassLoader，通过反射完成 

## Common 

这个类加载器是tomcat特有的，对于所有web app可见。这个类加载器默认会加载%CATALINA_HOME%/lib下的所有jar包，这些都是tomcat的核心 

## WebappX 

对于部署在容器中的每一个webapp，都有一个独立的ClassLoader，在这里实现了不同应用的类隔离 

这里的ClassLoader与标准的ClassLoader委托模型不同，当需要加载一个类的时候，首先是委托Bootstrap和System；然后尝试自行加载；最后才会委托Common 

加载顺序如下： 

```
Bootstrap -> System -> WEB-INF/classes -> WEB-INF/lib -> Common
```

# 疑问 

Bootstrap.class和Catalina.class是打在不同的JAR包里的。前者在bootstrap.jar里，后者在catalina.jar里 

并且这2个jar包，是由不同的ClassLoader加载的。前者由System ClassLoader加载，后者由Common ClassLoader加载 

这造成代码比较麻烦，Bootstrap.class要引用Catalina.class的时候，不是直接引用，而是通过反射实现 

我还没搞清楚tomcat这样设计的原因，跟类隔离貌似没有直接关系 

# tomcat启动 

tomcat启动是遵循下面的顺序

## 执行脚本 

tomcat启动是从运行startup.bat脚本开始的，在此脚本中首先会设置一系列环境变量，然后配置参数，最后实际上执行的catalina.bat脚本。所以用catalina.bat start命令，也可以启动tomcat 

## 加载Bootstrap类 

在catalina.bat中，会将classpath设置为%CATALINA_HOME%/bin/bootstrap.jar和%CATALINA_HOME%/bin/tomcat-juli.jar，然后根据此classpath创建System类加载器，加载bootstrap.jar中的Bootstrap.class，执行main()方法 

## 调用Catalina类 

在Bootstrap.class里，读取配置文件（%CATALINA_HOME%/conf/catalina.properties），然后创建Common ClassLoader，加载%CATALINA_HOME%/lib里的所有jar包 

之后根据实际的命令行参数，调用org.apache.catalina.startup.Catalina中相应的方法，比如start()等 

# 参考文档 

[http://tomcat.apache.org/tomcat-7.0-doc/class-loader-howto.html](http://tomcat.apache.org/tomcat-7.0-doc/class-loader-howto.html)