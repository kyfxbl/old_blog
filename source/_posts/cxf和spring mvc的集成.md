title: cxf和spring mvc的集成
date: 2013-09-24 11:00
categories: java
---
spring mvc通过DispatcherServlet来加载Spring配置文件，因此不需要在web.xml中配置ContextLoaderListener。但是CXF却需要通过ContextLoaderListener来加载Spring。这样就产生了一个矛盾，如果不配置ContextLoaderListener，cxf就无法正常使用。但如果配置ContextLoaderListener，又会造成Spring的重复加载（DispatcherServlet一次，ContextLoaderListener一次） 
<!--more-->

在网上查了一下资料，只看到一个国外的程序员提出不配置ContextLoaderListener，通过写一个CXFController，来替代默认的CXFServlet。但是这种HACK的方式总是不太好 

所以我采用一种折中的方式，来解决Spring MVC和cxf并存的问题

首先是web.xml的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="MyFramework" version="3.0">

	<display-name>MyFramework</display-name>

	<context-param>
    	<param-name>contextConfigLocation</param-name>
        <param-value> 
        	WEB-INF/beans.xml, 
            WEB-INF/cxf.xml
        </param-value> 
   	</context-param>

    <listener>  
        <listener-class>  
            org.springframework.web.context.ContextLoaderListener  
        </listener-class>  
    </listener>  

	<servlet>
		<servlet-name>framework</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>  
        	<param-name>contextConfigLocation</param-name>  
        	<param-value>WEB-INF/spring-mvc.xml</param-value>  
   		</init-param>  
   		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>framework</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<servlet>  
        <servlet-name>CXFServlet</servlet-name>  
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>  
        <load-on-startup>2</load-on-startup>  
    </servlet>  

    <servlet-mapping>  
        <servlet-name>CXFServlet</servlet-name>  
        <url-pattern>/webservice/*</url-pattern>  
    </servlet-mapping>  

</web-app>
```

在ContextLoaderListener里面，加载cxf和spring的公共配置信息。然后在DispatcherServlet中加载spring mvc所需的信息。利用Spring配置文件拆分的方式，既实现spring mvc和cxf的共存，又避免spring配置信息的重复加载 

然后是公共的配置信息，beans.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
       						http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
       						http://www.springframework.org/schema/context 
      			 			http://www.springframework.org/schema/context/spring-context-3.1.xsd
      			 			http://www.springframework.org/schema/tx
      		 				http://www.springframework.org/schema/tx/spring-tx-3.1.xsd">

	<context:component-scan base-package="com.huawei.framework" />

	<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:jdbc.properties</value>
			</list>
		</property>
	</bean>

	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.driver}" />
		<property name="jdbcUrl" value="${jdbc.url}" />
		<property name="user" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="minPoolSize" value="10" />
		<property name="maxPoolSize" value="50" />
	</bean>

	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mappingLocations"
			value="classpath:/com/huawei/framework/model/**/*.hbm.xml" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.OracleDialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
				<prop key="hibernate.jdbc.fetch_size">50</prop>
				<prop key="hibernate.jdbc.batch_size">25</prop>
				<prop key="hibernate.temp.use_jdbc_metadata_defaults">false</prop>
			</props>
		</property>
	</bean>

	<bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
		<property name="sessionFactory" ref="sessionFactory" />
		<property name="fetchSize" value="10" />
	</bean>

	<bean id="transactionManager"
		class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<tx:annotation-driven transaction-manager="transactionManager" />

</beans>
```

然后是cxf所需的信息，cxf.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
       						http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
        					http://cxf.apache.org/jaxws 
        					http://cxf.apache.org/schemas/jaxws.xsd">

	<import resource="classpath:META-INF/cxf/cxf.xml" />
	<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />

	<jaxws:endpoint id="helloWorld"
		implementorClass="com.huawei.framework.webservice.HelloWorldImpl"
		address="/HelloWorld" />

</beans>
```

最后是spring mvc所需的信息，spring-mvc.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans
       						http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
       						http://www.springframework.org/schema/context 
      			 			http://www.springframework.org/schema/context/spring-context-3.1.xsd
      			 			http://www.springframework.org/schema/mvc
        					http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd">

	<context:component-scan base-package="com.huawei.framework" />

	<mvc:annotation-driven />

	<mvc:default-servlet-handler />

	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver"
		p:prefix="/WEB-INF/view/" p:suffix=".jsp"
		p:viewClass="org.springframework.web.servlet.view.JstlView" />

</beans>
```

这顺便还带来一个附带的好处，即进行单元测试的时候，可以仅加载必须的类，比如做DAO组件的单元测试，就不需要加载spring-mvc.xml和cxf.xml了

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:WebContent/WEB-INF/beans.xml" })
public class UserDaoHibernateTest {

	@Autowired
	private UserDao dao;

	private static String ID;

	@Test
	public void testInsert() {
		User user = new User();
		user.setName("fme001");
		user.setAge(23);
		dao.insert(user);
	}

	@Test
	public void testQueryByName() {
		String name = "fme001";
		User user = dao.queryByName(name);
		assertEquals(23, user.getAge());
		ID = user.getId();
	}

	@Test
	public void testQueryByAge() {
		int age = 23;
		List<User> result = dao.queryByAge(age);
		assertEquals(1, result.size());
	}

	@Test
	public void testQueryAll() {
		List<User> result = dao.queryAll();
		assertEquals(1, result.size());
	}

	@Test
	public void testQuery() {
		User user = dao.query(ID);
		assertEquals(23, user.getAge());
		assertEquals("fme001", user.getName());
	}

	@Test
	public void testUpdate() {
		User user = dao.query(ID);
		user.setAge(24);
		dao.update(user);
		user = dao.query(ID);
		assertEquals(24, user.getAge());
	}

	@Test
	public void testDelete() {
		User user = dao.query(ID);
		dao.delete(user);
	}

}
```

可以看到，集成cxf和spring mvc确实不算很方便，需要绕一绕。上述方法只是一个折中的方案，希望新版本的cxf可以提供一个CXFController，这样的话就可以统一用DispatcherServlet来加载Spring，不配置ContextLoaderListener。这样的话web.xml可以更简洁