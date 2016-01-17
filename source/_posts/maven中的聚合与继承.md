title: maven中的聚合与继承
date: 2013-09-24 11:12
categories: java 
---
maven中的聚合和继承是2个不同的概念。但在实际应用中，将module和parent放在一个pom里，是一种常见的做法
<!--more-->
 
# 聚合 

聚合的作用是把子项目的构建过程串到一起：

```
<modules>
		<module>../project-moduleA</module>
		<module>../project-moduleB</module>
	</modules>
```

或者

```
<modules>
		<module>project-moduleA</module>
		<module>project-moduleB</module>
	</modules>
```

前者是对应平行结构的，后者是对应树形结构的 

在平行结构的情况下 

![](http://dl.iteye.com/upload/attachment/0074/3933/9ffced8e-687b-32a0-93c4-0020d691843f.png)

pom所在的当前目录是D:\\example\\project-aggregator，所以需要配置为../project-moduleA，才能找到D:\\example\\project-moduleA 

在树形结构的情况下 

![](http://dl.iteye.com/upload/attachment/0074/3936/fb64ec64-7eb1-31bd-a12d-1171d0d81f26.png)

pom所在的当前目录是D:\\example\\project-aggregator，所以可以直接配置为project-moduleA，即可找到D:\\example\\project-aggregator\project-moduleA 

聚合模块清楚地知道将要聚合哪些子模块，但是子模块对于自己被聚合毫不知情 

# 继承 

与聚合不同，继承的目的是为了在父模块中进行一些公共配置，以简化子模块的POM文件

```
<parent>
		<groupId>net.kyfxbl.modules.project</groupId>
		<artifactId>project-aggregator</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<relativePath>../project-aggregator</relativePath>
	</parent>
```

或者

```
<parent>
		<groupId>net.kyfxbl.modules.project</groupId>
		<artifactId>project-aggregator</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<relativePath>../</relativePath>
	</parent>
```

前者是对应平行结构的，后者是对应树形结构的 在平行结构的情况下 

![](http://dl.iteye.com/upload/attachment/0074/3933/9ffced8e-687b-32a0-93c4-0020d691843f.png)

pom所在的当前目录是D:\\example\\project-moduleA，所以需要配置为../project-aggregator，才能找到D:\\example\\project-aggregator\\pom.xml 

在树形结构的情况下 

![](http://dl.iteye.com/upload/attachment/0074/3936/fb64ec64-7eb1-31bd-a12d-1171d0d81f26.png)

pom所在的当前目录是D:\\example\\project-aggregator\\project-moduleA，所以可以直接配置为../，即可找到D:\\example\\project-aggregator\\pom.xml 

子模块知道自己继承自哪个父模块，但是父模块并不知道自己将会被哪些子模块继承 

