title: 小读spring ioc源码（四）——BeanDefinitionReader
date: 2013-09-24 11:06
categories: 源码阅读 
---
上一篇博客说到，ApplicationContext将解析BeanDefinition的工作委托给BeanDefinitionReader组件，这篇就接着分析一下BeanDefinition的解析过程
<!--more-->

入口是loadBeanDefinitions方法

```
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			for (String configLocation : configLocations) {
				reader.loadBeanDefinitions(configLocation);
			}
		}
	}
```

这是解析过程最外围的代码，首先要获取到配置文件的路径，这在之前已经完成了。然后将每个配置文件的路径，作为参数传给BeanDefinitionReader的loadBeanDefinitions方法里

```
public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(location, null);
	}
```

这个方法又调用了重载方法
```
public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);
				int loadCount = loadBeanDefinitions(resources);
				if (actualResources != null) {
					for (Resource resource : resources) {
						actualResources.add(resource);
					}
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
				}
				return loadCount;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int loadCount = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
			}
			return loadCount;
		}
	}
```

这个方法比较长，BeanDefinitionReader不能直接加载配置文件，需要把配置文件封装成Resource，然后才能调用重载方法loadBeanDefinitions()。所以这个方法其实就是2段，第一部分是委托ResourceLoader将配置文件封装成Resource，第二部分是调用loadBeanDefinitions()，对Resource进行解析 

而这里的ResourceLoader，就是前面的XmlWebApplicationContext，因为ApplicationContext接口，是继承自ResourceLoader接口的 

Resource也是一个接口体系，在web环境下，这里就是ServletContextResource 

接下来进入重载方法loadBeanDefinitions()

```
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int counter = 0;
		for (Resource resource : resources) {
			counter += loadBeanDefinitions(resource);
		}
		return counter;
	}
```

这里就不用说了，就是把每一个Resource作为参数，继续调用重载方法。读spring源码，会发现重载方法特别多

```
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
	}
```

还是重载方法，不过这里对传进来的Resource又进行了一次封装，变成了编码后的Resource，问题不大

```
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
		try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}
```

这个就是loadBeanDefinitions()的最后一个重载方法，比较长，可以拆看来看

```
Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isInfoEnabled()) {
			logger.info("Loading XML bean definitions from " + encodedResource.getResource());
		}

		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		if (currentResources == null) {
			currentResources = new HashSet<EncodedResource>(4);
			this.resourcesCurrentlyBeingLoaded.set(currentResources);
		}
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
```

这第一部分，是处理线程相关的工作，把当前正在解析的Resource，设置为当前Resource

```
try {
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
```

这里是第二部分，是核心，首先把Resource还原为InputStream，然后调用实际解析的方法doLoadBeanDefinitions()。可以看到，这种命名方式是很值得学习的，一种业务方法，比如parse()，可能需要做一些外围的工作，然后实际解析的方法，可以命名为doParse()。这种doXXX()的命名方法，在很多开源框架中都有应用，比如logback等 

接下来就看一下这个doLoadBeanDefinitions()方法
```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			int validationMode = getValidationModeForResource(resource);
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```

抛开处理异常的若干catch体，只看实体部分

```
int validationMode = getValidationModeForResource(resource);
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
			return registerBeanDefinitions(doc, resource);
```

首先确认校验模式，然后委托documentLoader，将InputStream读取成标准的Document对象，然后调用registerBeanDefinitions()，进行解析工作 

这里注意两点 首先这个Document对象，是W3C定义的标准XML对象，跟spring无关。其次这个registerBeanDefinitions方法，我觉得命名有点误导性。因为这个时候实际上解析还没有开始，怎么直接就注册了呢。比较好的命名，我觉得可以是parseAndRegisterBeanDefinitions() 

接下来就看一下这个核心方法registerBeanDefinitions

```
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		documentReader.setEnvironment(this.getEnvironment());
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

这个方法同样要注意2个事 

首先，BeanDefinitionReader组件，也不是自己完成解析的，它要委托给BeanDefinitionDocumentReader来完成（BeanDefinitionDocumentReader还会有一次委托）

其次，可以看到实际解析的方法registerBeanDefinitions()前后，还各有一个计数，这里调用了getRegistry()方法，来得到一个BeanRegistry。这里实际取到的是DefaultListableBeanFactory，因为这个类实现了BeanRegistry接口。（DefaultListableBeanFactory是一个非常重要的类，它不仅是BefanFactory的默认实现，而且大部分的ApplicationContext实现，内部都持有一个它的实例） 

接下来就进入registerBeanDefinitions()方法看看

```
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;

		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement();

		doRegisterBeanDefinitions(root);
	}
```

处理完外围事务之后，进入doRegisterBeanDefinitions()方法，这种命名规范，上文已经介绍过了

```
protected void doRegisterBeanDefinitions(Element root) {
		String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
		if (StringUtils.hasText(profileSpec)) {
			Assert.state(this.environment != null, "environment property must not be null");
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			if (!this.environment.acceptsProfiles(specifiedProfiles)) {
				return;
			}
		}

		// any nested <beans> elements will cause recursion in this method. In
		// order to propagate and preserve <beans> default-* attributes correctly,
		// keep track of the current (parent) delegate, which may be null. Create
		// the new (child) delegate with a reference to the parent for fallback purposes,
		// then ultimately reset this.delegate back to its original (parent) reference.
		// this behavior emulates a stack of delegates without actually necessitating one.
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createHelper(readerContext, root, parent);

		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}
```

这个方法也比较长，拆开来看
```
String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
		if (StringUtils.hasText(profileSpec)) {
			Assert.state(this.environment != null, "environment property must not be null");
			String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			if (!this.environment.acceptsProfiles(specifiedProfiles)) {
				return;
			}
		}
```

如果配置文件中<beans>元素，配有profile属性，就会进入这一段，不过一般都是不会的
```
BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createHelper(readerContext, root, parent);

		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
```
然后这里创建了BeanDefinitionParserDelegate对象，preProcessXml()和postProcessXml()都是空方法，核心就是parseBeanDefinitions()方法。这里又把BeanDefinition解析和注册的工作，委托给了BeanDefinitionParserDelegate对象，在parseBeanDefinitions()方法中完成 

总的来说，解析工作的委托链是这样的：XmlWebApplicationContext，XmlBeanDefinitionReader，DefaultBeanDefinitionDocumentReader，BeanDefinitionParserDelegate 

XmlWebApplicationContext作为最外围的组件，发起解析的请求

XmlBeanDefinitionReader将配置文件路径封装为Resource，读取出w3c定义的Document对象，然后委托给DefaultBeanDefinitionDocumentReader

DefaultBeanDefinitionDocumentReader就开始做实际的解析工作了，但是涉及到bean的具体解析，它还是会继续委托给BeanDefinitionParserDelegate来做 

接下来在parseBeanDefinitions()方法中发生了什么，以及BeanDefinitionParserDelegate类完成的工作，在下一篇博客中继续介绍