---
layout: post
title: "[番外]相位特性解卷绕"
date: 2013-06-02 18:19:42  +0800
tags: 数字信号处理
# categories: jekyll update
---

对于FIR滤波器的系数，也就是FIR滤波器的单位冲击响应，做离散时间的傅里叶变换。比如，像下面这样。

$$
\begin{aligned}
H(e^{j\omega}) &= \sum_{n=-\infty}^{+\infty}h(n)e^{-j \omega n} \\
\end{aligned}
$$

所得到的结果是这个FIR滤波器的频率响应。然而，频率响应又表示为振幅特性和相位特性，就像这样

$$
\begin{aligned}
H(e^{j\omega}) &= |H(e^{j\omega})|e^{j\theta(\omega)} \\
\end{aligned}
$$

所以，振幅特性和相位特性就按下式可以计算出来。

$$
\begin{aligned}
|H(e^{j\omega})| &= \sqrt{\Big[Re\big(H(e^{j\omega})\big)\Big]^2 + \Big[Im\big(H(e^{j\omega})\big)\Big]^2} \\
\theta(\omega) &= \tan^{-1}\left(\frac{Im\big(H(e^{j\omega})\big)}{Re\big(H(e^{j\omega})\big)}\right) \\
\end{aligned}
$$

既然如此，我们就把一个系统的相位特性作图，看看得到的是什么东西。就拿我们之前已经设计并实现的FIR来做试验，其相位特性如下图所示。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_1.jpeg" width="300"></div>

我们都知道，FIR滤波器是具有线性相位特性的，说的不严格一点，至少他是一条直线。而上图的拿Matlab画的相位特性，并是这样的。为什么呢？再次看看我们之前的算式，其使用到了反正切函数。假设，我们现在所设计的滤波器某一段相位特性，是由$-120°$转动到$-225°$的，就像下图所示一样的。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_2.jpeg" width="300"></div>

但是，Matlab所求得的反正切的值的范围是$[-\pi, \pi]$。所以，当角度转过实轴（Re轴），$-225°$所输出的值为$135°$，这里产生了一个$2\pi$的跳动。所以，我们看到的相位特性，在开始的部分，跳动都是$2\pi$。为了形象点，我在下图标注了。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_3.jpeg" width="300"></div>

这也就解决了一个问题，为什么明明是线性相位特性，用Matlab画出来的相位特性却如此不规则。所以，在这个时候，我们需要把反正切漏算的$2\pi$给补上，这个过程就叫做解卷绕，有的也称为解包裹。在Matlab中，使用函数unwrap()进行解卷绕操作，具体的代码就像这样。当然，在绘制相位特性前，你需要一组FIR滤波器系数$h$。

```matlab
w = 0:0.01:pi;
H = freqz(h,1,w);
figure;
plot(w,unwrap(angle(H)));
axis([0 pi -35 5]);grid; 
xlabel('Frequency \omega [rad]');
ylabel('\theta [rad]');
```

解完卷绕后的相位特性，就像下图一样，很清晰的就可以看出是线性相位特性。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_4.jpeg" width="300"></div>

即便是到此，我们解卷绕之后，还是有一个问题，红色的部分，还是在震荡。为什么会有这个震荡呢？

要弄清楚问题，我们先看看这个FIR滤波器的振幅特性和相位特性。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_5.jpeg" width="3450"></div>

可以看到，相位特性产生震荡的部分，好像就在截止频率附近。然而，通过窗函数实现的滤波器，通带和阻带是有纹波的。这里的振幅特性由于纵轴是对数特性，所以，我们可以很清晰的看到阻带纹波的存在。正是因为纹波的存在，才使得相位特性有震荡！为了方便理解，我们把之前离散时间傅里叶变换的计算结果拿出来，直接画在复平面上，如下图所示。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_6.jpeg" width="300"></div>

我们先看这个最外层的圆，这个圆必定是单位圆。这个单位圆代表了通带，在通带上，振幅特性为1。我们可以这么理解外面的最外层的圆。然后从某一点开始，这个圆逐渐向圆心收敛（嘛，不要在意不严密的言辞）。这个收敛的区域代表了过渡带，振幅特性在逐渐减小。最后到达阻带，严格来讲，理想情况下，阻带就是原点。因为阻带的振幅特性为0。

但是，我们之前提及了，纹波是存在的！通带有纹波，阻带也有纹波。通带由于存在纹波，不会是完美的单位圆。同样的，阻带由于存在纹波，也不是完美的。我们放大来看看。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_7.jpeg" width="300"></div>

由于纹波存在，阻带不是一个点。我们仔细观察，阻带的复平面的表现形状总是从第三象限，穿过原点，到达地一象限！按照之前的说的，反正切输出的的范围是$[-\pi, \pi]$，所以，这里产生的跳动是$\pi$，而不是$2\pi$！我们再将其标注在图上，便于理解。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter-Phase-Unwrapping/unwarpping_8.jpeg" width="300"></div>

到此，对于相位特性的理解又深刻了！！

### 不负责的几句话
再说一点题外话，这点我没考证过。在复平面内来说，理想滤波器的阻带应该是原点。但是，相位特性是这样计算的，如下图（其实这个前面说过）。

$$
\begin{aligned}
|H(e^{j\omega})| &= \sqrt{\Big[Re\big(H(e^{j\omega})\big)\Big]^2 + \Big[Im\big(H(e^{j\omega})\big)\Big]^2} \\
\theta(\omega) &= \tan^{-1}\left(\frac{Im\big(H(e^{j\omega})\big)}{Re\big(H(e^{j\omega})\big)}\right) \\
\end{aligned}
$$

也就是，虚部除以实部的反正切。这里就有问题了，要是原点的话，相位特性在原点无意义。也就是说，其实截止频率之后的相位特性不用管的，只要看通带的相位特性就好了。

要是觉得难以理解，可以换种思维。首先，相位特性反应了输入与输出之间的延迟。对于低通滤波器，输入的信号（单成分正弦波）频率越高，延迟越大。当频率超过截止频率，那么输入与输出之间的延迟，是无限的，输出永远不会到来，也就是阻带。

这里叠个甲，最后这几句话是我个人的理解，目前没有进行过严谨的研究和推定，只是我个人的理解。
