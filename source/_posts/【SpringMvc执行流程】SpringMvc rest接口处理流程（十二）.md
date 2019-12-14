title: 【SpringMvc执行流程】SpringMvc rest接口处理流程（十二）
date: 2019-07-21 22:00:15
tags: [springmvc]
categories: [springmvc]
---
### spring mvc rest 处理流程

根据前面学习的处理流程整理了下

<!--more-->

### 项目搭建

构建一个简单的spingboot项目，如下：

```java
@RestController
public class HelloWorldController {
    @RequestMapping("/")
    public String index(@RequestParam int value) {
        return "index"+ value;
    }
}
```

此项目中只有一个简单的controller接收请求。下面启动项目请求接口，一步步debug走起。

### dispatcherServlet

```java
//伪代码
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
 HandlerExecutionChain getHandler(processedRequest); //获取满足条件的handlerMethod
}
protected HandlerExecutionChain getHandler(HttpServletRequest request) {
		for (HandlerMapping hm : this.handlerMappings) {//通过循环遍历，匹配对应的HandlerMapping
			HandlerExecutionChain handler = hm.getHandler(request); //获取对应的handlerMethod
			if (handler != null) {
				return handler;
			}
}
```

当有请求过来时，执行doDispatch，通过调用getHandler获取请求对应的handlerMethod。debug时，可以在for循环中找到handlerMappings中匹配成功的是RequestMappingHandlerMapping。

```java
//伪代码
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
 HandlerExecutionChain  getHandler(processedRequest); //获取满足条件的handlerMethod
 HandlerAdapter getHandlerAdapter(HandlerExecutionChain.getHandler());//根据handlerMethod获取HandlerAdapter
}
```

当获取到包含handlerMethod的HandlerExecutionChain时，通过HandlerExecutionChain的handler获取HandlerAdapter。debug发现HandlerAdapter为RequestMappingHandlerAdapter。

### HandlerAdapter

执行handle方法去执行具体的方法

```java
//伪代码
ModelAndView invokeHandlerMethod(request, response, handlerMethod) {
  void ServletInvocableHandlerMethod.invokeAndHandle(webRequest, mavContainer);//解析参数和执行请求，返回结果。
  return getModelAndView(mavContainer, modelFactory, webRequest); //返回ModelAndView
}
```

handlerAdapter的意义在于分担DispatherServlet工作量和set一些必要的参数。这里handlerAdapter直接把任务委派给ServletInvocableHandlermethod。

### ServletInvocableHandlermethod

```java
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);//解析参数和执行请求
		HandlerMethodReturnValueHandlerComposite.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);//处理结果
```

执行完controller方法后，进行对结果进行处理。

### HandlerMethodReturnValueHandler

```java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
      ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

   HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
   } //根据返回参数的类型进行不同的处理
```

如上述中，我们使用的是@RestController，由于restController中包含了@ResponseBody，所以这里采用

RequestResponseBodyMethodProcessor直接返回结果。

### RequestResponseBodyMethodProcessor

```java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
		ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
		throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

	mavContainer.setRequestHandled(true);
	ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
	ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

	// Try even with null return value. ResponseBodyAdvice could get involved.
	writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
}
```

接口的处理流程大概就是这样的