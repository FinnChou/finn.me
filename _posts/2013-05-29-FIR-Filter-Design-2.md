---
layout: post
title: "[数字信号处理]3.单位冲击响应，频响与FIR滤波器实现(C语言)"
date: 2013-05-29 13:01:47 +0800
# categories: jekyll update
---

&nbsp;
## 单位冲击响应与频响 
就像大家知道的，如果需要设计下图所示的单位冲击响应的滤波器，事实上是无法实现的。

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response.jpeg" width="300"></div>

接下来，让我们分析该滤波器的频率响应。频率响应是通过计算单位冲击响应的离散时间傅里叶变换得到的。


$$
\begin{aligned}
H_{z}(e^{j\omega}) &= \sum_{n=-20}^{20}h_z(n)e^{-j \omega n} \\
                   &= h_z(0) + \sum_{n=1}^{20}h_z(n)(e^{-j \omega n} + e^{j \omega n}) \\
                   &= h_z(0) + \sum_{n=1}^{20}2h_z(n)cos(\omega n) \\
\end{aligned}
$$

我们可以观察到，这个滤波器的频率响应计算结果为实数，并没有虚数部分。这意味着其相位谱始终为0，表明该滤波器的输入与输出之间没有延迟。这种相位特性被称为零延迟相位特性。

然而，这种滤波器实际上是无法实现的。通过对该滤波器的输入和输出进行实际计算，我们可以更清楚地理解这一点。

$$
\begin{aligned}
y(n) &= \sum_{n=-20}^{20}h_z(k) x(n-k) \\
     &= h_z(-20)x(20) + h_z(-19)x(19) + \cdots + h_z(20) x(-20)
\end{aligned}
$$

在计算过程中，这个滤波器需要同时考虑过去和未来的值。然而，由于未来的值是不可预测的，这使得该滤波器无法实现。为了使这个滤波器可行，我们必须调整其单位冲击响应，从而不再依赖未来的值。例如，可以通过如下方式进行调整。

$$
\begin{aligned}
y(n) &= \sum_{n=0}^{40}h_z(k) x(n-k) \\
     &= h_z(0)x(0) + h_z(1)x(-1) + \cdots + h_z(40) x(-40)
\end{aligned}
$$

通过这种方式，我们可以实现这个滤波器，只需记录其40个历史输入值以及当前的输入值。然而，由于单位冲击响应的移动，这是否会对频率响应产生影响呢？让我们来探讨一下。

$$
\begin{aligned}
H_{line}(e^{j\omega}) &= \sum_{n=0}^{2L}h_{line}(n)e^{-j \omega n} \\
                      &= \sum_{n=0}^{2L}h_{zero}(n - L)e^{-j \omega n} \\
                      &= \sum_{n=-L}^{L}h_{zero}(n)e^{-j \omega (n + L)} \\
                      &= e^{-j \omega n} H_{zero}(e^{j \omega}) \\
                      &= \left| H_{zero}(e^{j \omega}) \right| e^{j \theta_{line}(\omega)} , \hspace{3mm} \theta _{line}(\omega) = \theta _{zero}(\omega) - \omega L
\end{aligned}
$$

为了更好地说明问题，我们用$L$代替之前例子中的20。

在移动之后的频响中，根据上述公式，我们可以得出两个结论：第一，移动不会对幅度谱产生影响；第二，移动会导致相位产生延迟，这个延迟主要取决于移动的长度，移动的长度越长，延迟越大。然而，这种移动是线性的。

因此，我们将这种移动的相位特性称为线性相位特性。至此，我们得到了移动后的因果可实现滤波器的单位冲击响应，如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response-delay.jpeg" width="300"></div>



&nbsp;
## 窗函数实现的FIR滤波器代码 (C语言)

{% highlight c++ %}
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>
 
 
#define   pi         (3.1415926)
 
/*-------------Win Type----------------*/
#define   Hamming    (1)
 
 
 
double Input_Data[] = 
{
0.000000 , 0.896802 , 1.538842 , 1.760074 ,  1.538842 ,  1.000000 ,  0.363271 , -0.142040 , -0.363271 , -0.278768,
0.000000 , 0.278768 , 0.363271 , 0.142020 , -0.363271 , -1.000000 , -1.538842 , -1.760074 , -1.538842 , -0.896802,
0.000000 , 0.896802 , 1.538842 , 1.760074 ,  1.538842 ,  1.000000 ,  0.363271 , -0.142040 , -0.363271 , -0.278768,
0.000000 , 0.278768 , 0.363271 , 0.142020 , -0.363271 , -1.000000 , -1.538842 , -1.760074 , -1.538842 , -0.896802,
0.000000 , 0.896802 , 1.538842 , 1.760074 ,  1.538842 ,  1.000000 ,  0.363271 , -0.142040 , -0.363271 , -0.278768,
0.000000 , 0.278768 , 0.363271 , 0.142020 , -0.363271 , -1.000000 , -1.538842 , -1.760074 , -1.538842 , -0.896802,
0.000000 , 0.896802 , 1.538842 , 1.760074 ,  1.538842 ,  1.000000 ,  0.363271 , -0.142040 , -0.363271 , -0.278768,
0.000000 , 0.278768 , 0.363271 , 0.142020 , -0.363271 , -1.000000 , -1.538842 , -1.760074 , -1.538842 , -0.896802,
0.000000 , 55
};
 
double sinc(double n)
{
    if(0==n) return (double)1;
    else return (double)(sin(pi*n)/(pi*n));
}
 
int Unit_Impulse_Response(int N,double w_c,
                          int Win_Type,
                          double *Output_Data)
{
    signed int Count = 0; 
  
    for(Count = -(N-1)/2;Count <= (N-1)/2;Count++)
    {
    	*(Output_Data+Count+((N-1)/2)) = (w_c/pi)*sinc((w_c/pi)*(double)(Count));
    }	
	
	
    switch (Win_Type)
    {
    	
    	case Hamming:   printf("Hamming \n");
    	                for(Count = -(N-1)/2;Count <= (N-1)/2;Count++)
    			{
    			    *(Output_Data+Count+((N-1)/2)) *= (0.54 + 0.46 * cos((2*pi*Count)/(N-1)));
   			} 
    	                break;
    	                
    	                
    	default:        printf("default Hamming \n");
    	                for(Count = -(N-1)/2;Count <= (N-1)/2;Count++)
    			{
    			    *(Output_Data+Count+((N-1)/2)) *= (0.54 + 0.46 * cos((2*pi*Count)/(N-1)));
   			} 
    	
                        break;
    }
    
    return (int)1;
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
 
 
 
 
 
int main(void)
{
    double w_p = pi/10;
    double w_s = pi/5;
    double w_c = (w_s + w_p)/2;
    printf("w_c =  %f \n" , w_c);
    
    int N = 0;  
    N = (int) ((6.6*pi)/(w_s - w_p) + 0.5);
    if(0 == N%2) N++;    
    printf("N =  %d \n" , N);    
  
    double *Impulse_Response;        
    Impulse_Response = (double *) malloc(sizeof(double)*N);  
    memset(Impulse_Response,
           0,
           sizeof(double)*N);
           
    Unit_Impulse_Response(N,w_c,
                          Hamming,
                          Impulse_Response);       
 
    double *Input_Buffer;
    double Output_Data = 0;
    Input_Buffer = (double *) malloc(sizeof(double)*N);  
    memset(Input_Buffer,
           0,
           sizeof(double)*N);
    int Count = 0;
    
    FILE *fs; 
    fs=fopen("FIR_Data.txt","w"); 
    
    while(1)
    {   
    	if(Input_Data[Count] == 55) break;
    	 
    	Save_Input_Date (Input_Data[Count],
        	         N,
                	 Input_Buffer);
       
        Output_Data = Real_Time_FIR_Filter(Impulse_Response,
                                           N,
                                           Input_Buffer);
                 		
          
        fprintf(fs,"%lf,",Output_Data);
        //if(((Count+1)%5 == 0)&&(Count != 0))  fprintf(fs,"\r\n"); 
   
        Count++;
    }
          
    /*---------------------display--------------------------------     
    for(Count = 0; Count < N;Count++)
    {
        printf("%d %lf  ",Count,*(Input_Buffer+Count));
        if(((Count+1)%5 == 0)&&(Count != 0)) printf("\n");        	 
    }      
    */  
    
    fclose(fs);
    printf("Finish \n");
    return (int)0;
}

{% endhighlight %}


&nbsp;
## 频响的问题
运行上面的代码， 参数如下设定，我们就实现了一个FIR滤波器。

- 通带边缘频率 $\omega_p=\frac{\pi}{3}[rad]$
- 通带纹波 $\delta_p=0.1$
- 阻带起始频率 $\omega_s=\frac{\pi}{2}[rad]$
- 阻带纹波 $\delta_s=0.01$

使用Matlab绘制出其频响如下: 

<div align=center><img src="{{ site.baseurl }}/assets/Design-FIR-Filter/Impulse-Response-fr.jpeg" width="300"></div>

从幅度特性来看，我们确实成功实现了一个低通滤波器。然而，相位特性却显得有些异常。为了更清楚地展示问题，需要进行相位解卷绕。关于解卷绕的定义及其原因，我将在后文中详细说明。

接下来，问题出现了！理论上，这个FIR滤波器应具备线性相位特性，但为何我们观察到的线性相位特性并不是一条直线呢？在接近阻带起始频率的地方，出现了震荡现象。这个问题我们稍后将进行深入探讨。


## 实际的滤波效果
为了验证效果，我们输入实际的信号看看。这里，我们选择采样周期为0.1ms，那么，其通带频率和阻带起始频率为:

$$
\begin{aligned}
\omega _p &= 2\pi f_p T_S = \frac{\pi}{3} \\
f_p &= 1.67 kHz \\
\omega _s &= 2\pi f_s T_s = \frac{\pi}{2} \\
f_s &= 2.5 kHz \\
\end{aligned}
$$

为了验证其性质，我选择了1KHz和3KHz的频率混合作为输入，执行该滤波操作。 其输入和输出如下图所示：

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

红色的“+”是Matlab计算的结果，黑的o是我用C语言实现的滤波器的计算结果，蓝的*号是1KHz的信号，也就是希望的输出。可以看出，这个滤波器虽然有一定的延迟，但是在延迟后的滤波效果是符合预期的。


&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{% link _posts/2013-06-05-IIR-Filter.md %}"> [下一回] >> </a>
    </span>　
        <a href="{% link _posts/2013-05-24-FIR-Filter-Design-1.md %}"> << [上一回] </a>
</div>
