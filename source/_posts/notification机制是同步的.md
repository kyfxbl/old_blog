title: notification机制是同步的
date: 2014-02-20 19:45
categories: iOS 
---
与Node.js中的Event机制不同，iOS里的事件机制是同步的
<!--more-->

默认情况下，广播一个通知，会阻塞后面的代码：

```
-(void) clicked
{
    NSNotificationCenter *center =  [NSNotificationCenter defaultCenter];
    [center postNotificationName:@"event_happend" object:self];

    NSLog(@"all handler done");
}
```

按下按钮后，发送一个广播，此前已经注册了2个此事件的侦听者

```
-(id) init
{
    self = [super init];
    if(self){
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(whenReceive:) name:@"event_happend" object:nil];
    }
    return self;
}

-(void) whenReceive:(NSNotification*) notification
{
    NSLog(@"im1111");
}
```

```
-(id) init
{
    self = [super init];
    if(self){
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(whenReceive:) name:@"event_happend" object:nil];
    }
    return self;
}

-(void) whenReceive:(NSNotification*) notification
{
    NSLog(@"im22222");
}
```

执行这段代码，首先会输出im1111，然后是im22222，最后才是all handler done

调试发现，代码始终是跑在同一个线程中（广播事件的线程），广播事件之后的代码被阻塞，直到所有的侦听者都执行完

所以，由于NotificationCenter的这个特性，如果希望广播的事件异步处理，则需要在侦听者的方法里开启新线程

Notification只是组件间解耦的方式，不是天然的异步处理