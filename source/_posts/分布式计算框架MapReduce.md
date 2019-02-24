---
title: 分布式计算框架MapReduce
date: 2019-01-25 15:16:45
tags:
- Hadoop
categories:
- 大数据

---

​	这两天因为换了新电脑，一直沉迷搞机，学习进度也降低许多，今天把博客也迁移了，以后要更努力的学习。

<!--more-->

# MapReduce概述

* 源自Google的MapReduce论文，发表于2004年12月
* Hadoop中的MapReduce是 Google MapReduce的克隆版
* MapReduce优点：海量数量离线处理&易开发（比不上spark）&易运行
* MapReduce缺点：无法进行实时流式计算

# MapReduce编程模型

## wordcount

wordcount：统计文件中每个单词出现的次数。

* 如果文件内容小：shell 编程
* 文件内容比较大：GB, TB
  * 借助分布式计算框架：MapReduce，分而治之

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzivf4rcw2j20fp07dgnq.jpg)

## map和reduce阶段

* 将作业拆分成Map阶段和Reduce阶段
* Map阶段：Map Tasks
* Reduce阶段： Reduce Tasks

## 执行步骤

* 准备map处理的输入数据
* Mapper处理
* Shuffle 
* Reduce 处理
* 结果输出

(input) `<k1, v1> ->` **map** `-> <k2, v2> ->` **combine** `-> <k2, v2> ->` **reduce** `-> <k3, v3>`(output)

![](https://ws1.sinaimg.cn/large/9d82e933gy1fziw01bywsj20tx0pmtj5.jpg)

## 核心概念

* Split
  * 交由MapReduce作业处理的数据块，是MapReduce中最小的计算单元
  * 默认情况下一一对应，当然也可以手工设置
  *  HDFS：blocksize是HDFS中最小的存储单元  128M
* InputFormat
  * 将我们的输入数据进行分片（split）
  * 用的比较多 TextInputFormat
* OutputFormat
  * 输出

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzj3lgd6cej20j40emgxq.jpg)



# MapReduce架构

## MapReduce1.x架构图

* JobTracker（JT）
  * 作业的管理者
  * 将作业分解成一堆的任务：Task（MapTask和ReduceTask）
  * 将任务分派给TaskTracker（从节点）运行
  * 作业的监控和容错处理（Task作业挂了，重启task的机制）
  * 在一定的时间间隔内，JT没有收到TT的心跳信息（HeartBeat），TT 可能是挂了，TT上运行的任务会被指派到其他TT 上去执行
* TaskTracker（TT）
  * 任务的执行者 
  * 在TT上执行Task（MapTask和ReduceTask）
  * 与JT进行交互：执行/启动/停止作业，发送heartbeat给JT
* MapTask
  * 自己开发的map任务交由该Task处理
  * 解析每行记录的数据交给自己的map方法处理
  * 将map的输出结果写到本地磁盘（有些作业仅有map没有reduce==》HDFS）
* ReduceTask
  * 将MapTask输出的数据进行读取
  * 按照数据进行分组传给我们自己编写的reduce方法进行处理
  * 输出结果写到HDFS

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzj3ogove8j20er0a2jtd.jpg)

## MapReduce2.x架构

- ResourceManager:	RM

  - 整个集群提供服务的RM在同一时间只有一个（生产环境通常有另外一个备份的RM），负责集群资源的统一管理和调度。
  - 处理客户端的请求：提交一个作业、杀死一个作业
  - 监控 NM，一旦某个  NM 挂了，该DM上运行的任务需要通知 AM来如何进行处理

  

- NodeManager:     NM

  - 整个集群中有多个，负责自己本身节点资源管理和使用
  - 定时向RM汇报本节点的资源使用情况
  - 接收并处理来自  RM 的各种命令：启动Container
  - 处理来自 AM 的命令
  - 单个节点的资源管理

  

- ApplicationMaster:     AM

  - 每个应用程序对应一个：MR、Spark，负责应用程序的管理
  - 为应用程序向  RM  申请资源（core、memory），分配给内部的task
  - 需要与 NM  通信：启动/停止task，task是运行在  container  里面，AM也是运行在 container 里面

- Container

  - 封装CPU、Memory 等资源的一个容器
  - 是一个任务运行环境的抽象

- Client

  - 提交作业
  - 查看作业的运行进度
  - 杀死作业 

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzj3xp8q7jj20es0ae40p.jpg)

# MapReduce编程

## Java版本wordcount功能实现

​	在Java中，我们新建一个MapReduce的app类，然后在其内部写两个 子类 分别继承 Mapper 类和 Reducer 类，重写Mapper类的Map方法（图中过程Map，分割），重写 Reducer 类的 reduce 方法（对应过程reduce，合并）

​	然后在主方法中设置相应的配置和参数。

​	导出成 jar  文件，放到 MapReduce  上执行即可。

​	这里附上源代码，如果输出预先存在，会报错，那么有两种方法：

* 手工shell删除输出结果
* 编写shell自动删除

```java
package com.tpp.hadoop.mapreduce;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * 使用MapReduce开发WordCount应用程序
 */
public class WordCountApp {

    /**
     * Map读取输入的文件
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable>{

        LongWritable one = new LongWritable(1);

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

            // 接收到的每一行数据
            String line = value.toString();

            // 接收到的每一行数据，按照指定分隔符进行拆分
            String[] words = line.split(" ");

            for(String word : words){
                // 通过上下文把map处理的结果输出
                context.write(new Text(word), one);
            }

        }
    }

    /**
     * Reduce:归并操作
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable>{

        @Override
        protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {

            long sum = 0;
            for(LongWritable value : values){
                // 求单词key出现的次数总和
                sum+=value.get();
            }

            context.write(key, new LongWritable(sum));
        }
    }

    /**
     * 定义Driver：封装MapReduce作业的所有信息
     * @param args
     */
    public static void main(String[] args) throws Exception{

        // 创建configuration
        Configuration configuration = new Configuration();

        // 创建Job
        Job job = Job.getInstance(configuration, "wordcount");

        // 设计job的处理类
        job.setJarByClass(WordCountApp.class);

        // 设置作业处理的输入路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        // 设置map相关的参数
        job.setMapperClass(MyMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        //设置reduce相关参数
        job.setReducerClass(MyReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 设置作业处理的输出路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);

    }

}

```

附上shell，注意这里的主类的名称。

```shell
hadoop jar /home/tpp/mapreduce/hadooptest.jar com.tpp.hadoop.mapreduce.WordCountApp hdfs://0.0.0.0:8020/hello.txt hdfs://0.0.0.0:8020/output/wc
```

## Combiner

* 本地的reducer（一个小的reducer）
* 能够减少Map Tasks 输出的数据量以及数据网络传输量
* combiner案例开发
* 适用场景：
  * 求和、次数（求平均数等结果不相同）

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzjrif1lm1j20ic0dtaee.jpg)

​	只需要在MapReduce编程中配置job的参数时，加上combiner的处理类。

```java
// 通过job的设置combiner处理类，其实逻辑上和reduce一模一样
job.setCombinerClass(MyReducer.class);
```

## Partitioner

* Partitioner 决定MapTask输出的数据交由哪个ReduceTask处理
* 默认实现：分发的key的hash值对Reduce Task个数取模
* Partitioner 案例开发

![](https://ws1.sinaimg.cn/large/9d82e933gy1fzjryj64g7j20dr0df793.jpg)

给任务写上 partition 类，然后给job设定好相应的配置即可。

```java
    public static class MyPartitioner extends Partitioner<Text, LongWritable>{

        @Override
        public int getPartition(Text key, LongWritable value, int i) {

            if(key.toString().equals("xiaomi")){
                return 0;
            }

            if(key.toString().equals("huawei")){
                return 1;
            }

            if(key.toString().equals("iphone7")){
                return 2;
            }

            return 3;
        }
    }
```

核心代码：

```java
 // 设置job的partition
 job.setPartitionerClass(MyPartitioner.class);
// 设置四个reducer，每个分区一个
job.setNumReduceTasks(4);
```

附上源码。

```java
package com.tpp.hadoop.mapreduce;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class PartitionerApp {

    /**
     * Map读取输入的文件
     */
    public static class MyMapper extends Mapper<LongWritable, Text, Text, LongWritable>{

        LongWritable one = new LongWritable(1);

        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

            // 接收到的每一行数据
            String line = value.toString();

            // 接收到的每一行数据，按照指定分隔符进行拆分
            String[] words = line.split(" ");

            context.write(new Text(words[0]), new LongWritable(Long.parseLong(words[1])));

        }
    }

    /**
     * Reduce:归并操作
     */
    public static class MyReducer extends Reducer<Text, LongWritable, Text, LongWritable>{

        @Override
        protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException {

            long sum = 0;
            for(LongWritable value : values){
                // 求单词key出现的次数总和
                sum+=value.get();
            }

            context.write(key, new LongWritable(sum));
        }
    }

    public static class MyPartitioner extends Partitioner<Text, LongWritable>{

        @Override
        public int getPartition(Text key, LongWritable value, int i) {

            if(key.toString().equals("xiaomi")){
                return 0;
            }

            if(key.toString().equals("huawei")){
                return 1;
            }

            if(key.toString().equals("iphone7")){
                return 2;
            }

            return 3;
        }
    }


    /**
     * 定义Driver：封装MapReduce作业的所有信息
     * @param args
     */
    public static void main(String[] args) throws Exception{

        // 创建configuration
        Configuration configuration = new Configuration();

        // 创建Job
        Job job = Job.getInstance(configuration, "wordcount");

        // 设计job的处理类
        job.setJarByClass(PartitionerApp.class);

        // 设置作业处理的输入路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        // 设置map相关的参数
        job.setMapperClass(MyMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);

        //设置reduce相关参数
        job.setReducerClass(MyReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(LongWritable.class);

        // 设置job的partition
        job.setPartitionerClass(MyPartitioner.class);
        // 设置四个reducer，每个分区一个
        job.setNumReduceTasks(4);

        // 设置作业处理的输出路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);

    }

}
```

## JobHistory

* 记录已运行完的MapReduce信息到指定的HDFS目录下
* 默认是不开启的
* 修改配置 /etc/hadoop/mapred-site.xml 中添加

```xml
<!-- 设置jobhistoryserver 没有配置的话 history入口不可用 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>localhost:10020</value>
</property>

<!-- 配置web端口 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>localhost:19888</value>
</property>

<!-- 配置正在运行中的日志在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.intermediate-done-dir</name>
    <value>/history/done_intermediate</value>
</property>

<!-- 配置运行过的日志存放在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.done-dir</name>
    <value>/history/done</value>
</property>
```

​	停止yarn再启动。

​	同时启动history服务器。

```shell
sbin/mr-jobhistory-daemon.sh start historyserver	//启动
sbin/mr-jobhistory-daemon.sh stop historyserver		//停止
```

​	jps可查看开启的history服务器，同时，浏览器可以访问 19888 端口查看history，记录在 HDFS  上的/history 目录查看日志。

​	此时浏览器报错，我们要开启日志聚合功能，修改 yarn-site.xml。

```xml
 <!-- 开启日志聚合 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
<!-- 日志聚合目录 -->
<property>
<name>yarn.nodemanager.remote-app-log-dir</name>
<value>/user/container/logs</value>
</property> 
```

