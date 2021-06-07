---
layout:     post
title:      "CricuitBreaker 熔断器的二三事"
subtitle:   ""
date:       2021-06-06 21:53:00
author:     "WS"
header-img: "img/cricuitbreaker.jpg"
catalog:    true
tags:
    - Java
    - Spring
    - Springboot
    - CricuitBreake
---



## 背景: 为了实现公司后端接口的被动降级功能, 业务表现为当某接口服务不可用时, 可自动按需返回默认配置的fallback内容, 一定程度上增强后端服务的稳定性, CricuitBreaker断路器由此引入



## 1. CricuitBreaker白话概念

> 计算机程序方面的问题, 都可以用“增加一个中间层”来解决.
>
> CricuitBreaker就相当于在接口调用和方法实现中的中间层, 当设置好失败成功配比, 并配置好备份路径以及半开状态观察时间后, 真实接口调用时, 一旦失败率达到配置的阈值, 后续再有请求, 自动切换到fallback备份路径调用;
>
> 当在观察时间内接口稳定成功调用后, 自动切换会原调用路径;
>
> 当再次报错超过阈值, 自动再进入fallback, 循环往复动态切换.

## 2. 相关资源

> 有很多可用的资源, Netflix的hystrix, spring retry 还有Resilience4j, 这里用的Resilience4j, 因为Netflix的已不再更新, retry功能不够强大

## 3. 功能配置项

```yaml
mode: circuit_breaker
      force-open-circuit-breaker: false
      default-circuit-breaker:
        sliding-window-size: 10
        failure-rate-threshold: 50
        record-exceptions: CircuitBreakException
```

> force-open-circuit-breaker: 是否全开, 如果为true, 所有请求都直接进入fallback, 一般用作debug
>
> sliding-window-size: 计数用的环形区大小
>
> failure-rate-threshold: 失败比例, 范围100>=count>0
>
> record-exceptions: 监控异常类型, 可按需自定义, 只有抛出此处定义的异常, 才会计算为调用失败, 并进入fallback备份路径

## 4. 熔断器的常用状态有三种：打开状态、半打开状态和关闭状态

> 关闭状态：这种是初始状态，即没有触发熔断的情况下，熔断器就保持关闭状态，说明此时服务稳定，不需要开启熔断
>
> 打开状态：当达到了熔断的阈值，熔断器就会进入打开状态，此时熔断器会阻断一些调用下游服务的请求
>
> 半打开状态：当熔断器持续保持打开状态一段时间后，会进入半打开状态，放行一定比例的请求下游服务的调用，根据这些请求的成功率判断，熔断器是恢复关闭状态还是再次进入打开状态

## 5. 特殊点理解

> 1. sliding-window-size有两个, 从关闭到打开之间计数有一个, 半开状态也有一个, 大小一致;
>
> 2. 服务刚开始运行, sliding-window-size必须计数满后才开始计算工作, 比如你配置size是10, 失败50%就熔断fallback, 然后新服务开始, 前9个请求都是失败的, 也不会进入熔断, 因为此时环形缓存区size10还未计数打满, 当第10个请求进来后, CricuitBreaker才会工作, 此时显然已超50%, 当11个请求进来后, 自动直接请求fallback路径, 而不再走远路径请求一次. 半开状态sliding-window-size计数原理一样
>
>    一句话概括:  如果请求次数低于环状缓冲区的大小，那么即使失败率已经低于了阈值，依然不会触发熔断关闭!!!
>
> 3. 熔断器从打开状态到半开状态的等待时间, 默认是60秒

相关链接:

https://cloud.tencent.com/developer/article/1082827

https://blog.csdn.net/nnsword/article/details/81116928

https://blog.csdn.net/weixin_38569499/article/details/89479754

https://www.jianshu.com/p/5531b66b777a