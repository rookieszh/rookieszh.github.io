---
title: Linear Algebra Review 04
date: 2020-04-19 23:05:23
categories:
    - Mathematics
tags:
    - LinearAlgebra
    - MIT
description: "MIT Linear Algebra Review ——04 LU Factorization and Transpose and Permutation"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 第四讲：$A$ 的 $LU$ 分解
### 转置矩阵
$AB$的逆矩阵：
$$
\begin{aligned}
A \cdot A^{-1} = I & = A^{-1} \cdot A\newline 
(AB) \cdot (B^{-1}A^{-1}) & = I\newline 
\textrm{则} AB \textrm{的逆矩阵为} & B^{-1}A^{-1}
\end{aligned}
$$

$A^{T}$的逆矩阵：
$$
\begin{aligned}
(A \cdot A^{-1})^{T} & = I^{T}\newline 
(A^{-1})^{T} \cdot A^{T} & = I\newline 
\textrm{则} A^{T} \textrm{的逆矩阵为} & (A^{-1})^{T}
\end{aligned}
$$

假设 $E_{32}E_{31}E_{21}A = U$ (no row exchanges)，那么转换成 $A = LU$ 的形式则为：$A=E_{21}^{-1}E_{31}^{-1}E_{32}^{-1}U=LU$

### 将一个 $n$ 阶方阵 $A$ 变换为 $LU$ 需要的计算量估计：

1. 第一步，将$a_{11}$作为主元，需要的运算量约为$n^2$
$$
\begin{bmatrix}
a_{11} & a_{12} & \cdots & a_{1n} \newline 
a_{21} & a_{22} & \cdots & a_{2n} \newline 
\vdots & \vdots & \ddots & \vdots \newline 
a_{n1} & a_{n2} & \cdots & a_{nn} \newline 
\end{bmatrix}
\underrightarrow{消元}
\begin{bmatrix}
a_{11} & a_{12} & \cdots & a_{1n} \newline 
0      & a_{22} & \cdots & a_{2n} \newline 
0      & \vdots & \ddots & \vdots \newline 
0      & a_{n2} & \cdots & a_{nn} \newline 
\end{bmatrix}
$$

2. 以此类推，接下来每一步计算量约为$(n-1)^2、(n-2)^2、\cdots、2^2、1^2$。

3. 则将 $A$ 变换为 $LU$ 的总运算量应为$O(n^2+(n-1)^2+\cdots+2^2+1^2)$，即$O(\frac{n^3}{3})$。（1/3是因为 从1到n对$x^2dx$进行积分）

### 置换矩阵(Permutation Matrix)：

3阶方阵的置换矩阵（即进行行互换）有6个：
$$
\begin{bmatrix}
1 & 0 & 0 \newline 
0 & 1 & 0 \newline 
0 & 0 & 1 \newline 
\end{bmatrix}
\begin{bmatrix}
0 & 1 & 0 \newline 
1 & 0 & 0 \newline 
0 & 0 & 1 \newline 
\end{bmatrix}
\begin{bmatrix}
0 & 0 & 1 \newline 
0 & 1 & 0 \newline 
1 & 0 & 0 \newline 
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 \newline 
0 & 0 & 1 \newline 
0 & 1 & 0 \newline 
\end{bmatrix}
\begin{bmatrix}
0 & 1 & 0 \newline 
0 & 0 & 1 \newline 
1 & 0 & 0 \newline 
\end{bmatrix}
\begin{bmatrix}
0 & 0 & 1 \newline 
1 & 0 & 0 \newline 
0 & 1 & 0 \newline 
\end{bmatrix}
$$

$n$阶方阵的置换矩阵有$\binom{n}{1}=n!$个。