cover: http://ciwei2.cn-sh2.ufileos.com/118.jpg
title: SpringBoot整合Mybatis-Plus
date: 2018-07-26 17:26:34
tags: [mybatis]
categories: [综合]
---
SpringBoot整合Mybatis-plus
<!--more-->
新建一个项目。pom文件中加入Mybatis依赖，完整pom如下：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dalaoyang</groupId>
    <artifactId>springboot_mybatisplus</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot_mybatisplus</name>
    <description>springboot_mybatisplus</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatisplus-spring-boot-starter</artifactId>
            <version>1.0.5</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

配置文件配置数据库配置和对应Mybatis-Plus实体信息，配置如下：

```java
##端口号
server.port=8888

##数据库url
spring.datasource.url=jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false
##数据库用户名
spring.datasource.username=root
##数据库密码
spring.datasource.password=root
##数据库驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver


##日志级别
logging.level.com.dalaoyang.dao.UserMapper=debug
##mybatis-plus mapper xml 文件地址
mybatis-plus.mapper-locations=classpath*:mapper/*Mapper.xml
##mybatis-plus type-aliases 文件地址
mybatis-plus.type-aliases-package=com.dalaoyang.entity
```

实体类User代码如下：
```java
package com.dalaoyang.entity;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.entity
 * @email yangyang@dalaoyang.cn
 * @date 2018/7/20
 */
public class User {
    private int id;
    private String user_name;
    private String user_password;

    public User() {
    }

    public User(String user_name, String user_password) {
        this.user_name = user_name;
        this.user_password = user_password;
    }

    public User(int id, String user_name, String user_password) {
        this.id = id;
        this.user_name = user_name;
        this.user_password = user_password;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUser_name() {
        return user_name;
    }

    public void setUser_name(String user_name) {
        this.user_name = user_name;
    }

    public String getUser_password() {
        return user_password;
    }

    public void setUser_password(String user_password) {
        this.user_password = user_password;
    }
}
```

下面要说的都是需要注意的地方，新增一个MybatisPlus配置类，其中没有做过多的设置，只是设置了一下方言，代码如下：
```java
package com.dalaoyang.config;

import com.baomidou.mybatisplus.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.config
 * @email yangyang@dalaoyang.cn
 * @date 2018/7/20
 */
@Configuration
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor(){
        PaginationInterceptor page = new PaginationInterceptor();
        //设置方言类型
        page.setDialectType("mysql");
        return page;
    }
}
UserMapper继承了MybatisPlus的BaseMapper，这里面列举一个普通的查询方法getUserList，完整代码如下：

package com.dalaoyang.dao;

import com.baomidou.mybatisplus.mapper.BaseMapper;
import com.dalaoyang.entity.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.dao
 * @email yangyang@dalaoyang.cn
 * @date 2018/7/20
 */
@Mapper
public interface UserMapper extends BaseMapper<User> {
    List<User> getUserList();

}
```

新建一个UserMapper.xml，里面写getUserList对应sql，代码如下：
```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.dalaoyang.dao.UserMapper">
    <resultMap id="user" type="com.dalaoyang.entity.User"/>
    <parameterMap id="user" type="com.dalaoyang.entity.User"/>

    <select id="getUserList" resultMap="user">
        SELECT  * FROM USER
    </select>
</mapper>
```

最后和往常一样，新建一个Controller进行测试，完整代码如下：
```java
package com.dalaoyang.controller;

import com.baomidou.mybatisplus.mapper.EntityWrapper;
import com.baomidou.mybatisplus.plugins.Page;
import com.dalaoyang.dao.UserMapper;
import com.dalaoyang.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.controller
 * @email yangyang@dalaoyang.cn
 * @date 2018/7/20
 */
@RestController
public class UserController {

    @Autowired
    private UserMapper userDao;

    //http://localhost:8888/getUserList
    @GetMapping("getUserList")
    public List<User> getUserList(){
        return userDao.getUserList();
    }

    //http://localhost:8888/getUserListByName?userName=xiaoli
    //条件查询
    @GetMapping("getUserListByName")
    public List<User> getUserListByName(String userName)
    {
        Map map = new HashMap();
        map.put("user_name", userName);
        return userDao.selectByMap(map);
    }

    //http://localhost:8888/saveUser?userName=xiaoli&userPassword=111
    //保存用户
    @GetMapping("saveUser")
    public String saveUser(String userName,String userPassword)
    {
        User user = new User(userName,userPassword);
        Integer index = userDao.insert(user);
        if(index>0){
            return "新增用户成功。";
        }else{
            return "新增用户失败。";
        }
    }

    //http://localhost:8888/updateUser?id=5&userName=xiaoli&userPassword=111
    //修改用户
    @GetMapping("updateUser")
    public String updateUser(Integer id,String userName,String userPassword)
    {
        User user = new User(id,userName,userPassword);
        Integer index = userDao.updateById(user);
        if(index>0){
            return "修改用户成功，影响行数"+index+"行。";
        }else{
            return "修改用户失败，影响行数"+index+"行。";
        }
    }


    //http://localhost:8888/getUserById?userId=1
    //根据Id查询User
    @GetMapping("getUserById")
    public User getUserById(Integer userId)
    {
        return userDao.selectById(userId);
    }

    //http://localhost:8888/getUserListByPage?pageNumber=1&pageSize=2
    //条件分页查询
    @GetMapping("getUserListByPage")
    public List<User> getUserListByPage(Integer pageNumber,Integer pageSize)
    {
        Page<User> page =new Page<>(pageNumber,pageSize);
        EntityWrapper<User> entityWrapper = new EntityWrapper<>();
        entityWrapper.eq("user_name", "xiaoli");
        return userDao.selectPage(page,entityWrapper);
    }

}
```

> 这里对上面代码稍作解释，其中包含了如下几个方法：
1.getUserList :这是普通的Mybatis查询的方法，没有用到Mybatis-Plus，这里不做过多解释。
2.getUserListByName：条件查询，根据名称查询用户列表，这里使用到了selectByMap方法，参数需要传一个Map，里面对应写好需要查询的字段名与对应查询值。
3.saveUser ：保存用户，这里使用的是insert方法，需要传一个实体对象，返回Integer值作为影响行数。
4.updateUser ：修改用户，这里使用的是updateByIdt方法，需要传一个实体对象，返回Integer值作为影响行数。
5.getUserById ：根据Id查询实体对象，需要传用户Id。
6.getUserListByPage ：条件分页查询，使用的是selectPage方法，方法需要一个分页对象Page和一个条件对象EntityWrapper。Page放入页码和每页数量，EntityWrapper使用eq方法放入对应字段名和对应查询值。

这里只是举例说明几个方法，其中方法还有很多，更多Mybatis-Plus使用请查看官方文档:[http://baomidou.oschina.io/mybatis-plus-doc/#/?id=%E7%AE%80%E4%BB%8B]()

源码下载 ：[https://gitee.com/dalaoyang/springboot_learn]()