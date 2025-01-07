---
layout: post
title: "6.IIR滤波器的直接设计"
date: 2013-06-11 18:18:57 +0800
tags: 数字信号处理
# categories: jekyll update
---

不利用模拟滤波器，直接进行数字滤波器的设计的方法，称为直接设计。回忆之前所说的IIR滤波器的直接设计，我们首先设计了巴特沃斯模拟滤波器，然后进行双线性变换，得到数字滤波器。

我所使用的是巴特沃斯低通滤波器作为原型滤波器，其振幅特性如下所示:

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }}  \Bigg | _{s=j \Omega} , \hspace{3mm} \Omega = - \infty \sim  + \infty 
\end{aligned}
$$

首先，我们先把数字滤波器的指标，根据下式转为模拟滤波器的指标。

$$
\begin{aligned}
\Omega = \frac{2}{T} \tan\frac{\omega}{2}
\end{aligned}
$$

然后根据模拟滤波器的设计指标，计算次数$N$，然后计算极点，最后选择出稳定的极点，计算模拟滤波器的传递函数。然后，我们通过拉普拉斯反变换，再使用差分替代微分，得到了$S$平面-$Z$平面的对应关系，将其转为数字滤波器。

现在，我们将这个过程，稍微调整一下。

&nbsp;
### 巴特沃斯低通数字滤波器的设计
首先，转换指标的公式，直接带入巴特沃斯低通模拟滤波器的振幅特性表达式，然后化简，我们可以得到一个新的振幅特性:

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+ \left( \frac{\tan{(\frac{\omega}{2})}}{\tan{(\frac{\omega_{c}}{2})}} \right)^{2N} }}  \Bigg | _{s=j \Omega}
\end{aligned}
$$

同样的，这个振幅特性具有巴特沃斯低通模拟滤波器的一些特性。函数为单调递减的，通带与阻带都没有纹波。这个新的滤波器，称为巴特沃斯低通数字滤波器。

与之前一样的，我们的首先先确定所需要的滤波器的次数N，根据次数计算出极点，选择稳定的极点来计算出传递函数，然后就可以得到滤波器的系数了。

首先是次数的计算，次数的计算，还是要根据阻带衰减指标，根据下式计算。

$$
\begin{aligned}
A_s = -20 \log_{10}|H_a(e^{j\omega})|
\end{aligned}
$$

将巴特沃斯低通数字滤波器的振幅特性带入，我们就可以计算出所需要的滤波器的次数$N$。




&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <!-- <a href="{% link _posts/2013-06-10-IIR-Filter-Design-1.md %}"> [下一回] >> </a> -->
    </span>　
        <a href="{% link _posts/2013-06-10-IIR-Filter-Design-1.md %}"> << [上一回] </a>
</div>