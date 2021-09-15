---
layout:     post
title:      "GUI Framework Desgin"
subtitle:   "GUI"
date:       2020-08-01 15:27:00
author:     "WS"
header-img: "img/gui.jpg"
catalog:    true
tags:
    - Java
    - Gui Test
---

## 背景

  在已有比较完善的api自动化前提下, gui回归测试依然存在大量手工操作，老框架局限太大, 计划有序重新开展少量gui自动化测试, 以丰富我们的测试手段，提高一定的测试效率，最终保证产品质量;

开源地址: [[WillingSasi/GuiFrameworkDemo: GuiFrameworkDemo (github.com)](https://github.com/WillingSasi/GuiFrameworkDemo)](https://github.com/WillingSasi/ApiSuperTest)

## 痛点:

1. 老gui框架设计思想不符当前环境的测试工作
2. 老框架维护case, 在case量上来后维护负担称倍增加
3. 老框架技术陈旧, 报告展示缺乏case的综合分类展示
4. 老框架代码和excel混合写法, 对使用和编写人员都不够友好

## 框架设计:

针对以上4痛点, 重新设计框架, 如下:

1. 考虑到都懂代码的前提下, case组装都设计在单个代码方法中, 不涉及单独的keyword和case

2. 老框架分层颗粒度混乱, 新框架重新划分页面为三层结构, page→service→testCase

3. 放弃老框架散装报告, 新框架引入开源报告框架Allure, 并私人定制开发, 好用好看

4. 杜绝大量excel操作, 数据存储用配置文件或单个对象, 编写case对于TE和SET角色人员更加顺手

> 框架纯个人开发，且公司敏感信息已全部脱敏，故开源万岁！！！
