title: Spring WebClient 和 RestTemplate阻塞与非阻塞客户端
date: 2019-07-31 10:17:58
tags: [resttemplate,webclient]
categories: [综合]
---
### 简介

本教程中，我们将对比 Spring 的两种 Web 客户端实现 —— RestTemplate 和 Spring 5 中全新的 Reactive 替代方案 WebClient。

<!--more-->

### 阻塞式 vs 非阻塞式客户端

Web 应用中，对其他服务进行 HTTP 调用是一个很常见的需求。因此，我们需要一个 Web 客户端工具。

#### RestTemplate 阻塞式客户端

很长一段时间以来，Spring 一直提供 RestTemplate 作为 Web 客户端抽象。在底层，RestTemplate 使用了基于每个请求对应一个线程模型（thread-per-request）的 Java Servlet API。

这意味着，直到 Web 客户端收到响应之前，线程都将一直被阻塞下去。而阻塞代码带来的问题则是，每个线程都消耗了一定的内存和 CPU 周期。

让我们考虑下有很多传入请求，它们正在等待产生结果所需的一些慢服务。

等待结果的请求迟早都会堆积起来。因此，程序将创建很多线程，这些线程将耗尽线程池或占用所有可用内存。由于频繁的 CPU 上下文（线程）切换，我们还会遇到性能下降的问题。

#### WebClient 非阻塞式客户端

另一方面，WebClient 使用 Spring Reactive Framework 所提供的异步非阻塞解决方案。

当 RestTemplate 为每个事件（HTTP 请求）创建一个新的 线程 时，WebClient 将为每个事件创建类似于“任务”的东东。幕后，Reactive 框架将对这些 “任务” 进行排队，并仅在适当的响应可用时执行它们。

Reactive 框架使用事件驱动的体系结构。它提供了通过 Reactive Streams API 组合异步逻辑的方法。因此，与同步/阻塞方法相比，Reactive 可以使用更少的线程和系统资源来处理更多的逻辑。

WebClient 是 Spring WebFlux 库的一部分。因此，我们还可以使用流畅的函数式 API 编写客户端代码，并将响应类型（Mono 和 Flux）作为声明来进行组合。

### 案例对比

为了演示两种方法间的差异，我们需要使用许多并发客户端请求来运行性能测试。在一定数量的并发请求后，我们将看到阻塞方法性能的显著下降。

另一方面，无论请求数量如何，反应式/非阻塞方法都可以提供恒定的性能。

就本文而言，让我们实现两个 REST 端点，一个使用 RestTemplate，另一个使用 WebClient。他们的任务是调用另一个响应慢的 REST Web 服务，该服务返回一个 Tweet List。

首先，我们需要引入 Spring Boot WebFlux starter 依赖：

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

创建对象：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Tweet {

    private String name;

    private String describe;

}
```

接下来，这是我们的慢服务 REST 端点：

```java

@GetMapping("/slow-service-tweets")
private List<Tweet> getAllTweets() {
    Thread.sleep(2000L); // delay
    return Arrays.asList(
      new Tweet("RestTemplate rules", "@user1"),
      new Tweet("WebClient is better", "@user2"),
      new Tweet("OK, both are useful", "@user1"));
}
```

### 使用 RestTemplate 调用慢服务

现在，让我们来实现另一个 REST 端点，它将通过 Web 客户端调用我们的慢服务。

首先，我们来使用 RestTemplate：

```java

@GetMapping("/tweets-blocking")
public List<Tweet> getTweetsBlocking() {
    log.info("Starting BLOCKING Controller!");
    final String uri = getSlowServiceUri();

    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<List<Tweet>> response = restTemplate.exchange(
      uri, HttpMethod.GET, null,
      new ParameterizedTypeReference<List<Tweet>>(){});

    List<Tweet> result = response.getBody();
    result.forEach(tweet -> log.info(tweet.toString()));
    log.info("Exiting BLOCKING Controller!");
    return result;
}
```

当我们调用这个端点时，由于 RestTemplate 的同步特性，代码将会阻塞以等待来自慢服务的响应。只有当收到响应后，才会执行此方法中的其余代码。通过日志，我们可以看到：

```
Starting BLOCKING Controller!
Tweet(text=RestTemplate rules, username=@user1)
Tweet(text=WebClient is better, username=@user2)
Tweet(text=OK, both are useful, username=@user1)
Exiting BLOCKING Controller!
```

### 使用 WebClient 调用慢服务

其次，让我们使用 WebClient 来调用慢服务：

```java

@GetMapping(value = "/tweets-non-blocking", 
            produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<Tweet> getTweetsNonBlocking() {
    log.info("Starting NON-BLOCKING Controller!");
    Flux<Tweet> tweetFlux = WebClient.create()
      .get()
      .uri(getSlowServiceUri())
      .retrieve()
      .bodyToFlux(Tweet.class);

    tweetFlux.subscribe(tweet -> log.info(tweet.toString()));
    log.info("Exiting NON-BLOCKING Controller!");
    return tweetFlux;
}
```

本例中，WebClient 返回一个 Flux 生产者后完成方法的执行。一旦结果可用，发布者将开始向其订阅者发送 tweets。注意，调用 /tweets-non-blocking 这个端点的客户端（本例中的 Web 浏览器）也将订阅返回的 Flux 对象。

让我们来观察这次的日志：

```java
Starting NON-BLOCKING Controller!
Exiting NON-BLOCKING Controller!
Tweet(text=RestTemplate rules, username=@user1)
Tweet(text=WebClient is better, username=@user2)
Tweet(text=OK, both are useful, username=@user1)
```

注意，此端点的方法在收到响应之前就已完成。

本文中，我们探讨了在 Spring 中使用 Web 客户端的两种不同方式。

RestTemplate 使用 Java Servlet API，因此是同步和阻塞的。相反，WebClient 是异步的，在等待响应返回时不会阻塞正在执行的线程。只有当程序就绪时，才会产生通知。

RestTemplate 仍将会被使用。但在某些情况下，与阻塞方法相比，非阻塞方法使用的系统资源要少得多。因此，在这些情况下，WebClient 不失为是更好的选择。