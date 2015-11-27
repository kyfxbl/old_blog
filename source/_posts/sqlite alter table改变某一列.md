title: sqlite alter table改变某一列
date: 2015-02-05 20:02
categories: database 
---
sqlite不支持alter table的时候修改某列的定义，只能全表DML
<!--more-->

所以如果需要改变某一列，做法是：

1、先建一张临时表，把原来表中的数据复制进去

2、删除旧表

3、新增表

4、从临时表中把数据复制回新表

5、删除临时表

```
NSString *sql1 = @"create table tb_users_temp as select * from tb_users";
NSString *sql2 = @"drop table tb_users;";
NSString *sql3 = @"CREATE TABLE IF NOT EXISTS tb_users (...);";
NSString *sql4 = @"insert into tb_users select * from tb_users_temp;";
NSString *sql5 = @"drop table tb_users_temp";
```