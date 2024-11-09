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
- 随着视角的高度变换也会调整cascade 的等级
- 会进行平滑处理？？？