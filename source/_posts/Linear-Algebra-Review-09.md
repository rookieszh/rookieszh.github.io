---
title: Linear Algebra Review 09
date: 2020-04-23 20:38:04
categories:
    - Mathematics
tags:
    - LinearAlgebra
    - MIT
description: "MIT Linear Algebra Review ——09 Linear Independence and Basis and Dimensions"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 第九讲：线性相关性、基、维数

$v_1,\ v_2,\ \cdots,\ v_n$是$m\times n$矩阵$A$的列向量：
* 如果$A$零空间中有且仅有$0$向量，则各向量线性无关，$rank(A)=n$。秩为主元数量，自由变量是主元的线性组合得到的。所以线性无关则没有自由变量
* 如果存在非零向量$c$使得$Ac=0$，则存在线性相关向量，$rank(A)\lt n$。（即解x的线性组合能得到0向量，又比如：$v_1=\left[\begin{array}{c c} 2\newline 1 \end{array}\right]$，$v_2=\left[\begin{array}{c c} 0\newline 0 \end{array}\right]$，0倍的$v_1$加上任意倍的$v_2$会得到$\left[\begin{array}{c c} 0\newline 0 \end{array}\right]$，则称$v_1$与$v_2$线性相关）

Vector $v_1, \cdots, v_2$ span a space means: the sapce consists of all combinations of these vectors.

向量空间$S$中的一组基（basis），具有两个性质：
1. 他们线性无关；
2. 他们可以生成$S$。

examples: space is $\mathbb{R}^3$, one basis is $\left[\begin{array}{c c c}1\newline 0\newline 0\end{array}\right], \left[\begin{array}{c c c}0\newline 1\newline 0\end{array}\right], \left[\begin{array}{c c c}0\newline 0\newline 1\end{array}\right]$
对于向量空间$\mathbb{R}^n$，如果$n$个向量组成的矩阵为可逆矩阵，则这$n$个向量为该空间的一组基，而数字$n$就是该空间的维数（dimension），注意不是矩阵的维数。
例如：$
A=
\begin{bmatrix}
1 & 2 & 3 & 1 \newline 
1 & 1 & 2 & 1 \newline 
1 & 2 & 3 & 1 \newline 
\end{bmatrix}
$，A的列向量线性相关，其零空间中有非零向量，所以$rank(A)=2=主元存在的列数=列空间维数$。

可以很容易的求得$Ax=0$的两个解，如$
x_1=
\begin{bmatrix}
-1 \newline 
-1 \newline 
1 \newline 
0 \newline 
\end{bmatrix}, 
x_2=
\begin{bmatrix}
-1 \newline 
0 \newline 
0 \newline 
1 \newline 
\end{bmatrix}
$，根据前几讲，我们知道特解的个数就是自由变量的个数，所以$n-rank(A)=2=自由变量存在的列数=零空间维数$

最后我们得到：列空间维数$dim C(A)=rank(A)$，零空间维数$dim N(A)=n-rank(A)$