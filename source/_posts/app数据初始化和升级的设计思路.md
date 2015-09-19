title: app数据初始化和升级的设计思路
date: 2014-05-14 23:06
categories: iOS
---
对于把数据保存在本地的APP，会涉及到本地数据库初始化和升级的场景，本文总结一种设计思路
<!--more-->

# 总体思路

一般app启动之后，都有一个初始化的过程。此外后续app升级，还需要考虑数据升级。所以初始化和数据迁移的框架，在初期的版本就要考虑好

总结一下我们的app采取的方案：

1、在持久化的文件夹内（比如UserDefaults或者Documents目录），用一个字段保存老版本号

2、在开始初始化之前，读取老版本号，以及当前版本号

3、如果该应用是第一次加载，那么老版本号就取不到（因为是初次加载，这个字段还没有保存），那么就可以执行初始化过程；如果取到了老版本号，就不执行初始化

4、初始化完成之后，执行数据迁移。因为有老版本号和新版本号，所以可以通过对比，实现增量式的迁移

5、上述动作都完成之后，刷新老版本号

6、下次正常启动，就不会再初始化，也不会执行数据迁移了；如果是安装新版本，由于当前版本号刷新，又会触发数据迁移

# 用户切换账户的场景

上面说的是比较简单的场景。如果应用允许多用户切换账号，而且不同用户的数据是分离的，就更复杂一些

首先标识老版本号的字段不能保存在UserDefaults里，因为UserDefaults是用户共享的。这样当A用户初始化之后，老版本号就存在了。切换到B用户，发现老版本号已存在，则不会执行初始化，其实这时候B用户的数据文件还没有创建好。所以需要把老版本号存在单独的地方，比如每个用户各自的sqlite文件中

然后，读取老版本号的时候，也要根据用户的独立标识去查询

# 改进

目前暂时是把老版本号保存在sqlite里，但是这样首次读取的时候，判断逻辑比较麻烦。需要判断sqlite文件是否存在，然后要判断table有没有，最后才能取值。如果用文本保存可能会稍微方便一点，比存在sqlite里，少了一个判断table是否存在的步骤

# 示例代码

```
-(BOOL) needInit
{
    return [oldVersion isEqual: @"0"];
}

-(NSString*) oldVersion
{
    return oldVersion;
}

-(NSString*) currentVersion
{
    return currentVersion;
}

#pragma mark - private method

-(void) initOldVersion
{
    // 数据库文件不存在，oldVersion设为0
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    if(![fileManager fileExistsAtPath:dbFilePath]){
        oldVersion = @"0";
        return;
    }

    // 数据库文件打开失败，oldVersion设为0
    FMDatabase *db = [FMDatabase databaseWithPath:dbFilePath];
    if(![db open]){
        oldVersion = @"0";
        return;
    }

    // tb_clientstage表不存在，oldVersion设为0
    int tableCount = 0;
    FMResultSet *rs = [db executeQuery:@"select count(*) as count from sqlite_master where type='table' and name='tb_clientstage'"];
    if([rs next]){
        tableCount = [rs intForColumn:@"count"];
    }
    if(tableCount == 0){
        oldVersion = @"0";
        [db close];
        return;
    }

    // 设置oldVersion
    rs = [db executeQuery:@"select * from tb_clientstage where id = '1' and tableno = '0'"];
    if([rs next]){
        oldVersion = [rs stringForColumn:@"version"];
    }else{
        oldVersion = @"0";
    }
    [db close];
}

-(void) initCurrentVersion
{
    NSDictionary* infoDict =[[NSBundle mainBundle] infoDictionary];
    NSString* versionNum =[infoDict objectForKey:@"CFBundleVersion"];
    currentVersion = versionNum;
}
```

然后，是否进行初始化的判断：

```
clientInfo = [YLSClientInfo new];

if([clientInfo needInit]){
    [self createEverythingForFirstTime];// 初始化时才执行
}
[self allTheTime];// 任何时候都执行

[migrationHelper doMigration:clientInfo];
```

增量迁移：

```
-(void) doMigration:(YLSClientInfo*)clientInfo
{
    NSString *oldVersion = [clientInfo oldVersion];
    NSString *currentVersion = [clientInfo currentVersion];

    // 正常登陆，不需要数据迁移
    if([oldVersion isEqualToString:currentVersion]){
        return;
    }

    // 全新安装，非升级，不需要数据迁移
    if([oldVersion isEqualToString:@"0"]){
        return;
    }

    // 以下均是版本升级，需要数据迁移
    if([oldVersion isEqualToString:@"1.0.0"]){
        [script1 doMigration];
        [script2 doMigration];
        [script3 doMigration];
        [script4 doMigration];
        return;
    }

    // 其他的情况
}
```