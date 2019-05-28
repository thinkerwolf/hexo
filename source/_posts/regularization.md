---
title: 机器学习-正则化
date: 2019-05-07 14:11:16
categories:
- 机器学习
tags: 
- machine-learning
- regularization
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

## 过拟合问题
如果有太多特性，预测函数可能与训练集拟合地很好($J(\theta)\approx 0$)，但是无法泛化（预测函数应用到新样本的能力）到新的样本中。

### 解决过度拟合
当有过多的特性，而训练数据数量多少，就会很容易出现拟合过度的问题。
- 减少特性数量：手动选择保留哪些特性；**模型选择**算法。
- 正则化：保留所有特性，减少$\theta_j$的量级或值；当有很多特性，每个特性对于预测都会有一些影响，运行的效果会很好。

## 代价函数

### 正则化的代价函数表示
$$J(\theta)=\frac{1}{2m}(\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})^2}+\lambda\sum_{j=1}^{n}{\theta_j^2})$$
- $\lambda$代表正则参数

### $\lambda$的选择
如果$\lambda$的值过大，可能远远超过问题本身（比如$\lambda=10^10$）
- 算法依然正常运行，过大的$\lambda$不会影响程序运行。
- 算法无法消除过度拟合问题。
- 算法导致前耦合问题。（甚至无法拟合原本的数据）
- 梯度下降法无法收敛。

## 线性回归正则化
### 梯度下降
Repeat {
	$$\theta_0:=\theta_0-a\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{_0^{(i)}}}$$
	$$\theta_j:=\theta_j-a(\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{_j^{(i)}}}+\frac{\lambda}{m}\theta_j)$$
	$(j=1,2,3,4....,n)$
}
### 正规方程
$$\theta=(X^TX+\lambda\frac{\delta}{\delta\theta_j}J(\theta))X^Ty$$
$$\frac{\delta}{\delta\theta_j}J(\theta)=
	\begin{bmatrix}
		0 & 0 & 0 & 0 & 0 \\\\
		0 & 1 & 0 & 0 & 0 \\\\
		0 & 0 & 1 & 0 & 0 \\\\
		0 & 0 & 0 & \cdots & 0
	\end{bmatrix}
$$
## Logistic回归正则化
### 代价函数
$$J(\theta)=-\frac{1}{m}(\sum_{i=1}^{m}{y^{(i)}\log(h_\theta(x^{(i)})+(1-y^{(i)})\log(1-h_\theta(x^{(i)})})
+\frac{\lambda}{2m}\sum_{j=1}^{n}{\theta_j^2}
$$
### $\theta$计算
Repeat {
	$$\theta_0:=\theta_0-a\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{_0^{(i)}}}$$
	$$\theta_j:=\theta_j-a(\frac{1}{m}\sum_{i=1}^{m}{(h_\theta(x^{(i)})-y^{(i)})x{_j^{(i)}}}+\frac{\lambda}{m}\theta_j)$$
	$(j=1,2,3,4....,n)$
}
## 编程作业 Logistic回归
[Github地址](http://www.googhttps://github.com/thinkerwolf/meachine-learning/tree/master/machine-learning-ex2)
