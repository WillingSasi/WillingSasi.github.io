---
layout:     post
title:      "为BDD而生的cucumber框架工作实践"
subtitle:   "BDD"
date:       2020-12-22 15:27:00
author:     "WS"
header-img: "img/cucumber.jpg"
catalog:    true
tags:
    - BDD
    - cucumber
---

## Cucumber feature 文件检查清单

- feature文件应面向业务的语言编写
- feature文件应该是客观并基于事实的
- 每个feature文件应该是短小而紧凑的，且只有一个主题
- 一个feature文件应该只包含与其主题相关的一组场景scenario
- 一个场景scenario应该是简洁的，且只和子主题相关
- feature文件包含业务数据，但不应该包含测试数据
- 不要使用示例数据和数据表（data table）来扩展代码中的变量
- 一个feature或scenario不应该依赖其他的feature和scenario
- 考虑将cucumber框架作为一个沟通的框架，而不是测试框架 



![javascript](/img/BDD.png)

