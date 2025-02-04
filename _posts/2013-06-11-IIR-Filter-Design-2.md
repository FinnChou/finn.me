---
layout: post
title: "6.IIR滤波器的直接设计"
date: 2013-06-11 18:18:57 +0800
tags: 数字信号处理
# categories: jekyll update
---
在数字滤波器设计中，除了通过模拟滤波器间接设计的方法外，还可以直接进行数字滤波器的设计，这种方法称为直接设计法。在前文中，我们介绍了IIR滤波器的间接设计方法，即先设计巴特沃斯模拟滤波器，再通过双线性变换得到数字滤波器。

本文依旧以巴特沃斯低通滤波器为原型，其振幅特性可表示为:

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }}  \Bigg | _{s=j \Omega} , \hspace{3mm} \Omega = - \infty \sim  + \infty 
\end{aligned}
$$

在间接设计法中，首先需要将数字滤波器的指标通过以下映射关系转换为模拟滤波器的指标:

$$
\begin{aligned}
\Omega = \frac{2}{T} \tan\frac{\omega}{2}
\end{aligned}
$$

基于转换后的指标，计算滤波器阶数$N$并求解极点，选取稳定极点后即可得到模拟滤波器的传递函数。最后通过拉普拉斯反变换并用差分替代微分，建立$S$平面到$Z$平面的映射关系，完成向数字滤波器的转换。

接下来，我们将对这一设计流程进行优化调整。

&nbsp;
### 巴特沃斯低通数字滤波器的设计
在直接设计法中，我们首先将转换指标的公式代入巴特沃斯低通模拟滤波器的振幅特性表达式，经过化简可得到巴特沃斯低通数字滤波器的振幅特性:

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+ \left( \frac{\tan{(\frac{\omega}{2})}}{\tan{(\frac{\omega_{c}}{2})}} \right)^{2N} }}  \Bigg | _{s=j \Omega}
\end{aligned}
$$

该滤波器保持了巴特沃斯低通模拟滤波器的主要特性:单调递减、通带与阻带无纹波。

设计流程与间接法类似,首先需要确定滤波器阶数N。根据阻带衰减指标:

$$
\begin{aligned}
A_s = -20 \log_{10}|H_a(e^{j\omega})|
\end{aligned}
$$

代入振幅特性, 可得需要的滤波器的次数$N$:

$$
\begin{aligned}
N = \frac{1}{2} \frac{\log_{10}(10^{\frac{A_s}{10}}-1)}{\log_{10}\bigg( \frac{\tan(\omega/2)}{\tan(\omega_c / 2)} \bigg)}
\end{aligned}
$$

接下来需要计算极点。首先对振幅特性平方:

$$
\begin{aligned}
|H(e^{j\omega})|^2 = \frac{\tan^{2N}(\omega_c/2)}{\tan^{2N}(\omega_c/2) + \tan^{2N}(\omega/2)} 
\end{aligned}
$$

然后，把 $\tan(\omega/2)$ 部分稍微做下变形，

$$
\begin{aligned}
\tan(\omega/2) &= \frac{\sin(\omega/2)}{\cos(\omega/2)} \\
               &= \frac{\frac{1}{j} \left[(\cos\frac{\omega}{2} + j\sin\frac{\omega}{2}) - (\cos\frac{\omega}{2} - j\sin\frac{\omega}{2} )\right]}{\left[(\cos\frac{\omega}{2} + j\sin\frac{\omega}{2}) + (\cos\frac{\omega}{2} - j\sin\frac{\omega}{2} )\right]} \\
               &= \frac{1}{j} \frac{e^{j\omega/2} - e^{-j\omega/2}}{e^{j\omega/2} + e^{-j\omega/2}} \\
               &= \frac{1}{j} \frac{1-e^{-j\omega}}{1+e^{-j\omega}} 
\end{aligned}
$$

然后，将 $e^{-j\omega}$ 替换为$z$，带入，则得到了如下式子

$$
\begin{aligned}
|H(z)|^2 &= \frac{\tan^{2N}(\omega_c/2)}{\tan^{2N}(\omega_c/2) + \left( \frac{1}{j} \cdot \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N}} \\
         &= \frac{\tan^{2N}(\omega_c/2)}{\tan^{2N}(\omega_c/2) + (-1)^{N}\left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N}} 
\end{aligned}
$$

通过这个式子，就可以很方便的计算极点与零点。很容易的能看出，这个滤波器的零点是-1，并且为$N$重极点（这里是振幅特性的平方所以不是$2N$）。此时，分母多项式为

$$
\begin{aligned}
\tan^{2N}(\frac{\omega_c}{2}) + (-1)^{N}\left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N} = 0
\end{aligned}
$$ 
 
我们暂且先将这个式子关于 $\left( \frac{1-z^{-1}}{1+z^{-1}} \right)$ 解开, 同时使用欧拉公式 $e^{j(2k + 1)\pi},\hspace{2mm}k=0,1,2,3,\dotsc,2N-1$ 带入，可以得到当$N$为偶数的时候，

$$
\begin{aligned}
\tan^{2N}(\frac{\omega_c}{2}) + \left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N} = 0 \\    
\left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N} = e^{j(2k + 1)\pi} \\ 
\left( \frac{1-z^{-1}}{1+z^{-1}} \right) = e^{j \frac{2k+1}{2N}\pi} \\
\end{aligned}
$$

同样的，当$N$为奇数的时候，

$$
\begin{aligned}
\tan^{2N}(\frac{\omega_c}{2}) - \left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N} = 0 \\    
\left( \frac{1-z^{-1}}{1+z^{-1}} \right)^{2N} = e^{j2k\pi} \\ 
\left( \frac{1-z^{-1}}{1+z^{-1}} \right) = e^{j \frac{k}{N}\pi} \\
\end{aligned}
$$

综合以上，我们可以得到

$$
\begin{aligned}
q_k = \left( \frac{1-z^{-1}}{1+z^{-1}} \right) = \left \{
\begin{array}{l}
\tan(\frac{\omega_c}{2}) \exp \left( j \frac{2k+1}{2N}\pi \right),& \hspace{2mm}N: \text{even number}, \hspace{2mm}k=0,1,2,\cdots,2N-1 \\
\tan(\frac{\omega_c}{2}) \exp \left( j \frac{k}{2N}\pi \right),& \hspace{2mm}N: \text{odd number}, \hspace{2mm}k=0,1,2,\cdots,2N-1 \\
\end{array}
\right.
\end{aligned}
$$

最后，我们解$z$，然后再求出极点，如下所示。

$$
\begin{aligned}
p_k = \frac{1+q_k}{1-q_k}
\end{aligned}
$$
通过上述计算可得到$2N$个极点。为保证滤波器的稳定性，需要从中选取稳定极点。稳定性判据为：极点的模值小于1，即所有极点均位于Z平面单位圆内。值得注意的是，由于FIR滤波器的极点均为0，因此FIR滤波器天然具有稳定性。

选取稳定极点后，可将其代入传递函数表达式，得到滤波器系统函数：

$$
\begin{aligned}
H(z) = \frac{K(1-z^{-1})^{N}}{\prod_{|p_k| < 1} (1-p_k z^{-1})}
\end{aligned}
$$

其中增益系数$K$可通过以下公式计算：

$$
\begin{aligned}
K = \frac{1}{2^N} \cdot \frac{1}{\prod_{|p_k| < 1} (1- p_k)}
\end{aligned}
$$

至此，我们完成了巴特沃斯低通IIR数字滤波器的设计。

&nbsp;
### 巴特沃斯低通数字滤波器设计的实现

首先，还是次数的计算。代码如下

```c++
N = Ceil(0.5*( log10 ( pow (10, IIR_Filter.Stopband_attenuation/10) - 1) / 
	 	   log10 (IIR_Filter.Stopband/IIR_Filter.Cotoff)));
```

 然后，是为了计算极点$p_k$ ，我们先计算$q_k$。这里，将指数函数换为了三角函数。

```c++  
poles_1.Real_part = (0.5)*Cotoff*cos((k+dk)*(pi/N));
poles_1.Imag_Part = (0.5)*Cotoff*sin((k+dk)*(pi/N));	
```

 计算出$q_k$之后，我们计算极点 $p_k$，然后选择稳定的极点。

```c++
for(k = 0;k <= ((2*N)-1) ; k++)
{
    poles_1.Real_part = (0.5)*Cotoff*cos((k+dk)*(pi/N));
    poles_1.Imag_Part= (0.5)*Cotoff*sin((k+dk)*(pi/N));	
 
    poles_2.Real_part = 1 - poles_1.Real_part ;
    poles_2.Imag_Part=   -poles_1.Imag_Part;   
 
    poles_1.Real_part = poles_1.Real_part + 1;
    poles_1.Real_part = poles_1.Real_part;
 
    Complex_Division(poles_1,poles_2,
 	                &poles[count].Real_part,
 	                &poles[count].Imag_Part);
	    
    if(Complex_Abs(poles[count])<1)
    {
        poles[count].Real_part = -poles[count].Real_part;
        poles[count].Imag_Part= -poles[count].Imag_Part;	 
	count++;
        if (count == N) break;
    }
}
``` 

这里的计算，用到了复数的乘法与绝对值。

```c++

int Complex_Division(COMPLEX a,COMPLEX b,
 	              double *Res_Real,double *Res_Imag)
{
    *(Res_Real) =  ((a.Real_part)*(b.Real_part) + (a.Imag_Part)*(b.Imag_Part))/
			       ((b.Real_part)*(b.Real_part) + (b.Imag_Part)*(b.Imag_Part));
 
    *(Res_Imag) =  ((a.Real_part)*(b.Imag_Part) - (a.Imag_Part)*(b.Real_part))/
			       ((b.Real_part)*(b.Real_part) + (b.Imag_Part)*(b.Imag_Part));
    return (int)1; 
}
double Complex_Abs(COMPLEX a)
{
    return (double)(sqrt((a.Real_part)*(a.Real_part) + (a.Imag_Part)*(a.Imag_Part)));
}
```

还有就是K的计算。

```c++
double K_z = 0.0;
for(count = 0;count <= N;count++)   {K_z += *(az+count);}
K_z = (K_z/pow ((double)2,N));
```

最后，使用之前在IIR的间接设计的乘开算法，我们就可以得到一个模拟滤波器的系数了。

所有的代码如下所示:
```c++
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>
 
 
#define     pi     ((double)3.1415926)
 
typedef struct 
{
    double Real_part;
    double Imag_Part;
} COMPLEX;
 
struct DESIGN_SPECIFICATION
{
    double Cotoff;   
    double Stopband;
    double Stopband_attenuation;
};
 
 
 
int Ceil(double input)
{
     if(input != (double)((int)input)) return ((int)input) +1;
     else return ((int)input); 
}
 
 
int Complex_Multiple(COMPLEX a,COMPLEX b,
	                 double *Res_Real,double *Res_Imag)
	
{
       *(Res_Real) =  (a.Real_part)*(b.Real_part) - (a.Imag_Part)*(b.Imag_Part);
       *(Res_Imag)=  (a.Imag_Part)*(b.Real_part) + (a.Real_part)*(b.Imag_Part);	   
	 return (int)1; 
}
 
 int Complex_Division(COMPLEX a,COMPLEX b,
 	                                  double *Res_Real,double *Res_Imag)
{
        *(Res_Real) =  ((a.Real_part)*(b.Real_part) + (a.Imag_Part)*(b.Imag_Part))/
			           ((b.Real_part)*(b.Real_part) + (b.Imag_Part)*(b.Imag_Part));
		
	 *(Res_Imag)=  ((a.Real_part)*(b.Imag_Part) - (a.Imag_Part)*(b.Real_part))/
			           ((b.Real_part)*(b.Real_part) + (b.Imag_Part)*(b.Imag_Part));
 
	 return (int)1; 
}
 
double Complex_Abs(COMPLEX a)
{
      return (double)(sqrt((a.Real_part)*(a.Real_part) + (a.Imag_Part)*(a.Imag_Part)));
}
 
double IIRFilter  (double *a, int Lenth_a,
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
 
 
int Direct( double Cotoff,
	           double Stopband,
	           double Stopband_attenuation,
	           int N,
	           double *az,double *bz)
{
      printf("Wc =  %lf  [rad/sec] \n" ,Cotoff);
      printf("Ws =  %lf  [rad/sec] \n" ,Stopband);
      printf("As  =  %lf  [dB] \n" ,Stopband_attenuation);
      printf("--------------------------------------------------------\n" );
      printf("N:  %d  \n" ,N);
      printf("--------------------------------------------------------\n" );
 
      COMPLEX poles[N],poles_1,poles_2;
      double dk = 0;
      int k = 0;
	int count = 0,count_1 = 0;;
	
      if((N%2) == 0) dk = 0.5;
      else dk = 0;
 
      for(k = 0;k <= ((2*N)-1) ; k++)
      {
             poles_1.Real_part = (0.5)*Cotoff*cos((k+dk)*(pi/N));
	     poles_1.Imag_Part= (0.5)*Cotoff*sin((k+dk)*(pi/N));	
 
             poles_2.Real_part = 1 - poles_1.Real_part ;
	     poles_2.Imag_Part=   -poles_1.Imag_Part;   
 
	     poles_1.Real_part = poles_1.Real_part + 1;
             poles_1.Real_part = poles_1.Real_part;
 
             Complex_Division(poles_1,poles_2,
 	                                &poles[count].Real_part,
 	                                &poles[count].Imag_Part);
	    
	     if(Complex_Abs(poles[count])<1)
	     {
	           poles[count].Real_part = -poles[count].Real_part;
		   poles[count].Imag_Part= -poles[count].Imag_Part;	 
	           count++;
		   if (count == N) break;
	     }
   
      } 
 
      printf("pk =   \n" );   
      for(count = 0;count < N ;count++)
      {
           printf("(%lf) + (%lf i) \n" ,-poles[count].Real_part
		                          	,-poles[count].Imag_Part);
      }
      printf("--------------------------------------------------------\n" );
 
      COMPLEX Res[N+1],Res_Save[N+1];
 
      Res[0].Real_part = poles[0].Real_part; 
      Res[0].Imag_Part= poles[0].Imag_Part;
 
      Res[1].Real_part = 1; 
      Res[1].Imag_Part= 0;
 
 
      for(count_1 = 0;count_1 < N-1;count_1++)
      {
	     for(count = 0;count <= count_1 + 2;count++)
	     {
	          if(0 == count)
		   {
 	                Complex_Multiple(Res[count], poles[count_1+1],
						           &(Res_Save[count].Real_part),
						           &(Res_Save[count].Imag_Part));
	          }
 
	          else if((count_1 + 2) == count)
	          {
	                 Res_Save[count].Real_part  += Res[count - 1].Real_part;
			    Res_Save[count].Imag_Part += Res[count - 1].Imag_Part;	
	          }		  
		    else 
		    {
 	                 Complex_Multiple(Res[count], poles[count_1+1],
						           &(Res_Save[count].Real_part),
						           &(Res_Save[count].Imag_Part));
				
			    Res_Save[count].Real_part  += Res[count - 1].Real_part;
			    Res_Save[count].Imag_Part += Res[count - 1].Imag_Part;
		    }
	     }
 
	     for(count = 0;count <= N;count++)
	     {
	           Res[count].Real_part = Res_Save[count].Real_part; 
                   Res[count].Imag_Part= Res_Save[count].Imag_Part;
		    *(az + N - count) = Res[count].Real_part;
	     }
      }
 
        double K_z = 0.0;
	for(count = 0;count <= N;count++)   {K_z += *(az+count);}
	K_z = (K_z/pow ((double)2,N));
	printf("K =  %lf \n" , K_z);
 
	for(count = 0;count <= N;count++)
	{
             Res[count].Real_part = 0;
	     Res[count].Imag_Part= 0;
	     Res_Save[count].Real_part = 0;
	     Res_Save[count].Imag_Part= 0;
	}
 
      COMPLEX zero;
 
      zero.Real_part  =  1;
      zero.Imag_Part =  0;
 
      Res[0].Real_part = 1; 
      Res[0].Imag_Part= 0;
      Res[1].Real_part = 1; 
      Res[1].Imag_Part= 0;
 
      for(count_1 = 0;count_1 < N-1;count_1++)
      {
	     for(count = 0;count <= count_1 + 2;count++)
	     {
	          if(0 == count)
		   {
 	                Complex_Multiple(Res[count], zero,
						           &(Res_Save[count].Real_part),
						           &(Res_Save[count].Imag_Part));
	          }
 
	          else if((count_1 + 2) == count)
	          {
	                 Res_Save[count].Real_part  += Res[count - 1].Real_part;
			    Res_Save[count].Imag_Part += Res[count - 1].Imag_Part;	
	          }		  
		    else 
		    {
 	                 Complex_Multiple(Res[count],zero,
						           &(Res_Save[count].Real_part),
						           &(Res_Save[count].Imag_Part));
				
			    Res_Save[count].Real_part  += Res[count - 1].Real_part;
			    Res_Save[count].Imag_Part += Res[count - 1].Imag_Part;
		    }
	     }
 
	     for(count = 0;count <= N;count++)
	     {
	           Res[count].Real_part = Res_Save[count].Real_part; 
                 Res[count].Imag_Part= Res_Save[count].Imag_Part;
		    *(bz + N - count) = Res[count].Real_part;
	     }
      }
 
	for(count = 0;count <= N;count++)
	{
           *(bz + N - count) = *(bz + N - count) * K_z;
	}
      	//------------------------display---------------------------------//
      printf("bz =  [" );   
      for(count= 0;count <= N ;count++)
      {
           printf("%lf ", *(bz+count));
      }
      printf(" ] \n" );
      printf("az =  [" );   
      for(count= 0;count <= N ;count++)
      {
           printf("%lf ", *(az+count));
      }
      printf(" ] \n" );
      printf("--------------------------------------------------------\n" );
	 
      return (int)1;
}
 

int main(void)
{
     int count;
 
     struct DESIGN_SPECIFICATION IIR_Filter;
 
     IIR_Filter.Cotoff      = (double)(pi/4);         //[red]
     IIR_Filter.Stopband = (double)((pi*3)/4);   //[red]
     IIR_Filter.Stopband_attenuation = 30;        //[dB]
 
     int N;
 
     IIR_Filter.Cotoff = 2 * tan((IIR_Filter.Cotoff)/2);            //[red/sec]
     IIR_Filter.Stopband = 2 * tan((IIR_Filter.Stopband)/2);   //[red/sec]
 
     N = Ceil(0.5*( log10 ( pow (10, IIR_Filter.Stopband_attenuation/10) - 1) / 
	 	              log10 (IIR_Filter.Stopband/IIR_Filter.Cotoff)));
 
   
     double az[N+1] , bz[N+1];
     Direct(IIR_Filter.Cotoff,
	         IIR_Filter.Stopband,
	         IIR_Filter.Stopband_attenuation,
               N,
	         az,bz);
 
    double *Memory_Buffer;
    Memory_Buffer = (double *) malloc(sizeof(double)*(N+1));  
    memset(Memory_Buffer,
                0,
                sizeof(double)*(N+1));
 
     FILE* Input_Data;
     FILE* Output_Data;
 
     double Input = 0 ;
    double Output = 0;
	 
     Input_Data   = fopen("input.dat","r"); 
     Output_Data = fopen("output.txt","w"); 
 
     while(1)
     {
          if(fscanf(Input_Data, "%lf", &Input) == EOF)  break;
 
          Output = IIRFilter(  az, (N+1),
                                      bz, (N+1),
                                      Input,
                                      Memory_Buffer );
          
          fprintf(Output_Data,"%lf,",Output);
     	}
 
        
	
     printf("Finish \n" );
	 
     return (int)0;
}
```

&nbsp;
## 直接设计实现的IIR滤波器的性能

&nbsp;
### 设计指标
- 截止频率 $\omega_c=\frac{\pi}{4}[rad]$
- 阻带起始频率 $\omega_s=\frac{3}{4}\pi[rad]$
- 阻带衰减 $A_s=30[dB]$

&nbsp;
### 程序执行结果
使用上述程序，执行结果如下。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-2/Design-IIR-Filter-2-1.jpeg" width="450"></div>

其频率响应如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-2/Design-IIR-Filter-2-2.jpeg" width="450"></div>

首先，设定采样频率为10kHz，计算出如下非正规化频率:

$$
\begin{aligned}
\omega _p &= 2\pi f_p T_S = \frac{\pi}{4},& f_c &= 1.25 kHz \\
\omega _s &= 2\pi f_s T_s = \frac{3\pi}{4},& f_s &= 3.75 kHz \\
\end{aligned}
$$

我们的为验证滤波器的性能，其输入信号确定为0.5kHz与4kHz的正弦叠加信号，经过滤波处理后的输入输出波形如下图所示：

<table>
    <tr>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-2/Design-IIR-Filter-2-3.jpeg" width="600">
            </div> 
        </td>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-2/Design-IIR-Filter-2-4.jpeg" width="600">
            </div> 
        </td>
    </tr>
</table> 

其中，红色的----是Matlab计算的输出，粉色的o是用Ｃ语言计算的输出，蓝色的线是理想输出（也就是混合前的0.5kHz信号）。
到此，我们就实现了一个IIR滤波器。

&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <!-- <a href="{% link _posts/2013-06-10-IIR-Filter-Design-1.md %}"> [下一回] >> </a> -->
    </span>　
        <a href="{{ site.baseurl }}/2013/06/10/IIR-Filter-Design-1"> << [上一回] </a>
</div>