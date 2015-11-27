title: NSDictionary判断key存在，且value非空
date: 2015-03-22 17:12
categories: iOS
---
这几天写一段数据迁移脚本，各种bug和闪退，定位以后发现大部分都是NSDictionary取key引发的错误
<!--more-->

# 判断key存在

第一个场景是判断key是否存在，NSDictionary并没有类似containsKey之类的API，网上找到的判断方法，大部分是

```
if([dict objectForKey:@"xxx"]){
    // key存在
}
```

如果一个@{}不包含某个key，那么调用objectForKey会返回nil，就走不进if的分支

# 判断key对应的value非空

但是这里的NSDictionary是用FMDB返回的结果，可能key是存在的，但是对应的value是null。那么下面的代码：

```
[[dict objectForKey:@"money"] intValue];
```
就会闪退，因为虽然money这个KEY存在，但是对应的value是NSNull。恶心的是，用简单的if方法判断不出value是否是NSNull

```
if([dict objectForKey:@"money"]){
    // logic
}
```

因为此时objectForKey方法返回的不是nil，而是NSNull，而NSNull是可以走进if分支的。所以正确的判断应该是：

```
if(![[dict objectForKey:@"money"] isEqual:[NSNull null]]){
    // logic
}
```

反正我是从来没见过别的语言里有这么恶心的非空判断

# 用category来解决

最后是写了一个NSDictionary的category来解决这个问题，只有当key存在，且key对应的value非空，才返回true

```
// judge nil
if(![dict objectForKey:key]){
    return NO;
}

id obj = [dict objectForKey:key];// judge NSNull

return ![obj isEqual:[NSNull null]];
```