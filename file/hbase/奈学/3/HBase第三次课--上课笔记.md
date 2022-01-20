[TOC]

# 上课须知

课程主题：HBase 分布式数据库 第三次课
上课时间：2020-09-18 20:00 - 23:00
课件休息：21:30 左右 休息10分钟
课前签到：如果能听见音乐，能看到画面，请在直播间扣 666 



# 上次课程总结

```
架构设计和工作原理
	zookeeper client master regionserver
	rowkey  family  qualifier  timestamp value 
	table  region  store  memstore  storefile  HFile/keyvalue
	两个重要的流程：put  get
	三个重要的行为： flush  split  compact 
建表设计高级操作
	建表的原则：
		表名
		列簇
		rowkey
		建表的高级属性/列簇的属性  
过滤器
	三个最重要的东西：
	1、比较运算符
	2、比较器
	3、过滤器
布隆过滤器
	
协处理器Coprocessor
	observer
	endpoint
```



# 课程大纲

## 完整的源码剖析

HDFS的源码：

```
1、集群启动
	namenode启动
		元数据加载
		安全模式
	datanode启动
2、上传流程
3、下载流程
```

HBase的源码：

```
1、集群启动
	master启动
	regionsever启动
2、put 插入数据源码分析
	delete
3、get 查询数据源码分析
	scan
4、三个重要的行为：
	flush
	split
	compact
5、寻路
	根据rowkey定位region的位置
```

注意要点：

```
1、版本问题：
	hbase-0.96  hbase-0.98 
	hbase-1.2.x  hbase-1.4.x
	hbase-2.x  这是我们选择的版本
	
2、版本越低，其他非核心流程的代码相对更少，对你的干扰更少

3、切记最好画图！

4、关于hbase的源码是否要编译呢。
	选择性的。如果要编译：推荐安装 cygwin 
	zookeeper 
	hive
	hdfs
	kafka
	hbase 
```



## HBase2.x源码分析-HMaster启动流程分析

正常的启动历程：

```
启动一个 master 
再启动 N 个 regionserver
启动一个 master 
```

在这个过程中，会有选举！大概率：先启动的master会成为 active 

```
HBase集群启动的命令：start-hbase.sh
这个命令的底层是：
	hbase-daemon.sh start master
这个命令的底层：
	java org.apache.hadoop.hbase.master.HMaster arg1 arg2 
底层转到
	HMaster.main()
```



```
class HMaster extends HRegionServer implements MasterServices
1、HMaster 是 HRegionServer 的子类
2、HMaster 绝大部分的服务，都是通过 MasterServices 来实现的
	比如：listTable()  craeteTable()
```



```
HMaster.main()
	startMaster()
		HMaster 的构造器
			父类regionserver的构造方法  super(conf);
				rpcServices = createRpcServices();
				RpcRetryingCallerFactory.instantiate(this.conf);
				regionServerAccounting = new RegionServerAccounting(conf);
				initializeFileSystem();
				zooKeeper = new ZKWatcher()
					this.znodePaths = new ZNodePaths(conf);
					ZKUtil.connect(conf, quorum, pendingWatcher, identifier);
					createBaseZNodes();
				this.rpcServices.start(zooKeeper);
				this.choreService = new ChoreService(getName(), true);
            	this.executorService = new ExecutorService(getName());
				putUpWebUI();
			HMaster 自己构造器里面的代码
				this.activeMasterManager = new ActiveMasterManager(....)
		HMaster 的run()
			int infoPort = putUpJettyServer(); 
			startActiveMasterManager(infoPort);
				竞争active 
					创建一个代表自己上线的临时znode节点
					阻塞
					activeMasterManager.blockUntilBecomingActiveMaster(timeout, status)
						MasterAddressTracker.setMasterAddress()  抢锁
							如果成功，则成为 active，同时删除之前创建的 backup znode 
							如果失败，就去发现到底谁是 active，监听当前active master的znode节点 
				完整hmaster的各种组件的初始化
					finishActiveMasterInitialization(status);
						大概50个各种组件（xxxManager）
```



## HBase2.x源码分析-HRegionServer启动流程分析



```
HBase集群启动的命令：start-hbase.sh
这个命令的底层是：
	hbase-daemon.sh start regionserver
这个命令的底层：
	java org.apache.hadoop.hbase.regionserver.HRegionServer arg1 arg2 
底层转到
	HMaster.main()
```

```
class HRegionServer extends HasThread implements RegionServerServices
1、HMaster 是 HRegionServer 的子类， HRegionServer 是 HasThread 的子类
HasThread 是 RUnnable 的子类，所以  HMaster 和  HRegionServer 都是线程
入口：main()  run() 方法
2、HRegionServer 的绝大部分服务都是通过 RegionServerServices 来提供的
	比如：put 和 get 等操作
```

HRegionServer的总结：

```
HRegionServer.main()
	HRegionServer 的构造方法
	HRegionServer 的 run()
		preRegistrationInitialization();
			initializeZooKeeper();
			setupClusterConnection();
			this.rpcClient = RpcClientFactory.createClient()
		reportForDuty();
			masterServerName = createRegionServerStatusStub(true);
			this.rssStub.regionServerStartup(null, request.build());
		handleReportForDutyResponse(w);
			createMyEphemeralNode();
			initializeFileSystem();
			ZNodeClearer.writeMyEphemeralNodeOnDisk(getMyEphemeralNodePath());
			setupWALAndReplication();
			startServices();
			startReplicationService();
		tryRegionServerReport(lastMsg, now);
			rss.regionServerReport(null, request.build());
```



## HBase2.x源码分析-Put流程源码分析

核心流程：

```
代码入口：
table.put(new Put())

大致流程：
1、首先客户端会请请求zookeepre拿到meta表的位置信息，这个meta表的region到底在那个regionserver1里面
2、发请求请求这个regionserver1扫描这个meta表的数据，确定我要插入的数据rowkey到底在那个用户表的region里面。并且还拿到这个region在那个regioinserver2的信息
3、发送请求，请求regionserver2扫描 当前用户表的regioin, 执行插入
	1、先记录日志
	2、写入数据到 memstore
		写到  ConcurrentSkipListMap delegatee
	3、判断是否需要进行flush
		1、再次判断是否需要进行 compact
		2、判断是否需要进行 split
```



```
batchMutate(BatchOperation<?> batchOp)
作用：完成一批次 Mutation 操作
参数：Mutation[] ===> BatchOperation
```

三大核心和步骤：

```
1、checkResources();
2、doMiniBatchMutate(batchOp);
3、requestFlushIfNeeded();
```



## HBase2.x源码分析-Region定位分析

```
connection.locateRegion(){
    locateMeta(tableName, useCache, replicaId);
    locateRegionInMeta(tableName, row, useCache, retry, replicaId);
}
```



## HBase2.x源码分析-Flush一些Memstore形成HFile

```
reqeustFlush()
	flushRegion(fqe)
		region.flushcache
		flushResult.isCompactionNeeded();
		region.checkSplit() +  this.server.compactSplitThread.requestSplit(region);
```



## HBase2.x源码分析-split分析

```
reqeustSplit()
```



## HBase2.x源码分析-Compact分析

```
reqeustCompact()
```



## HBase2.x源码分析-Get流程源码分析

```
代码入口：
table.get(new Get())

大致流程：
1、首先客户端会请请求zookeepre拿到meta表的位置信息，这个meta表的region到底在那个regionserver1里面
2、发请求请求这个regionserver1扫描这个meta表的数据，确定我要插入的数据rowkey到底在那个用户表的region里面。并且还拿到这个region在那个regioinserver2的信息
3、发送请求，请求regionserver2
	1、首先去blockcache ,进行查询  读缓存
	2、先去布隆过滤器中进行判断
		1、如果判断这个rowkey不存在，则不需要扫描 HFile
		2、如果判断这个rowkey存在，则需要扫描HFile
```



# 总结

