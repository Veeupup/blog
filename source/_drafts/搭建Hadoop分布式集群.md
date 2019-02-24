---
title: 搭建Hadoop分布式集群
date: 2019-02-03 12:44:09
tags:
- Hadoop
categories:
- 大数据
---

​	本地分布式集群搭建。

<!--more-->

# 环境搭建

​	本地采用virtualbox搭建分布式虚拟环境，具体ip地址如下：

* Hadoop1: 192.168.0.113

* Hadoop2: 192.168.0.114

* Hadoop3: 192.168.0.115

  设置HOSTNAME和hosts,方便后续安装调试。

  三台机器的角色和职责如下：

  Hadoop001: NameNode   ResourceManager

  Hadoop002: DataNode  NodeManager

  Hadoop003: DataNode  NodeManager

> 注：可以设置ssh-key，免密码登陆方便