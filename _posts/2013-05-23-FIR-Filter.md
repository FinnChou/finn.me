---
layout: post
title: "1.FIR滤波器基础"
date: 2013-05-23 17:36:36 +0800
tags: 数字信号处理
# categories: jekyll update
---

在数字信号处理领域中，滤波器可以根据其单位冲击响应的特性分为两大类。当滤波器的单位冲击响应是有限长度的数列时，我们称之为有限冲击响应(FIR)滤波器。相反地，如果单位冲击响应是无限长度的数列，则称之为无限冲击响应(IIR)滤波器。

让我们从时域的角度，通过线性差分方程来深入理解FIR和IIR数字滤波器的本质。根据卷积定理，系统的输出可以表示为单位脉冲响应与输入信号的卷积，即：

$$
\begin{aligned}
y(n) = \sum_{k=0}^{\infty}h(k)x(n-k)
\end{aligned}
$$

将此表达式改写为递归形式：

$$
\begin{aligned}
y(n) &= \sum_{k=0}^{\infty}a^kx(n-k) \\
     &= \sum_{k=1}^{\infty}a^kx(n-k) + a^0x(n - 0) \\
     &= a \sum_{k=0}^{\infty}a^kx((n - 1)-k) + x(n) \\
     &= ay(n-1) + x(n)
\end{aligned}
$$

这是一个一阶差分方程。对于更一般的情况，$N$阶数字滤波器的输入输出关系可以用$N$阶差分方程来描述：

$$
\begin{aligned}
y(n) = -\sum_{k=1}^{N}a_ky(n-k) + \sum_{k=0}^{M}b_kx(n-k) 
\end{aligned}
$$

从这个方程中我们可以看出一个关键特性：当系数$a_k \neq 0$时，输出$y(n)$依赖于其历史值，这意味着系统中存在反馈。这种结构可以用下面的系统框图来表示：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-1.jpeg" width="300"></div>

在这种情况下，如果输入单位脉冲信号，由于反馈的存在，系统的响应将会无限持续。然而，当所有的$a_k = 0$时，反馈路径消失，系统的结构简化为：

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-2.jpeg" width="300"></div>

这种无反馈的结构就是典型的FIR滤波器。当输入单位脉冲信号时，系统的响应是有限的，这也正是"有限冲击响应"名称的由来。在接下来的内容中，我们将通过C语言实现一个基本的FIR滤波器，以便更直观地理解其工作原理。虽然在这个示例中滤波器的系数是随意设置的，但这不影响我们理解FIR滤波器的基本概念。


```c
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>
 
double Real_Time_FIR_Filter(double *b,
                            int     b_Lenth,
                            double *Input_Data)
{    
    int Count;
    double Output_Data = 0;
    
    Input_Data += b_Lenth - 1;  
    
    for(Count = 0; Count < b_Lenth ;Count++)
    { 
            Output_Data += (*(b + Count)) *
                            (*(Input_Data - Count));
    }         
    
    return (double)Output_Data;
}
 
void Save_Input_Date (double Scand,
                      int    Depth,
                      double *Input_Data)
{
    int Count;
  
    for(Count = 0 ; Count < Depth-1 ; Count++)
    {
    	*(Input_Data + Count) = *(Input_Data + Count + 1);
    }
    
    *(Input_Data + Depth-1) = Scand;
}
 
 
int main(void)
{
    double b[] = {0.5 , -0.5 , 1};
    double Scand_Data = 0;
    char Command = 0;
   
    int b_Lenth = (sizeof(b)/sizeof(double));
    int Count = 0;
    
    double Input_Data[sizeof(b)/sizeof(double)] = {0};
    double Output_Data = 0;
    
    /*--------------------display----------------------------*/      
    printf("  b(k) : ");
    for(Count = 0; Count < b_Lenth ;Count++)
    {
    	printf("%f " , b[Count]);
    }
    printf("\n");
    /*-----------------------------------------------------*/
    
    Count = 0;
    while(1)
    {
    	if(Count == 0) printf("The Input : ");   
        else printf("The Next Input : ");   
   
    	scanf("%lf",&Scand_Data);
    	printf("Input x(%d) : %lf         ",Count,Scand_Data);    
    	
    	Save_Input_Date (Scand_Data,
                         b_Lenth,
                         Input_Data);
 
    	Output_Data = Real_Time_FIR_Filter(b,
                                           b_Lenth,
                                           Input_Data);        
                             
        printf("Output y(%d) : %lf  \n",Count,Output_Data);                    
 
    	scanf("%c",&Command);
    	if(Command == 27) break;    //ESC
    	
    	Count++;
    }
    
    printf("\n");
	
    return (int)0;
}
```

至此，我们已经成功实现了一个实时FIR数字滤波器。

该程序通过简单的命令行界面进行交互
    - 用户可以连续输入信号样本值，系统会实时计算并输出经过滤波后的结果。
    - 当需要结束程序时，只需按下ESC键即可。

为了直观地展示该FIR滤波器的特性，下图给出了其在MATLAB中绘制的单位冲激响应曲线。从图中我们可以清晰地观察到滤波器的时域特性。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-Sys.jpeg" width="300"></div>


<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{% link _posts/2013-05-24-FIR-Filter-Design-1.md %}"> [下一回] >> </a>
        <!-- [下一回] >>　 -->
    </span>　
        <!-- << [上一回]  -->
</div>
