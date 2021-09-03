---
layout:     post
title:      "Test result analysis program"
subtitle:   "Data Analysis"
date:       2020-07-01 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Java
    - AOP
    - Mongodb
---

## 引子

  API自动化用例数量庞大,运行不费事,但是执行完成后的分析需要占用大量时间,劳民伤财;

  从这个痛点出发,如果有一套专门分析运行完结果的系统,能把大量繁琐分析的时间节省下来,测试人员能有更多时间来针对那些真正需要人工排查的问题.

## 项目设计:

1. 数据来源

   方式一:  脚本运行完成后现成的RunLog.log或者html文件,需要按指定关键字正则,提取case名称,step名称,api 

   ​               名称,以及所有true/false状态

   方式二:  运行中日志框架增加写入数据池操作

   实现如下:

   1. aspectj + mongodb

   2. aop作用在框架打印日志的方法上,以及整个case执行方法上

      ```java
      @Pointcut("execution(void   com.xxx.framework.HtmlReport.addTestLogStepDescription(String, String))")
      public void autoStepLog(){
      }
      
      @Pointcut("execution(void com.xxx.framework.HtmlReport.updateTestLog(..))")
      public void autoTestLog(){
      }
      
      @Pointcut("execution(void businesscomponents.testCase.Normal.login.*.*(..))")
      public void autoLaunchLog(){
      }
      ```

   3. 按步骤拼接json字符串

   4. autoLaunchLog监控整个case执行结束后发送json到mongodbclient

      ![javascript](/img/dataAnalysis.png)

2. 数据整理

   转换为如下json格式数据

   ```json
   {
     CaseName:[
       {
         Step1:{
           CheckPoint1:true,
           CheckPoint2:true,
           CheckPoint3:false
         }
       },
       {
         Step2:{
           CheckPoint1:true,
           CheckPoint2:true,
           CheckPoint3:false
         }
       },
       {
         Step3:{
           CheckPoint1:true,
           CheckPoint2:true,
           CheckPoint3:false
         }
       }
     ]
   }
   ```

3. 数据分析

   遍历查找json,第一个CheckPoint为false的点进行判断,判断依据有:

   ```java
   Response code,
   
   Response message,
   
   NullPointerException,
   
   Response Context does not contain the jsonPath,
   
   ....
   ```

4. 自动重跑机制

   属于特定失败结果的,收集用例名称,分析完成后再次运行失败用例并分析

5. 结果展示

   1. 当次运行成功失败饼图

   2. 一条用例,历史运行,成功失败详细折线图
