title: 改造SQLitePlugin插件，解决database locked问题
date: 2014-12-17 20:52
categories: iOS
---
我们的是hybrid应用，基于cordova。做第一个版本的时候，原生代码用FMDB访问数据库，web部分用SQLitePlugin插件。早期原生代码和js访问数据库是错开的，所以没有发生问题。但是到了现在，有些场景需要2边一起访问数据库，由于sqlite不支持并发写，就产生了database locked问题。本文介绍解决的办法
<!--more-->

# 问题背景

SQLite是库级锁，支持并发读，但是不支持并发写。所以如果多个线程同时进行写操作，就有可能造成database locked。如果是纯原生应用，推荐使用FMDatabaseQueue避免锁库

但是如果是hybrid应用，就相对比较复杂，我们的APP就踩坑了。刚开始搭框架的时候，使用cordova搭建了hybrid框架，并使用SQLitePlugin，来支持js访问数据库。然后原生的部分，就用FMDB来访问数据库

一开始的时候，大部分的数据库访问都是在js里调用的，SQLitePlugin已经处理了并发写的场景，所以一直都没有出现过锁库的情况。原生的部分也通过FMDatabaseQueue避免锁库。当时大部分的操作都在js里，如果涉及到原生访问数据库的场景，一般都用模态窗阻止了用户的其他操作，所以2个sqlite的入口没有发生冲突，安稳了很长时间

但是最近需求变得更复杂，出现了js和原生代码同时操作数据库的场景，结果2个队列产生了竞争，database locked的问题又开始概率出现了

所以教训就是：如果是hybrid的应用，最好在一开始的时候就自己写一个cordova plugin，调用数据库访问的公共组件，这样无论是原生的代码，还是js代码，对数据库的操作都会被FMDatabaseQueue统一放在队列里管理，才不会出现多线程并发写的问题

对于我们这种已经踩坑的来说，只能想办法改造SQLitePlugin，因为业务代码已经大量调用了SQLitePlugin的js接口，所以不能动js部分，不然业务代码就都得改。所以最后决定采用的方案，就是把SQLitePlugin的原生部分，重新借助FMDB实现一次。这样应用的原生代码，和cordova插件，都是用FMDatabaseQueue来管理，就不会发生锁库的现象了

# 总体思路

改造的过程难度不大，主要就是2大块：

1、把SQLitePlugin里操作数据库的代码，比如sqlite3_step之类的，用FMDB提供的API替换掉

2、通过DEBUG，弄清楚js部分和原生代码部分的参数和返回值，然后适配接口，保证正确处理js传来的参数，并返回和原来的实现一样的返回值

把所有代码读了一遍，发现其实核心部分是这个方法：

```
-(CDVPluginResult*) executeSqlWithDict: (NSMutableDictionary*)options
```
options就是js传过来的参数，里面有sql，params，query，qid这4个key，所以接下来我们调用FMDB API所需要的参数，也是从其中取出来。而这个方法最后的返回值，必须包含qid，type，result，error，具体来说，qid是原样返回，type可能是success或error，result则必须包含rows，也就是select查询的结果（其实还有rowAffected和其他一些key，但是检查了发现没有用，所以新的实现就忽略了这些key）

# 具体实现

上述的参数和返回值搞清楚以后，剩下的部分就是重新实现。SQLitePlugin原来的代码有1000行左右，所做的也无外乎调用sqlite3的api，还有处理类型转换，参数绑定等常规动作，这些事情在FMDB里都已经做过了，所以借助FMDB重新实现是非常简单的。改造后的代码只有200行不到，核心也是这个方法：

```
-(CDVPluginResult*) executeSqlWithDict: (NSMutableDictionary*)options
{
    NSString *sql = [options objectForKey:@"sql"];
    NSArray *params = [options objectForKey:@"params"];

    NSMutableArray *resultRows = [NSMutableArray array];// 查询结果集
    __block NSDictionary *error = nil;

    [dbHelper doOperation:^(FMDatabase *db){

        if([[sql lowercaseString] hasPrefix:@"select"]){

            FMResultSet *rs = [db executeQuery:sql withArgumentsInArray:params];

            if(!rs){
                error = @{@"code":[NSNumber numberWithInt:[db lastErrorCode]], @"message": [db lastErrorMessage]};
            }else{
                while([rs next]){
                    [resultRows addObject:[rs resultDictionary]];
                }
                [rs close];
            }
        }else{

            BOOL result = [db executeUpdate:sql withArgumentsInArray:params];
            if(!result){
                error = @{@"code":[NSNumber numberWithInt:[db lastErrorCode]], @"message": [db lastErrorMessage]};
            }
        }
    }];

    if(error){
        return [CDVPluginResult resultWithStatus:CDVCommandStatus_ERROR messageAsDictionary:error];
    }

    NSDictionary *cdvResult = @{@"rows": resultRows};
    return [CDVPluginResult resultWithStatus:CDVCommandStatus_OK messageAsDictionary:cdvResult];
}
```
只要熟悉FMDB的API，上面的代码没有什么需要特别说明的

另外，这个plugin持有一个dbHelper的实例变量：

```
{
    YLSDatabaseHelper *dbHelper;
}

-(CDVPlugin*) initWithWebView:(UIWebView*)theWebView
{
    self = (SQLitePlugin*)[super initWithWebView:theWebView];
    if (self) {
        dbHelper = [YLSDatabaseHelper sharedInstance];
    }
    return self;
}
```

这个YLSDatabaseHelper是个单例，原生的代码以及这个插件，都通过这个唯一入口访问数据库，因此保证了不会并发写（FMDatabaseQueue的内部机制）。如果这个dbHelper不是单例，而是有多个实例，其实是无法避免database locked的
```
@implementation YLSDatabaseHelper

{
    FMDatabaseQueue* queue;
}

-(id) init
{
    self = [super init];
    if(self){
        NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
        queue = [FMDatabaseQueue databaseQueueWithPath:dbFilePath];
    }
    return self;
}

+(YLSDatabaseHelper*) sharedInstance
{
    static dispatch_once_t pred = 0;
    __strong static id _sharedObject = nil;
    dispatch_once(&pred, ^{
        _sharedObject = [[self alloc] init];
    });
    return _sharedObject;
}

+(void) refreshDatabaseFile
{
    YLSDatabaseHelper *instance = [self sharedInstance];
    [instance doRefresh];
}

-(void) doRefresh
{
    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    queue = [FMDatabaseQueue databaseQueueWithPath:dbFilePath];
}

-(void) doOperation:(void(^)(FMDatabase*))block
{
    [queue inDatabase:^(FMDatabase *db){
        block(db);
    }];
}

@end
```

这样改造之后，对js部分完全没有影响，WEB里的业务代码一点都不需要修改