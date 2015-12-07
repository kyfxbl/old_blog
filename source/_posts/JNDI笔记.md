title: JNDI笔记
date: 2013-09-24 10:31
categories: java 
---
JNDI服务是web容器提供的服务。web应用可以通过JNDI服务从容器中得到各种组件（包括但不限于数据源），实现各组件的解耦
<!--more-->

以下举一个例子。 在tomcat的conf/server.xml中配置：

```
<Context path="/jndi"> 

    <Resource name="bean/MyBeanFactory" auth="Container" 
            type="com.huawei.jndi.bean.MyBean" 
            factory="org.apache.naming.factory.BeanFactory" 
            bar="23"/> 

</Context> 
```

上面就在tomcat中声明了一个组件，接下来在代码中可以获取这个组件：

```
        try
        {
            Context initContext = new InitialContext();
            Context envCtx = (Context) initContext.lookup("java:comp/env");
            MyBean bean = (MyBean) envCtx.lookup("bean/MyBeanFactory");
            System.out.println(bean.getBar());
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
```

在tomcat中配置jndi组件，然后在代码中获取已配好的组件。各web容器的JNDI实现类不同，比如在JBOSS中，JNDI提供类是org.jnp.interfaces.NamingContextFactory

这样看来，JNDI的作用和spring的依赖注入倒是差不多。但是<span style="color: red;">通过JNDI，可以实现跨应用，甚至跨域获取组件</span>。在服务器A上配置的组件，在另一台服务器B上，可以通过JNDI获取到

spring也提供了对jndi的封装，使用起来更加方便，以下是一个例子。

```
<!-- JNDI模板 -->
<bean id="jndiTemplate" class="org.springframework.jndi.JndiTemplate">
	<property name="environment">
		<props>
			<prop key="java.naming.factory.initial">org.jnp.interfaces.NamingContextFactory</prop>
			<prop key="java.naming.provider.url">10.137.96.212:18199</prop>
			<prop key="java.naming.factory.url.pkgs">org.jnp.interfaces:org.jboss.naming</prop>
		</props>
	</property>
</bean>

<!-- 创建连接工厂 -->
<bean id="jmsConnectionFactory" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiTemplate" ref="jndiTemplate" />
	<property name="jndiName" value="TopicConnectionFactory" />
</bean>
```

先声明JndiTemplate，配置好目标地址、JNDI服务提供类。然后通过JndiObjectFactoryBean，就可以很方便地获取JNDI组件，并进行类型转换