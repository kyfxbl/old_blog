title: JPDA简单总结
date: 2013-09-24 11:13
categories: java 
---
我们平时经常使用的debug功能，其实背后大有文章。本文简要介绍JPDA
<!--more-->

# 参考资料

要深入了解，可以看看这个系列的文章：
[JPDA介绍](http://www.ibm.com/developerworks/cn/views/java/libraryview.jsp?search_by=%E6%B7%B1%E5%85%A5+Java+%E8%B0%83%E8%AF%95%E4%BD%93%E7%B3%BB)
 
# java平台调试的特点 

java平台的调试，与其他平台有很大的区别。以C/C++的调试为例，目前比较流行的调试工具是GDB和微软的Visual Studio自带的debugger 

首先，必须编译一个“debug”模式的程序，这个会比“release”模式的程序大很多（因为在elf中包含了大量的debug section） 

其次，在调试过程中，debugger将会深层介入程序的运行，获取运行时的信息，并将这些信息返回。这种介入对运行的效率和内存占用都有一定的需求。基于这些需求，这些Debugger本身创建和管理了一个运行态，因此体积都比较大 

而Java则不同，由于Java的运行态交给虚拟机管理，因此作为Java的Debugger无需再自己创造一个可控的运行态，而仅仅需要去操作虚拟机就可以了 

JPDA就是一套为调试和优化服务的虚拟机的操作工具，其中，JVMTI是整合在虚拟机中的接口，JDWP是一个通讯层，而JDI是前端为开发人员准备好的工具和运行库 

# JPDA

JPDA（Java Platform Debugger Architecture）是Java平台调试体系结构的缩写。由3个规范组成，分别是JVMTI(JVM Tool Interface)，JDWP(Java Debug Wire Protocol)，JDI(Java Debug Interface) 

既然是规范，当然就有实现。Sun公司自己在jdk中提供了一套实现，比如java调试工具jdb，就是sun公司提供的JDI实现 

其他厂商也可以提供自己的实现。目前，大多数的JDI实现都是通过Java语言编写的。比如，eclipse IDE，它的两个插件org.eclipse.jdt.debug.ui和org.eclipse.jdt.debug与其强大的调试功能密切相关，其中前者是eclipse调试工具界面的实现，而后者则是JDI的一个完整实现 

JPDA定义了一个完整独立的体系，它由三个相对独立的层次共同组成，而且规定了它们通信的接口。这三个层次由低到高分别是JVMTI，JDWP以及JDI。这三个模块把调试过程分解成几个概念：调试者（debugger）和被调试者（debuggee），以及它们中间的通信器 

被调试者运行于我们想调试的Java虚拟机之上，它可以通过JVMTI这个标准接口，监控当前虚拟机的信息 

调试者定义了用户可使用的JDI调试接口，通过这些接口，用户可以对被调试虚拟机发送调试命令，同时调试者接受并显示调试结果 

在调试者和被调试者之间，调试命令和调试结果，都是通过JDWP的通讯协议传输的。所有的命令被封装成JDWP命令包，通过传输层发送给被调试者，被调试者接收到JDWP命令包后，解析这个命令并转化为JVMTI的调用，在被调试者上运行。类似的，JVMTI的运行结果，被格式化成JDWP数据包，发送给调试者并返回给JDI调用 

这3个模块的关系和作用如下图： 

![](http://dl.iteye.com/upload/attachment/0074/9379/c0ddd67f-5fb1-3bcb-9da1-ae09907c26cd.png)
![](http://dl.iteye.com/upload/attachment/0074/9377/098593d0-c4ad-3f5b-8fce-3c729c99a305.png)

# JVMTI 

JVMTI是一套由虚拟机直接提供的native接口，它处于整个JPDA体系的最底层，所有调试功能本质上都需要通过JVMTI来提供 

通过这些接口，开发人员不仅能调试在该虚拟机上运行的Java程序，还能查看它们运行的状态，设置回调函数，控制某些环境变量，从而优化程序性能 

JVMTI的前身是JVMDI和JVMPI，它们原来分别被用于提供调试和性能调优的功能。在J2SE 5.0之后JDK取代了JVMDI和JVMPI这两套接口，JVMDI在最新的Java SE 6中已经不提供支持，而JVMPI也计划在Java SE 7后被彻底取代 

JVMTI并不一定在所有的Java虚拟机上都有实现，不同的虚拟机的实现也不尽相同。不过在一些主流的虚拟机中，比如Sun和IBM，以及一些开源的如Apache Harmony DRLVM中，都提供了标准JVMTI实现 

# JDWP 

JDWP是通讯交互协议，它定义了调试器和被调试程序之间传递信息的格式。它详细完整地定义了请求命令、回应数据和错误代码，保证了前端和后端的JVMTI和JDI的通信通畅 

比如在Sun公司提供的实现中，它提供了一个名为jdwp.dll（jdwp.so）的动态链接库文件，这个动态库文件实现了一个Agent，它会负责解析前端发出的请求或者命令，并将其转化为JVMTI调用，然后将JVMTI函数的返回值封装成JDWP数据发还给后端 

另外，这里需要注意的是JDWP本身并不包括传输层的实现，传输层需要独立实现，但是JDWP包括了和传输层交互的严格的定义。在Sun公司提供的JDK中，在传输层上，它提供了socket方式，以及在Windows上的shared memory方式 

这里要说明一下debugger和target vm。target vm中运行着我们希望要调试的程序，它与一般运行的Java虚拟机没有什么区别，只是在启动时加载了Agent JDWP从而具备了调试功能。而debugger就是调试器，它向运行中的target vm发送命令来获取target 

vm运行时的状态和控制Java程序的执行。Debugger和target vm分别在各自的进程中运行，他们之间的通信协议就是JDWP 

JDWP是语言无关的，也就是说理论上我们可以选用任意语言实现JDWP。然而实际上，在Target vm端，JDWP模块必须以Agent library的形式在JVM启动时加载，并且它必须通过JVM提供的JVMTI接口实现各种debug的功能，所以必须使用C/C++语言编写。而debugger端就没有这样的限制，可以使用任意语言编写，只要遵守JDWP规范即可 

JDWP大致分为两个阶段：握手和应答。握手是在传输层连接建立完成后，做的第一件事：Debugger发送14 bytes的字符串“JDWP-Handshake”到target vm；Target vm回复“JDWP-Handshake” 

握手完成之后，debugger就可以向target vm发送命令了。JDWP 是通过命令（command）和回复（reply）进行通信的，这与HTTP有些相似。JDWP本身是无状态的，因此对command出现的顺序并不受限制 

JDWP有两种基本的包（packet）类型：命令包（command packet）和回复包（reply packet） 

Debugger和target vm都有可能发送command packet。Debugger通过发送command packet获取target vm的信息以及控制程序的执行；Target vm通过发送command packet通知debugger某些事件的发生，如到达断点或是产生异常 

Reply packet是用来回复command packet该命令是否执行成功，如果成功，reply packet还有可能包含command packet请求的数据，比如当前的线程信息或者变量的值；从target vm发送的事件消息是不需要回复的 

还有一点需要注意的是，JDWP 是异步的：command packet的发送方不需要等待接收到reply packet就可以继续发送下一个command packet 

# JDI简介 

JDI是三个模块中最高层的接口，在多数的JDK中，它是由Java语言实现的。通过它，调试工具开发人员就能通过前端虚拟机上的调试器来远程操控后端虚拟机上被调试程序的运行，JDI不仅能帮助开发人员格式化JDWP数据，而且还能为JDWP数据传输提供队列、缓存等优化服务 

从理论上说，开发人员只需使用JDWP和JVMTI即可支持跨平台的远程调试，但是直接编写JDWP程序费时费力，而且效率不高。因此基于Java的JDI层的引入，简化了操作，提高了开发人员开发调试程序的效率 

# 一个实际的调试过程 

启动一个jboss的时候，如果直接启动，不加参数，则jvm不会挂载JDWP模块，那么前端的debugger就无法连上此jvm 

因此需要加上参数： 
```
JAVA_OPTS="-Xrunjdwp:transport=dt_socket,address=8787,server=y,suspend=n"
```
 
这样jboss启动jvm时，就会挂载上JDWP模块，这样就支持调试了 

然后在eclipse里启动debugger，也就是JDI的eclipse实现 

![](http://dl.iteye.com/upload/attachment/0074/9415/037406e6-a208-347f-b88c-40363a951417.png)

这个时候实际上有2个进程，一个是jboss开启的jvm，也就是target jvm（debuggee）；另一个是eclipse调试工具开启的jvm，也就是debugger 

另外一种情况，考虑在eclipse中把web应用发布到内置的jboss插件里，然后以debug模式启动 

![](http://dl.iteye.com/upload/attachment/0074/9418/b4df12ed-561d-37d8-835e-e6553df726cd.png)

eclipse的server plugin会自动开启一个挂载了JDWP的jvm，同时也启动一个debugger，不过传输层我估计应该用的是共享内存的方式，就不需要通过socket了 

第三种情况，在eclipse中直接用debug按钮运行一个程序 

![](http://dl.iteye.com/upload/attachment/0074/9423/063210dd-3216-313c-a67f-0b44cb56d8a4.png)

这样的话，eclipse会以类似jdb的方式运行这个类，同时也启动一个debugger，传输层可能也是用共享内存的方式 

总的来说，无论以何种方式，debugger和target vm一定分别在2个不同的进程中，也就是说，处于2个独立的jvm里

