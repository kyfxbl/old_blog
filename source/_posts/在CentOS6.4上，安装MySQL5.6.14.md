title: 在CentOS6.4上，安装MySQL5.6.14
date: 2013-10-17 20:58
categories: linux
---
今天在CentOS6.4下装MySQL5.6.14，又是痛苦的一天
<!--more-->

# RPM和yum

CentOS使用的包格式是RPM，所以通常在CentOS上安装软件的方法，就是找到对应的RPM包，然后用rpm命令进行安装。但是软件之间，往往存在依赖的关系，如果全都手动安装，就会很麻烦，所以在CentOS上还有yum工具，用来处理依赖

总的来说，RPM是CentOS上的包格式，而yum则是RPM包的管理工具。类似Google Play Store和APK的关系

如果用yum能正常装好，我建议就用yum，确实方便很多，尽量不要手工用rpm安装。但是有时候yum搞不定，比如版本不满足，或者速度太慢，或者像我今天这样出现了莫名其妙的情况，那就只能用rpm来装了

# 卸载CentOS自带的低版本MySQL

```
$ rpm -qa | grep mysql
```

或者

```
$ rpm -qa | grep MySQL
```

不同版本的mysql，大小写不一样，所以要注意。CentOS6.4自带了一个低版本的mysql（小写），先用
```
$ rpm -e mysql
```

卸载掉

# 下载RPM包

在这个链接下载：[MySQL Resource](http://dev.mysql.com/downloads/mysql/#downloads)

注意版本要选对，我这里选的是Linux Generic，因为没有找到针对CentOS6.4的，另外选32位还是64位，要根据目标硬件的实际情况。然后用WINSCP传到服务器上，解压以后得到很多个rpm

![](http://img.blog.csdn.net/20131017195129734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

后面暂时只会用到MySQL-server那个包

# 安装perl

安装的时候提示没有perl，这就看出RPM依赖麻烦的地方了。好在perl可以用yum装，不需要再去找perl的RPM包。用yum -y install perl完成安装

# 安装MySQL

用命令rpm -ivh /path/MySQL-server-xxx.rpm，自动安装，安装成功应该看到下面这个信息：

```
Preparing...               ########################################### [100%]
1:MySQL-server           ########################################### [100%] 
```

然后检查一下各相关的目录。这里就体现出RPM安装比node和mongodb这样的解压安装麻烦多了，文件零零散散地放在各种地方，主要是：

数据库目录：/var/lib/mysql

配置文件目录：/usr/share/mysql

命令目录：/usr/bin，ll mysql*

![](http://img.blog.csdn.net/20131017195918671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

启动脚本目录：/etc/init.d，ll mysql*

![](http://img.blog.csdn.net/20131017200208093?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

最后检查一下是否有mysql服务了

chkconfig --list

![](http://img.blog.csdn.net/20131017200254500?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

看到了mysql服务，说明已经安装成功了，接下来就可以用service mysql start启动

这里也有个坑，网上大部分帖子都说服务名是mysqld，其实应该是mysql，不知道mysqld是什么版本的

# 启动失败原因排查

启动居然失败了：

![](http://img.blog.csdn.net/20131017200628250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

到/var/lib/mysql目录下，可以看到启动失败的错误日志

![](http://img.blog.csdn.net/20131017200921078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

提示要先运行mysql_upgrade命令，又出现了另外一个错误

![](http://img.blog.csdn.net/20131017201358484?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上网搜索了一下，应该先执行这个命令

mysql_install_db --user=mysql --ldata=/var/lib/mysql/

[root@yilos-dev-image bin]# service mysql start Starting MySQL. SUCCESS!

初始化的root密码是空，注意是MySQL的root，不是CentOS的root

# 安装client

为了在本机管理MySQL，需要把客户端也装起来，过程同server的RPM安装，安装以后才能用mysql命令进行管理。注意作为CLI的mysql和作为service的mysql是不同的

![](http://img.blog.csdn.net/20131017204533328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

几个常用的命令：

show databases;

show tables;

use [dbname];