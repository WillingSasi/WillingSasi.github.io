---
layout:     post
title:      "Appium添加Listener运行报错"
subtitle:   "Appium"
date:       2020-08-10 15:27:00
author:     "WS"
header-img: "img/listener.jpg"
catalog:    true
tags:
    - Appium
---

1. 报错信息:

   Error creating bean with name 'object' defined in io.appium.java_client.even

 

2. 原因:

   Appium自定义Listener需要用到springframework的aop, java-client版本中依赖了springframework的版本

   Maven工程中的pom.xml文件, 在添加java-client依赖后自动下载相关依赖, 包含springframework

   但是我本地在pom.xml中单独又配置了高版本的springframework系列, 导致和java-client版本不匹配, 运行时报错

    

3. 解决:

   方法1.删除pom中单独配置的springframework系列依赖

   方法2.或者直接修改springframework系列依赖的版本号, 确保和java-client一致

   方法3.由于我配置的java-client版本不是最新的, 理论上java-client最新版本应该也能兼容更新的springframework, 需要实际依赖尝试下
