---
date: 2020-07-29 21:07:00
status: public
categories:
 - deep learning
tags: 
 - Batch Normalization
 - Layer Normalization
title: Batch Normalization与Layer Normalization的理解整理
---

### 问题背景　　
　接着引入covariate shift的概念：如果ML系统实例集合<X,Y>中的输入值X的分布老是变，这不符合IID假设，网络模型很难稳定的学规律，这不得引入迁移学习才能搞定吗，我们的ML系统还得去学习怎么迎合这种分布变化啊。对于深度学习这种包含很多隐层的网络结构，在训练过程中，因为各层参数不停在变化，所以每个隐层都会面临covariate shift的问题，也就是在训练过程中，隐层的输入分布老是变来变去，这就是所谓的“Internal Covariate Shift”，Internal指的是深层网络的隐层，是发生在网络内部的事情，而不是covariate shift问题只发生在输入层。　　
　我们知道网络一旦train起来，那么参数就要发生更新，除了输入层的数据外(因为输入层数据，我们已经人为的为每个样本归一化)，后面网络每一层的输入数据分布是一直在发生变化的，因为在训练的时候，前面层训练参数的更新将导致后面层输入数据分布的变化。以网络第二层为例：网络的第二层输入，是由第一层的参数和input计算得到的，而第一层的参数在整个训练过程中一直在变化，因此必然会引起后面每一层输入数据分布的改变。我们把网络中间层在训练过程中，数据分布的改变称之为：“Internal  Covariate Shift”。Paper所提出的算法，就是要解决在训练过程中，中间层数据分布发生改变的情况，于是就有了Batch  Normalization，这个牛逼算法的诞生。


### Normalization 的通用框架

$$
\hat{x}=\frac{x-\mu}{\sigma^2} 
$$
$$
y = g\hat{x}+b
$$

$\mu$和$\sigma^2$分别是均值和方差，它们是根据特征值计算出来的，g和b是需要训练过程中去学习的参数。


### 总结

**Layer Normalization与Batch Normalization对比：**　　
　BN针对一个minibatch的输入样本，计算均值和方差，基于计算的均值和方差来对某一层神经网络的输入X中每一个case进行归一化操作。但BN有两个明显不足：1、高度依赖于mini-batch的大小，实际使用中会对mini-Batch大小进行约束，不适合类似在线学习（mini-batch为1）情况；2、不适用于RNN网络中normalize操作：BN实际使用时需要计算并且保存某一层神经网络mini-batch的均值和方差等统计信息，对于对一个固定深度的前向神经网络（DNN，CNN）使用BN，很方便；但对于RNN来说，sequence的长度是不一致的，换句话说RNN的深度不是固定的，不同的time-step需要保存不同的statics特征，可能存在一个特殊sequence比其的sequence长很多，这样training时，计算很麻烦。但LN可以有效解决上面这两个问题。

  LN适用于LSTM的加速，但用于CNN加速时并没有取得比BN更好的效果。



**BN的特点**

但是，BN 的转换是针对单个神经元可训练的——不同神经元的输入经过再平移和再缩放后分布在不同的区间，而 LN 对于一整层的神经元训练得到同一个转换——所有的输入都在同一个区间范围内。如果不同输入特征不属于相似的类别（比如颜色和大小），那么 LN 的处理可能会降低模型的表达能力。



**LN的缺点**

BN 的转换是针对单个神经元可训练的——不同神经元的输入经过再平移和再缩放后分布在不同的区间，而 LN 对于一整层的神经元训练得到同一个转换——所有的输入都在同一个区间范围内。如果不同输入特征不属于相似的类别（比如颜色和大小），那么 LN 的处理可能会降低模型的表达能力。



---

参考：
[Batch Normalization（BN，批量归一化）](https://zhuanlan.zhihu.com/p/55852062)
[【深度学习】深入理解Batch Normalization批标准化](https://www.cnblogs.com/guoyaohua/p/8724433.html)
[深度学习（二十九）Batch Normalization 学习笔记](https://blog.csdn.net/hjimce/article/details/50866313)
[深度学习加速策略BN、WN和LN的联系与区别，各自的优缺点和适用的场景？？](https://www.zhihu.com/question/59728870)

