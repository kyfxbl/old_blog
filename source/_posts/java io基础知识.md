title: java io基础知识
date: 2013-09-24 11:15
categories: java
---
复习一下java io的相关基础知识
<!--more-->

# char数组、String、byte数组转换

首先需要清楚java中这3种类型的区别：

byte是字节，byte\[\]是字节数组，是字符在计算机中的实际存储。字节如何转换成字符，要看用什么编码。比如用utf-8编码，3个字节对应1个中文字符 
char是字符，char\[\]是字符数组，其实也就是字符串
String本质上就是char\[\]。char\[\]和String之间的转换，不需要指定编码 

## 从char\[\]转换成String

```
char[] c = new char[] { 0x5c71,0x4456,0x1234 };
String s = new String(c);
```

## 从String转换成char\[\]

```
String s = "这是一个字符串";
char[] c = s.toCharArray();
```

## 从byte\[\]转换成String

```
byte[] b = getBytes();// 某个方法返回了byte[]
String s = new String(b, "UTF-8");
```

## 从String转换成byte\[\]

```
String s = "这个一个字符串";
byte[] b = s.getBytes(Charset.forName("UTF-8"));
```

## byte\[\]和char\[\]转换

byte\[\]和char\[\]之间，不知道用什么方法可以直接转，一般用String作为过渡 

可以用一句话总结：<span style="color: red;">“字符编码就变成字节;字节解码就变成字符”</span> 

# BIO常用API简介

本文不涉及NIO，只总结BIO的常用API 

JAVA IO主要有2套基本的API，一套用于读写字节；另一套用于读写字符，可以认为是前者的快捷方式 

读写字节的基本接口是InputStream和OutputStream，InputStream从输入流中读出字节，读出后需要选择编码方式进行decode;OutputStream往输出流中写入字节，写入前需要选择编码方式进行encode 

读写字符的基本接口是InputStreamReader和OutputStreamWriter，InputStreamReader从输入流中读出字符;OutputStreamWriter往输出流中写入字符。编码方式在创建实例的时候已经指定了 

## 写数据的例子

OutputStream的例子：

```
String file = "/home/xxx/fortest.txt";
String charset = "UTF-8";

FileOutputStream fos = new FileOutputStream(file);
fos.write("這是要保存的中文字符串".getBytes(charset));
fos.close();
```

这里调用了基本的write(byte[])方法，将一个字符串编码后，写到文件里 

用针对字符的OutputStreamWriter的话，可以省去自行编码的步骤：

```
String file = "/home/xxx/fortest.txt";
String charset = "UTF-8";

FileOutputStream fos = new FileOutputStream(file);
OutputStreamWriter writer = new OutputStreamWriter(fos,charset);
writer.write("这是要保存的中文字符串");
writer.close();
```
可以看到，调用write()方法时，不需要再调用getBytes()方法，因为在创建OutputStreamWriter的时候，已经指定了编码 

## 读数据的例子

前面2个是写的例子，比较简单，读要稍微麻烦一点 

下面是InputStream的例子：

```
FileInputStream fis = new FileInputStream(file);

StringBuilder sb = new StringBuilder();
byte[] buffer = new byte[68];// 这行代码有BUG

int count = 0;

while ((count = fis.read(buffer)) != -1) {
    sb.append(new String(buffer, 0, count, charset));
}
fis.close();

System.out.println(sb.toString());
```

这里的read方法是
```
read(byte[] b)
```

会将读取到的内容写到字节数组b里，并返回此次读取的字节数。如果已经读完，则返回-1;此外还有一个重载方法read()，一次只读一个字节，返回的是读取到的字节内容。前者是比较常用的。（所以我觉得JDK这里的设计不太好，重载的方法，返回值的含义却完全不同，很容易造成误导） 

这个例子主要就是从InputStream中循环读取字节，然后decode为String。这里定义了一个count变量，String的构造方法也比较特殊。是因为最后一次读取时，往往不会读到字节数组的容量上限的内容，只有从0到count-1的内容才是最后一次读取的实际字节。所以需要用这个不常用的构造方法，把从count到size-1的部分排除掉 

但是这个方法有一个bug，就是定义byte\[\]那一行。如果定义的byte\[\]容量够大的话，一次性把输入流里的字节全部读出来了，那就没有问题;但是只要输入流一大，一般都会分若干次读取，那么就非常容易截断，造成decode失败。比如aa bb cc dd ee ff，前3个字节aa bb cc可以decode成一个字符，后3个字节dd ee ff可以decode成另一个字符。但是有可能aa bb cc dd先读出来，ee ff在下一个循环才读出来，那decode后就会形成乱码 

所以读字符的话，一般用InputStreamWriter会好一点，可以避免这个问题

```
FileInputStream fis = new FileInputStream(file);
InputStreamReader reader = new InputStreamReader(fis, charset);

StringBuffer sb = new StringBuffer();
char[] buf = new char[64];

int count = 0;

while ((count = reader.read(buf)) != -1) {
    sb.append(buf,0,count);// 截取掉buf中，最后一次循环的冗余部分
    // sb.append(buf);
}

reader.close();
System.out.println(sb.toString());
```

和上面的例子很相像，区别在于定义的是char\[\]数组，而不是byte\[\]数组。在调用InputStreamReader类的read()方法时，已经将字节decode为字符了，所以不会出现InputStream中字节错误截断的问题 

调用StringBuffer的append()方法时，同样要注意截取掉最后一次循环中的冗余数据，这个和上一个例子是一样的