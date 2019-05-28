---
title: 机器学习-多变量线性回归
date: 2019-05-06 10:07:16
categories:
- 机器学习
tags:
- machine-learning
- linear-regrassion
---
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

## 多特性
**n** 表示特性的数量，**m**表示测试用例

- Hypothesis：
$$h_\theta(x)=\theta^Tx=\theta_0+\theta_1x_1+\theta_2x_2+...+\theta_jx_j$$

## 多特性梯度下降
- Hypothesis：
$$h_\theta(x)=\theta^Tx=\theta_0+\theta_1x_1+\theta_2x_2+...+\theta_jx_j$$

- Parameters:
$$\theta$$

- Cost function:
$$J(\theta)=\frac{1}{2m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})^2}$$

- Gradient descent:
$$所有的x{_0^{(i)}}=1$$
repeat: {
	$$\theta_j:=\theta_j-a\frac{\delta}{\delta\theta_j}J(\theta)$$
}
repeat: {
	$$\theta_j:=\theta_1-a\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{^{(i)}_j}}$$
}

## 多元梯度下降-特征缩放
将每个特性合适地缩放到-1<=x<=1的范围内。

## 多元梯度下降-学习速率
$$\theta_j:=\theta_j-a\frac{\delta}{\delta\theta_j}J(\theta)$$
- 确保梯度下降正确运行。
- 选择合适的学习速率。

### 学习速率
- 如果a太小，收敛的会很慢。
- 如果a太大，每次遍历代价函数J都不是再下降，可能找不到收敛点。

选择学习速率可以尝试 ...,0.001,0.003,0.01,0.03,0.1,0.3等等..

## 多特性与多项式回归
预测函数可以使用多项式表示，比如：

$$h_\theta(x)=\theta_0+\theta_1x_1+\theta_2x{_1^2}$$

$$h_\theta(x)=\theta_0+\theta_1x_1+\theta_2\sqrt{x_1}$$

## 正规方程
$$\theta=(X^TX)^-1X^Ty$$
```
Octave: pinv(X'*X)*X'*y
```



