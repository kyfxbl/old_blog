title: 解决maven工程无法获取远程仓库上的构件
date: 2013-09-24 11:12
categories: java 
---
一次maven错误的解决过程
<!--more-->

有3个子工程：test-aggregator test-parent test-son 

其中test-aggregator是聚合工程，用于把所有子工程的构建过程串起来，并充当parent工程，定义公共的配置和变量。test-son依赖于test-parent 

现象：如果将所有工程都下载到本地，然后从task-aggregator里执行maven package，整体构建可以成功 

但是如果仅下载test-son，然后从test-son里执行maven package，则构建失败，报错：

[ERROR] Failed to execute goal on project test-son: 

Could not resolve dependencies for project net.kyfxbl.test:test-son:jar:0.0.1-SNAPSHOT: 
Could not find artifact net.kyfxbl.test:test-parent:jar:0.0.1-SNAPSHOT

实际上这个时候在远程仓库里已经有test-parent的构件了 
![](http://dl.iteye.com/upload/attachment/0074/3117/245b1020-5795-30c8-8fed-70b363e5fdc5.png)

当maven执行构建时，如果引入了依赖，首先会到本地仓库找，如果本地仓库找不到，则会去远程仓库找。如果远程仓库也没有找到，则构建报错 

这里由于只下载了test-son，并且此前没有在本地构建过test-parent，所以本地仓库里没有test-parent的构件。但是远程仓库里明明有，却获取不到，就比较奇怪了 

原因如下： 

首先我们的settings.xml里配置了一个镜像，将对中央仓库的请求，转发到nexus私服上
```
<mirrors>
  	<mirror>
  		<id>nexus</id>
  		<name>internal nexus repository</name>
  		<url>http://10.78.68.122:9090/nexus-2.1.1/content/groups/public/</url>
  		<mirrorOf>central</mirrorOf>
  	</mirror>
</mirrors>
```
然后在项目的pom里，没有配置任何远程仓库，所以就会默认使用pom4.0里的中央仓库作为远程仓库 

查看pom4.0里的配置：
```
<repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
</repositories>
```
这里关键是把<snapshots>设置为false，不允许下载SNAPSHOT版的构件，所以虽然我们已经把test-parent这个构建deploy到了私服里，但是无法被test-son下载到，因此test-son构建报错 

解决方法有2个： 

一个是在自己项目的pom中额外声明私服地址，把snapshots设置为true；但是这个私服地址与settings.xml里配置的镜像重复了 

另一个办法是在自己项目的pom中覆盖central的配置，把snapshots设置为true
```
<!-- 覆盖central的设置，允许下载snapshot的构件 -->
	<repositories>
		<repository>
			<id>central</id>
			<name>Maven Central Repository</name>
			<url>http://repo1.maven.org/maven2</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
```
最后综合考虑，还是选择了后一种方案