---
title: Ocean waves simulation with Fast Fourier transform
tag:
  - game
  - water
  - ocean
mathjax: true
category:
 - Realtime Water Rendering
sortValue: 10002
---

- [Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- [FFT 和 Oceanographic Spectrum 方法的论文](https://people.computing.clemson.edu/~jtessen/reports/papers_files/coursenotes2004.pdf)
- [Source Code](https://github.com/gasgiant/FFT-Ocean)
- [catlikecoding](https://catlikecoding.com/unity/tutorials/flow/waves/)
- [Ocean Simulation](https://dev.epicgames.com/community/learning/tutorials/qM1o/unreal-engine-ocean-simulation)

这个方法的核心是 FFT 和 Oceanographic Spectrum：
![Alt text](image.png)

这里省去关于 sine wave 的介绍，根据线性波理论，波浪是 Gerstner Wave 的叠加，表面上的每个点沿着一个圆形轨迹运动：
![Alt text](image-1.png)
振幅太大的时候会形成重叠，所以应该设小一点：
![Alt text](image-2.png)
![Alt text](image-3.png)
所以实际上要做的就一件事，就是把这些波叠加起来。比起摆弄参数，更好的方法是使用快速傅里叶变换（FFT）和 Oceanographic Spectrum 使用真实的海洋数据来模拟波浪。

## How Ocean Waves Work in Unreal Engine: FFT & Wave Simulation

https://www.youtube.com/watch?v=OWiyIc2bVwM

![Alt text](image-4.png)

statistical wave models 描述了海洋波浪的统计特性：
![Alt text](image-5.png)

FFT 是一种高效计算 DFT 的方法：
![Alt text](image-6.png)

Cooley-Tukey 算法把 N 个数据点分为奇数和偶数两组，然后递归计算每一半的 DFT，然后组合起来：
![Alt text](image-7.png)

Phillips Spectrum 是一个常用的频谱模型，表示风产生的海洋波浪高度的统计分布表示：
![Alt text](image-8.png)

共轭项表示时域上延两个方向传播的波：
![Alt text](image-9.png)

用 Cascade 表示不同分辨率的模拟：
![Alt text](image-10.png)

![Alt text](image-11.png)
用两个 RT 存储 WPO：
![Alt text](image-12.png)
