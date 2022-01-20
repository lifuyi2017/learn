[TOC]

# 上课须知

课程主题：Spark 分布式计算引擎 SparkCore 第一次课
上课时间：2020-09-21 20:00 - 23:00
课件休息：21:30 左右 休息10分钟
课前签到：如果能听见音乐，能看到画面，请在直播间扣 666 



# 课程预告

```
Spark课程：至少4次
	spark的核心基础知识
	spark的shuffle，内存模型，on yarn，调优
	spark的开发调优 和 数据倾斜调优
	sparkcore的源码
		Spark集群的启动
		Spark的任务提交和执行
	sparksql 两次
	sparkstreaming 两次

flink
	五次课
	
OLAP体系：
	clickhouse 
	kylin
	doris
	kudu 

flink + clickhouse　实时数仓
	三

大数据中台！项目
	五次
		数据治理 atlas 元数据管理 指标管理 数据资产目录管理
		任务调度系统
		数据集成平台（数据的收集和数据的ETL）
		数据的存储（FileSystem DataBase OLAP）
		数据的计算（离线，流式处理，OLAP）
		....
		业务需求的实现
	思维模式的转变
		只有超大型公司才有这个必要，一般的中型量力而为。
		
	核心思想：
		one data
		one service 
	
	中台会用到的技术：12个左右！ kafka hbase  另外10个技术大部分小伙伴，都是不了解的
	
后面至少还有三个项目
```

六个项目：

```
ＨＤＦＳ源码的二次改造
flink + clickhouse　实时数仓
大数据中台！项目
还有至少其他三位老师
```



# 上次课程总结

## HBase三次课

```
HBase架构设计
HBase工作原理
HBase源码
```

```
源码剖析核心三大类型：
1、hbase的集群启动：master启动和hregionserver启动
2、hbase的put和get流程，和region的定位
3、hbase的flush，split，compact三大核心流程
```



## 之前的技术体系

```
存储体系：
	hdfs
	zookeeper
	kafka
	hbase
```

```
计算体系：
	mapreduce
	hive
```

```
工具：
	flume
	canal
```

重头戏：

```
spark
flink
```



# 课程大纲

## MapReduce的缺点分析

见文档1：Spark-part01-01-概述.pdf

```
mapreduce -v1 版本的缺点  2.x 以前
	可扩展性差  JobTracker TaskTracker （包含了现在的 yarn 和 mapreduce）
		mapreduce 运行的时候：MRAppMaster  MapTask/ReduceTask
	可用性差
	资源利用率低
		slot（mapSlot  reudceSlot）  container 
	不能支持多种MapReduce框架
		yarn   专门用来做资源调度的（能运行除了MR以外的其他分布式计算组件的应用程序 spark flink）
		mapreduce 一套编写分布式应用程序的API （除了能运行在 YARN 当中，也能运行在 mesos ）

mapreduce -v2 版本的缺点  2.x
	解决了以上四点。但是依然残留一点是没有办法解决的：
		致命缺点：基于磁盘的分布式计算引擎
```

mapreduce 3.x 据说，也要基于 内存做运算

```
1、大概是这样的额情况：
	原来：spark:MR = 1:100
	现在：spark：mr = 1： 0.1
```

spark-3.x 

```
动态分区
```



## Spark产生背景、发展历程、应用场景

见文档1：Spark-part01-01-概述.pdf



spark产生背景“

```
1、关于 hadoop 体系复杂的问题
	开发，运维，测试。
	解决方案：开发一个一站式的计算引擎（离线计算，流式处理，迭代计算，关系计算，数据分析）
2、解决了关于 hadoop 体系效率低的问题
	mr:  磁盘
	spark: 内存  DAG引擎
```



spark的优势：

```
1、减少磁盘I/O
2、增加并行度
	mr: 任务是进程级别，
	spark： 线程级别
3、避免重新计算
	中间结果进行持久化到内存或者磁盘
4、可选的Shuffle和排序
	MR如果有reducer阶段，就一定会按照key排序！ 但是业务需要不要求key一定有序
	spark：提供多种可选的shuffle方案。根据需求来选择
5、灵活的内存管理策略
	堆外内存offheap 堆内内存onheap
	存储内存storagememory 执行内存executionmemory
```



DAG引擎！



dataRdd.xx1().xx2().xx3();

file.map().reduce()....   80多个



不管数据量多大，不管数据格式是什么，不管计算需求是什么，不管数据存储在哪里

我，spark，flink 都能完成！

source    compute   sink   



MR:  做离线批处理 

spark:  海量数据的各种计算需求



Spark 有一个缺点：

```
流式处理，是一个为实时的流式处理！
新的模块：structure streaming 
```

Flink 为什么 

```
真正的流式处理： storm  数据来一条算一条！
```



spark的发展历程：

```
spark-1.3  
spark-1.6  spark-1.6.3
spark-2.x  spark-2.4.7  spark-2.4.6
spark-3.x
```

spark-.1.6  把  RDD 和 SparkContext 转为幕后。 统一各种上层应用模块的编程的抽象



## Spark企业级高可用集群部署

见文档2：Spark-part01-02-环境搭建.pdf

同时附带录制视频



集群架构：主从架构

```
master
worker
```

两种基本使用方式：

```
spark-shell   交互式编程
spark-submit   shell方式提交任务
```

支持两种编程语言

```
java
scala
```



搭建的是 SPARK 的高可用，同时也安装了 spark的 hsitoryserver，依赖组件就有：

```
zookeeper
hadoop
spark
```



## SparkShell 和 SparkSubmit 使用详解

见文档：Spark-part01-03-Spark Shell 使用.pdf

见文档：Spark-part01-04-Spark Submit提交任务.pdf



启动spark-shell之后：spark-2.x

```
Spark context available as 'sc' (master = spark://bigdata02:7077,bigdata04:7077, app id = app-20200921210558-0000).
Spark session available as 'spark'.
```

启动spark-shell之后：spark-1.x版本“

```
sparkContext  sc  			sparkcore应用程序的编程入口
sqlContext sqlContext       sparkSQL的编程入口
```



spark-2.x中。把三个编程入口统一了：sparkCotnext， sqlContext,  HiveContext  ===> SparkSession

```
local 本地单线程 （指定一个内核）
local[K] 本地多线程（指定K个内核）
local[*] 本地多线程（指定所有可用内核）
spark://HOST:PORT  连接到指定的 Spark standalone cluster master，需要指定端口。
mesos://HOST:PORT  连接到指定的 Mesos 集群，需要指定端口。
yarn-client客户端模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR，然后重启
yarn-cluster集群模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR，然后重启
```

```
spark-shell --master local

$SPARK_HOME/bin/spark-shell \
--master spark://bigdata02:7077,bigdata04:7077 \
--executor-memory 512M \
--total-executor-cores 2
```

底层的区别：

```
spark-shell  没有链接上spark集群，所以你执行的程序就是单机程序
spark-shell  链接集群
```



spark-submit：http://spark.apache.org/docs/2.4.7/submitting-applications.html

```
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \      指定执行的class
  --master yarn \          指定提交任务到那个资源管理系统取运行
  --deploy-mode cluster \				spark的应用程序，有两种运行模式：cluster  client
  --executor-memory 20G \             参数的细节！
  --num-executors 50 \					参数的细节！
  http://path/to/examples.jar \        提交是的那个jar包，jar路径
  1000  x1 x2                               class 程序的参数
```



spark aapplication 

```
driver           mrappmaster			 主控程序 驱动程序  管理这个 application 中task运行的
executor         mapTask reduceTask      真正的运行的task
都是线程级别！     都是进程级别！
```



```bash
--supervise \
spark app driver + exeucotr    driver 有可能会宕机！  --supervise  自动恢复driver
注意要点：
1、不能使用kill 的方式来杀掉 driver
2、yarn -kill jobid
```





## Spark编程模型详解和WordCount

见 IDEA 代码

```
Spark的wordcount四种写法：
1、Spark-shell中的写法
2、java7 的写法  普通的java写法
3、java8 的写法  lambda写法
4、scala 的写法  scala
```

scala 的 spark的编写规范

java的7 和 8 的两种写法！

有一些源码也这么干！ 



## Spark的核心功能，应用模块，基本架构

见文档：Spark-part02-07-Spark的体系结构.pdf

核心功能：

```
1、SparkContext
	编程入口，spark计算引擎非常复杂，但是编程很简单，必然做了大量的封装。
		DAG引擎
			两大核心组件：
				DAGScheduler  帮助我们把一个 spark app 构建成一个 DAG 有向无环图，同时也切分stage
				TaskScheduler  接收一个stage转变成一个TaskSet去在提交执行
			通信组件：
				BackEnd   SchdudlerBackend   ExeutorBackend
				app		  driver             executor
2、存储体系
	BlockManager
		spark app 从HDFS 读取数据，写入结构到HDFS 都是通过 BlockManager 来实现
3、计算引擎	
	DAG
4、部署模式
	local          简单的任务，测试，开发的
	standalone      使用spark自带的资源管理系统：spark集群 master worker
	yarn			driver + executor ，这就不要启动spark集群，只要有spark-submit客户端就行
```

应用模块：一站式的

```
1、sparkcore        离线批处理核心引擎的实现
2、sparksql         基于 sparkcore 结构化数据
3、sparkstreaming   基于sparkcore实现的流式处理
4、spark graghx     图计算 
5、spark mllib       机器学习
6、sparkR            支持 R语言分析
7、PYSPARK           支持python数据分析
8、structure streaming     基于pipline的流式处理，为了补充和完善spark streaming的功能
```

基本架构：

```
注意几对概念：
1、master worker
2、driver executor
3、application job stage task
```

clusterManager

```
资源管理系统的主节点
resourcemanager， master, .....
```



## Spark的编程模型，运行机制，核心概念

```
*   注释： 编程套路
*   1、获取编程入口                    环境对象 链接对象
*   2、通过编程入口获取数据抽象         Source 对象
*   3、针对数据抽象对象执行各种计算      Action Transformation
*   4、提交任务运行                   Submit
*   5、输出结果                      Sink
*   6、回收资源                      stop close
```

```
sparksql sparkcore saprkstreaming 
```

见文档：Spark-part04-Spark应用程序运行机制.pdf

spark的计算程序：

```
从 编程入口入口 写代码
获取到了 数据抽象  执行各种操作
```

```
抽象：
1、HDFS： FileSytem
2、ZooKeeper: ZooKeeper　Watcher
3、MpaReduce: Job
4、Spark
	SparkSession DataSet
```

编程模型：Spark-1.x 版本

|          | SparkCore    | SparkSQL              | SparkStreaming   |
| -------- | ------------ | --------------------- | ---------------- |
| 编程入口 | SparkContext | SQLContext HiveCotext | StreamingContext |
| 数据抽象 | RDD          | DataFrame             | DStream          |

编程模型： Spark-2.x 版本， 原则是为了统一：

```
数据抽象：DataSet, 原有的这种组件都转为幕后
编程入口；SparkSession
```

|          | SparkCore                 | SparkSQL            | SparkStreaming   |
| -------- | ------------------------- | ------------------- | ---------------- |
| 编程入口 | SparkContext/SparkSession | SparkSession        | StreamingContext |
| 数据抽象 | RDD / DataSet             | DataFrame / DataSet | DStream          |

运行机制：

```
提交任务的时候：
	 JobSubmitted
	 SchedulerBackEnd  sunmitTask（LaunchTask）  ExecutorBackend
```

核心概念：

```
application
job
stage
task
```

RDD

```
分布式集合（由分散在多个不同worker节点上的多个partition组成）
map set 在一个JVM 里面
```

mapreduce

```
mapper			word ==> word,1     wordRDD  --- map ---> wordAndOneRDD 
reduce
```



```
mapreduce 
	mapper 
			shuffle
	reducer
mapreduce 
	mapper 
			shuffle
	reducer
mapreduce 
	mapper 
			shuffle
	reducer
	
	
spark
	stage1
		shuffle
	stage2
		shuffle
	stage3
		shuffle
	.....
	
	任何一个stage执行的结果，都可以进行持久化保存！　（磁盘，内存）
```



```
DAG 
	DAGScheduler 负责把 构建出来的   DAG 切分成多个stage   
	标准：shffle算子 ShuffleDenpedency
	RDD与RDD之间的转换关系依赖：
		宽依赖 ShuffleDenpedency
 		窄依赖  NarrowDependency
Stage
	一个stage中，其实可能包含多个RDD，和多个计算逻辑，
	但是没关系，上下RDD之间的依赖关系就只是普通的一一对应的关系
	RDD与RDD之间就是窄依赖
	在一个stage之间，所有的RDD的依赖都是窄依赖。
	上一个stage的最后䘝 RDD 和下一个stage的第一个RDD之间是宽依赖
	
一个stage中到底有多少个taask: 由当前这个stage的最后一个RDD的分区个数来决定
```



```
worker  master
exeuctor  driver
applicatioin  job  stage  task
```



```
如果一个stage中的一个Task执行失败，直接从这个stage的第一个RDD的对应分区直接从新计算即可
如果是下一个RDD的某个分区的数据丢失了，上一个stage中的所有的Task都得重新重新执行。

宽依赖：下一个RDD的某个分区，只依赖于上一个RDD的多个分区
窄依赖：下一个RDD的某个分区，只依赖于上一个RDD的一个分区

一个分区有多个爹， 宽依赖
一个分区如果只有一个爹, 窄依赖

一个爹只有一个儿子，独生，窄依赖
一个爹有多个儿子，超生，宽依赖
```



## RDD详解和SparkContext详解

见文档：Spark-part03-08-Spark的RDD详解.pdf

RDD: 

RDD[T]

RDD[<K,V>]

```
 *  - A list of partitions
 每个RDD有多个分区组成
 *  - A function for computing each split
 每个rdd都有作用于每个分区的一个算子
 
 每个RDD都有与上下游有依赖的RDD， RDD之间是有依赖关系的
 *  - A list of dependencies on other RDDs
 
 针对keyv-value 可以指定分区器
 *  - Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
 
 每个RDD的分区，其实都有副本的概念，到底选用那个副本来计算，给每个分区都计算出来了一个最后位置
 *  - Optionally, a list of preferred locations to compute each split on (e.g. block locations for
 *    an HDFS file)
```



## RDD的算子企业实战和原理剖析

见文档：Spark-part03-08-Spark的RDD详解.pdf



## RDD的缓存机制和Spark的共享变量

见文档：Spark-part03-08-Spark的RDD详解.pdf



# 总结