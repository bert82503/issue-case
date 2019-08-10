

Spring Boot MVC 文件上传错误
======================
> Michael Yang，2017-09-05


## 1.现象
今天有一个项目，是从 tomcat 迁移到 spring boot。

实测中发现，文件上传不进来。
```
public JsonResult createBpmn(@RequestParam Long businessId, @RequestParam(value = "file", required = false) MultipartsFile bpnmFile) {
  // 此时 bpnmFile == null
}
```


## 2.排查
文件上传的处理逻辑是在 `DispatcherServlet.checkMultipart()` 方法
```java
protected HttpServletRequest checkMultipart(HttpServletRequest request) throws MultipartException {
    if (this.multipartResolver != null && this.multipartResolver.isMultipart(request)) {
        if (WebUtils.getNativeRequest(request, MultipartHttpServletRequest.class) != null) {
            logger.debug("Request is already a MultipartHttpServletRequest - if not in a forward, " +
                    "this typically results from an additional MultipartFilter in web.xml");
        }
        else if (request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) instanceof MultipartException) {
            logger.debug("Multipart resolution failed for current request before - " +
                    "skipping re-resolution for undisturbed error rendering");
        }
        else {
            return this.multipartResolver.resolveMultipart(request);
        }
    }
    // If not returned before: return original request.
    return request;
}
```
debug 可以看到当前项目中用来支持文件上传的组件 `multipartResolver` 是 `CommonsMultipartResolver` 类型。

简单看了下这个类型的 resolver 逻辑是从 request 的 inputStream 中读取上传文件。

但是发现这里 request 的 inputStream 是空的。

在 `DispatcherServlet.doDispatch()` 方法开头，也就是所有请求要进行 controller 分发时，request 的 inputStream 也就已经空了。
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    // 此时有断点，使用 IDEA 的 debug 工具执行 IOUtil.toString(request.getInputStream())
    // 发现输出为空
```
这说明 inputStream 被读过了。

然后就在 Request 对象的 getInputStream 方法上打了断点。

发现是在一个 filter 的逻辑里，最终调用了 getInputStream 方法，从调用栈的方法名字们看，也有 uploadFile 字眼。

这个 filter 是 `HiddenHttpMethodFilter`，看了 doc 和代码。
这个 filter 的作用大概是：因为现在浏览器只支持 get 和 post，为了能支持更多的方法，诸如 put，delete 等，
就约定了一个参数名叫 _method 来指定这些不支持的方法，比如 _method=put，就是这次请求在之后的处理中会被当做 put 来处理。

这个 filter 读取了 inputStream，并把文件加入到了 request 的属性 parts 中。
这里的 request 是 tomcat 的实现的，因为内置容器是 tomcat。

这就`有问题`了，`这个 filter 把 inputStream 给读了，后面的 multipartResolver 就读不到了。`

这个 filter 的 doc 中有一段
```
NOTE: This filter needs to run after multipart processing in case of a multipart POST request,
due to its inherent need for checking a POST body parameter.
So typically, put a Spring org.springframework.web.multipart.support.MultipartFilter
before this HiddenHttpMethodFilter in your web.xml filter chain.
```
说我们要把 `MultipartFilter` 写在前面，前置执行。

但是，看了一眼 `HiddenHttpMethodFilter` 是在 `WebMvcAutoConfiguration` 中定义的，
按道理说，这个是标准化的，不应该由我们在做什么处理才能用拿到 upload 的 file。

其实，如果 multipartResolver 如果直接从 request 的 parts 属性中拿 file 不就行了，不从 inputStream 中读。
spring boot 一套应该是把这些都搞定得了。

后来看 `MultipartAutoConfiguration`，里面注册了个 multipartResolver，但类型是 `StandardServletMultipartResolver`。

看了 `StandardServletMultipartResolver` 的实现，果然是从 request 的 parts 属性中取。

所以，**原因**出来了：
是在项目迁移的过程中遗留了 mvc 的 xml 配置，里面配置了 multipartResolver，类型指定为 `CommonsMultipartResolver`。
去掉后就ok了。


## 3.结论
spring mvc 提供了两种 multipartResolver：

1. `CommonsMultipartResolver`：从 request 的 inputStream 中读取文件
2. `StandardServletMultipartResolver`：使用 `HttpServletRequest.getParts()` 方法来获取文件

spring boot 使用自动配置的一个 filter `HiddenHttpMethodFilter` 会比较早的时候读取 request 的 inputStream，导致 `CommonsMultipartResolver` 无法再获取到数据。
所以 spring boot mvc 默认是用 `StandardServletMultipartResolver`，用错 resolver 会导至上传文件读不到。

> `HttpServletRequest.getParts()` 方法是 Servlet 3.0 之后获取 `multipart/form-data` 类型的请求数据的一个标准方法，
> 凡是实现了 Servlet 3.0 的服务器都支持，可以尽量使用 `StandardServletMultipartResolver`。


[原文](https://blog.yangxiaochen.com/blog/java/spring-mvc-multipart-error.html)

