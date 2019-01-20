---
title: 分布式文件系统HDFS
date: 2019-01-20 14:10:34
tags:
- Hadoop
categories:
- 大数据
---

```
HDFS的学习笔记以及实践记录，全文较长。
多学习官网文档。
```

http://hadoop.apache.org

<!--more-->

# HDFS概述及设计目标

## 什么是HDFS

* Hadoop实现了一个分布式文件系统（Hadoop Distributed File System），简称HDFS
* 源自Google的GFS论文

（Spark也是根据Google的一篇论文的实现）

* 论文发表于2003年，HDFS是GFS的克隆版

## 设计目标

* 非常巨大的分布式文件系统
* 运行在普通廉价的硬件上
* 易拓展、为用户提供性能不错的文件存储服务



# HDFS架构

官网架构图, 建议看官网原文。

> HDFS has a master/slave architecture. An HDFS cluster consists of a single NameNode, a master server that manages the file system namespace and regulates access to files by clients. In addition, there are a number of DataNodes, usually one per node in the cluster, which manage storage attached to the nodes that they run on. HDFS exposes a file system namespace and allows user data to be stored in files. Internally, a file is split into one or more blocks and these blocks are stored in a set of DataNodes. The NameNode executes file system namespace operations like opening, closing, and renaming files and directories. It also determines the mapping of blocks to DataNodes. The DataNodes are responsible for serving read and write requests from the file system’s clients. The DataNodes also perform block creation, deletion, and replication upon instruction from the NameNode.

![](http://ww1.sinaimg.cn/large/9d82e933ly1fzd275cin3j20oa0gsjs7.jpg)

架构：

* 1 **Master**（NameNode/NN） 带 N个**Slaves** (DataNode/DN)

​	HDFS/YARN/HBase

* 一个文件被拆分成多个Block

  blocksize：128M， ex：130M =》 128M + 2M

  这些block被存放到一系列DN中

* NN：

  * 负责客户端请求的响应
  * 负责元数据（文件的名称、副本系数、Block存放的CDN）的响应

* DN：

  * 存储用户的文件对应的数据块（Block）
  * 要定期向NN发送心跳信息，汇报本身及其所有的block信息，健康状况

* Client

  * 客户端存放HDFS shell ， JAVA API 的程序等

* 运行在廉价的机器上（通常Linux系统，开发使用Java语言），部署广泛

  > A typical deployment has a dedicated machine that runs only the NameNode software. Each of the other machines in the cluster runs one instance of the DataNode software. The architecture does not preclude running multiple DataNodes on the same machine but in a real deployment that is rarely the case.

* 一个集群中一台机器仅仅运行一个NN节点+n个DN节点

# HDFS副本机制

* replication factor：副本系数，副本因子

* 信息存放到NN中

* 一个文件中除了最后一个block，其他文件大小都相同
* HDFS中的文件不支持多并发写，只能写一次 write-once
* DD定期发送  Heartbeat and Blockreport

> HDFS is designed to reliably store very large files across machines in a large cluster. It stores each file as a sequence of blocks. The blocks of a file are replicated for fault tolerance. The block size and replication factor are configurable per file.
>
> All blocks in a file except the last block are the same size, while users can start a new block without filling out the last block to the configured block size after the support for variable length block was added to append and hsync.
>
> An application can specify the number of replicas of a file. The replication factor can be specified at file creation time and can be changed later. Files in HDFS are write-once (except for appends and truncates) and have strictly one writer at any time.
>
> The NameNode makes all decisions regarding replication of blocks. It periodically receives a Heartbeat and a Blockreport from each of the DataNodes in the cluster. Receipt of a Heartbeat implies that the DataNode is functioning properly. A Blockreport contains a list of all blocks on a DataNode.

![](http://ww1.sinaimg.cn/large/9d82e933ly1fzd2xhhjmxj20oa0ewaal.jpg)

## 副本存放策略

将文件存放在不同的Rack 上。

![](http://ww1.sinaimg.cn/large/9d82e933ly1fzd35g6o75j20dc0a3acc.jpg)

# HDFS环境搭建

* Hadoop版本： hadoop-2.6.0-cdh5.7.0  [CDH](https://archive.cloudera.com/cdh5/cdh/5/)

  

# HDFS shell
# Java API操作HDFS
# HDFS文件读写流程
# HDFS优缺点