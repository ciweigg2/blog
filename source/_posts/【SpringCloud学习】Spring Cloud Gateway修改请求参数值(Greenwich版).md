title: 【SpringCloud学习】Spring Cloud Gateway修改请求参数值(Greenwich版)
date: 2019-07-21 15:25:03
tags: [springcloud]
categories: [springcloud]
---
### 介绍

Spring Cloud Gateway（以下简称 SCG）做为网关服务，是其他各服务对外中转站，通过 SCG 进行请求转发。
在请求到达真正的微服务之前，我们可以在这里做一些预处理，比如：来源合法性检测，权限校验，反爬虫之类…

因为业务需要，我们的服务的请求参数都是经过加密的。
之前是在各个微服务的拦截器里对来解密验证的，现在既然有了网关，自然而然想把这一步骤放到网关层来统一解决。

<!--more-->

参考：https://windmt.com/2019/01/16/spring-cloud-19-spring-cloud-gateway-read-and-modify-request-body/

本文使用的版本：

* Spring Cloud: Greenwich.RELEASE
* Spring Boot: 2.1.3.RELEASE

如果了解的 GatewayFilterFactory 和 GatewayFilter 的关系的话，不用我说你就知道该怎么办了。不知道也没关系，我们把 ModifyRequestBodyGatewayFilterFactory 中红框部分 copy 出来，粘贴到我们之前创建的 ValidateFilter#filter 中

![](/images/20190721131651561561.jpg)

我们稍作修改，即可实现读取并修改 Request Body 的功能了（核心部分见上图黄色箭头处）

废话不多说 直接上代码

```java
import com.alibaba.fastjson.JSONObject;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.cloud.gateway.support.BodyInserterContext;
import org.springframework.cloud.gateway.support.CachedBodyOutputMessage;
import org.springframework.cloud.gateway.support.DefaultServerRequest;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.server.reactive.ServerHttpRequestDecorator;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.BodyInserter;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Collectors;

@Component
public class ValidateFilter implements GlobalFilter, Ordered {

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		ServerRequest serverRequest = new DefaultServerRequest(exchange);
		// mediaType
		MediaType mediaType = exchange.getRequest().getHeaders().getContentType();
		Mono<JSONObject> modifiedBodyJson = null;
		Mono<String> modifiedBodyString = null;
		BodyInserter bodyInserter = null;
		//JSON请求
		if (MediaType.APPLICATION_JSON.isCompatibleWith(mediaType)) {
			// read & modify body
			modifiedBodyJson = serverRequest.bodyToMono(JSONObject.class)
					.flatMap(body -> {
						if (MediaType.APPLICATION_JSON.isCompatibleWith(mediaType)) {

							//这边可以改变request body的值
							System.out.println(body);

							body.put("name" ,"重新赋值了");

							return Mono.just(body);
						}
						return Mono.empty();
					});
			bodyInserter = BodyInserters.fromPublisher(modifiedBodyJson, JSONObject.class);
		}
		//form表单请求
		if (MediaType.APPLICATION_FORM_URLENCODED.isCompatibleWith(mediaType)) {
			// read & modify body
			modifiedBodyString = serverRequest.bodyToMono(String.class)
					.flatMap(body -> {
						if (MediaType.APPLICATION_FORM_URLENCODED.isCompatibleWith(mediaType)) {

							// origin body map
							Map<String, Object> bodyMap = decodeBody(body);

							// TODO decrypt & auth

							// new body map
							Map<String, Object> newBodyMap = new HashMap<>();
							newBodyMap.put("name","重新设置呀");

							return Mono.just(encodeBody(newBodyMap));
						}
						return Mono.empty();
					});
			bodyInserter = BodyInserters.fromPublisher(modifiedBodyString, String.class);
		}

		HttpHeaders headers = new HttpHeaders();
		headers.putAll(exchange.getRequest().getHeaders());

		// the new content type will be computed by bodyInserter
		// and then set in the request decorator
		headers.remove(HttpHeaders.CONTENT_LENGTH);

		CachedBodyOutputMessage outputMessage = new CachedBodyOutputMessage(exchange, headers);
		return bodyInserter.insert(outputMessage,  new BodyInserterContext())
				.then(Mono.defer(() -> {
					ServerHttpRequestDecorator decorator = new ServerHttpRequestDecorator(
							exchange.getRequest()) {
						@Override
						public HttpHeaders getHeaders() {
							long contentLength = headers.getContentLength();
							HttpHeaders httpHeaders = new HttpHeaders();
							httpHeaders.putAll(super.getHeaders());
							if (contentLength > 0) {
								httpHeaders.setContentLength(contentLength);
							} else {
								httpHeaders.set(HttpHeaders.TRANSFER_ENCODING, "chunked");
							}
							return httpHeaders;
						}

						@Override
						public Flux<DataBuffer> getBody() {
							return outputMessage.getBody();
						}
					};
					return chain.filter(exchange.mutate().request(decorator).build());
				}));
	}

	@Override
	public int getOrder() {
		return 0;
	}

	private Map<String, Object> decodeBody(String body) {
		return Arrays.stream(body.split("&"))
				.map(s -> s.split("="))
				.collect(Collectors.toMap(arr -> arr[0], arr -> arr[1]));
	}

	private String encodeBody(Map<String, Object> map) {
		return map.entrySet().stream().map(e -> e.getKey() + "=" + e.getValue()).collect(Collectors.joining("&"));
	}

}
```

上述例子中实现了form表单和json请求参数的解析

配置代码：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-clientA

使用postman测试即可 太简单了不做解释了

2个测试接口：

* http://localhost:8007/hello/form

* http://localhost:8007/hello/json

请求参数：

```
{
	"name":"123",
	"id":"1"
}
```

对了要先把路有规则配置好 gateway项目中设置

bootstrap.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: retry_test
          uri: lb://eureka-client-a
          predicates:
            - Path=/hello/**
```

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-gateway