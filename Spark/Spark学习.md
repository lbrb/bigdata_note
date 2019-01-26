# Spark学习

可以先学《Spark从入门到精通（Scala编程、案例实战、高级特性、Spark内核源码剖析、Hadoop高端）》

### 组成部分

- Spark Core

  包含Spark的基本功能，如内存计算、任务调度、部署模式，故障恢复、存储管理等，主要面向批数据处理

- Spark Sql

  允许开发人员直接处理RDD， 同时可以查询Hive,Hbase等外部数据源

- Spark Streaming

  支持高吞吐量，可容错处理的实时流数据处理

- MLLib

  提供常用机器学习算法的实现

- GraphX

  是Spark中用于图计算的API

### 基本原理

- 分布式

- 基于内存

- 迭代式计算

##### Spark 与MapReduce最大的不同

在于，迭代式计算，MapReduce分为两个阶段，map和reduce，两个阶段处理完后，该job就结束了。Spark可分为m个阶段，因为是内存迭代式的，可以处理多个阶段。所以Spark相对于mapReduce来说，可以通过更强大的功能。

### RDD及其特点

是Spark提供的核心抽象，弹性分布式数据集，数据的集合。

一个RDD,在逻辑上抽象地代表了一个HDFS文件，但是，他实际上是被分区的，分为多个分区，多个分区数据散落在spark集群中。

提供了容错性，可以自动从节点失败中恢复过来。

### Spark开发

- 核心开发：离线批处理、延迟性的交互式数据处理

  - 定义初始的RDD，要定义一个RDD是从哪里读取数据的，hdfs,程序中的集合，hive

    - 程序中的集合
    - 本地文件
    - HDFS文件

  - 定义对RDD的计算操作，这个在Spark里称为算子， map, reduce, flatmap,groupbykey。

    - flatmap 

      将RDD的一个元素，拆分为一个或多个元素。

    - mapToPair

      其实就是将每个元素，映射为一个（v1,v2)元素

    - reduceByKey

      对每个key对应的value,都进行reduce操作

  - 循环往复的过程，第一次计算完成后，数据会到新的RDD，存放到新的节点上。针对新的RDD定义计算操作。

  - 最后，获得最终的数据，将数据保存起来。

- SQL查询

- 实时计算

 ### Spark 架构原理

- Driver
- Master
- Worker
- Executor
- Task

### RDD操作

- transformation

  针对已有的RDD创建一个新的RDD，map, flatmap, join等等

- action

  针对RDD进行最后的操作，比如遍历、reduce、保存到文件，并可以返回结果给Driver程序

### RDD持久化

默认情况下，对于这种针对大量数据的action操作都是非常耗时的（例如：count），而且action操作时，才会从hdfs读取数据，action操作完成后，内存中的数据会清除，那么下次action操作时还要从fdfs中读取数据，处理数据。所以要进行RDD持久化。

总的来说两个坏处：

- 重复耗时计算
- 重复读取hdfs

对linesRDD进行持久化后，虽然第一次count操作执行完了，但也不会清除lineRDD, 反而会缓存在内存或磁盘中。再次执行count操作，不会从hdfs读取数据，而是从缓存中读取数据。

好处：

- 一个RDD执行多次操作，只有再其第一次计算时，才会从hdfs中读取，其后的操作都是针对缓存的操作
- 官方文档说：合理使用RDD持久化机制，甚至可以提升spark应用程序的性能10倍

cache persist

unpersist

##### 持久化策略

- MEMORY_ONLY（默认）

  没有序列化的java对象的方式持久化到内存中

- MEMORY_AND_DISK

  某些partition不能存储在内存中，会持久化到硬盘中

- MEMORY_ONLY_SER

  同MEMORY_ONLY，但是会将java对象序列化，序列化后在持久化可以减少内存，但使用时需要反序列化，消耗CPU

- MEMORY_AND_DSK_SER 

  同MEMORY_AND_DSK,但使用序列化

- DISK_ONLY

  使用非序列化存储到硬盘中



### 共享变量

##### 不使用共享变量的情况

a = 3;

rdd.map(v=>v*3)

不使用共享变量的情况下,外部变量a会拷贝到每一个task中，如果变量比较大的话，会浪费大量的网络传输和硬件资源。

##### Broadcast Variable(广播变量)

只读的

会将使用到的变量，仅仅为每个节点拷贝一份，更大的用处是优化性能，减少网络传输以及内存消耗

##### Accumulator（累加变量）

可以让多个task共同操作一份变量，主要可以进行累加操作;

task不能读，只有driver可以读；

### 排序机制

##### 排序

可能会用到sortByKey, mapToPair, reduceByKey

##### 二次排序

自定义排序对象，实现Order、Serializable接口

### 实践经验

##### 找不到类

打jar包的时候，不能将Windows中的其他包也打到jar包中，jar包只包含自己的代码。