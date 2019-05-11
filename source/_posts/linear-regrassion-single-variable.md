---
title: 单变量线性回归
date: 2019-05-06 10:06:16
categories:
- 机器学习
tags: 
- machine-learning
- linear-regrassion
---
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>


## 模型表示
监督学习中的回归问题。

### 符号声明
- m = 训练用例数量
- x's = 输入变量/特性。
- y's = 输出变量/目标变量。

## 代价函数
- Hypothesis:
$$h_\theta(x)=\theta_0+\theta_1x$$

- Parameters:
$$\theta_0,\theta_1$$

- Cost function:
$$J(\theta_0,\theta_1)=\frac{1}{2m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})^2}$$

- Goal: $$minimize J(\theta_0,\theta_1)$$ 

## 梯度下降
### 梯度下降算法
重复直到收敛 {
	$$\theta_j:=\theta_j-a\frac{\delta}{\delta\theta_j}J(\theta_0,\theta_1)$$
}

与预测函数和代价函数合并计算出
重复直到收敛 {
	$$\theta_0:=\theta_0-a\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})}$$
	$$\theta_1:=\theta_1-a\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x^{(i)}}$$
}




