---
title: Games001 13.Solution of Linear Systems（线性系统求解）
mathjax: true
date: 2025-01-07 00:00:00
category:
 - Mathematics
sortValue: 130003
---

## Introduction

线性系统要解的问题就是，求：$$Ax = b$$这个矩阵方程组的解

图形学中矩阵方程出现在很多地方：
![alt text](image.png)

- 几何处理中隐式的 Laplace Smoothing
- 物理模拟中每一个时间步内都需要解一个大的线性方程组
- 全局渲染中的 radiosity-based method
- 如何针对不同的问题选择合适的矩阵求解器，并且让它解的又稳定又快又好是这里的中心问题

## 直接求解

### LU Decomposition

最直接的方法是使用高斯消元直接求解：
![alt text](image-1.png)
![alt text](image-2.png)

- 先求得这么一个上三角矩阵

![alt text](image-3.png)

- 通过反向的高斯消元法，求解出最终的解

上面的每一步行变换我们都可以表述成矩阵运算的形式：
![alt text](image-4.png)
所有的行变换构成一个 L 矩阵（下三角矩阵），而求得的矩阵就是 U 矩阵（上三角矩阵）：
![alt text](image-5.png)

- 高斯消元法把$Ax=b$分解成了$LUx=b$，然后我们可以先把 Ux 看作一个未知向量 y 求解$Ly=b$，然后通过$Ux=y$求解 x

![alt text](image-7.png)

- 这个算法是 Inplace 的，L 矩阵和 U 矩阵都存在原本的 A 矩阵里边，其中 L 矩阵的对角线都是 1，所以存储的时候直接忽略了，左下角部分存 L，右上角部分存 U。

![alt text](image-6.png)

- 比如第一行消第二行不光把第一个元素消掉了，还把第二个元素消掉了，这样后面做除法的时候就会出现除以 0，会导致数值不稳定。

### Cholesky Decomposition

![alt text](image-8.png)

- LU 分解在对称半正定的矩阵中有更简单的算法，这也是一种非常常用的算法，叫做 Cholesky Decomposition，也称之为 LLT Decomposition
- 在 A 是对称半正定矩阵时下三角矩阵和上三角矩阵是转置的关系

![alt text](image-9.png)

- 从$L_{11}$开始一直往下走，最后得到整个 L 矩阵的各个系数
- 计算量可以比 LU 分解减小一半，不过还是 O(n^3)的复杂度

![alt text](image-10.png)

![alt text](image-11.png)

- 前面提到的这两种算法一般来说只适用于求解小的矩阵，一般 n 小于 1000
- 但是图形学通常会有非常大的矩阵，这种方法可能需要花几十分钟甚至几个小时来解决

## 迭代求解

![alt text](image-12.png)

- 对于大规模的矩阵方程，一般使用迭代求解的方法
- 对于矩阵 A 是个黑盒但是 Ax 已知的情况也适用

![alt text](image-13.png)

- 这个弹性模型有 20 w 个节点，对应的矩阵方程的自由度是 60 w 维的矩阵

### Stationary Iteration（不动点迭代）

![alt text](image-14.png)

- 不断把$cos(x_i)$带入进去计算，最后会收敛到一个特定的值，这个值就是$x = cos(x)$的解

![alt text](image-15.png)

- 不动点迭代就是构造一个两项相减的形式
- 对于矩阵来说，我们把 A 分解为 M 和 N 相减

### Jacobi Iteration（雅可比迭代）

![alt text](image-16.png)
![alt text](image-17.png)

Jacobi Iteration 有时候会收敛，有时候会发散：
![alt text](image-18.png)

- 对于 3x3 的矩阵，Jacobi Iteration 需要十几步才能收敛

![alt text](image-19.png)

那么具体是哪些情况会失败呢？

### 不动点迭代的收敛条件

$$x^{k+1} = M^{-1}(Nx^k+b)$$
我们令 $N = M - A$，那么：$$x^{k+1} = x^k + M^{-1}(b - Ax^k)$$
定义误差：$$e_k = x^* - x^k$$，其中 $x^*$ 是真实解，那么残差：$$r_k = b - Ax^k = Ax^* - Ax^k = Ae_k$$
用$Ae_k$替换$b - Ax^k$，那么：$$x^{k+1} \leftarrow x^k + M^{-1}Ae_k$$
则：$$x^* - x^{k+1} \leftarrow x^* - x^k - M^{-1}Ae_k$$
所以误差更新关系是：
$$e_{k+1} \leftarrow (I - M^{-1}A)e_k = M^{-1}N e_k$$

![Alt text](image-20.png)

- 里边的特征值越小，收敛越快
- 关于特征值分解可以参考：[一文解释 矩阵特征分解](https://zhuanlan.zhihu.com/p/314464267)

![Alt text](image-21.png)

- 前面的标准在实际应用中使用比较困难，因为我们不太可能先求出 T 矩阵再对它进行特征值分解来判断每个特征值是否小于 1。所以实际应用中会使用一个简单的判断标准。
- 简单来说就是每行对角元素大于非对角元素的绝对值之和。

![Alt text](image-22.png)

- 对于非对角占优矩阵，我们可以使用松弛的方法，每次更新的步长乘以一个常数 $\omega$
- 如果矩阵不收敛，要取一个尽量小的值

![Alt text](image-23.png)

### Gauss-Seidel Iteration（高斯赛德尔迭代）

![Alt text](image-24.png)

- 其中$M^{-1}b$的计算可以利用高斯消元法中计算$Mx=b$的方式计算出$x$，这个$x$也就是$M^{-1}b$
- Jacobi M 是对角矩阵，我们可以并行的求逆。但 Gauss-Seidel M 是下三角矩阵，我们只能串行的求逆。不过可以通过重排矩阵的方式得到并行版本的 Gauss-Seidel 矩阵

![Alt text](image-25.png)

### Conclusion

![Alt text](image-26.png)

- 关于 MultiGrid Method，可以参考：[A Multigrid Tutorial, 2nd Edition](https://www.math.hkust.edu.hk/~mamu/courses/531/tutorial_with_corrections.pdf)

## Subspace Methods（子空间方法）

![Alt text](image-27.png)

- 我们限制 x 只能由$(e_1,...,e_m)$构成的线性组合，然后只考虑 x 在这个 m 维空间上如何让这个$x-b$的残差最小。这样每一步我们只需要考虑子空间的问题，而不是整个优化空间。

### Krylov Subspace（克里洛夫子空间）

![alt text](image-28.png)

- 这里的 span 也就是指由后面这些基线性组合的空间，当然这些基不一定是正交的。

### Conjugate Gradient Method（共轭梯度法）

![alt text](image-29.png)

- CG 是最常用的 Krylov 子空间迭代方法
- 将$Ax=b$转化为一个上述形式的优化问题，当优化问题取最小值时，梯度应该是 0。而且由于 A 是对称半正定的，所以这个二次型是有唯一最小值的。
- 子空间迭代的过程中，什么时候能取到最小值呢？根据几何知识，当梯度正交于子空间时，就能取到最小值。也就是说，如果梯度正交于子空间，那么在子空间中无论怎么移动都无法让残差进一步变小。

![alt text](image-30.png)

- 我们把前面的每一步梯度正交于子空间的要求转为上诉构造共轭基的要求，这样每一步迭代只需要解一个一维优化问题，就可以得到从$x_{k-1}$更新到$x_k$。这就是 CG 算法的核心。

下面我们来证明为什么构造这样的共轭基，然后在每一步迭代中只需要解一个一维优化问题就可以保证梯度正交于子空间：
![alt text](image-31.png)

- $p_1$的时候可以直接解这个一维的优化问题，解完能够保证$r_1 \perp K_1$，
- 这里没有展开将是如何构造这样的 P 的，这个涉及到 Krylov 子空间的一些独特性质。

![alt text](image-32.png)

关于GPU实现的共轭梯度法，可以参考：[Parallel Algorithm of Conjugate Gradient Solver using OpenGL Compute Shader](https://koreascience.kr/article/JAKO202103440878912.pdf)

### CG 算法的效率

![alt text](image-33.png)

- $x_k$是迭代到第 k 步的解
- $\kappa$一定是大于等于 1 的，如果很大例如 1000，那么$\frac{\kappa-1}{\kappa+1}$就会趋向于 1，如果$\kappa$趋向于 1，那么这个值就会趋向于 0，也就是说如果 A 矩阵的条件数越小，那么这个幂指数的底数就越小，越接近于 0，整个收敛就会越快。
- 所以我们希望矩阵的条件数越接近于 1，这样能保证共轭梯度算法的收敛是非常快的。

### Preconditioning（预处理）

![alt text](image-34.png)

- preconditioning 是一个非常强大的一个技术，ICPCG 是我们求解对称半正定矩阵方程的时候非常常用的一个求解器。

### 其他 Krylov 子空间求解器

![alt text](image-35.png)

### 总结

![alt text](image-36.png)

## Questions

### 什么是对称半正定矩阵？

对称半正定矩阵是一个具有以下性质的方阵：

1. 对称性：矩阵等于其转置，即 $A = A^T$。
2. 半正定性：对于任何非零向量 $x$，都有 $ x^T A x \geq 0 $。

这意味着所有特征值都是非负的。如果一个对称半正定矩阵的所有特征值都是正的，那么它就是对称正定矩阵。对称半正定矩阵在许多数学和工程应用中都有重要的作用，例如在优化问题、统计学和机器学习中。

### 什么是复数特征值？

复数特征值是矩阵特征值的一种，它是一个复数。对于一个方阵 $ A $，如果存在一个复数 $ \lambda $ 和一个非零向量 $ v $，使得 $ A v = \lambda v $，那么 $ \lambda $ 就是矩阵 $ A $ 的一个特征值，而 $ v $ 是对应的特征向量。

复数特征值通常成对出现，即如果 $ \lambda = a + bi $ 是一个特征值，那么它的共轭复数 $ \overline{\lambda} = a - bi $ 也是特征值，这在实矩阵的情况下尤为常见。这是因为实矩阵的特征多项式具有实系数，而实系数多项式的复根总是成对出现的。

复数特征值在许多领域都有应用，例如在控制理论、信号处理和量子力学中。在这些领域中，矩阵的特征值可以提供系统的重要信息，如稳定性、振荡频率等。

### 什么是谱半径？

谱半径是矩阵理论中的一个概念，它指的是一个方阵的所有特征值的模（绝对值）中的最大值。对于一个方阵 $ A $，其谱半径 $ \rho(A) $ 定义为：

\[
\rho(A) = \max\{|\lambda| : \lambda \text{ 是 } A \text{ 的特征值}\}
\]

谱半径是衡量矩阵特征值大小的一个重要指标，它在矩阵的稳定性分析、收敛性研究以及控制理论等领域中都有广泛的应用。例如，在迭代法求解线性方程组时，谱半径可以用来判断迭代法的收敛性；在控制理论中，谱半径可以用来判断系统的稳定性。

谱半径与矩阵的范数之间存在一定的关系，但它们并不完全相同。矩阵的范数是衡量矩阵大小的一种方式，而谱半径则专注于特征值的模的最大值。在某些情况下，谱半径可以小于或等于矩阵的范数，但并不总是相等。

## Resources

- 关于线性系统相关数学推导：[Housz 的杂货铺](https://www.zhihu.com/column/c_1263955087426830336)
- 共轭梯度法：[共轭梯度法通俗讲义](https://flat2010.github.io/2018/10/26/%E5%85%B1%E8%BD%AD%E6%A2%AF%E5%BA%A6%E6%B3%95%E9%80%9A%E4%BF%97%E8%AE%B2%E4%B9%89/#1-%E7%AE%80%E4%BB%8B)
- [Golub, Van Loan - Matrix Computations](https://github.com/CompPhysics/ComputationalPhysicsMSU/blob/master/doc/Lectures/Golub%2C%20Van%20Loan%20-%20Matrix%20Computations.pdf)
- [MATH662 Numerical Linear Algebra](http://mitran-lab.amath.unc.edu/courses/MATH662/MATH662.xhtml)
