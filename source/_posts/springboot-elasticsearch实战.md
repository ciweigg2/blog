cover: http://ciwei2.cn-sh2.ufileos.com/109.jpg
title: springboot-elasticsearch实战
date: 2018-09-02 13:43:25
tags: [elasticsearch,SpringBoot]
categories: [综合]
---
spring-data-elasticsearch 的工程，介绍 Spring Data Elasticsearch 简单的 ES 操作。Spring Data Elasticsearch 可以跟 JPA 进行类比。其使用方法也很简单。
<!--more-->

* springboot版本2.0.4.RELEASE已经支持最新的elasticsearch6.4.0版本了

### 引入maven：
在springboot项目中引入spring-data-elasticsearch：
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

### 配置elasticsearch服务器：
```
spring:
  data:
    elasticsearch:
      cluster-nodes: 118.184.218.184:9300
      cluster-name: docker-cluster
```

### 继承ElasticsearchRepository：
核心接口类继承ElasticsearchRepository：
```
public interface EsBlogRepository extends ElasticsearchRepository<EsBlog, String> {

	/**
	 * <p class="detail">
	 * 功能:
	 * </p>
	 *
	 * @param title    :
	 * @param Summary  :
	 * @param content  :
	 * @param tags     :
	 * @param pageable :
	 * @return page
	 * @author Ciwei
	 * @date 2018.08.21 23:18:00
	 */
	Page<EsBlog> findDistinctEsBlogByTitleContainingOrSummaryContainingOrContentContainingOrTagsContaining(String title, String Summary, String content, String tags, Pageable pageable);

	/**
	 * <p class="detail">
	 * 功能:
	 * </p>
	 *
	 * @param blogId :
	 * @return es blog
	 * @author Ciwei
	 * @date 2018.08.21 23:18:00
	 */
	EsBlog findByBlogId(Long blogId);
}
```

### 自定义方法：

我们就用上面2个自定义方法举例：

#### 根据title Summary content tags pageable(分页信息) OR 条件查询分页的博客信息
```
findDistinctEsBlogByTitleContainingOrSummaryContainingOrContentContainingOrTagsContaining
```

#### 根据blogId查询博客信息
```
findByBlogId
```

#### 原生新增方法(id存在更新，不存在新增)：
```
@Autowired
private EsBlogRepository esBlogRepository;
esBlogRepository.save();
```

### 自定义方法关键字：

![](/images/springdataelasticsearch关键字.png)

### 中文分词

首先需要安装分词，博客中有安装教程

比如我现在有一些数据 字段tags的数据

代码需要添加分词类型：

```java
@Field(type = FieldType.Text,fielddata = true, searchAnalyzer = "ik_max_word", analyzer = "ik_max_word")
private String tags;
```

![](/images/ik分词数据.png)

#### 分词测试：
哈哈我小呀 这条数据分词可以把它拆分为如下结果，说明如下所有的关键词都能搜索到这条记录，包括完整"哈哈我小呀"都能搜到

![](/images/ik分词测试.png)

### 分词测试：

代码中有一个分词测试的接口（搜索关键词就能搜索到数据）：

![](/images/ik分词postman测试.png)

具体业务场景demo：https://github.com/ciweigg2/springboot-elasticsearch6