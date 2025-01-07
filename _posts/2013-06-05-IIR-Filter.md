---
layout: post
title: "4.IIR滤波器基础"
date: 2013-06-05 22:28:16 +0800
tags: 数字信号处理
# categories: jekyll update
---

&nbsp;
## IIR滤波器构造
之前在介绍FIR滤波器的时候，我们提到过，IIR滤波器的单位冲击响应是无限的！用差分方程来表达一个滤波器，应该是下式这个样子的。

$$
\begin{aligned}
y(n) = -\sum_{k=1}^{N}a_ky(n-k) + \sum_{k=0}^{N}b_kx(n-k)
\end{aligned}
$$

这个式子表示的是$N$次差分方程。我们可以明显看出，在计算输出$y(n)$时，需要依赖于之前的输出值和输入值。这意味着该表达式包含反馈环节。当$a_k = 0$时，由于没有反馈，这个滤波器的单位冲击响应是有限的，因此被称为FIR滤波器。而当$a_k \neq 0$时，则称为IIR滤波器。


&nbsp;
## 直接I型IIR滤波器
就如同之前所说一样，我们考虑这样一个滤波器。

$$
\begin{aligned}
y(n) = -a_1y(n-1) + b_0x(n) + b_1x(n-1)
\end{aligned}
$$

显然，这是一个一阶差分方程。 正因为 $a_k \ne 0$，这表明它是一个一阶IIR滤波器的方程。为了便于观察，我们将其绘制为系统框图。

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys1.jpeg" width="300"></div>

显而易见，要实现这个滤波器，我们需要两个单位的存储空间，以存储过去的输入值和输出值。因此，对于一个$N$阶的滤波器，我们需要$2N$个存储单元。这种结构被称为直接I型结构。

&nbsp;
## 直接II型IIR滤波器
我们首先观察上一小节的直接I型滤波器，将其视为由两个较小系统串联而成的整体。由于其串联结构，输入输出结果的顺序并不会影响最终计算结果，因此我们可以调整这两个系统的位置，形成如下图所示的新系统。

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys2.jpeg" width="300"></div>


接下来，我们可以发现，这个系统实际上并不需要使用两个延迟算子，而是可以将它们合并为一个。这一合并将使系统简化为如下所示的形式。

<div align=center><img src="{{ site.baseurl }}/assets/IIR-Filter/IIR-Filter-Sys3.jpeg" width="300"></div>

这个新系统与之前的直接I型系统具有完全相同的输入输出特性，同时节省了一半的延迟算子。实现这个滤波器所需的存储空间为$N$。


&nbsp;
## 直接II型IIR滤波器的实现（C语言）

{% highlight c++ %}

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
    
    if(Lenth_a >= Lenth_b) Memory_Lenth = Lenth_a;
    else Memory_Lenth = Lenth_b;
    
    Output_Data += (*a) * Input_Data;  //a(0)*x(n)             
    
    for(Count = 1; Count < Lenth_a ;Count++)
    {
        Output_Data -= (*(a + Count)) *
                       (*(Memory_Buffer + (Memory_Lenth - 1) - Count));                                       
    } 
    
    //------------------------save data--------------------------// 
    *(Memory_Buffer + Memory_Lenth - 1) = Output_Data;
    Output_Data = 0;
    //----------------------------------------------------------// 
    
    for(Count = 0; Count < Lenth_b ;Count++)
    {    	
        Output_Data += (*(b + Count)) *
                       (*(Memory_Buffer + (Memory_Lenth - 1) - Count));      
    }
    
    //------------------------move data--------------------------// 
    for(Count = 0 ; Count < Memory_Lenth -1 ; Count++)
    {
    	*(Memory_Buffer + Count) = *(Memory_Buffer + Count + 1);
    }
    *(Memory_Buffer + Memory_Lenth - 1) = 0;
    //-----------------------------------------------------------//
 
    return (double)Output_Data; 
}
 
int main(void)
{    
    double a[] = {1   , 0.5554 ,0.1542 };   
    double b[] = {0   ,      0 ,0.1542 };	
   
    int Lenth_a = sizeof(a)/sizeof(double);
    int Lenth_b = sizeof(b)/sizeof(double);
    int Memory_Lenth = 0;
    
    if(Lenth_a >= Lenth_b) Memory_Lenth = Lenth_a;
    else Memory_Lenth = Lenth_b;
    
    printf("Memory Lenth : %d  \n" , Memory_Lenth); 
   
    double Input = 0 ;
    double Output = 0;
    
    double *Memory_Buffer;
    Memory_Buffer = (double *) malloc(sizeof(double)*Memory_Lenth);  
    memset(Memory_Buffer,
           0,
           sizeof(double)*Memory_Lenth);
    
 
    FILE* Input_Data;
    FILE* Output_Data;
     
    Input_Data = fopen("input.dat","r"); 
    Output_Data= fopen("output.txt","w"); 
    
    while(1)
    {
        if(fscanf(Input_Data, "%lf", &Input) == EOF)  break;
        
        Output = IIR_Filter( a, Lenth_a,
                             b, Lenth_b,
                             Input,
                             Memory_Buffer);
 
        fprintf(Output_Data,"%lf,",Output);
        
    	//printf("Output:  %lf \n" ,Output);
    }	
    
    printf("Finish \n");
 
    return (int)1;
}
{% endhighlight %}

当然，这里的参数还是随意设置的。之后我会就IIR的滤波器的间接设计与直接设计，进行进一步的说明。

&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{% link _posts/2013-06-10-IIR-Filter-Design-1.md %}"> [下一回] >> </a>
    </span>　
        <a href="{% link _posts/2013-05-29-FIR-Filter-Design-2.md %}"> << [上一回] </a>
</div>
