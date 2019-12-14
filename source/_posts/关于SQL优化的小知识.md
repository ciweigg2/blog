cover: http://ciwei2.cn-sh2.ufileos.com/142.jpg
title: 关于SQL优化的小知识
date: 2019-02-21 11:25:54
tags: [sql优化]
categories: [sql优化]
---
### 负向查询不命中索引

<!--more-->

不命中
```java
select account from user where id not in (1,2,3);
```

命中
```java
select account from user where id in (4,5,6);
```

### 前置模糊查询不命中索引

不命中
```java
select account from user where name like '%lufei'
```
命中
```java
select account from user where name like 'lu%fei%'
```

建议可以考虑使用 Lucene 等全文索引工具来代替频繁的模糊查询。

### 数据区分不明显不建议索引
对非唯一的字段，例如“性别”这种大量重复的重复值的字段，增加索引也没有什么意义。可以采用唯一账号等字段。

### 越小越简单的数据类型建议索引
越小越简单的数据类型通常在磁盘、内存中占用少，处理起来更快，例如整型数据比字符处理开销小，因为字符串的比较更复杂，处理非常耗时。

### 尽量避免null
索引字段应该制定列为NOT NULL 。含有空值得列很难进行查询优化，因为他们使得索引、索引的统计信息以及比较运算增加复杂，应该用0或者特殊值、空字符代替。

### 在字段上进行计算不能命中索引
索引列不能参与计算，尽量保持列“干净”。比如，FROM_UNIXTIME(create_time) = ‘2016-06-06’ 就不能命中索引。
不命中

```java
select account from user where FROM_UNIXTIME(create_time) = CURDATE();
```

命中
```java
select account from user where create_time = FROM_UNIXTIME(CURDATE());
```

### 表表连接索引
表与表连接用于多表联合查询的约束条件的字段应当建立索引，并且进行 join 的字段两表的字段类型要相同，不然也不会命中索引。

### 字段类型强制转换不命中索引
不命中
```java
select account from user where phone = 1341111111
```

命中
```java
select account from user where phone = '1341111111'
```

### 如果知道是一条记录，使用limit

```java
select account from user where phone = '1341111111' limit 1
```
可以提高效率，让数据库停止游标移动。

### 最左匹配
最左前缀匹配原则，MySQL会一直向右匹配直到遇到范围查询（>,<,BETWEEN,LIKE）就停止匹配。
如有索引(a, b, c, d)，查询条件a = 1 and b = 2 and c > 3 and d = 4，则会在每个节点依次命中a、b、c，而无法命中d。(很简单：索引命中只能是相等的情况，不能是范围匹配)