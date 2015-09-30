title: iOS7里2个未文档化的API
date: 2014-02-28 11:30
categories: iOS
---
这几天看Pushing the Limits，看到2个未文档化的API（非私有API），还挺方便的，本文记录一下。补充说明：虽然并不是私有API，但是如果运气不好的话，似乎用未文档化的API，也会导致上架被拒。所以还是要谨慎使用
<!--more-->

# NSURLComponents

可以从URL中解析出schema，host等

```
NSURL *url = [NSURL URLWithString:@"http://www.yilos.com:5000/svc/graph?name='kyfxbl'"];

NSURLComponents *components = [NSURLComponents componentsWithURL:url resolvingAgainstBaseURL:YES];

NSLog(@"%@", components.host);// www.yilos.com
NSLog(@"%@", components.port);// 5000
NSLog(@"%@", components.scheme);// http
NSLog(@"%@", components.path);// /svc/graph
NSLog(@"%@", components.query);// name='kyfxbl'
```

# array firstObject

一般取数组的第一个对象，习惯这样写：

```
[@[] objectAtIndex:0];
```

这样有个问题，如果是空数组，会抛出异常。ios一直有lastObject方法，好像从7.0开始，终于有了firstObject方法

```
[@[@"1", @"2", @"3"] firstObject];
```

效果和objectAtIndex:0一样，但是在空数组上调用也是安全的，会返回nil，不会抛出异常