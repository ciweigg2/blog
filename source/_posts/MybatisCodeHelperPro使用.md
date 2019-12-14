cover: http://ciwei2.cn-sh2.ufileos.com/83.jpg
title: MybatisCodeHelperPro使用
date: 2018-10-20 10:07:48
tags: [mybatis]
categories: [综合]
---
> MybatisCodeHelperPro是个很强大的mybatis扩展插件 方便开发

### 功能
* mybatis接口和xml的互相跳转 支持一个mybatis接口对应多个xml
* mybatis接口中的方法名重构支持
* xml中的 param的自动提示 if test的自动提示 resultMap refid 等的自动提示
* resultMap中的property的自动提示
* xml中refid，resultMap等的跳转到定义
* 通过方法名(不需要方法的返回值和参数 会自动推导出来)来生成sql 可以生成大部分单表操作的sql 只需要一个方法的名字即可 会自动补全好方法的参数和返回值 和springdatajpa的语句基本一致
* 从java类生成mybatis crud代码 建表语句 支持生成service，建表支持生成多字段的索引
* 添加一个数据库 从数据库生成crud代码 支持mysql oracle sqlserver postgresql
* 直接从Intellij自带的数据库生成crud代码
* 检测没有使用的xml 可一键删除
* 检测mybatis接口中方法是否有实现，没有则报红 可创建一个空的xml
* 检测resultmap的property是否有误
* mybatis接口中一键添加param注解
* mybatis接口一键生成xml
* 支持spring 将mapper注入到spring中 intellij的spring注入不再报错 支持springboot
* 一键生成mybatis接口的testcase 无需启动spring，复杂sql可进行快速测试
<!--more-->

### 介绍

原生mybatis的所有功能都有 在此基础上添加了更多方法

### 文档

* 开发者文档地址：https://gejun123456.github.io/MyBatisCodeHelper-Pro/#/README
* qq群：527290836

### 插件激活地址
http://brucege.com/pay/getfreetrial

### 激活

激活码激活

未激活时IDEA打开项目时会有一个通知 在IDEA的eventLog可以找到 上面有一个一个register的按钮 点击按钮 输入激活码即可 

![](/images/激活mybatispro.gif)

### 解绑

> 购买后会发两个激活码 可以在两台设备上绑定 一个激活码绑定一个设备 解绑可进行下图的操作 (1.8.3版本支持) 

![](/images/解绑mybatispro.png)

### 插件下载&安装

> 使用 IDE 内置插件系统:

> Preferences(Settings) > Plugins > Browse repositories... > 搜索并找到"MybatisCodeHelper-Pro" > Install Plugin

> 下面demo中也有安装包 可能不是最新的 直接下载 本地安装

### 使用idea自带的datasource生成mybatis

![](/images/添加serialUID.gif)

### 根据mapper生成testcase(接口测试)

test目录下面要有resources目录

![](/images/generateTestCaseByClick.gif)

> 生成后的测试类 默认不提交事务 可以看日志是否更新成功 查询了几条数据

> 如果要提交事务 需要这样改 builder.openSession(true) 添加true 默认是false不提交的

```java
    @BeforeClass
    public static void setUpMybatisDatabase() {
        SqlSessionFactory builder = new SqlSessionFactoryBuilder().build(TTestMapperTest.class.getClassLoader().getResourceAsStream("mybatisTestConfiguration/TTestMapperTestConfiguration.xml"));
        //you can use builder.openSession(true) to commit to database
        mapper = builder.getConfiguration().getMapper(TTestMapper.class, builder.openSession(true));
    }
```

### 生成删除mapper

![](/images/deleteAutoCompletionExample.gif)

### 根据类生成mapper.xml中的对象映射

![](/images/generateUnUsedProperties.gif)

### 生成的mapper添加非空判断

![](/images/ifTestOnStringEmpty.gif)

### insertList添加指定字段sql语句

![](/images/insertListUseGeneratedKeys.gif)

### 在实现类生成分页方法

![](/images/usePageInfoInstead.gif)

生成对象的分页

![](/images/support_find_by_all.gif)

### 生成的mapper永远不会覆盖之前自定义的方法

不管你新增的方法是追加在最后还是中间 都不会覆盖

### 方法名的规则 和jpa几乎是一样的

数据库对象User

字段名  | 类型
-----   | ------
id      | Integer
userName | String
password | String

表名为user

xml中对应的resultMap为

	<resultMap id="AllCoumnMap" type="com.codehelper.domain.User">
	    <result column="id" property="id"/>
	    <result column="user_name" property="userName"/>
	    <result column="password" property="password"/>
	</resultMap>


以下是方法名与sql的对应关系(方法名的大小写无所谓)


可以跟在字段后面的比较符有

比较符  | 生成sql | 开始支持的版本号
------- | -------- |---------
between |  prop > {} and prop <{}  |v1.3
betweenOrEqualto | prop >={} and prop <={} | v1.3
lessThan  | prop < {} | v1.3
lessThanOrEqualto | prop <={}  |v1.3
greaterThan | prop > {} |v1.3
greaterThanOrEqualto | prop >={} | v1.3
isnull | prop is null |v1.3
notnull | prop is not null |v1.3
like   | prop like {} |v1.3
in     | prop in {} |v1.3
notin  | prop not in {} |v1.3
not    | prop != {} |v1.3
notlike | prop not like {} |v1.3
startingWith | prop like {}% |v1.6.0
endingWith | prop like %{} |v1.6.0
containing | prop like %{}% |v1.6.0



- find方法

支持获取多字段，by后面可以设置多个字段的条件 一个字段后面只能跟一个比较符
支持orderBy,distinct, findFirst

方法名       |  sql
-----------  |  --------------
find         | select * from user
findUserName | select user_name from user
findById	| select * from user where id = {}
findByIdGreaterThanAndUserName | select * from user where id > {} and user_name = {}
findByIdGreaterThanOrIdLessThan | select * from user where id > {} or id < {}
findByIdLessThanAndUserNameIn  | select * from user where id < {} and user_name in {}
findByUserNameAndPassword      | select * from user where user_name = {} and password = {}
findUserNameOrderByIdDesc   | select user_name from user order by id desc
findDistinctUserNameByIdBetween | select distinct(user_name) from user where id >= {} and id <={}
findOneById	| select * from user where id = {}
findFirstByIdGreaterThan | select * from user where id > {} limit 1
findFirst20ByIdLessThan  | select * from user where id < {} limit 20
findFirst10ByIdGreaterThanOrderByUserName  | select * from user where id > {} order by user_nam user_name limit 10
findMaxIdByUserNameGreaterThan | select max(user where user_name > {}
findMaxIdAndMinId   | select max(id) as maxId, min(id) as minId from user


- update方法 by后面设置的条件同上

方法名     | sql
---------- |  -------
updateUserNameById | update user set user_name = {} where id = {}
updateUserNameAndPasswordByIdIn  | update user set user_name = {} and password = {} where id in {}

- delete方法
by后面设置的条件同上

方法名  |  sql
------- | ---------
deleteById | delete from user where id = {}
deleteByUserNameIsNull  | delete from user where user_name is null

- count方法
by后面设置的条件同上 支持distinct

方法名  | sql
------- | ----------
count   | select count(1) from user
countDistinctUserNameByIdGreaterThan | select count(distinct(user_name)) from user where id > {}


## 注意的点  

- 使用方法名生成sql 需要在接口中提供一个insert或save或add方法并以表对应的java对象为第一参数 (类似于 insert(User user) 需要通过User来进行方法名的推断)
- 使用方法名生成的sql的字段会从数据库对象对应的resultMap中的数据库字段来设置。

- "please check with your resultMap dose it contain all the property of 
此时可以检查这个接口对应的对象 比如 这个接口有个 insert(User user) 即 User对象
是否有一个对应的完整的resultMap在xml中，resultMap中少了属性的话 无法生成 可以将属性设置为transient类型

- 当生成sql时 如果比如UserMapper对应的User对象中含有List或Set类型的属性时，sql会无法生成
请将这些属性设置为transient类型  比如 private transient List<Comment>

demo地址：https://github.com/ciweigg2/springboot-mybatispro