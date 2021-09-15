---
layout:     post
title:      "Jacoco搭配springMVC maven tomcat项目,单元测试&接口测试踩坑"
subtitle:   "Jacoco"
date:       2020-04-22 15:27:00
author:     "WS"
header-img: "img/jacoco.jpg"
catalog:    true
tags:
    - Jacoco
    - Java
---

jacoco 代码覆盖率工具,可以集成ant,maven,jenkins等.分别调试了jacoco用于单元测试和接口测试,大概总结下:

 

1. idea自带插件,或者maven pom引入,这两种引入jacoco的方式:

   1.只能用于本地项目路径下的代码测试,一般用于单元测试

   2.相关jacoco的包都可以在maven仓库引入

   3.maven项目只需要配置pom文件就可以

   4.如果只是用于单元测试,spring的tomcat都不用启动,只执行test路径的单元测试代码就可以

2. 配置idea启动配置的VM potions,或者修改tomcat本地cataline文件(如果服务是部署tomcat):

   1.一般用于接口测试或者功能测试覆盖率统计,浏览器,postman等工具调用的springMVC接口都会计入代码覆盖率,区别maven方式的jacoco

   2.jacoco通过jvm启动会开启一个服务端口,随着spring项目启动而一直监听

   3.jacococli.jar指定开启agent端口,通过命令可以获取当前项目运行中的覆盖率文件,解析文件即可得报告

   4.坑:此种方式的jacococli.jar需要用官网的包,maven仓库中的org.jacococli.jar无法使用

 

附:讲解jacoco原理的一篇文章,讲的很全,有条理,来自腾讯.

https://mp.weixin.qq.com/s?__biz=MzIxNzEyMzIzOA==&mid=2652314564&idx=1&sn=a93e6154c92acaef9204b8440e66a852&scene=21#wechat_redirect

