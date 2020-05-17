---
title: 初识Hadoop
date: 2019-01-20 13:24:15

---

	记录学习Hadoop记录的笔记。
	第一篇，对整个Hadoop的基本介绍和了解。
<!--more-->

名称由来：

![clipboard.png](http://ww1.sinaimg.cn/large/9d82e933gy1fzd0hzzt06j20h10b6qj4.jpg)

### *http://hadoop.apache.org/*

分布式存储 + 分布式计算平台

能做什么
--------

-   搜索引擎

-   日志分析

-   商业智能

-   数据挖掘

核心组件
========

分布式文件系统 HDFS
-------------------

-   源自 Google 的GFS 的论文，论文发表于 2003 年10月

-   HDFS 是 GFS 的克隆版

-   HDFS特点：拓展性、容错性、海量数量存储

-   将文件切分成指定大小的数据块（128MB）并以多副本存储的多个机器上

-   数据切分、多副本、容错等操作对用户透明

![clipboard.png](http://ww1.sinaimg.cn/large/9d82e933gy1fzd0ig9onsj20sr0gj7dd.jpg)

资源管理调度系统 YARN
---------------------

-   YARN —— Yet Another Resource Negotiator

-   负责整个集群资源的管理和调度

-   特点：拓展性、容错性、多框架资源统一调度

![clipboard.png](http://ww1.sinaimg.cn/large/9d82e933gy1fzd0ioa2onj20s20g5k77.jpg)

分布式计算框架 MapReduce
------------------------

-   源自 Google的MapReduce 论文 ，2004.12

-   MapReduce是Google MapReduce的克隆版

-   特点：拓展性、容错性、海量数据离线处理

![clipboard.png](http://ww1.sinaimg.cn/large/9d82e933gy1fzd0iur5bmj210o0ew7fk.jpg)

优势
====

高可靠性
--------

-   数据存储：数据块多副本

-   数据计算：重新调度作业计算

高拓展性
--------

-   存储/计算资源不够的时候，可以横向拓展线性增加

-   一个集群中可以包含数以千计的节点

其他
----

-   存储在廉价机器上，降低成本

-   成熟的生态圈

发展史
======

*https://www.infoq.cn/article/hadoop-ten-years-interpretation-and-development-forecast*

生态系统
========

狭义Hadoop
----------

一个适合大数据
分布式存储（HDFS）、分布式计算（MapReduce）、资源调度（YARN）的平台。

广义Hadoop
----------

Hadoop生态系统，Hadoop生态系统是一个很庞大的概念，Hadoop是其中最重要最基础的一个部分；生态系统中的每一子系统只解决某一个特定的问题域（甚至可能很窄），不搞统一的一个全能系统，而是小而精的多个小系统。

![clipboard.png](http://ww1.sinaimg.cn/large/9d82e933gy1fzd0jjbrjaj20jm09vqb8.jpg)

-   HDFS（文件系统）是GFS的一个克隆版，存储数据，多副本，高容错，是Google的GFS的论文的实现

-   MapReduce跑在YARN之上，编程较复杂

-   Hive：Facebook开源，类似SQL的语言对海量数据进行统计分析，离线分析

-   R Connectors：统计分析数据

-   Mahout：机器学习的库

-   Pig：脚本性的语言，Yahoo开源，脚本转换到MapReduce上实现，离线分析

-   Ooize：工作流调度引擎

-   Zookeeper：协调，管理框架，ex：Hbase的单点问题

-   Flume：日志收集框架，进行处理分析

-   Sqoop：（SQL+Hadoop）传统关系型数据库，和各个框架之间协调

-   Hbase：列式存储（基于BigTable的论文），Hadoop中的数据库

特点
----

-   开源、社区活跃

-   囊括了大数据处理的方方面面

-   成熟的生态圈

发行版的选择
============

-   Apache Hadoop （不建议生产环境，冲突难以解决）

-   CDH：Cloudera Distributed Hadoop（搭建方便，部分，有部分改变）

*http://archive.cloudera.com/*

区别版本，hadoop版本和cdh版本可以分开，相关的版本相同即可，避免冲突

-   HDP: Hortonworks Data Platform（市场份额最多，原装Hadoop）

企业应用案例
============

-   消费大数据

    -   Amazon ， 用户消费的大数据分析（在下单之前），提前发送包裹

-   商品零售大数据


