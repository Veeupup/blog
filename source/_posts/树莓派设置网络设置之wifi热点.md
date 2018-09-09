---
layout: 'n'
title: 树莓派设置网络设置之wifi热点+LAMP搭建
date: 2018-09-09 20:41:42
tags:
categories:
- 树莓派
---

​	当在外面的时候，需要切换树莓派网络但是没有屏幕的情况下，可以通过设置树莓派开机后寻找已知的网络，连接并返回它的IP地址，然后通过SSH连接即可。

<!--more -->

# 设定自动寻找WIFI并连接

## 网络配置

需要修改**/etc/network/interfaces**的文件，

并修改成

```
iface wlan0 inet dhcp
wpa_conf /etc/wpa_supplicant/wpa_supplicant.conf
```

这里是设定开机后寻找WIFI信号，通过**/etc/wpa_supplicant/wpa_supplicant.conf**的配置来寻找wifi并连接。

## 设定已知WIFI

需要修改 **/etc/wpa_supplicant/wpa_supplicant.conf **，这里会保存树莓派已经连接过的WIFI 并且可以手动添加，格式如下。

```conf
# 最常用的配置。WPA-PSK 加密方式。
network={
	ssid="WiFi1"
	psk="password1"
	priority=5
}
 
network={
	ssid="WiFi2"
	psk="password2"
	priority=4
}
```

 注意到**priority**属性，这个属性是指定连接WIFI 的优先级。

设定好之后重启树莓派应该就已经设定成功了，而且由于**DHCP**分配给固定MAC地址的设备的IP一般不变，可找到IP 并**SSH**连接。当然，也可以用树莓派自带的**VNC** 连接。

# 配置LAMP

其实这个根本没什么值得记录的点，就当做普通的**Linux**服务器配置就好，这里提一点，不要太相信网上的教程和别人的经验，自己去按照正常的步骤试一试就会知道，有些教程由于时间或者版本问题会不能配置好的。

配置好之后就可以愉快的玩耍了。



