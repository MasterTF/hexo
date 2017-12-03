---
title: springmvc 请求处理过程
date: 2016-11-03 22:03:07
tags:
    - Spring
categories:
    - 技术
    - Spring
---

一个请求到达服务器之后，springmvc处理过程：
![springmvc 请求处理过程](/img/springmvc%20%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B.png)

<!-- more -->
* (1) **Http请求**：客户端请求提交到DispatcherServlet。 
* (2) **寻找处理器**：由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller。 
* (3) **调用处理器**：DispatcherServlet将请求提交到Controller。 
* (4)(5)**调用业务处理和返回结果**：Controller调用业务逻辑处理后，返回ModelAndView。 
* (6)(7)**处理视图映射并返回模型**： DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图。 
* (8) **Http响应**：视图负责将结果显示到客户端。


图中有些过程没有说清楚，比如怎么找到HandlerMapping以及业务逻辑代码怎么被调用？
这就需要看DispatcherServlet源码了。
先上一段关键代码，DispatcherServlet.doDispatcher() 方法,核心流程都在这个方法里。
## DispatcherServlet.doDispatch
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;
 
   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
 
   try {
      ModelAndView mv = null;
      Exception dispatchException = null;
 
      try {
         processedRequest = checkMultipart(request);
         multipartRequestParsed = (processedRequest != request);
 
         // Determine handler for the current request.
         mappedHandler = getHandler(processedRequest);
         if (mappedHandler == null || mappedHandler.getHandler() == null) {
            noHandlerFound(processedRequest, response);
            return;
         }
 
         // Determine handler adapter for the current request.
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
 
         // Process last-modified header, if supported by the handler.
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (logger.isDebugEnabled()) {
               logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
            }
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }
 
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }
 
         // Actually invoke the handler.
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }
 
         applyDefaultViewName(processedRequest, mv);
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Error err) {
      triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
         // Clean up any resources used by a multipart request.
         if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```

第一个出现的核心接口：getHandler()

```java
mappedHandler = getHandler(processedRequest);
```

它是在org.springframework.web.servlet包中定义的HandlerMapping接口：

### HandlerMapping

```java
package org.springframework.web.servlet;
 
import javax.servlet.http.HttpServletRequest;
 
public interface HandlerMapping {
 
    String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";
 
    String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";
 
    String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";
 
    String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";
 
    String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";
 
    HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
 
} 
```

为了阅读方便，去掉了源码中的注释，类中定义的几个常量先不管。关键在于这个接口中唯一的方法：

```java
HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
```

回到DispatcherServlet的处理流程，当DispatcherServlet接收到web请求后，由标准Servlet类处理方法doGet或者doPost，经过几次转发后，遍历DispatcherServlet中所有
HandlerMapping实现类（容器启动时会将实现类注入进来，看源码！）。
web请求的HttpServletRequest对象为参数，依次调用HandlerMapping实现类的getHandler方法，第一个不为null的作为结果返回。
DispatcherServlet类中的这个遍历方法不长，贴一下:

### getHandler()
```java
/** * Return the HandlerExecutionChain for this request. * <p>Tries all handler mappings in order. * @param request current HTTP request * @return the HandlerExecutionChain, or <code>null</code> if no handler could be found */
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        for (HandlerMapping hm : this.handlerMappings) {
            if (logger.isTraceEnabled()) {
                logger.trace(
                        "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
            }
            HandlerExecutionChain handler = hm.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
        return null;
    }
```

可以看到，一个web请求经过处理后，会得到一个HandlerExecutionChain对象，这就是SpringMVC对URl映射给出的回答。
需要留意的是，HandlerMapping接口的getHandler方法参数是HttpServletRequest，这意味着，HandlerMapping的实现类可以利用HttpServletRequest中的 所有信息来做出这个HandlerExecutionChain对象的生成”决策“。这包括，请求头、url路径、cookie、session、参数等等一切你从一个web请求中可以得到的任何东西（最常用的是url路径）。
getHandler()的具体实现有兴趣饿的可以跟进去看下。


HandlerExecutionChain是我们接触的第二个核心类，从名字可以直观的看得出，这个对象是一个执行链的封装。
HandlerExecutionChain类的代码不长，它定义在org.springframework.web.servlet包中， 为了更直观的理解，先上代码:
### HandlerExecutionChain

```java
package org.springframework.web.servlet;
 
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
 
import org.springframework.util.CollectionUtils;
 
public class HandlerExecutionChain {
 
    private final Object handler;
 
    private HandlerInterceptor[] interceptors;
 
    private List<HandlerInterceptor> interceptorList;
 
    public HandlerExecutionChain(Object handler) {
        this(handler, null);
    }
 
    public HandlerExecutionChain(Object handler, HandlerInterceptor[] interceptors) {
        if (handler instanceof HandlerExecutionChain) {
            HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
            this.handler = originalChain.getHandler();
            this.interceptorList = new ArrayList<HandlerInterceptor>();
            CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
            CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
        }
        else {
            this.handler = handler;
            this.interceptors = interceptors;
        }
    }
 
    public Object getHandler() {
        return this.handler;
    }
 
    public void addInterceptor(HandlerInterceptor interceptor) {
        initInterceptorList();
        this.interceptorList.add(interceptor);
    }
 
    public void addInterceptors(HandlerInterceptor[] interceptors) {
        if (interceptors != null) {
            initInterceptorList();
            this.interceptorList.addAll(Arrays.asList(interceptors));
        }
    }
 
    private void initInterceptorList() {
        if (this.interceptorList == null) {
            this.interceptorList = new ArrayList<HandlerInterceptor>();
        }
        if (this.interceptors != null) {
            this.interceptorList.addAll(Arrays.asList(this.interceptors));
            this.interceptors = null;
        }
    }
 
    public HandlerInterceptor[] getInterceptors() {
        if (this.interceptors == null && this.interceptorList != null) {
            this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[this.interceptorList.size()]);
        }
        return this.interceptors;
    }
 
    @Override
    public String toString() {
        if (this.handler == null) {
            return "HandlerExecutionChain with no handler";
        }
        StringBuilder sb = new StringBuilder();
        sb.append("HandlerExecutionChain with handler [").append(this.handler).append("]");
        if (!CollectionUtils.isEmpty(this.interceptorList)) {
            sb.append(" and ").append(this.interceptorList.size()).append(" interceptor");
            if (this.interceptorList.size() > 1) {
                sb.append("s");
            }
        }
        return sb.toString();
    }
```

关注两行即可知道这个类的作用：

```java
private final Object handler;
 
private HandlerInterceptor[] interceptors;
```

一个实质执行对象，还有一堆拦截器。
HandlerInterceptor也是SpringMVC的核心接口，定义如下：

### HandlerInterceptor

```java
package org.springframework.web.servlet;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
public interface HandlerInterceptor {
 
    boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception;
 
    void postHandle(
            HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
            throws Exception;
 
    void afterCompletion(
            HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception;
 
}
```

拦截器就是我们定义的interceptor，关于拦截器的讲解在 后续博客中
至此，HandlerExecutionChain整个执行脉络也就清楚了：在真正调用其handler对象前，HandlerInterceptor接口实现类组成的数组将会被遍历，其preHandle方法会被依次调用，然后真正的handler对象将被调用。
handler对象被调用后，就生成了需要的响应数据，在将处理结果写到HttpServletResponse对象之前（SpringMVC称为渲染视图），其postHandle方法会被依次调用。视图渲染完成后，最后afterCompletion方法会被依次调用，整个web请求的处理过程就结束了。
 
这个HandlerExecutionChain类中以Object引用所声明的handler对象，到底是什么？它是怎么被调用的？
继续看DispatcherServlet.doDispatch()方法24行：

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

### getHandlerAdapter()：

```java
/**
 * Return the HandlerAdapter for this handler object.
 * @param handler the handler object to find an adapter for
 * @throws ServletException if no HandlerAdapter can be found for the handler. This is a fatal error.
 */
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
   for (HandlerAdapter ha : this.handlerAdapters) {
      if (logger.isTraceEnabled()) {
         logger.trace("Testing handler adapter [" + ha + "]");
      }
      if (ha.supports(handler)) {
         return ha;
      }
   }
   throw new ServletException("No adapter for handler [" + handler +
         "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

这段代码已经很明显了，HandlerExecutionChain中的handler对象会被作为参数传递进去，在DispatcherServlet类中注册的HandlerAdapter实现类列表会被遍历，然后返回第一个supports方法返回true的HandlerAdapter对象。
获得到HandlerAdapter对象后，调用handle方法处理handler对象，并返回ModelAndView这个包含了视图和数据的对象。

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

ModelAndView对象的代码就不贴了，它是SpringMVC中对视图和数据的一个聚合类。其中的视图，就是由SpringMVC的最后一个核心接口View所抽象：

### handle

```java
package org.springframework.web.servlet;
 
import java.util.Map;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
public interface View {
 
    String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";
 
    String PATH_VARIABLES = View.class.getName() + ".pathVariables";
 
    String getContentType();
 
    void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```
所有的数据，最后会作为一个Map对象传递到View实现类中的render方法，调用这个render方法，就完成了视图到响应的渲染。这个View实现类，就是来自HandlerAdapter中的handle方法的返回结果。当然从ModelAndView到真正的View实现类有一个解析的过程，ModelAndView中可以有真正的视图对象，也可以只是有一个视图的名字，SpringMVC会负责将视图名称解析为真正的视图对象。

至此，我们了解了一个典型的完整的web请求在SpringMVC中的处理过程和其中涉及到的核心类和接口。

在一个典型的SpringMVC调用中，HandlerExecutionChain中封装handler对象就是用@Controller注解标识的类的一个实例，根据类级别和方法级别的@RequestMapping注解，由默认注册的DefaultAnnotationHandlerMapping（3.1.3中更新为RequestMappingHandlerMapping类，但是为了向后兼容，DefaultAnnotationHandlerMapping也可以使用）生成HandlerExecutionChain对象，再由AnnotationMethodHandlerAdapter（3.1.3中更新为RequestMappingHandlerAdapter类，但是为了向后兼容，AnnotationMethodHandlerAdapter也可以使用）来执行这个HandlerExecutionChain对象，生成最终的ModelAndView对象后，再由具体的View对象的render方法渲染视图。


