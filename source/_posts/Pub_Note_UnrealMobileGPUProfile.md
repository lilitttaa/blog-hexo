---
title: 虚幻引擎移动游戏中的 GPU 性能分析与优化
category:
 - Game
sortValue: 016
---

- [虚幻引擎移动游戏中的 GPU 性能分析与优化 | 张强 侑虎科技](https://www.bilibili.com/video/BV15c411b7rq)

## 概述

![00-00-49](00-00-49.jpg)

优化方法：

- 把各种效果全开的这种画质的情况下尽可能的去优化
- 对中低端设备做逐层的 LOD（场景复杂度、面片数等等）优化，利用 UE 本身的画质分档

优化尺度：

- 优化尺度解决的问题就是在于帮助我们去找到核心的去需要去优化的点
- 导致 GPU 压力的情况很多，比如 overdraw、三角形面数非常多、使用的后处理计算的复杂度很高
- 各种不同的 GPU 压力产生的影响也会不一样，有些会导致帧率往下降、有些会导致能耗很快、还有一些会发烫很快

![00-05-01](00-05-01.jpg)

- 利用工具可视化的去衡量 GPU 的压力
- 对 GPU 来说更多影响的是帧率、耗电、发热。而卡顿和内存出现比较少，或者说对于内存优化的方式很固定：就是减少用的纹理、网格等资源。

![00-06-28](00-06-28.jpg)

- 掉帧：相当于说他一帧的时间会变长
- 耗电：GPU 运行的能耗，以及数据的传输也会产生能耗。从功率上说，可以分为静态功率、动态功率、传输功率
- 发热：本身从电能转为硬件的执行，这个过程中就会有能量转换，就涉及到发热。设备本身也具有散热的能力，所以越来越烫，主要还是发热过快的问题。

![00-08-19](00-08-19.jpg)

- 上边是各种各样的 GPU 的 counter，或者说性能指标
- 三种主流的 GPU:
  - 马里（Mali）
  - 高通（Adreno）
  - PowerVR

对上边的 counter 进行分类（每个类别不只是一个 counter）：
![00-08-49](00-08-49.jpg)

每个类别都能够产生数据，每一个类别都能够展开：

![00-09-09](00-09-09.jpg)

- 比如图元可以分为输入的图元、可见的图元、剔除的图元
- shader cycle 可以分为四种不同类型的 shader 的 cycle，不同架构上可能数量不一样，会有四种不同类型的一些计算量

![00-09-41](00-09-41.jpg)

- 有很多性能指标，主要分为核心和重要的两类，还有一些次要的指标
- 实际处理的时候，先看和核心指标里边有没有问题，然后再去看重要指标里边跟他相关的部分。

## 核心指标

### Clock

讲 Clock 之前，先看看一个叫做能效曲线的东西：
![00-10-37](00-10-37.jpg)

- 这是一个压力场景测试的曲线
- 横轴是频率，纵轴是执行的帧数
- 发现 GPU 提高两倍的频率，能耗不是提高两倍，而是非线性的。
- GPU 是希望用更低的频率去做事情，因为更低的频率就意味着更低的能耗

之所以会存在这样的曲线，是因为 GPU 有一个叫做 DVFS 的机制：

![00-11-35](00-11-35.jpg)

- 它会动态的去调整 GPU 的工作模式
- 从公式上看功率和频率是线性的，但实际上为了支持更高的频率，需要提高电压，这就导致功率是非线性的上涨
- 对于大多数芯片来说有一个 OPP 的机制，就是一个配置文件，配了几个挡位。任务量大的时候用高档，电压也高。当然它会有一个最大的频率，就是满频的跑。

所以这决定了 GPU 有两个属性：
![00-12-55](00-12-55.jpg)

- 运行频率是会非常影响整个 GPU 的工作的快慢
- 最大频率就已经决定了它的一个瓶颈
- 这里频率用 HZ 作为单位，代表每秒一个 clock，一般 GPU 都是几百兆赫兹，相当于每秒可以做几百兆个 clock

另外一个维度是时间，就是在这个频率下运行的时间：
![00-13-28](00-13-28.jpg)

- 这是马里的官方文档里的一个图
- GPU 的工作时长主要分为两类，一类是 fragment 相关的，一类是非 fragment 相关的。或者说是像素的处理和顶点的处理。它会在两条不同的队列里边去执行。
- 最长的那条就是 GPU 处于工作状态的时间，非 fragment 相关和 fragment 相关会有一段重合的时间。

![00-14-36](00-14-36.jpg)
![00-14-51](00-14-51.jpg)
![00-15-01](00-15-01.jpg)

- 这样我们就能评估出来 GPU 的工作量有多大

这个 Clock 到底是什么含义呢？

![00-15-39](00-15-39.jpg)

- 大概就是上边列出来的这些一个一个的步骤

![00-16-03](00-16-03.jpg)

- 对于相同设备，相同渲染内容，单帧的 clocks 基本上是不变的。因为画的内容就是这么多，要做的事情就是这么多。
- 所以 clock 是一个核心的指标，他能代表当前画面 GPU 的工作量。不管帧率是多少，他都是这么多 clocks。

![00-16-36](00-16-36.jpg)

- 根据这个公式就能衡量出是不是会掉帧

![00-17-05](00-17-05.jpg)

- 即使不掉帧，也不能让 GPU 满频跑，因为满频跑会导致发热很快，能耗也很高
- 所以至少要给他降一档，一般来说尽量控制在 GPU 最大频率的 80%左右
- 这个 80%主要是一个经验数值，后面我们也可以看具体的一些这个数据

![00-18-14](00-18-14.jpg)

- 前面说的 GPU 的 clock 产生的功率我们称为动态功率
- 传输功率就是带宽的部分，也会产生额外的功率，两个部分会共同产生作
- 所谓带宽就是 GPU 在 shader call 里边处理各种渲染任务的时候，会去采纹理，访问顶点。
- 如果数据是在 L1L2 的 cash 里面，产生的带宽相对比较有限，但如果不在这些 cash 里面，就要去显存里面去访问，产生的带宽就会影响就会比较大

![00-19-02](00-19-02.jpg)
![00-19-20](00-19-20.jpg)
![00-19-21](00-19-21.jpg)

- 带宽对于功耗的影响主要来源于一些论文和马里的一些官方文档
- 可以简单的估计为 1G/s 的带宽产生的功耗大概在 100mW 左右

![00-19-39](00-19-39.jpg)

- 这里的数值也都是一些经验数值（线上的统计和论文中的数值）
- 整个设备运行时跑游戏时，通常在 2000 到 4000 毫瓦。也有 6000 到 7000 的，但这种毫瓦下的话，整个耗电就会很快，所以比较合理的话应该是控制在 4000 左右。

![00-20-33](00-20-33.jpg)

- 按照前面说的 200-600mW 的传输功率，然后 80-100 是一个 G，所以带宽应该控制在 2 到 6 个 G 左右
- 优化的比较好的游戏，带宽可以控制在 3 到 4 个 G 左右
- 对画质要求高的，也能承受一定的发热，7 到 8 个 G 也是有的

![00-21-18](00-21-18.jpg)

- 所以这样我们就能通过单帧的 clock 和带宽去评估在目标设备上会不会产生压力
- 单帧的数据也能通过 fps 和每秒的带宽去换算

![00-22-15](00-22-15.jpg)

- 发热说到底就是功率，移动端设备 2-3 瓦的热量是可以承受的，4 瓦以上就会发热很快，6-7 瓦就会非常快

## Demo 分析

![00-23-01](00-23-01.jpg)

- 华为 v20，马里 G76 的一个测试
- 10 个 shader call，最大频率 720MHz
- 每 30 帧增加一层 overdraw，跑到 360 层左右

![00-23-41](00-23-41.jpg)

- 可以看到 clocks 跟 overdraw 是线性的，因为 overdraw 越多，要做的工作就越多，clocks 自然就越多
- 1200 万标记的那个地方，差不多是在 2200 多帧，这里刚好符合之前的公式：12.7M > 720MHz/60 = 12M，所以到这个点就跑不满 60 帧了

下面是帧率的走势：
![00-24-26](00-24-26.jpg)

- 可以看到到 2200 帧左右就开始掉帧了，这个点是非常明确的
- 发现一个奇怪的点：3900 帧的位置有个明显下跌，相当于帧率又突然断崖往下降

看一下 GPU 频率：
![00-24-52](00-24-52.jpg)

- 2200 帧左右到达满频，然后 3900 帧左右降频到 81%，也就是 500 多兆。然后就持续下降。
- 温度太高了，开始降频，一降频就是 FPS 再往下跌一档

这里的降频实际上跟功率有关：

![00-25-43](00-25-43.jpg)

但功率为什么要下降？

![00-25-55](00-25-55.jpg)

- 温度达到了 65 度，触发了保护机制，把整个 GPU 功率往下降了一档，导致帧率也下降了一档

![00-26-16](00-26-16.jpg)

- 所以这里也可以发现 80%是最终稳定的一个状态，所以控制到 80%的最大频率是一个比较好的点

所以最终对于 GPU 优化的这个指标，实际上就归纳成是：

- 把单帧的 GPU clocks 控制在 80%
- 带宽尽量控制到每秒 4 个 G 以下

## 重要指标

clocks 相关的指标：

![00-27-33](00-27-33.jpg)

- utilization：整个 GPU 的工作时间有多少是在处理这个非像素的，有多少是在处理像素的。可以看到是不是顶点处理的压力太多导致的 GPU 的压力。

![00-27-52](00-27-52.jpg)

shaded 去除以屏幕分辨率也就是 overdraw：
![00-28-09](00-28-09.jpg)

四种 cycle：
![00-28-15](00-28-15.jpg)

- 数学计算
- 顶点到这个像素的插值的计算
- 寄存器的存取
- 纹理的采样
- 这四种实际上是并行在做处理的，所以需要去看这四种曲线，到底是哪一种因素导致的压力比较大

图元：
![00-28-53](00-28-53.jpg)
![alt text](image.png)

- 一般来说回去看 3D 场景剔除的百分比，差不多在 50%左右，还会再更低一些
- 可能是由很大的模型，只有一小部分在视野里边

带宽还会有几个跟他相关的：
![00-29-25](00-29-25.jpg)

- 分为读和写
- 一般来讲读的带宽比例会更大一些
- 也分为读纹理和读顶点，一般来说读纹理的带宽会更大一些

![00-29-46](00-29-46.jpg)

- 不管是读纹理还是顶点，都会涉及到 stall，相当于是发送了一个传输请求，但是硬件层面说现在传不了。
- 这种请求的比例也会有一个统计，就是这个 stall 的百分比，能一定程度反映出带宽的压力

![00-30-11](00-30-11.jpg)

- 在 adreno 的 gpu 上边还能拿到 L1L2 的一些 cash miss 的比例
- miss 越多带宽也会越大

![00-30-27](00-30-27.jpg)

## 总结

![00-30-44](00-30-44.jpg)
