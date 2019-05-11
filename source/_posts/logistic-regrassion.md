---
title: Logistic回归
date: 2019-05-06 17:11:16
categories:
- 机器学习
tags: 
- machine-learning
- logistic-regression
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

## 分类 Classfication
分类问题可以使用线性回归方式去拟合数据，但是因为数据只有几个分类会导致拟合的效果不好，所以采用Logistics算法拟合分类数据。

Classfication: $$y = 0 \quad or \quad y = 1$$
Logistic Regrassion: $$0<=h_\theta(x)<=1$$

## 预测函数
使$0\leq h_\theta(x)\leq 1$
$$h_\theta(x)=g(\theta^Tx)=\frac{1}{1+e^{\theta^Tx}}$$

y=1的可能性，给定x，用$\theta$参数化

$$h_\theta(x)=P(y=1|x;\theta)$$

$$P(y=1)+P(y=0)=1$$

## 决策边界

## 代价函数
$$Cost(h_\theta(x),y)=-\log({h_\theta(x)}) \quad y=1$$
$$Cost(h_\theta(x),y)=-\log({1-h_\theta(x)}) \quad y=0$$

## 简化代价函数和梯度下降

### Logistic回归代价函数
$$J(\theta)=\frac{1}{m}\sum_{i=1}^{m}{Cost(h_\theta(x),y)}$$
$$Cost(h_\theta(x),y)=-\log(h_\theta(x))\quad y=1$$
$$Cost(h_\theta(x),y)=-\log(1-h_\theta(x))\quad y=0$$

### 简化后的代价函数
$$J(\theta)=\frac{1}{m}\sum_{i=1}^{m}{Cost(h_\theta(x),y)}=-\frac{1}{m}\sum_{i=1}^{m}{y^{(i)}\log(h_\theta(x^{(i)})+(1-y^{(i)})\log(1-h_\theta(x^{(i)})} $$

### 最小$J(\theta)$计算
与线性回归相似...
repeat : {
	$$\theta_j:=\theta_j-a\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{_j^{(i)}}}$$
}

## 高级优化
repeat: {
	$$\theta_j:=\theta_j-a\frac{\delta}{\delta\theta_j}J(\theta)$$
}
给定$\theta$，计算$J(\theta)$和$\frac{\delta}{\delta\theta_j}J(\theta)$
### 优化算法
- 梯度下降
- 共轭梯度
- BFGS
- L-BFGS

### 对比
- 无需手动计算学习速率。
- 比梯度下降更快。
- 但是相对更复杂。

## 多元分类-一对多
![Difference](/images/ll1.png)

每种分类采用独立的$h_\theta(x)$，如下图所示。
![Difference](/images/ll2.png)

为每个类别$i$采用独立的预测函数$h_\theta^{(i)}(x)$ ，用来预测当$y=i$时的概率。
当输入新的x用来预测时，找到$h_\theta^{(i)}(x)$最大的类别作为预测结果。即
$$max h{_\theta^{(i)}}(x)$$

