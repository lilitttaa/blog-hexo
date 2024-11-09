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
- 512x512x8
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
