---
titile: Multi-resolution water rendering in crest
category:
 - Realtime Water Rendering
sortValue: 10006
---

- [Crest, Siggraph 2019](https://advances.realtimerendering.com/s2019/index.htm)
- [Crest, Siggraph 2017](https://advances.realtimerendering.com/s2017/index.html)

![Alt text](image.png)

- 多级纹理使用 Texture Array
- 每行代表不同的分辨率，是上一行表示范围的两倍，最下面是最大的一张，代表 8km 宽的范围
- 从左到右依次是：
  - 水的深度图（黑色的是岛屿，水的深度为 0）
  - 带有 Flow 信息的输入图，可以表示漩涡
  - Displacement Map 定义了水表面的形状
  - Foam Map 从 Displacement Map 生成，用来添加 Foam 纹理
  - Shadow，添加软阴影
- 这里边的每张图都是动态算出来的，不是 bake 或者手动制作的

![Alt text](image-1.png)

- 基于 Gerstner Wave 和 Ripple simulation
- 这个框架也可以加上 FFT 和 SWE

![Alt text](image-2.png)

- 盗贼之海直接采样 FFT 纹理
- 育碧 2012 把 shape 写到了一张屏幕空间 buffer 上，这对于性能来说很友好（但是由于屏幕空间的限制就没法做动态水模拟了，因为没法保存屏幕外的状态）
- crest 采用写到世界空间 displacement map 上
- Insomniac for Resistance 2 用了多级 LOD 的 FFT 生成数据，然后 displacement map 由 FFT 生成
- Uncharted4 用 Wave Particles 生成

![Alt text](image-3.png)

- 没有使用 tessllation

![Alt text](image-4.png)

- 随着视角的高度变换也会调整 cascade 的等级
- 会进行平滑处理？？？

![Alt text](image-5.png)

- 通过纹理来产生 input，这里是一个 smoothstep 计算的高度
- foam 和动态波可以以 input 的形式添加

![Alt text](image-6.png)

![Alt text](image-7.png)

- 每个 gerstner wave 都是一个 wave component
- 自动将 gerstner wave 分配到特定的 cascade 中。
- 关于分配的计算，需要手动指定一个常量表示每个 wavelength 做多少次采样，默认是 3-6 次。

![Alt text](image-8.png)

- 使用一个 combine pass 从下到上叠加上去
- 这样可以避免走样
- combine pass 采用 biliner filter

![Alt text](image-9.png)

- 会导致平滑的效果（或者问题），cascade 之间的采样位置存在一个 offset

![Alt text](image-10.png)

目前把 displacement map 通过 async read back 到 cpu 用于物理和 gameplay。
之后考虑 cpu 传入查询点然后在 compute shader 里边计算，这样就不要回读整张纹理了。

https://www.youtube.com/watch?v=rkN-CMtafCg

- 这是一个动态交互的效果 demo
- 船划过后会留下涟漪，然后涟漪扩散开然后消失
- 这里为了美学上的考量，把重力设置为 16g

![Alt text](image-11.png)

- 红色表示高度、绿色表示速度
- 边缘为 0 为了避免玩家移动导致边缘出现一些错误的效果
- 每个 simulation 都是由一个单独的 compute shader dispatch 完成，然后在之前 displacement combine 的 pass 里边叠加上去

![Alt text](image-12.png)

- Dispersion 是指大 wavelength 的 wave 比小 wavelength 的 wave 传播的更快
- 这里简单的为每个 cascade 设置了一个 wave speed，同一个 cascade 里边的 wave speed 是一样的
- 还有其他的一些方法：
  - laplacians kernels（复杂）
  - 在频域中进行仿真（开销和复杂）

![Alt text](image-13.png)

- 按照 CFL 条件，模拟要保持稳定，时间步长需要满足一定条件
