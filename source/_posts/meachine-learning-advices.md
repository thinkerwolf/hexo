---
title: meachine-learning-advices
date: 2019-05-28 11:42:03
categories:
- 机器学习
tags:
- machine-learning
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
## 决定下一步做什么

### 调试学习算法

假设你已经实现了正则化的线性回归算法来预测房价。

但是，当你用一组新的数据测试预测函数时，你发现其预测会有很大的误差，下一步该怎么做。

- 获取更多的训练数据。
- 尝试更小的特性集合。
- 尝试获取更多特性。
- 尝试增加多项式特性。
- 尝试减小$\lambda$。
- 尝试增加$\lambda$。

上述方法可能有效，但是不是一个很系统的方法。特别是在实际生活中当使用第1个和第3个解决方法时，可能会相当耗时。所以需要一个系统性的诊断方法来判定。

### 机器学习诊断

一可以了解学习算法内部哪部分合适或者不合适的测试，并且可以获得建议来改进学习算法。

诊断发可能会花费一点时间，但绝对不会浪费时间。

## 评估预测

一个预测函数针对训练用例可能误差很小，但是仍然不正确（过拟合导致）。因此为了评估预测函数，并给定了一组训练用例，我们可以将数据分离成两部分：一组训练集合和一组测试集合。通常**训练集合**占总数据集合的70%，剩下的30%用作**测试集合**。

用这两个集合的新步骤如下：

1. 学习$\Theta$并使用训练集合最小化$J_{train}(\Theta)$。
2. 计算测试集合误差$J_{test}(\Theta)$

### 测试集合误差（代价）

#### 线性回归

$$
J_{test}(\Theta)=\frac{1}{2m_{test}}\sum_{i=1}^{m_{test}}{(h_\Theta(x_{test}^{(i)})-y_{test}^{(i)})^2}
$$

#### 分类误差

$$
err(h_\Theta(x),y)=1/0 \qquad if \; h_\Theta(x)\ge0.5 \,and \,y=0 \quad if \; h_\Theta(x)<0.5\,and\,y=1 
$$

这个给了我们基于一个误分类二进制0或1误差结果。测试集合的测试误差的平均值为：
$$
error=\frac{1}{m_{test}}\sum_{i=1}^{m_{test}}{err(h_\Theta(x_{test}^{(i)}),y_{test}^{(i)})}
$$












