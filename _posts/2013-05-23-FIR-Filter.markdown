---
layout: post
title: "[数字信号处理]1.FIR滤波器基础"
date: 2013-05-23 17:36:36 +0800
# categories: jekyll update
---

对于滤波器而言，如果其单位冲击响应是一个有限区间的数列，则该滤波器被称为FIR滤波器。相反，如果单位冲击响应是一个无限区间的数列，则该滤波器被称为IIR滤波器。

接下来，我们将通过线性差分方程在时域内解释FIR和IIR数字滤波器。通过对单位脉冲响应与输入信号进行卷积运算，可以得到以下公式：

$$
\begin{aligned}
y(n) = \sum_{k=0}^{\infty}h(k)x(n-k)
\end{aligned}
$$

将其改写为递归的方式，则

$$
\begin{aligned}
y(n) &= \sum_{k=0}^{\infty}a^kx(n-k) \\
     &= \sum_{k=1}^{\infty}a^kx(n-k) + a^0x(n - 0) \\
     &= a \sum_{k=0}^{\infty}a^kx((n - 1)-k) + x(n) \\
     &= ay(n-1) + x(n)
\end{aligned}
$$

上式是1次差分方程式，而对于N次数字滤波器的输入输出关系，表示为N次差分方程式，如下所示。

$$
\begin{aligned}
y(n) = -\sum_{k=1}^{N}a_ky(n-k) + \sum_{k=0}^{M}b_kx(n-k) 
\end{aligned}
$$

由上式看，输出 $y(n)$ 需要自己的历史值，也就是，含有反馈。$a_k \neq 0$的时候，反馈有作用，其系统框图如下。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-1.jpeg" width="300"></div>

此时，输入单位脉冲信号。由于反馈的影响，系统的单位冲击响应是无限的。当 $a_k = 0$ 时，反馈作用消失，系统的单位冲击响应则变为有限。这一现象可以通过以下单位框图进行说明。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-2.jpeg" width="300"></div>


在没有反馈的情况下，输入单位脉冲信号使得系统的单位冲击响应是有限的，这就是所谓的有限冲击响应（Finite Impulse Response），其字面意思即为此。接下来，我们将使用C语言实现一个FIR滤波器，在此过程中，滤波器的系数将随意设置。


{% highlight c++ %}
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
{% endhighlight %}


一个FIR滤波器已成功实现。只需不断输入信号，按下ESC键即可停止程序。
其单位冲击响应在Matlab中的表示如下。

<div align=center><img src="{{ site.baseurl }}/assets/FIR-Filter/FIR-Filter-Sys.jpeg" width="300"></div>


<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{{ site.baseurl }}/2013/05/24/Design-FIR-Filter-Window-Function.html"> [下一回] >> </a>
        <!-- [下一回] >>　 -->
    </span>　
        <!-- << [上一回]  -->
</div>
