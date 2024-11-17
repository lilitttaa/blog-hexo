---
title: Insomniac’s Water Rendering System
category:
 - Realtime Water Rendering
sortValue: 10007
---

- [Insomniac’s Water Rendering System](https://www.gamedevs.org/uploads/insomniac-water.pdf)



第一个在游戏中使用带有 disperion 属性的交互式水面

## LOD
![Alt text](image.png)
LOD：bisecting rightangled triangles

添加偶数和奇数LOD没有达到更好的效果，反而使得法线成为一个问题

40级LOD，从一个Grid表示128m，到一个Grid表示1mm

## Ambient Wave

32\*32 FFT

波的方向跟位置相关，速度由disperison决定，振幅由伪随机波谱决定，同时也跟波长有关，波长越长，振幅越大

每个波的时间相关的状态只由它的相位构成存储成一个angle。变化量由time step和angle velocity决定：
![Alt text](image-1.png)
$\omega$代表角速度，$g$代表重力加速度，$k$代表波数，$\sigma$代表表面张力，$\rho$代表水密度。

不用对每个wave component进行更新？？？用表格方法减少开销？？？

这个方法其实就是在频域上更新相位，生成频谱图，然后IFFT生成Complex HeightMap，丢弃虚部。（可以使用Two for one FFT technology避免浪费）

## Interactive Water

