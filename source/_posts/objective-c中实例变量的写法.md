title: objective-c中实例变量的写法
date: 2013-12-22 16:13
categories: iOS
---
这几天看了《ios7 programming fundamentals》，对objective-c中的实例变量比较理解了，本文总结一下
<!--more-->

# 没有类变量

objective-c中类定义包括interface块和implementation块，通常情况下，前者放在.h文件，后者放在.m文件。不过这不是必须的，全部写在一个文件里也没有问题，但是分开是推荐做法，也方便其他类import

与java不同，objective-c中没有类变量的说法，只有实例变量。在java里，类变量经常被当做常量来使用：

```
public static String URL = "http://www.xxx.com";
```

在objective-c里，类似的需求可以用define来实现：

```
#define URL @"http://www.xxx.com"
```

但是，这并不是类变量，objective-c里只有实例变量

# 实例变量的标准写法

实例变量声明放在implementation段里，所有方法的前面，一般用{}包起来，比如：

```
@implementation YLSBackupDelegate

{
    NSString *userName;
}
```

类似于java中的
```
private String userName;
```

此时userName变量无法被外部访问，就跟java中的private字段一样，这也符合OO私有变量隐藏的思想。如果希望这个字段暴露出去，就需要为这个字段写access方法：

```
-(NSString*) userName;
-(void) setUserName:(NSString*)name;
```

并且定义到interface段里。这个写法也跟java中类似：

```
public String getUserName(){
    return userName;
}

public void setUserName(String name){
    this.userName = name;
}
```

注意这里有一个重要的区别，objective-c中的getter方法，方法名就是实例变量的名字，而在java bean里，getter method的方法名是getXXX

# @property语法糖

当实例变量（字段）很多时，为每个实例变量写access方法是非常麻烦的事。在java中，没有办法在语法层面减轻这个工作量，但是IDE提供了自动生成access方法的功能

而在objective-c里，则在语言层面提供了语法糖，@property。不过这块糖随着版本变迁，也有几次细微的调整，所以网上搜到的帖子说法也都不太一样，最新的情况是这样的：

## @property写在interface段里

```
@interface YLSMainViewController : CDVViewController

@property YLSBackupDelegate *backupDelegate;
@property YLSResumeDelegate *resumeDelegate;

@end
```

这很合理，因为既然要将实例变量暴露出去，那么即使不用@property，access方法本来也要声明在interface里

## 不需要@synthesize语句

在某个版本之前，对应@property，在implementation中需要写对应的@synthesize语句，来合成access方法，不过在一次升级之后，现在已经不再需要了

## 不需要重复声明实例变量

实际上，@property声明的是属性，并不是实例变量。但是编译器会根据属性，自动生成实例变量，和对应的access方法。所以已经在interface里声明了@property，就不再需要在implementation里再声明实例变量了。如果重复声明，似乎还会报错，有时候引入一些比较老的第三方组件，比如ASIHTTPRequest，还会编译不通过

## 访问属性

访问属性也有语法糖，即.操作符。如果不使用@property，通过access方法访问实例变量，需要用标准的调用方法操作符：

```
[xxx userName];
[xxx setUserName:@"abc"];
```

但是如果用了@property，就可以使用.操作符来存取：
```
xxx.userName = "abc";
NSString *name = xxx.userName;
```

## 自动生成的实例变量命名规则

要记住，属性不是实例变量，而是根据属性会生成实例变量（和对应的access方法）。所以，属性名叫name，而实例变量的名字并不是name。命名规则是在属性前面加下划线

比如

```
@property NSString* name;
```
生成的实例变量名将是

```
_name
```

所以要访问这个实例变量，就有2种方法，或者使用.操作符

```
NSString *n = self.name;
```

这等价于：
```
NSString *n = [self name]
```

或者，直接使用实例变量名：
```
NSString *n = _name;
```