title: tomcat源码阅读（三）——ClassLoader背景知识
date: 2013-09-24 11:25
categories: 源码阅读 
---
本系列是阅读tomcat源码的总结。本文介绍classloader的背景知识，tomcat用到很多ClassLoader相关的代码，如果缺乏这方面的背景知识，阅读源码会遇到很多障碍，所以本文首先总结一下这方面的内容，和tomcat源码的关系不大
<!--more-->

# 标准的ClassLoader体系 

![](http://dl2.iteye.com/upload/attachment/0086/6318/cda2bded-7c0d-372b-907a-330b86416867.png)

## bootstrap 

bootstrap classloader是由JVM启动的，用于加载%JAVA_HOME%/jre/lib/下的JAVA平台自身的类（比如rt.jar中的类等）。这个classloader位于JAVA类加载器链的顶端，是用C/C++开发的，而且JAVA应用中没有任何途径可以获取到这个实例，它是JDK实现的一部分 

## extension 

entension classloader用于加载%JAVA_HOME%/jre/lib/ext/下的类，它的实现类是sun.misc.Launcher$ExtClassLoader，是一个内部类 

基本上，我们开发的JAVA应用都不太需要关注这个类 

## system 

system classloader是jvm启动时，根据classpath参数创建的类加载器（如果没有显式指定classpath，则以当前目录作为classpath）。在普通的JAVA应用中，它是最重要的类加载器，因为我们写的所有类，通常都是由它加载的。这个类加载器的实现类是sun.misc.Launch$AppClassLoader 

用ClassLoader.getSystemClassLoader()，可以得到这个类加载器 

## custom 

一般情况下，对于普通的JAVA应用，ClassLoader体系就到system为止了。平时编程时，甚至都不会感受到classloader的存在 

但是对于其他一些应用，比如web server，插件加载器等，就必须和ClassLoader打交道了。这时候默认的类加载器不能满足需求了（类隔离、运行时加载等需求），需要自定义类加载器，并挂载到ClassLoader链中（默认会挂载到system classloader下面） 

# 双亲委派模型

从上面的图可以看到，classloader链，是一个自上而下的树形结构。一般来说，java中的类加载，是遵循双亲委派模型的，即： 当一个classloader要加载一个类时，首先会委托给它的parent classloader来加载，如果parent找不到，才会自己加载。如果最后也找不到，则会抛出熟悉的ClassNotFoundException 

这个模型，是在最基础的抽象类ClassLoader里确定的：

```
protected synchronized Class<?> loadClass(String name, boolean resolve)
	throws ClassNotFoundException
    {
	// First, check if the class has already been loaded
	Class c = findLoadedClass(name);
	if (c == null) {
	    try {
		if (parent != null) {
		    c = parent.loadClass(name, false);
		} else {
		    c = findBootstrapClassOrNull(name);
		}
	    } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
            if (c == null) {
	        // If still not found, then invoke findClass in order
	        // to find the class.
	        c = findClass(name);
	    }
	}
	if (resolve) {
	    resolveClass(c);
	}
	return c;
    }
```

自定义ClassLoader的时候，一般来说，需要做的并不是覆盖loadClass()方法，这样的话就“破坏”了双亲委派模型；需要做的只是实现findClass()方法即可 

不过，从上面的代码也可以看出，双亲委派模型只是一种“建议”，并没有强制保障的措施。如果自定义的ClassLoader无视此规定，直接自行加载，不将请求委托给parent，当然也是没问题的 

在实际情况中，双亲委派模型被“破坏”也是很常见的。比如在tomcat里，webappx classloader就不会委托给上层的common classloader，而是先委托给system，然后自己加载，最后才委托给common；再比如说在OSGi里，更是有意完全打破了这个规则 

当然，对于普通的JAVA应用开发来说，需要自定义classloader的场景本来就不多，需要去违反双亲委派模型的场景，更是少之又少 

# 自定义ClassLoader

在某些情况下，我们也需要自定义ClassLoader

## 自定义ClassLoader的一般做法 

从上面的代码可以看到，自定义ClassLoader很简单，只要继承抽象类ClassLoader，再实现findClass()方法就可以了 

## 自定义ClassLoader的场景 

事实上，需要实现新的ClassLoader的场景是很少的 

注意：需要增加一个自定义ClassLoader的场景很多；但是，需要自己实现一个新的ClassLoader子类的场景不多。这是两回事，不可混淆 

比如，即使在tomcat里，也没有自行实现新的ClassLoader子类，只是创建了URLClassLoader的实例，作为custom classloader 

## ClassLoader的子类 

在JDK中已经提供了若干ClassLoader的子类，在需要的时候，可以直接创建实例并使用。其中最常用的是URLClassLoader，用于读取一个URL下的资源，从中加载Class

```
@Deprecated
public class StandardClassLoader
    extends URLClassLoader
    implements StandardClassLoaderMBean {

    public StandardClassLoader(URL repositories[]) {
        super(repositories);
    }

    public StandardClassLoader(URL repositories[], ClassLoader parent) {
        super(repositories, parent);
    }

}
```

可以看到，tomcat就是在URLClassLoader的基础上，包装了StandardClassLoader，实际上并没有任何功能上的区别 

## 设置parent 

在抽象类ClassLoader中定义了一个parent字段，保存的是父加载器。但是这个字段是private的，并且没有setter方法 这就意味着只能在构造方法中，一次性地设置parent classloader。如果没有设置的话，则会默认将system classloader设置为parent，这也是在ClassLoader类中确定的：
```
protected ClassLoader(ClassLoader parent) {
        this(checkCreateClassLoader(), parent);
    }

protected ClassLoader() {
        this(checkCreateClassLoader(), getSystemClassLoader());
    }
```

# ClassLoader隐性传递

“隐性传递”这个词是我乱造的，在网上和注释中没有找到合适的描述的词 

试想这样一种场景：在应用中需要加载100个类，其中70个在classpath下，默认由system来加载，这部分不需要额外处理；另外30个类，由自定义classloader加载，比如在tomcat里：
```
Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
Object startupInstance = startupClass.newInstance();
```

如上，org.apache.catalina.startup.Catalina是由自定义类加载器加载的，需要额外编程来处理（如果是system加载的，直接new就可以了） 

如果30个类，都要通过这种方式来加载，就太麻烦了。不过classloader有一个特性，就是“隐性传递”，即： 如果一个ClassA是由某个ClassLoader加载的，那么ClassA中依赖的需要加载的类，默认也会由同一个ClassLoader加载 

这个机制是由JVM保证的，对于程序员来说是透明的 

# current classloader

current classloader是一个很重要的概念

## 定义 

与前面说的extension、system等不同，在运行时并不存在一个实际的“current classloader”，只是一个抽象的概念。指的是一个类“当前的”类加载器。一个对象实例所属的Class，是由哪一个ClassLoader加载的，这个ClassLoader就是这个对象实例的current classloader 

获得的方法是：
```
this.getClass().getClassLoader();
```

## 实例 

current classloader概念的意义，主要在于它会影响Class.forName()方法的表现，贴一段代码进行说明：

```
public class Test {

	public void tryForName() {

		System.out.println("current classloader: "
				+ this.getClass().getClassLoader());

		try {
			Class.forName("net.kyfxbl.test.cl.Target");
                        System.out.println("load class success");
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}
```

这个类调用了Class.forName()方法，试图加载net.kyfxbl.test.cl.Target类（Target是一个空类，作为加载目标，不重要）。这个类在运行时能否加载Target成功，取决于它的current classloader，能不能加载到Target 

首先，将Test和Target打成jar包，放到classpath之外，jar包中内容如下： 

![](http://dl2.iteye.com/upload/attachment/0086/6328/633a03a6-f4aa-3e54-956e-591987666ca2.png)

然后在工程中删除Target类（classpath中加载不到Target了） 

在Main中用system 加载Test，此时Test的current classloader是system，加载Target类失败
```
public static void main(String[] args) {

		Test t = new Test();
		t.tryForName();

	}
```

然后，这次用自定义的classloader来加载

```
public static void main(String[] args) throws Exception {

		ClassLoader cl = createClassLoader();

		Class<?> startupClass = cl.loadClass("net.kyfxbl.test.cl.Test");
		Object startupInstance = startupClass.newInstance();

		String methodName = "tryForName";
		Class<?>[] paramTypes = new Class[0];
		Object[] paramValues = new Object[0];
		Method method = startupInstance.getClass().getMethod(methodName,
				paramTypes);
		method.invoke(startupInstance, paramValues);

	}

	private static ClassLoader createClassLoader() throws MalformedURLException {

		String filePath = "c://hehe.jar";
		File file = new File(filePath);
		URL url = file.toURI().toURL();
		URL[] urls = new URL[] { url };
		ClassLoader myClassLoader = new URLClassLoader(urls);
		return myClassLoader;

	}
```

在想象中，这次Test的current classloader应该变成URLClassLoader，并且加载Target成功。但是还是失败了 

这是因为前面说过的“双亲委派模型”，URLClassLoader的parent是system classloader，由于工程里的Test类没有删除，所以classpath里还是能找到Test类，所以Test类的current classloader依然是system classloader，和第一次一样 

![](http://dl2.iteye.com/upload/attachment/0086/6332/3540c9de-14cc-3dc5-a38e-18c65b50eb9b.png)

接下来把工程里的Test类也删除，这次就成功了 

![](http://dl2.iteye.com/upload/attachment/0086/6334/740a9cca-6bcb-31fc-885a-828e2f814016.png)

## 关于Class.forName() 

前面说的是单个参数的forName()方法，默认使用current ClassLoader 除此之外，Class类还定义了3个参数的forName()方法，方法签名如下：
```
public static Class<?> forName(String name, boolean initialize,
				   ClassLoader loader)
        throws ClassNotFoundException
```

这个方法的最后一个参数，可以传递一个ClassLoader，会用这个ClassLoader进行加载。这个方法很重要 

比如像JNDI，主体类是在JDK包里，由bootstrap加载。而SPI的实现类，则是由厂商提供，一般在classpath里。那么在JNDI的主体类里，要加载SPI的实现类，直接用Class.forName()方法肯定是不行的，这时候就要用到3个参数的Class.forName()方法了 

# ContextClassLoader

另一个重要的概念是ContextClassLoader

## 获取ClassLoader的API 

前面说过，已经有2种方式可以获取到ClassLoader的引用 

一种是ClassLoader.getSystemClassLoader()，获取的是system classloader 

另一种是getClass().getClassLoader()，获取的是current classloader 

这2种API都只能获取classloader，没有办法用来传递 

## 传递ClassLoader 

每一个thread，都有一个contextClassLoader，并且有getter和setter方法，用来在线程之间传递ClassLoader 

有2条默认的规则： 首先，contextClassLoader默认是继承的，在父线程中创建子线程，那么子线程会继承父线程的contextClassLoader 

其次，主线程，也就是执行main()方法的那个线程，默认的contextClassLoader是system classloader 

## 例子 

对上面例子中的Test和Main稍微改一下（Test和Target依然打到jar包里，然后从工程中删除，避免被system classloader加载到）

```
public class Test {

	public void tryForName() {

		System.out.println("current classloader: "
				+ getClass().getClassLoader());
		System.out.println("thread context classloader: "
				+ Thread.currentThread().getContextClassLoader());

		try {
			Class.forName("net.kyfxbl.test.cl.Target");
			System.out.println("load class success");
		} catch (Exception exc) {
			exc.printStackTrace();
		}

	}
}
```

```
public static void main(String[] args) throws Exception {

		ClassLoader cl = createClassLoader();

		Class<?> startupClass = cl.loadClass("net.kyfxbl.test.cl.Test");
		final Object startupInstance = startupClass.newInstance();

		new Thread(new Runnable() {

			@Override
			public void run() {
				String methodName = "tryForName";
				Class<?>[] paramTypes = new Class[0];
				Object[] paramValues = new Object[0];
				try {
					Method method = startupInstance.getClass().getMethod(
							methodName, paramTypes);
					method.invoke(startupInstance, paramValues);
				} catch (Exception exc) {
					exc.printStackTrace();
				}
			}
		}).start();

	}
```
这次的tryForName()方法在一个子线程中被调用，并依次打印出current classloader和contextClassLoader，如图： 
![](http://dl2.iteye.com/upload/attachment/0086/6341/6665a02e-213a-3d92-9ef0-29f29ed49f46.png)

可以看到，子线程继承了父线程的contextClassLoader 

同时可以注意到，contextClassLoader对Class.forName()方法没有影响，contextClassLoader只是起到在线程之间传递ClassLoader的作用 

## 题外话 

从这个例子还可以看出，一个方法在运行时的表现，在编译期是无法确定的 

在运行时的表现，有时候取决于方法所在的类是被哪个ClassLoader加载；有时候取决于是运行在单线程环境下，还是多线程环境下 

这在编译期是不可知的，所以在编程的时候，要考虑运行时的情况。比如所谓“线程安全”的类，并不是说它“一定”会运行在多线程环境下，而是说它“可以”运行在多线程环境下 

# 总结

本文大致总结了ClassLoader的背景知识。掌握了背景，再阅读tomcat的源码，基本就不会遇到ClassLoader方面的困难 

本文介绍了ClassLoader的标准体系、双亲委派模型、自定义ClassLoader的方法、以及current classloader和contextClassLoader的概念。其中最重要的是current classloader和contextClassLoader 

用于获取ClassLoader的API主要有3种： 

```
ClassLoader.getSystemClassLoader(); 
Class.getClassLoader(); 
Thread.getContextClassLoader(); 
```

第一个是静态方法，返回的永远是system classloader，也就是说，没什么用 

后面2个都是实例方法，一个是返回实例所属的类的ClassLoader；另一个返回当前线程的contextClassLoader，具体的结果都要在运行时才能确定 

其中，contextClassLoader可以起到传递ClassLoader的作用，所以特别重要