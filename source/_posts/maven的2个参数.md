title: maven指定pom.xml路径，及多线程构建
date: 2013-09-24 11:13
categories: java
---
本文介绍如何指定maven的pom.xml位置，以及开启多线程build
<!--more-->

# 指定pom文件路径 

```
mvn -f filePath
```

如果不加-f参数，则是默认在当前目录下查找pom.xml文件。如果要改变这一默认行为，可以用-f参数。当用hudson作为CI系统，并且工程是平行目录结构的时候很重要，因为hudson取完代码之后的目录结构是这样的：

![](http://dl.iteye.com/upload/attachment/0074/7162/112b7400-4d1f-331f-84ee-985712c7ca50.png)

在当前目录下，找不到pom.xml，因此会报错，需要配置pom.xml的路径
 
![](http://dl.iteye.com/upload/attachment/0074/7158/4d68b004-56ca-3642-bc4b-1329573604d0.png)

如上配置之后，实际的命令行如下： 

![](http://dl.iteye.com/upload/attachment/0074/7160/d14c92c5-392b-3971-b479-00cc4639c4a8.png)

# 多线程构建 

```
mvn -t 8
```

这个参数可以启动多个线程同时构建，目的是加快构建的速度。对于4核CPU，建议开启8个线程 不过在我的机器上，倒是没有感觉变快多少