title: 定位HttpClient超时问题
date: 2013-09-24 11:06
categories: java 
---
使用HttpClient超时问题的定位过程
<!--more-->

# 新版API设置超时时间的方法

HttpClient目前最新的版本是httpcomponents-client-4.2.1，是基于httpcomponents-core-4.2.1的，该库在版本升级过程中，发生过比较大的变动。之前这个库叫做HttpClient，现在统称为HttpComponents，拆分成了client和core，应该是重新写过。因为API的变化比较大，以前的方法，在这个新的版本里不好使了

查了一下，发现超时包括ConnectionTimeout和SocketTimeout，这点和老版本一样，分别用于设置建立HTTP连接超时的时间，以及从响应中读取数据超时的时间。用以下2个参数设置：

```
CoreConnectionPNames.CONNECTION_TIMEOUT
CoreConnectionPNames.SO_TIMEOUT
```

变化比较大的，是设置的方法，不再提供setTimeout()之类的方法。这个版本里所有的配置，都是通过HttpParams这个组件来配置的，然后可以附着在HttpClient上，或者HttpRequest上。在HttpRequest上的设置，优先于HttpClient的全局设置 

除了HttpClient和HttpRequest的HttpParams之外，还有另外2个HttpParams，最终都是聚合在ClientParamsStack里

```
public class ClientParamsStack extends AbstractHttpParams {

    /** The application parameter collection, or <code>null</code>. */
    protected final HttpParams applicationParams;

    /** The client parameter collection, or <code>null</code>. */
    protected final HttpParams clientParams;

    /** The request parameter collection, or <code>null</code>. */
    protected final HttpParams requestParams;

    /** The override parameter collection, or <code>null</code>. */
    protected final HttpParams overrideParams;
```

一起提供运行时参数，有先后顺序

```
public Object getParameter(String name) {
        if (name == null) {
            throw new IllegalArgumentException
                ("Parameter name must not be null.");
        }

        Object result = null;

        if (overrideParams != null) {
            result = overrideParams.getParameter(name);
        }
        if ((result == null) && (requestParams != null)) {
            result = requestParams.getParameter(name);
        }
        if ((result == null) && (clientParams != null)) {
            result = clientParams.getParameter(name);
        }
        if ((result == null) && (applicationParams != null)) {
            result = applicationParams.getParameter(name);
        }
        return result;
    }
```

可以看到，requestParams是优先于clientParams的

这段代码也有一点可以借鉴一下，当我们自己写代码，可能重复设置某一属性时，可以是将优先级高的属性放在前面，跳过后面的设置。也可以将优先级高的属性放在后面，覆盖掉前面的设置。这里用的是第一种方法，而且避免对属性的重复设置 

因此在这个版本的HttpClient里，设置超时时间的方法，是这样的：

```
HttpGet httpget = new HttpGet("http://www.baidu.com");
httpget.getParams().setParameter(CoreConnectionPNames.CONNECTION_TIMEOUT, 5000);
httpget.getParams().setParameter(CoreConnectionPNames.SO_TIMEOUT, 3000);
```

将建立HTTP连接的超时时间设置为5秒，读取响应的超时时间设置为3秒 

# 超时时间double的bug

但是实际运行后，我发现我设置的HTTP超时时间是5秒，但是实际上10秒才会超时。如果设置超时时间是2秒，就实际上4秒才会超时。实际超时时间，是我设置的2倍，为了搞清楚是怎么回事，只好打了断点跟踪进去，果然发现了问题 

在DefaultClientConnectionOperator中有一行这样的代码：

```
InetAddress[] addresses = resolveHostname(target.getHostName());
```

这句执行之后，addresses的结果是： 

```
[www.baidu.com/61.135.169.125, www.baidu.com/61.135.169.105]
```

原来通过DNS查询，我输入的域名www.baidu.com，返回了2个地址。 然后接下来的代码，会对2个地址都尝试建立连接，每次尝试的超时时间都是5秒，当所有连接都失败之后，才会抛出ConnectTimeoutException。所以实际超时时间，就成了我设置的2倍 

此外，我的方法其实设置了HttpRequestInterceptor，但是实际并没有执行，因为请求拦截器的调用，是发生在建立连接成功之后的。这里在建立连接时就超时了，所以HttpRequestInterceptor并没有执行的机会 

这个问题至此就定位完成了，可以总结几点： 

1、CoreConnectionPNames.CONNECTION_TIMEOUT定义的是建立HTTP连接超时的时间，CoreConnectionPNames.SO_TIMEOUT定义的是读取response超时的时间 

2、设置是通过HttpParams来完成的，HttpRequest的设置，优先于HttpClient的设置 

3、URI如果设置为域名，那可能会解析出多个地址，那么实际超时时间，就会是设置超时时间的若干倍 

4、HttpRequestInterceptor的调用，是发生在连接建立成功之后的 

# debug JDK

在定位这个问题的时候，我发现InetAddress这个类，我断点跟不进去，这个类位于rt.jar的java.net包里，按理说是应该能跟进去的

检查了一下，发现是eclipse的设置有错，Preferences-->Java-->Installed JREs里，我设置成了jre的路径，实际上要设置成jdk的路径 

![](http://dl.iteye.com/upload/attachment/0071/6873/cb007616-89ef-3583-95e9-f40a7a317360.png)

这样就可以跟进InetAddress这个类了，不过发现方法参数的名字都是看不见的，临时变量也看不见 

![](http://dl.iteye.com/upload/attachment/0071/6871/f170cf4b-6e14-3682-a95c-3e00df18d70f.png)

这还debug个鬼啊，表示很不开心，就查了一下资料

[为什么debug时看不到变量的值](http://hllvm.group.iteye.com/group/topic/25798) 

原来product版的JDK，都是非debug版本的，最终生成的.class，没有包含LocalVariableTable信息，看一下jdk的编译脚本：

```
# Any debug build should include all debug info inside the classfiles  
ifeq ($(VARIANT), DBG)  
  DEBUG_CLASSFILES = true  
endif  
ifeq ($(DEBUG_CLASSFILES),true)  
  JAVACFLAGS_COMMON += -g  
endif  

JAVACFLAGS     = $(JAVACFLAGS_COMMON) $(OTHER_JAVACFLAGS)  
JAVAC_CMD      = $(JAVAC) $(JIT_OPTION) $(JAVACFLAGS) $(LANGUAGE_VERSION) $(CLASS_VERSION) 
```

如果是product版的jdk，编译语句里是没有加-g参数的，所以编译出来的.class文件，只有LineNumberTable，没有LocalVariableTable，所以在debug的时候，就看不到本地变量了（包括方法参数，和临时变量） 

那要解决的方法，要么自己编译一份-g的jdk，要么直接下载debug版的jdk。不过公司没有编译jdk的环境，所以我就去下了一份debug版jdk。这样debug都可以一路跟到底了，只有native method没有办法

[debug jdk的下载地址](http://jdk6.java.net/download.html) 

总结来说，.java在编译的时候，需要加上-g参数，编译出的.class才会包含LocalVariableTable信息，以进行调试 

引申一下，我们平时用Eclipse写的代码，可以直接调试，是因为Eclipse默认会让编译器带上调试信息，设置的地方在Preferences-->Java-->Compiler 

![](http://dl.iteye.com/upload/attachment/0071/6869/78d95378-69db-3840-83e8-e5590614e51f.png)

所以，用Eclipse写的代码，默认就是可以直接调试的。Eclipse不是用javac来编译java源码的，而是用自己的编译器，ECJ，这个编译器也支持-g参数