title: 在CentOS6.4上，编译安装Nginx
date: 2013-10-21 13:01
categories: linux
---
貌似也可以用yum和RPM包安装，本文介绍的是用编译方式安装
<!--more-->

# 获取源码包，解压

![](http://img.blog.csdn.net/20131021114420125?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 配置

```
$ ./configure
```

由于我的CentOS是选择mini安装，没有编译环境，报了如下错误：

![](http://img.blog.csdn.net/20131021115400812?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行以下命令，安装编译环境

```
$ yum install -y gcc gcc-c++ ncurses-devel perl
```

现在报缺少PCRE

![](http://img.blog.csdn.net/20131021115647359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行以下命令，安装PCRE

```
$ yum install -y pcre-devel
```

然后又没有zlib。。继续安装

```
$ yum install -y zlib zlib-devel
```

这次可以了，配置摘要见下：

![](http://img.blog.csdn.net/20131021120302375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 准备MAKE

```
$ yum install -y make
```

# 安装

从上面的摘要可以看出，会装到/usr/local/nginx目录下，可以用下面的命令来自定义

```
$ ./configure --prefix=/home/nginx
$ make
$ make install
```