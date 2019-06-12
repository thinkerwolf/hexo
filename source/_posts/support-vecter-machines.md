---
title: 支持向量机（SVM）
date: 2019-06-12 17:46:55
categories: 
- 机器学习
tags:
- machine-learning
- SVM
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

## 优化目标

### 简介

目前为止的监督学习算法都具有近似的性能，所以经常要考虑的东西不是选择算法A或算法B，而是去考虑所使用的数据量。这就体现了我们应用这些算法的技巧，比如学习特性的选择或正则化参数的选择。目前有一种算法广泛的应用于工业和学术界，这个算法被称作**支持向量机**（Support Vector Machine，SVM）。与Logistics和神经网络相比，SVM在学习复杂的非线性方程式能够提供更加清晰和强大的方式。



