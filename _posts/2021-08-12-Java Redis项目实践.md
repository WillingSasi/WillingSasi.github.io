---
layout:     post
title:      "Java Redis项目实践"
subtitle:   "Redis"
date:       2021-08-12 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Java
    - Redis
---

## 背景

  Redis在企业中用处广泛，但是一般可能用的比较多的地方就是缓存数据上，redis KV可设置过期时间，而且是基于内存存储，及时性上比数据库存储好太多。

  实际业务上遇到的情况是根据用户信息从sql中查询出用户详细信息，然后查询redis是否存在，如果不存在，写入redis，并设置过期时间；相应地，其它接口在获取用户详细的时候，优先查询redis，没有再去走sql查询。基于 spring具体实现 如下：

## 实现代码

1. application.yaml配置集群地址和密码

```yaml
spring:
  redis:
    cluster:
      nodes: ${REDIS_CLUSTER_NODES}
    password: ${REDIS_PASSWORD:myPassword}
```

2. 接口实现

   ```java
   @Slf4j
   @Repository
   @RequiredArgsConstructor
   public class RedisCacheRepository implements CacheRepository {
   
       private static final String KEY_SEPARATOR = "::";
       private static final String REDIS_KEY_PREFIX = "confirm-redis";
       private static final Integer EXPIRATION_TIME_MINUTES = 10;
   
       private final StringRedisTemplate redisTemplate;
   
       @Override
       public String get(String key) {
           return redisTemplate.opsForValue().get(toKey(key));
       }
   
       @Override
       public void save(String key, String value) {
           String currentKey = toKey(key);
           redisTemplate.opsForValue().set(currentKey, value);
           redisTemplate.expire(currentKey, EXPIRATION_TIME_MINUTES, TimeUnit.MINUTES);
           log.info("redis save success, key is {}, value is {}", currentKey, value);
       }
   
       private String toKey(String key) {
           return String.join(KEY_SEPARATOR, REDIS_KEY_PREFIX, key);
       }
   
   }
   ```

   
