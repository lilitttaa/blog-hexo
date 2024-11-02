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

- [FFT 和 Oceanographic Spectrum 方法的论文](https://people.computing.clemson.edu/~jtessen/reports/papers_files/coursenotes2004.pdf)
- [Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- [Source Code](https://github.com/gasgiant/FFT-Ocean)
- [catlikecoding](https://catlikecoding.com/unity/tutorials/flow/waves/)

这个方法的核心是 FFT 和 Oceanographic Spectrum：
![Alt text](image.png)

这里省去关于 sine wave 的介绍，根据线性波理论，波浪是 Gerstner Wave 的叠加，表面上的每个点沿着一个圆形轨迹运动：
![Alt text](image-1.png)
振幅太大的时候会形成重叠，所以应该设小一点：
![Alt text](image-2.png)
![Alt text](image-3.png)
所以实际上要做的就一件事，就是把这些波叠加起来。比起摆弄参数，更好的方法是使用快速傅里叶变换（FFT）和 Oceanographic Spectrum 使用真实的海洋数据来模拟波浪。
