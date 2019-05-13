# Word2Vec

词是表义的基本单元，词向量是自然语言处理的基础。

##### 为什么不用one-hot

无法计算相似度（比如：余弦相似度）

我们希望找到一种向量表示方法，能够计算出词之间的相似度

### word2vec

是一种工具，包括两个模型：跳字模型（skip-gram和连续词袋模型(continuous bag of words CBOW)，以及两种高效训练的方法：负采样（negative sampling）和层序（hierachical softmax)，word2vec词向量可以较好的表达不同词之间的相似和类比关系。

有个例子：男人-女人=国王-王后

### 跳字模型

给定文本序列： the man hit his son . 跳字模型所关心的是，给定hit,生成它邻近词the, man, his, son的概率。在这个例子中，hit是中心词，the，man， his，son叫背景词，由于hit只生成与它距离不超过2的背景词，该时间窗口的大小为2.

### 连续词袋法



### 负采样



### 文本生成（Word2Vec + RNN/LSTM）

##### 下一个字母

学习一个小说，自动写小说。47:00

##### 下一个单词



##### 下一个句子（问答机器人)



##### 下一个图片、音符





### Word2Vec  + CNN

