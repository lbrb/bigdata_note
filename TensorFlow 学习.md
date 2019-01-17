# TensorFlow 学习

##### 有时间的时候看下源码

是一个采用数据流图，用于数值计算的开源软件库。

- Tensor(张量)：N维数组
- Flow(流)：基于数据流图的计算

TensorFlow的边有两种链接关系：

数据依赖：

​	前向传播：张量在数据流图中从前往后流动一遍就完成一个前向传播。

​	后向传播：残差在数据流图中从后往前流动一遍就完成一个后向传播。

控制依赖：



TensorFlow可以认为是一种编程工具，使用TensorFlow来实现具体的业务需求，然后我们使用TensorFlow这个工具箱中的各种工具来实现各种功能。TensorFlow实现基本的数值计算、机器学习、深度学习等，使用TensorFlow必须理解下列概念：

- 使用图来表示计算任务
- 在会话的上下文中执行图
- 使用tensor表示数据
- 通过变量来维护状态
- 使用feed/fetch可以为任意的操作赋值或者从其中获取数据



##### 程序结构

- 构建阶段

  使用TensorFlow提供的API构建这个图

- 执行阶段

  将构建好的执行图在给定的会话中执行，并得到执行结果

##### TensorFlow图

TensorFlow编程的重点是根据业务需求，使用TensorFlow的API将业务转换为执行图（有向无环图）

可以通过Vairable和assign完成变量的定义和更新。

##### 控制依赖

tf.control_dependencies

通过TensorFlow提供的一组函数来处理不完全依赖的情况下的操作排序问题。

##### TensorFlow设备

为了在执行操作的时候，充分利用计算资源，可以明确指定操作在哪个设备上执行。

一般情况下，不需要指定使用CPU还是GPU，TensorFlow会自动检测。模型情况下，只会使用第一个GPU，所以，在实际编程中，经常需要明确给定使用的CPU和GPU。



### TensorFlow可视化

tensorboard

##### tensorflow

- FIFOQueue: 先进先出队列
- RandomShuffleQueue:随机混淆队列

