title: 删除NSString的最后一个字符
date: 2013-12-16 21:42
categories: iOS
---
删除NSString的最后一个字符是一个常见需求，特别是在循环拼接字符串的时候，记录一下代码片段
<!--more-->

以下代码可以删除NSString的最后一个字符，可以考虑用category实现
```
+(NSString*) removeLastOneChar:(NSString*)origin
{
    NSString* cutted;
    if([origin length] > 0){
        cutted = [origin substringToIndex:([origin length]-1)];// 去掉最后一个","
    }else{
        cutted = origin;
    }
    return cutted;
}
```