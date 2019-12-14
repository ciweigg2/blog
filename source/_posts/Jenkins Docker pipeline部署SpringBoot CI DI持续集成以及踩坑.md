title: Jenkins Docker pipeline部署SpringBoot CI DI持续集成以及踩坑
date: 2019-08-04 12:14:43
tags: [jenkins]
categories: [综合]
---
### 安装

参考：https://blog.csdn.net/qq_37143673/article/details/97613633

<!--more-->

demo：https://github.com/ciweigg2/test-walle

寻找需要的 Jenkins 镜像：https://hub.docker.com/r/jenkinsci/blueocean

```
docker pull jenkinsci/blueocean
```

我选择的镜像是 Jenkins-blueocean Jenkins 海洋版，为什么选这个？

* 踩坑：普通的 Jenkins 在部署的时候不少人都遇到过，插件下不下来，但是在海洋版没有这个问题（最主要原因）

* blueocean 的页面更加人性化，流程的监控上看着让人舒服的多，当然普通 Jenkins 也可以通过安装插件添加这个功能

启动镜像

```
docker run --name jenkinsci-blueocean -u root --rm  -d -p 7005:8080 -p 50000:50000 -v /data/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

* -u root：以 root 权限启动，防止出现权限问题

* -p 7005:8080：端口映射，服务器的 7005 端口映射容器的 8080 端口

* -p 50000:50000：Jenkins代理默认通过TCP端口50000与Jenkins主机通信

* -v /data/jenkins:/var/jenkins_home：把容器内的 Jenkins 目录挂载到服务器的 /data/jenkins 目录以防容器没了，数据也没了

* -v /var/run/docker.sock:/var/run/docker.sock：保证容器内的 docker 与 服务器上 docker 的通讯

附带下删除 jenkinsci/blueocean 容器

```
# 删除对应绑定网桥
docker network disconnect --force bridge jenkinsci-blueocean
 
# 删除 jenkinsci-blueocean 容器，xxxx  容器 ID
docker rm -f xxxx
```

输入密码进入

![](/images/20190804105300.png)

由于我们挂载映射到服务器，所以可以直接通过服务器路径找密码

```
cat /data/jenkins/secrets/initialAdminPassword
```

接下来，我选择推荐插件安装

![](/images/20190804105356.png)

创建第一个账号，我用的 root 123456

![](/images/20190804105432.png)

然后完成安装，由于是用的是镜像，所以安装起来非常的简单，海洋版也没有出现插件无法下载的问题

### 创建项目与持续集成

> 踩坑PS：为什么说难用，就是在这。一开始我是把 jdk 和 maven 也挂载到容器里的，但是配置完全局变量后，根本无法使用，可能是容器内也要配置好这些才行，我就搞不懂容器内配了那还要专门配个全局环境变量干啥。。。自动安装也是完全没用，所以我放弃了，用我自己的方法进行持续集成部署。

接下来配置项目，选择流水线

![](/images/20190728133823637123.png)

配置

这里我们通过 webhook，让项目提交后，自动调用钩子函数进行部署，进行持续集成
配置构建触发器，也就是钩子函数

![](/images/20190728134315492123.png)

身份验证令牌自己写的，所以这里钩子函数的路径就是

```
http://127.0.0.1:7005/job/spring-cloud/build?token=123456
```

PS：身份验证令牌框下面就是钩子函数的请求路径说明，不太明白的，可以自己看看，我上面给的这个是示例

配置项目

![](/images/20190728135051271123.png)

![](/images/20190728133823637123.png)

公私密钥的可参考：https://blog.csdn.net/weixin_41235146/article/details/81780894 或者 https://ciweigg2.github.io/2019/08/04/jenkins-docker-pipeline-pei-zhi-gitlab-jenkins-gong-si-yao/

然后保存，这里暂时就配好了

root 创建 API token

![](/images/20190728140528768123.png)

![](/images/20190728140708145123.png)

![](/images/20190728140758785123.png)

记住这串 token，后面要和钩子函数一起使用，离开这个页面后再进入就看不到这串 token 了

然后保存呀

系统管理 => 全局安全配置(Manage Jenkins => Configure Global Security)

![](/images/20190728141354974123.png)

PS踩坑：这里不配置的话，后面钩子函数的请求都会 403 失败

接下来配置钩子函数的请求了，这里以 gitee 为例

进入项目 => 管理 => WebHook

![](/images/20190728141813399123.png)

请求历史可以看到请求成功 201

![](/images/20190728142040545123.png)

先说下本人用于测试的这个项目是父子项目，打包时候把每个子项目打成 jar 包，然后通过 Dockerfile 生产 docker 镜像，然后启动镜像生成容器，这就是我的部署流程

项目下新增 Jenkinsfile

PS踩坑：由于 maven 配置不成功，我也不想进容器配置 maven，因为服务器才 1G ，进容器就死机了，所以我改了依赖 maven 镜像部署，然后再通过另一个任务把 jar 包生成镜像，因为 maven 容器里没有 docker 环境

```
pipeline {
    agent any
    stages {
        stage('package') {
            agent {
        		docker {
            		        image 'maven'
            		        args '-v /root/.m2:/root/.m2 -v /data/maven/apache-maven-3.6.0/conf/settings.xml:/data/maven/conf/settings.xml --entrypoint='
        		}
    	    }
            steps {
                script{
                    echo "WORKSPACE：${env.WORKSPACE}"
                    echo "Branch：${env.NODE_NAME}"
                    if ("${env.NODE_NAME}" == "master") {
                        sh "sh package-prod.sh"
                    }
                }
            }
        }
        stage('build') {
            agent none
            steps {
                script{
                    echo "WORKSPACE：${env.WORKSPACE}"
                    echo "Branch：${env.NODE_NAME}"
                    if ("${env.NODE_NAME}" == "master") {
                        sh "sh build-prod.sh"
                    }
                }
            }
        }
    }
}
```

先 pull 个 maven 镜像，以及在构建镜像时候所需要的 jdk8 的镜像(提高构建速度)

```
# maven 用于 Jenkins 构建有 maven 环境的容器
docker pull maven
 
# jdk8 的镜像用于 Dockerfile 中设置构建拥有 jdk8 环境镜像的基础镜像
docker pull kdvolder/jdk8
```

* image 'docker.io/maven'：基于这个镜像生成容器部署，所以环境有 maven 环境

* args 中 -v /root/.m2:/root/.m2：把依赖挂载出来，不可能每次打包都下依赖

* args 中 -v /data/maven/apache-maven-3.6.0/conf/settings.xml:/data/maven/conf/settings.xml：阿里环境配置挂进去 本地这个目录存放settings.xml /data/maven/apache-maven-3.6.0/conf/settings.xml 这个目录可以随便写只要里面有settings.xml就行了

* --entrypoint=：配置下好像就不会打开进入容器了，没深入研究

* ${env.WORKSPACE}：工作空间路径

* ${env.NODE_NAME}：进行的分支

Jenkins 内置环境变量可参考：https://www.cnblogs.com/puresoul/p/4828913.html 也可以百度jenkins的环境变量

可以看到，我这部署先判度为 master 分支的情况下，package 任务中调用脚本进行打包，build 任务下调用脚本部署

下面贴下这两个脚本，大伙可以参考下 都放在项目根目录

package-prod.sh

```
#!/bin/sh
#进入文件根目录
#cd "$WORKSPACE"
 
#项目打包
mvn clean install package '-Dmaven.test.skip=true'
```

build-prod.sh

里面有多模块和单模块的区分

```
#!/bin/sh
#进入文件根目录
#cd "$WORKSPACE"
 
#启用 prod 配置
ActiveProfiles=prod
 
#基本信息需要配置
#内部端口
targetPort=8080
#旧镜像版本号
oldVendor=1.0.0
#镜像版本号
vendor=1.0.0
#项目名
projectName=test-walle-0.0.1
 
#进入target文件夹
#直接的构建是再容器里，这个是在 Jenkins 容器里，所以空间不一样
#容器的空间是原空间路径后面多了 @2
#多模块使用
#cd $WORKSPACE@2/$projectName/target
#单模块使用
cd $WORKSPACE@2/target
 
#创建Dockerfile文件
#-jar -Duser.timezone=GMT+08 保证生成出来的容器的时区与服务器一致
cat << EOF > Dockerfile
FROM kdvolder/jdk8
MAINTAINER $projectName
VOLUME /tmp
LABEL app="$projectName" version="$vendor" by="$projectName"
COPY $projectName.jar $projectName.jar
EXPOSE $targetPort
#多环境使用
#CMD java -Xmx100m -Xms100m -jar -Duser.timezone=GMT+08 $projectName.jar --spring.profiles.active=$ActiveProfiles
#单环境使用
CMD java -Xmx100m -Xms100m -jar -Duser.timezone=GMT+08 $projectName.jar
#ENTRYPOINT java
EOF
 
#删除镜像下所有容器
docker rm -f $(docker ps -a | grep "$projectName" | awk '{print $1}')
 
#删除旧镜像
docker rmi -f $projectName:$oldVendor
 
#创建镜像
docker build -t $projectName:$vendor .
 
#启动镜像生成容器
docker run --name $projectName -d -p $targetPort:$targetPort $projectName:$vendor
```

踩坑：xxx.jar包中不能包含大写的

我写配置喜欢统一性配置，后面改起来简单，上面参数换下下面都不需要改，这里我的文件夹名，镜像名和生产的 jar 名都用一样的也是这个目的，如果要替换一些文件或者修改一些配置参数的话在package-prod.sh中替换就行

![](/images/20190804111418.png)

打包成功了 服务器也自动发布了 如果只想打包不想重新发布可以去掉docker run 的命令呀

上面说的是单分支流水线 那么多分支很简单的 构建个多分支的流水线就行了

![](/images/20190804120628.png)