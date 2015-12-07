title: 使用FMDB多线程访问数据库，及database is locked的问题
date: 2014-07-25 18:38
categories: iOS
---
今天终于通过FMDatabaseQueue解决了多线程同时访问数据库时，数据库锁定的问题，错误信息是：Unknown error finalizing or resetting statement (5: database is locked)，本文总结一下：
<!--more-->

# FMDatabase不能多线程使用同一个实例

多线程访问数据库，不能使用同一个FMDatabase的实例，否则会发生异常。如果线程使用单独的FMDatabase实例是允许的，但是同样有可能发生database is locked的问题。这是由于多线程对sqlite的竞争引起的

我的app一开始就是多线程使用单独的FMDatabase实例访问数据库，虽然没有引起crash，但是还是出现了database is locked问题，造成很多数据没有如预期写入数据库

# 使用FMDatabaseQueue，问题依旧

后来上FMDB的官网看了文档，确认用FMDatabaseQueue可以解决这个问题，API也比较简单：

```
NSString *dbFilePath = [PathResolver databaseFilePath];
queue = [FMDatabaseQueue databaseQueueWithPath:dbFilePath];
[queue inDatabase:^(FMDatabase *db){
    // access db
}];
```
但是实际测试了一下，还是database is locked

读了一下相关的源码，FMDatabaseQueue解决这个问题的思路是：创建一个队列，然后将放入队列的block顺序执行，这样避免了多线程同时访问数据库

而我的代码是多线程各创建FMDatabaseQueue的实例，所以其实有多个队列，因此还是存在数据库竞争的问题，和用FMDatabase时是一样的

# 共享同一个FMDatabaseQueue实例

于是接下来我让每个线程使用同一个Queue实例，问题就顺利解决了

实现的方式，一开始我想给FMDatabase增加一个单例方法，但是这样以后升级FMDB会比较麻烦，所以最后我是创建了一个Helper类

```
@implementation LosDatabaseHelper

{
    FMDatabaseQueue* queue;
}

-(id) init
{
    self = [super init];
    if(self){
        NSString *dbFilePath = [PathResolver databaseFilePath];
        queue = [FMDatabaseQueue databaseQueueWithPath:dbFilePath];
    }
    return self;
}

+(LosDatabaseHelper*) sharedInstance
{
    static dispatch_once_t pred = 0;
    __strong static id _sharedObject = nil;
    dispatch_once(&pred, ^{
        _sharedObject = [[self alloc] init];
    });
    return _sharedObject;
}

-(void) inDatabase:(void(^)(FMDatabase*))block
{
    [queue inDatabase:^(FMDatabase *db){
        block(db);
    }];
}

@end
```

系统中其他的类，使用这个Helper类的单例，这样保证了全局只有唯一的FMDatabaseQueue实例。注意，因为Helper内部持有的是FMDatabaseQueue，所以可以这么做，如果包装的是FMDatabase类，就绝对会有问题。因为FMDatabase实例不能在多线程环境共享

# 使用FMDatabaseQueue之后，管理db

原本使用FMDatabase类，需要手工调用db的open和close方法

但是用FMDatabaseQueue，不需要调用open，因为查看代码发现，Queue已经open了。至于要不要close，我也不确定，因为官方的sample code没有调用close。实际应用中，我也没有调用，好像没有问题。如果需要close的话，我想可以在Helper类的公共方法里增加调用close queue就可以了。下面是close的源码：

```
- (void)close {
    FMDBRetain(self);
    dispatch_sync(_queue, ^() { 
        [_db close];
        FMDBRelease(_db);
        _db = 0x00;
    });
    FMDBRelease(self);
}
```

所以，使用Queue，是不需要自己打开和关闭db的。但是如果使用了FMResultSet，rs倒是需要关闭，否则会报warning：

```
if ([db hasOpenResultSets]) {
    NSLog(@"Warning: there is at least one open result set around after performing [FMDatabaseQueue inDatabase:]");
```
为了不看到warning，我都在block里调用了

```
[rs close]
```

# 刷新数据库文件路径

具体到我们的应用，还有一个特殊问题需要考虑。因为我们的APP可以切换账户，而账户的db文件是独立的。所以当用户重新登录的时候，需要刷新一下Helper的queue

```
+(void) refreshDatabaseFile
{
    LosDatabaseHelper *instance = [self sharedInstance];
    [instance doRefresh];
}

-(void) doRefresh
{
    NSString *dbFilePath = [PathResolver databaseFilePath];
    queue = [FMDatabaseQueue databaseQueueWithPath:dbFilePath];
}
```

如果不这么做，由于Helper是单例，那么切换账户以后，用户B访问的还是用户A的数据库。刷新的调用，一般放在登录之后，进入主页面之前就可以了

# 队列和线程

在debug过程中，顺便看到一个现象。虽然多个block都是放到同一个队列里，但是其实是跑在不同的thread里

不要混淆队列和线程的概念，使用GCD时，开发者关注的是把block放到队列中，但是同一个队列其实可以对应多个thread，为block分配thread，是GCD框架负责的，开发者不需要关注。只要把操作放到合适的队列里，GCD就会完成线程的创建，分配与回收