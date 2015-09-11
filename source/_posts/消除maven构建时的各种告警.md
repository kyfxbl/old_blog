title: 消除maven构建时的各种告警
date: 2013-09-24 11:12
categories: java 
---
消除maven构建时的各种告警
<!--more-->

# 字符集告警

如果没有指定字符集，maven会在构建时显示warning

```
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform dependent! 
```

解决方法：
```
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-resources-plugin</artifactId>
			<configuration>
				<encoding>UTF-8</encoding>
			</configuration>
		</plugin>

		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<configuration>
				<encoding>UTF-8</encoding>
			</configuration>
		</plugin>
	</plugins>
</build>
```

# war文件合并告警

如果忽略了web.xml文件，也会提示warning

```
[WARNING] Warning: selected war files include a WEB-INF/web.xml which will be ignored (webxml attribute is missing from war task, or ignoreWebxml attribute is specified as 'true')
```
 
解决方法：
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-war-plugin</artifactId>
	<version>2.1.1</version>
	<configuration>
		<packagingExcludes>WEB-INF/web.xml</packagingExcludes>  
	</configuration>
</plugin>
```

关于告警2，附带一提，很多时候会有将多个war合并成一个war的场景，这是用的是overlays选项，但是只有一个maven项目的web.xml会最终生效 所以其他的maven项目中可以不放web.xml，但是对于packaging类型是war的maven工程，默认是必须要有web.xml的，这时候可以使用以下配置：
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-war-plugin</artifactId>
	<configuration>
		<failOnMissingWebXml>false</failOnMissingWebXml>
	</configuration>
</plugin>
```