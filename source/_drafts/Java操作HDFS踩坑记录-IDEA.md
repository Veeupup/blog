---
title: Java操作Hadoop踩坑记录(IDEA)
date: 2019-01-23 21:52:24
---

​	家里有事，好几天没写博客了，学习的进度停滞了一两天。本来这个内容实在是应该记录在笔记上而不是博客中，但是我这两天在这方面踩得坑实在让我非常不爽，而且这些内容感觉还比较重要而且可能会对他人有些价值，所以记录下来。

<!--more-->

# 环境：

* win10

* IDEA

* Maven 3

  这里采用了IDEA，用 Maven 3 快速构建项目，免去一些繁琐的重复性操作。

  先说一下步骤，看起来很简单的几步操作。

* IDEA 新建 Maven 3 项目

* 在 pom.xml 中添加 Hadoop 的依赖， 添加 cloudera 的远程仓库

* 在 setting.xml 中添加阿里云加速 （可选，国内）

# 第一个坑

  首先第一个坑，应该是最简单但我耗时最长的一个点，就是添加依赖后使用 Maven 引入，自动下载依赖的过程。

  我反复更改cdh的版本，添加国内的源，Hadoop 相关的很多包都能正常下载完成，但是下载到某一个jar文件的时候，总是失败，现在想想应该是 cdh 的版本没填对，或者是网络问题， 建议填写 依赖的名称和版本的时候去https://mvnrepository.com/ 官网查找响应的名称。

# 第二个坑

​	hdfs正常运行情况下，采用 Java 连接Hadoop中hdfs的时候，总是连接失败，报无法连接的错误，然后在 [stackoverflow ](https://stackoverflow.com/questions/28661285/hadoop-cluster-setup-java-net-connectexception-connection-refused) 上找到了解决办法。

* 首先重置节点
  * `stop-all.sh`
  * `hadoop namenode -format`
  * start-dfs.sh`
* 启动节点并可以用 jps 或者 nmap 等工具查看端口打开情况
* **坑！！！**前面两步还是不能连接的话，按照 帖子中的方法，修改 conf/core-site.xml ，将 localhost 改成 0.0.0.0 （这个点现在都没弄懂为什么）

然后即可连接

# 第三个坑

​	关闭hdfs的安全模式。

```shell
hadoop dfsadmin -safemode leave
```

​	

---

​	希望大家也少踩坑，在有限的时间里，学到更多有实质性内容的东西而不是在这些问题上浪费时间。