---
title: Ocean waves simulation with Fast Fourier transform
tag:
  - game
  - water
  - ocean
mathjax: true
category:
 - Realtime Water Rendering
sortValue: 10002
---

- [Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)
- [FFT 和 Oceanographic Spectrum 方法的论文](https://people.computing.clemson.edu/~jtessen/reports/papers_files/coursenotes2004.pdf)
- [Source Code](https://github.com/gasgiant/FFT-Ocean)
- [catlikecoding](https://catlikecoding.com/unity/tutorials/flow/waves/)
- [Ocean Simulation](https://dev.epicgames.com/community/learning/tutorials/qM1o/unreal-engine-ocean-simulation)

这个方法的核心是 FFT 和 Oceanographic Spectrum：
![Alt text](image.png)

这里省去关于 sine wave 的介绍，根据线性波理论，波浪是 Gerstner Wave 的叠加，表面上的每个点沿着一个圆形轨迹运动：
![Alt text](image-1.png)
振幅太大的时候会形成重叠，所以应该设小一点：
![Alt text](image-2.png)
![Alt text](image-3.png)
所以实际上要做的就一件事，就是把这些波叠加起来。比起摆弄参数，更好的方法是使用快速傅里叶变换（FFT）和 Oceanographic Spectrum 使用真实的海洋数据来模拟波浪。

## How Ocean Waves Work in Unreal Engine: FFT & Wave Simulation

https://www.youtube.com/watch?v=OWiyIc2bVwM

![Alt text](image-4.png)

statistical wave models 描述了海洋波浪的统计特性：
![Alt text](image-5.png)

FFT 是一种高效计算 DFT 的方法：
![Alt text](image-6.png)

Cooley-Tukey 算法把 N 个数据点分为奇数和偶数两组，然后递归计算每一半的 DFT，然后组合起来：
![Alt text](image-7.png)

Phillips Spectrum 是一个常用的频谱模型，表示风产生的海洋波浪高度的统计分布表示：
![Alt text](image-8.png)

共轭项表示时域上延两个方向传播的波：
![Alt text](image-9.png)

用 Cascade 表示不同分辨率的模拟：
![Alt text](image-10.png)

![Alt text](image-11.png)
用两个 RT 存储 WPO：
![Alt text](image-12.png)

## UE4_FFT_Ocean

[UE4_FFT_Ocean](https://github.com/tigershan1130/UE4_FFT_Ocean)

![Alt text](image-13.png)
![Alt text](image-14.png)

这段代码描述了一个使用Compute Shader进行逆快速傅里叶变换（IFFT）的流程，通常用于计算物理模拟中的波的传播，如水面波动等。每个"pass"代表一个计算阶段，下面是每个pass的详细解释：

### 0. Compute Twiddle Indices for IFFT
- **目的**：计算IFFT中使用的Twiddle因子（旋转因子）。
- **操作**：如果初始数据未生成，则运行`FTwiddleFactorPass` Compute Shader来生成Twiddle因子，并标记初始数据已生成。

### 1. Compute HZero Pass
- **目的**：计算IFFT的H0（零阶）传递。
- **操作**：运行`FHZeroPass` Compute Shader，这是IFFT计算的第一步，通常涉及对输入数据的预处理。

### 2. Compute H0 Initial Spectrum Pass
- **目的**：计算IFFT的H0初始频谱。
- **操作**：如果存在H0的输出资源，则运行`FConjSpectrumPass` Compute Shader来计算H0的共轭频谱。

### 3. Frequency Spectrum Update Pass
- **目的**：更新IFFT的频率谱。
- **操作**：如果存在H0和预计算波的输出资源，则运行`FFreqSpectrumPass` Compute Shader来更新频率谱。

### 4. Calculate IFFT Complex Amplitudes
- **目的**：计算IFFT复振幅的导数。
- **操作**：包括四个子步骤，分别计算DxDz、DyDxz、DyxDyz和DxxDzz，这些是复振幅的二阶导数，用于模拟波的传播。

### 5. Final Wave Compute Pass
- **目的**：计算最终的波形。
- **操作**：如果所有必要的导数资源都存在，则运行`FFinalWaveComputePass` Compute Shader来合成最终的波形。

### 6. Displacement Render Target
- **目的**：将计算出的位移数据渲染到目标纹理。
- **操作**：如果存在位移渲染目标和输出资源，则使用`FCopyTexturePixelShader`将最终波形的位移数据复制到渲染目标。

### 7. Derivatives Render Target
- **目的**：将计算出的导数数据渲染到目标纹理。
- **操作**：如果存在导数渲染目标和输出资源，则使用`FCopyTexturePixelShader`将最终波形的导数数据复制到渲染目标。

### 8. Fold Render Target
- **目的**：将计算出的湍流数据渲染到目标纹理。
- **操作**：如果存在湍流渲染目标和输出资源，则使用`FCopyTexturePixelShader`将最终波形的湍流数据复制到渲染目标。

这些pass共同构成了一个完整的IFFT计算流程，从生成Twiddle因子开始，到计算最终的波形和相关的物理数据，最后将这些数据渲染到不同的目标纹理中，以供后续的渲染或进一步的计算使用。这个过程充分利用了GPU的并行计算能力，以高效地处理复杂的物理模拟。
