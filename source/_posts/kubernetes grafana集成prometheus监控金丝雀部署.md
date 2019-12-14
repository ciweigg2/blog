cover: http://ciwei2.cn-sh2.ufileos.com/65.jpg
title: kubernetes grafana集成prometheus监控金丝雀部署
date: 2019-06-15 17:56:06
tags: [kubernetes,grafana,prometheus]
categories: [综合]
---
### 搭建springboot项目

demo地址：https://github.com/ciweigg2/test-walle.git

springboot2.x集成prometheus

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <version>1.1.4</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

application.properties

```java
spring.application.name=springboot_prometheus
management.endpoints.web.exposure.include=*
management.metrics.tags.application=${spring.application.name}
```

核心aop监控请求代码 也可以认为是埋点呀

```java
/**
 * Created by Administrator on 2019/6/15.
 *
 * 集成prometheus监控接口流量 实现金丝雀部署
 */
@Component
@Aspect
public class PrometheusAop {

	@Pointcut("execution(public * com.example.testwalle.controller.*.*(..))")
	public void pointCut(){}

	//统计请求的处理时间
	ThreadLocal<Long> startTime = new ThreadLocal<>();

	@Autowired
	MeterRegistry registry;

	private Counter counter;

	@PostConstruct
	private void init(){
		counter = registry.counter("requests_total","status","success");
	}

	@Before("pointCut()")
	public void doBefore(JoinPoint joinPoint) throws Throwable{
		startTime.set(System.currentTimeMillis());

		counter.increment(); //记录系统总请求数
	}

	@AfterReturning(returning = "returnVal" , pointcut = "pointCut()")
	public void doAfterReturning(Object returnVal){
		//处理完请求后
		System.out.println("方法执行时间:"+ (System.currentTimeMillis() - startTime.get()));
	}

}
```

下载demo访问 http://ip:8080/walle

### kubernetes搭建grafana prometheus

参考我的博客：kubernetes上用helm部署loki prometheus grafana

### 部署金丝雀

> 首先部署一个1.11.1版本的程序

prometheus.io/scrape: "true" 如果需要prometheus监控必须开启

prometheus.io/path: /actuator/prometheus springboot程序开启prometheus的地址

prometheus.io/port: "8080" 程序的端口

vi k8s-demo.yaml

```java
apiVersion: v1
kind: Service
metadata:
  name: k8s-service
  namespace: default
  labels:
    app: k8s-service
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30002
    targetPort: 8080
  selector:
    app: k8s-service

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-service
  labels:
    app: k8s-service
spec:
  replicas: 4
  selector:
    matchLabels:
      app: k8s-service
      release: 1.11.1
  template:
    metadata:
      labels:
        app: k8s-service
        release: 1.11.1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: /actuator/prometheus
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: k8s-service
        image: demo:1.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        livenessProbe:
          tcpSocket:
            port: 8080
          periodSeconds: 15
          initialDelaySeconds: 30
```

kubectl create -f k8s-demo.yaml

> 再部署一个1.11.2版本的程序

不需要Service了 一个Service可以对应多个Deployment

需要修改的地方 

* image: demo:1.0 改成 demo:2.0 (新版本，我测试就没打2个镜像呀)
* release: 1.11.1 改成 release: 1.11.2
* 添加env(可以省略的)
* name: k8s-service-v1.11.1 改成 name: k8s-service-v1.11.2

vi k8s-demo2.yaml

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-service-v1.11.2
  labels:
    app: k8s-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: k8s-service
      release: 1.11.2
  template:
    metadata:
      labels:
        app: k8s-service
        release: 1.11.2
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: /actuator/prometheus
        prometheus.io/port: "8080"
    spec:
      containers:
      - name: k8s-service
        image: demo:1.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: VERSION
          value: v1.11.2
        livenessProbe:
          tcpSocket:
            port: 8080
          periodSeconds: 15
          initialDelaySeconds: 30
```

kubectl create -f k8s-demo2.yaml

### grafana界面滴配置

![](/images/20190615183027.png)

其实kubernetes安装好已经自动配置了prometheus 这边还是自己添加一下

![](/images/20190615183231.png)

查看prometheus是否2个版本的服务正常呀

![](/images/20190615183556.png)

多访问几次 http://ip:30002/walle

![](/images/20190615183642.png)

### 添加金丝雀监控图表

添加这个公式：sum(rate(requests_total{app="k8s-service"}[5m])) by (release)

解释：requests_total对应代码中的埋点 app代表yaml中的标签 release代表yaml中的版本 统计这些条件成立后的总和呀

![](/images/20190615183949.png)

下面这些图表格式设置一下

![](/images/20190615185635.png)

![](/images/20190615185658.png)

![](/images/20190615185722.png)

![](/images/20190615185744.png)

### 演示金丝雀发布

先看看现在两个服务都有1.11.1版本4个副本，1.11.2版本1个副本的情况下

可以看到流量集中在1.11.1版本 因为部署了4个副本

![](/images/20190615191137.png)

调整1.11.1为1个副本 1.11.2为3个 看到流量都集中在1.11.2版本中了

![](/images/20190615191924.png)

最后删除1.11.1版本 全部流量转到1.11.2版本 完成金丝雀的部署呀哈哈哈

![](/images/20190615192104.png)

可以集成traefik去控制流量完成金丝雀部署 但是一定要带着监控哦代码中一定要有埋点哦 和开启了监控