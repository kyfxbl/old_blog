title: 集成spring3、hibernate4、junit
date: 2013-09-24 11:07
categories: java 
---
多年以前一次搭积木的过程记录
<!--more-->

论坛上有另外一篇更全面的帖子，开涛写的：[最新SpringMVC + spring3.1.1 + hibernate4.1.0 集成及常见问题总结](http://www.iteye.com/topic/1120924)

# 简述

本文的环境是： spring-framework-3.1.0 hibernate-4.1.6 junit-4.10 

这里大部分是参考我以前熟悉的配置方法，只是把hibernate3升级到hibernate4，其实差不了很多，只要注意几个要点： 

1、以前集成hibernate3和spring的时候，spring的ORM包里提供了HibernateSupport和HibernateTemplate这两个辅助类，我用的是HibernateTemplate。不过在Hibernate4里，spring不再提供这种辅助类，用的是hibernate4的原生API 

2、集成hibernate4之后，最小事务级别必须是Required，如果是以下的级别，或者没有开启事务的话，无法得到当前的Session

```
sessionFactory.getCurrentSession();
```

执行这行代码，会抛出No Session found for current thread 对于运行时，这个可能不是很大的问题，因为在Service层一般都会开启事务，只要保证级别高于Required就可以了。可是由于在Dao层是不会开启事务的，所以针对Dao层进行单元测试就有困难了。 解决的办法是，或者在Dao层的单元测试类上，开启事务。或者专门准备一个for unit test的配置文件，在Dao层就开启事务。我采用的是前者 

# 目录结构

这里暂时还没有集成struts2、spring-mvc等web框架，也尚未包含js、css、jsp等目录 

![](http://dl.iteye.com/upload/attachment/0072/4211/9522b1a6-a6c8-385d-bf6c-594191344b12.png)

这里除了servlet规范规定的web.xml必须放在WEB-INF下之外，其他的所有配置文件，都放在src根目录下。这样做的好处是，后续所有需要引用配置文件的地方，都可以统一用classpath:前缀找到配置文件。之前试过有的文件放在WEB-INF下，有的放在src根目录下，所以在引用的地方会不太统一，比较麻烦。 当然无论配置文件怎么放，只要恰当使用classpath:和file:前缀，都是能找到的，只是个人选择的问题。另外，由于现在配置文件还比较少，所以直接扔到src根目录下没什么问题，如果配置文件增多了，可以再进行划分 

# web.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns="http://java.sun.com/xml/ns/javaee" 
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" 
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" 
    id="WebApp_ID" version="3.0">

	<display-name>DevelopFramework</display-name>

  	<context-param>
    	<param-name>contextConfigLocation</param-name>
       	<param-value>classpath:beans.xml</param-value> 
   	</context-param>

  	<listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  

	<servlet>  
        <servlet-name>CXFServlet</servlet-name>  
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>  
        <load-on-startup>1</load-on-startup>  
    </servlet>  

    <servlet-mapping>  
        <servlet-name>CXFServlet</servlet-name>  
        <url-pattern>/webservice/*</url-pattern>  
    </servlet-mapping>

</web-app>
```

这里没有什么要特别注意的，只是声明了beans.xml的路径。这里的servlet是配置cxf的，与hibernate没有关系。因为目标是要搭一个完整的开发框架，所以把cxf也事先放上了 

# spring配置

接下来是spring的配置文件beans.xml

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:jaxws="http://cxf.apache.org/jaxws"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
       						http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
       						http://www.springframework.org/schema/context
      			 			http://www.springframework.org/schema/context/spring-context-3.1.xsd
      			 			http://www.springframework.org/schema/tx
      		 				http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
      			 			http://cxf.apache.org/jaxws 
							http://cxf.apache.org/schemas/jaxws.xsd">

    <import resource="classpath:META-INF/cxf/cxf.xml" />

	<context:component-scan base-package="com.huawei.inoc.framework" />

	<bean id="propertyConfigurer"
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>classpath:jdbc.properties</value>
			</list>
		</property>
	</bean>

	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
		<property name="driverClass" value="${driverClass}" />
		<property name="jdbcUrl" value="${jdbcUrl}" />
		<property name="user" value="${user}" />
		<property name="password" value="${password}" />
	</bean>

	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mappingLocations" value="classpath:/com/huawei/inoc/framework/model/**/*.hbm.xml" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
				<prop key="hibernate.jdbc.fetch_size">50</prop>
				<prop key="hibernate.jdbc.batch_size">25</prop>
				<prop key="hibernate.temp.use_jdbc_metadata_defaults">false</prop>
			</props>
		</property>
	</bean>

	<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<tx:annotation-driven transaction-manager="transactionManager" />

	<jaxws:endpoint id="helloWorld" implementor="#helloWorldWebserviceImpl" address="/HelloWorld" />

	<jaxws:client id="client"  
        serviceClass="com.huawei.inoc.dummy.webservice.IDemoSupport"  
        address="http://localhost:8080/Dummy/webservice/getDate" />  

</beans>
```

这里有几点要注意的：

```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
		<property name="driverClass" value="${driverClass}" />
		<property name="jdbcUrl" value="${jdbcUrl}" />
		<property name="user" value="${user}" />
		<property name="password" value="${password}" />
	</bean>
```

这里把jdbc驱动的参数，放到了专门的配置文件里，改动起来会比较方便。另外数据库连接池在实际生产环境可以考虑切换一下，比如听说阿里巴巴出的druid就挺不错，jboss和WAS自带的连接池也是不错的

```
<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mappingLocations" value="classpath:/com/huawei/inoc/framework/model/**/*.hbm.xml" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
				<prop key="hibernate.jdbc.fetch_size">50</prop>
				<prop key="hibernate.jdbc.batch_size">25</prop>
				<prop key="hibernate.temp.use_jdbc_metadata_defaults">false</prop>
			</props>
		</property>
	</bean>
```

这里的sessionFactory改成org.springframework.orm.hibernate4.LocalSessionFactoryBean，如果ORM映射采用的不是配置文件，是用注解的话，以前hibernate3有一个AnnotationSessionFactoryBean，在hibernate4里没看到。这里ORM映射用的是配置文件，其实用注解也差不多 

这一行：
```
<prop key="hibernate.temp.use_jdbc_metadata_defaults">false</prop>
```

可以避免启动容器时报的一个错误： Disabling contextual LOB creation as createClob() method threw error : java.lang.reflect.InvocationTargetException 

这个错误其实是无所谓的，不过还是不要报错好看一点

```
<bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>

	<tx:annotation-driven transaction-manager="transactionManager" />
```

这里是开启事务，用的是注解，比用配置文件简单一点。用配置文件的好处，是事务声明比较集中，不需要在每个Service层接口上单独声明。缺点是Service中的方法，命名规范需要事先约定好，否则事务就不能生效 

用注解的好处，是Service中的方法命名不需要特别规定，缺点是没有做到集中声明，如果在某个Service层的接口忘记声明事务，那么事务就无法生效 

两种方法各有好处，我个人更喜欢用注解 

# DAO层

![](http://dl.iteye.com/upload/attachment/0072/4229/0ab83587-fbbf-30ed-9995-5cffad37d65f.png)

首先有一个通用的DAO接口，然后有一个通用的DAO抽象实现类。每个具体业务DAO接口，继承通用DAO接口，具体业务DAO实现，继承通用DAO抽象实现类

```
public interface IGenericDAO<T> {

	void insert(T t);

	void delete(T t);

	void update(T t);

	T queryById(String id);

	List<T> queryAll();
}
```
因为只是示例，这里的方法不是很多，只包含了基本的增删改查方法

```
public abstract class GenericDAO<T> implements IGenericDAO<T> {

	private Class<T> entityClass;

	public GenericDAO(Class<T> clazz) {
		this.entityClass = clazz;
	}

	@Autowired
	private SessionFactory sessionFactory;

	@Override
	public void insert(T t) {
		sessionFactory.getCurrentSession().save(t);
	}

	@Override
	public void delete(T t) {
		sessionFactory.getCurrentSession().delete(t);
	}

	@Override
	public void update(T t) {
		sessionFactory.getCurrentSession().update(t);
	}

	@SuppressWarnings("unchecked")
	@Override
	public T queryById(String id) {
		return (T) sessionFactory.getCurrentSession().get(entityClass, id);
	}

	@Override
	public List<T> queryAll() {
		String hql = "from " + entityClass.getSimpleName();
		return queryForList(hql, null);
	}

	@SuppressWarnings("unchecked")
	protected T queryForObject(String hql, Object[] params) {
		Query query = sessionFactory.getCurrentSession().createQuery(hql);
		setQueryParams(query, params);
		return (T) query.uniqueResult();
	}

	@SuppressWarnings("unchecked")
	protected T queryForTopObject(String hql, Object[] params) {
		Query query = sessionFactory.getCurrentSession().createQuery(hql);
		setQueryParams(query, params);
		return (T) query.setFirstResult(0).setMaxResults(1).uniqueResult();
	}

	@SuppressWarnings("unchecked")
	protected List<T> queryForList(String hql, Object[] params) {
		Query query = sessionFactory.getCurrentSession().createQuery(hql);
		setQueryParams(query, params);
		return query.list();
	}

	@SuppressWarnings("unchecked")
	protected List<T> queryForList(final String hql, final Object[] params,
			final int recordNum) {
		Query query = sessionFactory.getCurrentSession().createQuery(hql);
		setQueryParams(query, params);
		return query.setFirstResult(0).setMaxResults(recordNum).list();
	}

	private void setQueryParams(Query query, Object[] params) {
		if (null == params) {
			return;
		}
		for (int i = 0; i < params.length; i++) {
			query.setParameter(i, params[i]);
		}
	}
}
```

这个抽象类实现了IGenericDAO的所有方法，具体业务DAO的实现类，就不需要重复实现这些方法了。这里因为session.get()和session.load()方法，都需要传入一个Class类型的参数，所以定义了entityClass字段，在具体业务类的构造方法中传入，下面会看到。另外有一个办法是用反射的方法，来获取entityClass字段，就不需要在具体子类的构造方法中再传入了。不过我个人觉得传入也不是很麻烦，就没有这么做 

这个类除了实现了IGenericDAO里定义的public方法之外，还提供了protected的queryForObject()和queryForList()方法，可以为具体子类提供一些便利 

这个通用DAO还不是很完善，主要是还可以补充更多的方法，以及考虑分页。为了简化的需要，这里省略了

```
public interface IUserDAO extends IGenericDAO<User> {

	public User queryByName(String userName);

}
```

这是具体业务DAO的接口，除了通用的方法之外，增加了一个按照name查询的方法，所以就要单独定义此方法

```
@Repository
public class UserDAO extends GenericDAO<User> implements IUserDAO {

	public UserDAO() {
		super(User.class);
	}

	@Override
	public User queryByName(String userName) {
		String hql = "from User u where u.name = ?";
		return queryForObject(hql, new Object[] { userName });
	}

}
```

这是具体业务DAO的实现类，实现了接口里的queryByName()方法，并且在构造参数中传入了User.class，用于初始化GenericDAO里的entityClass字段 

此外，这个类需要用@Repository注解，声明为spring bean DAO层里是不能声明事务的，也不能自行捕获异常，如果有特殊需求必须捕获的话，也要在处理之后，重新抛出来。否则Service层的事务就失效了 

# Service层

```
@Transactional(propagation = Propagation.REQUIRED, readOnly = false)
public interface IBookService {

	void addBook(Book book);

}
```

只要在接口上用@Transactional注解，此接口内的所有方法就自动声明为事务了，方法即是事务的边界。 

注意事务是在接口上声明的，一般不在实现类上声明 

后面的propagation参数，至少要到REQUIRED，否则No Session found for current thread，我也不知道这算不算一个BUG，还是spring认为是一个强制要求

```
@Service
public class BookService implements IBookService {

	@Autowired
	private IBookDAO bookDAO;

	@Override
	public void addBook(Book book) {
		bookDAO.insert(book);
	}
}
```

这个Service的实现类就很简单了，不需要重复声明事务，但是需要用@Service注解将自身声明为一个spring bean（因为可能还会注入上层），另外用@Autowired注解，将之前声明的DAO注入 

# 单元测试

接下来说明一下单元测试的方法 

![](http://dl.iteye.com/upload/attachment/0072/4234/7951a257-d88e-3015-88d8-4660ca848bee.png)

这里要注意Source folder选到test，不然就会生成到src目录下了，然后可以视情况勾选setUp() 

生成的单元测试类

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:beans.xml")
@Transactional
public class BookDAOTest {

	@Autowired
	private BookDAO bookDAO;

	@Test
	public void testQueryByIsbn() {
		String isbn = "123";
		Book result = bookDAO.queryByIsbn(isbn);
		String name = result.getName();
		assertEquals("thinking in java", name);
	}

	@Test
	public void testInsert() {
		Book book = new Book();
		book.setName("bai ye xing");
		book.setIsbn("be bought yesterday");
		bookDAO.insert(book);
	}

	@Test
	public void testDelete() {
		String id = "test_1";
		Book target = bookDAO.queryById(id);
		bookDAO.delete(target);
	}

	@Test
	public void testUpdate() {
		String id = "test_1";
		Book target = bookDAO.queryById(id);
		target.setName("i am changeid");
		bookDAO.update(target);
	}

	@Test
	public void testQueryById() {
		String id = "test_1";
		Book target = bookDAO.queryById(id);
		String name = target.getName();
		assertEquals("thinking in java", name);
	}

	@Test
	public void testQueryAll() {
		List<Book> books = bookDAO.queryAll();
		assertEquals(3, books.size());
	}

}
```

注解为@Test的方法，会被认为是单元测试方法被执行，注解为@Before的方法，会在每个单元测试方法执行之前被执行

```
@Autowired
	private BookDAO bookDAO;
```

这里是把要单元测试的目标类注入进来 

下面重点介绍一下类上面的几个注解：

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:beans.xml")
```

加上@RunWith注解之后，单元测试类会在spring容器里执行，这会带来很多便利。 @ContextConfiguration注解，可以指定要加载的spring配置文件路径。如果对spring配置文件进行了恰当的拆分，就可以在不同的单元测试类里，仅加载必要的配置文件

```
@Transactional
```

这行注解是最关键的，前面已经提到，因为在DAO层是没有声明事务的，所以如果直接执行的话，就会抛出No Session found for current thread 所以需要加上这句注解，在执行单元测试时，开启事务，就可以规避这个问题。同时也不会影响到实际的事务 

此外还引入了一个额外的好处，就是加上了这个注解之后，单元测试对数据库的改动会被自动回滚，避免不同单元测试方法之间的耦合。这个特性在实际跑单元测试里是很方便的 

实际运行一下这个单元测试类，可以在控制台看到以下输出： 

2012-8-16 19:37:42 org.springframework.test.context.transaction.TransactionalTestExecutionListener startNewTransaction 信息: Began transaction (1): transaction manager [org.springframework.orm.hibernate4.HibernateTransactionManager@183d59c]; rollback [true] 1903 [main] WARN  o.h.hql.internal.ast.HqlSqlWalker - [DEPRECATION] Encountered positional parameter near line 1, column 60.  Positional parameter are considered deprecated; use named parameters or JPA-style positional parameters instead. Hibernate:     select         book0_.ID as ID0_,         book0_.NAME as NAME0_,         book0_.ISBN as ISBN0_     from         developframeworkschema.book book0_     where         book0_.ISBN=? 
2012-8-16 19:37:42 org.springframework.test.context.transaction.TransactionalTestExecutionListener endTransaction 信息: Rolled back transaction after test execution for test context 

每个方法开始之前，都会开启一个新事务，在执行完毕之后，该事务都会被回滚 

其中还有一行警告信息：[DEPRECATION] Encountered positional parameter near line 1, column 60.  Positional parameter are considered deprecated; use named parameters or JPA-style positional parameters instead. 这是因为在GenericDAO中采用了hibernate4不推荐的写法：

```
private void setQueryParams(Query query, Object[] params) {
		if (null == params) {
			return;
		}
		for (int i = 0; i < params.length; i++) {
			query.setParameter(i, params[i]);
		}
	}
```

hibernate4的建议，是把

```
String hql = "from User u where u.name = ?";
Query query = sessionFactory.getCurrentSession().createQuery(hql);
query.setParameter(0, name);
```

改成

```
String hql = "from User u where u.name = :name";
Query query = this.getSession().createQuery(hql);
query.setParameter("name", name);
```

鉴于自动回滚这个特性很方便，对Service层组件进行单元测试的时候，也推荐加上@Transactional注解