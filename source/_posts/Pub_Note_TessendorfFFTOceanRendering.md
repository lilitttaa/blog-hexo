---
title: Tessendorf's FFT Ocean Rendering
mathjax: true
category:
 - Realtime Water Rendering
sortValue: 10008
---

## 表面波动的数学解

不可压缩的 NS 方程

海洋表面的运动源于重力这样的保守力，所以可以对运动做一些约束，可以把四个 NS 方程简化为两个方程。其中一个方程叫做伯努利方程（bernoulli equation）。

伯努利方程可以模拟各种动态，从 wave breaking 到不同深度的水体

根据海洋的特征还可以进一步做出两个约束：

1. 线性化伯努利方程，这样表面波的运动不会非常的激烈
2. 仅评估水面而不是考虑整个水体，把整个 3d 的位置描述成一个 2d 的高度场描述

## Dispersion Relation

表面波方程是线性微分方程，通解需要用特殊解来获得。带入特殊解后，可以得到时间频率$\omega$和波数$k$之间的关系，这个关系叫做 dispersion relation：
$$ \omega = \pm \sqrt{gk} $$

ns 方程没有对 h 有任何表述，我们可以表面高度和垂直速度施加一个初始条件。

对于振幅，我们通过开放海洋的测量数据来进行统计学的随机生成。

## Ocean Algorithm

- Linear Wave/ Gravity Wave
- Capillary Wave

### Gerstner Wave

$$ \vec{x} = \vec{x_0} + (\vec{k}/k) \cdot A \cdot sin(\vec{k} \cdot \vec{x_0} - \omega \cdot t) $$
$$ \vec{y} = A \cdot cos(\vec{k} \cdot \vec{x_0} - \omega \cdot t) $$

其中：

- $\vec{k}$表示 wave vector，它的方向表示波的传播方向，它的长度 k 叫做 wave number 跟波长成反比

kA < 1 且接近1时，波的顶端会变得尖锐，kA > 1 时，会出现环

### Disperion Relation

对于深水不考虑深度：
$$ \omega = \sqrt{gk} $$

对于浅水：
$$ \omega = \sqrt{gk tanh(kD)} $$

对于非常小的波Dispersion Relation被表面张力所影响：
$$ \omega = \sqrt{gk(1+k^2L^2)} $$
其中 L 是长度单位，它决定了表面张力的影响尺度

我们需要让波能够循环（也就是算出来的displacement map能够无缝的拼接起来）