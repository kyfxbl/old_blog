title: 用overlays合并多个war
date: 2013-09-24 11:12
categories: java 
---
在一个大项目中拆分maven工程时，很有可能会把js、css、jsp等文件放在不同的工程里（根据业务模块划分）。因为如果都集中在一个maven webapp里，那么这个maven webapp会太大，而且在业务上也比较分散 

但是这些持有js、css、jsp的maven工程，如果packaging设置为jar是不合适的，因为外围要读取内部的这些文件就会很困难。在这种场景下，一个很自然的想法就是打成war包，然后用某种方式将多个war包归并起来，得到最终的war包 

这就是overlays发挥作用的地方 
<!--more-->

以下举一个例子： 这里有2个web工程，一个是task-sla-web，一个是task-web-dist，packaging类型都是war，目录结构如下： 

![](http://dl.iteye.com/upload/attachment/0073/7975/a881a33b-ef23-3b9e-8254-c26206dc004d.png)
![](http://dl.iteye.com/upload/attachment/0073/7977/b1923208-cd9b-3d6a-a904-db027085f0a0.png)

下面是task-sla-web的pom文件：

```
<modelVersion>4.0.0</modelVersion>
	<groupId>com.huawei.inoc.wfm.task</groupId>
	<artifactId>task-sla-web</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>task-sla-web</name>
```

该工程就是打成一个war包，但是这个war是无法运行的，而是稍后用来合并的。（其中放了一个空的web.xml，因为maven-war-plugin的package goal有强制要求） 

下面是task-web-dist的pom文件：

```
<modelVersion>4.0.0</modelVersion>
	<groupId>com.huawei.inoc.wfm.task</groupId>
	<artifactId>task-web-dist</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>task-web-dist</name>
```

```
<!-- 合并多个war -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.1.1</version>
				<configuration>
					<overlays>
						<overlay>
							<groupId>com.huawei.inoc.wfm.task</groupId>
							<artifactId>task-sla-web</artifactId>
						</overlay>
					</overlays>
				</configuration>
			</plugin>

```

```
<!-- 配置依赖 -->
	<dependencies>
		<dependency>
			<groupId>com.huawei.inoc.wfm.task</groupId>
			<artifactId>task-sla-web</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<type>war</type>
		</dependency>
	</dependencies>
```

以上片段主要要注意几点： 

1、task-web-dist自身的packaging类型也是war 

2、在<overlay>中配置要归并的webapp的groupId和artifactId，注意的是，该pom所在的webapp工程是主工程，会覆盖掉所有待归并工程的同名文件，包括web.xml 

3、要归并的webapp，必须声明为依赖 

归并后的最终war包如下： 

![](http://dl.iteye.com/upload/attachment/0073/7984/64421942-d5e8-3612-a447-e6aef58dd1ea.png)

其中的文件和.class都是由2个war包归并得到的，task-web-dist是主war包，如果多个war包中存在重名文件，则会被task-web-dist的文件覆盖，比如web.xml