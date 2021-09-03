---
layout:     post
title:      "Java web之Jetty是个啥?"
subtitle:   "Jetty"
date:       2021-07-18 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Java
    - Jetty
---

## 1.源

  学习sandbox项目中发现有说内部开启了一个jetty服务, 在调用sandbox的时, 不仅可以直接用命令, 也都可走http请求, 这不就是内置tomcat?

  答对了一半, jetty的功能和tomcat很类似, 都属于java web范畴技术, 都是servlet的容器，都符合servlet标准, 也就是提供给外部访问的入口, 只不过设计理念不同, 实现也有所差异. 

## 2.特点对比

Jetty架构图

![javascript](/img/jetty-1.png)

Tomcat架构图

![javascript](/img/jetty-2.gif)

对比:

> 架构: jetty相对简单轻量, 实现调用都是基于handler, 更多用作程序内嵌服务;
>
> ​          tomcat开启实现的是一个容器, 一般作为单独的服务开启载入程序运行.
>
> 性能: jetty的性能优势在于处理长链接请求上, 例如Web聊天应用非常适合用Jetty服务器, 默认采用NIO处理
>
> ​          I/O请求, 在处理静态资源时, 性能较高;
>
> ​          tomcat在处理少数非常繁忙的连接上更有优势, 也就是连接的生命周期如果比较短, 默认采用BIO处理         
>
> ​          I/O请求, 在处理静态资源时 性能较差.

总的来说就是, 没有绝对的优缺点, 任何技术的出现都是为了解决一类的问题!!!

## 3.Jetty demo

1. 引入依赖, 最新已出到11.0.x系列

   ```xml
           <dependency>
               <groupId>org.eclipse.jetty</groupId>
               <artifactId>jetty-server</artifactId>
           </dependency>
           
           <dependency>
               <groupId>org.eclipse.jetty</groupId>
               <artifactId>jetty-servlet</artifactId>
           </dependency>
   ```

2. HttpServlet继承重写

   ```java
   public class ExampleServlet extends HttpServlet {
       @Override
       protected void doGet(HttpServletRequest req, HttpServletResponse resp)
               throws ServletException, IOException {
   
           resp.setStatus(HttpStatus.OK_200);
           resp.getWriter().println("EmbeddedJetty");
       }
   }
   ```

3. main handler

   ```java
   public class EmbeddedJettyMain{
       public static void main(String[] args) throws Exception {
           Server server = new Server(7070);
           ServletContext handler = new ServletContext(server, "/example");
           handler.addServlet(ExampleServlet.class, "/");
           server.start();
       }
   }
   ```

4. 启动后在浏览器访问以上定义好的路径即可

## 4.Jvm Sandbox源码中Jetty服务

1. jetty服务所在位置

   > ```shell
   > com.alibaba.jvm.sandbox.core.server.jetty
   > ```

2. 初始化代码, 也就是sandbox http访问路径调用路径规则定义部分

   ```java
     /*
        * 初始化Jetty's ContextHandler
        */
       private void initJettyContextHandler() {
           final String namespace = cfg.getNamespace();
           final ServletContextHandler context = new ServletContextHandler(NO_SESSIONS);
   
           final String contextPath = "/sandbox/" + namespace;
           context.setContextPath(contextPath);
           context.setClassLoader(getClass().getClassLoader());
   
           // web-socket-servlet
           final String wsPathSpec = "/module/websocket/*";
           logger.info("initializing ws-http-handler. path={}", contextPath + wsPathSpec);
           //noinspection deprecation
           context.addServlet(
                   new ServletHolder(new WebSocketAcceptorServlet(jvmSandbox.getCoreModuleManager())),
                   wsPathSpec
           );
   
           // module-http-servlet
           final String pathSpec = "/module/http/*";
           logger.info("initializing http-handler. path={}", contextPath + pathSpec);
           context.addServlet(
                   new ServletHolder(new ModuleHttpServlet(cfg, jvmSandbox.getCoreModuleManager())),
                   pathSpec
           );
   
           httpServer.setHandler(context);
       }
   ```

3. sandbox 实际命令行操作和http请求对应可参考链接:

   [4. jvm-sandbox之调用方式(命令行和http) - 月色深潭 - 博客园 (cnblogs.com)](https://www.cnblogs.com/moonpool/p/14510443.html)

   
