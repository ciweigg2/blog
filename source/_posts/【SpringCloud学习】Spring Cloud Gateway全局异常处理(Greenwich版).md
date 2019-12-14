title: 【SpringCloud学习】Spring Cloud Gateway全局异常处理(Greenwich版)
date: 2019-07-21 17:25:46
tags: [springcloud]
categories: [springcloud]
---
### 介绍

大家不要试着去用Gateway去捕获后端异常，回到最初的起点，API 网关（API Gateway）主要负责服务请求路由、组合及协议转换，异常同样也是一样，Gateway只负责转发单体应用的异常，不要试图Gateway捕获后端服务异常，然后再输出给前端。

Spring Cloud Gateway中的全局异常处理不能直接用@ControllerAdvice来处理，通过跟踪异常信息的抛出，找到对应的源码，自定义一些处理逻辑来符合业务的需求。

网关都是给接口做代理转发的，后端对应的都是REST API，返回数据格式都是JSON。如果不做处理，当发生异常时，Gateway默认给出的错误信息是页面，不方便前端进行异常处理。

<!--more-->

需要对异常信息进行处理，返回JSON格式的数据给客户端。下面先看实现的代码，后面再跟大家讲下需要注意的地方。

### 自定义异常处理逻辑

自定义异常处理逻辑：

```java
import org.springframework.boot.autoconfigure.web.ErrorProperties;
import org.springframework.boot.autoconfigure.web.ResourceProperties;
import org.springframework.boot.autoconfigure.web.reactive.error.DefaultErrorWebExceptionHandler;
import org.springframework.boot.web.reactive.error.ErrorAttributes;
import org.springframework.context.ApplicationContext;
import org.springframework.http.HttpStatus;
import org.springframework.web.reactive.function.server.*;

import java.util.HashMap;
import java.util.Map;
/**
 * 自定义异常处理
 *
 * <p>异常时用JSON代替HTML异常信息<p>
 *
 * @author ciwei
 *
 */
public class JsonExceptionHandler extends DefaultErrorWebExceptionHandler {
	public JsonExceptionHandler(ErrorAttributes errorAttributes, ResourceProperties resourceProperties,
								ErrorProperties errorProperties, ApplicationContext applicationContext) {
		super(errorAttributes, resourceProperties, errorProperties, applicationContext);
	}
	/**
	 * 获取异常属性
	 */
	@Override
	protected Map<String, Object> getErrorAttributes(ServerRequest request, boolean includeStackTrace) {
		int code = 500;
		Throwable error = super.getError(request);
		if (error instanceof org.springframework.cloud.gateway.support.NotFoundException || error instanceof org.springframework.web.server.ResponseStatusException) {
			code = 404;
		}
		return response(code, this.buildMessage(request, error));
	}
	/**
	 * 指定响应处理方法为JSON处理的方法
	 * @param errorAttributes
	 */
	@Override
	protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
		return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
	}
	/**
	 * 根据code获取对应的HttpStatus
	 * @param errorAttributes
	 */
	@Override
	protected HttpStatus getHttpStatus(Map<String, Object> errorAttributes) {
		int statusCode = (int) errorAttributes.get("code");
		return HttpStatus.valueOf(statusCode);
	}
	/**
	 * 构建异常信息
	 * @param request
	 * @param ex
	 * @return
	 */
	private String buildMessage(ServerRequest request, Throwable ex) {
		StringBuilder message = new StringBuilder("Failed to handle request [");
		message.append(request.methodName());
		message.append(" ");
		message.append(request.uri());
		message.append("]");
		if (ex != null) {
			message.append(": ");
			message.append(ex.getMessage());
		}
		return message.toString();
	}
	/**
	 * 构建返回的JSON数据格式
	 * @param status        状态码
	 * @param errorMessage  异常信息
	 * @return
	 */
	public static Map<String, Object> response(int status, String errorMessage) {
		Map<String, Object> map = new HashMap<>();
		map.put("code", status);
		map.put("message", errorMessage);
		map.put("data", null);
		return map;
	}
}
```

### 覆盖默认的配置

覆盖默认的配置：

```java
import java.util.Collections;
import java.util.List;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.boot.autoconfigure.web.ResourceProperties;
import org.springframework.boot.autoconfigure.web.ServerProperties;
import org.springframework.boot.context.propertiesEnableConfigurationProperties;
import org.springframework.boot.wweb.reactive.error.ErrorAttributes;
import org.springframework.boot.web.reactive.error.ErrorWebExceptionHandler;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.http.codec.ServerCodecConfigurer;
import org.springframework.web.reactive.result.view.ViewResolver;
/**
 * 覆盖默认的异常处理
 * 
 * @author ciwei
 *
 */
@Configuration
@EnableConfigurationProperties({ServerProperties.class, ResourceProperties.class})
public class ErrorHandlerConfiguration {
    private final ServerProperties serverProperties;
    private final ApplicationContext applicationContext;
    private final ResourceProperties resourceProperties;
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;
    public ErrorHandlerConfiguration(ServerProperties serverProperties,
                                     ResourceProperties resourceProperties,
                                     ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                     ServerCodecConfigurer serverCodecConfigurer,
                                     ApplicationContext applicationContext) {
        this.serverProperties = serverProperties;
        this.applicationContext = applicationContext;
        this.resourceProperties = resourceProperties;
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public ErrorWebExceptionHandler errorWebExceptionHandler(ErrorAttributes errorAttributes) {
        JsonExceptionHandler exceptionHandler = new JsonExceptionHandler(
                errorAttributes,
                this.resourceProperties,
                this.serverProperties.getError(),
                this.applicationContext);
        exceptionHandler.setViewResolvers(this.viewResolvers);
        exceptionHandler.setMessageWriters(this.serverCodecConfigurer.getWriters());
        exceptionHandler.setMessageReaders(this.serverCodecConfigurer.getReaders());
        return exceptionHandler;
    }

}
```

### 重点解释呀

* 异常时如何返回JSON而不是HTML?

在org.springframework.boot.autoconfigure.web.reactive.error.DefaultErrorWebExceptionHandler中的getRoutingFunction()方法就是控制返回格式的，原代码如下：

```java
@Override
protected RouterFunction<ServerResponse> getRoutingFunction(
            ErrorAttributes errorAttributes) {
       return RouterFunctions.route(acceptsTextHtml(), this::renderErrorView)
                .andRoute(RequestPredicates.all(), this::renderErrorResponse);
}
```

这边优先是用HTML来显示的，想用JSON的改下就可以了，如下：

protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
       return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
}

* getHttpStatus需要重写

原始的方法是通过status来获取对应的HttpStatus的，代码如下：

```java
protected HttpStatus getHttpStatus(Map<String, Object> errorAttributes) {
      int statusCode = (int) errorAttributes.get("status");
      return HttpStatus.valueOf(statusCode);
}
```

如果我们定义的格式中没有status字段的话，这么就会报错，找不到对应的响应码，要么返回数据格式中增加status子段，要么重写，我这边返回的是code，所以要重写，代码如下：

```java
@Override
protected HttpStatus getHttpStatus(Map<String, Object> errorAttributes) {
    int statusCode = (int) errorAttributes.get("code");
    return HttpStatus.valueOf(statusCode);
}
```

总结：SpringCloudGateway的全局异常捕获是针对路由报错的而不是服务端的报错异常捕获的

比如你有个路由：

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

那么你访问：http://localhost:8080/hello2 就会进入全局异常的处理逻辑了

demo：https://github.com/ciweigg2/spring-cloud-examples/tree/spring-cloud-nacos/spring-cloud-gateway