title: 用nexus搭建maven私服
date: 2013-09-24 11:12
categories: java 
---
首先介绍一下背景，前公司访问外网有限制，项目组大部分人员不能访问maven的central repository，因此在局域网里找一台有外网权限的机器，搭建nexus私服，然后开发人员连到这台私服上。本文总结一下搭建过程
<!--more-->

# 环境

nexus-2.1.1、maven-3.0.4、jdk-1.6.0_32 

# 配置

nexus的下载和安装都很简单，网上也有很多介绍，本文就不重复了。主要介绍一下安装之后的配置

首先用admin用户登陆nexus，nexus的配置需要用admin角色完成，默认的密码是admin123，进入nexus首页之后，点击右上角，进行登录 

![](http://dl.iteye.com/upload/attachment/0073/6753/a7412b77-326c-3d3e-90a4-0c28b2ef99f2.png)

然后就可以在左边的菜单中进行配置了 

![](http://dl.iteye.com/upload/attachment/0073/6758/c1d65ab5-43a5-38e3-8252-ef6bba6a0210.png)

# 为nexus配置代理服务器 

由于这台机器需要通过代理才能访问外网，所以首先要配置代理服务器，在Administration - Server中进行配置 

![](http://dl.iteye.com/upload/attachment/0073/6763/2b6112d4-0dc3-3a7c-b872-27321f4122dd.png)

配置之后，nexus才能连上central repository，如果私服所在机器可以直接上外网，则可以省略这一步 

# 配置repository 

在Views/Repositories - Repositories里进行配置 

![](http://dl.iteye.com/upload/attachment/0073/6768/24dff94f-346b-388d-bcdf-efa93989d7ff.png)

nexus里可以配置3种类型的仓库，分别是proxy、hosted、group 

proxy是远程仓库的代理。比如说在nexus中配置了一个central repository的proxy，当用户向这个proxy请求一个artifact，这个proxy就会先在本地查找，如果找不到的话，就会从远程仓库下载，然后返回给用户，相当于起到一个中转的作用 

hosted是宿主仓库，用户可以把自己的一些构件，deploy到hosted中，也可以手工上传构件到hosted里。比如说oracle的驱动程序，ojdbc6.jar，在central repository是获取不到的，就需要手工上传到hosted里 

group是仓库组，在maven里没有这个概念，是nexus特有的。目的是将上述多个仓库聚合，对用户暴露统一的地址，这样用户就不需要在pom中配置多个地址，只要统一配置group的地址就可以了 

nexus装好之后，已经初始化定义了一些repository，我们熟悉之后，就可以自行删除、新增、编辑 

右边那个Repository Path可以点击进去，看到仓库中artifact列表。不过要注意浏览器缓存。我今天就发现，明明构件已经更新了，在浏览器里却看不到，还以为是BUG，其实是被浏览器缓存了 

# 配置Central Repository的proxy 

最关键的一个配置，可能就是Central Repository的proxy配置，因为大部分的构件，都是要通过这个proxy得到的 

![](http://dl.iteye.com/upload/attachment/0073/6789/b0bdbedd-577f-3015-83cd-9700e299d4ab.png)

在安装完nexus之后，这个proxy是预置的，需要做的就是把Download Remote Indexes改为true，这样nexus才会从central repository下载索引，才能在nexus中使用artifact search的功能 

网络上有一些其他公开的maven仓库，可以用同样的办法，在nexus中设置proxy，但是并不是所有maven仓库，都提供了nexus index，这种情况下，就无法建立索引了 

# 配置hosted repository 

一般会配置3个hosted repository，分别是3rd party、Snapshots、Releases，分别用来保存第三方jar（典型的比如ojdbc6.jar），项目组内部的快照、项目组内部的发布版 

![](http://dl.iteye.com/upload/attachment/0073/6800/4aa68576-96e4-3d7c-9d22-0d73d22c4842.png)

这里并没有什么特别的配置，只是Deployment Policy这个选项，一般Snapshots会配置成允许，而Releases和3rd party会设置为禁止 

# 配置group repository 

前面说过，group其实是一个虚拟的仓库，通过对实体仓库（proxy、hosted）进行聚合，对外暴露一个统一的地址 

![](http://dl.iteye.com/upload/attachment/0073/6803/320775f3-9b11-3473-b2ad-777aa0acaa2a.png)

这里要注意的是，放到左边的仓库，才是会被聚合的仓库。我昨天一直搞错了，把仓库都放到右边，结果group什么都没有聚合到，是一个空的仓库。。。 

# 配置用户密码 

在Security - Users中配置，在deployment用户上点击右键，选择Set Password，然后设置一个密码，做这个操作是为了后面提交做准备 

![](http://dl.iteye.com/upload/attachment/0073/6806/3101d978-0d51-3bcc-b981-b1ae1976be5e.png)

# 在用户机器上配置settings.xml 

经过前面的7个步骤，nexus就配置好了，接下来需要在每个开发人员的开发机器上进行配置了 

配置文件在%USER_HOME%/.m2/settings.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>

  	<server>
  		<id>nexus-snapshots</id>
  		<username>deployment</username>
  		<password>deployment</password>
  	</server>

  </servers>

  <mirrors>

  	<mirror>
  		<id>nexus</id>
  		<name>internal nexus repository</name>
  		<url>http://10.78.68.122:9090/nexus-2.1.1/content/groups/public/</url>
  		<mirrorOf>central</mirrorOf>
  	</mirror>

  </mirrors>

</settings>
```

这里只配置了2个元素<mirrors>和<servers> 

首先这里配置了一个id为nexus的镜像仓库，地址是前面配置的public group的URL，然后镜像目标是central maven里的超级pom，里面配置了这样一段：
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

  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
```

因此，当本地的maven项目，找不到需要的构件（包括jar包和插件）的时候，默认会到central里获取 所以我们刚刚配置的镜像仓库，id也是central，这样本地maven项目对central repository的请求，就会转到镜像仓库上，也就是我们设置的nexus私服上 

由于我们在项目的pom里，不会再配置其他的<repositories>和<pluginRepositories>元素，所以只要配置一个central的mirror，就足以阻止所有的外网访问。如果pom中还配置了其他的外网仓库，比如jboss repository等，可以把<mirrorOf>改为* 

至于<servers>元素，是因为我们把项目内部的构件上传到nexus的仓库中时，nexus会进行权限控制，所以这里需要设置权限相关的信息。注意这里的<id>nexus-snapshots</id>，和后面maven工程里的pom设置是一致的 

由于我们这里已经屏蔽了对外网仓库的请求，所以就不需要配置代理服务器了，如果需要配置代理服务器，可以用<proxies>元素 

# 配置maven项目的pom文件 

下面是简化后的pom文件：
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>
	<groupId>com.huawei.inoc.wfm.task</groupId>
	<artifactId>task-sla</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>task-sla</name>

	<!-- 配置部署的远程仓库 -->
	<distributionManagement>
		<snapshotRepository>
			<id>nexus-snapshots</id>
			<name>nexus distribution snapshot repository</name>
			<url>http://10.78.68.122:9090/nexus-2.1.1/content/repositories/snapshots/</url>
		</snapshotRepository>
	</distributionManagement>

</project>
```
这里配置了<distributionManagement>元素，其中的<id>nexus-snapshots</id>，与前面说的settings.xml中的<servers>元素中的配置必须一致 

配置这个的目的，是当执行maven deploy时，才知道要将生成的构件部署到哪个远程仓库上，注意这里的URL填的就不是public group的地址：10.78.68.122:9090/nexus-2.1.1/content/groups/public/，而是snapshots的地址：10.78.68.122:9090/nexus-2.1.1/content/repositories/snapshots/ 

但是在nexus中，snapshots也是聚合到public group里的，所以开发人员A提交到snapshots的构件，开发人员B也可以从public group里获取到 

# eclipse中的设置 

经过前面的配置，已经可以通过命令行进行maven操作了。不过实际开发中，一般都是使用eclipse的m2e插件，所以还需要对eclipse进行一些额外的配置 

在Preferences - Maven - User Settings中，点击Update Settings，加载刚才我们对settings.xml的更改 

![](http://dl.iteye.com/upload/attachment/0073/6834/4cd1385e-95a2-30e3-bdd5-b65eb3a8ac74.png)

然后在Maven Repositories视图里，可以看到仓库的情况 

![](http://dl.iteye.com/upload/attachment/0073/6837/f0dea1d2-fc69-35e8-a84a-5dadbc713beb.png)

可以看到，从超级pom继承来的central被置灰了，不可用，后面的mirrored by nexus表示对该仓库的所有请求，都会转到镜像nexus中 

# nexus的目录结构 

nexus会安装在%USER_HOME%/sonatype-work/nexus下，有以下目录 

![](http://dl.iteye.com/upload/attachment/0073/6841/47371133-2411-3dea-95da-d9f1883e854e.png)

其中的storage目录，就是构件实际存放的地址了