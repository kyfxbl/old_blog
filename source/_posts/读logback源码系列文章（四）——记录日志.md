title: 读logback源码系列文章（四）——记录日志
date: 2013-09-24 10:57
categories: 源码阅读
---
本系列是阅读logback源码的总结。本文介绍logback如何通过Logger这个核心类来记录日志
<!--more-->

上篇博客介绍了LoggerContext怎么生成Logger，Logger是logback的核心类，也是所有日志框架的核心类。这篇博客详细介绍一下Logger的各字段和方法，重点介绍Logger类是怎样记录日志的 

# 整体类图

老规矩，首先看图： 

![](http://dl.iteye.com/upload/attachment/552879/576c3b42-cb9e-3bfb-8b85-1c72650e1485.jpg)

Logger类实现了slf4j框架定义的Logger接口，然后这个类和LoggerContext是互相引用的（因为Logger需要依赖LoggerContext的TurboFilter等组件）。并且Logger实现了AppenderAttachable接口，它实现该接口的方式，是持有AppenderAttachableImpl类，然后委托该类来实现AppenderAttachable接口定义的方法，这里用到了代理模式，是一个比较精巧的设计 

# Logger的字段解析

看到了Logger类的全景图，接下来我们逐一介绍Logger类中的各个字段，最后重点介绍Logger类是怎样记录日志的 

首先看看Logger的字段

```
static int instanceCount = 0;

  /**
   * The name of this logger
   */
  private String name;

  // The assigned levelInt of this logger. Can be null.
  private Level level;

  // The effective levelInt is the assigned levelInt and if null, a levelInt is
  // inherited form a parent.
  private int effectiveLevelInt;

  /**
   * The parent of this category. All categories have at least one ancestor
   * which is the root category.
   */
  private Logger parent;

  /**
   * The children of this logger. A logger may have zero or more children.
   */
  private List<Logger> childrenList;

  /**
   * It is assumed that once the 'aai' variable is set to a non-null value, it
   * will never be reset to null. it is further assumed that only place where
   * the 'aai'ariable is set is within the addAppender method. This method is
   * synchronized on 'this' (Logger) protecting against simultaneous
   * re-configuration of this logger (a very unlikely scenario).
   * 
   * <p>
   * It is further assumed that the AppenderAttachableImpl is responsible for
   * its internal synchronization and thread safety. Thus, we can get away with
   * *not* synchronizing on the 'aai' (check null/ read) because
   * <p>
   * 1) the 'aai' variable is immutable once set to non-null
   * <p>
   * 2) 'aai' is getAndSet only within addAppender which is synchronized
   * <p>
   * 3) all the other methods check whether 'aai' is null
   * <p>
   * 4) AppenderAttachableImpl is thread safe
   */
  private transient AppenderAttachableImpl<ILoggingEvent> aai;
  /**
   * Additivity is set to true by default, that is children inherit the
   * appenders of their ancestors by default. If this variable is set to
   * <code>false</code> then the appenders located in the ancestors of this
   * logger will not be used. However, the children of this logger will inherit
   * its appenders, unless the children have their additivity flag set to
   * <code>false</code> too. See the user manual for more details.
   */
  private boolean additive = true;

  final transient LoggerContext loggerContext;
  // loggerRemoteView cannot be final because it may change as a consequence
  // of changes in LoggerContext
  LoggerRemoteView loggerRemoteView;
```

1、name是Logger的名称 

2、level是该Logger的分配级别，当配置文件中没有配置时，这个分配级别可以为null 

3、effectiveLevelInt是该Logger的生效级别，会从父Logger继承得到 

4、parent和childrenList是这个Logger的父Logger和子Logger，体现了logback的Logger层次结构 

5、aai上面已经说到了，Logger是委托这个类实现AppenderAttachable接口，也是委托这个类来调用Appender组件来实际记录日志，所以这个字段是最关键的 

6、additive是这个类的Appender叠加性，具体看我的另一篇博客。该字段也是在配置文件中配置的，默认为true 

7、loggerRemoteView也是一个VO对象，作用不是很大 

介绍完了字段，可以看到Logger的设计还是相当清晰易懂的

# Logger方法解析

接下来逐一看看Logger中的方法，getter和setter方法就不废话了

```
private final boolean isRootLogger() {
    // only the root logger has a null parent
    return parent == null;
  }
```

这个方法用来判断一个Logger是否是根Logger

```
Logger getChildByName(final String childName) {
    if (childrenList == null) {
      return null;
    } else {
      int len = this.childrenList.size();
      for (int i = 0; i < len; i++) {
        final Logger childLogger_i = (Logger) childrenList.get(i);
        final String childName_i = childLogger_i.getName();

        if (childName.equals(childName_i)) {
          return childLogger_i;
        }
      }
      // no child found
      return null;
    }
  }
```

这个方法用来得到子Logger，是在LoggerContext的getLogger()方法里调用的，在上一篇博客里已经介绍过了

```
public synchronized void setLevel(Level newLevel) {
    if (level == newLevel) {
      // nothing to do;
      return;
    }
    if (newLevel == null && isRootLogger()) {
      throw new IllegalArgumentException(
          "The level of the root logger cannot be set to null");
    }

    level = newLevel;
    if (newLevel == null) {
      effectiveLevelInt = parent.effectiveLevelInt;
    } else {
      effectiveLevelInt = newLevel.levelInt;
    }

    if (childrenList != null) {
      int len = childrenList.size();
      for (int i = 0; i < len; i++) {
        Logger child = (Logger) childrenList.get(i);
        // tell child to handle parent levelInt change
        child.handleParentLevelChange(effectiveLevelInt);
      }
    }
    // inform listeners
    loggerContext.fireOnLevelChange(this, newLevel);
  }

  /**
   * This method is invoked by parent logger to let this logger know that the
   * prent's levelInt changed.
   * 
   * @param newParentLevel
   */
  private synchronized void handleParentLevelChange(int newParentLevelInt) {
    // changes in the parent levelInt affect children only if their levelInt is
    // null
    if (level == null) {
      effectiveLevelInt = newParentLevelInt;

      // propagate the parent levelInt change to this logger's children
      if (childrenList != null) {
        int len = childrenList.size();
        for (int i = 0; i < len; i++) {
          Logger child = (Logger) childrenList.get(i);
          child.handleParentLevelChange(newParentLevelInt);
        }
      }
    }
  }
```

这2个方法是用来改变Logger的生效级别，并且连带改变子Logger的生效级别

```
Logger createChildByName(final String childName) {
    int i_index = getSeparatorIndexOf(childName, this.name.length() + 1);
    if (i_index != -1) {
      throw new IllegalArgumentException("For logger [" + this.name
          + "] child name [" + childName
          + " passed as parameter, may not include '.' after index"
          + (this.name.length() + 1));
    }

    if (childrenList == null) {
      childrenList = new ArrayList<Logger>(DEFAULT_CHILD_ARRAY_SIZE);
    }
    Logger childLogger;
    childLogger = new Logger(childName, this, this.loggerContext);
    childrenList.add(childLogger);
    childLogger.effectiveLevelInt = this.effectiveLevelInt;
    return childLogger;
  }
```

这个方法是用来创建子Logger的，并且会设置Logger的父子关系，也是在LoggerContext的getLogger()方法里调用的 

# Logger记录日志的流程

接下来就重点介绍Logger组件怎么记录日志了，slf4j定义了Logger接口记录日志的方法是info()、warn()、debug()等，这些方法只是入口，logback是这样实现这些方法的
```
public void info(String msg) {
    filterAndLog_0_Or3Plus(FQCN, null, Level.INFO, msg, null, null);
  }
```

当客户端代码调用Logger.info()时，实际上会进入filterAndLog_0_Or3Plus方法，Logger类中还有很多名字很相似的方法，比如filterAndLog_1、filterAndLog_2。据作者自己说，之所以定义这一系列的方法，是为了提高logback的性能

```
/**
   * The next methods are not merged into one because of the time we gain by not
   * creating a new Object[] with the params. This reduces the cost of not
   * logging by about 20 nanoseconds.
   */

  private final void filterAndLog_0_Or3Plus(final String localFQCN,
      final Marker marker, final Level level, final String msg,
      final Object[] params, final Throwable t) {

    final FilterReply decision = loggerContext
        .getTurboFilterChainDecision_0_3OrMore(marker, this, level, msg,
            params, t);

    if (decision == FilterReply.NEUTRAL) {
      if (effectiveLevelInt > level.levelInt) {
        return;
      }
    } else if (decision == FilterReply.DENY) {
      return;
    }

    buildLoggingEventAndAppend(localFQCN, marker, level, msg, params, t);
  }
```

该方法首先要请求TurboFilter来判断是否允许记录这次日志信息。TurboFilter是快速筛选的组件，筛选发生在LoggingEvent创建之前，这种设计也是为了提高性能 

如果经过过滤，确定要记录这条日志信息，则进入buildLoggingEventAndAppend方法
```
private void buildLoggingEventAndAppend(final String localFQCN,
      final Marker marker, final Level level, final String msg,
      final Object[] params, final Throwable t) {
    LoggingEvent le = new LoggingEvent(localFQCN, this, level, msg, t, params);
    le.setMarker(marker);
    callAppenders(le);
  }
```

在这个方法里，首先创建了LoggingEvent对象，然后调用callAppenders()方法，要求该Logger关联的所有Appenders来记录日志 

LoggingEvent对象是承载了日志信息的类，最后输出的日志信息，就来源于这个事件对象
```
/**
   * Invoke all the appenders of this logger.
   * 
   * @param event
   *          The event to log
   */
  public void callAppenders(ILoggingEvent event) {
    int writes = 0;
    for (Logger l = this; l != null; l = l.parent) {
      writes += l.appendLoopOnAppenders(event);
      if (!l.additive) {
        break;
      }
    }
    // No appenders in hierarchy
    if (writes == 0) {
      loggerContext.noAppenderDefinedWarning(this);
    }
  }
```

经过前面的Filter过滤、日志级别匹配、创建LoggerEvent对象，终于进入了记录日志的方法。该方法会调用此Logger关联的所有Appender，而且还会调用所有父Logger关联的Appender，直到遇到父Logger的additive属性设置为false为止，这也是为什么如果子Logger和父Logger都关联了同样的Appender，则日志信息会重复记录的原因 

继续看下来
```
private int appendLoopOnAppenders(ILoggingEvent event) {
    if (aai != null) {
      return aai.appendLoopOnAppenders(event);
    } else {
      return 0;
    }
  }
```

实际上调用的AppenderAttachableImpl的appendLoopOnAppenders()方法
```
/**
   * Call the <code>doAppend</code> method on all attached appenders.
   */
  public int appendLoopOnAppenders(E e) {
    int size = 0;
    r.lock();
    try {
      for (Appender<E> appender : appenderList) {
        appender.doAppend(e);
        size++;
      }
    } finally {
      r.unlock();
    }
    return size;
  }
```

到这里，为了记录一条日志信息，长长的调用链终于告一段落了，通过调用Appender的doAppend(LoggingEvent e)方法，委托Appender来最终记录日志（其实Appender记录日志信息也是委托其他的类来完成的， 在后面的博客中再介绍） 

但是这里没这么简单，AppenderAttachableImpl类为了处理并发情况，是用了读写锁的
```
final private ReadWriteLock rwLock = new ReentrantReadWriteLock();
  private final Lock r = rwLock.readLock();
  private final Lock w = rwLock.writeLock();
```

总结来说，Logger类中定义的字段和方法，是出于以下目的： 

1、持有LoggerContext，是为了使用TurboFilter来进行快速过滤 

2、定义parent和childList，用于实现父子Logger的树形结构 

3、定义createChildByName()、getChildByName()方法，是供LoggerContext创建Logger 

4、定义level、effectiveLevelInt，是为了判定日志级别是否足够 

5、最后，filterAndLog()、buildLoggingEventAndAppend()、callAppenders()、appendLoopOnAppenders()方法，是Logger类的核心方法，一步步地委托AppenderAttachableImpl类来实际记录日志 

下一篇博客，准备介绍Appender组件怎么记录日志