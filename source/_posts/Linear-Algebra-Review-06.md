---
title: Linear Algebra Review 06
date: 2020-04-22 21:31:41
categories:
    - Mathematics
tags:
    - LinearAlgebra
    - MIT
description: "MIT Linear Algebra Review ——06 Column Space and Null Space"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 第六讲：列空间和零空间
A plane through $\left[\begin{array}{c}{0} \newline  {0} \newline  {0} \end{array}\right]$ is subspace of $\mathbb{R}^3$, called P; A line through $\left[\begin{array}{c}{0} \newline  {0} \newline  {0} \end{array}\right]$ is subspace of $\mathbb{R}^3$, called L. L is not in the plane.
$P \cup L$ is not a subspace, 因为无法满足加法封闭性。
$P \cap L$ is a subspace, 因为是个零向量。
即对向量子空间$S$和$T$，有$S \cap T$也是向量子空间。

对$m \times n$矩阵$A$（column space of matrix A is a subspace of $\mathbb{R}^m$），$n \times 1$矩阵$x$，$m \times 1$矩阵$b$，运算$Ax=b$：

$\left[\begin{array}{ccccc}a_{11} & a_{12} & \cdots & a_{1(n-1)} & a_{1 n} \newline  a_{21} & a_{22} & \cdots & a_{2(n-1)} & a_{2 n} \newline  \vdots & \vdots & \ddots & \vdots & \vdots \newline  a_{m 1} & a_{m 2} & \cdots & a_{m(n-1)} & a_{m n}\end{array}\right] \cdot\left[\begin{array}{c}x_{1} \newline  x_{2} \newline  \vdots \newline  x_{n-1} \newline  x_{n}\end{array}\right]=\left[\begin{array}{c}b_{1} \newline  b_{2} \newline  \vdots \newline  b_{m}\end{array}\right]$

1. 由$A$的列向量的linear combination生成的子空间为$A$的列空间
2. $Ax=b$有非零解当且仅当$b$属于$A$的列空间
3. A的零空间($\mathbb{R}^n$)是$Ax=0$中$x$的解组成的集合。
if Av=0  and Aw=0, then A(v+w)=0, then A(12v)=0