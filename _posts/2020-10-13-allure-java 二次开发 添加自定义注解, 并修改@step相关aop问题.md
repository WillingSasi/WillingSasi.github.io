---
layout:     post
title:      "allure-java 二次开发 添加自定义注解, 并修改@step相关aop问题"
subtitle:   "allure"
date:       2020-10-13 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Allure
    - Java
    - AOP
---

由于定制化需求, 需要对allure-java一些注解和aop方法进行修改. 按预期修改打包引入测试工程, 运行报错:

circular advice precedence: can't determine precedence between two or more p

 

问题点在于@after @before @around 顺序, allure-java原方法中没有@around, 顺序是:

  @Pointcut -> @Before -> @AfterThrowing -> @AfterReturning

 

解决办法是修改顺序:

Change the advice order to after/before/around:

 

然后重新打包引入项目, 确实没有aspect编译报错, 项目也能按预期切入点运行, 根据报错提醒找到貌似是aspect的bug? 还是aspect的规则? 相关链接如下:

https://www.eclipse.org/lists/aspectj-users/msg11018.html

https://bugs.eclipse.org/bugs/show_bug.cgi?id=30439

https://bugs.eclipse.org/bugs/show_bug.cgi?id=40655

