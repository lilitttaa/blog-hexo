---
title: 上帝视角看GPU（5）：图形流水线里的不可编程单元
category:
 - Game
sortValue: 019
---

- https://www.bilibili.com/video/BV1dL4y1c789

这期将深入 GPU 图形流水线的一些细节，看看那些不可编程的模块是怎么工作的。

![Alt text](image.png)

- 对于图形流水线来说，最核心最重要的一个组件就是光栅化器。
- 它直接决定了 GPU 在实时渲染方面的优势。以至于很多时候光栅化就是 GPU 图形流水线的代称。

![Alt text](image-1.png)
经过 vertex shader 之后，每个顶点上都有了转换后的属性，包括位置、法线方向、颜色、纹理坐标等。经过 primitive assembler 之后来到光栅化阶段，转换成像素。

![Alt text](image-2.png)
光栅化这个操作，本质上就是把三个顶点上的信息插值到这个三角形覆盖的每个像素上，交给 pixel shader。

![Alt text](image-3.png)
Primitive assembler 和光栅化总是连在一起，因此成为了广义的光栅化器。

![Alt text](image-4.png)
要完成这样的插值，常见的方法称为扫描线算法。顶点上有一系列的属性。根据三个顶点的位置和属性变化的程度，可以算出这个三角形覆盖的区域里，从一个像素挪到右边或者下边，各个属性会改变多少。这称为 ddx 和 ddy。

![Alt text](image-5.png)
对于一个三角形来说，属性的 ddx 和 ddy 都是常量，只要算一次就行。接着，从最靠上的顶点开始，沿着轮廓一行一行往下扫。每一行根据三角形轮廓就能知道应该从哪里开始，到哪里结束。往右一个像素，属性增加一次 ddx，往下一行，属性增加一次 ddy。

![Alt text](image-6.png)
那么，像素在什么情况下认为被三角形覆盖？这叫做光栅化规则。普通模式下，光栅化三角形看的是像素中心是否在三角形之内。光栅化线看的是线是否经过像素里的一个菱形区域。

![Alt text](image-7.png)
另一种模式是只要沾到一点就算覆盖。这叫保守式光栅化，常用于体素化的需求。

![Alt text](image-8.png)
比如前几年很热门的 SVO cone tracing，就用保守式光栅化来把整个场景变成体素的表达。

![Alt text](image-9.png)
光栅化的算法有了，直接放到硬件上，就成了立即式光栅化。这个做法很淳朴，用 ASIC 把刚才描述的算法变成硬连线。

![Alt text](image-10.png)
在算法完全固定的情况下，ASIC 的效率远高于 FPGA 和可编程单元。立即式光栅化生成的像素，经过流水线后面的几个阶段，写入内存里的渲染目标。渲染目标可能是纹理也可能是帧缓存。

![Alt text](image-11.png)
对于大三角形来说，这么做性能非常高。因为只要算一次 ddx、ddy，后面一路累加过去就行，可以不被打断地一直执行同样的操作。

![Alt text](image-12.png)
但是，如果三角形层层叠叠，就得反复写入内存，带宽占用很大，功耗高。

对 PC 渲染的需求来说，只要性能高，功耗高一点也可以接受的，所以往往会选择立即式光栅化的方案。

![Alt text](image-13.png)
这张图是我在 AMD FirePro V3900 的 GPU 上得到的，其中的颜色表达了像素到达 pixel shader 的顺序。

从这里可以看出，这个 GPU 采取了 32 个像素的高度，一条一条的光栅化方式。

![Alt text](image-14.png)
而 CPU 光栅化的 WARP，就是一个一个像素扫过去的方式。

![Alt text](image-15.png)
到了移动平台，这就够呛了。移动平台更看重的是性能功耗比。如果性能只有一半，但功耗只有四分之一，也会考虑采纳。于是，移动平台上，光栅化往往采用 tile-based 方案。

![Alt text](image-16.png)
它把渲染目标划分成很多固定大小的 tile，常见的是 32x32。每个 tile 包含一个列表，存有和这个 tile 相交的所有三角形。所以 tile-based 光栅化不再是一个一个三角形处理，而是一批一批处理。

![Alt text](image-17.png)
这样的 GPU，需要有一个片上内存充当 cache 的角色，不需要大，但访问速度远远高于内存。

![Alt text](image-18.png)
对于每个 tile，先会把渲染目标的对应区域载入 GPU 的片上内存。接着用扫描线算法，把列表里的三角形都渲染上去。最后把片上内存里的结果存到渲染目标。然后开始处理第二个 tile。

不管三角形如何层层叠叠，每个 tile 每次对内存的读写总是只有 32x32 个像素，远低于立即式。但因为一个三角形没法一直填充下去，会因为 tile 而被打断，性能其实是降低的。只是相比之下功耗降得更多。

![Alt text](image-19.png)
这是在高通的 GPU(Qualcomm Adreno 430) 上跑出来的光栅化顺序图。可以看到每个 tile 大小固定，一个一个 tile 铺成了整个屏幕。为了让大家加深印象，这里举两个实际的场景，比较一下立即式和 tile-based 在工作流程上的区别。

同样是渲染两个三角形。第一种情况，把它们渲染到同一张纹理。第二种情况，把它们分别渲染到两张纹理。

![Alt text](image-20.png)
对于立即式来说，都是把三角形光栅化出去。两个三角形是不是到同一个纹理无所谓。操作是一样的。因此两者的性能和功耗区别可以忽略不计。

![Alt text](image-21.png)
对于 tile-based，这俩就很不一样了。第一种情况，光栅化的流程是，第一个 tile 载入到片上内存，渲染两个三角形，存到纹理；
![Alt text](image-22.png)
第二个 tile 载入到片上内存，渲染两个三角形，存到纹理；依此处理完所有 tile。

![Alt text](image-23.png)
第二种情况的流程就变成，纹理 A 的第一个 tile 载入到片上内存，渲染三角形 A，存到纹理 A；纹理 A 的第二个 tile 载入到片上内存，渲染三角形 A，存到纹理 A；依此处理完纹理 A 的所有 tile；纹理 B 的第一个 tile 载入到片上内存，渲染三角形 B，存到纹理 B；纹理 B 的第二个 tile 载入到片上内存，渲染三角形 B，存到纹理 B；依此处理完纹理 B 的所有 tile。

注意，在片上内存里的操作(Rasterize)非常快，访问内存里的纹理(Load/Store Tile)，慢得多而且耗电得多。因此第一种情况比第二种好情况得多。这也是为什么在 tile-based GPU 上，切换渲染目标成为大忌。

![Alt text](image-24.png)
![Alt text](image-25.png)
![Alt text](image-26.png)
这几年 API 的改进集中于提供 subpass 和 invalidate 等操作，可以让开发者决定是否把纹理的 tile 载入片上内存，以及渲染后是否存到纹理，以减少这部分的开销和功耗。

![Alt text](image-27.png)
继续往后看。光栅化产生的像素，会进入 pixel shader，然后是 output merger。场景是存在遮挡的，近的会挡住远的。之前说过，这是在 output merger 里通过深度测试来完成。

![Alt text](image-28.png)
假设光栅化产生了像素 A，跑完后面的流程，写入渲染目标，又在同一位置产生像素 B。这里会产生两个问题。第一，如果像素 B 比像素 A 还近，那它跑完后面的流程之后，会覆盖掉像素 A。这使得像素 A 经过的 pixel shader 和 output merger 完全浪费了。

![Alt text](image-29.png)
第二，如果像素 B 比像素 A 还远，也得等到运行了 pixel shader，进入 output merger 才能发现有遮挡，才抛弃掉像素 B。那么像素 B 的 pixel shader 也白运行了。能不能把深度测试从 output merger 挪到 pixel shader 之前呢？

![Alt text](image-30.png)
不总是可以的，因为 pixel shader 不但能输出颜色，也能输出深度。

![Alt text](image-31.png)
如果光栅化产生的深度和 pixel shader 输出的深度顺序不一致，在 pixel shader 之前执行深度测试就会出错。

![Alt text](image-32.png)
为解决第二个问题，GPU 引入的功能称为 early-z。在渲染状态符合条件的情况下，驱动会检查一下 pixel shader，如果不输出深度、不用 discard 丢弃像素，就启用 early-z。像素在进入 pixel shader 之前提前进行一次深度测试。

如果已经被挡住，就不往下走，直接丢弃掉。

![Alt text](image-33.png)
在 tile-based 的光栅化上，有的 GPU 会有个称为 TBDR 的模式，tile-based deferred rendering。TBDR 在开启深度测试的情况下，把光栅化插值得到的像素属性都写入片上内存。

这时候不可见的像素就被抛弃了，只有可见的继续往下走，同时解决那两个问题。但是它无法独立存在的，也是要一系列条件都符合的情况下才能启用，否则退回到 tile-based 模式。

![Alt text](image-34.png)
既然立即式性能高，tile-based 性能功耗比高，能不能取长补短一下？有的。Maxwell 之后的 NVIDIA GPU，就采用了两者的结合，称为 tiled caching。

![Alt text](image-35.png)
它的 tile 巨大，256x256 这个级别，cache 也很大，不光像素，还可以把 tile 需要的几何也载入 cache。

![Alt text](image-36.png)

- 这么做降低了内存访问，性能和性能功耗比都更高了。这里仍然可以用一张 Nvidia Geforce GTX 960 的光栅化顺序图来看到这个情况。

前面说的，都是用硬件直接构造光栅化。它的性能优势来自于 ASIC 上执行固定的算法，但因此牺牲了灵活性。有没有可能用软件来构造光栅化呢？这里说的软件，不是指在 CPU 运行，而是在 GPU 的可编程单元里运行。

有些需求使得软件光栅化变得很重要：

![Alt text](image-37.png)
第一个需求，小三角形的光栅化性能。在实际中，硬件光栅化的输出并不是一个一个像素，而是一个一个 2x2 的像素块。这称为一个 quad。

![Alt text](image-38.png)
quad 之内每个像素都有邻居，于是可以在 pixel shader 里获得任何变量的 ddx 和 ddy，只要和邻居一减就出来了。

![Alt text](image-39.png)
这四个像素如果有在三角形之外的，之后才会被丢弃。对于大三角形来说，最多也就是边缘的像素存在浪费。

![Alt text](image-40.png)
但对于小于一个像素的三角形，这就浪费了 3/4。因此，对于大量三角形都小于一个像素的时候，构造一个以像素为单位的软件光栅化器，可以避免浪费，性能反而更高。

![Alt text](image-41.png)
在 UE5 的 Nanite 里就是这么做的优化。

![Alt text](image-42.png)

另一个需求，在无法使用硬件光栅化的时候执行光栅化。典型的是 Intel 在 08 年的 Larrabee。

![Alt text](image-43.png)
上面没有常规的流式处理器，而是堆了 48 个奔腾的 x86 CPU，加上很宽的 SIMD 指令集。

![Alt text](image-44.png)
在那些 CPU 上执行的是个特制的软件光栅化算法，自适应细分的 tile-based 光栅化。

![Alt text](image-45.png)
首先，像普通的 tile-based 一样，把整个渲染目标分成一系列 tile，但是每个 tile 较大，至少 64x64。同样也是每个 tile 包含与它相交的三角形列表。

接着把这个 tile 等分成 16 个小 tile，测试每个小 tile 和三角形的相交情况。完全覆盖了，就全部填充。完全不相交，就跳过。如果部分覆盖，就把小 tile 再继续分成 16 个更小的 tile，再次测试，以此类推直到像素级别为止。每次都是进行宽度为 16 的 SIMD 计算。

这部分细节，有兴趣的朋友可以看 [Salvia 渲染器](https://github.com/wuye9036/SalviaRenderer)，里面有这个算法的开源实现。后来，Larrabee 项目停止，砍掉显示输出能力后改造成 Xeon Phi 运算卡，那个 SIMD 指令集成了 AVX-512。普遍猜测是功耗控制不住。

[high-performance software rasterization on gpus](https://research.nvidia.com/sites/default/files/pubs/2011-08_High-Performance-Software-Rasterization/laine2011hpg_paper.pdf)

![Alt text](image-46.png)
NVIDIA 也有个类似的工作，用 CUDA 构造软件光栅化。

![Alt text](image-47.png)
既然光栅化器也可以用软件构造了，那么其他固定流水线模块呢？挨个过一遍吧。

![Alt text](image-48.png)
Input assembler 负责读取和组装顶点。在 shader 能直接访问 buffer 之后，这很容易就能改成让 shader 读取。有时候性能还能更高。比如位置和法线用不同的 index 的情况，如果必须用硬件 input assembler，就得重新组织 buffer，有些数据得复制组合多份。

![Alt text](image-49.png)
在 shader 里写代码读取的话，可以支持多个 index buffer，总的数据量更小。这在虚幻引擎里面叫做 manual vertex fetch 模式，用得越来越多。Stream output 原理也一样，无非就是改成写入 UAV。

Tessellator 是在三角形内生成新的顶点，连起来变成一组三角形。

![Alt text](image-52.png)
我发明过一个方法，通过查找表迅速定位顶点之间的连接关系，在 compute shader 里实现 tessellator。只要修改查找表就能更换连接的 pattern，不一定要按照硬件写死的。

![Alt text](image-50.png)
![Alt text](image-51.png)
![Alt text](image-53.png)
![Alt text](image-54.png)
这个算法被《文明》系列游戏借鉴了，用于地形的细分，通过可定制 pattern 的思路，做到比硬件 tessellator 更高的速度。

![Alt text](image-55.png)
最后的 output merger，执行的是这个流程图，变成代码是非常直接了当的事情。

唯一需要注意的是 alpha blending 如果没有硬件帮助，性能会受一些影响。如此看来，图形流水线里的几乎每一部份，都可以用可编程单元来构造。

![Alt text](image-56.png)
唯一必须用硬件才有绝对性能优势的是纹理采样。它需要对纹理进行读取、解压、插值，计算量不小，算法很固定，适合硬件连线去做。按照 Larrabee 的研究，用软件实现性能会降低 12 到 40 倍。

![Alt text](image-57.png)
而对于那些所谓“通用 GPU”，只有计算流水线没有图形流水线的诈骗货，其实也可以用这样的软件方式构造图形流水线，在驱动里无缝衔接，成为真正的 GPU。
