title: System.getProperty
date: 2013-09-24 11:14
categories: java 
---
最近看tomcat源码，里面有好多System.getProperty()的调用。关于这个方法，以前都没注意过，今天学习了一下
<!--more-->
 
在jdk源码里的注释就说得很清楚了，但我不知道过期没过期，发现jdk里很多注释和代码都不匹配，有的代码更新了，注释还是旧的 

# 汇总

网上也有一些汇总，可读性更好

| *属性* | *含义* | 
| java.version | java版本 |
| java.vendor | java提供商 |
| java.vendor.url | java提供商链接 |
| java.home | java安装目录 |
| java.vm.specification.version | java虚拟机规范版本 |
| java.vm.specification.vendor | java虚拟机规范提供商 |
| java.vm.specification.name | java虚拟机规范名称 |
| java.vm.version | java虚拟机版本 |
| java.vm.vendor | java虚拟机提供商 |
| java.vm.name | java虚拟机名称 |
| java.specification.version | java规范版本 |
| java.specification.vendor | java规范提供商 |
| java.specification.name | java规范名称 |
| java.class.version | java类格式版本号 |
| java.class.path | java类路径 |
| java.library.path | 加载库时搜索的路径列表 |
| java.io.tmpdir | 临时io目录 |
| java.compiler | JIT编译器名称 |
| java.ext.dirs | 扩展目录 |
| os.name | 操作系统名称 |
| os.arch | 操作系统架构 |
| os.version | 操作系统版本 |
| file.separator | 文件分隔符 |
| path.separator | 路径分隔符 |
| line.separator | 换行符 |
| user.name | 用户名 |
| user.home | 用户主目录 |
| user.dir | 用户当前目录 |

# 查看系统变量的代码

```
public static void main(String[] args) {

		String[] keys = new String[] { "java.version", "java.vendor",
				"java.vendor.url", "java.home",
				"java.vm.specification.version",
				"java.vm.specification.vendor", "java.vm.specification.name",
				"java.vm.version", "java.vm.vendor", "java.vm.name",
				"java.specification.version", "java.specification.vendor",
				"java.specification.name", "java.class.version",
				"java.class.path", "java.library.path", "java.io.tmpdir",
				"java.compiler", "java.ext.dirs", "os.name", "os.arch",
				"os.version", "file.separator", "path.separator",
				"line.separator", "user.name", "user.home", "user.dir" };

		int size = keys.length;

		for (int i = 0; i < size; i++) {
			System.out.println(keys[i] + ": " + System.getProperty(keys[i]));
		}

	}
```

# 临时系统变量

除了上述的28个系统变量之外，在启动java程序时，也可以用-D，增加临时的系统变量

```
public static void main(String[] args) {

		String tempProperty = "hero.name";
		System.out.println(System.getProperty(tempProperty));

	}
```
执行效果：

![](http://dl.iteye.com/upload/attachment/0075/8404/0bc2b2b3-619f-3904-94eb-411840b53542.png)

# 区分environment properties

以前一直都混淆了system properties和environment properties

## system properties指的是JVM system 

通过

```
System.getProperty(key);
```
来获取 

而设置的方法主要有3种： 

第一种是JVM内置的，包括java.vm.version等

第二种是在启动时，通过-D参数设置的 

第三种则是通过

```
System.setProperty(key, value);
```
来设置 

## environment properties指的是操作系统 

获取这种参数的方法是

```
System.getenv();
```
在操作系统里设置的环境变量，可以通过这个方法取到 

通过以下代码，可以取到全集

```
Map<String, String> env = System.getenv();

		Set<String> keys = env.keySet();

		for (String key : keys) {
			System.out.println(key);
		}
```