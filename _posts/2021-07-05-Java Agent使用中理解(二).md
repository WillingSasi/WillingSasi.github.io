---
layout:     post
title:      "Java Agent使用中理解"
subtitle:   "Java Agent"
date:       2021-07-05 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Java
    - JVM
    - Agent
    - AOP
---

## 1.背景介绍

  Jvm Sandbox中底层就是一种agent的上层详细实现, 其实基于此的上层实现已经有很多, 比如idea中的热部署功能, lombok自动添加, skywalking等, 都是一种aop, 以下按照技术演变, 依次说明java Intrumentation, java agent, attach, JVMTI

## 2.Java Intrumentation

  源码位于java.lang.instrument, 主要是Instrumentation的实现.

   Instrumentation提供了控制Java程序代码的方法，`Instrumentation`可以实现在方法插入额外的字节码从而达到收集使用中的数据到指定工具的目的。由于插入的字节码是附加的，这些更变不会修改原来程序的状态或者行为。例如可以使开发者可以通过Java语言来操作和监控JVM内部的一些状态，进而实现Java程序的监控分析，甚至实现一些特殊功能（如AOP、热部署）, 良性工具包括监控代理、分析器、覆盖分析程序和事件日志记录程序等等

`java.lang.instrument`包的最大功能就是可以**在已有的类上附加（修改）字节码来实现增强的逻辑**

## 3.Java Agent

  Java agent是一种特殊的Java程序（Jar文件），它是Instrumentation的客户端。与普通Java程序通过main方法启动不同，agent并不是一个可以单独启动的程序，而必须依附在一个Java应用程序（JVM）上，与它运行在同一个进程中，通过Instrumentation API与虚拟机交互。

 Java agent与Instrumentation密不可分，二者也需要在一起使用。因为Instrumentation的实例会作为参数注入到Java agent的启动方法中。

  如果想自己写一个java agent程序，只需定义一个包含premain或者agentmain的类，在方法中实现你的逻辑，然后在打包jar时配置一下manifest即可

两种使用方式:

- 启动时加载：通过vm的启动参数-javaagent:**.jar来启动
- 启动后加载：在vm启动后的任何时间点，通过attach api，动态地启动agent 

网上流程图:

![javascript](/img/agent.png)

## 4.Java Attach

  Attach API可不仅仅是为了实现动态加载agent，**Attach API其实是跨JVM进程通讯的工具**，能够将某种指令从一个JVM进程发送给另一个JVM进程. 加载agent只是Attach API发送的各种指令中的一种， 诸如jstack打印线程栈、jps列出Java进程、jmap做内存dump等功能，都属于Attach API可以发送的指令

## 5.JVMTI

  JVM Tool Interface(JVMTI)是JVM提供的native编程接口，开发者可以通过JVMTI向JVM监控状态、执行指令，其目的是开放出一套JVM接口用于 profile、debug、监控、线程分析、代码覆盖分析等工具。JVMTI和Instumentation API的作用很相似，都是**一套JVM操作和监控的接口**，且都**需要通过agent来启动**



> 参考链接:
>
> [Java Agent入门实战（一）-Instrumentation介绍与使用 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1580382)
>
> [谈谈Java Intrumentation和相关应用 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1633977?from=article.detail.1580382)
>
> [IDEA + maven 零基础构建 java agent 项目 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1602018?from=article.detail.1580382)
>
> [Java Agent入门实战（三）-JVM Attach原理与使用 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1574129?from=article.detail.1580382)
>
> 
