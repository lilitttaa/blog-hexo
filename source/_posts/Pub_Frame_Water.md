---
title: The roadmap of realtime water rendering
tag:
- roadmap
- game
- water
- ocean
category:
 - Realtime Water Rendering
sortValue: 10000
---

- 介绍了 Sine 和 Gerstner 波的实现方式，添加了噪声图、菲涅尔反射[How Games Fake Water](https://www.youtube.com/watch?v=PH9q0HNBjT4)
- 从波的实现开始，然后到 FFT，各种数学公式，以及后续的具体的实现有 GIthub 参考：[Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- [How Ocean Waves Work in Unreal Engine: FFT & Wave Simulation](https://www.youtube.com/watch?v=OWiyIc2bVwM)
- 这篇硕士论文主要谈论怎么把光谱模型给结合起来，用于不同距离下进行观察。[Ocean Surface Generation and Rendering](https://publik.tuwien.ac.at/files/publik_272334.pdf)
- [水体渲染技术发展史](https://github.com/QianMo/Game-Programmer-Study-Notes/tree/master/Content/%E7%9C%9F%E5%AE%9E%E6%84%9F%E6%B0%B4%E4%BD%93%E6%B8%B2%E6%9F%93%E6%8A%80%E6%9C%AF%E6%80%BB%E7%BB%93)
- [Rendering Water in Horizon Forbidden West](https://advances.realtimerendering.com/s2022/SIGGRAPH2022-Advances-Water-Malan.pdf)
- [Simulation and Representation of Topology-Changing Rolling Waves for Massive Open Ocean Games](https://dl.acm.org/doi/abs/10.1145/3641233.3664308)
- [Shadertoy Seascape](https://www.shadertoy.com/view/Ms2SD1)
- [A deep dive into my process of creating this animated stylized ocean in UE](https://www.youtube.com/watch?v=UWGwq-_w08c)
- [The Technical Art of Sea of Thieves ](https://www.youtube.com/watch?v=y9BOz2dFZzs)
- [怎么才能边做游戏边划水: 基于浅水方程的水面交互](https://zhuanlan.zhihu.com/p/649003961)
- [游戏中的实时水体模拟技术](https://zhuanlan.zhihu.com/p/21573239)
- [Crest: Novel Ocean Rendering Techniques in an Open Source Framework](https://advances.realtimerendering.com/s2017/index.html)
- [Multi-resolution Ocean Rendering in Crest Ocean System](https://advances.realtimerendering.com/s2019/index.htm)

目标：Rolling Water、Close distance、Interactive、variability、performance、artistic

- 每次海洋的效果总是不一样的，而不是一个静态的 loop 动画一直播
- 艺术家可以控制海浪的 size、shape、以及一些 time 等参数
- 性能要求对纹理 memory 要求尽可能的小，速度快

过去的一些方案包括：
![alt text](image.png)

- Gerstner 波是一种波叠加的方法
- FFT 可以让我们在时域和频域上进行变换，做法通常是拿到真实世界海洋的频谱，分解成一系列的正弦波，换算到时域，然后叠加这些波

## reference

## TODO

- [x] 看几个概述性的视频，了解一下一些基本的概念
- [ ] 理清前两个视频的具体内容，整理一下，弄清里边的数学公式
- [ ] 看 ShaderToy 代码
  - [ ] 理清代码
  - [ ] 实现正弦波形图
  - [ ] 实现 Gerstner 波
- [ ] 理清 Siggraph 这篇论文
- [ ] 看 Houdini
- [ ] 用 ShaderToy 实现
- [ ] 看 UE 实现教程
- [ ] 用 UE 实现
- [ ] 找找其他的资料，整理其他方案，做可交互水
  - [ ] 盗贼之海
  - [ ] 交互水

