title: 【SpringMvc执行流程】SpringMvc中的View（十）
date: 2019-07-20 12:50:54
tags: [springmvc]
categories: [springmvc]
---
### View接口内容展示

<!--more-->

```java
public interface View {
    String RESPONSE_STATUS_ATTRIBUTE = MyView.class.getName() + ".responseStatus";
    String PATH_VARIABLES = MyView.class.getName() + ".pathVariables";
    //渲染页面
    void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
            throws Exception;
    //获取网页文件的类型和编码
    @Nullable
    default String getContentType() {
        return null;
    }

}
```
### 入口
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```
`processDispatchResult`转发请求
```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable MyHandlerExecutionChain mappedHandler, @Nullable MyModelAndView mv,
                                   @Nullable Exception exception) throws Exception {
    // 渲染页面
    render(mv, request, response);
}
```
`render`进行页面渲染
```java
protected void render(MyModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    //  // 根据请求决定回复消息的 locale
    Locale locale = (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
    response.setLocale(locale);
    View view;
    String viewName = mv.getViewName();
    if (viewName != null) {
        // We need to resolve the view name.
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    view.render(mv.getModelInternal(), request, response);
}
```
* `resolveLocale` 根据请求决定回复消息的 locale
* `resolveViewName`选择合适的视图处理，获取处理后的View。
* `view.render`视图真正的渲染处理。

```java
@Override
public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

    Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
    prepareResponse(request, response);
    renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
}
```
prepareResponse默认是false不需要关注，进入`renderMergedOutputModel`看下具体操作。
```java
protected void renderMergedOutputModel(
        Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        
    String dispatcherPath = prepareForRendering(request, response);
    RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
        rd.forward(request, response);
    }
}
```
* prepareForRendering获取jsp的路径。
* `rd.forward`是servlet提供做转发，到此也就交给servlet去处理view。