title: 读logback源码系列文章（七）——配置的实际工作类Action
date: 2013-09-24 10:57
categories: 源码阅读
---
本系列是阅读logback源码的总结。本文介绍Action组件是如何进行配置的实际工作的
<!--more-->

上篇博客介绍了ContextInitializer类如何把框架的配置工作委托给各个Action具体实现类，这篇博客就接下来介绍一下，Action组件是如何进行配置的实际工作的 

# 整体类图

老规矩，先上图 

![](http://dl.iteye.com/upload/attachment/568456/1d89cae7-de54-3b72-851e-098695503af6.jpg)

如图所示，首先Action是一个抽象类，定义了begin()、body()、end()方法，这些方法如上篇博客所说，是给Interpreter调用的，Interpreter的endElement()方法会调用private的callEndAction()方法，然后callEndAction()方法调用实际Action的end()方法 

然后针对GenericConfigurator中addInstanceRules()方法定义的每种元素，比如<appender>、<appender-ref>，都有一个对应的Action，负责对这种元素进行处理 

# Action工作流程

Action组件的类别是很多的，比较常见的有LoggerAction、AppenderAction、AppenderRefAction等，下面就以AppenderAction和AppenderRefAction为例子，来说明Action的工作方式

```
public void begin(InterpretationContext ec, String localName,
      Attributes attributes) throws ActionException {
    // We are just beginning, reset variables
    appender = null;
    inError = false;

    String className = attributes.getValue(CLASS_ATTRIBUTE);
    if (OptionHelper.isEmpty(className)) {
      addError("Missing class name for appender. Near [" + localName
          + "] line " + getLineNumber(ec));
      inError = true;
      return;
    }

    try {
      addInfo("About to instantiate appender of type [" + className + "]");

      appender = (Appender) OptionHelper.instantiateByClassName(className,
          ch.qos.logback.core.Appender.class, context);

      appender.setContext(context);

      String appenderName = ec.subst(attributes.getValue(NAME_ATTRIBUTE));

      if (OptionHelper.isEmpty(appenderName)) {
        addWarn("No appender name given for appender of type " + className
            + "].");
      } else {
        appender.setName(appenderName);
        addInfo("Naming appender as [" + appenderName + "]");
      }

      // The execution context contains a bag which contains the appenders
      // created thus far.
      HashMap<String, Appender> appenderBag = (HashMap) ec.getObjectMap().get(
          ActionConst.APPENDER_BAG);

      // add the appender just created to the appender bag.
      appenderBag.put(appenderName, appender);

      ec.pushObject(appender);
    } catch (Exception oops) {
      inError = true;
      addError("Could not create an Appender of type [" + className + "].",
          oops);
      throw new ActionException(oops);
    }
  }
```

比如配置文件
```
<appender name="ma" class="MyAppender" />
```

当解析到这行时，就会调用AppenderAction的begin()方法，把"MyAppender"这个属性给读出来，然后根据类名创建一个MyAppender的实例

```
public static Object instantiateByClassName(String className,
      Class superClass, Context context) throws IncompatibleClassException,
      DynamicClassLoadingException {
    ClassLoader classLoader = Loader.getClassLoaderOfObject(context);
    return instantiateByClassName(className, superClass, classLoader);
  }
```

```
public static Object instantiateByClassName(String className,
      Class superClass, ClassLoader classLoader)
      throws IncompatibleClassException, DynamicClassLoadingException {

    if (className == null) {
      throw new NullPointerException();
    }

    try {
      Class classObj = null;
      classObj = classLoader.loadClass(className);
      if (!superClass.isAssignableFrom(classObj)) {
        throw new IncompatibleClassException(superClass, classObj);
      }
      return classObj.newInstance();
    } catch (IncompatibleClassException ice) {
      throw ice;
    } catch (Throwable t) {
      throw new DynamicClassLoadingException("Failed to instantiate type "
          + className, t);
    }
  }
```

以上代码其实就是根据"MyAppender"这个类名来创建了一个类实例并返回，这种写法使得创建类实例的动作延迟到框架运行期间，实现动态创建实例，很值得学习 

之后就是把"ma"属性调用setAppenderName()方法，赋给刚创建的这个MyAppender实例 

最后关键性的一步，就是把这个初始化完毕的MyAppender放到InterpretationContext的AppenderBag里，至于有什么用，我们接下来看AppenderRefAction就会明白

```
public void begin(InterpretationContext ec, String tagName, Attributes attributes) {
    // Let us forget about previous errors (in this object)
    inError = false;

    // logger.debug("begin called");

    Object o = ec.peekObject();

    if (!(o instanceof AppenderAttachable)) {
      String errMsg = "Could not find an AppenderAttachable at the top of execution stack. Near ["
          + tagName + "] line " + getLineNumber(ec);
      inError = true;
      addError(errMsg);
      return;
    }

    AppenderAttachable appenderAttachable = (AppenderAttachable) o;

    String appenderName = ec.subst(attributes.getValue(ActionConst.REF_ATTRIBUTE));

    if (OptionHelper.isEmpty(appenderName)) {
      // print a meaningful error message and return
      String errMsg = "Missing appender ref attribute in <appender-ref> tag.";
      inError = true;
      addError(errMsg);

      return;
    }

    HashMap appenderBag = (HashMap) ec.getObjectMap().get(
        ActionConst.APPENDER_BAG);
    Appender appender = (Appender) appenderBag.get(appenderName);

    if (appender == null) {
      String msg = "Could not find an appender named [" + appenderName
          + "]. Did you define it below in the config file?";
      inError = true;
      addError(msg);
      addError("See " + CoreConstants.CODES_URL
          + "#appender_order for more details.");
      return;
    }

    addInfo("Attaching appender named [" + appenderName + "] to "
        + appenderAttachable);
    appenderAttachable.addAppender(appender);
  }
```

代码的含义一目了然，由于<appender-ref>元素肯定是嵌套在<logger>里，所以ec.peekObject()方法取出的就是刚刚创建的Logger实例，接下来就从InterpretationContext的AppenderBag中取出来，然后调用setAppender()方法，把Appender赋给Logger。如果设置了多个<appender-ref>，那么这些Appender都会被赋给Logger 

其他如LoggerAction、RootLoggerAction的代码也是类似的，而且方法体都不大，就不重复叙述了，大家可以自己去看 

阅读了这部分源代码，我觉得颇有体会，主要学习到以下2个设计的思路： 

1、通过在配置文件中指定类名，然后调用ClassLoader的方法，可以在程序运行期间动态地创建类的实例 

2、利用一个Context，来保存配置期间的类实例和变量，把不同的元素给串联起来 

其实看完ContextInitializer类，再来深入地看Action类的实现，是很简单的。但是读这部分代码，却让我觉得心情很愉悦，因为代码中包含的设计思想，不仅仅是用在logback框架中，对我们自己程序的配置怎么写也很有指导意义。掌握了这种设计思路，有助于写出更具灵活性和可扩展性的程序 

本系列博客到本篇为止，已经对logback的整体框架有了一个high level的了解。去配置使用logback框架是毫无困难的了，而且如果日志模块出现了问题，要定位也是非常简单的了。不过接下来我们还要继续探究一下更深层次的代码，下一篇博客介绍一下Appender类中的Encoder，是怎么将ILoggingEvent实际记录成日志的