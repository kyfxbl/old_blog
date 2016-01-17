title: linux下资源的路径
date: 2013-09-24 11:15
categories: java 
---
为了给自己添堵，我把一台笔记本的开发环境装成linux了，希望督促自己熟悉linux。可是我刚才发现在linux下，用java加载资源文件和在windows下还不一样，正好总结一下
<!--more-->

# windows和linux的区别

在windows环境，假设在eclipse里建一个工程helloworld，我一般会把配置文件，比如spring.xml、logback.xml什么的都放在src目录下面，然后用下面的代码就能加载到

```
File file = new File("spring.xml");
```
可是在linux环境，用上面的路径是加载不到的，必须要把文件放在helloworld目录下才可以 

针对这个问题，上网搜索了好久，这里总结一下。定位资源有2种方式，一种是file system，另一种是classpath

# file system定位

这种比较简单，有2种情况

```
File file = new File("abc.txt");

File file = new File("/abc.txt");
```

第一种是相对路径；第二种是绝对路径 

绝对路径比较简单，在linux下就是/目录。但是绝对路径虽然很方便定位路径，但是完全不实用，因为基本上换个部署环境，绝对路径定位就不好使了 

相对路径是从“工作目录”算起，所谓的“工作目录”，就是“执行java命令的目录” 

比如说例子里的这个代码，如果在eclipse里执行的话，eclipse会在项目的根目录helloworld下执行java命令，所以这里的file路径就是/home/username/workspace/helloworld/abc.txt 

如果在命令行里自己随便找个目录去执行，比如在/opt下执行java -cp /home/username/workspace/helloworld/bin xxx.xxx.xxx，那这个file的路径就变成了/opt/abc.txt 

这个相对路径也不太靠谱，因为会随着执行java命令的目录不同而改变 

# classpath定位 

个人感觉，classpath定位就要好多了。因为配置文件往往是和代码一起部署的，相对位置一般也比较确定，不会因为安装目录变化，操作系统变化就改变。所以根据classpath来定位资源，我觉得是比较好的，下面举个例子 

## 路径

classpath是/home/username/workspace/helloworld/bin/classes 

有一个类的FQCN是net.kyfxbl.test.Main 

在/home/username/workspace/helloworld/bin/classes下有一个111.test文件 

在/home/username/workspace/helloworld/bin/classes/net/kyfxbl/test下有一个222.test文件 

现在通过classpath定位的方式，在Main这个类中把111.test和222.test给定位到。不管这个项目部署在哪里，只要上面说的目录结构不变就不影响 

## 加载111.test 

111.test是在classpath的根目录下，有2种办法都能加载到它

```
Main.class.getResourceAsStream("/111.test");
```

或者

```
Main.class.getClassLoader().getResourceAsStream("111.test");
```

Main.class.getResourceAsStream()方法是从这个类的包路径开始加载，而111.test是在classpath根路径下，所以要用这个方法加载的话，就需要在111.test前面加上"/" 

Main.class.getClassLoader().getResourceAsStream()方法是从classpath根路径开始加载，所以直接写"111.test"就行了 

## 加载222.test

222.test在Main类的包路径下，也有2种方法能加载它

```
Main.class.getResourceAsStream("222.test");
```

或者

```
Main.class.getClassLoader().getResourceAsStream("net/kyfxbl/test/222.test");
```

和上面的例子正好是相反的，就不重复解释了 

# 总结

1、加载文件有file system定位和classpath定位2种方式，我觉得classpath定位是更好的 

2、不管哪种定位方式，其实都有绝对路径和相对路径的区别。只是classpath定位方式，根目录是确定的，就是classpath；而file system定位方式，根目录不确定，这也是最大的问题 

# 几个关键的概念

以下是几个关键的概念

## classpath

类加载的根目录，一般就是project/bin，或者project/target/classes等。在tomcat里是WEB-INF/classes，和WEB-INF/lib 

## 包路径

就是classpath之下的子目录，比如一个abc.def.Main的类，其包路径是classpath/abc/def 

## 工作目录

这个是给file system用的，就是执行java命令的目录，跟实现有关。但是在eclipse里，一般就是project根目录 

## 绝对路径和相对路径

在classpath定位和file system定位中都有的概念，以"/"打头的是绝对路径，会从根目录开始找；不以"/"打头的就是相对路径，会从当前目录开始找