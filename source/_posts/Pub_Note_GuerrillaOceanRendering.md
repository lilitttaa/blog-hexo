---
title: Simulation and Representation of Topology-Changing Rolling Waves for Massive Open Ocean Games
tag:
  - game
  - water
  - ocean
category:
 - Realtime Water Rendering
sortValue: 10003
---

- [Guerrilla, SIGGRAPH 2024](https://dl.acm.org/doi/abs/10.1145/3641233.3664308)
- [Wave particles](http://www.cemyuksel.com/research/waveparticles/waveparticles.pdf)

目标：Rolling Water、Close distance、Interactive、variability、performance、artistic

- 每次海洋的效果总是不一样的，而不是一个静态的 loop 动画一直播
- 艺术家可以控制海浪的 size、shape、以及一些 time 等参数
- 性能要求对纹理 memory 要求尽可能的小，速度快

过去的一些方案包括：
![alt text](image.png)

- Gerstner 波叠加的方法
- FFT 可以让我们在时域和频域上进行变换，做法通常是拿到真实世界海洋的频谱，分解成一系列的正弦波，换算到时域，然后叠加这些波

存在的问题：

- 高频数据不容易模拟和捕获，也对内存消耗比较大。
- 基于高度的 displacement 不能表示 overhang（浪掀起来的效果）

新的方法：

- 从模拟方法中进行捕获
- 表示成 Curve（Spline），存在一张 texture 里边

这种方法的优点：

- 模拟方法中可能会因为 air pockets 导致表面出现空洞，而这种方法可以处理这种情况
- 适合艺术家进行调整和创作
- 允许 overhang
- 16 位的浮点数，256\*256 的纹理存储 displacement 数据
- 用 BC7 RGBA 存储颜色数据和泡沫

![Alt text](image-1.png)
纹理的每一列代表一条曲线，每个像素代表这个曲线中的一个点，像素颜色表示点的位置。

## Wave Particle

- 支持 two-way coupling，即物体对 wave 的影响，以及 wave 对物体的影响。
- 用于局部 wave 引起的流动，不能处理全局的流动
- 可以处理有边界和没有边界的情况

早期的工作包括：

- 参数化表示
- FFT

Tessendorf 方法是 one-way coupling，仅支持物体对 wave 的影响。

以及一些求解 shallow water equation（本身是 Navier-Stokes 方程简化）的方法。
