---
title: Linear Algebra Review 05
date: 2020-04-22 21:31:38
categories:
    - Mathematics
tags:
    - LinearAlgebra
    - MIT
description: "MIT Linear Algebra Review ——05 Permutation , Transpose, Symmetric and VectorSpace "
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 第五讲：转换、置换、向量空间R
### 置换矩阵（Permutation Matrix）
$P$为置换矩阵，对任意可逆矩阵$A$有：
$PA=LU$
$n$阶方阵的置换矩阵$P$有$\binom{n}{1}=n!$个
对置换矩阵$P$，有$P^TP = I$
即$P^T = P^{-1}$

### 转置矩阵（Transpose Matrix）
$(A^T)_{ij} = A_{ji}$

### 对称矩阵（Symmetric Matrix）
$A^T$ = $A$
对任意矩阵$R$有$R^TR$为对称矩阵：
$$
(R^TR)^T = (R)^T(R^T)^T = R^TR
\textrm{，即}(R^TR)^T = R^TR
$$

### 向量空间（Vector Space）
$R^2$ = all 2-dim real vectors
所有向量空间都必须包含原点 Origin，因为向量空间中任意向量的数乘、求和运算得到的向量也在该空间中。一个向量乘以0得零向量。即向量空间要满足加法封闭和数乘封闭。任何的subspaces都必须过原点
Subspaces of $R^2$: 
1. all of $R^2$
2. any line through $\left[\begin{array}{c}{0} \newline  {0} \end{array}\right]$
3. zero vector only

columns in $R^3$, like $A=\left[\begin{array}{cc} 1 & 1 \newline  2 & 3 \newline  4 & 1\end{array}\right]$, all their($\left[\begin{array}{c}{1} \newline  {2} \newline  {4} \end{array}\right]$ and $\left[\begin{array}{c}{1} \newline  {3} \newline  {1} \end{array}\right]$) combinations form a subspace, called column space $C(A)$