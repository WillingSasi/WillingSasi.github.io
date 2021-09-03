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



## 4.Java Attach



## 5.JVMTI



