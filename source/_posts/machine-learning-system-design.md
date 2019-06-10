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

- 手机许多数据，比如‘“honeypot”工程。
- 基于邮箱线路构建复杂特性。
- 为消息正文构建复杂特性。比如“discount”和“discounts”是否应该视为同一个词。
- 构建复杂算法检测混淆拼写。（med1cine，w4ches等）



## 误差分析