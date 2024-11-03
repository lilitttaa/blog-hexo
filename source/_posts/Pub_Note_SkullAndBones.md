---
title: Skull and Bones Water Rendering
category:
 - Realtime Water Rendering
sortValue: 10004
---

- [Advancements in Water and Procedural Technology | Ubisoft [SG]](https://www.youtube.com/watch?v=9qIgA2H90o0)
- [Making Waves for 'Skull and Bones': Advancements in Water Tech](https://www.bilibili.com/video/BV1Ux4y1X7Xe)

## Skull and Bones

河流跟海洋不一样，做法不同
育碧做了整合

FFT 处理海洋
FBM（分形噪声）处理河流

用不同的参数集控制不同的形态

四叉树控制 LOD
![Alt text](image-4.png)
![Alt text](image-3.png)

2d curve 控制海岸的 rolling mesh deformation，如果算是时间就是 3d
![Alt text](image-2.png)

船的 mass 影响物理交互
![Alt text](image-1.png)

shading 部分包括：
![Alt text](image.png)
