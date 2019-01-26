---
title: 搭建Hadoop单节点伪分布式记录
date: 2019-01-20 16:34:40
tags:
- Hadoop
categories:
- 大数据
---

​	记录搭建hadoop搭建单节点过程，中间不可避免踩了很多坑，都记录下来。

<!--more-->

环境：

* 服务器版本：ubuntu-18.04-live-server-amd64
* JDK版本： 1.7.0_79
* CHD版本：hadoop-2.6.5-src.tar.gz

# jdk安装

​	由于目前ORACLE官网需要 同意协议才能下载，所以直接 wget下载jdk后无法正常解压，出现 错误。

```shell
gzip：stdin not in gzip format
```

​	所以在其他平台下载之后采用FTP 上传至服务器。

​	然后解压

```shell
sudo mkdir /usr/lib/jvm     # 为jdk解压创建文件夹
sudo tar zxvf jdk-7u79-linux-x64.tar.gz -C /usr/lib/jvm  
sudo vim /etc/profile 		#设置环境变量
```

​	设置好环境变量

```shell
#set java environment
 export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_79
 export JRE_HOME=${JAVA_HOME}/jre
 export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
 export PATH=${JAVA_HOME}/bin:$PATH
```

​	设置默认的jdk

```shell
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.7.0_79/bin/java 300
 sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.7.0_79/bin/javac 300
```

​	测试是否配置成功

```shell
java -version
```

​	如果输出如下

```shell
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```

​	则配置成功。

# 安装配置ssh

```shell
sudo apt-get install ssh
sudo apt-get install rsync	
```

配置ssh

```shell
ssh-keygen -t rsa
cp id_rsa.pub ~/.ssh/authorized_keys
```



# 下载配置Hadoop

下载地址   http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0.tar.gz>

```shell
wget http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.7.0.tar.gz
```

解压

```shell
mkdir app
tar -zxvf hadoop.xxxxx.tar.gz -C ~/app/
```

目录解释

```shell
drwxr-xr-x  2 1106 4001  4096 Mar 23  2016 bin	# 客户端，其中 cmd文件为windows下的脚本
drwxr-xr-x  2 1106 4001  4096 Mar 23  2016 bin-mapreduce1
drwxr-xr-x  3 1106 4001  4096 Mar 23  2016 cloudera
drwxr-xr-x  6 1106 4001  4096 Mar 23  2016 etc	# 其中hadoop中有配置文件
drwxr-xr-x  5 1106 4001  4096 Mar 23  2016 examples
drwxr-xr-x  3 1106 4001  4096 Mar 23  2016 examples-mapreduce1
drwxr-xr-x  2 1106 4001  4096 Mar 23  2016 include
drwxr-xr-x  3 1106 4001  4096 Mar 23  2016 lib
drwxr-xr-x  2 1106 4001  4096 Mar 23  2016 libexec
-rw-r--r--  1 1106 4001 17087 Mar 23  2016 LICENSE.txt
-rw-r--r--  1 1106 4001   101 Mar 23  2016 NOTICE.txt
-rw-r--r--  1 1106 4001  1366 Mar 23  2016 README.txt
drwxr-xr-x  3 1106 4001  4096 Mar 23  2016 sbin	# 启动、停止脚本集合
drwxr-xr-x  4 1106 4001  4096 Mar 23  2016 share# 其中hadoop中有很多例子包
drwxr-xr-x 17 1106 4001  4096 Mar 23  2016 src

```

​	配置，

```shell
sudo vim etc/hadoop/hadoop-env.sh
```

​	修改JAVA_HOME的值。

​	看情况是否需要添加环境变量

```
export JAVA_PREFIX=**地址**
```

# hdfs

## 配置hdfs

修改站点等设置。

​	core-site.xml

```xml
<property>
       	<name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:9000</value>
</property>
<property>
       	<name>hadoop.tmp.dir</name>
        <value>/home/tpp/app/tmp</value>
</property>
```

​	hdfs-site.xml

此处副本系数为1，默认hadoop为3，所以此处需要注意。

```xml
<property>
        <name>dfs.replication</name>
        <value>1</value>
</property>
```

(单机伪分布式不需要)

slaves

- 默认localhost

## 启动hdfs

* 格式化文件系统（仅仅第一次执行，不要重复执行）：

```shell
bin/hdfs namenode -format
```

* 启动 NameNode 和 DataNode 守护进程

```shell
sbin/start-dfs.sh
```

The hadoop daemon log output is written to the `$HADOOP_LOG_DIR` directory (defaults to `$HADOOP_HOME/logs`).

* 验证是否启动 
  * jps 查看是否启动
    * 如果未安装，apt安装即可
  * 在浏览器访问
    * http://192.168.224.131:50070/

## 停止hdfs

​	执行命令

```shell
sbin/stop-dfs.sh
```

​	可以用jps命令查看是否停止，或者浏览器也可以查看。

# YARN

## 配置YARN

* 将MapReduce 配置到 YARN 上，修改 

  etc/hadoop/mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

​	etc/hadoop/yarn-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

## 启动YARN

```shell
sbin/start-yarn.sh
```

## 浏览YARN

http://localhost:8088/

## 停止YARN

```shell
sbin/stop-yarn.sh
```

