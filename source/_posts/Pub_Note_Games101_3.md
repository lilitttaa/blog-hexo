---
title: Games101 3.Transformation
category:
 - Games101
sortValue: 30002
---

## Why study transformation?

涉及到两种不同的变换：

- modeling：模型变换
- viewing：视图变换

### Modeling

变换表示位移：
![Alt text](image.png)

- 在这么一个静态场景中，相机以一种平滑的曲线移动

变换表示各种各样复杂的旋转：
![Alt text](image-1.png)

- 跳舞的机器人通过的各个关节的运动形成舞蹈，这部分涉及到计算机动画。
- 另外，还有一种运动，比如拿着机器人的手往空中放，然后问这个手的指尖停在哪里，那么它的关节应该怎么弯曲才能使得手移动到这里来，这部分涉及到逆运动学方向的知识。

变换表示缩放：
![Alt text](image-2.png)

- 字母 i 被压扁，然后变胖，最终完全被压扁
- 动画都是由各种各样的变化合成的。

### Viewing

不只是动画，光栅化成像里面也是大量涉及到变换：
![Alt text](image-3.png)

- 从三维到二维的投影变换

## 2D transformations

### Scale

首先是缩放变换：
![Alt text](image-4.png)

要把左边的图片变小，横轴和纵轴都缩小 0.5，表示为矩阵的形式：
![Alt text](image-5.png)

我们把这个对角矩阵称为缩放矩阵。

如果 x 和 y 方向的缩放比例不同，比如 x 方向缩小 0.5，y 方向不变，那么我们可以写成这样的矩阵形式：
![Alt text](image-6.png)

### Reflection

反射变换：

![Alt text](image-7.png)

### Shear

切变：
![Alt text](image-8.png)

- 切变可以理解为拽着图像的一条边，然后拉动，这样图像就会变形。
- 首先，我们这里垂直方向上没有发生任何变化，所以矩阵的第二行是 `[0, 1]`。
- 在水平方向上，底下这条边没有发生变化，最上边的边向右移动了 a 个单位，简单列一下方程就能得到$x' = x + ay$。
- 我们可以在里边找到一个规律：要想写出一个变换，我们需要找到变化之前的 xy 和变化之后的 x' y'之间的关系。

### Rotation

旋转变换：

![Alt text](image-10.png)
首先我们规定：

- 我们默认旋转都是绕着原点进行的。
- 默认逆时针方向旋转。

旋转的矩阵应该如何表述呢？

最终的旋转矩阵的结果是这样的：
![Alt text](image-9.png)
我们来做一下推导：

![Alt text](image-11.png)
根据右下角的这个点列一个方程，这样我们可以得到 A 和 C 的值，然后根据左上角的这个点列一个方程，可以求得剩下的两个值。

### Linear transforms

可以发现一个共同点，这几个变换都可以写成这样的形式：

![Alt text](image-12.png)
x' 是 a 和 b 的线性组合：$x' = ax + by$
y' 是 c 和 d 的线性组合：$y' = cx + dy$

我们把这样的变换叫做线性变换。

## Homogeneous Coordinates

### Why Homogeneous Coordinates?

前面线性变换里涉及到一个很重要的点：用相同的维度的矩阵来乘以这个向量。

有什么特别呢？这主要涉及到齐次坐标的概念。而齐次坐标是用来解决平移变换的问题的。

![Alt text](image-13.png)
![Alt text](image-14.png)

- 平移变换不能表示为前面提到的线性变换的形式，即平移变换不属于线性变换。
- 但对于变换来说，我们总是想要将其表示为矩阵乘以向量的形式，不想带上$\begin{bmatrix}tx\\ty\end{bmatrix}$这样的尾巴。(人类总是懒的，懒惰是一个美德，人类的进步很多都是因为懒惰带来的)

### Homogeneous Coordinates

![Alt text](image-15.png)

- 齐次坐标把二维的点或者向量增加了一个维度
- 这样平移变换可以表示为矩阵乘以向量的形式。
- 其他的线性变换也可以这样表示。

在齐次坐标下把点和向量区别对待，这里涉及到一个概念叫做平移不变性：

- 向量只用来表示方向，它是平移不变的，最后维度的零是为了保护它，在平移变换之后还是自身。

另外，把点最后设为 1，向量最后设为 0，还考虑到：
![Alt text](image-16.png)

- 对于点加上点，本来就没什么意义。加上过后最后一位变成 2 了，人们扩充了它的定义，在齐次坐标下，点的坐标是 x 除以 w 和 y 除以 w。所以点加上点的结果是它们的中点。

### Affine Transform

仿射变换：
![Alt text](image-17.png)

- 最后一行永远是 001
- 平移写在最后一列的头两个数上
- 左边还有一个 2x2，是原来的线性变换的一部分

当 tx 和 ty 都等于 0 时，就是线性变换：
![Alt text](image-18.png)

- 齐次坐标的引入，让我们可以很方便地把各种不同的变换写成同一个统一的表示形式。
- 但代价是引入了一个额外的数字，任何一个点 x y 都要写成 x y 1，任何一个向量都要写成 xy 0。矩阵原本是 2x2 加上两个数，现在是 3x3。（实际上最后一行都是 001，所以只需要存储前面这几个就可以了）

需要强调的是，最后一行是 001 指的是二维情况下的仿射变换，不是说任何情况下都是 001，例如后面要讲的投影变换。

### Inverse Transform

![Alt text](image-19.png)

逆变换简单来说就是把一个变换的操作反过来。

### Composing Transforms

![Alt text](image-20.png)
![Alt text](image-21.png)

要把左边的图变为右边的图，需要先逆时针旋转 45 度，然后再向右平移。

这里涉及到两件事：

- 复杂的变换可以通过一系列简单的变换得到
- 变换的顺序是非常重要的：如果先进行平移再进行旋转，得到的结果和先进行旋转再进行平移是不一样的。
  ![Alt text](image-22.png)

![Alt text](image-23.png)
在这门课里，我们默认向量是列向量，所以如果要乘一个矩阵，那么这个矩阵应该放在向量的左边。即把左边的矩阵应用到右边的向量上。因此，组合变换也应该是从右到左逐个应用这些矩阵。

![Alt text](image-24.png)
矩阵没有交换律，但有结合律。这意味着我们可以把很多不同的矩阵合成一个矩阵，把很多不同的变换合成一个变换。

### Decomposing Transforms

变换不只能够合成，还能够分解：
![Alt text](image-25.png)

## 3D transformations

![Alt text](image-27.png)
![Alt text](image-26.png)
