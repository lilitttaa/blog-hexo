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

## Talks

- [Guerrilla, SIGGRAPH 2024](https://dl.acm.org/doi/abs/10.1145/3641233.3664308)
- [Diablo IV, GDC 2024](https://gdcvault.com/play/1034779/Technical-Artist-Summit-H2O-in)
- [Ubisoft, GDC 2023](https://www.bilibili.com/video/BV1Ux4y1X7Xe)
- [腾讯光子, GDC 2023](https://gdcvault.com/play/1028829/Advanced-Graphics-Summit-Open-World)
- [Guerrilla, SIGGRAPH 2022](https://advances.realtimerendering.com/s2022/SIGGRAPH2022-Advances-Water-Malan.pdf)
- [Far Cry 6, GDC 2022](https://gdcvault.com/play/1027675/Simulating-Tropical-Weather-in-Far)
- [The Last of Us Part II, GDC 2021](https://gdcvault.com/play/1027356/Enhancement-of-Particle-Simulation-Using)
- [The Last of Us Part II, GDC 2021](https://gdcvault.com/play/1027370/Creative-and-Experimental-VFX-in)
- [Ubisoft, Siggraph 2020](https://www.youtube.com/watch?v=9qIgA2H90o0)
- [Atlas, GDC 2019](https://gdcvault.com/play/1025819/Advanced-Graphics-Techniques-Tutorial-Wakes)
- [Crest, Siggraph 2019](https://advances.realtimerendering.com/s2019/index.htm)
- [Unreal Engine, GDC 2019](https://gdcvault.com/play/1026262/Technical-Artist-Bootcamp-Distance-Fields)
- [Sea of Thieves, GDC 2018](https://www.youtube.com/watch?v=y9BOz2dFZzs)
- [Call of Duty WWII, GDC 2018](https://www.bilibili.com/video/BV1j4411A7zo)
- [Far Cry 5, GDC 2018](https://gdcvault.com/play/1025555/Advanced-Graphics-Techniques-Tutorial-Water)
- [Crest, Siggraph 2017](https://advances.realtimerendering.com/s2017/index.html)
- [Uncharted 4, Siggraph 2016](https://advances.realtimerendering.com/s2016/)
- [Ubisoft, 2012](https://www.fxguide.com/fxfeatured/assassins-creed-iii-the-tech-behind-or-beneath-the-action/)
- [Killzone 3, 2011](https://www.sidefx.com/community/guerrilla-games-killzone-3/)
- [Portal2, Siggraph 2010](https://advances.realtimerendering.com/s2010/index.html)

其中：

- Guerrilla 2024 海岸的 Rolling Mesh 使用 2d 曲线表示，使用 houdini 生成。渲染时使用了参考于 Wave Particles 的内部方法。他们用的是一种混合的方法，不过没有提到常规的渲染方法，他们的前作 Killzone 3 参考了 Tessendorf 的工作，所以猜测可能是 FFT。
- 腾讯光子 2023 使用 SWE 进行实时模拟和离线生成 Flow Map
- Ubisoft 2023 使用 FFT 表示海洋，FBM 表示河流，海岸的 Rolling Mesh 使用 2d 曲线表示
- Far Cry 6 2022 使用 FBM
- COD 2018 基于 Houdini offline bake 的方法
- Uncharted 4 2016 使用 Flow Map + Wave Particles
- Portal2 2010 使用 Flow Map

## Resouces

- 介绍了 Sine 和 Gerstner 波的实现方式，添加了噪声图、菲涅尔反射：[How Games Fake Water](https://www.youtube.com/watch?v=PH9q0HNBjT4)
- 从波的实现开始，然后到 FFT，各种数学公式，以及后续的具体的实现，有 Github 参考：[Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- 浅墨的整理，很细致，很值得一看：[水体渲染技术发展史](https://github.com/QianMo/Game-Programmer-Study-Notes/tree/master/Content/%E7%9C%9F%E5%AE%9E%E6%84%9F%E6%B0%B4%E4%BD%93%E6%B8%B2%E6%9F%93%E6%8A%80%E6%9C%AF%E6%80%BB%E7%BB%93)
- Shadertoy 上一个用分形实现的海洋渲染：[Shadertoy Seascape](https://www.shadertoy.com/view/Ms2SD1)
- [How Ocean Waves Work in Unreal Engine: FFT & Wave Simulation](https://www.youtube.com/watch?v=OWiyIc2bVwM)
- [A deep dive into my process of creating this animated stylized ocean in UE](https://www.youtube.com/watch?v=UWGwq-_w08c)
- [#notGDC 2023 - FFT Ocean Flipbook : How to create & sample one using Blender & UE](https://www.youtube.com/watch?v=rV6TJ7YDJY8)
- [怎么才能边做游戏边划水: 基于浅水方程的水面交互](https://zhuanlan.zhihu.com/p/649003961)
- SWE 方程的推导：[Games103 Surface Waves](https://www.bilibili.com/video/BV12Q4y1S73g)
- [Realtime GPGPU FFT Ocean Water Simulation](https://d-nb.info/1143691342/34)
- [游戏中的实时水体模拟技术](https://zhuanlan.zhihu.com/p/21573239)
- UE 插件：[Waterline PRO](https://www.fab.com/listings/0c1fc983-db84-4df3-b623-03db76d552c6)
- 游戏中的水渲染发展：[The Evolution of Water Effects In Video Games](https://www.youtube.com/watch?v=JW9UZeTnVhk)

## Courses

- 关于 Sine 和 Gerstner 波的实现：[GPU Gems: Chapter 1. Effective Water Simulation from Physical Models](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)

## Comparison

- [Horizon Forbidden West - Incredible Water Physics and Details](https://www.youtube.com/watch?v=M3Lbyn-c7Hw)
- [Sea of Thieves Has Some Gorgeous Water Shots](https://www.youtube.com/watch?v=aGogFt4bhTM)
- [Skull and Bones: Creating the Ocean](https://www.youtube.com/watch?v=JiZ4hFgE5tE)
- [Uncharted 4 Water effect](https://www.youtube.com/watch?v=FFaXXzcr8Mc)
- [Far Cry 6 Amazing Water Physics and Stunning Graphics](https://www.youtube.com/watch?v=9d9V9jjTh3w)

## Papers

- [A survey of ocean simulation and rendering techniques in computer graphics 2011](https://arxiv.org/pdf/1109.6494)
- [Wave particles 2007](http://www.cemyuksel.com/research/waveparticles/waveparticles.pdf)
- [Ocean Surface Generation and Rendering 2018](https://publik.tuwien.ac.at/files/publik_272334.pdf)
- [Real-time Animation and Rendering of Ocean Whitecaps](https://inria.hal.science/hal-00967078/file/Whitecaps-presentation.pdf)

## Questions

怎么表示浮动的效果，比如说船在水上起伏？

- 这个问题叫做 two-way coupling，有很多论文有对此的讨论，Wave Particles 和 SWE 都支持这一点。
- 因为物理是在 CPU 上的，所以要么在 CPU 上算，要么在 GPU 上算完回读到 CPU。可以使用异步回读，延迟个几帧。
- Ubisoft 2012 有相关讨论

怎么做交互？

- shallow water equation
- wave particles???

白沫怎么表示？

- 白沫的方案有很多种：
  - 认为 Gerstner 波最尖锐的部分会产生白沫，可以用雅可比绝对值计算，小于某个阈值就是白沫。

LOD 怎么处理？

- CDLOD
- water virtual texture

Tile 的重复怎么处理？

- 多级 FFT 贴图

音频怎么处理？

## A survey of ocean simulation and rendering techniques in computer graphics

- We will see that this type of methods essentially allows the simulation of ocean scenes in the deep water domain, without breaking waves.

- physically-based methods use Navier-Stokes Equations (NSE) to represent breaking waves and more generally ocean surface near the shore

- In deep waters, the free surface defined by the interface between air and water is generally subjected to a large oscillatory behavior, whereas in shallow waters waves break near the shore.
- parametric description, spectral description as well as models from Computational Fluid Dynamics (CFD) and more specifically Navier-Stokes Equations (NSE).

分为深水区和浅水区，深水区主要表现为一种震荡行为，通常使用参数和频谱描述。而浅水区存在 wave break，使用 NSE 模拟。

参数和频谱描述来自于海洋学，其中参数方法来自于浮标的测量，频谱描述则是对海洋波动的统计描述。

而 NSE 则是一种物理方法，可以处理各种流体，尤其是处理动态行为

光学上，浮游植物会影响海洋颜色，以及其他的泡沫、喷雾，以及光与水的相互作用。

深水区的方法主要分为三类：

- 时域方法：使用高度图作为周期函数叠加的结果
- 谱域方法：使用波谱描述海洋表面的谱域，然后使用 FT 获取时域的变换
- 混合方法
