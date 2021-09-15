---
layout:     post
title:      "精简开源测试框架ApiSuperTest"
subtitle:   ""
date:       2021-08-01 15:27:00
author:     "WS"
header-img: "img/super.jpg"
catalog:    true
tags:
    - Java
    - Api Test
---

## ApiSuperTest介绍

#### 开发背景:

1. 公司使用的测试框架耦合度太高, 且难于维护, 由于是基于专门咨询公司定制而来, 后期修改集成新的技术时也很繁琐, 需要反复修改咨询公司提供的定制包代码, 急需一个取代老框架的测试框架;
2. 前几年测试的UI,API测试框架, 比较多的都是基于TestNG, Junit等测试框架, 然后再逐步添加各项需要的功能, 比如用数据库时加mybatis各种配置, 发送http请求时封装jdk原生, 产出报告时要么自己拼接结果然后组装html, 要么用一些比较老旧的报告框架比如ReportNG等, 相对于开发技术的发展, 测试技术也需要与时俱进;
3. UI/API测试框架, 应该能很好的使用相同的配置, 减少开发资源, 两者的区别就是UI测试框架多一个底层驱动来操作APP, 比如最知名的Appium, 其它的开发配置完全和API框架使用一致即可

综上, 结合现有成熟技术, 新的测试框架的开发体系选择直接在SpringBoot项目上开发, 当然, 这里需要的是SpringBoot非web项目, 因为不需要启动tomcat容器等web资源, 咱们需要的是IOC容器和spring简单易用的技术集成.

开源地址: [WillingSasi/ApiSuperTest: ApiSuperTest (github.com)](https://github.com/WillingSasi/ApiSuperTest)

#### 框架图:

![javascript](/img/FlowChart.png)

#### 框架配置说明:

1. 能用注解的就用注解，比如springboot全家桶中的各种服务注解，mybatis中的数据注解，testng测试定义注解，allure报告的各种层级定义注解，还引入Lombok，能尽量节省手写不重要代码的资源
2. application.yaml配置通过@Value或者@ConfigurationProperties直接引入变量中
3. 一些配置在配置中心Apollo上，这是携程开源的一个服务配置中心，需要的可搜索介绍配置文档，本项目只引入必要配置，通过@EnableApolloConfig开启，用@Value按变量规则引入即可，当然，你得首先在application中定义好Apollo服务地址

> 框架纯个人开发，且公司敏感信息已全部脱敏，故开源万岁！！！

