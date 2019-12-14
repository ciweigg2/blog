title: 【Mysql优化】常见业务SQL处理
date: 2019-08-04 16:02:53
tags: [mysql]
categories: [mysql]
---
### 如何对评论进行分页展示

一般情况下都是这样写

```sql
 SELECT customer_id,title,content FROM product_comment WHERE audit_status = 1 AND product_id =199726 LIMIT 0,15;
```

<!--more-->

我们来看看它的执行计划

![](/images/1195739-20190108104003876-1059942822.png)

可以看到possible_keys、key、key_len的值均为NULL，说明这条SQL在product_comment 表上是没有可用的索引的，取出9593行过滤度为1%

建立索引，优化评论分页查询

根据我们索引规范可以考虑在where条件上建立索引

where条件有两个字段，我们可以通过以下语句计算一下两列数据在表中的区分度

计算字段数据区分度，建立索引

```sql
SELECT COUNT(DISTINCT audit_status)/COUNT(*) AS audit_rate,COUNT(DISTINCT product_id)/COUNT(*) AS product_rate FROM product_comment;
```

![](/images/1195739-20190108102138681-339634594.png)

比值越接近1，代表区分度越好，我们应该把区分度好的列放到联合索引的左侧

我们现在建立索引后，再来看看执行计划

![](/images/1195739-20190108104122391-1515856423.png)

可以看到查询时运用到了联合索引，只查询出一条数据，就能返回我们需要的数据了，过滤程度是百分之百，我们完成了第一步优化

数据库的访问开销 = 索引 IO + 索引全部记录结果所对应的一个表数据的 IO

> 缺点

这种SQL语句查询的缺点是，越往后翻页，比如几千页之后，效率会越来越差，查询时间也会越来越长，尤其表数据量大的时候更是如此

> 适用场景

它的适用场景是表的结果集很小，比如一万行以下时，或查询条件非常复杂，比如涉及到多个不同的查询判断，或是表关联时使用

### 进一步优化评论分页查询，SQL语句改写

改写后的SQL语句

```sql
SELECT t.customer_id,t.title,t.content 
FROM (
SELECT customer_id  FROM product_comment WHERE  product_id =199726 AND audit_status = 1 LIMIT 0,15
)a JOIN product_comment t 
ON a.customer_id = t.customer_id;
```

改写前的SQL和改写后的SQL查询出来的结果集是一样的，但是效率要高于改写前的SQL

> 使用前提

使用这个SQL有一个前提是，商品评论表的主键是customer_id ，且是有覆盖索引（也就是刚刚我们建立的联合索引）

> 优化原理

先根据过滤条件利用覆盖索引取出主键的customer_id，然后再进行排序，取出我们所需要的数据的行数，然后再和评论表通过主键进行排序来取出其他的字段，
这种方式的数据开销是索引 IO +索引分页后的结果（15行数据）的表的IO

> 优点

比改写前的SQL在IO上要节省很多，这种改写方式的优点是在每次翻页的所消耗的资源和时间基本是相同的，不会越往后翻页，效率越差

> 应用场景

当查询和排序字段（即where子句和order by子句所涉及的字段），有对应的覆盖索引的情况下使用
并且查询的结果集很大的情况下也是适用于这种情况的