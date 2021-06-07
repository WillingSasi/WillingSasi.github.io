---
layout:     post
title:      "Spring Interceptor 拦截器技术细节"
subtitle:   ""
date:       2021-06-06 23:52:00
author:     "WS"
header-img: "img/Interceptor.jpg"
tags:
    - Java
    - Spring
    - Springboot
    - HandlerInterceptor
---



## 背景: 实现后台接口服务主动“降级”, 配置开关在Apollo, 按需动态指定接口返回特定的页面, 比如系统升级时希望相关接口返回系统维护页, 只需打开拦截标记即可



##1. Interceptor拦截器白话概念

> SpringWebMVC的处理器拦截器，类似于Servlet开发中的过滤器Filter，用于处理器进行预处理和后处理。
>
> Spring中的Servlet, 就相当于后端服务请求接口后的一个传递分发站, Request进来后, 都需要由Servlet分发到Handler Mapping处理, 也就是Controller层, 由此再进行各种数据的操作.
>
> HandlerInterceptor就是作用在Servlet这一层的拦截器, 注意这个拦截是全局所有请求的拦截, 可能需要考虑性能问题

## 1. Spring的HandlerInterceptor接口详解

```java
public interface HandlerInterceptor {
  
    //preHandle：拦截于请求刚进入时，进行判断，需要boolean返回值，如果返回true将继续执行，如果返回false，将不进行执行。一般用于登录校验
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    //postHandle：拦截于方法成功返回后，视图渲染前，可以对modelAndView进行操作
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    //afterCompletion：拦截于方法成功返回后，视图渲染前，可以进行成功返回的日志记录
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
```



## 2. 具体实现

#### 1. 自定义注解

```java
/**
 * 在需要拦截返回error code的接口方法上使用此注解
 */
@Target({ ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
public @interface MaintenanceAdapter {
}

```

#### 2. HandlerInterceptorAdapter继承实现, 目前只需要preHandle方法

> 多种实现方式, HandlerInterceptorAdapter需要继承，HandlerInterceptor需要实现
>
> 实际代码中用了HandlerInterceptorAdapter, 能按需方法覆盖

```java
//spring中注意需要申明此类为Configuration
@Configuration
public class MaintenanceInterceptor extends HandlerInterceptorAdapter {

    //注解获取Apollo配置的动态值, 主动配置
    @Value("${maintenance}")
    private boolean maintenance;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        boolean flag = true;
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        MaintenanceAdapter maintenanceAdapter = handlerMethod.getMethodAnnotation(MaintenanceAdapter.class);
        System.out.println(maintenance);
        if (maintenanceAdapter != null || maintenance) {
            //返回自定义错误码, 便于前端捕获处理
            response.setStatus(Errors.NOT_FOUND());
            flag = false;
        }

        return flag;
    }
}
```

#### 3. 注册写好的Interceptor, 也就是配置在WebMvcConfigurer中, Springboot自动发现

```java
//注意也要申明为Configuration
@Configuration
public class InterceptorConf implements WebMvcConfigurer {

    @Autowired
    private MaintenanceInterceptor maintenanceInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(maintenanceInterceptor);
    }
}
```

#### 4. @MaintenanceAdapter 再需要拦截的方法上标记注解即可



题外话: 最后公司实现还是没有采取这种拦截器的实现, 而是用了aop实现, 相比拦截更精准一些, 减少不必要的性能损耗