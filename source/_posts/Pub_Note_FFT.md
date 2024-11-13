---
title: 傅里叶变换
mathjax: true
category:
 - Mathematics
sortValue: 100002
---

- [三角函数的正交性及其公式推导](https://blog.csdn.net/zhaohongfei_358/article/details/118461151)
- [三角恒等式](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E6%81%92%E7%AD%89%E5%BC%8F#%E8%A7%92%E7%9A%84%E5%92%8C%E5%B7%AE%E6%81%86%E7%AD%89%E5%BC%8F)
- [傅里叶变换一步步详细推导](https://blog.csdn.net/wd18508423052/article/details/100940771)
- [Games001-傅里叶变换与球谐函数](https://www.bilibili.com/video/BV1MF4m1V7e3)

## Prerequisite

积化和差公式：
$$ sin(a)cos(b) = \frac{1}{2} [sin(a+b) + sin(a-b)] $$
$$ cos(a)sin(b) = \frac{1}{2} [sin(a+b) - sin(a-b)] $$
$$ cos(a)cos(b) = \frac{1}{2} [cos(a+b) + cos(a-b)] $$
$$ sin(a)sin(b) = -\frac{1}{2} [cos(a+b) - cos(a-b)]$$

和角公式：

![alt text](image-2.png)
$$ sin(a+b) = sin(a)cos(b) + cos(a)sin(b) $$
$$ cos(a+b) = cos(a)cos(b) - sin(a)sin(b) $$
$$ sin(a-b) = sin(a)cos(b) - cos(a)sin(b) $$
$$ cos(a-b) = cos(a)cos(b) + sin(a)sin(b) $$

三角函数系：{1, sin(x), cos(x), sin(2x), cos(2x), ...}，他们具有一个很好的性质：正交性。也就是说，如果两个不同的三角函数相乘，然后在一个周期内积分，那么结果为 0。也就是：
$$ \int_{-\pi}^{\pi} sin(nx)sin(mx)dx = 0, n \neq m, n, m \in Z $$
$$ \int_{-\pi}^{\pi} cos(nx)cos(mx)dx = 0, n \neq m, n, m \in Z $$
$$ \int_{-\pi}^{\pi} sin(nx)cos(mx)dx = 0, n, m \in Z $$

<!-- 推导一下：
$$ \int*{-\pi}^{\pi} sin(nx)sin(mx)dx = 0, n \neq m, n, m \in Z $$
根据积化和差公式：
$$ = -\frac{1}{2}\int*{-\pi}^{\pi} \frac{1}{2} [cos((n+m)x) - cos((n-m)x)]dx $$ -->

## 傅里叶级数

$$ f(t) = a_0 + \sum_{n=1}^{\infty} [a_n cos(n\omega t) + b_n sin(n\omega t)] n \in Z $$

傅里叶级数通过欧拉公式表示为更简洁的写法
![alt text](image-3.png)

## 傅里叶变换

an和bn可以进一步表示（思路就是对函数积分，然后转为对傅里叶级数积分，通过三角函数系的正交性消去其中的一些项）

对于非周期函数，T趋于无穷，delta w趋于0，nw变为连续

表示为W的函数，这样就得到了傅里叶变换F(W)

![alt text](image.png)
![alt text](image-1.png)
