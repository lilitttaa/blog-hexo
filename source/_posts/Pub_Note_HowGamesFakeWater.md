---
title: How Games Fake Water
tag:
  - game
  - water
  - ocean
category:
 - Realtime Water Rendering
sortValue: 10001
---

- [How Games Fake Water](https://www.youtube.com/watch?v=PH9q0HNBjT4)

我们先从正弦波开始
![Alt text](image.png)
用$\alpha$表示波的振幅，$\omega$表示频率
为了让波动起来，我们可以引入时间t，和相位常数speed，最终可以表述为：
![Alt text](image-2.png)

我们把不同的波叠加起来，相位相同的波叠加看起来没什么特别：
![Alt text](image-3.png)
如果我们把相位不同的波叠加起来就能得到像海浪一样的效果：
![Alt text](image-4.png)

好了，我们可以在Vertex Shader中对顶点进行置换了：
![Alt text](image-5.png)
![Alt text](image-6.png)