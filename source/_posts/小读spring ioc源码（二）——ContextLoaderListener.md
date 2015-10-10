title: 小读spring ioc源码（二）——ContextLoaderListener
date: 2013-09-24 11:05
categories: 源码阅读  
---
web应用中，spring的启动过程
<!--more-->

# 通过listener启动spring

实际开发中，比较多的项目是web项目，这时候加载spring，是在web.xml里配置一个Listener

```
<listener>
    	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

这个ContextLoaderListener就是web应用中，加载spring的入口。如果不是web应用，一般通过实例化一个ApplicationContext的子类来加载spring。过程是大同小异的，这部分在后面的博客里再说，不是这篇的重点 

接下来就从ContextLoaderListener说起，入口是contextInitialized方法
```
public void contextInitialized(ServletContextEvent event) {
		this.contextLoader = createContextLoader();
		if (this.contextLoader == null) {
			this.contextLoader = this;
		}
		this.contextLoader.initWebApplicationContext(event.getServletContext());
	}
```

这里面的createContextLoader()是一个@Deprecated方法，直接返回null。这里ContextLoaderListener是继承自ContextLoader，所以有一句看起来比较奇怪的代码

```
this.contextLoader = this;
```

为什么这么做的原因，我想可能是为了把大部分的功能代码移到ContextLoader里，可以保持ContextLoaderListener自身的代码比较简单 

# 创建ApplicationContext

然后接下来关键的代码

```
this.contextLoader.initWebApplicationContext(event.getServletContext());
```

这个initWebApplicationContext()方法是在ContextLoader里定义的

```
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
		if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
			throw new IllegalStateException(
					"Cannot initialize context because there is already a root application context present - " +
					"check whether you have multiple ContextLoader* definitions in your web.xml!");
		}

		Log logger = LogFactory.getLog(ContextLoader.class);
		servletContext.log("Initializing Spring root WebApplicationContext");
		if (logger.isInfoEnabled()) {
			logger.info("Root WebApplicationContext: initialization started");
		}
		long startTime = System.currentTimeMillis();

		try {
			// Store context in local instance variable, to guarantee that
			// it is available on ServletContext shutdown.
			if (this.context == null) {
				this.context = createWebApplicationContext(servletContext);
			}
			if (this.context instanceof ConfigurableWebApplicationContext) {
				configureAndRefreshWebApplicationContext((ConfigurableWebApplicationContext)this.context, servletContext);
			}
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

			ClassLoader ccl = Thread.currentThread().getContextClassLoader();
			if (ccl == ContextLoader.class.getClassLoader()) {
				currentContext = this.context;
			}
			else if (ccl != null) {
				currentContextPerThread.put(ccl, this.context);
			}

			if (logger.isDebugEnabled()) {
				logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
						WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
			}
			if (logger.isInfoEnabled()) {
				long elapsedTime = System.currentTimeMillis() - startTime;
				logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
			}

			return this.context;
		}
		catch (RuntimeException ex) {
			logger.error("Context initialization failed", ex);
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
			throw ex;
		}
		catch (Error err) {
			logger.error("Context initialization failed", err);
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
			throw err;
		}
	}

```

比较长，可以分解开来看，首先是：

```
if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
			throw new IllegalStateException(
					"Cannot initialize context because there is already a root application context present - " +
					"check whether you have multiple ContextLoader* definitions in your web.xml!");
		}

		Log logger = LogFactory.getLog(ContextLoader.class);
		servletContext.log("Initializing Spring root WebApplicationContext");
		if (logger.isInfoEnabled()) {
			logger.info("Root WebApplicationContext: initialization started");
		}
		long startTime = System.currentTimeMillis();
```

首先检查是不是已经注册了ApplicationContext作为ServletContext的一个属性，如果是的话，就抛出异常，避免重复注册 

然后获取日志组件，这里其实有个问题，就是Common Logging实际上比较老，现在很多开源框架，都改用slf4j作为日志组件的facade，但是spring这里把它写死了，所以就很难无缝地引入slf4j了 

然后记录了启动日志，并记录开始启动的时间 

接下来就是本方法的实体，省略了try/catch，只看业务部分

```
if (this.context == null) {
				this.context = createWebApplicationContext(servletContext);
			}
			if (this.context instanceof ConfigurableWebApplicationContext) {
				configureAndRefreshWebApplicationContext((ConfigurableWebApplicationContext)this.context, servletContext);
			}
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

			ClassLoader ccl = Thread.currentThread().getContextClassLoader();
			if (ccl == ContextLoader.class.getClassLoader()) {
				currentContext = this.context;
			}
			else if (ccl != null) {
				currentContextPerThread.put(ccl, this.context);
			}

			if (logger.isDebugEnabled()) {
				logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
						WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
			}
			if (logger.isInfoEnabled()) {
				long elapsedTime = System.currentTimeMillis() - startTime;
				logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
			}

			return this.context;
```

这段代码里，有几个方法是比较重要的，createWebApplicationContext()实际创建了WebApplicationContext，configureAndRefreshWebApplicationContext()初始化了刚创建的WebApplicationContext 

在创建和初始化完成之后，将其设置为ServletContext的一个属性，这是WebApplicationContextUtils能够获取到spring容器的基础 

然后是对ClassLoader的一些处理

```
ClassLoader ccl = Thread.currentThread().getContextClassLoader();
			if (ccl == ContextLoader.class.getClassLoader()) {
				currentContext = this.context;
			}
			else if (ccl != null) {
				currentContextPerThread.put(ccl, this.context);
			}
```

这段我没有看懂它是要干嘛 

之后又是日志记录和时间记录 

从上面我们可以看到，ContextLoader的加载，整个流程是很清晰的。其中的核心是WebApplicationContext的创建和初始化，然后将其设置为ServletContext的一个属性，这样就可以很容易地从工具类中获取到spring容器了 

# 初始化ApplicationContext的细节

接下来就看一下WebApplicationContext是怎么创建和初始化的

```
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
		Class<?> contextClass = determineContextClass(sc);
		if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
			throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
					"] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
		}
		ConfigurableWebApplicationContext wac =
				(ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
		return wac;
	}
```

这个方法比较简单，首先通过determineContextClass()方法，来确定WebApplicationContext的类型

```
protected Class<?> determineContextClass(ServletContext servletContext) {
		String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
		if (contextClassName != null) {
			try {
				return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
			}
			catch (ClassNotFoundException ex) {
				throw new ApplicationContextException(
						"Failed to load custom context class [" + contextClassName + "]", ex);
			}
		}
		else {
			contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
			try {
				return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
			}
			catch (ClassNotFoundException ex) {
				throw new ApplicationContextException(
						"Failed to load default context class [" + contextClassName + "]", ex);
			}
		}
	}
```

如果没有在web.xml中特别配置ServletContext的InitParameter的话，会返回XmlWebApplicationContext，这也是大部分情况 

然后就通过BeanUtils.instantiateClass()方法，创建XmlWebApplicationContext并返回。XmlWebApplicationContext是WebApplicationContext接口一个最下层的实现类，这条继承路径上，自上而下分别是WebApplicationContext、ConfigurableWebApplicationContext、AbstractRefreshableWebApplicationContext、XmlWebApplicationContext。ApplicationContext接口的继承体系是比较复杂的

创建完毕之后，就要对刚创建的XmlWebApplicationContext进行初始化：

```
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
		if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
			// The application context id is still set to its original default value
			// -> assign a more useful id based on available information
			String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
			if (idParam != null) {
				wac.setId(idParam);
			}
			else {
				// Generate default id...
				if (sc.getMajorVersion() == 2 && sc.getMinorVersion() < 5) {
					// Servlet <= 2.4: resort to name specified in web.xml, if any.
					wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
							ObjectUtils.getDisplayString(sc.getServletContextName()));
				}
				else {
					wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
							ObjectUtils.getDisplayString(sc.getContextPath()));
				}
			}
		}

		// Determine parent for root web application context, if any.
		ApplicationContext parent = loadParentContext(sc);

		wac.setParent(parent);
		wac.setServletContext(sc);
		String initParameter = sc.getInitParameter(CONFIG_LOCATION_PARAM);
		if (initParameter != null) {
			wac.setConfigLocation(initParameter);
		}
		customizeContext(sc, wac);
		wac.refresh();
	}
```

这个方法也比较长，需要拆开看

```
if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
			// The application context id is still set to its original default value
			// -> assign a more useful id based on available information
			String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
			if (idParam != null) {
				wac.setId(idParam);
			}
			else {
				// Generate default id...
				if (sc.getMajorVersion() == 2 && sc.getMinorVersion() < 5) {
					// Servlet <= 2.4: resort to name specified in web.xml, if any.
					wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
							ObjectUtils.getDisplayString(sc.getServletContextName()));
				}
				else {
					wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
							ObjectUtils.getDisplayString(sc.getContextPath()));
				}
			}
		}
```

这个部分是给XmlWebApplicationContext设置一个id，作为一个标识，不是很重要 

然后是设置继承体系：
```
ApplicationContext parent = loadParentContext(sc);
		wac.setParent(parent);
```

这里也是依赖ServletContext的InitParameter，一般来说不会发生 

然后是给XmlWebApplicationContext设置ServletContext，这步很简单，但这样设置之后，从spring容器内部，就可以获取到ServletContext了，所以是很重要的

```
wac.setServletContext(sc);
```

接下来是设置spring配置文件的路径
```
String initParameter = sc.getInitParameter(CONFIG_LOCATION_PARAM);
		if (initParameter != null) {
			wac.setConfigLocation(initParameter);
		}
```

如果没有配置的话，之后就默认会在WEB-INF下寻找applicationContext.xml，如果配置了的话：
```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value> 
        	WEB-INF/beans.xml
        	WEB-INF/cxf.xml
        </param-value>
  </context-param>
```

就会在指定的路径加载配置文件 

然后是一个留给用户扩展的方法customizeContext()，大部分情况下，什么事情也不会发生 

最后就是核心方法
```
wac.refresh();
```

这个refresh()方法，是在ApplicationContext继承体系中，比较上层的ConfigurableApplicationContext里定义的，是XmlWebApplicationContext初始化的核心，在这个方法里，会完成资源文件的加载、配置文件解析、Bean定义的注册、组件的初始化等核心工作，在下一篇博客里，再详细介绍