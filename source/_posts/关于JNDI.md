title: 关于JNDI
date: 2013-09-24 11:23
categories: java
---
今天跟开涛讨论了一番，对JNDI的概念清楚了一些，本文总结一下 
<!--more-->

# JNDI规范的归属 

JNDI本身是java se最早的一批规范，但是似乎不像JDBC等，有专门的JSR编号，在jsr.org上，找不到专门描述JNDI的规范 

[http://jcp.org/en/jsr/all](http://jcp.org/en/jsr/all) 

另外在JAVA EE规范中第5章，也专门提到了JNDI 

[http://java.chinaitlab.com/Special/java_ee/Chapter5.html](http://java.chinaitlab.com/Special/java_ee/Chapter5.html) 

关于JNDI的资料，还有以下2个官方链接： 

[http://docs.oracle.com/cd/E17802_01/products/products/jndi/javadoc/](http://docs.oracle.com/cd/E17802_01/products/products/jndi/javadoc/) 
[http://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/index.html](http://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/index.html) 

可以这么理解，JNDI规范虽然本身不是JAVA EE规范的一部分，但是JAVA EE规范要求，所有的JAVA EE容器，都需要实现JNDI规范 

但是，就算脱离了JAVA EE环境，只要有专门的JNDI实现，应用程序一样可以使用JNDI的服务，比如SUN公司的遗产，fscontext 

[http://docstore.mik.ua/orelly/java-ent/jenut/ch06_02.htm#ch06-pgfid-982058](http://docstore.mik.ua/orelly/java-ent/jenut/ch06_02.htm#ch06-pgfid-982058) 

# JNDI规范的价值 

在EJB2是企业应用开发主流技术的时候，JNDI很有用 

[http://blog.csdn.net/zhaosg198312/article/details/3979435](http://blog.csdn.net/zhaosg198312/article/details/3979435) 

但是现在，java ee without EJB的风潮已经兴起很久了，特别是以spring为代表的IOC容器大行其道，通过IOC容器已经可以实现组件注入的功能，JNDI规范的应用已经比以前少得多了 

很多近年才接触JAVA EE的开发者，甚至都没有听过JNDI的概念；大量的简单应用，也没有涉及到JNDI 

以下几个介绍JNDI的链接，似乎也不错 

[http://blog.csdn.net/zhaosg198312/article/details/3979435](http://blog.csdn.net/zhaosg198312/article/details/3979435) 
[http://stackoverflow.com/questions/1350816/what-is-the-purpose-of-jndi](http://stackoverflow.com/questions/1350816/what-is-the-purpose-of-jndi) 

我个人有2个理解： 

第一，JAVA EE对一个端到端开发过程中，涉及到的角色进行了划分，一种是写应用的人，叫Developer；另一种是部署应用的人，叫Deployer。JAVA EE规范认为，作为developer，无需关心数据源来自哪里，这是deployer的职责。所以developer只需要写：
```
Context context = new InitialContext();
dataSource = (DataSource) context.lookup("Database");
```

但是，实际上现在的开发过程中，这2者似乎都是程序员的事，并没有这么明确的分工，所以JNDI的这层意义被削弱了。但是，将资源获取，和资源配置分离开的思想，依然是正确的，也是主流的做法 

第二，JNDI起到了很好的抽象层的作用。还是上面的那段代码，就算不考虑角色分工，代码本身也是非常简洁和优雅的。现在我们一般通过spring管理数据源，将DataSource定义为一个bean，然后注入到DAO对象，或者DAO代理对象中，这种做法其实起源于JNDI，历史远远早于IOC容器之前 

# 其他 

对于JNDI，Java Naming and Directory Interface中，“命名”和“目录”的理解 

命名：JNDI的本质，就是为了容易地获取应用外部的种种资源。相当于有一个外部服务，托管了应用将会用到的所有资源。而应用要获取资源，就需要传递一个唯一的标识。就像使用Map数据结构，需要传递一个key，才能获取value。对JNDI也一样，资源的唯一标识，就是资源的命名，这是JNDI里Naming的含义 

目录：设想JNDI服务管理了大量的资源，当然就希望有一个方式对资源进行分类，就像硬盘里的文件多了，也会建立子目录来管理是一个道理。Context接口也提供了createSubcontext()、destroySubcontext()等方法，来创建目录。这是JNDI里Directory的含义 上面也是我看了一堆资料，然后猜测的，不一定对，但是基本能自圆其说 

另外，JNDI可以跨应用地获取资源，以实现分布式，这个也是比Spring这样的普通IOC容器强的地方，当然是否用得到，则是另一回事。“跨应用获取资源”的范围可大可小，部署在同一个容器中的不同应用，获取global resource也是跨应用；获取不同服务器上的资源，也是跨应用，但是到目前为止，我还没有见过这种用法