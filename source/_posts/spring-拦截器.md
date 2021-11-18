---
title: spring 拦截器
date: 2016-10-09 22:22:36
tags:
    - Spring
categories:
    - 技术
typora-root-url: ../../source
typora-copy-images-to: ../../source/img
---

## 处理器拦截器简介
Spring Web MVC的处理器拦截器（如无特殊说明，下文所说的拦截器即处理器拦截器）
类似于Servlet开发中的过滤器Filter，用于对处理器进行预处理和后处理。
## 常见应用场景
1. 日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。
2. 权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；
3. 性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；
4. 通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。
5. OpenSessionInView：如Hibernate，在进入处理器打开Session，在完成后关闭Session。


…………本质也是AOP（面向切面编程），也就是说符合横切关注点的所有功能都可以放入拦截器实现。

<!-- more -->

## 拦截器接口

```java
public interface HandlerInterceptor {
    boolean preHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;
 
    void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception;
 
    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception;
}
```

1. preHandle：

* 预处理回调方法，实现处理器的预处理（如登录检查），第三个参数为响应的处理器（如Controller实现）；
* 返回值：true表示继续流程（如调用下一个拦截器或处理器）；false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应。

2. postHandle：后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。
3. afterCompletion：整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion。

## 拦截器适配器
有时候我们可能只需要实现三个回调方法中的某一个，如果实现HandlerInterceptor接口的话，三个方法必须实现，不管你需不需要，此时spring提供了一个HandlerInterceptorAdapter适配器（一种适配器设计模式的实现），允许我们只实现需要的回调方法。

```java
public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
    public HandlerInterceptorAdapter() {
    }
 
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }
 
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }
 
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }
 
    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    }
}
```

## 运行流程图
### 正常流程
![正常流程](/img/%E6%AD%A3%E5%B8%B8%E6%B5%81%E7%A8%8B.png)
### 中断流程
![中断流程](/img/%E4%B8%AD%E6%96%AD%E6%B5%81%E7%A8%8B.png)

中断流程中，比如是HandlerInterceptor4中断的流程（preHandle返回false），此处仅调用它之前拦截器的preHandle返回true的afterCompletion方法。

## 参考
http://jinnianshilongnian.iteye.com/blog/1670856

