cover: http://ciwei2.cn-sh2.ufileos.com/111.jpg
title: Springboot使用elasticsearch-analysis-pinyin中文分词
date: 2018-11-10 16:21:08
tags: [elasticsearch,elasticsearch-analysis-pinyin]
categories: [综合]
---
### 介绍

* 这个拼音分析插件用于在汉字和拼音之间进行转换

* 比如 数据库有一条记录是"卡莎"

* 使用 ks kas 卡 等接近的单词 都能搜索出来

<!--more-->

![](/images/20181110162524.png)

```java
github:https://github.com/medcl/elasticsearch-analysis-pinyin
```

### 安装

对应elasticsearch版本 这边是6.4.2

```java
cd /usr/local/elasticsearch-6.4.2/plugins
wget https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.4.2/elasticsearch-analysis-pinyin-6.4.2.zip

解压：
unzip elasticsearch-analysis-pinyin-6.4.2.zip -d pinyin

重启elasticsearch6.4.2
cd /usr/local/elasticsearch-6.4.2
kill -9 pid
bin/elasticsearch

代码中配置：@Field(type = FieldType.text, searchAnalyzer="pinyin_analyzer",analyzer="pinyin_analyzer")

使用blog索引的数据库：
使用postman添加pinyin分词
PUT http://118.184.218.184:9200/blog
{
    "index" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                    }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_first_letter":true,
                    "keep_separate_first_letter" : true,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
    }
}

启动项目：
项目中配置的tags分词字段会自动创建并添加拼音分词滴功能

使用springboot的测试项目 添加数据后 测试：
Demo:https://github.com/ciweigg2/springboot-elasticsearch6
新增接口：localhost:8080/blogs/saveOrUpdate?blogId=3&title=测试&summary=哈哈&content=测试咯&readSize=12&commentSize=13&voteSize=2&tags=卡莎
查询接口：localhost:8080/blogs/ik?tags=ks
```