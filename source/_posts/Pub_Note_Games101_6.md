---
title: Games101 6.Rasterization 2(Antialiasing and Z-Buffering)
category:
 - Games101
sortValue: 30005
---

一幅照片就是所有到达感光元件所在的这个平面的一些光学的信息，离散成这一系列的图像上的像素
![Alt text](image.png)

视频实际上也是采样，是在时间中进行的采样，可能是每秒 24 帧图像连在一起播放。
![Alt text](image-1.png)

在图形学中有这样一个词叫 artifact，表示错误、不准确或者我们不希望看到的结果，就是一切我们觉得看上去不太对的东西。

![Alt text](image-2.png)
除了锯齿之外，还有一种 artifact 叫做 moire pattern（摩尔纹）：
![Alt text](image-3.png)
就是当我们对一个图像进行采样的时候，我们会发现一些图像上的纹理会变形，这也是采样带来的问题。

![Alt text](image-4.png)
再来看一个问题（Wagon Wheel Illusion），画了各种条纹的纸片，顺时针旋转纸片，会发现有些条纹看起来像是逆时针旋转的。这是因为人眼在时间上的采样跟不上运动的速度。

走样的问题本质是因为什么呢？
![Alt text](image-5.png)

- 其实相当于是信号（也就是函数）变化太快了，以至于你的采样的速度跟不上它
- 需要通过频域来分析这个问题

先看一下如何做反走样：
![Alt text](image-6.png)

- 在采样之前先做一个模糊或者说滤波（filtering），把三角形变成一个模糊的三角形，采样到模糊的边界才变成这样的粉红色。
- 也就是说对原始的函数或者信号做一个模糊，然后再去做采样，就可以解决锯齿的问题，也就是抗锯齿或者说反走样。

对比一下效果：

- 原本的采样效果
  ![Alt text](image-7.png)
- 反走样之后的效果
  ![Alt text](image-8.png)

![Alt text](image-9.png)

但如果反过来，先做采样再做模糊（blurred aliasing）
![Alt text](image-10.png)
这样走样之后又被模糊了，是不行的。

![Alt text](image-11.png)

- 为什么采样速度跟不上信号变化的速度就会产生走样？
- 为什么先采样后模糊达不到反走样的效果？

这就需要频域（frequency domain）相关的知识了。

这分别是正弦与余弦波：
![Alt text](image-12.png)

调整前面的系数，就可以得到不同频率的余弦波：
![Alt text](image-13.png)

- 频率可以用来定义正弦波或者余弦波的变化速度
- 频率越高，变化越快
- 或者说定义波的周期，周期是频率的倒数，也就是说每隔多久这个函数会重复一次

微积分里边存在各种各样的展开，其中有一个叫做傅里叶级数展开：
就是说任何一个周期函数都可以写成无限多项正弦和余弦函数的线性组合加上一个常数项。
![Alt text](image-14.png)
傅里叶展开与傅里叶变换是两个不同的概念，但是它们紧密相关。

![Alt text](image-15.png)

- 给定任意函数，都可以通过傅里叶变换变成另一个函数
- 反过来，也可以通过逆傅里叶变换变回原来的函数

前面的各个余弦函数，它们的频率系数是不一样的，它们代表不同的频率。也就是说任意函数都可以分解成不同的频率段

如果把这些频率段给显示出来：
![Alt text](image-16.png)

- 从上到下，频率越高
- 频率越高，需要的采样间隔就越小

我们来看一下在频率上走样是什么样的：
![Alt text](image-17.png)

- 采样间隔不够小，恢复出来的函数跟原本的函数差别很大。
- 或者说蓝色的曲线和黑色的曲线，用同样的采样方法得到的结果无法区分，这就是走样。

相关的还有一个重要的概念叫做滤波（filtering）：

- 从频率的角度上来说，滤波就是把一些频段的信息给抹掉

傅里叶变换可以把一个函数从时域变到频域：
![Alt text](image-18.png)

- 虽然图像本身不代表任何时间的信息，但是我们认为这个空间上的不同位置也算是时域
- 右边的频域图
  - 中心看作是最低频区域，周围是高频区域，然后亮度表示不同频率上的信息量。
  - 这里可以看到图像的大部分信息都集中在低频上，中间比较亮。（很多图片都是这样）
  - 为什么会有水平和竖直的一道呢？信号我们认为是周期性重复的，图片被认为右边界过后又重复左边的内容，这条边界上会有剧烈的信号变化，产生高频。不过我们一般忽略这两条线。

把图像的低频信息抹掉，只留下高频信息：
![Alt text](image-19.png)
其实高频表示的就是图像的边界（跟周围的像素发生了突变，信号变化大），这种滤波叫做高通滤波。

低通滤波：
![Alt text](image-20.png)

- 边界看上去就很模糊了
- 图像上的类似于水波纹一样的东西实际上是应用了一个不完美的低通滤波器产生的问题

去掉高频信息和低频信息，留下中间的频率：
![Alt text](image-21.png)

- 提取到一些不是那么明显的边界特征
- 中间大面积的相同色块被去掉

这个部分图形学不会讲太多，如果要学的话需要学一门叫做数字图像处理的课。

滤波从另一个角度分析，滤波又等于平均，又等于卷积（convolution）：

- 前面提到低通滤波器，就好像图像被模糊了一样，而模糊就是一种平均操作

卷积是什么呢？

![Alt text](image-22.png)
用这样三个格子的滑动窗口，覆盖的三个数和窗口所覆盖的信号的三个数做一个点乘，得出来的结果写回这个窗口的中心值就可以了。
![Alt text](image-23.png)
窗口不断地向右移动，就可以得到整个图像的卷积结果。
![Alt text](image-24.png)

![Alt text](image-25.png)

- 对两个信号进行卷积，就是对应到两个信号的频域上，是两个信号的频率的乘积。
- 所以也可以通过傅里叶变换先把信号变到频域上，然后做乘法，再变回时域

![Alt text](image-26.png)

- 对于上边的的例子，应用 3x3 的卷积核进行卷积操作
- 这里乘上 1/9 是因为要让整体的颜色值跟以前不会发生变化，不然会让图像变亮
- 这个滤波器叫做 box filter，也就是一个盒子形状的滤波器，它是个低通滤波器

现在有个问题是，如果这个 box 在时域上变大了，它在频率上会怎么变化呢？
答案是，它在频率上会变小。因为这个 box 越大说明模糊越厉害，所以留下的低频部分就越小。

采样在频率上是什么意思呢？
![Alt text](image-27.png)

- 采样其实就是在重复频率或者说频域上的内容
- 要把这些函数变成一系列离散的点，就像是把这个函数乘上一个冲击函数一样
- 时域的卷积对应到频域是乘积，反过来也成立，时域的乘积对应到频域上是卷积。
- 在频域上就是把原始的频谱给复制粘贴了很多

![Alt text](image-28.png)

- 如果采样的不够快，复制粘贴过后的信号之间的间隔就会很小。在时域上采样的间隔越大，但是在频域上频谱的间隔就越小（时域跟频域有好多相反的关系）。
- 原始的信号复制粘贴的信号混在一起了，就会产生走样。

那要如何解决走样的问题呢？
![Alt text](image-29.png)

- 最简单的方法就是增加采样率，不过往往受到硬件的限制，不可能开启一个选项然后就让屏幕的分辨率变高。
- 另外一个方法就是做反走样，也就是先做模糊，然后再做采样。

![Alt text](image-30.png)

为什么先做模糊有效，原本的这样一个函数在频域上就是这么一个梯形，我们先做模糊，其实也就是把这个梯形两边的高频信号给砍掉，然后再用原本的稀疏采样率去采样，就不会发生混叠了。

在实践上：
![Alt text](image-31.png)
先对像素做一个卷积或者说叫平均操作，然后再采样。
![Alt text](image-32.png)
具体来说，我们根据每个像素的覆盖情况来混合像素的颜色。比如说这个三角形覆盖了这个像素的 1/8，那么这个像素的值就应该是 1/8 的亮度，如果完全覆盖了，那就是全黑。

但是要判断像素的覆盖情况很复杂，所以人们提出了一种近似的方法，叫做 MSAA（Multi-Sample Anti-Aliasing）：
![Alt text](image-33.png)

- 用更多的采样点来近似反走样
- 比如这里一个像素内用 4x4 的采样点（4x4 Supersampling），判断每个采样点在三角形内的情况，然后再取平均
  ![Alt text](image-34.png)
- 需要注意的是，MSAA 通过增加采样点做的只是一个模糊的操作，并不是增加分辨率，也没有直接增加采样率。在这之后才是真正的采样。

最后得到这样的结果：
![Alt text](image-35.png)

![Alt text](image-36.png)
MSAA 也是有代价的：

- 因为要判断更多的采样点在三角形内的情况，所以需要更多的计算量。
- 在工业界，并不是在一个像素上规则的划分为 4x4 个点，而是用一些更加有效的图案来分布这些采样点，有一些点还会被临近的不同像素所复用。这也是为什么打游戏的时候开启 MSAA 会降低帧率，但是降低的幅度没有那么大。
- 除了 MSAA 之外，还有很多其他的抗锯齿方法，这里介绍两种：
  - FXAA（Fast Approximate Anti-Aliasing）：是一种后期处理的方法，先得到有锯齿的图像，通过一些图像匹配的方法把，然后再把这些边界换为没有锯齿的边界。这个方法和采样无关，是在图像层面上做的抗锯齿，非常快。
  - TAA（Temporal Anti-Aliasing）：是一种近几年才兴起的抗锯齿方法，它在相邻帧上可以用一个像素内部的不同位置上的点来感知三角形。其实就像是 MSAA 在时间上的分布，但是在当前帧上并没有引入额外的操作。这里说的还是一个静态的场景，如果是对于动态的物体怎么办，这个在实时光线追踪的部分再说。

最后再提一个概念，叫做超分辨率（Super Resolution），它和抗锯齿的概念很相似，把一张小图放大到大图，少到那些信息怎么办呢？就要靠猜，深度学习就适合做这个事情。
