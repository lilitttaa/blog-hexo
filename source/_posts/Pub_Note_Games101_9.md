---
title: Games101 9.Shading 3 (Texture Mapping cont.)
category:
 - Games101
sortValue: 30008
---

## Barycentric Coordinates

为什么要对三角形内部的点进行插值？
![Alt text](image.png)

- 已知三个顶点属性时，希望在三角形内部的任何一个点得到一个平滑的过渡。
- 以及纹理映射时，三角形内部的点映射到纹理上的哪个部分。
- 顶点的法线插值，得到三角形内部的法线。

重心坐标到底是什么？

![Alt text](image-1.png)

- 首先重心坐标是定义在一个三角形上，任意一个点 x y 都可以表示成三个顶点 a b c 的线性组合，即$\alpha a + \beta b + \gamma c$。
- $\alpha + \beta + \gamma = 1$。因此，已知两个数就可以求出第三个数。
- 且这三个数都是非负的，否则可能在三角形外部。

![Alt text](image-2.png)

- a 点可以写作$\alpha a + 0b + 0c$，所以 a 点的重心坐标是(1, 0, 0)。
- b 点是(0, 1, 0)，c 点是(0, 0, 1)。

重心坐标有个定义，它其实是可以通过面积比求出来的
![Alt text](image-3.png)

- 顶点它对面的三角形的面积占整个三角形的面积的比例

那么根据这个定义就能得到一个特殊的点，就是三角形自己的重心
![Alt text](image-4.png)

- 三角形的重心跟顶点连起来，把三角形面积等分为了三份。

我们根据：

- 上面的面积比例
- 向量叉乘等于平行四边形面积

的这样两个性质，可以推导出重心坐标的一般表达式。
![Alt text](image-5.png)
推导过程：
$A_A = \frac{1}{2} \vec{BC} \times \vec{BP} = \frac{1}{2} ((x_c-x_b)(y_p-y_b)-(x_p-x_b)(y_c-y_b))$
$A_B = \frac{1}{2} \vec{CA} \times \vec{CP} = \frac{1}{2} ((x_a-x_c)(y_p-y_c)-(x_p-x_c)(y_a-y_c))$
$A_C = \frac{1}{2} \vec{AB} \times \vec{AP} = \frac{1}{2} ((x_b-x_a)(y_p-y_a)-(x_p-x_a)(y_b-y_a))$
$A_{Sum} = \frac{1}{2} \vec{AB} \times \vec{AC} = \frac{1}{2} \vec{CB} \times \vec{CA}$
$\alpha = \frac{A_A}{A_Sum} = \frac{\vec{BC} \times \vec{BP}}{\vec{AB} \times \vec{AC}}$
$\beta = \frac{A_B}{A_Sum} = \frac{\vec{CA} \times \vec{CP}}{\vec{CB} \times \vec{CA}}$

现在有了重心坐标，我们就可以对三角形内部的点进行插值了。
![Alt text](image-6.png)

- 任意点的属性，就可以表示为三个顶点的属性的线性组合
- 属性可以是纹理坐标，颜色，深度等等

大家看 opengl 或者一些其他文章可能会遇到一个问题，重心坐标在投影后能保证不变吗？

- 答案是不能，所以如果要做三维空间的插值，应该取三维空间的坐标计算重心坐标，而不是在投影后的三角形里做插值。
- 这个主要涉及到光栅化的时候深度的一个问题，这时候三角形已经投影到屏幕上了，那么投影的三角形内的深度直接插值是不行的。需要通过逆变换，将屏幕上的点映射回三维空间，然后再计算重心坐标。

## Applying Textures

![Alt text](image-7.png)

- 我们可以根据三角形的顶点的 uv 进行插值，得到内部点的 uv
- 然后使用这个 uv 去纹理上查找颜色，直接把这个颜色作为 kd

这就是我们通常使用纹理映射的方式，但是会遇到一些问题

### Texture Magnification（纹理放大）

比起渲染出来的像素，纹理太小了会怎么样？比如一堵墙渲染出来是 4k，但是纹理只有 256x256。
![Alt text](image-8.png)

- 这个问题叫做纹理放大
- 查询纹理的时候，会查到一些非整数的值，这些值会被 round 到整数
- 纹理上的像素叫做纹素（texel）

要想得到更好的平滑的效果，需要用到双线性插值（bilinear interpolation）这个概念。
![Alt text](image-9.png)
先找到周围四个采样点
![Alt text](image-10.png)
![Alt text](image-11.png)

- 在垂直或者水平方向，用两个点进行插值只是线性插值
- 但是在两个方向上都进行插值，就是双线性插值
- 这两个方向插值顺序不影响结果

![Alt text](image-12.png)

- 当然还有比 bilinear interpolation 效果更好的插值方法，比如 bicubic interpolation，它会用周围 16 个点进行插值，但是运算量更大。
- bi 代表两个方向，cubic 代表三次插值（用四个点进行三次插值），而不是线性插值。

纹理采样还有可能遇到另一个问题，就是纹理太大。我们来看这张图：
![Alt text](image-13.png)

- 近处有锯齿，远处有摩尔纹

![Alt text](image-14.png)

- 右边代表近处，一个像素对应纹理上的区域较小
- 左边代表远处，一个像素覆盖了一整个纹理
- 对于左边来说，我们要用像素中心采样的值去近似整个区域的值，得到的结果显示是完全不准确的

理论上，我们可以通过超采样来解决这个问题：
![Alt text](image-15.png)

- 每个像素内有 512 个采样点，然后取平均值

但是这样会导致运算量大，我们首先来分析一下这个问题：
![Alt text](image-16.png)

- 以远处一个像素覆盖整个纹理的情况来看，实际上就是在一个像素内纹理是一直在变换的，只用一个采样点是没法表示这个像素区域的平均情况。换成走样的表诉，就是信号变化过快，采样频率跟不上。
- 如果能直接获取这个区域的平均值，就不需要超采样了。这个就是 mip map 的概念。

这里简单提一下：
![Alt text](image-17.png)

- 点查询：以 texture 采样为例，给定一个点，它的值是多少。前面说的双线性插值就是在做这个事。
- 范围查询：给定任何一个区域，立刻可以得到它里面内部的平均值。范围查询是一个经典问题，也不只是平均值，还有最大值和最小值等等。

### Mip Map

![Alt text](image-18.png)

- Mip Map 是一个经典的图形学概念，它允许做范围查询
- 速度快，但是是近似的。它只能做近似的正方形范围查询，其他形状不行。

![Alt text](image-19.png)

- MipMap 就是从一张图生成一系列图，每一层都是上一层缩小一倍
- 一直到第 Log2 层，$\log_2{128} = 7$，算上第 0 层，总共有 8 层
  ![Alt text](image-20.png)
- 计算机视觉里面，大家不管这个叫 Mip Map，管它叫 Image Pyramid

MipMap 会带来多大的额外存储开销呢？

- 1/3 个第 0 层的大小
- 你可以把每一层都复制三份，然后从第 0 层开始进行填充，你会发现 3 倍的 MipMap 刚好能填充一个 4 倍的第 0 层的大小，所以额外存储开销是 1/3。

MipMap 的查询过程是怎么样的呢？怎么知道要查询第几层呢？
![Alt text](image-21.png)
![Alt text](image-22.png)

- 分别在 x 和 y 方向上做微分（在屏幕上 x/y 方向上移动一个单位，纹理上移动多少单位）

可视化展示 MipMap 的效果：
![Alt text](image-23.png)

- 远处用到更深的层，近处用到浅的层

但是现在有一个问题，就是 MipMap 也是离散的，所以还是会有不连续的问题。比如我想查第 1.8 层，要怎么办呢？
![Alt text](image-24.png)
我们跟之前的 bilinear interpolation 一样，先查第一层，再查第二层，然后再这两层之间再做一次线性插值，这样就是三线性插值（trilinear interpolation）。
这样就能得到一个连续的值：
![Alt text](image-25.png)

对比 supersampling：
![Alt text](image-26.png)
![Alt text](image-27.png)

我们发现远处看上去已经糊掉了，这是什么问题呢？

- 一方面是 MipMap 本身，它只能做正方形范围查询，其他形状不行
- 另外一个问题是三线性插值本身，它也是近似的，毕竟不是真正算出来的

### Anisotropic Filtering

对于正方形查询这一点，人们又发明了另外的方法，比如各项异性过滤（anisotropic filtering）：
![Alt text](image-28.png)
![Alt text](image-29.png)
在像素上的一块区域对应到纹理上通常是一个矩形区域，各项异性过滤允许我们对这种长条形区域做一个快速的范围查询，而不用限制在一个正方形区域内。
![Alt text](image-30.png)

- 各项异性过滤可以解决水平和垂直的矩形查询，但是对于斜着的区域不行
- 各向异性过滤的存储开销增加了 3 倍
- 打游戏的时候各向异性开多少层没有太大影响，最终也会收敛到 3 层，主要是对显存有要求。

### EWA Filtering

![Alt text](image-31.png)

- 对于任意的不规则形状，都可以拆成多个圆形去覆盖
- 比如这里的椭圆，拆成三个不同的圆形去覆盖，进行多次查询。

（更详细的讲解可以参考 PBRT）
