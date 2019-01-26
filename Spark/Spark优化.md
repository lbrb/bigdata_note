# Spark优化

### 性能优化概述

性能瓶颈：CPU、网络宽带、内存

- 使用高性能序列化类库

  - java序列化机制
  - Kryo序列化机制，占用内存少，比较麻烦

- 优化数据结构

  - 目的：减少对内存的消耗
  - 优先使用数组、字符串，而不是集合类。优先使用array，而不是ArrayList、LinkedList、HashMap集合
  - 避免使用多层嵌套的对象结构。可以用json替代
  - 尽量用int替代String

- RDD持久化、checkpoint

  - 对反复操作的RDD，进行transform、checkpoint

- 使用序列化的持久化级别

  ​	先序列化再持久化，可减少对内存的消耗，对io的消耗，但会导致CPU的消耗增大

  - 比如：MEMORY_ONLY_SER

- Java虚拟机垃圾回收调优

  垃圾回收的性能开销，是跟内存中的对象的数量成正比的。

  - 使用更高效的数据结构，比如array或String
  - 使用序列化
  - 分配内存：一般调节Executor内存的比例就可以了
    - Executor 给RDD partition分配更大的内存
      - 优化Executor内存比例（内存分了两部分：RDD partition、RDD partition cache(60%))，可以根据实际情况调节内存分配比例，减少RDD partition的垃圾回收次数。
    - RDD partition 给Eden分配更多的内存
      - Java内存分为两大块：新生代、老年代
      - 新生代分为Eden、survivor1、survivor2
      - gc操作分两个minor gc和full gc
      - Eden和Survivo1满了触发minor gc ; 老年代满了触发full gc; full gc较慢，而且会导致其他进程停止

- 提高并行度

  spark官网设置集群总数量的2倍到3倍的并行度。因为每个任务完成的时间不一样。这样可以更高效的使用硬件资源。

- 广播共享数据

  每个节点有一份数据，而不是每个Task一份数据，浪费内存和宽带。

- 数据本地化

  - Process_local 数据和代码在同一个JVM进程中
  - Node_Local 数据和代码在同一个节点上，但不在一个进程中或者数据在HDFS
  - No_Pree 数据从哪里来，性能都一样
  - Rack_Local 数据和代码在同一个机架上
  - ANY 数据可能在任何地方

- reduceByKey和groupByKey性能对比

  以wordcount举例

  优先使用reduceByKey,会首先在map端做一部分相加的处理，然后在到reduce端做处理

  groupByKey, 不会进行本地聚合。

- shuffle性能优化

  - consolidation机制：没有开启的话，创建写入的文件太多，影响性能。默认是false，先开启。开启后：shuffle写磁盘的数量，大大减少。
  - reduce端拉取时有缓存大小限制，可以提高缓存大小，减少拉取次数。
  - map端bucket缓存大小，也可以适当提高，减少刷新到硬盘的次数。
  - 拉取失败的最大重试次数。reduce拉取数据的时候，可能会碰到map task正在jvm，此时工作进程停止，拉取失败。
  - 拉取失败的重试间隔