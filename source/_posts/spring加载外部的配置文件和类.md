title: spring加载外部的配置文件和类
date: 2013-09-24 11:14
categories: java 
---
今天同事遇到一个需求： 在外部以jar包的形式存放若干个插件，其中包含插件的类，以及spring配置文件；jar包不在classpath里 
<!--more-->

要实现这个需求，需要用到自定义的ClassLoader，并调用一些spring提供的API 

首先是jar包的结构： 

![](http://dl.iteye.com/upload/attachment/0076/8637/6b5a59cb-340a-33d3-a124-4d0eb1fb4a51.png)

其中net文件夹下面，放了要从外部加载的目标类

```
package net.kyfxbl.test;

public class User {

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public void sayName() {
		System.out.println(getName());
	}
}
```

配置文件spring-plugin.xml：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="myUser" class="net.kyfxbl.test.User">
		<property name="name" value="kyfxbl" />
	</bean>

</beans>
```

现在目标，就是读取hehe.jar中的spring-plugin.xml文件，然后实例化一个User对象 

示例代码如下，要说明的是，代码很简陋，仅供参考。主要是由于本人spring学艺不精，所以API调用那块，肯定不是最佳实践，绝对是可以优化的：

```
public class Main {

	public static void main(String[] args) {

		try {

			ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
					"spring-config.xml");

			DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) applicationContext
					.getAutowireCapableBeanFactory();

			String configurationFilePath = "jar:file:/C:/hehe.jar!/spring-plugin.xml";

			URL url = new URL(configurationFilePath);

			UrlResource urlResource = new UrlResource(url);

			XmlBeanFactory xbf = new XmlBeanFactory(urlResource);

			String[] beanIds = xbf.getBeanDefinitionNames();

			for (String beanId : beanIds) {
				BeanDefinition bd = xbf.getMergedBeanDefinition(beanId);
				beanFactory.registerBeanDefinition(beanId, bd);
			}

			// 以下这行设置BeanFactory的ClassLoader，以加载外部类
			setBeanClassLoader(beanFactory);

			Object pluginBean = applicationContext.getBean("myUser");

			tryInvoke(pluginBean);

		} catch (Exception exc) {
			exc.printStackTrace();
		}

	}

	private static void setBeanClassLoader(
			DefaultListableBeanFactory beanFactory)
			throws MalformedURLException {
		String jarFilePath = "c://hehe.jar";
		URL jarUrl = new File(jarFilePath).toURI().toURL();
		URL[] urls = new URL[] { jarUrl };
		URLClassLoader cl = new URLClassLoader(urls);
		beanFactory.setBeanClassLoader(cl);
	}

	private static void tryInvoke(Object bean) throws SecurityException,
			NoSuchMethodException, IllegalArgumentException,
			IllegalAccessException, InvocationTargetException {

		Class<?> paramTypes[] = new Class[0];
		Method method = bean.getClass()
				.getDeclaredMethod("sayName", paramTypes);
		Object paramValues[] = new Object[0];
		method.invoke(bean, paramValues);

	}

}
```
如果注释掉

```
setBeanClassLoader(beanFactory);
```

则报异常： Caused by: java.lang.ClassNotFoundException: net.kyfxbl.test.User 

所以关键就是通过setBeanClassLoader()方法，修改beanFactory默认的ClassLoader 

原理比较简单：如果不做特殊配置的话，spring将使用默认的ClassLoader（也就是App ClassLoader），那么就不会加载hehe.jar中的类。所以通过自定义URLClassLoader，并将其设置为BeanFactory的BeanClassLoader，就可以将hehe.jar加载进来了。 

附带的，为了证明spring默认的ClassLoader就是App ClassLoader，补充了几行测试代码

```
// 打印ClassLoader看看
			printClassLoader(beanFactory);

			// 以下这行设置BeanFactory的ClassLoader，以加载外部类
			setBeanClassLoader(beanFactory);

```

```
private static void printClassLoader(DefaultListableBeanFactory beanFactory) {
		ClassLoader defaultBeanClassLoader = beanFactory.getBeanClassLoader();
		System.out.println(defaultBeanClassLoader);

		ClassLoader currentClassLoader = Main.class.getClassLoader();
		System.out.println(currentClassLoader);
	}

```

运行结果： sun.misc.Launcher$AppClassLoader@addbf1 sun.misc.Launcher$AppClassLoader@addbf1