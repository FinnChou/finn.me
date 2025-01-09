---
layout: post
title: "[番外]相位特性解卷绕"
date: 2013-06-02 18:19:42  +0800
tags: 数字信号处理
# categories: jekyll update
---

在分析FIR滤波器特性时,我们需要对其系数(单位冲击响应)进行离散时间傅里叶变换,表达式如下:

$$
\begin{aligned}
H(e^{j\omega}) &= \sum_{n=-\infty}^{+\infty}h(n)e^{-j \omega n} \\
\end{aligned}
$$

该变换结果即为滤波器的频率响应,可以分解为幅度响应和相位响应:

$$
\begin{aligned}
H(e^{j\omega}) &= |H(e^{j\omega})|e^{j\theta(\omega)} \\
\end{aligned}
$$

其中幅度响应和相位响应可通过以下公式计算:

$$
\begin{aligned}
|H(e^{j\omega})| &= \sqrt{\Big[Re\big(H(e^{j\omega})\big)\Big]^2 + \Big[Im\big(H(e^{j\omega})\big)\Big]^2} \\
\theta(\omega) &= \tan^{-1}\left(\frac{Im\big(H(e^{j\omega})\big)}{Re\big(H(e^{j\omega})\big)}\right) \\
\end{aligned}
$$

对我们之前设计的FIR滤波器进行相位特性分析,得到如下图所示结果:

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_1.jpeg" width="300"></div>

理论上,FIR滤波器应具有线性相位特性,但图中显示的相位响应并不呈现线性。这是由于反正切函数的计算特性导致的。以一段从$-120°$到$-225°$的相位变化为例:

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_2.jpeg" width="300"></div>

由于反正切函数的输出范围限制在$[-\pi, \pi]$区间内,当相位角度跨越实轴时,$-225°$会被计算为$135°$,产生$2\pi$的跳变。这种跳变在相位响应中表现为:

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_3.jpeg" width="300"></div>

通过上述分析，我们可以理解到相位响应中出现不规则现象的根本原因。尽管FIR滤波器具有线性相位特性，但由于反正切函数计算范围的限制，导致相位响应呈现出不连续的特征。为了恢复真实的线性相位特性，需要对相位响应进行解卷绕(Phase Unwrapping)处理，即补偿反正切函数计算过程中丢失的$2\pi$项。在MATLAB环境中，可以通过内置的unwrap()函数实现相位解卷绕，具体实现代码如下：

```matlab
w = 0:0.01:pi;
H = freqz(h,1,w);
figure;
plot(w,unwrap(angle(H)));
axis([0 pi -35 5]);grid; 
xlabel('Frequency \omega [rad]');
ylabel('\theta [rad]');
```

在获得解卷绕后的相位响应后，我们可以清晰地观察到FIR滤波器的线性相位特性，如下图所示：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_4.jpeg" width="300"></div>

然而，在相位响应中仍然存在一些需要关注的现象。特别是在红色标记区域，相位响应出现了明显的震荡。为了深入分析这一现象的成因，我们需要同时考察滤波器的幅频特性和相位特性。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_5.jpeg" width="3450"></div>

通过对比分析可以发现，相位响应的震荡主要出现在截止频率附近。这与窗函数设计法固有的特性有关。由于窗函数设计的FIR滤波器在通带和阻带都存在纹波，这些纹波直接影响了相位特性。在幅频特性图中，由于采用了对数坐标，阻带纹波表现得尤为明显。

为了更直观地理解这一现象，我们可以将离散时间傅里叶变换的结果映射到复平面上进行分析：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_6.jpeg" width="300"></div>

在复平面表示中，最外层的单位圆对应于通带特性，其中振幅接近1。随着频率增加，响应点逐渐向圆心收敛，形成过渡带区域。理想情况下，阻带对应于复平面原点，表示完全衰减。

然而，由于滤波器的非理想特性，实际响应会出现纹波。我们通过放大观察可以更清楚地看到这一现象：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_7.jpeg" width="300"></div>

在阻带区域，响应轨迹呈现出从第三象限经过原点到第一象限的特征。这种跨越导致了相位的$\pi$跳变，而不是之前讨论的$2\pi$跳变。为了更好地说明这一点，我们在复平面上标注了相关区域：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_8.jpeg" width="300"></div>

这种深入的分析帮助我们更好地理解了FIR滤波器的相位特性及其与幅度响应之间的关联。

### 相位特性的深入分析
在复平面分析中，理想滤波器的阻带对应于原点。然而，这引出了一个值得探讨的问题。回顾相位特性的计算公式：

$$
\begin{aligned}
|H(e^{j\omega})| &= \sqrt{\Big[Re\big(H(e^{j\omega})\big)\Big]^2 + \Big[Im\big(H(e^{j\omega})\big)\Big]^2} \\
\theta(\omega) &= \tan^{-1}\left(\frac{Im\big(H(e^{j\omega})\big)}{Re\big(H(e^{j\omega})\big)}\right) \\
\end{aligned}
$$

从数学角度分析，当频率响应趋近于原点时，相位计算会出现不确定性。这表明在阻带区域，相位特性的物理意义需要重新审视。实际上，我们主要关注通带区域的相位特性。

从信号处理的角度来看，这一现象可以通过延迟特性来理解。在低通滤波器中，输入信号的频率与输出延迟呈正相关。当信号频率超过截止频率进入阻带时，理论上输入与输出之间的延迟趋于无穷，这与阻带的衰减特性相符。

需要说明的是，这种解释是基于我个人的理解下的推理分析，目前没有进行过严谨的研究和推定，仍需进一步的理论验证和实验支持。
