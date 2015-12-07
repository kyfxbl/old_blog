title: android的Log组件和logcat命令
date: 2013-09-24 10:58
categories: android  
---
项目需要选型一个日志框架，原本设想的方案是导入slf4j+logback，但是dalvik转编译没成功。后来想了一个办法，就是用一个后台的Service，把Log组件记录的日志信息给捞出来，写到文件里，本文总结一下
<!--more-->

# Log接口

首先android提供的日志框架的界面很简单，就是一个Log类，里面的方法包括i()、w()、d()什么的，和log4j、logback等常见的日志框架都差不多，这不过Log类做了一些精简，去掉了配置日志格式、路径、父子关系等 

只有2个方法需要重点说明一下： 

一个是getStackTraceString(Throwable tr)，这个方法是传进去一个异常对象，返回一个String的字符串。其实这个方法一般不会直接用到 

另一个是w(String tag,String msg,Throwable tr)，这个方法比一般的w(String tag,String msg)多传了一个Throwable类型的参数，内部会自动调用上面说的getStackTraceString()方法，把message和异常栈都记录下来。所以在记录异常日志的时候，这个方法是比较好用的 

# logcat命令

logcat工具可以通过命令行管理日志

```
adb logcat 
```

这行命令会打印日志信息，并且是阻塞的，有新的日志信息，都会立刻在屏幕上打印出来 

```
adb logcat -d 
```

这行命令也是打印日志信息，不同在于，加上-d参数会退回到命令行模式，不阻塞 

```
adb logcat -g 
```

这行命令是显示日志缓冲区的大小和位置 

```
adb logcat -c
```

这行命令是清除日志 

```
adb logcat -v [brief,process,tag,thread,raw,time,threadtime,long]
```

这行命令是设置日志的显示格式，一般比较常用的是adb logcat -v time，因为会显示时间 

```
adb logcat -d -v time -s [TAG]:[Level]
```

这行命令是关键了，举个实际例子

```
adb logcat -d -v time -s MainActivity:I
```
 
这个命令会对日志进行过滤，只显示出级别是I以上，并且TAG标签是MainActivity的日志 

最后项目记录日志的方式，就采用Log组件进行记录，然后通过logcat命令，把日志信息捞到文件里并保存 

# logging包

Log是在android.util.log包里，原本以为android就只提供了这个日志框架。可是刚刚突然又发现，还有一个包java.util.logging，里面也提供了另外一种日志框架，而且似乎比较强大，可以通过FileHandler类，直接把日志信息记录到文件里（Handler类似于log4j和logback中appender的组件）