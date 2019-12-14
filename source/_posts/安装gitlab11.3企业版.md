cover: http://ciwei2.cn-sh2.ufileos.com/147.jpg
title: 安装gitlab11.3企业版
date: 2018-09-23 14:52:04
tags: [gitlab]
categories: [综合]
---
11.3企业版新增了maven仓库功能

开发团队通过直接将 Maven 仓库构建到 GitLab 中，扩展了对 Java 项目和开发者的支持。这为 Java 开发者提供了一种安全、标准化的方式来共享 Maven 库中的版本控制，并通过在项目中重用这些库来节省时间。该功能在 GitLab Premium 版本中提供

GitLab Premium 收费的这么好用的功能只能放弃了哦

<!--more-->

### 企业版

```
sudo docker run --detach \
    --hostname 118.184.218.184 \
    --publish 443:443 --publish 80:80 --publish 1122:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ee:latest
```

### 社区版

```
sudo docker run --detach \
    --hostname 118.184.218.184 \
    --publish 443:443 --publish 80:80 --publish 1122:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

![](/images/20180923152536.png)

### 设置中文

![](/images/1537687481.jpg)

> 好吧，我还是要介绍下怎么使用这个功能 虽然企业版和社区版免费的没有

导航到项目的“设置”>“通用”>“Permissions”。

找到“Packages ”功能并启用它。

单击“ 保存更改”以使更改生效。

你会发现项目中有个Packages按钮了

然后点击用户->设置->访问令牌->创建token->设置所有权限

记住生成的token(一定要保存)

maven settings.xml中添加：

```java
	<server>
      <id>gitlab-com</id>
      <configuration>
        <httpHeaders>
          <property>
            <name>token名</name>
            <value>生成的token</value>
          </property>
        </httpHeaders>
      </configuration>
    </server>
```

pom.xml (替换gitlab的地址，替换project-id是项目的id)

```java
	<distributionManagement>
		<snapshotRepository>
			<id>gitlab-com</id>
			<url>http://gitlab的地址/api/v4/projects/project-id/packages/maven</url>
		</snapshotRepository>
		<repository>
			<id>gitlab-com</id>
			<url>http://gitlab的地址/api/v4/projects/project-id/packages/maven</url>
		</repository>
	</distributionManagement>
	<scm>
		<connection>scm:git:http://gitlab的地址/root/test.git</connection>
		<url>http://gitlab的地址/root/test</url>
		<developerConnection>scm:git:http://gitlab的地址/root/test.git</developerConnection>
		<tag>HEAD</tag>
	</scm>
```

```java
mvn deploy
```

将会发现已经发布到gitlab了 在项目的Packages里面可以找到
