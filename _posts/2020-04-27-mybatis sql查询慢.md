---
layout:     post
title:      "mybatis sql查询慢"
subtitle:   "mybatis"
date:       2020-04-27 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - mybatis
    - sql
---

在mybatis为持久化的java框架中，mapper和xml文件映射的sql，有时在实际执行时会很慢，甚至一直查询不出来，调查发现原因有二：

1. 参数化写法不同，执行逻辑不同。例如：#{param}，${'param'}

   \#符号标记的参数，在mybatis执行sql时，使用PreparedStatement对象，包含预编译sql操作，能防止sql注入安全问题，单次执行不如Statement，批量执行时能提高效能；

   $符号标记的参数，在mybatis执行sql时，使用Statement对象，不包含预编译操作，直接拼装sql语句成string执行，无法避免sql注入，单次执行速度快；

   这两种方式优缺点互为，一般建议使用#{param}，但在实际项目中，如果只有单次的sql操作，在不影响安全的前提下，可以使用${'param'}方式

   注：#{param}参数化到sql会自动为param加引号，${'param'}方式不会，所以调用${'param'}需要在括号中加引号，也由于这种特性，一般参数化select，table等sql标签用$方式，避免自动加入引号

 

2. sql语句大小写替换

   同事提供，本来项目中为了图方便都是小写sql语句，替换小写语句为大写也能提高一些sql执行速度，待实验

