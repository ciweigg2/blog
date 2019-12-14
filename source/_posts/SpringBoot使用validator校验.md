cover: http://ciwei2.cn-sh2.ufileos.com/116.jpg
title: SpringBoot使用validator校验
date: 2018-07-26 10:08:48
tags: [SpringBoot]
categories: [微服务]
thumbnail: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1533363191&di=6da2a77162e913e2b961d02d334a865c&imgtype=jpg&er=1&src=http%3A%2F%2Fqn.18touch.com%2Fphpcms%2F2016%2F0721%2F20160721064804287.jpg
---

在前台表单验证的时候，通常会校验一些数据的可行性，比如是否为空，长度，身份证，邮箱等等，那么这样是否是安全的呢，答案是否定的。因为也可以通过模拟前台请求等工具来直接提交到后台，比如postman这样的工具，那么遇到这样的问题怎么办呢，我们可以在后台也做相应的校验。
<!--more-->
新建项目，因为本文会使用postman模拟前端请求，所以本文需要加入web依赖，pom文件如下：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.dalaoyang</groupId>
	<artifactId>springboot_validator</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot_validator</name>
	<description>springboot_validator</description>

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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
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
创建一个demo类，说一下本文使用demo中校验使用的注解：
@NotEmpty：非空
@Length：长度，最长或者最短
@Email：校验email
@Pattern：使用正则校验，本文使用的是身份证的正则
，代码如下：
```java
package com.dalaoyang.entity;

import org.hibernate.validator.constraints.Email;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotEmpty;

import javax.validation.constraints.Pattern;
import java.io.Serializable;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.entity
 * @email yangyang@dalaoyang.cn
 * @date 2018/5/1
 */
public class Demo implements Serializable {

    @NotEmpty(message="用户名不能为空")
    @Length(min=6,max = 12,message="用户名长度必须位于6到12之间")
    private String userName;


    @NotEmpty(message="密码不能为空")
    @Length(min=6,message="密码长度不能小于6位")
    private String passWord;

    @Email(message="请输入正确的邮箱")
    private String email;

    @Pattern(regexp = "^(\\d{18,18}|\\d{15,15}|(\\d{17,17}[x|X]))$", message = "身份证格式错误")
    private String idCard;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getIdCard() {
        return idCard;
    }

    public void setIdCard(String idCard) {
        this.idCard = idCard;
    }

}
```
创建一个TestDemoController，来测试本文的校验，代码如下：

```java
package com.dalaoyang.controller;

import com.dalaoyang.entity.Demo;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.List;

/**
 * @author dalaoyang
 * @Description
 * @project springboot_learn
 * @package com.dalaoyang.controller
 * @email yangyang@dalaoyang.cn
 * @date 2018/5/1
 */
@RestController
public class TestDemoController {

    @PostMapping("/")
    public String testDemo(@Valid Demo demo,BindingResult bindingResult){
        StringBuffer stringBuffer = new StringBuffer();
        if(bindingResult.hasErrors()){
            List<ObjectError> list =bindingResult.getAllErrors();
            for (ObjectError objectError:list) {
                stringBuffer.append(objectError.getDefaultMessage());
                stringBuffer.append("---");
            }
        }
        return stringBuffer!=null?stringBuffer.toString():"";
    }
}
```
启动项目使用postman分别做了三次请求，第一次所有属性都是随便填写的，如图

![](/images/1.jpg)

第二次输入正确的身份证和邮箱，用户名和密码为空，如图

![](/images/2.jpg)

第三次全部输入正确，如图

![](/images/3.jpg)

本文只是使用的简单的几种校验，Hibernate-validator还有很多种校验的方法，大家可以参考这篇文章https://blog.csdn.net/xgblog/article/details/52548659
