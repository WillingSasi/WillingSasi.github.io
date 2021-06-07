---
layout:     post
title:      "Springboot配置ElasticsearchRepository接口使用"
subtitle:   " \"springboot use es interface\""
date:       2021-04-18 01:00:00
author:     "WS"
header-img: "img/springboot-bg.jpg"
catalog:    true
tags:
    - Springboot
    - ELK
    - Devops
---



## 1.maven pom 引入包

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

## 2.实体类TestLog.java

```java
@Component
@Document(indexName = "apidemo",indexStoreType = "testLog", shards = 1,replicas = 0, refreshInterval = "-1")
public class TestLog implements Serializable {
    // 必须指定一个id，
    @Id
    private String id;
    private String message;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getMmessage() {
        return message;
    }

    public void setMessage(String loginfo) {
        this.message = loginfo;
    }
}
```

*注意添加@Document注解参数

## 2.properties配置

```properties
spring.data.elasticsearch.client.reactive.endpoints=XXX.XXX.XXX.XXX:9300
spring.data.elasticsearch.client.reactive.connection-timeout=10s
spring.data.elasticsearch.client.reactive.socket-timeout=10s
```

*elasticsearch基本信息配置

## 3.接口继承调用

```java
@Repository
public interface TestlogRepository extends ElasticsearchRepository<TestLog, String> {
}
```

*ElasticsearchRepository已经包含常用的查询等crud操作, 直接可使用

##4.方法中调用查询

```java
@Autowired
   private TestlogRepository testlogRepository;

@Test
void contextLoads() {
       System.out.println(testlogRepository.findById("TJcu6XgBm56pu8b2mi35").get().getMessage());
}
```

*查询id为TJcu6XgBm56pu8b2mi35的es日志信息