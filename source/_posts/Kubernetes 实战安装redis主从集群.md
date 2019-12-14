cover: http://ciwei2.cn-sh2.ufileos.com/66.jpg
title: Kubernetes 实战安装redis主从集群
date: 2018-09-09 19:28:54
tags: [Kubernetes,redis,springboot]
categories: [综合]
---
 Kubernetes 创建redis集群的时候 会在所有的子节点创建 master节点不参与创建
 有两个redis后端节点：一个redis-master和两个redis-slave，两个redis-slave从redis-master进行同步数据
 使用程序写入的时候写入master 从数据库都有数据了
 <!--more-->
 
 ### 创建redis-master-controller.yaml
 
 redis-master-controller.yaml
 
 ```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      name: redis-master
      labels:
        name: redis-master
    spec:
      containers:
      - name: redis-master
        image: kubeguide/redis-master
        ports:
        - containerPort: 6379
```

发布到kubernetes集群,自动创建pod

```
kubectl create -f redis-master-controller.yaml
kubectl get rc
kubectl get pods
```

### 创建redis-master-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  type: NodePort
  ports:
  - port: 6379
    targetPort: 6379
    nodePort: 30001
  selector:
    name: redis-master
```

创建service

```
kubectl create -f redis-master-service.yaml
kubectl get services
```

### 创建redis-slave-controller.yaml

redis-slave-controller.yaml

```
apiVersion: v1
kind: ReplicationController
metadata:
  name namenameameeis-slave
spec:
  replicas: 2
  selector:
    name: ave
  template:
    metadata:
      name: redis-slave
      labels:
        name: redis-slave
    spec:
      containers:
      - name: redis-slave
        image: kubeguide/guestbook-redis-slave
        env:
        - name: GET_HOSTS_FROM
          value: env
        ports:
        - containerPort: 6379
```

创建

```
kubectl create -f redis-slave-controller.yaml
kubectl get rc
kubectl get pods
```

### 创建redis-slave-service.yaml

redis-slave-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  type: NodePort
  ports:
  - port: 6379
    nodePort: 30002
  selector:
    name: redis-slave
```

创建

```
kubectl create -f redis-slave-service.yaml
kubectl get services
```

删除

```
kubectl delete -f redis-master-controller.yaml
kubectl delete -f redis-master-service.yaml
kubectl delete -f redis-slave-controller.yaml
kubectl delete -f redis-slave-service.yaml
```

暴露的master端口 30001
node端口 30002 因为node是副本创建的 创建了2个 连接一个测试就可以了

![](/images/redis主从.png)

从只读 无法删除新增 master可以新增删除

### SpringBoot连接redis主从配置

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

```
spring.redis.database=0
spring.redis.host=masterip
spring.redis.port=30001
spring.redis.pool.testOnBorrow=true
spring.redis.pool.blockWhenExhausted=true
spring.redis.pool.numTestsPerEvictionRun=3
spring.redis.pool.timeBetweenEvictionRunsMillis=-1
```

```
@SpringBootApplication
@RestController
public class DemoApplication {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @RequestMapping(value = "/")
    public void init(){
        stringRedisTemplate.opsForValue().set("messages" ,String.valueOf(System.currentTimeMillis()));
        String s = stringRedisTemplate.opsForValue().get("messages").toString();
        System.out.println(s);
    }

}
```

新增的时候会同步到从库，读取的时候回随机从从库读取的