---
title: The Roadmap of Realtime Water Rendering
tag:
  - roadmap
  - game
  - water
  - ocean
category:
 - Realtime Water Rendering
sortValue: 10000
---

- 介绍了 Sine 和 Gerstner 波的实现方式，添加了噪声图、菲涅尔反射：[How Games Fake Water](https://www.youtube.com/watch?v=PH9q0HNBjT4)
- 从波的实现开始，然后到 FFT，各种数学公式，以及后续的具体的实现，有 Github 参考：[Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- 这篇硕士论文主要谈论在不同距离下进行观察怎么把不同光谱模型给结合起来。[Ocean Surface Generation and Rendering](https://publik.tuwien.ac.at/files/publik_272334.pdf)
- 浅墨的整理，很细致，很值得一看：[水体渲染技术发展史](https://github.com/QianMo/Game-Programmer-Study-Notes/tree/master/Content/%E7%9C%9F%E5%AE%9E%E6%84%9F%E6%B0%B4%E4%BD%93%E6%B8%B2%E6%9F%93%E6%8A%80%E6%9C%AF%E6%80%BB%E7%BB%93)
- 神海 4 的水体渲染：[Rendering Rapids in Uncharted 4](https://advances.realtimerendering.com/s2016/)
- Guerrilla 上一个关于水的 Talk：[Rendering Water in Horizon Forbidden West](https://advances.realtimerendering.com/s2022/SIGGRAPH2022-Advances-Water-Malan.pdf)
- Guerrilla2024 最新的 Talk：[Simulation and Representation of Topology-Changing Rolling Waves for Massive Open Ocean Games](https://dl.acm.org/doi/abs/10.1145/3641233.3664308)
- 育碧 2020Siggraph：[Advancements in Water and Procedural Technology | Ubisoft [SG]](https://www.youtube.com/watch?v=9qIgA2H90o0)
- Shadertoy 上一个用分形实现的海洋渲染：[Shadertoy Seascape](https://www.shadertoy.com/view/Ms2SD1)
- [How Ocean Waves Work in Unreal Engine: FFT & Wave Simulation](https://www.youtube.com/watch?v=OWiyIc2bVwM)
- [A deep dive into my process of creating this animated stylized ocean in UE](https://www.youtube.com/watch?v=UWGwq-_w08c)
- 盗贼之海的 TA 分享：[The Technical Art of Sea of Thieves ](https://www.youtube.com/watch?v=y9BOz2dFZzs)
- [怎么才能边做游戏边划水: 基于浅水方程的水面交互](https://zhuanlan.zhihu.com/p/649003961)
- [游戏中的实时水体模拟技术](https://zhuanlan.zhihu.com/p/21573239)
- Unity 的一个很强的水插件，它们 2017 和 2019 都上了 Siggraph，水体的交互性很惊艳：
  - [Crest: Novel Ocean Rendering Techniques in an Open Source Framework](https://advances.realtimerendering.com/s2017/index.html)
  - [Multi-resolution Ocean Rendering in Crest Ocean System](https://advances.realtimerendering.com/s2019/index.htm)
- 2024 GDC
  - [Technical Artist Summit: H2O in H3LL: The Various Forms of Water in ‘Diablo IV’](https://gdcvault.com/play/1034779/Technical-Artist-Summit-H2O-in)
- 2023 GDC
  - [Advanced Graphics Summit: Open-World Water Rendering and Real-Time Simulation](https://gdcvault.com/play/1028829/Advanced-Graphics-Summit-Open-World)
  - [Making Waves for 'Skull and Bones': Advancements in Water Tech](https://www.bilibili.com/video/BV1Ux4y1X7Xe)
- 2022 GDC
  - [Simulating Tropical Weather in ‘Far Cry 6’](https://gdcvault.com/play/1027675/Simulating-Tropical-Weather-in-Far)
- 2021 GDC
  - [Enhancement of Particle Simulation Using Screen Space Techniques in 'The Last of Us Part II'](https://gdcvault.com/play/1027356/Enhancement-of-Particle-Simulation-Using)
  - [Creative and Experimental VFX in 'The Last of Us Part II'](https://gdcvault.com/play/1027370/Creative-and-Experimental-VFX-in)
- 2019 GDC
  - [Advanced Graphics Techniques Tutorial: Wakes, Explosions and Lighting: Interactive Water Simulation in 'Atlas'](https://gdcvault.com/play/1025819/Advanced-Graphics-Techniques-Tutorial-Wakes)
  - [Technical Artist Bootcamp: Distance Fields and Shader Simulation Tricks](https://gdcvault.com/play/1026262/Technical-Artist-Bootcamp-Distance-Fields)
- 2018 GDC
  - [Advanced Graphics Techniques Tutorial: Water Rendering in 'Far Cry 5'](https://gdcvault.com/play/1025555/Advanced-Graphics-Techniques-Tutorial-Water)

插件：

- [Waterline PRO](https://www.fab.com/listings/0c1fc983-db84-4df3-b623-03db76d552c6)

游戏中的水渲染：

- [The Evolution of Water Effects In Video Games](https://www.youtube.com/watch?v=JW9UZeTnVhk)

## 分为三个部分

- Shape
- Shading
- Interaction

## 一些问题

- 怎么表示浮动的效果，比如说船在水上起伏？
- 小波的渲染怎么表示？
- 怎么做交互？
- Whitecaps 怎么表示？

## TODO

- [x] 看几个概述性的视频，了解一下一些基本的概念
- [ ] 理清前两个视频的具体内容，整理一下，弄清里边的数学公式
- [ ] 看 ShaderToy 代码
  - [ ] 理清代码
  - [ ] 实现正弦波形图
  - [ ] 实现 Gerstner 波
- [ ] 看论文
  - [ ] 硕士论文和浅墨的整理弄清楚 Relative Work
  - [ ] Guerilla, 2024
  - [ ] Ubisoft, 2023
  - [ ] 腾讯光子，2023
  - [ ] Far Cry 6, 2022
  - [ ] The Last of Us Part II, 2021
  - [ ] Atlas, 2019
  - [ ] Sea of Thieves, 2018
  - [ ] Uncharted 4, 2016
- [ ] 看 Houdini
- [ ] 用 ShaderToy 实现
- [ ] 看 UE 实现教程
- [ ] 用 UE 实现
- [ ] 找找其他的资料，整理其他方案，做可交互水
  - [ ] 盗贼之海
  - [ ] 交互水

## 推荐阅读

- 关于 Sine 和 Gerstner 波的实现：[GPU Gems: Chapter 1. Effective Water Simulation from Physical Models](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)
