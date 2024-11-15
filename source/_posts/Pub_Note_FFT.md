---
title: Fourier Transform
mathjax: true
category:
 - Mathematics
sortValue: 100002
---

- [三角函数的正交性及其公式推导](https://blog.csdn.net/zhaohongfei_358/article/details/118461151)
- [三角恒等式](https://zh.wikipedia.org/wiki/%E4%B8%89%E8%A7%92%E6%81%92%E7%AD%89%E5%BC%8F#%E8%A7%92%E7%9A%84%E5%92%8C%E5%B7%AE%E6%81%86%E7%AD%89%E5%BC%8F)
- [如果看了此文你还不懂傅里叶变换，那就过来掐死我吧](https://www.zhihu.com/search?type=content&q=%E5%82%85%E9%87%8C%E5%8F%B6%E5%8F%98%E6%8D%A2)
- [傅里叶变换一步步详细推导](https://blog.csdn.net/wd18508423052/article/details/100940771)
- [Games001-傅里叶变换与球谐函数](https://www.bilibili.com/video/BV1MF4m1V7e3)
- [快速理解 FFT 算法（完整无废话）](https://zhuanlan.zhihu.com/p/407885496)
- [FFT 蝶形图](https://www.zhihu.com/question/20456490/answer/25440654)
- [Python Numerical Methods](https://pythonnumericalmethods.studentorg.berkeley.edu/notebooks/chapter24.02-Discrete-Fourier-Transform.html)
- [库利-图基快速傅里叶变换算法](https://zh.wikipedia.org/wiki/%E5%BA%93%E5%88%A9-%E5%9B%BE%E5%9F%BA%E5%BF%AB%E9%80%9F%E5%82%85%E9%87%8C%E5%8F%B6%E5%8F%98%E6%8D%A2%E7%AE%97%E6%B3%95)

## Prerequisite

### 积化和差公式

$$ sin(a)cos(b) = \frac{1}{2} [sin(a+b) + sin(a-b)] $$
$$ cos(a)sin(b) = \frac{1}{2} [sin(a+b) - sin(a-b)] $$
$$ cos(a)cos(b) = \frac{1}{2} [cos(a+b) + cos(a-b)] $$
$$ sin(a)sin(b) = -\frac{1}{2} [cos(a+b) - cos(a-b)]$$

### 和角公式

![alt text](image-2.png)
$$ sin(a+b) = sin(a)cos(b) + cos(a)sin(b) $$
$$ cos(a+b) = cos(a)cos(b) - sin(a)sin(b) $$
$$ sin(a-b) = sin(a)cos(b) - cos(a)sin(b) $$
$$ cos(a-b) = cos(a)cos(b) + sin(a)sin(b) $$

### 三角函数系

{1, sin(x), cos(x), sin(2x), cos(2x), ...}，他们具有一个很好的性质：正交性。也就是说，如果两个不同的三角函数相乘，然后在一个周期内积分，那么结果为 0。也就是：
$$ \int_{-\pi}^{\pi} sin(nx)sin(mx)dx = 0, n \neq m, n, m \in Z $$
$$ \int_{-\pi}^{\pi} cos(nx)cos(mx)dx = 0, n \neq m, n, m \in Z $$
$$ \int_{-\pi}^{\pi} sin(nx)cos(mx)dx = 0, n, m \in Z $$

### 欧拉公式

$$ e^{ix} = cos(x) + isin(x) $$

### 冲激函数（Dirac函数）

$$ \delta(x) = \begin{cases} +\infty, x = 0 \\ 0, x \neq 0 \end{cases} $$
$$ \int_{-\infty}^{\infty} \delta(x)dx = 1 $$
![alt text](image.png)

Dirac函数有个特殊的性质：
$$ \int_{-\infty}^{\infty} f(t)\delta(t-t_0)dt = f(t_0) $$
因此，Dirac函数常用于采样，不过由于它每次只能选择一个函数值，所以我们通常将多个Dirac函数叠加，得到一个梳状函数：
$$ \delta_s(t) = \sum_{n=-\infty}^{\infty} \delta(t-nT_s) $$
其中，$T_s$是采样周期。
采样可以表示为：
$$ f_s = \sum_{n=-\infty}^{\infty} f(t)\delta(t-nT_s) $$

## Thinking

1. 根据傅里叶级数一个周期函数可以展开为基频为w、频率为nw、振幅分别为a_n和b_n的sin和cos函数之和的累计。其中a_n和b_n跟n有关，频率越大幅值越小，他们也都是可以表示出来。
   这就引入了一个频域的视角，此时自变量是n，因变量是a_n和b_n。
2. 我们有两个因变量，通过欧拉公式，可以把傅里叶展开写成复数的形式。
3. 此时我们的频率是通过nw来表示的，所以实际上在频域上是一个离散的函数。在时域上是周期函数，我们再考虑上非周期函数，那么T趋于无穷，delta w趋于0，这样nw就变为连续了。也就是说时域上的非周期函数，在频域上就是连续的函数，反之是离散的。
4. 在连续上我们直接把nw写作W，原本的求和也写作积分，我们把dW前面影响幅值的那部分提出来叫做F(W)，其实它表示的也就是幅值在频域中的分布密度。这个F(W)可以代表函数在频域中的表示。用f(t)求F(W)就是傅里叶变换。用F(W)求f(t)就是傅里叶逆变换。
5. 前面说完了FT，下面来说一下DFT。对于计算机来说，函数在时域上是离散的，我们需要进行采样。所以引入了冲激函数（也就是Dirac函数），为了进行多次采样，我们引入采样周期，把Dirac函数进行叠加得到一个梳状函数。这样我们就可以表示出这个离散采样后的函数。
6. 除了离散，我们采样的次数是有限的，比如说我们只采样N次，那么我们也只涉及到了一部分作用域。为了将其扩展到整个作用域，我们进行周期延拓，也就是把这个函数进行复制表示为周期函数。这个时域上的周期函数，到了频域就使得原本连续的函数变为了离散的函数。这样我们得到了离散版本的傅里叶变换，也就是DFT。
7. DFT的复杂度是O(n^2)，为了提高效率，我们引入了FFT。我们将时间取值拆分为偶数项和奇数项，根据其中公共项的周期性，我们可以发现某些时间取值得到的频域密度的偶数项和奇数项是重复的。并且偶数项和奇数项是一个N更小的DFT，这样我们就能够通过递归分治的方式来降低复杂度。FFT的复杂度是O(nlogn)。

## Fourier Series（傅里叶级数）

周期函数：
$$f(t) = f(t+T)$$
傅里叶级数就是说一个周期函数在周期内可以展开为无数多个三角函数之和来表示：
$$ f(t) = a_0 + \sum_{n=1}^{\infty} [a_n cos(n\omega t) + b_n sin(n\omega t)] $$
其中：$a_n$ 和 $b_n$表示振幅，$\omega$表示基频，$n\omega$表示频率。其中$a_n$和$b_n$跟n有关，频率越大幅值越小。他们也都是可以表示出来的。
![alt text](image-4.png)

傅里叶级数带来了一个新的关于频域的视角，此时自变量是n，因变量是$a_n$和$b_n$。
![alt text](image-5.png)
![alt text](image-1.png)
（注意，频谱的振幅实际上是两个数，这里只表示了其中一个。）

我们有两个因变量，可以通过欧拉公式，可以把傅里叶展开写成复数的形式：
$$ e^{in\omega t} = cos(n\omega t) + isin(n\omega t) $$
$$ e^{-in\omega t} = cos(n\omega t) - isin(n\omega t) $$
则：
$$ cos(n\omega t) = \frac{1}{2} [e^{in\omega t} + e^{-in\omega t}] $$
$$ sin(n\omega t) = \frac{1}{2i} [e^{in\omega t} - e^{-in\omega t}] $$
带入傅里叶级数：
$$ f(t) = a_0 + \sum_{n=1}^{\infty} [a_n \frac{1}{2} [e^{in\omega t} + e^{-in\omega t}] + b_n \frac{1}{2i} [e^{in\omega t} - e^{-in\omega t}]] $$
$$ f(t) = a_0 + \sum_{n=1}^{\infty} [\frac{a_n - ib_n}{2} e^{in\omega t} + \frac{a_n + ib_n}{2} e^{-in\omega t}] $$
把$a_0$也写成类似的形式：
$$ f(t) = \sum_{n=0}^{0} a_n e^{in\omega t} + \sum_{n=1}^{\infty} [\frac{a_n - ib_n}{2} e^{in\omega t} + \frac{a_n + ib_n}{2} e^{-in\omega t}] $$
$$ f(t) = \sum_{n=0}^{0} a_n e^{in\omega t} + \sum_{n=1}^{\infty} [\frac{a_n - ib_n}{2} e^{in\omega t}] + \sum_{n=-1}^{-\infty} [\frac{a_{-n} + ib_{-n}}{2} e^{in\omega t}] $$
所以：
$$ f(t) = \sum_{n=-\infty}^{\infty} c_n e^{in\omega t} $$
其中：
$$ c_n = \frac{a_n - ib_n}{2}, n = 1, 2, 3, ... $$
$$ c_n = \frac{a_{-n} + ib_{-n}}{2}, n = -1, -2, -3, ... $$
$$ c_0 = a_0, n = 0 $$
下面我们再来推导一下$a_0$、$a_n$和$b_n$是多少：
$$ f(t) = a_0 + \sum_{n=1}^{\infty} [a_n cos(n\omega t) + b_n sin(n\omega t)] $$
我们两边同时乘上cos(0t)，也就是1，然后在一个周期内积分：
$$ \int_{0}^{T} f(t) cos(0t) dt = \int_{0}^{T} a_0 cos(0t) dt + \sum_{n=1}^{\infty} [a_n \int_{0}^{T} cos(n\omega t) cos(0t) dt + b_n \int_{0}^{T} sin(n\omega t) cos(0t) dt] $$
根据前面三角系的正交性，我们可以消去后面的项，得到：
$$ \int_{0}^{T} f(t) dt = \int_{0}^{T} a_0 dt = a_0 T $$
所以：
$$ a_0 = \frac{1}{T} \int_{0}^{T} f(t) dt $$

用类似的方式对$n = 1, 2, 3, ...$进行处理，两边同时乘上$cos(m\omega t)$，然后在一个周期内积分：
$$f(t) = a_0 + \sum_{n=1}^{\infty} [a_n cos(n\omega t) + b_n sin(n\omega t)]$$
$$\int_{0}^{T} f(t) cos(m\omega t) dt = \int_{0}^{T} a_0 cos(0t) cos(m\omega t) dt + \sum_{n=1}^{\infty} [a_n \int_{0}^{T} cos(n\omega t) cos(m\omega t) dt + b_n \int_{0}^{T} sin(n\omega t) cos(m\omega t) dt]$$
利用正交性，只有$cos(n\omega t) cos(m\omega t)$项在n=m时不为0，所以：（中间用积化和差公式，后边那项是2m倍的cos的频率，其中m为正整数，所以积分为0）
$$\int_{0}^{T} f(t) cos(m\omega t) dt = \int_{0}^{T} a_m cos(m\omega t)^2 dt = a_m \int_{0}^{T} \frac{1}{2} (1 + cos(2m\omega t)) dt = a_m \frac{T}{2} = a_n \frac{T}{2}$$
所以：
$$a_n = \frac{2}{T} \int_{0}^{T} f(t) cos(n\omega t) dt, n = 1, 2, 3, ...$$
$b_n$同理：
$$b_n = \frac{2}{T} \int_{0}^{T} f(t) sin(n\omega t) dt, n = 1, 2, 3, ...$$
把$a_0$、$a_n$和$b_n$带入$c_n$的公式：
n = 0：
$$c_n = a_0 = \frac{1}{T} \int_{0}^{T} f(t) dt = \frac{1}{T} \int_{0}^{T} f(t) e^{-i\omega0} dt$$
n = 1, 2, 3, ...：
$$c_n = \frac{1}{2} (a_n - ib_n) = \frac{1}{2} (\frac{2}{T} \int_{0}^{T} f(t) cos(n\omega t) dt - i \frac{2}{T} \int_{0}^{T} f(t) sin(n\omega t) dt) $$
$$= \frac{1}{T} \int_{0}^{T} f(t) [cos(n\omega t) - isin(n\omega t)] dt = \frac{1}{T} \int_{0}^{T} f(t) e^{-in\omega t} dt$$
n = -1, -2, -3, ...：
$$c_n = \frac{1}{2} (a_{-n} + ib_{-n}) = \frac{1}{2} (\frac{2}{T} \int_{0}^{T} f(t) cos(-n\omega t) dt + i \frac{2}{T} \int_{0}^{T} f(t) sin(-n\omega t) dt) $$
$$= \frac{1}{T} \int_{0}^{T} f(t) [cos(n\omega t) - isin(n\omega t)] dt = \frac{1}{T} \int_{0}^{T} f(t) e^{in\omega t} dt$$
我们发现不管n是正还是负，$c_n$都使用同一个表示，最终：
$$f(t) = \sum_{n=-\infty}^{\infty} c_n e^{in\omega t}$$
其中：
$$c_n = \frac{1}{T} \int_{0}^{T} f(t) e^{-in\omega t} dt$$

## Fourier Transform（傅里叶变换）

自此我们已经得到周期函数的傅里叶级数展开式，此时在频域上自变量是$n\omega$，其中n是整数，所以这实际上是一个离散函数。

下一步我们将这个转换扩展到非周期函数：

对于非周期函数来说，我们可以将其看作是$T \to \infty$，也就是$\delta \omega \to 0$，这样自变量$n\omega$就变成了连续的，换句话说非周期函数的频域表示是连续函数。
$$f(t) = \lim_{T \to \infty} \sum_{n=-\infty}^{\infty} c_n e^{in\omega t} = \lim_{T \to \infty} \sum_{n=-\infty}^{\infty} \frac{1}{T} \int_{0}^{T} f(t) e^{-in\omega t} dt e^{in\omega t}$$
$$= \lim_{\delta \omega \to 0} \sum_{n=-\infty}^{\infty} \frac{\delta \omega}{2\pi} \int_{-\infty}^{\infty} f(t) e^{-in\omega t} dt e^{in\omega t}$$
$n \omega $ 作为一个连续量我们将其写作 $W$，再把求和写成积分：
$$f(t) = \int_{-\infty}^{\infty} \frac{1}{2\pi}( \int_{-\infty}^{\infty} f(t) e^{-iWt} dt) e^{iWt} dW$$
其中, $\int_{-\infty}^{\infty} f(t) e^{-iWt} dt$ 是关于$W$的函数，我们将其记作$F(W)$（实际上这个函数就可以表示函数在频域上的分布密度）。
这样我们就得到了非周期函数的傅里叶变换：
$$F(W) = \int_{-\infty}^{\infty} f(t) e^{-iWt} dt$$
傅里叶变换的逆变换：
$$f(t) = \frac{1}{2\pi}\int_{-\infty}^{\infty} F(W) e^{iWt} dW$$

## Discrete Fourier Transform（离散傅里叶变换）

在前面FT中我们涉及到的时域函数是连续的，但计算机采集的信号在时域上是离散的，我们需要引入冲激函数（Dirac函数）来表示这个离散采样后的函数：
$$F_s(W) = \int_{-\infty}^{\infty} (\sum_{n=-\infty}^{\infty} f(t)\delta(t-nT_s)) e^{-iWt} dt$$
根据Dirac函数的性质，也可以写作：
$$F_s(W) = \sum_{n=-\infty}^{\infty} f(nT_s)e^{-iWnT_s}$$

计算机上采集的信号不仅是离散的，而且是有限的，比如我们采样N个点，这N个点只涉及到了一部分作用域。为了将其扩展到整个作用域，我们需要进行周期延拓，也就是把这个函数进行复制，表示为周期函数。
原本函数采样周期为$T_s$，我们采样N次，那么整个需要做复制的函数的周期就是$T_o = NT_s$。
也就是说对于计算机信号来说，我们需要处理的是一个周期函数，因此我们又得重新回到傅里叶级数展开：
$$F(k\omega_o) = \frac{1}{T_o} \int_{0}^{T_o} (\sum_{n=0}^{N-1} f(t)\delta(t-nT_s)) e^{-ik\omega_o t} dt$$
前面用了n，这里用k区别一下。
根据Dirac函数的性质，我们可以得到：
$$F(k\omega_o) = \frac{1}{T_o} \sum_{n=0}^{N-1} f(nT_s)e^{-ik\omega_o nT_s}$$
把$\omega_o$和$T_o$都用$T_s$和N表示：
$$F(k\omega_o) = \frac{1}{NT_s} \sum_{n=0}^{N-1} f(nT_s)e^{-ik\frac{2\pi}{NT_s} nT_s}$$
写的更简洁一点，令$F[k] = F(k\omega_o)T_s$，$f[n] = f(nT_s)$：
$$F[k] = \frac{1}{N} \sum_{n=0}^{N-1} f[n]e^{-i\frac{2\pi}{N} kn}$$
这样我们就得到了离散傅里叶变换（DFT），下面我们来看一下DFT的计算复杂度：
对于每个k，我们都需要计算N次$f[n]$和$e^{-i\frac{2\pi}{N} kn}$的乘积，然后做N-1次加法，所以对于单个k来说时间复杂度是O(N)。

时域上N个采样的信号，到了频域会分别分解为$\frac{1}{N}+1$个cos和sin，为什么是$\frac{1}{N}+1$呢？
![alt text](image-3.png)
![alt text](image-6.png)
可以看到最后，频域上再取更多的k已经没有意义了，因为此时的采样周期已经比信号周期都要大了。
所以最后整个DFT的复杂度是O(N^2)。

## Fast Fourier Transform（快速傅里叶变换）

重新把DFT写一遍：
$$F[k] = \frac{1}{N} \sum_{n=0}^{N-1} f[n]e^{-i\frac{2\pi}{N} kn}$$
写的更简洁一点，令$W_N = e^{-i\frac{2\pi}{N}}$：
$$F[k] = \frac{1}{N} \sum_{n=0}^{N-1} f[n]W_N^{kn}$$
FFT的思路是把信号样本分为偶数部分和奇数部分，这里我们用2n表示偶数部分，2n+1表示奇数部分：
$$F[k] = \frac{1}{N} \sum_{n=0}^{N/2-1} f[2n]W_N^{k(2n)} + \frac{1}{N} \sum_{n=0}^{N/2-1} f[2n+1]W_N^{k(2n+1)}$$
把奇数项指数部分的1提出来：
$$F[k] = \frac{1}{N} \sum_{n=0}^{N/2-1} f[2n]W_N^{2kn} + W_N^k \frac{1}{N} \sum_{n=0}^{N/2-1} f[2n+1]W_N^{2kn}$$
因为：
$$W_N^{2kn} = e^{-i\frac{2\pi}{N} 2kn} = e^{-i\frac{2\pi}{N/2} kn} = W_{N/2}^{kn}$$
所以：
$$F[k] = \frac{1}{N} (\sum_{n=0}^{N/2-1} f[2n]W_{N/2}^{kn} + W_N^k \sum_{n=0}^{N/2-1} f[2n+1]W_{N/2}^{kn})$$
其中$f[2n]$和$f[2n+1]$是已知的，对比$F[k]$的形式，我们可以发现奇数项和偶数项就是两个N/2更小的DFT。我们把它们提出来，写作：
$$E[k] = \sum_{n=0}^{N/2-1} f[2n]W_{N/2}^{kn}$$
$$O[k] = \sum_{n=0}^{N/2-1} f[2n+1]W_{N/2}^{kn}$$
所以：
$$F[k] = \frac{1}{N} (E[k] + W_N^k O[k])$$
$W_N$，我们通常叫做Twiddle Factor（旋转因子），它有很有趣的周期性：
$$W_N^{N/2} = e^{-i\pi} = -1$$
$$W_N^{N} = e^{-i2\pi} = 1$$
所以：
$$W_N^{k+N/2} = W_N^k W_N^{N/2} = -W_N^k$$
$$W_N^{k+N} = W_N^k W_N^N = W_N^k$$
因此：
$$E[k] = E[k+N/2]$$
$$O[k] = O[k+N/2]$$
由此可得：
$$F[k+N/2] = \frac{1}{N} (E[k+N/2] + W_N^{k+N/2} O[k+N/2]) = \frac{1}{N} (E[k] - W_N^k O[k])$$
这样F[k]和F[k+N/2]的关系看上去就很清晰了，只要把偶数项和奇数项分别计算出来，就能同时得到F[k]和F[k+N/2]，这就把计算量减少了一半。
另外，前面提到了$E[k]$和$O[k]$是两个更小的DFT，所以我们可以继续递归分治，直到N=1。