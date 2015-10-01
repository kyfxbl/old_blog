title: maven的一些知识整理
date: 2013-09-24 11:11
categories: java 
---
总结一些maven的细节知识
<!--more-->

# 构件的路径 

坐标是构件的逻辑表示方式，而物理表示方式则是文件。构件所在的文件路径，是由GAV决定的 

比如log4j:log4j:1.2.15，所在的仓库路径是：%repository_path%/log4j/log4j/1.2.15/log4j-1.2.15.jar 

其中%repository_path%是跟仓库的实现有关，构件自身的命名规则是：groupId/artifactId/version/artifactId-version.packaging 

# 超级POM的位置 

超级POM在这个路径：%M2_HOME%/lib/maven-model-builder-3.0.jar，解压之后的org/apache/maven/model/pom-4.0.0.xml 

# 远程仓库的认证 

如果远程仓库需要认证信息的话，是在settings.xml文件里配置的，而不是在maven项目的pom.xml里配置，这主要是出于安全性的考虑 

# 跟仓库相关的几个POM配置 

以下配置都是在pom.xml设置的 

获取依赖的远程仓库：
```
<repositories>
    <repository>
        <id />
        <name />
        <url />
    </repository>
</repositories>
```

获取插件的远程仓库：
```
<pluginRepositories>
    <pluginRepository>
        <id />
        <name />
        <url />
    </pluginRepository>
</pluginRepositories>
```

可以看到，插件仓库和依赖仓库的配置是很像的，只是元素名有点不同 

构件部署的远程仓库：
```
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
```

# 镜像仓库 

镜像仓库是在settings.xml里配置的，常见的用法是跟私服结合起来。将一个私服的地址，配置为所有远程仓库的镜像 

# 生命周期阶段，与插件目标的关系 

是多对多的关系。另外插件目标，不一定要绑定到生命周期的某个阶段上才能执行，也是可以独立运行的，比如mvn dependence:tree 

# 插件配置的方法 

第一种方法，是在命令行用-D参数配置，这种方法仅对当次操作有效。比如maven-surefire-plugin插件的test目标提供了skip参数，对应的表达式是${maven.test.skip} 

那么可以在命令行输入，mvn install -Dmaven.test.skip = true 

要注意的是，并不是所有的插件目标参数都提供表达式的，对于这种没有提供表达式的参数，就只能在pom文件里配置了 

第二种方法，是在pom文件中进行插件全局配置
```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>   
    </plugins>
</build>
```

第三种方法，是在pom文件中针对特定的插件任务来配置
```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <executions>
                <execution>
                    <id />
                    <phase />
                    <goals />
                    <configuration>
                        <skip>true</skip>
                    </configuration>
                </execution>
            </executions>
        </plugin>   
    </plugins>
</build>
```

这个方法与第二个方法的区别在于，第二种方法配置的参数是全局的，而第三种方法只是针对某个任务的 

# pom.xml和settings.xml的一些元素 

pom.xml：
```
<project>
    <properties />
    <dependencies />
    <build />
    <repositories />
    <pluginRepositories />
    <distributionManagement />
</project>
```

settings.xml：
```
<settings>
    <localRepository />
    <servers />
    <mirrors />
    <proxies />
    <profiles />
    <activeProfiles />
</settings>
```