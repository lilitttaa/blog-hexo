---
title: Simulation and Representation of Topology-Changing Rolling Waves for Massive Open Ocean Games
tag:
  - game
  - water
  - ocean
category:
 - Realtime Water Rendering
sortValue: 10002
---

- 每次海洋的效果总是不一样的，而不是一个静态的 loop 动画一直播
- 艺术家可以控制海浪的 size、shape、以及一些 time 等参数
- 性能要求对纹理 memory 要求尽可能的小，速度快

过去的一些方案包括：
![alt text](image.png)

- Gerstner 波是一种波叠加的方法
- FFT 可以让我们在时域和频域上进行变换，做法通常是拿到真实世界海洋的频谱，分解成一系列的正弦波，换算到时域，然后叠加这些波
