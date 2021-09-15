---
layout:     post
title:      "Jvm Sandbox详细解读和实践(一)"
subtitle:   "Jvm Sandbox"
date:       2021-07-01 15:27:00
author:     "WS"
header-img: "img/sandbox.jpg"
catalog:    true
tags:
    - Java
    - Jvm
    - 流量回放
    - Sandbox 
---

## 1. 药引子

  随着计算机技术的发展, 这两年在测试领域, 也有很多技术落地和探索, 而最近半年在公司听到最多次的两个名词, 一个是全链路压测, 另一个就是流量回放平台. 

  其实大概在一年多前就在技术社区看到过酷家乐开放出来的流量回放平台, 基于jvm-sandbox, 当时就只记得看过, 但是对原理没有详细钻研, 后来和同事又讨论起, 了解了好未来开源的conan流量回放平台, 和基于jvm字节码的思路不同, 遂重新拾起, 先起底这个sandbox.

  流量回放说白了, 就是网络上所有的请求就叫流量, 回放的目的目前主要是为了系统的质量保障, 比如可以不用手工写代码就快速的全量自动化测试, 比如可以抽象模型后在测试环境全链路压测, 以及日常的为QA提供及时的api调用支持, 总之, 有个这一批批的数据, 可以做分析也可以做测试, 相比起以前的一个个接口的手写测试, 直接自动的全量记录无疑是超级提效的. 个人总结流量回放分为两部分: 

>  一部分是前置步骤, 也是流量的采集, 从哪里采集, 如何采集, 有两个名词是“主路复制”,“旁路复制”;
>
>  二部分就是后置步骤, 就是对流量的分类, 降噪, 结果对比, 以及部分mock系统的依赖

这其实也是大体上系统的逻辑划分, 任何一部分都能研究出各种花样, 后面本文在此只实践研究下流量采集中的主路复制中的一种解决方案, 阿里的jvm-sandbox. 旁路复制的一种就是把Elasticsearch作为流量来源, 都是为了获取系统日志. 

## 2. 什么是Jvm Sandbox

官方: JVM沙箱容器，一种JVM的非侵入式运行期AOP解决方案

特点: 无侵入(个人感觉最重要), 类隔离, 可插拔, 多租户, 高兼容 

AOP咱们都知道, java项目使用最广的就是aspectj aop了, aspectj其中一种使用方法就是挂载在jvm启动参数上, -javaagent:XXXX方式, 相当于在启动项目的时候, 先运行加载的jar包方法, 再执行修改或增强方法, 最后才到实际的代码中, 这种最大的弊端就是“侵入”性, 项目必须启动时带上agent, 没法做到灵活的随用随加载, 随停随删除. Sandbox支持Attack方法, 就是通过启动服务jvm的线程号, 来动态的选择修改增强服务, 不用时也可立即卸载, sandbox底层都是属于基于Instrumentation的动态编织类的AOP框架, sandbox依赖的是jdk 工具 JVMTI和Attach Api而实现出来, 各种关系都可单独研究, 关系如下:

![javascript](/img/sandbox.jpeg)



JVM Sandbox主要包含SandBox Core、Jetty Server和自定义处理模块三部分, 框架关系图如下:

![javascript](/img/sandbox-1.png)

用 Attach 将sandbox挂载到目标 JVM 进程上，沙箱的启动依赖 Java Agent，启动之后沙箱会一直维护着 Instrument 对象引用，在沙箱中 Instrument 对象是一个非常重要的角色，它是沙箱访问和操作 JVM 的唯一通道，后续修改字节码和重定义类都要经过 Instrument。另外，沙箱启动之后同时会启动一个内部的 Jetty 服务器，这个服务器用于外部进程和沙箱进行通信，./sandbox.sh -p 8080 -d ‘my-sandbox-module/addLog’ 这行代码，实际上就是通过 HTTP 协议来告诉沙箱执行 my-sanbox-module 这个模块的 addLog 这个功能的

## 3. 使用操作

1. 安装

   > 下载最新release包
   >
   > ```shell
   > wget https://github.com/alibaba/jvm-sandbox/releases
   > ```
   >
   > 解压进入bin目录
   >
   > ```shell
   > ./install-local.sh -p ~/opt
   > ```
   >
   > 输出install sandbox successful表示成功
   >
   > 其实不安装也可直接使用了, 直接用bin下的sandbox.sh启动

2. 启动一个待测springboot项目

   > sandbox装好了, 缺一个实验用的jvm项目, 除了在公司后端项目上实验, 可以临时用这个demo
   >
   > https://github.com/WillingSasi/springbootDemo.git
   >
   > 运行项目, 并查看jvm占用进程号PID
   >
   > ```shell
   > pro-2:springbootdemo grahamliu$ lsof -i:8080
   > COMMAND   PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
   > java    92629 grahamliu   83u  IPv6 0x2520b8c85f0bba79      0t0  TCP *:http-alt (LISTEN)
   > ```

3. 实现sandbox Module逻辑并打包

   ```java
   @Information(id = "test-attach")
   public class TestAttach implements Module {
       @Resource
       private ModuleEventWatcher moduleEventWatcher;
   
       @Command("addLog")// 模块命令名
       public void addLog() {
           new EventWatchBuilder(moduleEventWatcher)
                   .onClass("com.example.springbootdemo.demo.controller.DemoController")// 想要对 DemoController 这个类进行切面
                   .onBehavior("sandboxMessage")// 想要对上面类的 sandboxMessage 方法进行切面
                   .onWatch(new AdviceListener() {
                       //对方法执行之前执行, 修改入参
                       @Override
                       protected void before(Advice advice) throws Throwable {
                           //获取方法的所有参数
                           Object[] parameterArray = advice.getParameterArray();
                           System.out.println(parameterArray[0].toString());
                           advice.changeParameter(0, "sandboxMessage");
                       }
                   });
       }
   }
   ```

   > 打包完成的jar文件需要放在sandbox-module目录下, sandbox有两个目录, 一个系统模块目录, 一个用户模块, 一般放在用户目录, ~/.sandbox-module
   >
   > 详细步骤可直接参考官方wiki: [jvm-sandbox/doc at master · alibaba/jvm-sandbox (github.com)](https://github.com/alibaba/jvm-sandbox/tree/master/doc)

4. 启动sandbox并挂载至目标JVM, 如下就表示成功

   > ```java
   > Pro-2:bin grahamliu$ ./sandbox.sh -p 92629
   >                     NAMESPACE : default
   >                       VERSION : 1.2.1
   >                          MODE : ATTACH
   >                   SERVER_ADDR : 0.0.0.0
   >                   SERVER_PORT : 63991
   >                UNSAFE_SUPPORT : ENABLE
   >                  SANDBOX_HOME : /Users/grahamliu/sandbox/bin/..
   >             SYSTEM_MODULE_LIB : /Users/grahamliu/sandbox/bin/../module
   >               USER_MODULE_LIB : /Users/grahamliu/sandbox/sandbox-module;~/.sandbox-module;
   >           SYSTEM_PROVIDER_LIB : /Users/grahamliu/sandbox/bin/../provider
   >            EVENT_POOL_SUPPORT : DISABLE
   > 
   > ```

5. 激活module

   > -l查看所有模块, test-attach已经是loaded状态, 但是后面两个00, 表示还没有激活具体方法

   > ```shell
   > Pro-2:bin grahamliu$ ./sandbox.sh -p 93707 -l
   > sandbox-info        	ACTIVE  	LOADED  	0    	0    	0.0.4          	luanjia@taobao.com
   > sandbox-module-mgr  	ACTIVE  	LOADED  	0    	0    	0.0.2          	luanjia@taobao.com
   > sandbox-control     	ACTIVE  	LOADED  	0    	0    	0.0.3          	luanjia@taobao.com
   > test-attach         	ACTIVE  	LOADED  	0    	0    	UNKNOW_VERSION 	UNKNOW_AUTHOR
   > mock                	ACTIVE  	LOADED  	0    	0    	1.0.0          	fusheng.chu
   > repeater            	ACTIVE  	LOADED  	0    	0    	1.0.0          	zhaoyb1990
   > total=6
   > ```

   > -d指定加载模块和方法, 也就是实现Module的定义名称
   >
   > ```shell
   > Pro-2:bin grahamliu$ ./sandbox.sh -p 93707 -d 'test-attach/addLog'
   > Pro-2:bin grahamliu$ ./sandbox.sh -p 93707 -l
   > sandbox-info        	ACTIVE  	LOADED  	0    	0    	0.0.4          	luanjia@taobao.com
   > sandbox-module-mgr  	ACTIVE  	LOADED  	0    	0    	0.0.2          	luanjia@taobao.com
   > sandbox-control     	ACTIVE  	LOADED  	0    	0    	0.0.3          	luanjia@taobao.com
   > test-attach         	ACTIVE  	LOADED  	1    	1    	UNKNOW_VERSION 	UNKNOW_AUTHOR
   > mock                	ACTIVE  	LOADED  	0    	0    	1.0.0          	fusheng.chu
   > repeater            	ACTIVE  	LOADED  	0    	0    	1.0.0          	zhaoyb1990
   > total=6
   > ```
   >
   > 以上显示表示已成功挂载并激活

6. Attach结果

   > 原方法正常请求打印httpMessage信息, Attach后打印module中的方法, 也就是修改后的信息“sandboxMessage”
   >
   > ![javascript](/img/sandbox-2.png)



> 参考资料:
>
> [alibaba/jvm-sandbox: Real - time non-invasive AOP framework container based on JVM (github.com)](https://github.com/alibaba/jvm-sandbox)
>
> [JVM SandBox的技术原理与应用分析-InfoQ](https://www.infoq.cn/article/TSY4lGjvSfwEuXEBW*Gp)
>
> [JVM SandBox实现原理详解_hxt的博客-CSDN博客](https://blog.csdn.net/qq_40378034/article/details/116255652)
>
> [JVM-SANDBOX模块编写EXAMPLE_weixin_34268843的博客-CSDN博客](https://blog.csdn.net/weixin_34268843/article/details/89807966)
>
> [[jvm-sandbox - 标签 - 月色深潭 - 博客园 (cnblogs.com)](https://www.cnblogs.com/moonpool/tag/jvm-sandbox/)]
>
> [4. jvm-sandbox之调用方式(命令行和http) - 月色深潭 - 博客园 (cnblogs.com)](https://www.cnblogs.com/moonpool/p/14510443.html)

