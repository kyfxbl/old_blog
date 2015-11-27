title: 优化TableView索引排序的性能
date: 2014-07-24 19:42
categories: iOS
---
app中的一个TableView，使用原生的UILocalizedIndexedCollation进行索引排序（A-Z），实测速度很慢，400多条数据需要3秒多才能显示出来，本文介绍优化的思路
<!--more-->

定位后发现，瓶颈不在数据库访问和UI渲染上，就是索引排序太慢。优化前有性能问题的代码如下：

```
// slow method
-(void) assembleMembers:(NSArray*)origin
{
    [members removeAllObjects];

    UILocalizedIndexedCollation *collation = [UILocalizedIndexedCollation currentCollation];

    // slow point 1: takes 1.5 seconds when 400 records
    for (Member *member in origin) {
        NSInteger sect = [collation sectionForObject:member collationStringSelector:@selector(name)];
        member.sectionNumber = sect;
    }

    NSInteger highSection = [[collation sectionTitles] count];
    NSMutableArray *sectionArrays = [NSMutableArray arrayWithCapacity:highSection];
    for (int i = 0; i < highSection; i++) {
        NSMutableArray *sectionArray = [NSMutableArray arrayWithCapacity:1];
        [sectionArrays addObject:sectionArray];
    }

    for (Member *member in origin) {
        [(NSMutableArray*)[sectionArrays objectAtIndex:member.sectionNumber] addObject:member];
    }

    // slow point 2: takes 1.3 seconds when 400 records
    for (NSMutableArray *sectionArray in sectionArrays) {
        NSArray *sortedSection = [collation sortedArrayFromArray:sectionArray collationStringSelector:@selector(name)];
        [members addObject:sortedSection];
    }
}
```
有2段很慢，第一段是给Member分配sectionNumber，第二段是对26个子数组进行深度排序

发现了瓶颈，就针对瓶颈进行优化

分配sectionNumber的逻辑，放到每次插入数据库时。在表中增加section_number字段，保存这个值。这样每次从数据库取到的数据，就不需要在运行时排序了

深度排序耗时长的问题，改成首字母排序，速度也快了很多

最后的代码如下：

```
-(void) assembleMembers:(NSArray*)origin
{
    [members removeAllObjects];

    UILocalizedIndexedCollation *collation = [UILocalizedIndexedCollation currentCollation];

    NSInteger highSection = [[collation sectionTitles] count];
    NSMutableArray *sectionArrays = [NSMutableArray arrayWithCapacity:highSection];
    for (int i = 0; i < highSection; i++) {
        NSMutableArray *sectionArray = [NSMutableArray arrayWithCapacity:1];
        [sectionArrays addObject:sectionArray];
    }

    for (Member *member in origin) {
        [(NSMutableArray*)[sectionArrays objectAtIndex:member.sectionNumber] addObject:member];
    }

    for (NSMutableArray *sectionArray in sectionArrays) {

        [sectionArray sortUsingComparator:^NSComparisonResult(id obj1, id obj2) {
            Member *m1 = (Member*) obj1;
            Member *m2 = (Member*) obj2;
            return [m2.name localizedCompare:m1.name];
        }];

        [members addObject:sectionArray];
    }
}
```

这样改完之后，400条数据的处理时间缩短到0.3秒，基本可以接受了