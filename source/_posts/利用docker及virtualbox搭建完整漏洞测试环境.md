---
title: vulhub和DVWA两种靶场环境的搭建
date: 2018-08-03 11:48:17
categories:
- 网络安全
---

​	最近在iMiss实验室进行关于Web蜜罐的实现，涉及到自己搭建测试环境，在此记录分享下环境搭建的过程。

​	感谢开源社区，感谢vulhub和DVWA。。

## vulhub搭建

> Vulhub是一个基于`docker`和`docker-compose`的漏洞环境集合，进入对应目录并执行一条语句即可启动一个全新的漏洞环境，让漏洞复现变得更加简单，让安全研究者更加专注于漏洞原理本身。 

* 虚拟机环境 VirtualBox : ubuntu 18.04LTS
* 运行环境：docker 
* 工具
  * docker-compose	利用开源靶场image
  * docker
* 靶场环境：由 [vulhub](https://github.com/vulhub/vulhub) 提供

<!-- more --> 

举个栗子：

下面是一个关于Nginx的漏洞搭建实例以及步骤。

* 安装上述工具和环境
* clone [vulhub](https://github.com/vulhub/vulhub)  
* docker-compose：
  * docker-compose build 
  * docker-compose up -d
* 运行成功

## DVWA搭建

> Damn Vulnerable Web App (DVWA) is a PHP/MySQL web application that is damn vulnerable. Its main goals are to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and aid teachers/students to teach/learn web application security in a class room environment.

* 虚拟机环境 VirtualBox : Centos7
* 运行环境：LAMP
* 具体 ：

其实主要是Centos的LAMP的配置，把LAMP配置好即可。

这里就不写坑了，在另外一个分类。

