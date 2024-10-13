---
title: 上帝视角看GPU（4）：完整的软件栈
category:
 - Game
sortValue: 018
---

- https://www.bilibili.com/video/BV1QT4y1r7Vq

本期来看看程序是如何控制 GPU 做事情的。

## 图形 API 软件栈

![Alt text](image.png)

- 很久以前，程序需要通过操作系统提供的硬件端口读写，直接操作图形硬件。

![Alt text](image-1.png)

- 如果每个程序都需要对每个操作系统的每个硬件写一遍，开发效率非常低。

![Alt text](image-2.png)

- 人们利用抽象的思路，形成了一个公用的接口层，也就是应用程序编程接口，API。
- 程序只要针对图形 API 写一遍就行，几乎不必考虑操作系统和硬件的区别。
- 图形 API 由硬件厂商实现，往下翻译成对硬件的操作。

逐渐的，人们又发现，同个 API 的不同实现，也存在大量可以公用的部分：
![Alt text](image-4.png)

- API 的实现又进行了分 层，增加的一个抽象层称为设备驱动接口，Device Driver Interface（DDI）。
- DDI 往上属于操作系统，负责数据有效性检查、内存分配等。
- DDI 往下是驱动，负责各个硬件特别的部分。
- 相当于操作系统把 API 翻译成 DDI，驱动把 DDI 翻译成对硬件的操作。这就是图形 API 软件栈的架构。

当然，这只是理想状况，现实中往往会有所调整。尤其是操作系统本身就分为用户态和内核态，使得组合更为复杂。

下面以几个有代表性的 API 为例，看看它们现实中的架构。

## D3D

![Alt text](image-3.png)

- 微软的 Direct3D（D3D），这里只讨论 Windows 上的官方实现。
- 这个 API 不跨平台，但跨厂商。

![Alt text](image-6.png)

- 在 Windows XP 的时代，软件栈就和理想状况下一样。
- 操作系统提供一个 D3D runtime，往上是 API，往下是 DDI。厂商提供一个内核态驱动。
- 这个框架称为 XDDM。

![Alt text](image-5.png)

- 随着对稳定性、性能、共享资源的需求不断增加，到了 Vista 时代，runtime 和厂商驱动都进一步分成了用户态和内核态两部分。这两部分里分别有自己的 DDI。
- 当程序调用 D3D API 的时候：
  - D3D runtime 会进行一些数据验证，经过用户态 DDI 到达厂商提供的用户态驱动 UMD。
  - UMD 把 shader 字节码编译成厂商专用的指令、转换命令队列等，传到内核态。
  - 内核里的 runtime 部分叫做 dxg kernel，做显存分配、设备中断管理等。
  - 经过内核态 DDI 调用厂商提供的内核态驱动 KMD，做一些地址翻译等厂商专用的操作，最后传给 GPU 执行。
- 这个架构称为 WDDM。

![Alt text](image-7.png)

- 把驱动分为用户态和内核态，并把大部分代码移到用户态，能大大提高稳定性。
- 又因为 D3D runtime 和 dxg kernel 这两个操作系统组件的存在，厂商开发驱动的过程从作文题直接变成了填空题，工作量大大减小。
- 这还使得不同厂商之间的区别变小，总体质量有所提升。
- 结果，Vista 之后因为驱动造成的蓝屏，远少于 XP 的时代。

![Alt text](image-8.png)

- D3D 有多个版本，目前 9、11、12 最常用。
- 每个版本之间的 API 和用户态 DDI 大不相同，代码没有多少兼容性。每出一版，程序和 UMD 都得大改甚至重写才能用上。

## OpenGL

![alt text](image-10.png)

- OpenGL 是一个跨平台（操作系统）且跨厂商（GPU）的图形 API，由 Khronos 发布。
- Khronos 只组织标准协商会议。API 支持的内容，还得看组织里的软硬件厂商。

windows 上：

![alt text](image-9.png)

- OpenGL 和微软自己的 D3D 存在直接竞争关系，以至于微软一直想方设法要在 Windows 上掐死 OpenGL，但多次尝试都因为用户的强烈反对而作罢。

![alt text](image-11.png)

- Windows 并没有为 OpenGL 做多少事情，只是提供了一个框架叫做可安装用户驱动，Installable Client Driver（ICD），让硬件厂商实现 OpenGL runtime 的 UMD。
- 到了内核态，也要经过 dxg kernel 和同一个 KMD。

Linux 上：
![alt text](image-13.png)

- OpenGL 有两种实现方式：
  - 一种是完全由厂商实现整个 OpenGL
  - 另一种方式是基于 Mesa 的框架。Mesa 提供了一个开源的 OpenGL runtime，并通过 DDI 调用厂商驱动来操作 GPU，后来进一步扩展出了对 OpenGL ES、Vulkan 等 API 的支持。

![alt text](image-14.png)

- OpenGL 从 90 年代初的 1.0 到最后一版 4.6，都是向下兼容的。当时的代码现在也能用。
- 不管在哪个平台上，OpenGL 本身都是一样的，只是和窗口打交道的部分略有不同。程序的平台适配并不难。
- OpenGL ES 甚至把窗口系统都给抽象出来，成为 EGL。进一步简化了跨平台。
  ![alt text](image-12.png)

![alt text](image-15.png)

- 注意，虽然 OpenGL 和 OpenGL ES 非常相似，但只要提到它们，几乎总是只涉及它们不同的部分。因此应该把它们当作两个不同的 API 来看待。

## CUDA

![alt text](image-16.png)

- CUDA 是跨平台的，但不跨厂商。正式来说只能在 NVIDIA 的 GPU 上跑。
- 这是一个只有计算的 API，需要的话可以和其他的图形 API 交互，把计算的结果交给图形流水线。
- 当然，更多时候 CUDA 被用来做纯计算，比如有限元模拟，神经网络训练等。

![alt text](image-17.png)

- CUDA 提供的一些功能并不存在于图形 API 里，并不是更高层的抽象。
- 比如 CUDA 一开始就提出了 shared memory 这个概念，用好的话可以显著提高 GPGPU 的效率。这在当时的图形 API 里是没有的，只能通过 CUDA。后来的 compute shader 也是受到 CUDA 的影响而设计出来的。

## API 对比与发展趋势

横向比较一下现在的 API，有这么一个规律：
![alt text](image-18.png)

- CUDA 和 Metal 这种从软件到硬件都是一个厂商拥有的 API，在硬件有了新功能之后，可以直接通过 API 暴露出来，不需要跟别的厂商讨论，反应很快。

![alt text](image-20.png)

- 而 OpenGL、OpenGL ES、Vulkan 这三个 API 的模式都是 Khronos 拥有接口，硬件厂商拥有实现。
- 在设计中使用了自底向上的方式。
- 它们都提供了扩展机制，可以在不更新 API 版本的情况下扩充 API。一个厂商的硬件有了个新功能，它只要提供一个扩展说明，写清楚如何使用，在驱动里实现这个扩展，程序就能用上这个新功能。
- 如果其他厂商对这个新功能也有兴趣，就能经过讨论升格成多厂商扩展，之后还能进一步升格成 Khronos 公认的扩展，最终进入新版本的 API。
- 这个流程，使得新功能可以让一部分人先用起来，在使用中逐步完善。

![alt text](image-19.png)

- 而 D3D 的模式是微软拥有接口和上层实现，硬件厂商拥有底层实现。
- 在设计中则是自顶向下的方式。
- 没有正式的扩展机制。流程由微软牵头，跟硬件厂商讨论新版本的 D3D 应该有什么新功能，接口应该怎么样。一旦定下来，就都定死了。即便又有了支持新功能的硬件，也至少得等到下一个版本的 D3D 才行，周期在 6 个月以上，甚至有过 3 年才出新版本的。
- 所以长期以来 D3D 对 GPU 新功能的支持往往慢一拍。

![alt text](image-21.png)

- 顺着时间纵向来看 API，可以看到它们发展趋势是变薄。把更多的事情交给程序去做，而不是 runtime 和驱动。
- 因为程序知道自己的意图，不需要让 API 去猜。这个改进的结果就是执行效率更高。
- 这几年出现的 D3D12 和 Vulkan，都是响应了这个趋势。这样的 API，显得更底层。而用它们来开发，更像是在写驱动，要做大量的细节操作。
- 不过一般来说 API 往上还有个渲染引擎的抽象层（RHI），可以把不同 API 抽象成同样的接口，这就把新 API 使用麻烦的缺点抹平了，同时获得新 API 带来的效率优势。

![alt text](image-22.png)

- 从前面说的分层架构可以看到，GPU 执行的是驱动发来的操作，并不知道来自于哪个 API。
- 所谓的 GPU 支持哪个 API，其实指的是 GPU 厂商提供了哪个 API 的驱动。所以说，GPU 支持什么 API 的什么功能，都取决于驱动。

![alt text](image-24.png)

- 以前出现过这种情况：NVIDIA GeForce 6800 硬件并不支持 32 位浮点混合，但它的 OpenGL 驱动说支持。
- 当程序用到这个功能的时候，驱动切换到软件模式，模拟实现浮点混合。仍然是个有效的实现，只是效率有严重损失。

![alt text](image-23.png)

- 另一方面，驱动和操作系统高度相关。即便是同一个 API，在不同操作系统上，驱动也完全不一样的。换一个操作系统就得重写一次驱动。

![Alt text](image-25.png)

- 比如高通的 Adreno GPU，在 Android 上支持 OpenGL ES，而在 Windows 上支持 D3D，就是因为它们在不同平台提供了不同的驱动。
- 并不能因为在 Android 上支持 OpenGL ES，就以为它们自然而然地也能在 Windows 上支持 OpenGL ES。这一点上，很多自以为是的人都翻过车。

## 非常规实现

### 翻译层

![Alt text](image-26.png)

- 正如前面所示，在整个架构里面，每一层做的事情就是往下翻译。
- 上下层之间通过接口隔离开来，上层并不需要知道下层是怎么做的。

![Alt text](image-27.png)

- 因此 API 不一定总是要往下到达 DDI，也可以翻译成另一个 API。

![Alt text](image-28.png)

- 比如 ANGLE 是个在 Windows 上最常用的 OpenGL ES 实现。
- 它只是个用户态的库，把 OpenGL ES 翻译成 D3D11、OpenGL、Vulkan 这些。这样就能在不改操作系统和驱动的情况下提供 OpenGL ES 的支持。

![Alt text](image-30.png)

- 同类的还有 MoltenVK，把 Vulkan 翻译成 Metal，解决苹果的平台不支持 Vulkan 的问题。

![Alt text](image-29.png)

- 还有更奇葩的 D3D11on12。它不是在上面加一个翻译层，而是用 D3D12 实现了一个 D3D11 的 UMD。
- 程序还是调用原来的 D3D11 runtime，但到了 UMD 之后往上返回到 D3D12 API，再往下走。

### 软件模拟 GPU

![Alt text](image-32.png)
![Alt text](image-31.png)

- 这样的驱动并不连到 GPU 硬件，而是在 CPU 上做了所有的事情。

![Alt text](image-33.png)

- Mesa 就内置了一个软件模拟的驱动。

![Alt text](image-34.png)

- D3D 也有一个，叫做 WARP。

CPU 模拟 GPU 可以辅助软硬件开发调试：

- 比如发明一个新功能，硬件还没做出来之前，可以进行模拟，以此定义功能的细节，作为硬件设计的参考。
- 在没有安装 GPU 的服务器上，软件模拟也可以临时用来跑一些 GPU 程序。

![Alt text](image-35.png)

- 驱动还可以把硬件操作发到另一台机器，远程执行。这就开启了虚拟 GPU 和云 GPU 的途径。

### Compositor

程序调用了图形 API，走完整个栈，让 GPU 渲染之后，就直接写入帧缓存吗？
![Alt text](image-36.png)

- 合成器，compositor，用来在同个系统中，多个程序同时运行时，指定谁写入帧缓存的特定位置，保证不冲突。

在不同系统上有不同组件来充当这个角色

![Alt text](image-37.png)

- Windows 上是 DWM

![Alt text](image-38.png)

- Android 上是 SurfaceFlinger。

![Alt text](image-40.png)
![Alt text](image-39.png)

- 每个程序渲染出自己的内容，不写入帧缓存，而是写入一张纹理。
- 这张纹理提交给操作系统，compositor 拿到之后，再次调用图形 API，把它们合成到帧缓存，然后实现到屏幕上。
- 这叫做窗口模式，每个程序有自己的窗口，所有程序共存于桌面。

![Alt text](image-41.png)

- 有的游戏会启用全屏独占模式，性能高一点，就是因为绕过了 compositor，直接到达帧缓存。
- 但因为独占了，其他窗口就没法显示，甚至输入法都可能显示不正常。

![Alt text](image-42.png)

- Compositor 在各个操作系统里都对程序透明，一般程序根本不需要知道它的存在。以至于在图形软件栈里，很少有提到 compositor 。
- 但在系统中，compositor 的意义是肉眼可见的。Windows 上的毛玻璃窗体、macOS 上的窗口动画特效这些，也都是 compositor 进行的渲染。

至此，GPU 的软硬栈已经补全，我们看到了从上到下的整条通路。
