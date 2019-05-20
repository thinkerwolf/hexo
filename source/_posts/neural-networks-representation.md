---
title: 神经网络表示
date: 2019-05-06 10:07:16
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

## 模型表示I
神经元是基本的运算单位，将输入作为电信号输入引导至输出中。其中树突就像输入特性$X_1...X_n$，输出就是假设函数的结果。在这个模型中，$X_0$输入节点又称为“偏移单位”。使用与logsitic函数相同的函数作为预测函数$\frac{1}{1+e^{\theta^Tx}}$。在这种情况下，theta参数被称作“weights”。
通常一个简单的表示如下所示：
$$
    \begin{bmatrix}
		X_0 \\\\
		X_1 \\\\
        X_2 \\\\
		X_n
	\end{bmatrix}
    ->
    []
    ->
    h_\theta(x)
$$
我们的输入节点（层级1），被称作“输入层”，进入另一个节点（层级2），最后输出预测函数，称作“输出层”。

在输入层和输出层之间称作“隐藏层”。在这个例子中，我们将隐藏层的节点用$a_0^2...a_n^2$标记，称为“激活单位”。
$a_i^j=层级j的激活单位i$
$\Theta^j=从层级j到层级j+1的权重矩阵$
$$
    \begin{bmatrix}
		X_0 \\\\
		X_1 \\\\
        X_2 \\\\
		X_3
	\end{bmatrix}
    ->
    \begin{bmatrix}
		a_1^2 \\\\
		a_2^2 \\\\
        a_3^2 \\\\
	\end{bmatrix}
    ->
    h_\Theta(x)
$$
每个激活点获取途径如下：
$$
a_1^{(2)}=g(\Theta_{10}^{(1)}x_0+\Theta_{11}^{(1)}x_1+\Theta_{12}^{(1)}x_2+\Theta_{13}^{(1)}x_3)
$$
$$
a_2^{(2)}=g(\Theta_{20}^{(1)}x_0+\Theta_{21}^{(1)}x_1+\Theta_{22}^{(1)}x_2+\Theta_{23}^{(1)}x_3)
$$
$$
a_3^{(2)}=g(\Theta_{30}^{(1)}x_0+\Theta_{31}^{(1)}x_1+\Theta_{32}^{(1)}x_2+\Theta_{33}^{(1)}x_3)
$$
$$
h_\Theta^{(x)}=a_1^{(3)}=g(\Theta_{10}^{(2)}a_0^{(2)}+\Theta_{11}^{(2)}a_1^{(2)}+\Theta_{12}^{(2)}a_2^{(2)}+\Theta_{13}^{(2)}a_3^{(2)})
$$
假设函数输出就是应用Logistic函数求出所有激活点的和。每个层级拥有其自身的权重矩阵，$\Theta^{(j)}$。

**如果网络中在层级j有$s_j$个单元，层级j+1有$s_j+1$个单位，$\Theta^{(j)}$的维度就是$s_j+1x(s_j+1)$**
+1是$\Theta^{(j)}$的“偏置节点”$\Theta_0^{(j)}$。换句话说就是输出节点不包括偏置节点，但是输入节点包括。

## 模型表示II
一下是神经网络的一个例子：
$$
a_1^{(2)}=g(\Theta_{10}^{(1)}x_0+\Theta_{11}^{(1)}x_1+\Theta_{12}^{(1)}x_2+\Theta_{13}^{(1)}x_3)
$$
$$
a_2^{(2)}=g(\Theta_{20}^{(1)}x_0+\Theta_{21}^{(1)}x_1+\Theta_{22}^{(1)}x_2+\Theta_{23}^{(1)}x_3)
$$
$$
a_3^{(2)}=g(\Theta_{30}^{(1)}x_0+\Theta_{31}^{(1)}x_1+\Theta_{32}^{(1)}x_2+\Theta_{33}^{(1)}x_3)
$$
$$
h_\Theta^{(x)}=a_1^{(3)}=g(\Theta_{10}^{(2)}a_0^{(2)}+\Theta_{11}^{(2)}a_1^{(2)}+\Theta_{12}^{(2)}a_2^{(2)}+\Theta_{13}^{(2)}a_3^{(2)})
$$
定义一个新的变量$z_k^{(j)}$来压缩g函数。
$$
a_1^{(2)}=g(z_1^{(2)})
$$
$$
a_2^{(2)}=g(z_2^{(2)})
$$
$$
a_3^{(2)}=g(z_3^{(2)})
$$
$$
z_k^{(j)}=\Theta_{k,0}^{(j-1)}x_0+\Theta_{k,1}^{(j-1)}x_1+...+\Theta_{k,n}^{(j-1)}x_n
$$
$x$和$z^{(j)}的向量表示是$
$$
x = \begin{bmatrix}
		X_0 \\\\
		X_1 \\\\
        ... \\\\
		X_n
	\end{bmatrix}
z^{(j)}=\begin{bmatrix}
		z_1^{(j)} \\\\
		z_2^{(j)} \\\\
        ... \\\\
		z_n^{(j)}
	\end{bmatrix}
$$
设置$x=a^{(1)}$，将等式重写成：
$$
z^{(j)}=\Theta^{(j-1)}a^{(j-1)}
z^{(j+1)}=\Theta^{(j)}a^{(j)}
$$

$$
h_\Theta(x)=a^{(j+1)}=g(z^{(j+1)})
$$

## 多元分类


