title: 在objective-c中模拟静态类变量
date: 2013-12-02 20:05
categories: iOS 
---
最近在写的代码，在oc中模拟了静态类变量
<!--more-->

在类UserData中保存运行时的必要信息，比如当前登录用户的id，所属的企业id，token是否失效等。这些信息都写在UserDefaults里，不缓存，每次需要时重新从UserDefaults里加载

然后客户端代码并不直接操作UserDefaults，而是通过一个辅助类UserDataUtil，得到UserData对象。UserData的组装是由UserDataUtil来完成的。其实也可以考虑把UserData和UserDataUtil合并成一个类，这样当UserData增加字段时，就不需要在两处修改了，似乎也不错，不过这次没有这么做

代码中有一个简单的需求，就是需要一系列静态类变量，来保存所有UserDefaults的KEY，这在JAVA中非常容易实现：

```
public class UserDataUtil {

	public static final String ENTERPRISE_ID_KEY = "enterprise_id";

}
```

不过在objective-c里，似乎没有静态类变量这个概念。。最后写了一个类似的类，来实现这种效果：

```
@interface YLSUserDataUtil : NSObject

+(YLSUserData*) readUserData;

+(NSString*) ID;
+(NSString*) USER_ID;
+(NSString*) ENTERPRISE_ID;;

@end
```
上面是.h文件，定义了一组静态方法。下面是.m文件：

```
static NSString* ID = @"id";
static NSString* USER_ID = @"userId";
static NSString* ENTERPRISE_ID = @"enterpriseId";

@implementation YLSUserDataUtil

+(NSString*) ID
{
    return ID;
}

+(NSString*) USER_ID
{
    return USER_ID;
}

+(NSString*) ENTERPRISE_ID
{
    return ENTERPRISE_ID;
}

+ (YLSUserData*) readUserData
{
    // 从UserDefaults中加载数据，组装UserData并返回
}

@end
```

最后是使用这些静态变量的客户端代码：

```
NSString *key = [YLSUserDataUtil ENTERPRISE_ID];
```