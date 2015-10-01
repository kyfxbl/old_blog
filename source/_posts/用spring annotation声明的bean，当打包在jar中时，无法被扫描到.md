title: 用spring annotation声明的bean，当打包在jar中时，无法被扫描到
date: 2013-09-24 11:11
categories: java 
---
通过注解声明的bean，打包在jar中时，无法被扫描到
<!--more-->

我们项目是由N个工程组成的，外围工程是web工程，内部的工程打包成jar，放入外围工程的WEB-INF/lib 

内部的工程用到了spring的注解，例如@Service、@Controller等，在打成jar包之前，是可以扫描到的，但是打成jar包之后，就扫描不到了，报NoSuchBeanException 

在网上搜索了一下，发现了一个办法，就是在用eclipse export jar的时候，勾选add directory entries 

![](http://dl.iteye.com/upload/attachment/0073/6176/60c8474d-398e-3911-a805-0d4671aec3c3.png)

这样打出来的jar包，可以解决这个问题，在外围也可以扫描到jar包内用注解声明的bean。如果没有勾上add directory entries，就不行了 

用jar命令，比较了一下两种方法打出的jar包的区别，如图： 

![](http://dl.iteye.com/upload/attachment/0073/6180/7e6dae64-2022-37cc-bc30-99c7ae291eca.png)

![](http://dl.iteye.com/upload/attachment/0073/6182/ae3f380c-812f-3d7a-a955-5119b818e868.png)

可以看到，勾选了add directory entries之后打出的jar包，多了路径的信息，可能这就是区别 

不过现在问题是，我们不可能都用手工export jar的方式来一个个导出jar包，不知道在maven中，要配置插件的什么参数，可以达到同样的效果