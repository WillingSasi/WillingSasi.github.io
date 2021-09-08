---
layout:     post
title:      "Oracle 设置TO_DATE('13-OCT-20', 'dd-MON-yy'), 报错 ORA-01843: 无效的月份"
subtitle:   "Oracle"
date:       2020-04-22 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - Oracle
---

1. Oracle执行SQL语句:

   select * from xxxx xx where STATUS='a' and xx.time >= TO_DATE('13-OCT-20', 'dd-MON-yy') and rownum < 200 ;

 

2. 报错:

   ORA-01843: 无效的月份, 01843. 00000 - "not a valid month"

 

3. 原因:

   因为客户端是中文环境，月份格式就不能用英文的月份写法，必须用中文的“10月”

 

4. 延伸:

   查询不通时区系统表示方式, 可以用sql

   SELECT TO_CHAR(sysdate, 'DD-MON-YYYY','NLS_DATE_LANGUAGE = ''SIMPLIFIED CHINESE''') Chn,
   TO_CHAR(sysdate, 'DD-MON-YYYY', 'NLS_DATE_LANGUAGE = American') Ame,
   TO_CHAR(sysdate, 'DD-MON-YYYY', 'NLS_DATE_LANGUAGE = Japanese') Jap,
   TO_CHAR(sysdate, 'DD-MON-YYYY', 'NLS_DATE_LANGUAGE = english') Eng
   FROM DUAL;

   需要用到DUAL这个特殊表
