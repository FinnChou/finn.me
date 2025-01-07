---
layout: post
title: "2.使用窗函数设计FIR滤波器"
date: 2013-05-24 14:25:02 +0800
tags: 数字信号处理
# categories: jekyll update
---
&nbsp;
## 滤波器设计参数
在开始设计滤波器之前，首先需要理解几个关键概念：通带、阻带、过渡带、通带纹波和阻带纹波。请参考下图：

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter-Window-Function/Design-FIR-Filter-1.jpeg" width="400"></div>

&nbsp;
- $[0, \omega_p]$ 这个范围被称为通带。
- $[1-\delta_p, 1+\delta_p]$ 这个范围表示允许的误差，称为通带纹波。
- $[\omega_s, \pi]$ 这个范围被称为阻带。
- $[0, \delta_s]$ 这个范围称为阻带纹波。
- 中间的黑色部分是过渡带，角频率 $\omega_p [rad]$ 被称为通带边缘频率，而角频率 $\omega_s [rad]$ 则被称为阻带起始频率。

在滤波器的设计过程中，通常会明确这些参数的要求，最终设计的滤波器必须满足这些条件。需要注意的是，这里举的例子是低通滤波器，对于高通或带通滤波器，这些参数的定义则相反。此外，设计过程中应考虑实际应用场景，以确保滤波器性能的最佳化。

&nbsp;
## 理想FIR低通滤波器
首先，先由理想低通滤波器为出发点开始考虑。理想低通滤波器的频响如下所示

$$
\begin{aligned}

H(e^{(j\omega)}) = \left \{
\begin{array}{l}
1, \hspace{3mm} |\omega| \leq \omega_c \\
0, \hspace{3mm} otherwide \\
\end{array}
\right.

\end{aligned}
$$

这里的 $\omega_c [rad]=\frac{\omega_p+\omega_s}{2}$ 表示截止频率。

我们首先从理想滤波器出发，求解其单位冲击响应。通过获得单位冲击响应，我们可以确定滤波器的系数，从而设计出一个理想的滤波器。这是一个完美的构思，现在让我们开始动手，寻找它的单位冲击响应。运用离散时间傅里叶逆变换，我们可以得到如下公式。

$$
\begin{aligned}
h_d(n) &= \frac{1}{2\pi} \int_{-\omega_c}^{\omega_c} 1 \cdot e^{j\omega n} d\omega \\
       &= \frac{1}{2\pi} \cdot \frac{1}{jn} \int_{-\omega_c}^{\omega_c} e^{j\omega n} d\omega \\
       &= \frac{1}{n\pi} \cdot \frac{e^{j \omega_c n} - e^{-j \omega_c n}}{2j} \\ 
       &= \frac{sin(\omega_c n)}{\pi n} \\
       &= \frac{\omega_c}{\pi} sinc(\frac{\omega_c}{\pi}n), \hspace{3mm} n \in (-\infty, +\infty)
\end{aligned}
$$

通过上述分析，我们得到了单位冲击响应的表达式（sinc函数即为辛格函数）。这是否意味着我们可以设计出一个理想的滤波器呢？让我们再仔细确认一下。

首先，这个表达式是离散的。单位冲击响应本身就是离散的，这一点没有问题。因此，我们距离理想滤波器又迈进了一步。其次，这个表达式所求出的单位冲击响应的数量，遗憾的是，它是无限的。至此，我们基本可以得出结论：理想滤波器是无法实现的。

尽管理想滤波器无法实现，我们仍然可以采取折中方案。从无限的理想滤波器单位冲击响应中选择一部分，构建一个不那么理想但能满足一定标准的滤波器。我们只能“将就”使用这个不太理想的滤波器。那么，接下来我们需要解决一个问题：如何从无限的数列中选取有限的一部分，以满足我们的设计要求呢？

&nbsp;
## 窗函数
首先，我们将从最简单的情况入手。理想单位冲击响应的形状与高斯分布非常相似（当然，这只是一个近似）。在$n=0$时，单位冲击响应的值达到最大，随后两侧的值逐渐减小，可能还会出现负值。因此，为了使滤波器的性能尽可能接近理想滤波器，我们选择保留其最主要的部分，即值较大的部分，而将其余部分舍弃。基于以上描述，我们可以使用以下公式进行表示。


$$
\begin{aligned}

\omega(n) = \left \{
\begin{array}{l}
1, \hspace{3mm} |n| \leq \frac{N-1}{2} \\
0, \hspace{3mm} otherwide \\
\end{array}
\right.

\end{aligned}
$$

这个公式确实可以帮助我们选择一部分有限的数列。这个公式被称为矩形窗。可以看出，$N$越大，性能越接近理想滤波器。在这里，$N$被称为窗函数的长度。

这就是我们选定的窗口，接下来我们将窗口函数应用于我们的数据，也就是进行加窗处理！实际上，这只是一个乘法操作，如下所示。


$$
\begin{aligned}
h_d(n) &= h_d(n) \cdot \omega(n) \\
       &= \frac{\omega_c}{\pi} sinc(\frac{\omega_c}{\pi}n) \cdot \omega(n), \hspace{3mm} n \in \left(-\frac{N-1}{2}, +\frac{N-1}{2} \right)
\end{aligned}
$$

至此，就完成了加窗的步骤。
然而，在实际应用中，矩形窗的使用相对较少。这主要是因为矩形窗的阻带衰减不足，仅为$21[dB]$。因此，许多不同类型的窗函数应运而生，各具特色。在我们初步的设计学习中，阻带衰减是我们关注的主要指标。以下是各种窗函数的性能比较。

| 窗函数       | 过渡带大小    |  阻带衰减   |
| ----------- | ----------- |----------- |
|   矩形窗     | $1.8\pi/N$  |   $21[dB]$ |
|   汉宁窗     | $6.2\pi/N$  |   $44[dB]$ |
|   汉明窗     | $6.6\pi/N$  |   $58[dB]$ |
| 布莱克曼窗    | $11\pi/N$  |   $74[dB]$ |


&nbsp;
## 用窗函数实现一个FIR滤波器
在之前的讨论中，我们详细探讨了窗函数的原理，现在让我们整理一下设计步骤。

1. 根据设计规格和参数要求，选择一个合适的窗函数。此时，我们主要关注阻带衰减，确保其大于给定值。如果没有提供阻带衰减的具体数值，我们需要通过通带纹波和阻带纹波来进行计算，如下式所示。

$$
\begin{aligned}
R_p &= -20 \cdot log_{10}\frac{1  - \delta_p}{1  + \delta_p} \\
A_s &= -20 \cdot log_{10}\frac{\delta_s}{1  + \delta_p} \\
\end{aligned}
$$

2. 根据要求的通带边缘频率和阻带起始频率，计算过渡区的大小，从而确定窗函数的长度。
3. 最后，依据窗函数和理想滤波器的单位冲击响应，计算出所需滤波器的单位冲击响应。

现在，让我们进行实战演练，假设我们需要设计如下滤波器。
- 通带边缘频率 $\omega_p=\frac{\pi}{3}[rad]$
- 通带纹波 $\delta_p=0.1$
- 阻带起始频率 $\omega_s=\frac{\pi}{2}[rad]$
- 阻带纹波 $\delta_s=0.01$

规格中，没有给定阻带衰减，我们只能自己计算。

$$
\begin{aligned}
R_p &= -20 \cdot log_{10}\frac{1  - \delta_p}{1  + \delta_p} = 1.743[dB] \\
A_s &= -20 \cdot log_{10}\frac{\delta_s}{1  + \delta_p} = 40.828[dB] \\
\end{aligned}
$$

在这里，阻带衰减为 $40.8[dB]$。我们选择的窗的阻带衰减必须不低于这个值。实际上，汉宁窗就可以满足这个要求。但我还是决定使用汉明窗进行计算。  
接下来，我们将计算窗的长度。


$$
\begin{aligned}
\Delta \omega &= \omega_s - \omega_p = \frac{\pi}{2} - \frac{\pi}{3} = \frac{\pi}{6} \\ 
N &= \frac{6.6 \pi}{\Delta \omega} = \frac{6.6 \pi}{ \frac{\pi}{6} } = 39.6 \\
\end{aligned}
$$

窗长度计算完毕，接下来是计算单位冲击响应。

$$
\begin{aligned}
h_d(n) &= h_d(n) \cdot \omega(n) \\
       &= \frac{\omega_c}{\pi} sinc(\frac{\omega_c}{\pi}n) \cdot \omega(n), \hspace{3mm} n \in \left(-\frac{N-1}{2}, +\frac{N-1}{2} \right) \\

\omega (n) &= 0.54 + 0.46 \cdot cos \frac{2\pi}{N-1}

\end{aligned}
$$

这样，我们就得到了一个FIR滤波器。下面是我们计算出来的这个滤波器的单位冲击响应。

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter-Window-Function/Design-FIR-Filter-2.jpeg" width="400"></div>


&nbsp;
## 写在最后的话
我们已经成功“设计”出一个滤波器，并得出了它的单位脉冲响应。

然而，这个滤波器真的能够实现吗？我们是否真的能够设计出这样一个滤波器？

事实上，只有通过实际计算这个单位脉冲响应的结果，才能发现这样的滤波器是无法实现的！正如教科书所述，这个滤波器是非因果的。

解决这个问题的方法，以及窗函数的FIR设计代码，将在下一节中详细介绍。


&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{% link _posts/2013-05-29-FIR-Filter-Design-2.md %}"> [下一回] >> </a>
    </span>　
        <a href="{% link _posts/2013-05-23-FIR-Filter.md %}"> << [上一回] </a>
</div>