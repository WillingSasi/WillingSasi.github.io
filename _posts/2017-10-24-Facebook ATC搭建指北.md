---
layout:     post
title:      "Facebook ATC搭建指北"
subtitle:   "Facebook ATC"
date:       2017-10-24 15:27:00
author:     "WS"
header-img: "img/bookstore-2.jpg"
catalog:    true
tags:
    - ATC
    - Linux
---

## 简介：

网络测试中，主要是限制网速来模拟弱网络环境，而实际中弱网络时，网络延时，丢包率都会有变化，所以使用网速+延时+丢包率来定义一种网络环境更加合理。ATC是FaceBook开源的移动网络测试工具Augmented Traffic Control（ATC），能够方便的让我们模拟各种网络环境进行测试。

## 优点：

1. 在手机上通过Web界面就可以随时切换不同的网络环境

2. 多个手机可以连接到同一个WIFI下，相互之间模拟的网络环境各不影响

3. 具有一定的拓展性

## 安装要求：

1. Linux系统（推荐ubuntu）

2. 无线接入设备（笔记本无线网卡/usb无线网卡/路由器/树莓派）

## 安装部署：

创建无线接入点à分配IP地址和路由转发à部署ATCà链接ATC热点à浏览器访问

 

1.VMWare安装Ubuntu 14.04系统。

![javascript](/img/atc-1.png)

 

2.台式机usb插入无线网卡。

实际使用的无线网卡型号为TP-LINK TL-WN722N，插入后需在Ubuntu系统同意usb连接

![javascript](/img/atc-2.png)

 

3.检查无线网卡是否被读取到

输入：lsusb

![javascript](/img/atc-3.png)

 

4.检查无线网卡确定支持AP模式。

注：Ubuntu系统自建的wifi为Ad-hoc模式，无法和Android手机连接，故须搭建AP模式的

输入：iwlist

![javascript](/img/atc-4.png)

 

5.安装hostapd。

通过hostapd可以将无线网卡切换为AP模式,通过修改配置文件,可以建立一个开放式的的,WEP,WPA或WPA2的无线网络

输入：apt-get install hostapd

等待安装完成即可

 

6.新建hostapd配置文件，加入需要新建的无线接入点信息配置。

(1)在/etc/hostapd/目录下新建配置文件hostapd.conf并编辑

输入：vim hostapd.conf

(2)加入如下配置的无线网信息

ssid=AtcTest       // ssid是无线终端搜索网络时看见的名字

interface=wlan0     // wlan0是无线网卡的名字，如果是其他名字修改了即可

driver=nl80211     // driver一定要设置为nl80211

channel=10

hw_mode=g

ignore_broadcast_ssid=0

macadd_acl=0

wpa=3

wpa_passphrase=xxxxxx  //无线网的密码

wpa_key_mgmt=WPA-PSK

wpa_pairwise=TKIP

![javascript](/img/atc-5.png)

 

7.开启此无线网络。

输入：hostapd /etc/hostapd/hostapd.conf -B

![javascript](/img/atc-6.png)

注：此时就可以在手机wlan中搜索出来新建的热点了，但是无法连接上，会一直显示“查找ip”中

 

8.安装dhcp。

DHCP（Dynamic Host Configuration Protocol，动态主机配置协议）通常被应用在大型的局域网络环境中，主要作用是集中的管理、分配IP地址，使网络环境中的主机动态的获得IP地址、Gateway地址、DNS服务器地址等信息，并能够提升地址的使用率。

输入：apt-get install isc-dhcp-server

等待安装完成即可

 

9.编辑dhcpd.conf配置ip地址等信息。

(1)打开/etc/dhcp/dhcpd.conf文件，如果没有则建立一个即可

(2)加入如下配置信息

 

\#设置子网申明

subnet 10.1.1.0 netmask 255.255.255.224 {

\#地址池，这个范围表示你可以连接的终端数

 range 10.1.1.2 10.1.1.20;

\#设置DNS服务器地址

 option domain-name-servers ns1.internal.example.org;

\#设置DNS域

 option domain-name "internal.example.org";

\#设置无线网卡的IP地址

 option routers 10.1.1.1;

\#广播地址

 option broadcast-address 10.1.1.21;

\#设置默认租期，单位为秒

 default-lease-time 600;

\#设置客户端最长租期，单位为秒

 max-lease-time 7200;

}

![javascript](/img/atc-7.png)

 

10.手动给wlan0配置ip地址和子网掩码

输入：ifconfig wlan0 10.1.1.1 netmask 255.255.255.224

 

11.重新启动dhcp服务

输入：service isc-dhcp-server restart

注：此时手机就可以成功连接上此热点了，但是还不能联网

 

12.配置路由转发

输入：

\#打开linux内核ip转发

sysctl -w net.ipv4.ip_forward=1

\#清除所有规则

iptables –F

\#开启防火墙NAT转发

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

注：此时手机就可以正常访问网络了

 

13.安装pip

输入：apt-get install python-pip

 

14.安装atc所有组件

![javascript](/img/atc-8.png)

输入：pip install atc_thrift atcd django-atc-api django-atc-demo-ui django-atc-profile-storage

 

15.新建django工程

输入：django-admin startproject atcui

 

16.修改atcui/settings.py，加入ATC相关的内容

在INSTALLED_APPS部分加入如下内容：

INSTALLED_APPS = (

  ...

  \# Django ATC API

  'rest_framework',

  'atc_api',

  \# Django ATC Demo UI

  'bootstrap_themes',

  'django_static_jquery',

  'atc_demo_ui',

  \# Django ATC Profile Storage

  'atc_profile_storage',

)

 

17.修改atcui/urls.py，urlpatterns 中加入atc的url页面

导入如下2个模块，并在urlpatterns中添加如下内容

from django.views.generic.base import RedirectView

from django.conf.urls import include

 

urlpatterns = [

  ...

  \# Django ATC API

  url(r'^api/v1/', include('atc_api.urls')),

  \# Django ATC Demo UI

  url(r'^atc_demo_ui/', include('atc_demo_ui.urls')),

  \# Django ATC profile storage

  url(r'^api/v1/profiles/', include('atc_profile_storage.urls')),

  url(r'^$', RedirectView.as_view(url='/atc_demo_ui/', permanent=False)),

]

 

18.更新django数据库

输入：python manage.py migrate

 

19.指定新建wifi热点的内网网卡名称wlan0（eth0是外网网卡名称）

输入：atcd --atcd-lan wlan0

 

20.启动Django的工程

输入：python manage.py runserver 0.0.0.0:8000

 

21.已连接上AtcTest的手机浏览器访问该无线热点的8000端口（10.1.1.1:8000）

![javascript](/img/atc-9.png)

 

22.点击Turn On就可以开始使用了，当选择profile后面的Select按钮后(下面的)，最上面的开关按钮旁边会出现一个Update Shaping按钮，点击一下，你的网络就会变成你选择的profile所设置的网络环境

![javascript](/img/atc-10.png)

 

23.常用网络参数配置

网络带宽（bandwidth）   延迟（latency）      丢包率（packet loss）

错包率（corrupted packets） 乱序率（packets ordering）

![javascript](/img/atc-11.png)

 

 

## 参考文献

Atc工程官方GitHub地址：

https://github.com/facebook/augmented-traffic-control

 

配置网络参考：

[www.cnblogs.com/coderzh/p/AugmentedTrafficControl.html](http://www.cnblogs.com/coderzh/p/AugmentedTrafficControl.html)

[blog.csdn.net/w263044840/article/details/46469285](blog.csdn.net/w263044840/article/details/46469285)

[www.jianshu.com/p/fb4824fd5bbc](www.jianshu.com/p/fb4824fd5bbc)

 

专业名词解释：

[https://baike.baidu.com/item/Ad hoc/534288](https://baike.baidu.com/item/Ad hoc/534288)

https://baike.baidu.com/item/AP/2760808

[https://baike.baidu.com/item/%E5%B9%BF%E6%92%AD%E5%9C%B0%E5%9D%80/8614095?fr=aladdin&fromid=2624716&fromtitle=Broadcast+address](https://baike.baidu.com/item/广播地址/8614095?fr=aladdin&fromid=2624716&fromtitle=Broadcast+address)ss
