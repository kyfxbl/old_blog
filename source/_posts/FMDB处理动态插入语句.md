title: FMDB处理动态插入语句
date: 2013-12-18 15:28
categories: iOS 
---
昨天做一个需求，参数的数量不确定，本文总结一下
<!--more-->

无法使用这个API：

```
- (BOOL)executeUpdate:(NSString*)sql, ...
```

但是用
```
- (BOOL)executeUpdate:(NSString*)sql withParameterDictionary:(NSDictionary *)arguments
```

可以很方便地处理这个场景

# 两类insert语句

因为不管使用哪种API，都需要先拼接sql语句，所以先了解一下sql insert语句的写法。有2种，比较常见的是指定列名的insert：

```
insert into table_name (c1,c2,c3) values (v1,v2,v3);
```

如果自己指定了目标列名，那么插入的列的数量可以少于table schema，列的顺序也可以任意指定。第2种是不指定列名的insert：

```
insert into table_name values (v1,v2,v3);
```

这种写法不需要自己指定列名，比较方便，但是<span style="color:#ff0000">values里需要包含所有的列，而且顺序要和table schema一致</span>

# FMDB的2种常用API

我自己比较常用的FMDB API有2个。如果参数的数量是确定的，而且不多，可以用这个API：

```
NSString *sql = @"insert into users values(:id, :name, :age)";
[db executeUpdate:sql, id, name, age];
```

上面的代码用了:key的语法，或者用?，效果也是一样的

```
NSString *sql = @"insert into users values(?,?,?)";
[db executeUpdate:sql, id, name, age];
```

另外一种场景，是参数的数量不确定，或者参数数量非常多，那上面这个API就不太方便，FMDB提供了另一个API，允许传入一个NSDictionary作为参数：

```
NSString *sql = @"insert into users values(:id, :name, :age)";
NSDictionary *dict = [NSDictionary dictionaryWithObjectsAndKeys:@"id123", @"id", @"kyfxbl", @"name", 23, @"age", nil];
[db executeUpdate:sql withParameterDictionary:dict];
```
注意Dictionary里的key需要和sql里的:key一致，FMDB根据key值来决定如何绑定

# 遇到的问题

我昨天就是用上面的第二种API来实现，不过遇到一个错误，总是报id may not be NULL。最后发现原因，是拼的sql采用了不指定列名的方式，类似于：

```
NSString *sql = @"insert into table_name values (:k1,:k2,:k3)";
```

然后刚好NSDictionary里元素的顺序又是错的，第一个元素的值是null，造成id为空，插入失败。其实关键不在于FMDB，而是sql没拼对。主要是我不了解FMDB的这个API，我以为它会自己处理顺序，实际上，<span style="color:#ff0000">FMDB不处理SQL拼接，只处理参数变量替换</span>

有2个办法可以解决这个问题，一个办法是调整Dictionary里元素的顺序，使它和table schema列定义的顺序一致，不过比较困难。另一个办法就是重新拼sql，指定好列名

# 实际的代码

下面是实现的代码，删除了无关代码，以免干扰

```
-(void) handleJsonObject:(NSDictionary*) tableName
{
    NSString *sql = [self assembleInsertSql:tableName];// 拼装sql语句：insert into table_name (c1,c2,c3) values (:a,:b,:c);

    if(!sql){
        return;// 说明data里无数据，不需要操作数据库
    }

    NSString *dbFilePath = [YLSGlobalUtils getDatabaseFilePath];
    FMDatabase *db = [FMDatabase databaseWithPath:dbFilePath];
    [db open];

    NSArray *datas = [tableName objectForKey:@"data"];
    for(NSDictionary *data in datas){
        NSMutableDictionary *mutable = [NSMutableDictionary dictionaryWithDictionary:data];
        [mutable removeObjectForKey:@"_id"];
        BOOL result = [db executeUpdate:sql withParameterDictionary:mutable];
        if(!result){
            NSLog([NSString stringWithFormat:@"%d", [db lastErrorCode]], nil);
            NSLog([db lastErrorMessage], nil);
        }
    }

    [db close];
}

// 返回格式：insert into table_name (c1,c2,c3) values (:a,:b,:c);
-(NSString*) assembleInsertSql:(NSDictionary*) tableName
{
    NSArray *datas = [tableName objectForKey:@"data"];

    if([datas count] == 0){
        return nil;
    }

    NSDictionary *record = [datas objectAtIndex:0];// 取出第一条记录，为了拿到列名
    NSArray *columns = [record allKeys];

    NSString *table = [tableName objectForKey:@"tableName"];
    NSString *prefix = [NSString stringWithFormat:@"insert into %@ (", table];

    NSMutableString *middle = [NSMutableString new];
    for(int i=0;i<[columns count];i++){
        NSString *columnName = [columns objectAtIndex:i];// 列名
        if(![@"_id" isEqualToString:columnName]){
            [middle appendString:columnName];
            [middle appendString:@","];
        }
    }
    NSString* cuttedMiddle = [YLSGlobalUtils removeLastOneChar:middle];

    NSMutableString *suffix = [NSMutableString new];
    [suffix appendString:@") values ("];
    for(int i=0;i<[columns count];i++){
        NSString *columnName = [columns objectAtIndex:i];// 列名
        if(![@"_id" isEqualToString:columnName]){
            [suffix appendString:@":"];
            [suffix appendString:columnName];
            [suffix appendString:@","];
        }
    }
    NSString *cuttedSuffix = [YLSGlobalUtils removeLastOneChar:suffix];

    NSMutableString *sql = [NSMutableString new];
    [sql appendString:prefix];
    [sql appendString:cuttedMiddle];
    [sql appendString:cuttedSuffix];
    [sql appendString:@");"];
    return sql;
}
```

通过上面拼接sql的方法，虽然NSDictionary里的元素顺序不确定，但是列名和values里的顺序总是保证一致的，不会影响最后的插入