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

### Logistics回归的另一种见解

{% asset_img ml12_1.png This is an example image %}

首先看看Logistics回归的假设函数。

- 如果y=1，想让$h_\theta(x)\approx1$，$\theta^Tx \gg 0$

- 如果y=0，想让$h_\theta(x)\approx0$，$\theta^Tx \ll 0$

{% asset_img ml12_2.png This is an example image %}

再看看单样本的代价函数（省略了所有用例求和）。从上图可以看出当y分别为1和0时相应的单样本代价函数的曲线图。

- 当y=1时，从点(1, 0)开始，首先向右的一条平坦的直线，然后向左的一条与Logistic代价函数相似趋势的一条曲线，这就是当y=1时SVM需要用到的代价函数。
- 当y=0时，从点(-1, 0)开始，首先向左的一条平坦的直线，然后向右的一条与Logistic代价函数相似趋势的一条曲线，这就是当y=0时SVM需要用到的代价函数。

左边的代价函数就为$cost_1(z)$，右边的代价函数为$cost_0(z)$。

SVM代价函数与Logistic代价函数的效果很相似，但是SVM代价函数可以使SVM拥有计算上的优势并使之后的优化问题更容易解决。

{% asset_img ml12_3.png This is an example image %}

Logistic代价函数和SVM代价函数对比如上图所示。

Losigtic使用$\lambda$参数针对后一项进行优化，而SVM使用$C$参数针对前一项进行优化。
$$
A + \lambda B
\\\\
CA + B
$$


## 大间距直觉理解

{% asset_img ml12_4.png This is an example image %}

为了使代价函数的值最小化（无限趋向于0）

- 针对正样本y=1，$\theta^Tx \ge 1$，但是其实保证$\theta^T x \ge 0$就能够正确的分类。
- 正对负样本y=0，$\theta^Tx \le -1$，但是其实保证$\theta^T x < 0$就能够正确的分类。

这样就相当于在SVM的安全因子中构建一个安全间距。

{% asset_img ml12_5.png This is an example image %}

假设C非常大的情况下。在处理这种代价函数的参数$\theta$的优化时，会得到一个非常有趣的决策边界。

{% asset_img ml12_6.png This is an example image %}

{% asset_img ml12_7.png This is an example image %}

现在的大间距的决策边界都是在$C$很大的情况下得到的，实际上SVM对决策边界划分更加敏感，尤其是在大间距分类器的情况下，一个样本都有可能影响决策边界的划分。所以这时需要权衡正则化参数$C$的选择来决策一个相对较好的决策边界。

## 大间距分类机制

### 向量内积

给定两个向量：
$$
u =\begin{bmatrix}
		u_1 \\\\
		u_2  
	\end{bmatrix}
$$

$$
v =\begin{bmatrix}
		v_1 \\\\
		v_2  
	\end{bmatrix}
$$

根据**毕达哥拉斯定理**，向量u的**欧几里得**长度为：
$$
||u|| = \sqrt{u_1^2 + u_2^2}
$$
{% asset_img ml12_8.png This is an example image %}

可以得出（P为向量v到向量u的投影长度）：
$$
u^Tv=p ||u|| = u_1 v_1 + u_2 v_2
$$




### SVM决策边界

{% asset_img ml12_9.png This is an example image %}

{% asset_img ml12_10.png This is an example image %}

Note：决策边界直线与参数$\theta$向量是垂直的关系。
$$
\Theta^T x^{(i)} = p^{(i)}||\Theta||
$$

## SVM内核1

### SVM特征

在Logistic回归中，如果要预测y=1，则：
$$
\theta_0 + \theta_1 x_1 + \theta_2 x_2 + \theta_3 x_1 x_2 + \theta_4 x_1^2 + ... \ge 0
$$
在SVM中，使用$f_j$去代表每个特征，所以上面的等式变成：
$$
\theta_0 + \theta_1 f_1 + \theta_2 f_2 + \theta_3 f_3 + \theta_4 f_4 + ... \ge 0
$$
如果使用多项式等方式去产生相应$f_j$，相应的计算量会相当的大。有没有一个更好的方式去生成特征？

### 内核和相似度

{% asset_img ml12_11.png This is an example image %}

给定x，使用邻近地标（$l^{(1)} ,l^{(2)}, l^{(3)} ...$）的数计算新特性。
$$
f_1 = similarity(x,l^{(1)})=exp(-\frac{||x-l^{(1)}||^2}{2\sigma^2})
$$

$$
f_2 = similarity(x,l^{(2)})=exp(-\frac{||x-l^{(2)}||^2}{2\sigma^2})
$$

$$
f_3 = similarity(x,l^{(3)})=exp(-\frac{||x-l^{(3)}||^2}{2\sigma^2})
$$

$$
f_j = similarity(x,l^{(j)})=exp(-\frac{||x-l^{(j)}||^2}{2\sigma^2})
$$

这个就被称为**内核**（Kernel），**高斯内核**（Gaussian Kernal）。

如果$x \approx l^{(1)}$：
$$
f_1 \approx exp(-\frac{0}{2\sigma^2}) \approx 1
$$
如果$x$与$l^{(1)}$差距很大：
$$
f_1=exp(-\frac{large\\_num^2}{2\sigma^2})\approx 0
$$

### 例子

下图展示了参数$\sigma$对内核结果的影响。

{% asset_img  ml12_12.png This is an example image %}



## SVM内核2

### SVM算法

给定训练数据：
$$
(x^{(1)}, y^{(1)}),(x^{(2)}, y^{(2)}),(x^{(3)}, y^{(3)}),...,(x^{(m)}, y^{(m)}),
$$
地标$l$的选择为：
$$
l^{(1)}=x^{(1)},l^{(2)}=x^{(2)},l^{(3)}=x^{(3)},...,l^{(m)}=x^{(m)}
$$
给定训练用例$x$得到新的特性为：
$$
f_1=similarity(x,l^{(1)})
$$

$$
f_2=similarity(x,l^{(2)})
$$

$$
...
$$

**假设函数**：给定$x$，计算特性$f\in \Re^{(m+1)}$

​	预测$y=1$，如果$\theta^T \ge 0$	

**训练代价函数**：
$$
C\sum_{i=1}^{m}{y^{(i)}cost_1(\theta^Tf^{(i)}) + (1-y^{(i)})cost_0(\theta^Tf^{(i)})} + \frac{1}{2}\sum_{j=1}^{m}{\theta_j^2}
$$

### SVM参数

参数$C=\frac{1}{\lambda}$：

- 过大：低偏差，高方差。
- 过小：高偏差，低方差。

参数$\sigma^2$：

- 过大：特性$f^{(i)}$会非常平稳，导致高偏差，低方差。
- 过小：特性$f^{(i)}$会起伏比较大，导致低偏差，高方差。

## 使用SVM

一般使用SVM软件包去求解参数$\theta$。此时需要自行选择参数$C$和**内核函数**。

目前内核函数有两种选择，一个是**线性内核**（Liner Kernal），另一个是**高斯内核**（Gaussian Kernal）。

线性内核：
$$
\theta^T x
$$
高斯内核：
$$
f_i = exp(-\frac{||x-l^{(i)}||^2}{2\sigma^ 2}) \qquad  l^{(i)}=x^{(i)}
$$

$$
\theta^Tf_i
$$

### 其他类型内核

并不是所有的相似函数$similarity(x,l)$都是合法的内核，需要满足技术条件（明格尔定理），确保SVM包优化正确运行，不产生偏离。

目前现成的内核有：

- 高阶内核
- 更加晦涩的内核：字符串内核（String kernal），chi-square kernal，histogram intersection kernal。

### 多元分类

{% asset_img  ml12_12.png This is an example image %}

许多SVM包已经内建多元分类功能。如果不存在，使用一对多方式。（训练$K$维SVM，从余下中用一个值去区分$y=i$），获取$\theta^{(1)},\theta^{(2)},....,\theta^{(K)}$。选择$(\theta)^T x$值最大的类别。

### Logisti回归 vs. SVM

$n = $特性数量，$m=$训练用例。

- 如果$n$相对于$m$很大：使用Logistic回归或者没有内核的SVM。
- 如果$n$比较小，$m$中等大小：使用带有高斯内核的SVM。
- 如果$n$比较小，$m$很大：创建/添加更多的特性，使用Logistic回归或者没有内核的SVM。

神经网络似乎在大多数情况下都能运行的很好，但是训练更加缓慢。









































