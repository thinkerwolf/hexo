---
title: 神经网络参数和反向传播算法
date: 2019-05-20 10:37:05
categories:
- 机器学习
tags:
- machine-learning
- neural-networks
---
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    extensions: ["tex2jax.js"],
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      <!--$表示行内元素，$$表示块状元素 -->
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true
    },
    "HTML-CSS": { availableFonts: ["TeX"] }
  });
</script>
<!--加载MathJax的最新文件， async表示异步加载进来 -->
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js">
</script>

## 代价函数
首先定义一下变量
- L=网络上所有层数
- $sl$=层级l中的单元数量(不包括偏置单元)
- K=输出单元/类型的数量
神经网络的消耗函数$J(\theta)$将是Logistics回归的概括。正则化的Logistic回归消耗函数为：
$$
J(\theta)=-\frac{1}{m}\sum_{i=1}^{m}{y^{(i)}\log(h_\theta(x^{(i)})+(1-y^{(i)})\log(1-h_\theta(x^{(i)}))}
$$
神经网络的消耗函数$J(\theta)$更为复杂：
$\newcommand{\subk}[1]{ #1_k }$
$$
h_\theta\left(x\right)\in \mathbb{R}^{K}
$$

$$
{\left({h_\theta}\left(x\right)\right)}_{i}={i}^{th} \text{output}
$$


$$
J(\Theta) = -\frac{1}{m} \left[ \sum\limits_{i=1}^{m} \sum\limits_{k=1}^{k} {y_k}^{(i)} \log \subk{(h_\Theta(x^{(i)}))} + \left( 1 - y_k^{(i)} \right) \log \left( 1- \subk{\left( h_\Theta \left( x^{(i)} \right) \right)} \right) \right] + \frac{\lambda}{2m} \sum\limits_{l=1}^{L-1} \sum\limits_{i=1}^{s_l} \sum\limits_{j=1}^{s_{l+1}} \left( \Theta_{ji}^{(l)} \right)^2
$$


添加一些“和嵌套$\sum$”计算多重输出节点。在等式的第一个部分，方括号之前，有额外的“和嵌套$\sum$”循环计算所有的输出节点。

在正则化部分，必须计算多重theta矩阵。当前theta矩阵的列数与当前层节点数量相同（包括偏置节点）。当前theta矩阵行数量与下一层节点数量相同（偏置节点除外）。

笔记：

- 双重求和只是对输出层每个logistic回归的消耗花费进行求和。
- 三重求和只是将整个网络中所有单个$\Theta$s的方阵进行相加。
- 在三种求和中的**i**并不是训练样本中的**i**。



## 反向传播算法
“反向传播”是神经网络中最下化花费函数。我们的目标是计算：
$$
min_\theta J(\Theta)
$$
就是，使用一组最优的theta参数最小化花费函数。下面看看计算偏导数的公式：
$$
\frac{\partial}{\partial\Theta_{i,j}^{(l)}}J(\Theta)
$$
### 反向传播例子
以下面这个神经网络作为例子

![pic1](/images/pic1.png)

$\delta_j^{(j)}$=层级$l$中节点$j$的“误差”。

每个输出单位（层级L=4）
$$
\delta_j^{(4)}=a_j^{(4)}-y_i \to \delta^{(4)}=a^{(4)}-y
$$

$$
\delta^{(3)}={(\Theta^{(3)})}^T\delta^{(4)} {.\ast} g^{\prime}(z^{(3)}) \to g^{\prime}(z^{(3)})=a^{(3)} {.\ast} (1-a^{(3)}) 
$$

$$
\delta^{(2)}=(\Theta^{(2)})^T\delta^{(3)}{.\ast}g^{\prime}(z^{(2)}) \to g^{\prime}(z^{(2)})=a^{(2)}{.\ast}(1-a^{(2)})
$$

根据公式推导可以得出(忽略正则项)
$$
\frac{\partial}{\partial\Theta_{i,j}^{(l)}}J(\Theta)=a_j^{(l)}\delta_i^{(l+1)}
$$

### 反向传播算法步骤

- 训练集合${(x^{(1)},y^{(1)}),....,(x^{(m)},y^{(m)})}$

- 设置$\Delta_{ij}^{(l)}=0$（针对所有的l,i,j）（用来计算 $\frac{\partial}{\partial\Theta_{i,j}^{(l)}}J(\Theta)$）

- For i=1 to m

&emsp;1. Set $a^{(1)}=x^{(i)}$

&emsp;2. 使用前向传播来计算$a^{(l)}$ l = 2,3,....,L

&emsp;3. 使用$,y^{(i)}$，计算$\delta^{(L)}=a^{(L)}-y^{(i)}$

&emsp;4. 计算$\delta^{(L-1)},\delta^{(L-2)},....,\delta^{(2)}$，使用
$$
\delta^{(l)}=({(\Theta^{(l)})}^T\delta^{(l+1)}) {.\ast} a^{(l)}  {.\ast}(1-a^{(l)})
$$
&emsp;5. $\Delta_{ij}^{(l)} := \Delta_{ij}^{(l)}+a_j^{(l)}\delta_i^{(l+1)}$或者向量化
$$
\Delta^{(l)}:=\Delta^{(l)}+\delta^{(l+1)}({a^{(l)}}^T)
$$


- $D_{ij}^{(l)} := \frac{1}{m}\Delta_{ij}^{(l)}+\lambda\Theta_{ij}^{(l)}$ &emsp; if$ j\ne0$&emsp;&emsp;$D_{ij}^{(l)}:=\frac{1}{m}\Delta_{ij}^{(l)}$ &emsp;if$ j=0$

## 梯度检测

梯度检测假设反向传播已经运算好。我们可以大约将偏导数近似为：
$$
\frac{\partial}{\partial\Theta_{i,j}^{(l)}}J(\Theta)=\frac{J(\Theta+\epsilon)-J(\Theta-\epsilon)}{2\epsilon}
$$




























