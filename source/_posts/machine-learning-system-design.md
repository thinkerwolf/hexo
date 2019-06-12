---
title: 机器学习系统设计
date: 2019-06-10 08:39:50
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
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js">
</script>

## 确定执行的优先级：垃圾分类案例

### 构建垃圾分类器

![](/images/machine/ml5.png)

监督学习，$x$=邮件特性，$y$=垃圾（1）或非垃圾（0）。

特性$x$：选择100个象征着垃圾或非垃圾邮件的单词  
$$
x=\begin{bmatrix}
		0 \\\\
		1 \\\\
		1 \\\\
        \vdots \\\\
        1 \\\\
        0 \\\\
		\vdots
	\end{bmatrix}
	\begin{bmatrix}
		andrew \\\\
		buy \\\\
		deal \\\\
        \vdots \\\\
        now \\\\
         discount \\\\
		\vdots
	\end{bmatrix}
$$

$$
x_j = 1 ,如果单词j出现了,否则
x_j = 0
$$

### 如何降低分类器误差

- 收集许多数据，比如‘“honeypot”工程。
- 基于邮箱线路构建复杂特性。
- 为消息正文构建复杂特性。比如“discount”和“discounts”是否应该视为同一个词。
- 构建复杂算法检测混淆拼写。（med1cine，w4ches等）



## 误差分析

解决机器学习问题的推荐方法如下：

- 首先从简单算法开始，快速实现后尽早地应用到交叉验证上。
- 绘制出学习曲线看看更多的数据，更多的特性是否有帮助。
- 手动测试交叉验证例子上的错误，尝试定位大多数的误差再哪里产生。

例如，假设有500邮件，我们将100封错误地分类。我们可以手动分析这100封邮件，基于它们的类型手动进行分类。我们随后就能想到新的线索和特性帮助我们正确的将这100封邮件分类。因此，如果大多数错误分类的是窃取密码的邮件，我们就可以发现这类邮件的特性并加入到模型中。我们也可以看看每个单词的变化对误差的影响。

获取单个数字化的误差结果非常重要。否则很难知晓算法的性能。例如使用次干提取，得到了一个3%的误差而不是5%，然后我们应该将其加入模型。但是如果区分大小写得到3.2%的误差代替3%的误差，我们应该避免使用新特性。因此我们应该尝试新事物，为误差率计算一个数字化的值，根据结果决定是否将新特性加入模型中。

## 不对称分类的误差和评估

### 精确度和召回率

|                 | 真实（y=1）    | 真实（y=0）    |
| --------------- | -------------- | -------------- |
| **预测（y=1）** | True positive  | False positive |
| **预测（y=0）** | False negative | True negative  |

- 精确度

$$
\frac{TruePositive}{PredictedPositive} = \frac{TruePositive}{TruePositive + FalsePositive}
$$

- 召回率

$$
\frac{TruePositive}{ActualPositive} = \frac{TruePositive}{TruePositive + FalseNegative}
$$

### 权衡精确度和召回率

如何比较精确度和召回率

|       | 精确度（P） | 召回率（R） |
| ----- | ----------- | ----------- |
| 算法1 | 0.5         | 0.4         |
| 算法2 | 0.7         | 0.1         |
| 算法3 | 0.02        | 1.0         |

$$
2\frac{PR}{P+R}
$$

