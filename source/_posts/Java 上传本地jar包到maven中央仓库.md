title: Java 上传本地jar包到maven中央仓库
date: 2019-07-27 14:07:45
tags: [maven]
categories: [综合]
---
### 介绍

一定要记住项目版本必须1.0.RELEASE这样 必须要有RELEASE

后来尝试了下发现不加RELEASE也可以成功 难道是首次需要RELEASE版本 哈哈哈不知道

<!--more-->

### 注册sonatype账号：【申请上传资格】

https://issues.sonatype.org/secure/Signup!default.jspa

![](/images/20190624114719621.png)

如下注册成功！

![](/images/20190624115239574.png)

### 登录

https://issues.sonatype.org/secure/Dashboard.jspa

![](/images/20190624115900673.png)

登录成功进来之后可选择自己喜欢的语言显示~

接下来的就是创建头像等等了，这里不多说

### 新建issue

点击新建按钮

![](/images/20190727141916.png)

这里可以采用github作为Group Id、Project URL、SCM url

如何使用github的信息呢?

Group Id：填写com.github.xx -> xx为github用户名(代码的包名和groupId都要对应)

Project URL：填写github项目的地址

SCM url：填写github中的仓库名.git

附件可以不传的 如果传的话就项目打包后的jar

好了重点来了

创建好后 左侧栏会出现jira号 现在我们需要把Project URL和SCM url都改成这个JIRA号的地址(问题一定要是打开状态的 不然他审核太慢了 如果审核失败重新创建都比修改快 等待几分钟就好了 会收到github绑定邮箱的通知的)

然后去github修改项目名为这个JIRA号的名字就行了(我一开始不知道直接新建了个jira号的仓库将代码上传上去的 我觉得新建个和jira号一样的空仓库也可以的)

![](/images/20190727141434.png)

### 构件仓库上传jar包

https://oss.sonatype.org/#welcome

将jar包上传到这里，Release 之后就会同步到maven中央仓库

#### 本地安装gpg，并使用gpg生成密钥对

注：发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。

#### 下载安装gpg4win

Windows系统下载地址： https://www.gpg4win.org/download.html

#### 安装很简单，如下：

![](/images/20190627150325388.png)

cmd执行如下命令验证是否安装成功：

```
gpg --version
```

使用gpg生成密钥对

cmd执行如下命令：

```
gpg --gen-key
```

添加github账号和绑定的邮箱

![](/images/20190727142440.png)

如果ok之后出现如下界面，是提示密码安全度不高，需要包含至少一个数字或特殊字符~ 重新输入一下即可

![](/images/2019062715292373.png)

![](/images/20190627153128810.png)

ok之后，我们的密钥对就设置好了

【注】，下图中的生成的key，后面要用到

![](/images/20190727142719.png)

#### 上传GPG公钥

查看公钥

```
gpg --list-keys
```

其中5C61C0A461C1B457EFB99DE0716F132F80263AB7为公钥id

![](/images/20190727142931.png)

将公钥或key发布到 PGP 密钥服务器（注：这里我暂时未发现有何区别~）

```
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 公钥ID或上面提到的key
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 公钥ID或上面提到的key
```

查询公钥是否发布成功

```
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 公钥ID或上面提到的key
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys 公钥ID或上面提到的key
```

### 在maven的setting.xml配置文件中添加如下节点信息

```
<servers>
  <!-- 上传jar包到maven中央仓库配置start -->
  <server>
      <id>ossrh</id>
      <username>Sonatype账号</username>
      <password>Sonatype密码</password>
  </server>
  <!-- 上传jar包到maven中央仓库配置end -->
</servers>
```

### 配置项目的pom.xml文件

```xml
    <licenses>
        <license>
            <name>GNU Lesser General Public License v3.0</name>
            <url>https://www.gnu.org/licenses/lgpl-3.0.txt</url>
        </license>
    </licenses>

    <organization>
        <name>ciweigg</name>
        <url>https://github.com/ciweigg</url>
    </organization>

    <developers>
        <developer>
            <name>ciweigg</name>
            <email>ciweigg@qq.com</email>
        </developer>
    </developers>

    <scm>
        <url>https://github.com/baomidou/redisson-spring-boot-starter</url>
        <connection>scm:git:https://github.com/ciweigg/redisson-spring-boot-starter.git</connection>
        <developerConnection>scm:git:https://github.com/ciweigg/redisson-spring-boot-starter.git
        </developerConnection>
        <tag>HEAD</tag>
    </scm>
    <profiles>
        <profile>
            <id>ossrh</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugins>
                    <!-- 要生成Javadoc和Source jar文件，您必须配置javadoc和源Maven插件 -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!--  必须配置GPG插件用于使用以下配置对组件进行签名 -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <!-- 【注】snapshotRepository 与 repository 中的 id 一定要与 setting.xml 中 server 的 id 保持一致！ -->
            <distributionManagement>
                <snapshotRepository>
                    <id>ossrh</id>
                    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
                </snapshotRepository>
                <repository>
                    <id>ossrh</id>
                    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
        </profile>
```

可以添加一些作者版权声明的(可选的最好还是加上吧中央仓库貌似要搜得到需要加的)

```
    <licenses>
        <license>
            <name>GNU Lesser General Public License v3.0</name>
            <url>https://www.gnu.org/licenses/lgpl-3.0.txt</url>
        </license>
    </licenses>

    <organization>
        <name>ciweigg</name>
        <url>https://github.com/ciweigg</url>
    </organization>

    <developers>
        <developer>
            <name>ciweigg</name>
            <email>ciweigg@qq.com</email>
        </developer>
    </developers>

    <scm>
        <url>https://github.com/baomidou/redisson-spring-boot-starter</url>
        <connection>scm:git:https://github.com/ciweigg/redisson-spring-boot-starter.git</connection>
        <developerConnection>scm:git:https://github.com/ciweigg/redisson-spring-boot-starter.git
        </developerConnection>
        <tag>HEAD</tag>
    </scm>
```

### 部署和发布Jar包

代码中的javadoc注释一定要规范否则没办法生成的

进入自己的项目 idea中点击deploy就行啦(我这边使用命令mvn clean install deploy测试成功的)

当我们的项目中含有多个模块时，我们可以使用 -projects 来指定部署哪一个模块

举例：

部署一个模块如下： 【demo和demo2为模块名】

```
mvn clean deploy -projects demo
```

部署两个模块如下：

```
mvn clean deploy -projects demo,demo2ails/94381467
```

【注】第一次执行时需要输入之前设置的passphrase密码 ~

> 如果不想出现此，也可在一开始直接执行如下命令：
mvn clean deploy -P sonatype-oss-release -Darguments="gpg.passphrase=设置gpg密钥时输入的Passphrase"

![](/images/20190627171347817.png)

### 上传所遇问题

如果出现上传问题或者发布问题可以先删除原来的，可以尝试将 https://oss.sonatype.org/#stagingRepositories 上之前上传的错误项目全部删除【选中点击Drop即可删除】，然后再次上传~

![](/images/20190701135516639.png)

### 同步到maven中央仓库

到 https://oss.sonatype.org/#stagingRepositories 中勾选自己上传的构件（我们的jar包上传到这里哦）点击Close然后再Release，Release之后就会同步到maven中央仓库

![](/images/20190702092911688.png)

查看构建过程是否有失败的

![](/images/20190727145233.png)

最终到 [maven中央仓库](https://mvnrepository.com/) 中就可以搜索到了 不过可能时间比较久哦

我发现maven中央仓库的界面同步实现太慢了

aliyun的maven仓库和http://repo.maven.apache.org/maven2 的中央仓库索引早就有了

![](/images/20190728095757.png)

参考：https://blog.csdn.net/qq_38225558/article/details/94381467