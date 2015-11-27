title: tomcat的classloader机制
date: 2013-09-24 11:13
categories: java 
---
tomcat的classloader机制分析
<!--more-->

# 参考文档

[tomcat classloader](http://tomcat.apache.org/tomcat-7.0-doc/class-loader-howto.html) 

# 过时的模型 

在网上搜索“tomcat classloader”，很容易搜索到下图，但是这是一个过时的模型 

![](http://dl.iteye.com/upload/attachment/0075/7815/7cc11eb2-032a-3817-860f-899afc3ae263.png)

这个模型是在tomcat5.x使用的，可以看一下tomcat5.x的目录结构 

![](http://dl.iteye.com/upload/attachment/0075/7869/72816da8-342b-343e-943f-07ee03107878.png)

再对比一下tomcat7.x的目录结构 

![](http://dl.iteye.com/upload/attachment/0075/7820/28ae1d7f-5383-37d0-9c4f-cfb0a8247e50.png)

可以看到，5.x里的server、shared、common目录，在7.x中已经废弃了。所以上图中的ClassLoader模型也是过时的 

在tomcat7.x里，ClassLoader的模型应该是下图这样： 

![](http://dl.iteye.com/upload/attachment/0075/7827/4bfd2e34-b46f-35fc-bc60-054010f2a980.png)

至于这个模型中，各个ClassLoader的具体作用，下文会说明 

# JVM默认的classloader机制 

jvm默认定义了三种classloader，分别是bootstrap classloader、extension classloader、system classloader 

bootstrap是jvm的一部分，用C写的，每一个java程序都会启动它，去加载%JAVA_HOME%/jre/lib/rt.jar 

extension也差不多，它会去加载%JAVA_HOME%/jre/lib/ext/下的类 

system则是会去加载系统变量CLASSPATH下的所有类 

这3个部分，在上面的tomcat classloader模型图中都有体现。不过可以看到extension没有画出来，可以理解为是跟bootstrap合并了，都是去%JAVA_HOME%/jre/lib下面加载类 

另外，java的classloader一般是采用委托机制，即classloader都有一个parent classloader，当它收到一个加载类的请求时，会首先请求parent classloader加载，如果parent classloader加载不到，才会自己去尝试加载（如果自己也加载不到，则抛出ClassNotFoundException）

采用这种机制的目的，主要是从安全角度考虑。比如用户自己定义了一个java.lang.Object，把jdk中的覆盖了，那显然是有问题的 

当然，这个机制不是绝对的，比如在OSGi中，就故意违反了这个模式。后面可以看到，tomcat里的webapp classloader也违反了这个规定 

# tomcat为什么要自定义classloader 

主要有2个目的，首先是要实现servlet规范中对类加载的要求，其次是实现不同web app的类隔离 

servlet规范中对类加载要求如下： 

This specification defines a hierarchical structure used for deployment and packaging purposes that can exist in an open file system, in an archive file, or in some other form. It is recommended, but not required, that servlet containers support this structure as a runtime representation. Web applications can be packaged and signed into a Web ARchive format (WAR) file using the standard Java archive tools. For example, an application for issue tracking might be distributed in an archive file called issuetrack.war. When packaged into such a form, a META-INF directory will be present which contains information useful to Java archive tools. This directory must not be directly served as content by the container in response to a Web client’s request, though its contents are visible to servlet code via the getResource and getResourceAsStream calls on the ServletContext. Also, any requests to access the resources in META-INF directory must be returned with a SC_NOT_FOUND(404) response. 

# 各classloader详细说明 

## Bootstrap 

This class loader contains the basic runtime classes provided by the Java Virtual Machine, plus any classes from JAR files present in the System Extensions directory ($JAVA_HOME/jre/lib/ext). Note: some JVMs may implement this as more than one class loader, or it may not be visible (as a class loader) at all. 

## System 

这个classloader通常是由CLASSPATH这个环境变量初始化的，通过这个classloader加载的所有类，都对tomcat自身的类，以及所有web应用的类可见。但是，标准的tomcat启动脚本（$CATALINA_HOME/bin/catalina.bat），完全无视默认的CLASSPATH环境变量，而是加载了以下3个.jar 

$CATALINA_HOME/bin/bootstrap.jar — Contains the main() method that is used to initialize the Tomcat server, and the class loader implementation classes it depends on. 

$CATALINA_HOME/bin/tomcat-juli.jar — Logging implementation classes. These include enhancement classes to java.util.logging API, known as Tomcat JULI, and a package-renamed copy of Apache Commons Logging library used internally by Tomcat. 

$CATALINA_HOME/bin/commons-daemon.jar — The classes from Apache Commons Daemon project.（这个类不是直接在$CATALINA_HOME/bin/catalina.bat里加进来的，不过在bootstrap.jar的manifest文件中包含进来了） 

## Common 

这个classloader加载的类，对tomcat的类和web app的类都是可见的。通常来说，应用程序的类不应该放在这里。该加载器的加载路径是在$CATALINA_BASE/conf/catalina.properties文件里，通过common.loader属性来定义的，默认是：

```
common.loader=${catalina.base}/lib,${catalina.base}/lib/*.jar,${catalina.home}/lib,${catalina.home}/lib/*.jar
```

By default, this includes the following: 

annotations-api.jar — JavaEE annotations classes. 
catalina.jar — Implementation of the Catalina servlet container portion of Tomcat. 
catalina-ant.jar — Tomcat Catalina Ant tasks. 
catalina-ha.jar — High availability package. 
catalina-tribes.jar — Group communication package. 
ecj.jar — Eclipse JDT Java compiler. 
el-api.jar — EL 2.2 API. 
jasper.jar — Tomcat Jasper JSP Compiler and Runtime. 
jasper-el.jar — Tomcat Jasper EL implementation. 
jsp-api.jar — JSP 2.2 API. 
servlet-api.jar — Servlet 3.0 API. 
tomcat-api.jar — Several interfaces defined by Tomcat. 
tomcat-coyote.jar — Tomcat connectors and utility classes. 
tomcat-dbcp.jar — Database connection pool implementation based on package-renamed copy of Apache Commons Pool and Apache Commons DBCP. 
tomcat-i18n.jar — Optional JARs containing resource bundles for other languages. As default bundles are also included in each individual JAR, they can be safely removed if no internationalization of messages is needed. 
tomcat-jdbc.jar — An alternative database connection pool implementation, known as Tomcat JDBC pool. See documentation for more details. 
tomcat-util.jar — Common classes used by various components of Apache Tomcat. 

## WebappX 

该classloader加载所有WEB-INF/classes里的类，以及WEB-INF/lib里的jar 

该classloader就有意违反了上述的委托模型，它首先看WEB-INF/classes和WEB-INF/lib里是否有请求的类，而不是委托parent classloader去加载。但是，JRE里定义的类不能被覆盖（比如java.lang.String），以及Servlet API会明确地被忽略。 前面说的bootstrap、system、common，都遵循普通的委托模型 

## 总结

从web app的角度来看，类或者资源加载是按照以下的顺序来查找的： 

Bootstrap classes of your JVM（rt.jar） 
System class loader classes（bootstrap.jar、tomcat-juli.jar、commons-deamon.jar） 
/WEB-INF/classes of your web application 
/WEB-INF/lib/\*.jar of your web application 
Common class loader classes （在$CATALINA_HOME/lib里的jar包）