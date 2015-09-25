title: classloader简介
date: 2013-09-24 11:14
categories: java 
---
java classloader简介
<!--more-->

# 基本classloader体系 

默认有3个classloader，分别是bootstrap、extension、app(system) 

bootstrap是在JVM启动时加载的，会读取$JAVA_HOME/lib下的class 

extension会读取$JAVA_HOME/lib/ext下的class 

app，也称为system，加载应用程序所需的class，是由classpath变量指定的，如果在运行时不加-classpath参数，则默认会以当前目录为根路径 

这3个classpath存在父子关系，遵循基本的委托模型，即当classloader被要求加载一个类时，它会首先委托parent classloader进行加载，如果没有找到，才会自己负责加载。这个设计主要是从安全性方面考虑 

# classloader相关API 

如果只是做应用层面的开发的话，可能都不会涉及到classloader的知识，只要知道上述的基本模型就可以了。但是如果要理解一些更复杂的东西，比如JNDI，或者开发应用服务器，或者实现一个插件体系的框架，那就需要对classloader的知识有更深入的了解 

classloader存在的目的就是为了实现对class的加载。可以认为有3种classloader 

第一种是整个应用总的classloader，可以通过以下API得到这个classloader实例

```
ClassLoader.getSystemClassLoader();
```

sun.misc.Launcher$AppClassLoader@addbf1 

这个API我感觉作用并不大，因为用到这个API的场景并不多，如果是开发一个简单的应用，那么根本就不关注classloader是什么。如果是开发比较复杂的应用的话，得到系统总的classloader作用也不大，因为本来就是要替换它 

第二种是当前的classloader。这里首先要明白一点。就是类A依赖类B的话，那么加载类B的classloader，也就是加载类A的classloader。比如说

```
ClassLoader appClassLoader = BindCase.class.getClassLoader();
```

等价于
```
ClassLoader appClassLoader = bindCase.getClass().getClassLoader();
```

都是得到加载BindCase这个类的classloader实例，如果bindCase实例又依赖了另一个类ClassB，那么默认情况下，ClassB也会由同一个classloader负责加载 

当使用Class.forName()方法时，如果不显式传递一个classloader对象，则默认是使用当前的classloader。比如说
```
MyClassLoader mcl  = new MyClassLoader();// 该ClassLoader可以读取外部类
Class.forName("xxx.xxx.xxx");
```

这段代码会抛ClassNotFoundException，虽然mcl可以加载外部类，但是它并不是“当前classloader”，当执行Class.forName()时，用的还是当前的classloader，所以外部类是加载不到的 

第三种是当前线程的classloader。这里也有一个背景，每个线程都有一个ContextClassLoader，开启子线程之后，子线程的ContextClassLoader就是父线程的ContextClassLoader 

以下两个API可以分别用来获取和设置ContextClassLoader
```
thread.getContextClassLoader();
thread.setContextClassLoader();
```

这2个API非常重要，因为可以起到传递classloader的作用，很多自定义的classloader体系，都用到了这2个API 

# 实例测试代码 

这段代码是模拟tomcat的加载过程 

Bootstrap类在app classloader中加载，Catalina类在$CATALINA_HOME/lib/catalina.jar下，通过自定义classloader动态加载

![](http://dl.iteye.com/upload/attachment/0076/1828/5f5a3434-38bb-3be5-944b-944a6627e82f.png)

```
public class Bootstrap {

	public static void main(String[] args) throws MalformedURLException,
			ClassNotFoundException {

		String FQCN = "org.apache.catalina.startup.Catalina";// 要加载的目标类

		try {
			Class.forName(FQCN);// it's NOT OK
		} catch (Exception e) {
			e.printStackTrace();
		}

		try {
			ClassLoader.getSystemClassLoader().loadClass(FQCN);// it's NOT OK
		} catch (Exception e) {
			e.printStackTrace();
		}

		try {
			Bootstrap.class.getClassLoader().loadClass(FQCN);// it's NOT OK
		} catch (Exception e) {
			e.printStackTrace();
		}

		try {
			Thread currentThread = Thread.currentThread();
			currentThread.getContextClassLoader().loadClass(FQCN);// it's NOT OK
		} catch (Exception e) {
			e.printStackTrace();
		}

		ClassLoader myClassLoader = createClassLoader();

		myClassLoader.loadClass(FQCN);// it's OK

		System.out.println("over");

	}

	/**
	 * 实例化一个ClassLoader，加载了C盘根目录下的dummy_catalina.jar，其中包含Catalina类的定义
	 */
	private static ClassLoader createClassLoader() throws MalformedURLException {
		String filePath = "c://dummy_catalina.jar";
		File file = new File(filePath);
		URL url = file.toURI().toURL();
		URL[] urls = new URL[] { url };
		ClassLoader myClassLoader = new URLClassLoader(urls);
		return myClassLoader;
	}

}
```