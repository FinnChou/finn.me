---
layout: post
title: "4.IIR滤波器基础"
date: 2013-06-05 22:28:16 +0800
tags: 数字信号处理
# categories: jekyll update
---

在前文中，我们深入探讨了FIR滤波器的理论基础与实现方法。本节我们将把目光转向另一类重要的数字滤波器——无限冲击响应(IIR)滤波器。顾名思义，IIR滤波器最显著的特征在于其单位冲击响应序列具有无限长度的特性。从系统理论的角度来看，一个通用的数字滤波器可以用如下$N$阶差分方程来描述：

$$
\begin{aligned}
y(n) = -\sum_{k=1}^{N}a_ky(n-k) + \sum_{k=0}^{N}b_kx(n-k)
\end{aligned}
$$

仔细分析这个差分方程的结构，我们可以发现系统输出$y(n)$不仅依赖于输入序列$x(n-k)$，还依赖于系统的历史输出值$y(n-k)$。这种包含反馈路径的结构中，系数$a_k$起着决定性的作用：当所有$a_k = 0$时，系统不存在反馈路径，其单位冲击响应具有有限长度，此时系统即为FIR滤波器；而当存在非零的$a_k$时，由于反馈路径的存在，系统的单位冲击响应将具有无限长度，这正是IIR滤波器的本质特征。

&nbsp;
## 直接I型IIR滤波器
首先，我们分析一个一阶IIR滤波器的差分方程：

$$
\begin{aligned}
y(n) = -a_1y(n-1) + b_0x(n) + b_1x(n-1)
\end{aligned}
$$

该方程为一阶差分方程，其中系数$a_1 \ne 0$表明这是一个典型的一阶IIR滤波器。为了深入理解其结构特性，我们可以将其表示为系统框图：

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys1.jpeg" width="450"></div>

从系统实现的角度分析，该结构需要两个存储单元分别用于保存输入和输出的历史值。推广至$N$阶系统，总共需要$2N$个存储单元。这种实现方式在数字信号处理中被称为直接I型结构。

&nbsp;
## 直接II型IIR滤波器
对直接I型结构进行分析可知，该系统可视为两个子系统的级联。基于线性系统的可交换性原理，这两个子系统的位置可以互换而不影响系统的传递函数。据此，我们可以构造如下所示的等效系统结构：

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys2.jpeg" width="450"></div>

通过对系统结构的深入分析，可以发现两个延迟单元的功能存在重叠。通过将这些冗余的延迟单元合并，可以得到一个更为紧凑的系统结构，如下图所示：

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys3.jpeg" width="450"></div>

这种优化后的结构被称为直接II型结构。该结构在保持系统传递函数不变的同时，将所需存储单元数量从$2N$降低至$N$，显著提高了实现效率。

&nbsp;
## 直接II型IIR滤波器的实现（C语言）

```c
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>

double IIR_Filter(double *a, int Lenth_a,
                 double *b, int Lenth_b,
                 double Input_Data,
                 double *Memory_Buffer)
{
    int Count;
    double Output_Data = 0;
    int Memory_Lenth = 0;
    
    if (Lenth_a >= Lenth_b) 
    {
        Memory_Lenth = Lenth_a;
    } 
    else 
    {
        Memory_Lenth = Lenth_b;
    }
    
    Output_Data += (*a) * Input_Data;  //a(0)*x(n)             
    
    for (Count = 1; Count < Lenth_a; Count++) 
    {
        Output_Data -= (*(a + Count)) * (*(Memory_Buffer + (Memory_Lenth - 1) - Count));                                       
    } 
    
    //------------------------save data--------------------------// 
    *(Memory_Buffer + Memory_Lenth - 1) = Output_Data;
    Output_Data = 0;
    //----------------------------------------------------------// 
    
    for (Count = 0; Count < Lenth_b; Count++) 
    {    	
        Output_Data += (*(b + Count)) * (*(Memory_Buffer + (Memory_Lenth - 1) - Count));      
    }
    
    //------------------------move data--------------------------// 
    for (Count = 0; Count < Memory_Lenth - 1; Count++) 
    {
        *(Memory_Buffer + Count) = *(Memory_Buffer + Count + 1);
    }
    *(Memory_Buffer + Memory_Lenth - 1) = 0;
    //-----------------------------------------------------------//

    return (double)Output_Data;
}

int main(void)
{    
    double a[] = {1.0000, 0.5554, 0.1542};   
    double b[] = {0.0000, 0.0000, 0.1542};	
   
    int Lenth_a = sizeof(a) / sizeof(double);
    int Lenth_b = sizeof(b) / sizeof(double);
    int Memory_Lenth = 0;
    
    if (Lenth_a >= Lenth_b) 
    {
        Memory_Lenth = Lenth_a;
    } 
    else 
    {
        Memory_Lenth = Lenth_b;
    }
    
    printf("Memory Lenth : %d\n", Memory_Lenth); 
   
    double Input = 0;
    double Output = 0;
    
    double *Memory_Buffer;
    Memory_Buffer = (double *)malloc(sizeof(double) * Memory_Lenth);  
    memset(Memory_Buffer, 0, sizeof(double) * Memory_Lenth);
    
    FILE *Input_Data;
    FILE *Output_Data;
     
    Input_Data = fopen("input.dat", "r"); 
    Output_Data = fopen("output.txt", "w"); 
    
    while (1) 
    {
        if (fscanf(Input_Data, "%lf", &Input) == EOF) 
        {
            break;
        }
        
        Output = IIR_Filter(a, Lenth_a,
                          b, Lenth_b,
                          Input,
                          Memory_Buffer);

        fprintf(Output_Data, "%lf,", Output);
        
        //printf("Output:  %lf \n" ,Output);
    }	
    
    printf("Finish \n");

    return (int)1;
}
```

需要说明的是，上述示例中的滤波器系数仅用于演示目的。关于IIR滤波器的系统设计方法，包括间接法和直接法的具体实现细节，将在后续章节中详细探讨。

&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{{ site.baseurl }}/2013/06/10/IIR-Filter-Design-1"> [下一回] >> </a>
    </span>　
        <a href="{{ site.baseurl }}/2013/05/29/FIR-Filter-Design-2"> << [上一回] </a>
</div>
