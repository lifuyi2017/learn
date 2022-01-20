[TOC]

# 上课须知

课程主题：Spark 分布式计算引擎 SparkCore 第三次课 - 源码剖析
上课时间：2020-09-28 20:00 - 23:00
课件休息：21:30 左右 休息10分钟
课前签到：如果能听见音乐，能看到画面，请在直播间扣 666 



# 上次课程总结

数据倾斜调优：10个招术

```
具体见上次文档
```

开发调优：10个招术

```
具体见上次文档
```



# 本次内容大纲

内容大纲：

```
# Spark RPC 分析
# start-all.sh 脚本分析
# Master 启动分析
# Worker 启动分析
# spark-submit.sh 脚本分析
# SparkSubmit 分析
# SparkContext 初始化
```



总共三大块

```
1、rpc
2、集群启动
	启动脚本分析
	master启动
	worker
3、任务提交执行
```



# Spark RPC

见文档！



# start-all.sh 脚本分析

启动命令

```
start-all.sh
```

内部执行：

```
start-master.sh  
start-slaves.sh
```

都共同调用：

```
spark-daemon.sh 
```

内部调用：

```
spark-class
```

最终转到执行：

```
Master启动：spark-class org.apache.spark.deploy.master.Master
Worker启动：spark-class org.apache.sprak.deploy.worker.Worker
```



# Master 启动分析

根据以上的脚本分析：

```
org.apache.spark.deploy.master.Master.main()
```



# Worker 启动分析

根据以上的脚本分析：

```
org.apache.sprak.deploy.worker.Worker.main()
```



# 休息 12 分钟，22:00 继续





# spark-submit 脚本分析

写好一个程序，打成jar包，然后通过 spark-submit 提交

```
spark-class org.apache.spark.deploy.SparkSubmit $CLASS
spark-class org.apache.spark.deploy.SparkSubmit SparkPi
```



# SparkSubmit 分析



# SparkContext 初始化

重点关注对象：

```
1、TaskScheduler， 实现类是 ： TaskSchdulerImpl
2、DagScheduler 
	DAGSchedulerEventProcessLoop 初始化和启动两件事
3、SchdulerBackEnd
	backend.start()
	有两个成员变量：
	1、负责和 worker 通信： DriverEndpoint
	2、负责和 master 通信： StandaloneAppClient 
							 封装了 ClientEndpoint
```



# 总结

