title: 【SpringCloud学习】Spring Cloud Gateway获取response的值(Greenwich版)
date: 2019-07-21 18:35:07
tags: [springcloud]
categories: [springcloud]
---
SpringCloudGateway获取、修改客户端请求Request的参数，我们在上一篇已经讲过了。那么网关发起请求后，微服务返回回来的response的值，还是要经过网关才发给客户端的。很多时候，我们希望能看到响应的值，或者修改它，其实很简单

一定要切记接口必须返回标准的json格式 不能用string字符串的json格式 这样会无法修改的

<!--more-->

```java
import com.alibaba.fastjson.JSONObject;
import org.reactivestreams.Publisher;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferFactory;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.http.server.reactive.ServerHttpResponseDecorator;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.nio.charset.Charset;

/**
 * @author ciwei
 */
@Component
public class WrapperResponseGlobalFilter implements GlobalFilter, Ordered {
	@Override
	public int getOrder() {
		// -1 is response write filter, must be called before that
		return -2;
	}

	@Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		ServerHttpResponse originalResponse = exchange.getResponse();
		DataBufferFactory bufferFactory = originalResponse.bufferFactory();
		ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(originalResponse) {
			@Override
			public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
				if (body instanceof Flux) {
					Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
					return super.writeWith(fluxBody.map(dataBuffer -> {
						// probably should reuse buffers
						byte[] content = new byte[dataBuffer.readableByteCount()];
						dataBuffer.read(content);
						//释放掉内存
						DataBufferUtils.release(dataBuffer);
						String s = new String(content, Charset.forName("UTF-8"));
						JSONObject jsonObject = JSONObject.parseObject(s);
						jsonObject.put("name" ,"修改name了好不好的呀额");
						//TODO，s就是response的值，修改、查看都可以
//						byte[] uppedContent = new String(content, Charset.forName("UTF-8")).getBytes();
						byte[] uppedContent = jsonObject.toJSONString().getBytes();
						return bufferFactory.wrap(uppedContent);
					}));
				}
				// if body is not a flux. never got there.
				return super.writeWith(body);
			}
		};
		// replace response with decorator
		return chain.filter(exchange.mutate().response(decoratedResponse).build());
	}

}
```

就是定义这样一个GlobalFilter，注意order要小于-1.通过上面的类，就能查看服务端响应的值了。

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-gateway
