---
date: 2020-10-16 21:07:00
status: public
markup: mmar
categories:
 - deep learning
tags: 
 - CRF
title: DeepLearning中CRF计算原理
---



主要内容来处：[https://createmomo.github.io](https://createmomo.github.io/)：

- [CRF Layer on the Top of BiLSTM - 1](https://createmomo.github.io/2017/09/12/CRF_Layer_on_the_Top_of_BiLSTM_1/) Outline and Introduction
- [CRF Layer on the Top of BiLSTM - 2](https://createmomo.github.io/2017/09/23/CRF_Layer_on_the_Top_of_BiLSTM_2/) CRF Layer (Emission and Transition Score)
- [CRF Layer on the Top of BiLSTM - 3](https://createmomo.github.io/2017/10/08/CRF-Layer-on-the-Top-of-BiLSTM-3/) CRF Loss Function
- [CRF Layer on the Top of BiLSTM - 4](https://createmomo.github.io/2017/10/17/CRF-Layer-on-the-Top-of-BiLSTM-4/) Real Path Score
- [CRF Layer on the Top of BiLSTM - 5](https://createmomo.github.io/2017/11/11/CRF-Layer-on-the-Top-of-BiLSTM-5/) The Total Score of All the Paths
- [CRF Layer on the Top of BiLSTM - 6](https://createmomo.github.io/2017/11/24/CRF-Layer-on-the-Top-of-BiLSTM-6/) Infer the Labels for a New Sentence
- [CRF Layer on the Top of BiLSTM - 7](https://createmomo.github.io/2017/12/06/CRF-Layer-on-the-Top-of-BiLSTM-7/) Chainer Implementation Warm Up
- [CRF Layer on the Top of BiLSTM - 8](https://createmomo.github.io/2017/12/07/CRF-Layer-on-the-Top-of-BiLSTM-8/) Demo Code



通常在序列标注模型的最后一层layer会添加CRF计算，因为序列标注任务中lable之前有较强的约束性，例如，B-Person与I-Person之前有强关联，B-Person和I-Locations之间有强“非关联”，而CRF模型中的转移矩阵则可以很好体现这些特性。

PS: 如果实际经验中使用LSTM+CRF模型后发现，发现学习到的转移矩阵很弱，这可能是由于学习率和初始化发射矩阵的问题；



### CRF计算原理

CRF的计算分为三个部分，第一部分先介绍输入参数，第二部分说明在训练阶段如何计算-损失函数，第三部分是预测时的计算方式。

#### 输入参数

CRF的输入参数有两个，一是**发射分数**，它可以是LSTM的输出结果，即每个word对应每个tag的得分。二是**转移矩阵**，它的内容是一个tag转移到下一个tag的权重。
假设我们有2个tag，和3个word，对应的发射分数和转移矩阵如下：
发射分数:
| word/tag | t0 | t1 |
| --- | --- | --- |
| w0 | 0.3 | 0.7 |
| w1 | 0.6 | 0.4 |
| w2 | 0.2 | 0.8 |
例如$Emission_{w0, t0}$，表示为w0映射到t0的概率是0.3。

转移矩阵：
| tag-index | 0 | 1 |
| --- | --- | --- |
| 0 | 0.6 | 0.4 |
| 1 | 0.1 | 0.9 |
例如$Transition_{t0, t0}$，t0转移到t0的概率是0.6。

#### 损失函数

$p_i$ 是每i条路径，则所有路径的得分是：
$$
P_{total} = P_1 + P_2 + … + P_N = e^{S_1} + e^{S_2} + … + e^{S_N}
$$
那么crf的损失函数如下：
$$
Loss Function = \frac{P_{RealPath}}{P_1 + P_2 + … + P_N} 
$$
那么：
$$
LogLossFunction = \log \frac{P_{RealPath}}{P_1 + P_2 + … + P_N}
$$
因此要最小化：
$$
\begin{aligned}
LogLossFunction &= -\log \frac{P_{RealPath}}{P_1 + P_2 + … + P_N} \\
&= - \log \frac{P_{RealPath}}{P_1 + P_2 + … + P_N} \\
&= - (\log(e^{S_{RealPath}}) - \log(e^{S_1} + e^{S_2} + … + e^{S_N})) \\
&= - (S_{RealPath} - \log(e^{S_1} + e^{S_2} + … + e^{S_N})) \\
\end{aligned}
$$


即真实路径在所有路径中的概率最大。因此我们只有计算出$P_{RealPath}$和$P_{total}$这两部分，就可以进行训练了。

##### 单路径分数
$S_i$由两部分组成：
$$
S_i = EmissionScore + TransitionScore
$$

举例，计算 t0，t1，t0 这条路径的值：
$$
EmissionScore=Emission_{w0,t0}+Emission_{w1,t1}+Emission_{w2,t0}
$$
$$
TransitionScore=Transition_{to->t1} + Transition_{t1->t0}
$$

##### 所有路径总分数

计算所有路径总分数需要先定义2个变量：
**obs**：obj表示当前的信息
**previous**：previous表示上一步各个tag经过这个tag的所有路径的得分总各。
另外用**transition**表示转移矩阵
$w_0$->$w_1$
当前变量值 ：
$$
\begin{aligned}
obs = [x_{11}, x_{12}] \\
previous = [x_{01}, x_{02}]
\end{aligned}
$$

$$
scores = previous + obj + transition
$$

$$
scores =
\begin{pmatrix}
x_{01}&x_{01}\\
x_{02}&x_{02}
\end{pmatrix} +
\begin{pmatrix}
x_{11}&x_{12}\\
x_{11}&x_{12}
\end{pmatrix} +
\begin{pmatrix}
t_{11}&t_{12}\\
t_{21}&t_{22}
\end{pmatrix}
$$


$$
previous=[\log (e^{x_{01}+x_{11}+t_{11}} + e^{x_{02}+x_{11}+t_{21}}), \log (e^{x_{01}+x_{12}+t_{12}} + e^{x_{02}+x_{12}+t_{22}})]
$$





#### 解码


$$
scores =
\begin{pmatrix}

x_{01}+x_{11}+t_{11}&x_{01}+x_{12}+t_{12}\\
x_{02}+x_{11}+t_{21}&x_{02}+x_{12}+t_{22}

\end{pmatrix}
$$

$$
previous=[\max (scores[00], scores[10]),\max (scores[01],scores[11])]
$$

$$
scores =
\begin{pmatrix}

x_{01}+x_{11}+t_{11}&x_{01}+x_{12}+t_{12}\\
x_{02}+x_{11}+t_{21}&x_{02}+x_{12}+t_{22}

\end{pmatrix}=
\begin{pmatrix}
0.2&0.3\\
0.5&0.4
\end{pmatrix}
$$

$$
previous=[\max (scores[00], scores[10]),\max (scores[01],scores[11])] = [0.5, 0.4]
$$





