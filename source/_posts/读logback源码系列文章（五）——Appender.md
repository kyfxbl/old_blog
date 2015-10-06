title: 读logback源码系列文章（五）——Appender
date: 2013-09-24 10:57
categories: 源码阅读
---
本系列是阅读logback源码的总结。本文介绍Appender组件是怎么记录日志的
<!--more-->

上一篇我们说到Logger类的info()方法通过层层调用，最后委托Appender来记录日志，这篇博客我们就接着说一下，Appender组件是怎么记录日志的 

实际上Appender可能是logback框架中最重要的组件之一，虽然Logger是记录日志的接口，但是如果一个Logger没有关联到任何Appender的话，那么这个Logger就无法记录任何信息。此外虽然logback提供了很多扩展点，但是在应用中，我们可能很少会扩展filter，很少扩展layout和encoder，但是我们扩展Appender的机会却是很多的 

# 整体类图

老规矩，首先上图，看一下Appender的大图景，这里要说明的是，实现Appender接口有2个base类，一个是AppenderBase，另一个是UnsynchronizedAppenderBase，这2个类非常接近，80%以上的代码都是相同的。如果我们自己要自定义Appender的话，只要写一个类继承自这2个base类就好 

![](http://dl.iteye.com/upload/attachment/555497/91743e11-7bbb-38d5-b7a0-0501a65968a6.jpg)

首先是有一个Appender接口，然后如上文所说，UnsynchronizedAppenderBase类实现了这个接口，但是它本身是一个抽象类，需要继承它才能得到真正的实现类。Appender接口继承了FilterAttachable接口，而UnsynchronizedAppenderBase类持有一个FilterAttachableImpl类，委托这个类来实现FilterAttachable接口里定义的方法 

然后OutputStreamAppender是继承自UnsynchronizedAppenderBase的Appender实现类，虽然它已经不是抽象类了，但是实际也是不能直接使用的，它的实现类就是最常见的ConsoleAppender和FileAppender 

只要自己使用过logback的朋友都知道，appender元素下面还需要配置encoder元素，这里的Encoder接口就是对应这个encoder元素的，因为其实Appender组件还不是最终实际记录日志信息的组件，它要委托encoder组件来完成LoggingEvent的格式化和记录 

# Appender工作流程

介绍完了大体的结构，我们接下来就看看，从Appender接口的doAppend()方法，是怎么一步步地最终记录日志的 

首先是UnsynchronizedAppenderBase里面的doAppend()方法，它主要是记录了Status状态，然后检查Appender上的Filter是否满足过滤条件，最后再调用实现子类的appender()方法。很眼熟是吗，这里用到了一个设计模式——模板方法

```
public void doAppend(E eventObject) {
    // WARNING: The guard check MUST be the first statement in the
    // doAppend() method.

    // prevent re-entry.
    if (Boolean.TRUE.equals(guard.get())) {
      return;
    }

    try {
      guard.set(Boolean.TRUE);

      if (!this.started) {
        if (statusRepeatCount++ < ALLOWED_REPEATS) {
          addStatus(new WarnStatus(
              "Attempted to append to non started appender [" + name + "].",
              this));
        }
        return;
      }

      if (getFilterChainDecision(eventObject) == FilterReply.DENY) {
        return;
      }

      // ok, we now invoke derived class' implementation of append
      this.append(eventObject);

    } catch (Exception e) {
      if (exceptionCount++ < ALLOWED_REPEATS) {
        addError("Appender [" + name + "] failed to append.", e);
      }
    } finally {
      guard.set(Boolean.FALSE);
    }
  }

  abstract protected void append(E eventObject);
```

上面的代码非常简单，就不用说了，我们就直接看看实现类的append()方法是怎么实现的，这里我们选择OutputStreamAppender实现类

```
@Override
  protected void append(E eventObject) {
    if (!isStarted()) {
      return;
    }

    subAppend(eventObject);
  }
```

首先检查一下这个Appender是否已经启动，如果没启动就直接返回，如果已经启动，则又进入一个subAppend()方法

```
/**
   * Actual writing occurs here.
   * <p>
   * Most subclasses of <code>WriterAppender</code> will need to override this
   * method.
   * 
   * @since 0.9.0
   */
  protected void subAppend(E event) {
    if (!isStarted()) {
      return;
    }
    try {
      // this step avoids LBCLASSIC-139
      if (event instanceof DeferredProcessingAware) {
        ((DeferredProcessingAware) event).prepareForDeferredProcessing();
      }
      // the synchronization prevents the OutputStream from being closed while we
      // are writing. It also prevents multiple thread from entering the same
      // converter. Converters assume that they are in a synchronized block.
      synchronized (lock) {
        writeOut(event);
      }
    } catch (IOException ioe) {
      // as soon as an exception occurs, move to non-started state
      // and add a single ErrorStatus to the SM.
      this.started = false;
      addStatus(new ErrorStatus("IO failure in appender", this, ioe));
    }
  }
```

这个方法居然什么事也不干。。做了一些检查以后，又进入writeOut()方法。。。
```
protected void writeOut(E event) throws IOException {
    this.encoder.doEncode(event);
  }
```

writeOut()方法委托配置给它的Encoder组件来记录
```
public void doEncode(E event) throws IOException {
    String txt = layout.doLayout(event);
    outputStream.write(convertToBytes(txt));
    outputStream.flush();
  }
```

到这里，终于完了。Encoder组件又委托其Layout组件来将LoggingEvent进行格式化，返回一个String，然后通过OutputStream.write()方法，把格式化之后的日志信息写到目的地 

耐心看到这里的朋友，可能已经有点被绕晕了，怎么Appender需要这么麻烦吗？其实我们这里说的只是ConsoleAppender的doAppend()全流程，并不是所有Appender都这么复杂的，当然也有一些更复杂的。。 

下面看一个简单的Appender，就是我自己写的MyAppender
```
public class MyAppender extends AppenderBase<LoggingEvent> {

	@Override
	protected void append(LoggingEvent eventObject) {
		System.out.println(eventObject.getMessage());
	}
}
```

好吧，非常简单是不是，如果把这个Appender配置到logback.xml中，那么当Logger.info()调用的时候，就会先走进AppenderBase类的doAppend()方法里，进行Filter校验等等，然后进入MyAppender的append()方法，不做其他的操作，直接把message给打印到Console上。当然，由于这个类是极度简化的，没有Encoder和Layout，也就没办法控制输出日志的时间，也没有办法对%thread等标记进行解析处理了。但是这个类可能可以很清晰地表达出，Appender组件是怎么工作的：由AppenderBase类来调用Filter链，然后由Appender实现类来委托Encoder解析LoggingEvent，再输出到目的地 

这篇博客到这里就结束了。到目前为止，5篇博客是这样的： 

1、首先介绍logback怎么和slf4j对接 

2、然后介绍logback的LoggerFactory，也就是LoggerContext是怎么创建的 

3、接下来介绍LoggerFactory怎么创建Logger 

4、然后是Logger怎么记录日志，这其中涉及了级联调用Appender，和调用TurboFilter来过滤的问题 

5、本篇博客又以最常见的ConsoleAppender为例子，介绍了Appender组件怎么把日志信息输出到目的地