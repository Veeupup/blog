---
title: 分布式资源调度YARN
date: 2019-01-24 18:17:02
tags:
- Hadoop
categories:
- 大数据
---

​	学习笔记记录。

<!--more-->

# YARN产生背景

* MapReduce1.x 存在的问题：单点故障&节点压力&不能够支持除了MapReduce以外的计算框架
* ![MapReduce架构](http://ww1.sinaimg.cn/large/9d82e933gy1fzhui9zkzzj20ij0bpadi.jpg)



1.x时：

MapReduce：Master / Slave 架构，一个 JobTracker 带多个 TaskTracker



JobTracker： 负责资源管理和作业调度

TaskTracker：

定期向JT汇报本节点的健康状况、资源使用情况、作业执行情况，接收来自JT的命令：启动任务/杀死任务

* 资源利用率低、运维成本高

![](http://ww1.sinaimg.cn/large/9d82e933gy1fzisv7cg5jj20kz08ewia.jpg)

* 催生了YARN的诞生

![](http://ww1.sinaimg.cn/large/9d82e933gy1fzisuj3vgrj20jb07yae2.jpg)

​	不同的处理可以采用不同的框架。

​	YARN，操作系统级别资源调度框架，不同计算框架可以共享同一个HDFS集群上的数据，享受整体的资源调进而提高集群资源的利用率：

​	XXX ON YARN 也就是 Spark/MapReduce/Storm/Flink ON  YARN

# YARN概述

* Yet Another Resouce Negotiator
* 通用的资源管理系统
* 为上层应用提供统一的资源管理和调度

# YARN架构

* ResourceManager:	RM

  * 整个集群提供服务的RM在同一时间只有一个（生产环境通常有另外一个备份的RM），负责集群资源的统一管理和调度。

  * 处理客户端的请求：提交一个作业、杀死一个作业

  * 监控 NM，一旦某个  NM 挂了，该DM上运行的任务需要通知 AM来如何进行处理

  

* NodeManager:     NM

  * 整个集群中有多个，负责自己本身节点资源管理和使用

  * 定时向RM汇报本节点的资源使用情况

  * 接收并处理来自  RM 的各种命令：启动Container

  * 处理来自 AM 的命令

  * 单个节点的资源管理

  

* ApplicationMaster:     AM

  * 每个应用程序对应一个：MR、Spark，负责应用程序的管理

  * 为应用程序向  RM  申请资源（core、memory），分配给内部的task

  * 需要与 NM  通信：启动/停止task，task是运行在  container  里面，AM也是运行在 container 里面

* Container

  * 封装CPU、Memory 等资源的一个容器
  * 是一个任务运行环境的抽象

* Client

  * 提交作业
  * 查看作业的运行进度
  * 杀死作业 

  

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzit0nkfa1j20ig0dv77e.jpg)







# YARN执行流程

步骤：

* 客户端向RM提交作业
* RM分配第一个 container （运行在一个 NM 上）
* container 启动一个 AM
* AM 向RM 注册（此时客户端可以向RM查询运行情况），AM 告知 RM 需要多少CPU、memory 等资源，
* RM 向AM 分配响应的 NM
* 在对应的 NM （RM 分配的NM）之上开始启动任务（以container方式运行）



耦合性在 AM 这里，不同框架只需要实现 YRAN 这里  AM 的接口。

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzitk6pcyfj20gb0ab40w.jpg)



# YARN环境搭建

见搭建Hadoop。

# 提交作业到YARN

使用自带MapReduce 作业  使用蒙特卡洛算法计算 pi 的值测试。

hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.7.0.jar

```shell
hadoop jar hadoop-mapreduce-examples-2.6.0-cdh5.7.0.jar pi 2 3
```

本机跑性能不行确实比较慢，这么简单一个计算跑了几分钟……等我的 mbp  到了就开心了。

![](https://ws1.sinaimg.cn/large/9d82e933gy1fziumk8r6zj20x204nglx.jpg)

```
Job Finished in 246.11 seconds
Estimated value of Pi is 4.00000000000000000000
```

