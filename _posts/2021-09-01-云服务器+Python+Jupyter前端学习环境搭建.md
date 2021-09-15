---
layout:     post
title:      "云服务器+Python+Jupyter前端学习环境搭建"
subtitle:   "Jupyter"
date:       2021-09-01 15:27:00
author:     "WS"
header-img: "img/cloud.jpg"
catalog:    true
tags:
    - Jupyter
    - Python
    - Cloud
---

## 短信

  小程序一直在运行, 由于WX小程序的数据部署资源其实你属于腾讯云范围, 可能看我也有开发需求, 几天内收到腾讯云的短信电话销售....云服务器, 好吧, 很少的money体验2G4核 100G的云服务器半年!!!

![javascript](/img/cloud-1.png)

## 搭建

  最开始想用来部署训练模型啥的, 资源不用就是浪费, 首先搭建python环境, 在查找深度学习模型训练资料时, 发现了一个好玩的, 就是Jupyter, 没想到能在web页面上直接写代码, 直接运行, 甚至写注释, 记笔记, 便携markdown文档....我看懂了, 也大受震撼

  《使用python做数据分析》一书我只看了前几章, 其实开始就介绍了Jupyter, 要是准备学习数据分析等相关知识, 强烈建议先安装好开发环境Jupyter, 比直接在conselo中写代码舒服多了, 比idea中都舒服...

   我由于实在centos7服务器中搭建, 搭建使用的资源时JupyterLab, 基于python3.6+, 搭建两步:

1. centos先升级python

   [CentOS 7 安装 Python 3_Z-CSDN博客_centos7 python3](https://blog.csdn.net/achi010/article/details/113628513)

2. 使用pip等命令安装

   [Jupyter Notebook介绍、安装及使用教程 - 简书 (jianshu.com)](https://www.jianshu.com/p/91365f343585)

3. 一些配置, 比如web登录密码, 比如云服务保持后台运行, 以及一些实用的插件

   [Jupyter Lab 的一些简单配置 - LSKReno - 博客园 (cnblogs.com)](https://www.cnblogs.com/lskreno/p/10844315.html)

   [Linux服务器上配置Jupyter并在后台运行_fangzi1123的博客-CSDN博客_jupyter后台运行](https://blog.csdn.net/qq_41699621/article/details/103064684)

4. 启动后, 基于云服务的话, 要访问得用IP+Port, IP就是云服务器的公网ip, port需要在安全组中添加, 默认云服务提供商都只提供一个22的供ssh链接端口, 比如你启动的JupyterLab端口是8080, 需要在腾讯云上添加一个端口映射的策略组绑定在实例上, 当然, 你也可以选择端口全部开放访问和映射....不考虑安全因素的话

5. 浏览器访问, 首页可新建这几种类型文件

![javascript](/img/cloud-2.png)

6. 实时编写和结果展示

   ![javascript](/img/cloud-3.png)

