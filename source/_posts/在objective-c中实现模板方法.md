title: 在objective-c中实现模板方法
date: 2013-12-02 12:49
categories: iOS 
---
模板方法，是我原本在java开发中用得比较多的一种设计模式。最近在做IOS开发，也遇到一个很合适的场景，但是我不知道怎么在Objective-C中实现，因为没有abstract关键字，在网上搜索了一番，最后在stackoverflow中找到一种办法，本文记录一下
<!--more-->

首先是一个protocol，相当于java里的interface：
```
// 初始化脚本协议
@protocol YLSInitialScript <NSObject>

- (void) doInit:(YLSClientInfo*) clientInfo;

@end
```

然后定义一个抽象类，实现这个接口的总体框架性算法，但是具体的实现声明为抽象方法：

```
@interface YLSInitialScriptTemplate : NSObject<YLSInitialScript>

-(id) initOrigin:(YLSInitialOperator *)operator;

// 抽象方法，由子类实现
- (void) createEverythingForFirstTime;
- (void) update;
- (NSString*) stepMsg;

@end
```

```
@implementation YLSInitialScriptTemplate

YLSInitialOperator *origin;

-(id) initOrigin:(YLSInitialOperator *)operator
{
    origin = operator;
    return self;
}

- (void) doInit:(YLSClientInfo*) clientInfo
{
    if ([clientInfo shouldInit]) {
        [self createEverythingForFirstTime];// 无表，初始化
    } else if ([clientInfo shouldUpdate]) {
        [self update];// 升级
    }
    [origin notifyStepDone:[self stepMsg]];// 通知Bootstrap View Controller刷新进度条
}

// 以下3个是抽象方法，延迟到子类实现
- (void) createEverythingForFirstTime
{
    [self doesNotRecognizeSelector:_cmd];
}

- (void) update
{
    [self doesNotRecognizeSelector:_cmd];
}

- (NSString*) stepMsg
{
    [self doesNotRecognizeSelector:_cmd];
    return nil;
}

@end
```

最后是具体的子类，不需要重新实现协议里规定的doInit()方法，只要实现抽象类里的3个抽象方法就可以了：

```
@interface YLSServiceDataInitScript : YLSInitialScriptTemplate

@end
```

```
@implementation YLSServiceDataInitScript

- (void) createEverythingForFirstTime
{
    // 具体逻辑
}

- (void) update
{
}

- (NSString*) stepMsg
{
    // 具体逻辑
}

@end
```

语法没有java里这么清楚，关键就是在抽象类里用

```
[self doesNotRecognizeSelector:_cmd];
```
这行代码实现类似java中abstract关键字的效果

最后是实现调用的客户端代码：

```
        scripts = [NSMutableArray new];

        // 需要执行的脚本依次添加在下面
        [scripts addObject:[[YLSShowDataInitScript new] initOrigin:self]];
        [scripts addObject:[[YLSServiceDataInitScript new] initOrigin:self]];
        [scripts addObject:[[YLSMemberDataInitScript new] initOrigin:self]];
        [scripts addObject:[[YLSBillDataInitScript new] initOrigin:self]];
        [scripts addObject:[[YLSEmployeeDataInitScript new] initOrigin:self]];
        [scripts addObject:[[YLSBackupDataInitScript new] initOrigin:self]];

for (int i = 0; i < [scripts count]; i++) {
        [[scripts objectAtIndex: i] doInit:clientInfo];
    }
```
stackoverflow原文见：

[template method pattern in Objective-C](http://stackoverflow.com/questions/8146439/objective-c-template-methods-pattern)

[abstract class in Objective-C](http://stackoverflow.com/questions/1034373/creating-an-abstract-class-in-objective-c)