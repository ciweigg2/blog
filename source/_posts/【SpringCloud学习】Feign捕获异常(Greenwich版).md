title: 【SpringCloud学习】Feign捕获异常(Greenwich版)
date: 2019-07-21 21:38:31
tags: [springcloud]
categories: [springcloud]
---
### 介绍

feign不使用熔断器的时候捕获异常

实现ErrorDecoder获取抛出的异常

<!--more-->

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import feign.Response;
import feign.Util;
import feign.codec.ErrorDecoder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.Configuration;

import java.io.IOException;

@Slf4j
@Configuration
public class ExceptionErrorDecoder implements ErrorDecoder {

	ObjectMapper objectMapper = new ObjectMapper();

	public Exception decode(String methodKey, Response response) {
		ObjectMapper om = new ObjectMapper();
		ExceptionInfo resEntity = null;
		Exception exception = null;
		try {
			resEntity = om.readValue(Util.toString(response.body().asReader()), ExceptionInfo.class);
		} catch (IOException ex) {
			log.error(ex.getMessage(), ex);
		}
		return new Exception(resEntity.getMessage());
	}
}
```

创建异常转换类

```java
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

@Data
public class ExceptionInfo implements Serializable {

	private Date timestamp;

	private Integer status;

	private String message;

	private String path;

	private String error;

}
```

### 关闭feign的熔断器

feign.hystrix.enabled=false

默认为false的 如果打开了要关闭


### 测试

随便找个客户端调用一下就知道了