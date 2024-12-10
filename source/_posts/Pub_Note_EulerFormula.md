---
title: Euler's Formula
tag:
  - math
mathjax: true
category:
 - Mathematics
sortValue: 110001
---

- [e^(iπ) in 3.14 minutes, using dynamics | DE5](https://www.youtube.com/watch?v=v0YEaeIClKY)

$e^x$ 具有一个非常特殊的属性，即它的导数等于它自己。它是具有这个属性的唯一的函数。
![Alt text](image.png)

如果我们把$e^t$看作是描述时间轴上位置的函数，那么速度（位置的导数）总是等于位置。
![Alt text](image-1.png)
离 0 越远，速度就越快：
![Alt text](image-2.png)

如果我们给这个指数加上一个常数，比如 2：
![Alt text](image-3.png)
![Alt text](image-4.png)
现在速度始终是位置的两倍的方式移动。

但是，如果这个常数是 i，即负 1 的平方根呢？

我们把整个坐标系看作是在复数平面上，复数表示为 a+bi，a 作为横坐标，b 作为纵坐标。所以有：
$z = a + bi \Rightarrow z i = a i + b i^2 = -b + ai$
可以看到乘上一个 i 实际上就是逆时针旋转 90 度：
![Alt text](image-5.png)
对于任意位置都能得到一个 90 度旋转的速度：
![Alt text](image-6.png)

初始条件，t 为 0 的时候，$e^{it}$将为 1，也就是实数部分为为 1，虚数部分为 0。
![Alt text](image-7.png)

然后就可以发现这实际上是在做一个速度为 1，半径为 1 的圆周运动。
![Alt text](image-8.png)
![Alt text](image-9.png)

最后我们可以得到：
$e^{it} = \cos t + i \sin t$

还有一点遗留的问题，将 i 放到指数中是否是合理的？不过这里就不再讨论了。
