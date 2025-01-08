---
title: Conjugate Gradient Method（共轭梯度法）
date: 2025-01-08 00:00:00
mathjax: true
category:
 - Mathematics
sortValue: 130004
---

- [An Introduction to the Conjugate Gradient Method Without the Agonizing Pain](https://www.cs.cmu.edu/~quake-papers/painless-conjugate-gradient.pdf)

是一种迭代方法，适合解稀疏系统的线性方程组

用于求解$$Ax=b$$，其中$A$是已知的对称正定系数矩阵，$x$是未知向量，$b$是已知向量

如果是密集矩阵适合用高斯消元法（先求上三角矩阵，然后再替代变量不断回溯），如果是稀疏矩阵适合用共轭梯度法

positive-definite matrix（正定矩阵）：对于任意非零向量$x$，都有$x^TAx>0$

quadratic form（二次型）：$f(x)=\frac{1}{2}x^TAx-b^Tx$

![Alt text](image.png)
二次型的最低点就是线性方程组的解
![Alt text](image-1.png)
![Alt text](image-2.png)
二次型的梯度定义如下：$$f'(x)=\begin{bmatrix}\frac{\partial f}{\partial x_1}\\\frac{\partial f}{\partial x_2}\\\vdots\\\frac{\partial f}{\partial x_n}\end{bmatrix} = \frac{1}{2}A^Tx+\frac{1}{2}Ax-b$$
如果A是对称矩阵，那么$f'(x)=Ax-b$