---
title: 上帝视角看GPU（1）：图形流水线基础
category:
 - Game
sortValue: 017
---

- https://www.bilibili.com/video/BV1P44y1V7bu

首先，显示器是如何显示画面的呢？

![Alt text](image.png)

- 它的基本单元叫做像素。每个像素都包含红绿蓝，也就是 RGB 三个分量，通过组合形成各种各样的颜色。
- 每个分量可以用一个数字表示。最常见的是用 8 个二进制位来表示一个分量，也就是一个分量的取值范围是 0-255，数字越大亮度越高。
- 这样的话三个分量组合之后，就能表示 256\*256\*256 = 16,777,216 种颜色。
- 随着内容和显示技术的发展，这几年也出现了每个分量用 10 位、12 位、16 位等更高的位数来表示。它们可以表达更多的细节，或者更大的范围。

我们在这里主要就讲 8 位的情况：

电脑如果要把内容输出到显示器，要做什么呢？
![Alt text](image-1.png)

- 这里的一个概念叫帧缓存，frame buffer。它是内存的一块区域。这块区域里的内容和显示器上显示的每个像素是一一对应的。
- 8 位是一个字节，所以帧缓存里每一个字节表示一个像素的一个分量，连续排列下去。
- 当然，现代的电脑更适合 32 位对齐的处理方式，所以在帧缓存里，每 32 位，也就是 4 个字节来表示一个像素。除了 RGB 占用的 24 位，后面还跟了一个表示透明度的 Alpha。虽然输出到显示器的时候，这个信息会被忽略。

![Alt text](image-2.png)

- 显卡是把帧缓存的内容输出到显示器上的中间设备。它有一个显示输出端口，通往显示器。
- 还有个显示电路，把帧缓存转成显示输出的信号。

![Alt text](image-3.png)

- 注意，这只是个显卡，上面没有计算能力，只是图像的搬运工。

现在需求增加了，我们想把显示的图像亮度翻倍，也就是 RGB 每个分量都乘 2。

当然，我们可以在 CPU 上做这件事情，把每个数字在写入帧缓存之前，先乘个 2。这样必然要占用大量 CPU 资源。

一个更好的方法，是加入一个处理器：

![Alt text](image-4.png)

- 在大量数据上执行同样的操作。
- 如果操作的算法固定，那就只需要设置不同的参数就能达到目的。
- 但如果操作需要灵活多变，就得挂上一个程序才行。

![Alt text](image-5.png)

- 这样的程序就叫做 shader，当然这里处理像素，所以叫 pixel shader。
- 这种可以绑上 shader 的单元叫做可编程流水线单元。
- 每个 pixel shader 仅仅处理一个像素，单入单出。

![Alt text](image-6.png)

- 这里，pixel shader 的输入是一个坐标。它根据坐标从图像上采到颜色，执行操作后返回结果。
- 我们管输入的图像叫做纹理（texture）

![Alt text](image-7.png)

- 如何从坐标到纹理里面采数据，以及如何写入帧缓存，并不需要 shader 来管，有专门的硬件负责。

以上就是一个最初级的针对图像的处理器。
![Alt text](image-8.png)

- 在任天堂红白机上就有这么一个处理器，称为 PPU。

但如果输入不是简单的一张图，而是一个几何网格，该怎么办呢？

![Alt text](image-9.png)

- 这样的几何由一系列简单形状组成，比如点、线、三角形、四边形等。这些简单形状称为图元，primitive。

简化起见我们这里只看三角形，这一堆三角形怎么和像素搭上关系呢？

![Alt text](image-10.png)

- 在到达 pixel shader 之前，有个环节叫做光栅化（rasterize）。把三角形所覆盖区域的像素填上。
- 这是个算法固定的操作，为了效率一般由硬件直接构成，不可编程，这叫做固定流水线单元。

另外，我们如何几何存在前后遮挡关系呢？也就是说，如何确定哪个像素显示，哪个不显示呢？
![Alt text](image-11.png)

- 这就需要在 pixel shader 之后加一个环节，叫做 output merger。里面会做深度判断，根据规则来决定哪个像素最后能存活下来。
- 这也是固定流水线单元。

还没完，几何是怎么存储的呢？

![Alt text](image-12.png)

- 首先有一堆的点，分布于空间里。这些点叫做顶点 vertex，用(x, y, z)坐标表示。以及方向、颜色、纹理坐标等信息。
- 存放顶点的 buffer 叫做 vertex buffer。

![Alt text](image-13.png)

- 存放它们连线的 buffer 叫做 index buffer。每个单元是一个整数，表示 vertex 的索引。

在开始光栅化之前，还有一个固定流水线环节，叫做 primitive assembler：
![Alt text](image-16.png)
![Alt text](image-14.png)
![Alt text](image-15.png)

- 把 vertex 们组装成一个一个三角形
- 把屏幕之外的裁剪掉
- 计算三条边的方程
- ……

然后才送入 rasterizer。

除此之外，还需要考虑一点：

- 同一个几何体，我们可以放到不同位置，从不同角度看，或者调整摄像机的焦距，它在屏幕上就该看起来不一样。
- 也就是说每个 vertex buffer 中的顶点都需要做一个变换，把它从几何自己的空间变换到屏幕的空间。
  ![Alt text](image-17.png)
- 这里用到三个不同的变换，每个变换用一个 4x4 的矩阵来表示。
- 也就是俗称的 MVP 变换，Model-View-Projection。
- M 将模型从模型空间变换到世界空间，用来决定物体在空间中的位置、朝向、放缩等。
- V 将世界空间变换到摄像机空间。
- P 将摄像机空间变换到裁剪空间，主要用来调整摄像机的参数，比如视野宽窄，视域远近范围。

同样，如果只是固定不变的算法操作，可以用固定的硬件和可设置的参数来完成。早期的 GPU 确实也是这么做的，比如 2000 年的 GeForce 256。
![alt text](image-18.png)
![alt text](image-19.png)

- 这一步被称为硬件变换和光照，T&L

但为了灵活性的需求，很快就进化到可编程的方式
![alt text](image-20.png)

- 用一个 shader 对每个顶点过一遍。这就是 vertex shader。
- 顶点的信息，可以根据需要用不同格式来保存。

![alt text](image-21.png)

- Vertex shader 并不需要知道顶点存储的格式。所以这里需要调用者提供一个顶点格式的描述。

![alt text](image-22.png)

- 由一个固定流水线单元 input assembler 从 vertex buffer 里组装出一个顶点，送给 vertex shader 处理。
- 每个 vertex shader 仅仅处理一个 vertex，单入单出。

至此，我们已经得到一条基本的图形流水线。这样的流水线，就是 2000 年代主流 GPU 的构成。

GPU（Graphics Processing Unit）把 CPU 从图形渲染过程中简单但大数据量的操作中解放出来。

另外不管是 vertex shader 还是 pixel shader，它们的程序功能集中，只处理一个硬件塞过来的数据单元，返回处理后的结果。

不需要考虑具体如何从内存读取这些数据，以及处理完怎么写出去。前后都有别的单元来处理。换句话说，shader 就像个回调函数，GPU 在需要的时候在大量数据上对每个单元调用一下这个回调函数。

![alt text](image-23.png)

- 也可以看到 CPU 与 GPU 的不同，CPU 擅长在小数据上做相对复杂的串行计算和逻辑控制，而 GPU 擅长在大量数据上做相对简单的并行计算。
