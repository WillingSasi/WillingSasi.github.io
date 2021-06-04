---
layout:     post
title:      "Docker搭建ELK日志记录框架"
subtitle:   " \"ELK In Docker\""
date:       2021-04-15 01:00:00
author:     "WS"
header-img: "img/docker-elk.jpg"
tags:
    - docker
    - elk
    - devops
---



# Docker 搭建 ELK 日志记录框架详细步骤:

1. `docker create network elknet`

2. `docker run -d --name elasticsearch --network elknet --network-alias elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.9.1`

3. `docker run -d --name kibana --network elknet --network-alias kibana -p 5601:5601 kibana:7.9.1`

4. `docker run -d --name logstash --network elknet --network-alias logstash -p 5044:5044 logstash:7.9.1`

5. 配置的log4j2发送tcp到logstash

6. `vi /usr/share/logstash/pipeline/logstash.conf`

7. ````text
   input {
     tcp {
       port => 5044
       mode => "server"
     }
   }
   
   output {
     elasticsearch {
       hosts => ["elasticsearch:9200"]
       index => "apidemo"
     }
   
     stdout{
       codec => rubydebug
     }
   }
   ````

8. `vim log4j2.xml`

9. ```xml
   <appenders>
     +
     <Socket name="LogstashTcp" host="你的localhost" port="5044" protocol="TCP">
       <PatternLayout>
         <Pattern>%d{HH:mm:ss.SSS} %-5level method:%l%n%m%n</Pattern>
       </PatternLayout>
     </Socket>
   </appenders>
   
   <loggers>
     <root level="trace">
       +
       <appender-ref ref="LogstashTcp"/>
     </root>
   </loggers>
   
   ```

10. Kibana上添加apidemo index, 然后查询refash就显示数据了

11. ![javascript](/img/docker-kibana.png)

