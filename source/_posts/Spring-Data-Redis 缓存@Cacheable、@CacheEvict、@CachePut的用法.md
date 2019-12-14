cover: http://ciwei2.cn-sh2.ufileos.com/104.jpg
title: Spring-Data-Redis 缓存@Cacheable、@CacheEvict、@CachePut的用法
date: 2018-07-22 14:26:45
tags: [redis]
categories: [redis]
---
 Spring为我们提供了几个注解来支持Spring Cache。
 * @Cacheable 每次查询，将查询结果放入缓存中 相同id，第二次查询时只从缓存中取数据，不从数据库中取，提高数据查询效率
 * @CacheEvict CacheEvict 删除的数据，要将缓存中的信息清除
 * @CachePut 每次都触发真实的方法调用，将执行结果放入缓存中，更新数据库的时候会更新缓存
 <!--more-->

```java
   /**
     * Cacheable 每次查询，将查询结果放入缓存中
     *           相同id，第二次查询时只从缓存中取数据，提高数据查询效率
     *           key = "#id" : 表示在redis中k-v(key:id,value:PageDemoDto)
     * @param id
     * @return
     */
    @Override
    @Cacheable(value = "pageCache",key = "#id")
    public PageDemoDto findById(String id) {
        System.out.println("从关系数据库中查询！");
        return pageDemoDao.findById(id);
    }
```

```java
    /**
     * CacheEvict 删除的数据，要将缓存中的信息清除
     * @param pdd
     */
    @Override
    @Transactional
    @CacheEvict(value = "pageCache",key = "#pdd.id")
    public void deleteData(PageDemoDto pdd) {
        pageDemoDao.deleteById(pdd);
    }
```

```java
    /**
     * CachePut 每次都触发真实的方法调用，将执行结果放入缓存中
     *          update 和 insert 实现方式一样
     * @param pdd
     * @return
     */
    @Override
    @CachePut(value = "pageCache",key = "#pdd.id")
    public PageDemoDto insertData(PageDemoDto pdd) {
//        insert(pdd);
        pageDemoDao.updateData(pdd);
        //插入（或更新）关系数据库后，重新按id查询出来，放入缓存中
        return pageDemoDao.findById(pdd.getId());
    }
```

```java
    /**
     * List<?> 对象不建议纳入缓存管理，
     *         数据精确度降低缓存刷新难度大
     *         key利用配置的生成器来生成
     * @return
     */
    @Override
    @Cacheable(value = "pageCache",keyGenerator = "wiselyKeyGenerator")
    public List<PageDemoDto> findAll() {
        return pageDemoDao.findAll();
    }
```