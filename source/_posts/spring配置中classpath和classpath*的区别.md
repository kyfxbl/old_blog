title: spring配置中classpath和classpath*的区别
date: 2013-09-24 11:11
categories: java 
---
在spring中配置classpath和classpath\*的区别
<!--more-->

在spring配置文件里，可以用classpath:前缀，来从classpath中加载资源。比如在src下有一个jdbc.properties的文件，可以用如下方法加载：

```
<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:jdbc.properties</value>
			</list>
		</property>
	</bean>
```

对于打包在jar包中的资源，也可以用同样的方式：

```
<import resource="classpath:META-INF/cxf/cxf.xml" />
```

另外一种很像的方式，是使用classpath\*:前缀，比如

```
<property name="mappingLocations">
			<list>
				<value>classpath*:/hibernate/*.hbm.xml</value>
			</list>
		</property>
```

classpath:与classpath\*:的区别在于，前者只会从第一个classpath中加载，而后者会从所有的classpath中加载 

如果要加载的资源，不在当前ClassLoader的路径里，那么用classpath:前缀是找不到的，这种情况下就需要使用classpath\*:前缀 

另一种情况下，在多个classpath中存在同名资源，都需要加载，那么用classpath:只会加载第一个，这种情况下也需要用classpath\*:前缀 

可想而知，用classpath\*:需要遍历所有的classpath，所以加载速度是很慢的，因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用classpath\*