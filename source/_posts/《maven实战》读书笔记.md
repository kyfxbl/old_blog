title: 《maven实战》读书笔记
date: 2013-09-24 11:07
categories: java
---
这本书详细介绍了maven的基本概念和最佳实践。本文记录一些要点
<!--more-->

# 安装与配置

在windows环境下，安装完maven之后，需要配置3个环境变量：path、M2_HOME、MAVEN_OPTS 

其中前2个是必须的，MAVEN_OPTS变量配置以后，可以避免一些场景下的内存溢出。一般配置为MAVEN_OPTS:-Xms128m -Xmx512m 

然后将%M2_HOME%/conf/settings.xml，复制到%USER_HOME%/.m2/下 

对于公司内禁止了外网访问的，需要配置代理服务器，在settings.xml里配置

```
<proxies>

    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>

  </proxies>
```

在eclipse中安装了m2e插件之后，需要配置一下。否则的话eclipse使用的maven版本和命令行maven版本不一致，可能会造成构建行为不一致 

![](http://dl.iteye.com/upload/attachment/0072/6850/da9eb134-a97f-39c2-a397-5ffba94ec2ff.png)

# 坐标与依赖

坐标是maven的核心概念之一，另外几个核心概念是依赖、仓库、生命周期、插件 

## 坐标

“坐标”是maven引入项目构建的概念，此前ant是没有这个概念的。maven将jar包、项目的构建成品等，都统一看做是“构件”，而坐标就是构件的唯一标识。通过坐标，maven就能找到任何一个构件，并且管理依赖关系 

坐标由以下元素组成：groupId、artifactId、version、packaging、classifier 

groupId，是当前maven项目隶属的实际项目。而不仅仅是到组织层面，因为一个组织往往会有多个实际项目，如果groupId仅仅指向组织，那就无法标识实际项目的模块了。实际项目和maven项目，是一个“一对多”的关系。比如说，我们接下来要做的任务管理重构项目，准备基于maven构建，那这个项目的groupId就应该是com.huawei.wfm.task 

artifactId，即当前的maven项目，可以视为实际项目的一个模块，推荐的做法，是用实际项目名称，作为artifact的前缀。比如说，接下来这个任务子系统，有一个模块是流程引擎，那么这个maven项目的aftifactId就应该是task-process 

version，项目的版本号 

packaging，是项目的打包方式，常见的有jar、war等 

classifier，是用来帮助定义构建输出的一些附属构建，如javadoc、source等，需要依赖插件，不能直接定义 

## 依赖

首先需要知道，maven构建项目的时候，会有3套classpath。编译项目主代码的classpath、编译和执行测试代码的classpath、实际运行时的classpath 

依赖范围，和上面说的第一点密切相关。maven有以下几种依赖范围，包括compile、test、provided、runtime、system、import 

如果在pom中没有特别声明scope的话，默认是compile的依赖范围 

依赖传递，比如项目依赖了spring-framework，而spring-framework又是依赖common-logging的，那么项目也就依赖common-logging。要注意的是，依赖传递的概念，需要和上面说的依赖范围一起理解 

另一个问题是，在实践中，有时候需要排除传递性依赖。比如说项目A依赖项目B，而项目B依赖项目C的1.0版本。但是项目A希望依赖项目C的2.0版本，这个时候就需要在项目A的pom中显式的声明

```
<dependencies>
    <dependency>
        <artifactId>project-b</artifactId>
        <exclusions>
            <exclusion>
                <artifactId>project-c</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <artifactId>project-c</artifactId>
        <version>2.0</version>
    </dependency>
</dependencies>
```

依赖调解有2个原则：第一个是路径最近者优先，第二个是第一声明者优先

# 仓库

maven的仓库只有2类，第一种是本地仓库，默认在%USER_HOME%/.m2/repository目录下；第二种是远程仓库，默认的是maven提供的中央仓库，另外还有很多中央仓库的镜像仓库，以及第三方仓库。一般来说，项目组会在自己的maven服务器上建私服 

## 私服

私服的一个重要作用，是代替中央仓库来提供构件下载。maven项目需要在pom文件中设置私服的位置

```
<project>
    <repositories>
        <repository>
            <id></id>
            <name></name>
            <url></url>
            <releases><enable></enable></releases>
            <snapshots><enabled></enabled></snapshots>
            <layout>default</layout>
        </repository>
    </repositories>
</project>
```

如果私服需要用户名和密码校验的话，是在settings.xml里进行配置 

私服的另一个重要作用，是把项目构建之后得到的成品，部署到私服上，这样才能提供给别的项目依赖，这个也是在pom中设置的

```
<project>
    <distributionManagement>
        <repository>
            <id />
            <name />
            <url />
        </repository>
        <snapshotRepository>
            <id />
            <name />
            <url />
        </snapshotRepository>
    </distributionManagement>
</project>
```

## snapshot

maven的版本管理中一个很重要的概念就是SNAPSHOT，如果没有这个机制的话，那么如果项目A依赖项目B，而项目B还处于开发之中，那么双方都要一直修改版本号，很麻烦，而且版本号变更后的知会也是一个问题 

有了snapshot机制，则maven会自动检测，开发人员可以从中解脱出来 

在settings.xml中，还可以设置镜像
```
<settings>
    <mirrors>
        <mirror>
            <id />
            <name />
            <url />
            <mirrorOf></mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

## 仓库搜索服务

以下几个网址，可以提供仓库搜索服务。输入项目的名称之后，可以找到项目构件对应的坐标 

[http://repository.sonatype.org](http://repository.sonatype.org)
[http://www.jarvana.com/jarvana](http://www.jarvana.com/jarvana)
[http://www.mvnbrowser.com](http://www.mvnbrowser.com) 
[http://mvnrepository.com](http://mvnrepository.com)

# 聚合和继承

聚合和继承也是maven中重要的概念

## 聚合

```
<project>
    <groupId />
    <artifactId />
    <version />
    <packaging>pom</packaging>
    <modules>
        <module>abc</module>
        <module>def</module>
    </modules>
</project>
```

以上的配置和一般的POM有2个区别，一个是packaging要声明为pom，第二个是多了<modules>的配置 

这里的abc和def都是子目录的目录名。也就是说，abc和def都是这个聚合项目的子目录，如果要平级的话，这里需要改成../abc和../def 

## 继承 

以下列表中的pom元素是可以被继承的： 

groupId，version，description，distributionManagement，properties，dependencies，dependencyManagement，repositories，build

不过一般不会直接继承dependencies，因为未来增加的子项目，是否一定就需要依赖parent里声明的依赖，是很难说的。所以一般在parent pom里，只会配置dependencyManagement，以限定依赖的版本号。实际的依赖还是在具体子项目里声明的，但是对版本号进行了控制 

## 聚合与继承的关系 

聚合主要是为了多个maven工程协同构建项目，而继承主要是为了消除重复配置 

对于聚合模块来说，它知道有哪些被聚合的模块，被聚合的模块并不知道聚合模块的存在 

对于继承关系的parent pom来说，它并不知道哪些子模块继承自它，而子模块则必须知道父模块是什么 

在实际项目中，往往会把聚合POM和父POM合一，这样也是有道理的，主要是为了方便起见。至于目录结构，可以是子目录，也可以是平行关系 

## 裁剪Reactor 

当构建一组存在聚合和继承关系的maven项目时，就存在一个Reactor的关系，也就是构建的顺序 

默认情况下，构建顺序会形成一个有向非循环图 

但是可以通过一些参数，来对构建顺序进行裁剪

```
mvn -pl aa,bb 仅构建aa和bb模块 

mvn -pl aa -am 构建aa，及aa的依赖模块 

mvn -pl aa -amd 构建aa，及依赖aa的模块 

mvn -pl aa -amd bb 首先根据aa，及依赖aa的模块，计算出构建顺序之后，从bb模块开始构建（bb模块之前的模块不会被构建）
```