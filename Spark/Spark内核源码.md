# Spark内核源码

### 宽依赖与窄依赖

- 窄依赖

  Narrow Dependency，一个RDD，对他的父RDD，只有简单的一对一关系；RDD的一个partition，仅仅依赖父RDD中的一个partition。

- 宽依赖

  Shuffle Dependency，一个子RDD的partition来自多个父RDD的partition。具有错综复杂的关系，这种情况就叫做宽依赖。同时他们之间的操作叫shuffle

### 任务提交模式

- standalone模型

  基于Spark自己的Master-Worker集群。建议在不同的机器上执行任务，平衡网卡流量。

- 与Hadoop结合

  将原来的spark-submit脚本，添加-master参数，设置为yarn-cluster,yarn-client，如果没有设置，默认是standalone模式

  - yarn-cluster模式

    用于生产模式，因为driver运行在nodemanager，没有网卡激增的问题；缺点：调试不方面，本地spark-submit提交后，看不到日志，只能通过yarn-application的命令查看，很麻烦。

  - yarn-client模式

    主要用于测试，因为driver运行在本地客户端，负责调度application，会导致yarn集群产生超大量的网络通信，导致网卡流量激增；好处是本地可以看到所有的日志，方面调试。

  - 两种模式的区别在于，Driver所在位置，cluster模式的driver在nodemanager的Application master；client的driver在client上。

### SparkContext初始化

##### TaskScheduler

1. 底层通过操作一个ScheduleBackend，针对不同种类的cluster,调度task
2. 负责处理一些通用的逻辑，比如决定多个job的调度顺序，启动推测任务执行
3. 客户端应用首先调用他的initialize方法和start方法，然后通过runtask方法提交taskset

​	TaskSchedulerImp

​	Pool

​	SparkDeploySchedulerBackend

​		Appclient

​		ClientAction

​		Executor

##### DAGScheduler

实现了面向stage的调度机制的高层次的调度层。他会为每个job计算一个stage的DAG，追踪RDD和stage的输出是否物化（物化是指写入磁盘或者内存），并且寻找一个最优的调度机制运行job,它会将stage作为tasksets提交到底层的TaskSchedulerImp上，在集群上运行他们。

除了处理stage的DAG，他还负责决定运行每个task的最佳位置，基于当前的缓存状态，将这些最佳位置提交给底层的TaskSchedulerImpl。此外，他会处理由于shuffle输出文件丢导致的失败。在这种情况下，旧的stage可能会被重新提交。

##### Spark UI

### Master

##### 主备切换机制

​	备份机器会读取：storedApps, storedDrivers, storedWorkers

​	两种方式读取：FileSystemPersistenceEngine，ZookeeperPersistenceEngine

 	1. 从文件系统或者zookeeper读取保存的app,drivers,workers信息，
 	2. 将这些信息重新注册到Master上
 	3. master发送消息查看这些组件是否正常
 	4. 删除不正常的APP，driver,worker
      	1. 从内存中删除
      	2. 从依赖组件中删除
      	3. 从持久化中删除
 	5. 调用scheduler调度，为等待的任务分配driver/application

##### 注册机制

1. worker注册
2. driver注册
3. application注册

##### 状态改变机制

1. executor状态改变
2. worker状态改变
3. Driver状态改变

##### 资源调度机制（两种资源调度算法）

1. driver调度机制（只有yarn-cluter)
2. application调度机制, 
   1. spreadOutApp，根据需要的cpu和内存，在workers上平均启动executor。
   2. 非spreadOutApp，将每一个app，尽可能少的分配到worker上，

### Worker原理

##### 启动driver

master 发送 launchDriver，worker通过driverRunner线程启动Driver进程，并负责管理该进程。

##### 启动executor

master 发送 launchExecutore，worker通过ExecutorRunner线程启动Executor进程，并负责管理该进程。

### Job触发流程原理

一个action操作会触发一个job。

transaction是RDD之间的转换

action会触发sc.runJob方法，进而调用dagscheduler的方法

### DAGSchedule原理

stage划分算法：倒推方法，遇到宽依赖，就生成一个stage

寻找每个task的最优partition, 根据缓存位置返回task的最佳位置，如果没有缓存则交给taskSchedule分配

BlockManager内存管理

### TaskSchedule原理

task分配算法：优先最优级别

### Executor原理

启动一个线程，将task任务放在线程池中等待运行

### Task原理

### Shuffle 原理

1. 发生时机：reduceByKey、groupByKey、sortByKey、countByKey、Join、cogroup等操作
2. ShuffledRDD, MapPartitionRDD
3. BlockManager

### CacheManager

### CheckPoint

如果rdd没有持久化，却设置了checkpoint,就悲剧了，因为checkpoint方法触发的时机是当前job执行完成，job执行完成后会释放rdd,这是checkpoint就只能重新计算rdd，在保持到hdfs中。

通过建议对要checkpoint的rdd，先执行persist(StorgeLevel.DISK_ONLY)

### 

