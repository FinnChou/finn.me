---
layout: post
title: "3.单位冲击响应，频响与FIR滤波器实现"
date: 2013-05-29 13:01:47 +0800
tags: 数字信号处理
# categories: jekyll update
---

根据前文的理论分析，具有如下图所示单位冲击响应特性的理想滤波器在实际系统中是不可实现的。这种不可实现性源于其非因果特性。

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response.jpeg" width="300"></div>

为了深入理解该滤波器的特性，我们需要对其频率响应进行分析。滤波器的频率响应可通过对其单位冲击响应进行离散时间傅里叶变换获得。

$$
\begin{aligned}
H_{z}(e^{j\omega}) &= \sum_{n=-20}^{20}h_z(n)e^{-j \omega n} \\
                   &= h_z(0) + \sum_{n=1}^{20}h_z(n)(e^{-j \omega n} + e^{j \omega n}) \\
                   &= h_z(0) + \sum_{n=1}^{20}2h_z(n)cos(\omega n) \\
\end{aligned}
$$

从频率响应的计算结果可以观察到，该滤波器的频率响应为纯实数，不含虚部分量。这表明其相位谱恒为零，即滤波器的输入输出之间不存在任何时延。这种特性在滤波器理论中被称为零相位特性。

然而，从系统实现的角度来看，这种零相位特性的滤波器在实际中是不可实现的。通过分析该滤波器的输入输出关系，我们可以更深入地理解其不可实现性。

$$
\begin{aligned}
y(n) &= \sum_{n=-20}^{20}h_z(k) x(n-k) \\
     &= h_z(-20)x(20) + h_z(-19)x(19) + \cdots + h_z(20) x(-20)
\end{aligned}
$$
从上述计算过程可以看出，该滤波器的输出计算需要同时依赖输入信号的过去值和未来值。由于系统无法预知未来的输入信号，这种非因果特性使得该滤波器在实际中无法实现。为了使滤波器具有可实现性，我们需要对其单位冲击响应进行时移变换，消除对未来值的依赖。具体的变换方式如下所示。

$$
\begin{aligned}
y(n) &= \sum_{n=0}^{40}h_z(k) x(n-k) \\
     &= h_z(0)x(0) + h_z(1)x(-1) + \cdots + h_z(40) x(-40)
\end{aligned}
$$
通过时移变换，我们可以构建一个实用的滤波器系统，该系统仅需存储40个历史输入值及当前输入值即可完成滤波运算。然而，这种时移变换对滤波器的频率响应特性会产生何种影响？这需要通过严格的数学分析来阐明。

$$
\begin{aligned}
H_{line}(e^{j\omega}) &= \sum_{n=0}^{2L}h_{line}(n)e^{-j \omega n} \\
                      &= \sum_{n=0}^{2L}h_{zero}(n - L)e^{-j \omega n} \\
                      &= \sum_{n=-L}^{L}h_{zero}(n)e^{-j \omega (n + L)} \\
                      &= e^{-j \omega n} H_{zero}(e^{j \omega}) \\
                      &= \left| H_{zero}(e^{j \omega}) \right| e^{j \theta_{line}(\omega)} , \hspace{3mm} \theta _{line}(\omega) = \theta _{zero}(\omega) - \omega L
\end{aligned}
$$

为便于理论分析，我们采用参数$L$替代先前示例中的20。

通过对上述频率响应表达式的分析，可以得出两个重要结论：
1. 时移变换不改变滤波器的幅频特性；
2. 时移变换引入了相位延迟，且延迟量与时移长度$L$呈正比关系。
 值得注意的是，这种相位延迟随频率呈线性变化。

基于相位响应的这种线性特性，我们将此类滤波器定义为线性相位FIR滤波器。经过时移变换后，我们获得了一个具有因果性和可实现性的滤波器单位冲激响应，其特性如下图所示。

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response-delay.jpeg" width="300"></div>


&nbsp;
## 窗函数实现的FIR滤波器代码 (C语言)

```c
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>

#define pi (3.1415926)

/* Win Type */
#define Hamming (1)

double Input_Data[] = 
{
    0.000000,  0.896802,  1.538842,  1.760074,  1.538842,  1.000000,  0.363271, -0.142040, -0.363271, -0.278768,
    0.000000,  0.278768,  0.363271,  0.142020, -0.363271, -1.000000, -1.538842, -1.760074, -1.538842, -0.896802,
    0.000000,  0.896802,  1.538842,  1.760074,  1.538842,  1.000000,  0.363271, -0.142040, -0.363271, -0.278768,
    0.000000,  0.278768,  0.363271,  0.142020, -0.363271, -1.000000, -1.538842, -1.760074, -1.538842, -0.896802,
    0.000000,  0.896802,  1.538842,  1.760074,  1.538842,  1.000000,  0.363271, -0.142040, -0.363271, -0.278768,
    0.000000,  0.278768,  0.363271,  0.142020, -0.363271, -1.000000, -1.538842, -1.760074, -1.538842, -0.896802,
    0.000000,  0.896802,  1.538842,  1.760074,  1.538842,  1.000000,  0.363271, -0.142040, -0.363271, -0.278768,
    0.000000,  0.278768,  0.363271,  0.142020, -0.363271, -1.000000, -1.538842, -1.760074, -1.538842, -0.896802,
    0.000000,  55
};

double sinc(double n) 
{
    if (0 == n) 
    {
        return (double)1;
    } 
    else 
    {
        return (double)(sin(pi * n) / (pi * n));
    }
}

int Unit_Impulse_Response(int N, double w_c, int Win_Type, double *Output_Data) 
{
    signed int Count = 0;

    for (Count = -(N - 1) / 2; Count <= (N - 1) / 2; Count++) 
    {
        *(Output_Data + Count + ((N - 1) / 2)) = (w_c / pi) * sinc((w_c / pi) * (double)(Count));
    }

    switch (Win_Type) 
    {
        case Hamming:
            printf("Hamming \n");
            for (Count = -(N - 1) / 2; Count <= (N - 1) / 2; Count++) 
            {
                *(Output_Data + Count + ((N - 1) / 2)) *= (0.54 + 0.46 * cos((2 * pi * Count) / (N - 1)));
            }
            break;

        default:
            printf("default Hamming \n");
            for (Count = -(N - 1) / 2; Count <= (N - 1) / 2; Count++) 
            {
                *(Output_Data + Count + ((N - 1) / 2)) *= (0.54 + 0.46 * cos((2 * pi * Count) / (N - 1)));
            }
            break;
    }

    return (int)1;
}

void Save_Input_Date(double Scand, int Depth, double *Input_Data) 
{
    int Count;

    for (Count = 0; Count < Depth - 1; Count++) 
    {
        *(Input_Data + Count) = *(Input_Data + Count + 1);
    }

    *(Input_Data + Depth - 1) = Scand;
}

double Real_Time_FIR_Filter(double *b, int b_Lenth, double *Input_Data) 
{
    int Count;
    double Output_Data = 0;

    Input_Data += b_Lenth - 1;

    for (Count = 0; Count < b_Lenth; Count++) 
    {
        Output_Data += (*(b + Count)) * (*(Input_Data - Count));
    }

    return (double)Output_Data;
}

int main(void) 
{
    double w_p = pi / 10;
    double w_s = pi / 5;
    double w_c = (w_s + w_p) / 2;
    printf("w_c =  %f \n", w_c);

    int N = 0;
    N = (int)((6.6 * pi) / (w_s - w_p) + 0.5);
    if (0 == N % 2) N++;
    printf("N =  %d \n", N);

    double *Impulse_Response;
    Impulse_Response = (double *)malloc(sizeof(double) * N);
    memset(Impulse_Response, 0, sizeof(double) * N);

    Unit_Impulse_Response(N, w_c, Hamming, Impulse_Response);

    double *Input_Buffer;
    double Output_Data = 0;
    Input_Buffer = (double *)malloc(sizeof(double) * N);
    memset(Input_Buffer, 0, sizeof(double) * N);
    int Count = 0;

    FILE *fs;
    fs = fopen("FIR_Data.txt", "w");

    while (1) 
    {
        if (Input_Data[Count] == 55) break;

        Save_Input_Date(Input_Data[Count], N, Input_Buffer);

        Output_Data = Real_Time_FIR_Filter(Impulse_Response, N, Input_Buffer);

        fprintf(fs, "%lf,", Output_Data);
        Count++;
    }

    fclose(fs);
    printf("Finish \n");
    return (int)0;
}
```

&nbsp;
## 频响的问题
基于上述代码实现，我们设计了一个具有以下规格的FIR低通滤波器：

- 通带截止频率 $\omega_p=\frac{\pi}{3}[rad]$ 
- 通带纹波$\delta_p=0.1$
- 阻带起始频率 $\omega_s=\frac{\pi}{2}[rad]$
- 阻带纹波 $\delta_s=0.01$

通过MATLAB对该滤波器进行频率响应分析，得到如下频响特性曲线：

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response-fr.jpeg" width="300"></div>

从频率响应分析结果来看，该滤波器的幅频特性确实体现了低通滤波器的基本特征。然而，在相位特性方面存在一些值得关注的问题。为了对相位特性进行准确分析，需要采用相位解卷绕技术进行处理。相位解卷绕的具体原理及实现方法将在后续章节中详细阐述。

通过对相位特性的深入分析发现，该FIR滤波器的相位响应与理论预期的线性特性存在偏差。特别是在接近阻带起始频率$\omega_s$的区域，相位响应出现了明显的波动。这种非理想特性的成因及其对系统性能的影响将在后文中进行系统性分析。


## 滤波器性能验证
为了验证所设计滤波器的实际性能，我们进行了实验验证。基于采样周期$T_s=0.1ms$的条件下，通带频率和阻带起始频率可通过以下关系式计算得到：

$$
\begin{aligned}
\omega _p &= 2\pi f_p T_S = \frac{\pi}{3} \\
f_p &= 1.67 kHz \\
\omega _s &= 2\pi f_s T_s = \frac{\pi}{2} \\
f_s &= 2.5 kHz \\
\end{aligned}
$$

为了对滤波器的频率选择性能进行定量评估，我们构造了由$1kHz$和$3kHz$正弦信号叠加的复合输入信号。经过滤波处理后的输入输出波形如下图所示：

<table>
    <tr>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/input_signal.jpeg" width="600">
            </div> 
        </td>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/output_signal.jpeg" width="600">
            </div> 
        </td>
    </tr>
</table> 

图中红色"+"标记表示MATLAB仿真结果，黑色"o"标记表示C语言实现的滤波结果，蓝色"*"标记表示1kHz的理想输出信号。从结果可以观察到，虽然滤波器引入了固有的群延时，但在稳态条件下表现出良好的频率选择性能，成功实现了对高频分量的抑制。

&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{% link _posts/2013-06-05-IIR-Filter.md %}"> [下一回] >> </a>
    </span>　
        <a href="{% link _posts/2013-05-24-FIR-Filter-Design-1.md %}"> << [上一回] </a>
</div>
