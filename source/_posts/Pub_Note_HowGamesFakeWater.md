---
title: How Games Fake Water
tag:
  - game
  - water
  - ocean
mathjax: true
category:
 - Realtime Water Rendering
sortValue: 10001
---

- [How Games Fake Water](https://www.youtube.com/watch?v=PH9q0HNBjT4)
- [GPU Gems: Chapter 1.Effective Water Simulation from Physical Models](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)

我们先从正弦波开始
![Alt text](image.png)
用$\alpha$表示波的振幅，$\omega$表示频率
为了让波动起来，我们可以引入时间 t，和相位常数 speed，最终可以表述为：
![Alt text](image-2.png)

我们把不同的波叠加起来，相位相同的波叠加看起来没什么特别：
![Alt text](image-3.png)
如果我们把相位不同的波叠加起来就能得到像海浪一样的效果：
![Alt text](image-4.png)

好了，我们可以在 Vertex Shader 中对顶点进行置换了：
![Alt text](image-5.png)
![Alt text](image-6.png)

在着色时，我们需要法线来计算光照：
![Alt text](image-7.png)
为了计算法线，需要先计算 x-axis 上的切线和 z-axis 上的切线，然后通过叉乘得到法线（这里 xoz 表示水平面）：
![Alt text](image-8.png)
而这两个切线可以利用 Central Difference 来计算：
![Alt text](image-9.png)
不过这种方法的准确性受到距离选取的影响，另外也需要对周围像素做计算。
水面是使用数学定义的，更好的办法是采用微积分，直接计算偏导数。
![Alt text](image-10.png)
![Alt text](image-11.png)
我们可以选择定向波或者圆形波：
![Alt text](image-14.png)
其中定向波每个顶点的方向都是一样的，而圆形波需要计算每个顶点的方向。对于大片水域通常定向波是首选，而对于小片水域圆形波更适合。

现实中的波浪会有一些更尖锐的峰值：
![Alt text](image-12.png)
我们可以把 sine 函数的叠加改为其他函数，比如 Gerstner Wave：
![Alt text](image-13.png)
或者是一个简单的 sine 函数变体：$e^{sin(x)-1}$：
![Alt text](image-15.png)
这样就能得到这样的一个效果：
![Alt text](image-16.png)

然后可以使用白噪声来添加更多的细节：
![Alt text](image-17.png)
根据 noise 选择不同的振幅和频率：
![Alt text](image-18.png)
在大浪的基础上，添加随机小浪：
![Alt text](image-19.png)
![Alt text](image-20.png)

另外，如果我们添加前一个波的导数（domain warp），就可以产生波与波之间相互推动的效果。
![Alt text](image-26.png)

加上天空和太阳高光：
![Alt text](image-21.png)

最后我们再添加反射效果，通常有三种方法：

- 光追
- 屏幕空间反射 SSR（屏幕外的物体会没有反射）
- 环境贴图

这里采用 Cubemap 的环境贴图：
![Alt text](image-22.png)

考虑上菲涅尔项：
![Alt text](image-23.png)
![Alt text](image-24.png)

不过糟糕的是我们使用 Noise 添加细节，平铺的 Noise 贴图会产生这样的效果：
![Alt text](image-27.png)