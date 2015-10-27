title: 小读spring ioc源码（三）——XmlWebApplicationContext初始化的整体过程
date: 2013-09-24 11:05
categories: 源码阅读 
---
上一篇说到，ContextLoaderListener在web应用启动之后，经过一系列前置步骤，将初始化XmlWebApplicationContext的工作，委托给AbstractApplicationContext。这篇就继续介绍一下，是怎样一步步完成配置文件的加载、解析、注册的 
<!--more-->

# 整体流程

先看一下这里的refresh()方法

```
public void refresh() throws BeansException, IllegalStateException {

		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}
		}
	}
```

首先这个方法是同步的，以避免重复刷新。然后刷新的每个步骤，都放在单独的方法里，比较清晰，可以按顺序一个个看 

首先是prepareRefresh()方法

```
protected void prepareRefresh() {
		this.startupDate = System.currentTimeMillis();

		synchronized (this.activeMonitor) {
			this.active = true;
		}

		if (logger.isInfoEnabled()) {
			logger.info("Refreshing " + this);
		}

		// Initialize any placeholder property sources in the context environment
		initPropertySources();

		// Validate that all properties marked as required are resolvable
		// see ConfigurablePropertyResolver#setRequiredProperties
		this.environment.validateRequiredProperties();
	}
```

这个方法里做的事情不多，记录了开始时间，输出日志，另外initPropertySources()方法和validateRequiredProperties()方法一般都没有做什么事 

然后是核心的obtainFreshBeanFactory()方法，这个方法是初始化BeanFactory，是整个refresh()方法的核心，其中完成了配置文件的加载、解析、注册，后面会专门详细说 

这里要说明一下，ApplicationContext实现了BeanFactory接口，并实现了ResourceLoader、MessageSource等接口，可以认为是增强的BeanFactory。但是ApplicationContext并不自己重复实现BeanFactory定义的方法，而是委托给DefaultListableBeanFactory来实现。这种设计思路也是值得学习的 

后面的prepareBeanFactory()、postProcessBeanFactory()、invokeBeanFactoryPostProcessors()、registerBeanPostProcessors()、initMessageSource()、initApplicationEventMulticaster()、onRefresh()、registerListeners()、finishBeanFactoryInitialization()、finishRefresh()等方法，是添加一些后处理器、广播、拦截器等，就不一个个细说了

其中的关键方法是finishBeanFactoryInitialization()，在这个方法中，会对刚才注册的Bean（不延迟加载的），进行实例化，所以也是一个核心方法 

# spring.xml处理

从整体上介绍完了流程，接下来就重点看obtainFreshBeanFactory()方法，上文说到，在这个方法里，完成了配置文件的加载、解析、注册

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
```

这个方法做了2件事，首先通过refreshBeanFactory()方法，创建了DefaultListableBeanFactory的实例，并进行初始化

```
protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

首先如果已经有BeanFactory实例，就先清空。然后通过createBeanFactory()方法，创建一个DefaultListableBeanFactory的实例

```
protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
	}
```

接下来设置ID唯一标识
```
beanFactory.setSerializationId(getId());
```

然后允许用户进行一些自定义的配置
```
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
		if (this.allowBeanDefinitionOverriding != null) {
			beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.allowCircularReferences != null) {
			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
		}
		beanFactory.setAutowireCandidateResolver(new QualifierAnnotationAutowireCandidateResolver());
	}
```

最后，就是核心的loadBeanDefinitions()方法
```
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// Create a new XmlBeanDefinitionReader for the given BeanFactory.
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
		initBeanDefinitionReader(beanDefinitionReader);
		loadBeanDefinitions(beanDefinitionReader);
	}
```

这里首先会创建一个XmlBeanDefinitionReader的实例，然后进行初始化
```
// Create a new XmlBeanDefinitionReader for the given BeanFactory.
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// Configure the bean definition reader with this context's
		// resource loading environment.
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// Allow a subclass to provide custom initialization of the reader,
		// then proceed with actually loading the bean definitions.
		initBeanDefinitionReader(beanDefinitionReader);
```

这里要说明一下，ApplicationContext并不自己负责配置文件的加载、解析、注册，而是将这些工作委托给XmlBeanDefinitionReader来做。

```
loadBeanDefinitions(beanDefinitionReader);
```

这行代码，就是Bean定义读取实际发生的地方。这里的工作，主要是XmlBeanDefinitionReader来完成的，下一篇博客会详细介绍这个过程