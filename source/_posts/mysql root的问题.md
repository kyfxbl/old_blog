title: mysql root的问题
date: 2013-10-21 20:58
categories: database  
---
昨天装好了MySQL，暂时没有设置root密码（root密码为空），暂时跑着也没问题。不过今天对数据库做了一些操作，突然发现用mysql登陆进去之后，看不到内部管理用的数据库"mysql"了，用命令use mysql，显示如下错误信息：Access denied for user ''@'localhost' to database 'mysql'。google了一下，说是需要设置root密码

# 用安全模式启动mysql

```
service mysql stop

/usr/bin/mysqld_safe --skip-grunt-tables
```

# 设置root密码

新开一个SSH连接

```
mysql

mysql> use mysql

mysql> update user set password=password('the_root_password') where user = 'root';

mysql> flush privileges;

mysql> exit
```

# 恢复普通模式启动mysql

```
service mysql stop

service mysql start
```

# 登陆

现在再用mysql或者mysql -u root登陆，就会报错了，错误信息：

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

需要用mysql -u root -p访问，如果输错密码，则会显示错误信息：

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

输入正确密码之后，则会以root身份登陆，就又可以看到"mysql"数据库了

![](http://img.blog.csdn.net/20131021205734625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)