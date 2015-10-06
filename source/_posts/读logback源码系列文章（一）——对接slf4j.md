title: 读logback源码系列文章（一）——对接slf4j
date: 2013-09-24 10:34
categories: 源码阅读
---
本系列是阅读logback源码的总结。本文介绍logback如何对接slf4j
<!--more-->

slf4j为各种日志框架提供了一个统一的界面，使用户可以用统一的接口记录日志，但是动态地决定真正的实现框架。logback，log4j，common-logging等框架都实现了这些接口。所以分析logback，首先要从它怎么对接slf4j说起

slf4j框架本身是非常简单的，一共只有3个package，不到30个class。而且其中很多类在读源码的时候是不必深究的

PS：插一句题外话，很多朋友看到一个项目里动辄几十个package，数不清的类，当时就蒙了，然后读了几天就放弃了

这里我说2个建议： 

1、源代码的类虽然很多，但是一般是有一个清晰的结构的（当然有些项目确实结构混乱，东一撮西一撮的，那没关系，不读就是了）。所以在深入到具体的细节之前，应该先把整个代码的结构梳理清楚，再一块一块地往下读。梳理清楚了脉络和结构，可以直接跳过不想深究的细节，只要知道它是干什么的就行了，当用到的时候再来看 

2、源代码的类虽然很多，但是一般会有一个入口。比如hibernate的SessionFactory，spring的ApplicationContext，logback的LoggerFactory。从入口进去，一点点地读，必要的时候使用debug功能跟着走一走，相信会清晰很多 

回到slf4j，slf4j最关键的是2个接口，和一个入口类。搞清楚了这3个，对slf4j就会比较清楚了

最关键的2个接口，分别是Logger和ILoggerFactory。最关键的入口类，是LoggerFactory 

所有的具体实现框架，一定会实现Logger接口和ILoggerFactory接口。前者实际记录日志，后者用来提供Logger，重要性不言自明。而LoggerFactory类，则是前面说过的入口类

```
Logger logger = LoggerFactory.getLogger(Main.class);
```


不管实现框架是什么，要获取Logger对象，都是通过这个类的getLogger()方法，所以这个类也非常重要。具体实现框架和slf4j的对接，就是通过这个类 

![](http://dl.iteye.com/upload/attachment/545181/ab7a9f31-dffe-3bee-ac3e-ba69a9b6d118.jpg)

如图，每个日志框架都需要实现ILoggerFactory接口，来说明自己是怎么提供Logger的。像log4j、logback能够提供父子层级关系的Logger，就是在ILoggerFactory的实现类里实现的。同时，它们也需要实现Logger接口，以完成记录日志。为啥log4j和logback可以一个Logger对应多个Appender？这就要去分析它们的Logger实现类

slf4j自带的NOPLoggerFactory，实现了ILoggerFactory，其getLogger()方法很简单，就是返回一个NOPLogger

```
public class NOPLoggerFactory implements ILoggerFactory {

  public NOPLoggerFactory() {
    // nothing to do
  }

  public Logger getLogger(String name) {
    return NOPLogger.NOP_LOGGER;
  }
}
```

NOPLogger实现了Logger接口，它就更简单，如同它的名字一样：什么都不做

```
/** A NOP implementation. */
  final public void info(String msg) {
    // NOP
  }

  /** A NOP implementation. */
  final  public void info(String format, Object arg1) {
    // NOP
  }

  /** A NOP implementation. */
  final public void info(String format, Object arg1, Object arg2) {
    // NOP
  }
```

下面就是关键的LoggerFactory类了，这个入口类是怎么提供Logger的呢？

```
public static Logger getLogger(String name) {
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    return iLoggerFactory.getLogger(name);
  }
```

首先它调用一个private的getILoggerFactory()方法，获得一个ILoggerFactory的实现类，然后委托这个实现类提供一个Logger实现类。就是这样把获取实际Logger的工作，委托给具体的日志框架上，比如log4j、logback、common-logging等

```
public static ILoggerFactory getILoggerFactory() {
    if (INITIALIZATION_STATE == UNINITIALIZED) {
      INITIALIZATION_STATE = ONGOING_INITILIZATION;
      performInitialization();

    }
    switch (INITIALIZATION_STATE) {
    case SUCCESSFUL_INITILIZATION:
      return StaticLoggerBinder.getSingleton().getLoggerFactory();
    case NOP_FALLBACK_INITILIZATION:
      return NOP_FALLBACK_FACTORY;
    case FAILED_INITILIZATION:
      throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
    case ONGOING_INITILIZATION:
      // support re-entrant behavior.
      // See also http://bugzilla.slf4j.org/show_bug.cgi?id=106
      return TEMP_FACTORY;
    }
    throw new IllegalStateException("Unreachable code");
  }
```

为了获得ILoggerFactory的实现类，它首先判断初始化是否成功，如果没成功，它会委托NOPLoggerFactory来提供Logger，最终结果就是一个没实现的NOPLogger。如果初始化成功了，就是和日志框架对接成功，就委托日志框架的ILoggerFactory来提供Logger的实现

```
private final static void performInitialization() {
    singleImplementationSanityCheck();
    bind();
    if (INITIALIZATION_STATE == SUCCESSFUL_INITILIZATION) {
      versionSanityCheck();

    }
  }
```

在初始化里，它要做3个工作。首先是检查classpath里是否存在多个日志框架，如果有就会抛出警告信息，提示用户只保留一个。然后是关键的bind()方法，绑定实现框架。最后再检查一下版本完整性

```
private final static void bind() {
    try {
      // the next line does the binding
      StaticLoggerBinder.getSingleton();
      INITIALIZATION_STATE = SUCCESSFUL_INITILIZATION;
      emitSubstituteLoggerWarning();
    } catch (NoClassDefFoundError ncde) {
      String msg = ncde.getMessage();
      if (msg != null && msg.indexOf("org/slf4j/impl/StaticLoggerBinder") != -1) {
        INITIALIZATION_STATE = NOP_FALLBACK_INITILIZATION;
        Util
            .report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
        Util.report("Defaulting to no-operation (NOP) logger implementation");
        Util.report("See " + NO_STATICLOGGERBINDER_URL
            + " for further details.");
      } else {
        failedBinding(ncde);
        throw ncde;
      }
    } catch(java.lang.NoSuchMethodError nsme) {
      String msg = nsme.getMessage();
      if (msg != null && msg.indexOf("org.slf4j.impl.StaticLoggerBinder.getSingleton()") != -1) {
        INITIALIZATION_STATE = FAILED_INITILIZATION;
        Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
        Util.report("Your binding is version 1.5.5 or earlier.");
        Util.report("Upgrade your binding to version 1.6.x. or 2.0.x");
      }
      throw nsme;
    } catch (Exception e) {
      failedBinding(e);
      throw new IllegalStateException("Unexpected initialization failure", e);
    }
  }
```

这段代码提到了一个最关键的StaticLoggerBinder类，检查是否有这个类存在，以及这个类有没有getSingleton()方法，如果有，就视为绑定成功。其实这个类还必须有getLoggerFactory()方法，否则虽然绑定成功，但是到了运行期，一样会抛出NoSuchMethodException。我认为这里是slf4j设计不好的地方，应该在bind()方法里，就检查一下StaticLoggerBinder有没有实现getLoggerFactory()方法。 这个StaticLoggerBinder类，就是具体实现框架和slf4j框架对接的接口

![](http://dl.iteye.com/upload/attachment/545186/46dbea34-ed32-3048-8e15-71c989f6c939.jpg)

这就介绍完了logback是怎么和slf4j对接的。不止是logback，任何日志框架，一定都是通过自己的StaticLoggerBinder类，来和slf4j对接。这个类的实现，在不同的框架中不同，比如后面会说到，在logback中，这个类被设计为或者简单的返回一个默认的LoggerContext（LoggerContext是ILoggerFactory在logback中的实现），或者通过ContextSelector（logback特有的）来选择一个LoggerContext并返回

大家可能会有一个疑问：在LoggerFactory类的bind()方法里，依赖了StaticLoggerBinder类，但是slf4j框架里又没有这个类，那么框架一开始是怎么编译通过并发布成jar包的呢？ 

我认为应该是这样的： 

![](http://dl.iteye.com/upload/attachment/545188/17c30081-f661-3f6f-a715-ee1549766878.jpg)

作者一开始的时候，是以类似图里的包结构组织代码的。为了编译通过，作者写了一个StaticLoggerBinder类
```
private final ILoggerFactory loggerFactory;

  private StaticLoggerBinder() {
    loggerFactory = new NOPLoggerFactory();
  }

  public ILoggerFactory getLoggerFactory() {
    return loggerFactory;
  }
```

然后，作者把org.slf4j、org.slf4j.helpers、org.slf4j.spi这3个package发布为slf4j-api.jar。把org.slf4j.impl发布为slf4j-nop.jar 用户实际使用的时候，必须要引入slf4j-api.jar和具体实现框架，比如log4j.jar，以及对接用的slf4j-log4j.jar，不需要引入slf4j-nop.jar 

最后，大家可能还有一个疑问：为什么log4j和logback这么相似，又为什么log4j和logback可以和slf4j结合地这么好呢？这是因为，这3个框架的作者，根本是同一个人。这位大哥应该可以算作日志界的战斗机了