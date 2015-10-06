title: 读logback源码系列文章（六）——ContextInitializer
date: 2013-09-24 10:57
categories: 源码阅读
---
本系列是阅读logback源码的总结。本文介绍logback是怎么读取配置文件并初始化整个框架的
<!--more-->

这篇博客我们接着上一篇的主题，来介绍一下logback是怎么读取配置文件并初始化整个框架的。还是老规矩，先上总览图 

# 整体类图

![](http://dl.iteye.com/upload/attachment/566379/1f7bb83b-99e1-3bfc-a1ef-ebde7992ee02.jpg)

从图中可以看到，logback框架的初始化是由ContextInitializer类来负责完成的，而实际进行配置的是GenericConfigurator类，它调用SaxEventRecorder类来负责读取logback.xml文件，然后由Interpreter类来进行解析，而最后真正的初始化工作，是由一系列Action组件来完成

# 读取配置文件流程

接下来就实际看看代码
```
try {
        new ContextInitializer(defaultLoggerContext).autoConfig();
      } catch (JoranException je) {
        Util.report("Failed to auto configure default logger context", je);
      }
```

以上代码来自StaticLoggerBinder类，它在提供LoggerContext之前，就对LoggerContext进行初始化，初始化的入口方法，即是ContextInitializer类的autoConfig()方法

```
public void autoConfig() throws JoranException {
    StatusListenerConfigHelper.installIfAsked(loggerContext);
    URL url = findURLOfDefaultConfigurationFile(true);
    if (url != null) {
      configureByResource(url);
    } else {
      BasicConfigurator.configure(loggerContext);
    }
  }
```

这里首先调用findURLOfDefaultConfigurationFile()方法，来寻找一个配置文件，一般就是logback.xml文件，如果没有找到，则用BasicConfigurator来进行默认配置，否则就调用configureByResource()方法，根据配置文件进行配置
```
public void configureByResource(URL url) throws JoranException {
    if (url == null) {
      throw new IllegalArgumentException("URL argument cannot be null");
    }
    if (url.toString().endsWith("groovy")) {
      if (EnvUtil.isGroovyAvailable()) {
        // avoid directly referring to GafferConfigurator so as to avoid
        // loading  groovy.lang.GroovyObject . See also http://jira.qos.ch/browse/LBCLASSIC-214
        GafferUtil.runGafferConfiguratorOn(loggerContext, this, url);
      } else {
        StatusManager sm = loggerContext.getStatusManager();
        sm.add(new ErrorStatus("Groovy classes are not available on the class path. ABORTING INITIALIZATION.",
                loggerContext));
      }
    }
    if (url.toString().endsWith("xml")) {
      JoranConfigurator configurator = new JoranConfigurator();
      configurator.setContext(loggerContext);
      configurator.doConfigure(url);
    }
  }
```

这里如果配置文件是以.groovy结尾，就调用另一种方法进行配置，我没有跟进去看；如果是常规以.xml文件结尾，那么就用JoranConfigurator类来进行配置
```
final public void doConfigure(URL url) throws JoranException {
    try {
      informContextOfURLUsedForConfiguration(url);
      URLConnection urlConnection = url.openConnection();
      // per http://jira.qos.ch/browse/LBCORE-105
      // per http://jira.qos.ch/browse/LBCORE-127
      urlConnection.setUseCaches(false);

      InputStream in = urlConnection.getInputStream();
      doConfigure(in);
      in.close();
    } catch (IOException ioe) {
      String errMsg = "Could not open URL [" + url + "].";
      addError(errMsg, ioe);
      throw new JoranException(errMsg, ioe);
    }
  }
```

这个方法则是打开配置文件的URL，转化成InputStream，然后继续调用
```
// this is the most inner form of doConfigure whereto other doConfigure
  // methods ultimately delegate
  final public void doConfigure(final InputSource inputSource)
      throws JoranException {

    if(!ConfigurationWatchListUtil.wasConfigurationWatchListReset(context)) {
      informContextOfURLUsedForConfiguration(null);
    }
    SaxEventRecorder recorder = new SaxEventRecorder();
    recorder.setContext(context);
    recorder.recordEvents(inputSource);
    buildInterpreter();
    // disallow simultaneous configurations of the same context
    synchronized (context.getConfigurationLock()) {
      interpreter.play(recorder.saxEventList);
    }
  }
```

上面的WatchList，一般是走不进这个分支的，我也就没深入地看。接下来创建一个SaxEventRecorder类，由它负责读取XML文件，并解析成流，创建SaxEvent，这部分功能是JDK提供的。然后是一个核心方法buildInterpreter()
```
protected void buildInterpreter() {
    RuleStore rs = new SimpleRuleStore(context);
    addInstanceRules(rs);
    this.interpreter = new Interpreter(context, rs, initialPattern());
    InterpretationContext ec = interpreter.getInterpretationContext();
    ec.setContext(context);
    addImplicitRules(interpreter);
    addDefaultNestedComponentRegistryRules(ec.getDefaultNestedComponentRegistry());
  }
```

这里的核心是初始化了interpreter字段，这种设计其实我觉得是不太好的，因为把interpreter字段的初始化隐藏得很深，不容易看到 

Interpreter在框架的配置中是一个很关键的类，logback是依赖这个类来进行初始化工作的。这里用到了RuleStore接口，RuleStore只有一个唯一的实现类SimpleRuleStore，这个类的作用是，把Pattern匹配到Action上，因为所有的配置工作，最终是由Action来完成的。作者对这个类的设计是十分精巧的，个人认为logback框架的初始化设计思路，是非常值得学习的，这部分内容也是我阅读logback源码感觉收获最大的地方 

buildInterpreter()方法调用之后，调用play(recorder.saxEventList)方法，开始进行配置。注意，这个时候logback.xml文件已经被解析成了SAX流，并且保存在SaxEventRecorder的saxEventList字段里了。接下来就要重点分析一下Interpreter这个核心类的字段和方法，一些不重要和一目了然的字段和方法，这里就忽略了

# Interpreter关键字段解析

```
final private RuleStore ruleStore;
  final private InterpretationContext interpretationContext;
private Pattern pattern;
Stack<List> actionListStack;
```

以上的字段中，ruleStore正如前面提到的，是用来根据Sax节点，来匹配对应的Action。ruleStore是在Interpreter创建时初始化的，而且只初始化一次，因此logback.xml中的每个元素，用哪个Action来处理，都已经规定好了。如果在解析logback.xml的过程中，遇到了无法识别的元素，就会抛出异常 

interpretationContext也非常关键，在Action组件实际处理过程中，大部分对象的出栈和入栈，都依赖这个类来实现 

pattern存放了当前正在处理的元素 

actionListStack则是对负责处理某元素的Action进行出栈、进栈的操作 

# Interpreter关键方法解析

介绍完了关键的字段，接下来看方法

```
public void play(List<SaxEvent> eventList) {
    player.play(eventList);
  }
```

这是入口方法，委托EventPlayer来处理SAX流

```
public void play(List<SaxEvent> seList) {
    eventList = seList;
    SaxEvent se;
    for(currentIndex = 0; currentIndex < eventList.size(); currentIndex++) {
      se = eventList.get(currentIndex);

      if(se instanceof StartEvent) {
        interpreter.startElement((StartEvent) se);
        // invoke fireInPlay after startElement processing
        interpreter.getInterpretationContext().fireInPlay(se);
      }
      if(se instanceof BodyEvent) {
        // invoke fireInPlay before  characters processing
        interpreter.getInterpretationContext().fireInPlay(se);
        interpreter.characters((BodyEvent) se);
      }
      if(se instanceof EndEvent) {
        // invoke fireInPlay before endElement processing
        interpreter.getInterpretationContext().fireInPlay(se);
        interpreter.endElement((EndEvent) se);
      }

    }
  }
```

这个方法是对整个logback.xml配置文件的元素来循环遍历，并根据SaxEvent的类型，还给Interpreter类来处理，SaxEvent有3种，基本大同小异，我们用StartEvent来举例
```
public void startElement(StartEvent se) {
    setDocumentLocator(se.getLocator());
    startElement(se.namespaceURI, se.localName, se.qName, se.attributes);
  }
```

这里设置了一下文档的Locator，然后继续调用
```
private void startElement(String namespaceURI, String localName,
      String qName, Attributes atts) {

    String tagName = getTagName(localName, qName);
    pattern.push(tagName);

    if (skip != null) {
      // every startElement pushes an action list
      pushEmptyActionList();
      return;
    }

    List applicableActionList = getApplicableActionList(pattern, atts);
    if (applicableActionList != null) {
      actionListStack.add(applicableActionList);
      callBeginAction(applicableActionList, tagName, atts);
    } else {
      // every startElement pushes an action list
      pushEmptyActionList();
      String errMsg = "no applicable action for [" + tagName
          + "], current pattern is [" + pattern + "]";
      cai.addError(errMsg);
    }
  }
```

这个方法先得到元素的TagName，然后通过RuleStore来匹配一下，得到处理的Action列表，然后调用callBeginAction()方法来处理，如果SaxEvent类型是EndEvent，则这里是调用callEndAction()方法
```
void callBeginAction(List applicableActionList, String tagName,
      Attributes atts) {
    if (applicableActionList == null) {
      return;
    }

    Iterator i = applicableActionList.iterator();
    while (i.hasNext()) {
      Action action = (Action) i.next();
      // now let us invoke the action. We catch and report any eventual
      // exceptions
      try {
        action.begin(interpretationContext, tagName, atts);
      } catch (ActionException e) {
        skip = (Pattern) pattern.clone();
        cai.addError("ActionException in Action for tag [" + tagName + "]", e);
      } catch (RuntimeException e) {
        skip = (Pattern) pattern.clone();
        cai.addError("RuntimeException in Action for tag [" + tagName + "]", e);
      }
    }
  }
```

这里就简单啦，循环遍历负责处理的Action，依次调用其begin()方法。类似的callEndAction()，则会循环遍历负责处理的Action，依次调用其end()方法 

# 小结

总结一下整个配置流程： 

1、首先读取配置文件，如果没有找到配置文件，则用默认配置 

2、如果找到配置文件logback.xml，则调用SAX解析器来解析该配置文件 

3、初始化一个Interpreter对象，调用其play()方法 

4、Interpreter对象委托EventPlayer对象，循环遍历XML文件中的所有节点 

5、根据节点的类型（<element>、</element>、content），调用Interpreter中的startElement()、characters()、endElement()方法 

6、Interpreter委托RuleStore来匹配元素名，选择合适的Action 

7、委托Action组件来进行实际的配置 

本篇博客的内容就结束了，把logback框架是怎么配置的讲了一半，但Action组件是怎么进行实际配置的，暂时没有涉及，下一篇博客就介绍这个方面的内容