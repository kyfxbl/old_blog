title: 使用事务提升sqlite insert的性能
date: 2014-07-30 11:05
categories: database 
---
昨天发现sqlite插入性能很低，搜索了一下发现，其实sqlite的插入可以做到每秒50000条，但是处理事务的速度慢
<!--more-->

看下面这个官方FAQ的第19条，大体的意思是对于客户端来说，其实sqlite处理insert是足够快的，但是处理事务确实很慢
[sqlite FAQ#19](http://www.sqlite.org/faq.html#q19)

我原本的代码没有使用事务，所以每条insert语句都默认为一个事务。解决的办法是加上事务，效果浮夸，执行SQL的时间从10秒缩短到了0.07秒

发现了这个以后，我就尝试把可能的地方都加上事务，但是原本程序有一处逻辑，是执行一大堆insert，如果主键冲突就自然无视。但是如果把这堆sql变成事务，就会影响正确数据的插入，所以又把insert语句改成insert or ignore：

```
insert or ignore into test (id, key) values (20001, 'kyfxbl');
```

然后再放到一个事务里，效率大大提升