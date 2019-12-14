title: springboot接口统一封装返回实现
date: 2019-11-30 14:48:39
tags: [springboot，统一封装返回]
categories: [综合]
---
### 封装springboot返回

优雅的方式就是使用这种方式 最常见的是直接接口使用ResponseView.success()...这种方式 但是下面这种方式更优雅

yapi上传插件也特地为此方式封装了统一返回的格式

太完美了不说了 试试就知道了 非常好用接口层很清晰呀

<!--more-->

1. 拦截请求，是否此请求返回的值需要包装，其实就是运行的时候，解析@ResponseResult注解

```java
import lombok.extern.slf4j.Slf4j;
import net.transino.bl.annotation.ResponseResult;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.lang.reflect.Method;

/**
 * @author maxiucheng
 * @className ResponseResultInterceptor
 * @description 拦截请求，是否此请求返回的值需要包装，其实就是运行的时候，解析@ResponseResult注解
 * @date 2019/11/29 6:03 下午
 * @menu
 **/
@Slf4j
@Component
public class ResponseResultInterceptor implements HandlerInterceptor {

    /**
     * 标记名称
     */
    public static final String RESPONSE_RESULT_ANN = "RESPONSE_RESULT_ANN";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //请求的方法
        if (handler instanceof HandlerMethod){
            final HandlerMethod handlerMethod = (HandlerMethod) handler;
            final Class<?> clazz = handlerMethod.getBeanType();
            // TODO: 2019/11/29 可以做个缓存不需要每次都反射操作的 
            final Method method = ((HandlerMethod) handler).getMethod();
            //判断是否在类对象上加了注解 或者方法体上有注解
            if (clazz.isAnnotationPresent(ResponseResult.class) || method.isAnnotationPresent(ResponseResult.class)){
                //设置此请求返回体，需要包装，往下传递，在ResponseBodyAdvice接口进行判断
                request.setAttribute(RESPONSE_RESULT_ANN, clazz.getAnnotation(ResponseResult.class));
            }
        }
        return true;
    }

}
```

2. 重写返回体

```java
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import lombok.extern.slf4j.Slf4j;
import net.transino.bl.annotation.ResponseResult;
import net.transino.bl.utils.ResponseView;
import org.springframework.core.MethodParameter;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice;

import javax.servlet.http.HttpServletRequest;

/**
 * @author maxiucheng
 * @className ResponseResultHandler
 * @description 重写返回体
 * @date 2019/11/29 8:36 下午
 * @menu
 **/
@Slf4j
@ControllerAdvice
public class ResponseResultHandler implements ResponseBodyAdvice<Object> {

    /**
     * 标记名称
     */
    public static final String RESPONSE_RESULT_ANN = "RESPONSE_RESULT_ANN";

    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> aClass) {
        //是否请求包含了包装注解 没有直接返回 如果有就重写返回体
        ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = sra.getRequest();
        //判断请求是否包含标记
        ResponseResult responseResultAnn = (ResponseResult) request.getAttribute(RESPONSE_RESULT_ANN);
        return responseResultAnn == null ? false : true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter methodParameter, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        log.info("override responseview starting");
        if (!(body instanceof ResponseView)) {
            //如果无法转换统一返回说明请求过程是成功的
            return ResponseView.success(body);
        }
        //肯定可以转换ResponseView 其实可以直接else返回body 但是可能以后会有多种情况所以这么写的
        ResponseView responseView = (ResponseView) body;
        Object clazz = responseView.getResult();
        //分页特殊处理 因为要配合yapi生成文档 所以必须特殊处理
        if (clazz instanceof Page) {
            return body;
        } else {
            //如果请求体包含ResponseView说明进入异常处理过了
            return body;
        }
    }

}
```

3. 拦截器注册

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;

/**
 * @author maxiucheng
 * @className WebAppConfig
 * @description 拦截器注册
 * @date 2019/11/29 9:17 下午
 * @menu
 **/
@Configuration
public class WebAppConfig extends WebMvcConfigurationSupport {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册自定义拦截器，添加拦截路径和排除拦截路径
        registry.addInterceptor(new ResponseResultInterceptor()).addPathPatterns("/api/**").excludePathPatterns("/api/login");
        super.addInterceptors(registry);
    }

}
```

4. 通用返回包装

```java
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

/**
 * 通用返回包装
 *
 * @param <T>
 */
@Data
@NoArgsConstructor
public class ResponseView<T> implements Serializable {

	/**
	 * 是否正确返回
	 */
	private boolean success;

	/**
	 * 返回码
	 */
	private int code;

	/**
	 * 返回信息
	 */
	private String message;

	/**
	 * 返回结果
	 */
	private T result;

	/**
	 * 拼装输出View的MAP对象
	 * @param result 结果
	 * @param success 是否成功
	 * @param code 状态码
	 * @param message 信息
	 */
	public ResponseView(T result , boolean success , int code, String message) {
		this.result = result;
		this.success = success;
		this.code = code;
		this.message = message;
	}

	/**
	 * 返回success为true的map对象
	 *
	 * @param result 响应map对象
	 * @return 返回success为true的对象
	 */
	public static ResponseView success(Object result) {
		return new ResponseView(result, Boolean.TRUE, 200, "");
	}
	/**
	 * @param message 消息
	 * @return 返回success为false的结果
	 */
	public static ResponseView fail(String message) {
		return new ResponseView((Object)null, Boolean.FALSE, 200, message);
	}
	/**
	 * @param message 消息
	 * @return 返回success为false的结果
	 */
	public static ResponseView fail(Object result, String message) {
		return new ResponseView(result, Boolean.FALSE, 200, message);
	}
	/**
	 * @param message 消息
	 * @return 返回success为false的结果
	 */
	public static ResponseView fail(Integer code, String message) {
		return new ResponseView((Object)null, Boolean.FALSE, code, message);
	}

}
```

5. 业务异常封装

```java
import lombok.Data;

/**
 * @author maxiucheng
 * @className ExtensionException
 * @description 业务异常封装
 * @date 2019/11/30 11:38 上午
 * @menu
 **/
@Data
public class BusinessException extends RuntimeException {

    /**
     * 业务异常码 200成功 500失败
     */
    private int code = 200;

    /**
     * 业务异常信息
     *
     */
    private String message;

    /**
     * 业务异常封装 错误码和错误信息
     *
     * @param code 异常码
     * @param message 错误信息
     */
    public BusinessException(int code ,String message) {
        this.code = code;
        this.message = message;
    }

    /**
     * 业务异常封装 错误信息
     *
     * @param message
     */
    public BusinessException(String message) {
        this.message = message;
    }

}
```

6. 异常处理枚举

```java
import lombok.Getter;

/**
 * @author maxiucheng
 * @className EnumExceptionMessageWebMvc
 * @description 异常处理枚举
 * @date 2019/11/30 11:05 上午
 * @menu
 **/
@Getter
public enum EnumExceptionMessageWebMvc {

    // 非预期异常
    UNEXPECTED_ERROR("服务发生非预期异常，请联系管理员！"),
    PARAM_VALIDATED_UN_PASS("参数校验(JSR303)不通过，请检查参数或联系管理员！"),
    NO_HANDLER_FOUND_ERROR("404未找到对应的处理器，请检查 API 或联系管理员！"),
    HTTP_REQUEST_METHOD_NOT_SUPPORTED_ERROR("不支持的请求方法，请检查 API 或联系管理员！"),
    HTTP_MEDIA_TYPE_NOT_SUPPORTED_ERROR("不支持的互联网媒体类型，请检查 API 或联系管理员");

    private final String message;

    EnumExceptionMessageWebMvc(String message) {
        this.message = message;
    }

}
```

7. 全局异常处理

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.HttpMediaTypeNotSupportedException;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.NoHandlerFoundException;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.text.MessageFormat;

/**
 * @author 马秀成
 * @date 2019/10/28
 * @jdk.version 1.8
 * @desc 全局异常处理
 */
@ControllerAdvice
@Slf4j
@RestController
public class GlobalExceptionHandler {

    /**
     * 针对业务异常的处理
     *
     * @param exception 业务异常
     * @param request   http request
     * @param response  http response
     * @return 异常处理结果
     */
    @ExceptionHandler(value = BusinessException.class)
    public ResponseView extensionException(BusinessException exception,
                                           HttpServletRequest request, HttpServletResponse response) {
        log.warn("请求发生了预期异常，出错的 url [{}]，出错的描述为 [{}]",
                request.getRequestURL().toString(), exception.getMessage());
        return ResponseView.fail(exception.getMessage());
    }

    /**
     * 空指针异常
     *
     * @param exception 异常信息
     * @return
     */
    @ExceptionHandler(value = NullPointerException.class)
    public ResponseView nullPointerErrorHandler(Exception exception) {
        exception.printStackTrace();
        return ResponseView.fail("空指针异常");
    }

    /**
     * 针对spring web 中的异常的处理
     *
     * @param exception Spring Web 异常
     * @param request   http request
     * @param response  http response
     * @return 异常处理结果
     */
    @ExceptionHandler(value = {
            NoHandlerFoundException.class,
            HttpRequestMethodNotSupportedException.class,
            HttpMediaTypeNotSupportedException.class
    })
    public ResponseView springWebExceptionHandler(Exception exception, HttpServletRequest request, HttpServletResponse response) {
        log.error(MessageFormat.format("请求发生了非预期异常，出错的 url [{0}]，出错的描述为 [{1}]",
                request.getRequestURL().toString(), exception.getMessage()), exception);
        if (exception instanceof NoHandlerFoundException) {
            return ResponseView.fail(EnumExceptionMessageWebMvc.NO_HANDLER_FOUND_ERROR.getMessage());
        } else if (exception instanceof HttpRequestMethodNotSupportedException) {
            return ResponseView.fail(EnumExceptionMessageWebMvc.HTTP_REQUEST_METHOD_NOT_SUPPORTED_ERROR.getMessage());
        } else if (exception instanceof HttpMediaTypeNotSupportedException) {
            return ResponseView.fail(EnumExceptionMessageWebMvc.HTTP_MEDIA_TYPE_NOT_SUPPORTED_ERROR.getMessage());
        } else {
            return ResponseView.fail(EnumExceptionMessageWebMvc.UNEXPECTED_ERROR.getMessage());
        }
    }

    /**
     * 针对全局异常的处理
     *
     * @param exception 全局异常
     * @param request   http request
     * @param response  http response
     * @return 异常处理结果
     */
    @ExceptionHandler(value = Throwable.class)
    public ResponseView throwableHandler(Exception exception, HttpServletRequest request, HttpServletResponse response) {
        log.error(MessageFormat.format("请求发生了非预期异常，出错的 url [{0}]，出错的描述为 [{1}]",
                request.getRequestURL().toString(), exception.getMessage()), exception);
        return ResponseView.fail(EnumExceptionMessageWebMvc.UNEXPECTED_ERROR.getMessage());
    }

}
```

8. 接口使用方法

```java
public User selectUser(){
    return new User();
}
```

接口会带上ResponseView格式统一返回

9. 效果是这样的

```bash
{
    "success": false,
    "code": 200,
    "message": "success",
    "result": {"id": "1233231231231"}
}
```