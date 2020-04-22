---
title: Linear Algebra Review 08
date: 2020-04-22 21:31:47
categories:
    - Mathematics
tags:
    - LinearAlgebra
    - MIT
description: "MIT Linear Algebra Review ——08 Solve Ax=b"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 第八讲：求解$Ax=b$：可解性和解的结构

举例，同上一讲：$3 \times 4$ 的矩阵$
A=
\begin{bmatrix}
1 & 2 & 2 & 2\newline 
2 & 4 & 6 & 8\newline 
3 & 6 & 8 & 10\newline 
\end{bmatrix}
$，求$Ax=b$的特解：

先写出其增广矩阵（augmented matrix）$\left[\begin{array}{c|c}A & b\end{array}\right]$：
$$
\left[
\begin{array}{c c c c|c}
1 & 2 & 2 & 2 & b_1 \newline 
2 & 4 & 6 & 8 & b_2 \newline 
3 & 6 & 8 & 10 & b_3 \newline 
\end{array}
\right]
\underrightarrow{消元}
\left[
\begin{array}{c c c c|c}
1 & 2 & 2 & 2 & b_1 \newline 
0 & 0 & 2 & 4 & b_2-2b_1 \newline 
0 & 0 & 0 & 0 & b_3-b_2-b_1 \newline 
\end{array}
\right]
$$

显然，有解的必要条件为$b_3-b_2-b_1=0$。

讨论$b$满足什么条件才能让方程$Ax=b$有解（solvability condition on b）：当且仅当$b$属于$A$的列空间时。另一种描述：如果$A$的各行线性组合得到$0$行，则$b$端分量做同样的线性组合，结果也为$0$时，方程才有解。

解法：令所有自由变量取$0$，则有$\begin{cases}x_1 &+&2x_3 & = 1 \newline  && 2x_3& = 3 \newline \end{cases}$，解得$\begin{cases}x_1 & = & -2 \newline x_3 & = & \frac{3}{2} \newline \end{cases}$
，代入$Ax=b$求得特解$
x_p=
\begin{bmatrix}
-2 \newline  0 \newline  \frac{3}{2} \newline  0
\end{bmatrix}
$。然后令$Ax=0$求解得零空间解$x_n$(自由变量分别取1)

令$Ax=b$成立的所有解：
$$\begin{cases}
A{x_p} = & b \newline 
A{x_n} = & 0 \newline 
\end{cases}
\quad
\underrightarrow{两式相加}
\quad
A(x_p+x_n)=b
$$

即$Ax=b$的解集为其特解加上零空间，对本例有：
$
x_{complete}=
\begin{bmatrix}
-2 \newline  0 \newline  \frac{3}{2} \newline  0
\end{bmatrix}
+
c_1\begin{bmatrix}-2\newline 1\newline 0\newline 0\newline \end{bmatrix}
+
c_2\begin{bmatrix}2\newline 0\newline -2\newline 1\newline \end{bmatrix}
$

对于$m \times n$矩阵$A$，有矩阵$A$的秩$r \leq min(m, n)$

列满秩$r=n$情况（即没有自由变量）：$
A=
\begin{bmatrix}
1 & 3 \newline 
2 & 1 \newline 
6 & 1 \newline 
5 & 1 \newline 
\end{bmatrix}
\underrightarrow{消元} 
\begin{bmatrix}
1 & 0 \newline 
0 & 1 \newline 
0 & 0 \newline 
0 & 0 \newline 
\end{bmatrix}
$，$rank(A)=2$，要使$Ax=b, b \neq 0$有非零解，$b$必须取$A$中各列的线性组合，此时A的零空间中只有$0$向量，列之间的线性组合无法组成0向量。$x=x_p$，0或1个解

行满秩$r=m$情况：$
A=
\begin{bmatrix}
1 & 2 & 6 & 5 \newline 
3 & 1 & 1 & 1 \newline 
\end{bmatrix}
$，$rank(A)=2$，$\forall b \in R^m都有x \neq 0的解$，因为此时$A$的列空间为$R^m$，$b \in R^m$恒成立，组成$A$的零空间的自由变量有n-r个。

行列满秩情况：$r=m=n$，如$
A=
\begin{bmatrix}
1 & 2 \newline 
3 & 4 \newline 
\end{bmatrix}
$，则$A$最终可以化简为$R=I$，其零空间只包含$0$向量。同时矩阵可逆。

总结：

$$\begin{array}{c|c|c|c}r=m=n&r=n\lt m&r=m\lt n&r\lt m,r\lt n\newline R=I&R=\begin{bmatrix}I\newline 0\end{bmatrix}&R=\begin{bmatrix}I&F\end{bmatrix}&R=\begin{bmatrix}I&F\newline 0&0\end{bmatrix}\newline 1\ solution&0\ or\ 1\ solution&\infty\ solution&0\ or\ \infty\ solution\end{array}$$