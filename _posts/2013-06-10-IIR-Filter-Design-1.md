---
layout: post
title: "5.IIR滤波器的间接设计"
date: 2013-06-10 12:46:10 +0800
tags: 数字信号处理
# categories: jekyll update
---

在数字滤波器设计中,间接设计法是一种重要的方法。其基本思路是先根据给定参数设计模拟滤波器,然后通过变数变换得到数字滤波器。作为数字滤波器设计基础的模拟滤波器称为原型滤波器。本文将介绍最基础的原型滤波器——巴特沃斯低通滤波器。

首先, 由于IIR滤波器不具有线性相位特性,因此我们只需考虑其振幅特性:

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }}  \Bigg | _{s=j \Omega} , \hspace{3mm} \Omega = - \infty \sim  + \infty 
\end{aligned}
$$

其中,$N$是滤波器的次数,$\Omega_c$是截止频率。从振幅特性可以看出,这是一个单调递减的函数,不存在纹波。设计时需要先计算满足设计参数的次数$N$。首先由阻带频率计算阻带衰减:

$$
\begin{aligned}
A_s = -20 \log_{10}|H_a(j\Omega_s)|
\end{aligned}
$$

将巴特沃斯低通滤波器的振幅特性代入上式:

$$
\begin{aligned}
A_s &= -20 \log_{10} \left | \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }} \right | \\
    &= 10 \log_{10} \left [ 1 + \left ( \frac{\Omega_s}{\Omega_c} \right )^{2N}  \right ]
\end{aligned}
$$

可解得次数$N$为:

$$
\begin{aligned}
N = \frac{1}{2} \frac{\log_{10} (10^{\frac{A_s}{10}} - 1)}{\log_{10} \left ( \frac{\Omega_s}{\Omega_c} \right )}
\end{aligned}
$$

注意,$N$只能为正数,若结果为小数则需向上取整。

&nbsp;
### 巴特沃斯滤波器的传递函数
巴特沃斯低通滤波器的传递函数可由其振幅特性的分母多项式求得:

$$
\begin{aligned}
1 + (-1)^{N} \left( \frac{s}{\Omega_c}  \right)^{2N} = 0
\end{aligned}
$$

根据$S$解开可得极点。为方便处理,分两种情况讨论。当$N$为偶数时:

$$
\begin{aligned}
1 + \left( \frac{s}{\Omega_c}  \right)^{2N} &= 0 \\
\left( \frac{s}{\Omega_c}  \right)^{2N} &= e^{j(2k + 1)\pi} \\
s &= \Omega_c e^{j \frac{2k+1}{2N}\pi}
\end{aligned}
$$

这里使用了欧拉公式$e^{j(2k + 1)\pi},\hspace{2mm}k=0,1,2,3,\dotsc,2N-1$。

当$N$为奇数时:

$$
\begin{aligned}
1 - \left( \frac{s}{\Omega_c}  \right)^{2N} &= 0 \\
\left( \frac{s}{\Omega_c}  \right)^{2N} &= e^{j2k\pi} \\
s &= \Omega_c e^{j \frac{k}{2N}\pi}
\end{aligned}
$$

通过使用欧拉公式，我们可以得到巴特沃斯滤波器极点的一般表达式。根据滤波器次数$N$的奇偶性，极点位置可表示为:

$$
\begin{equation}
p_k = \left \{
\begin{array}{l}
\Omega_c \exp \left( j \frac{2k+1}{2N}\pi \right), \hspace{2mm}N: \text{even number}, \hspace{2mm}k=0,1,2,\cdots,2N-1 \\
\Omega_c \exp \left( j \frac{k}{2N}\pi \right), \hspace{2mm}N: \text{odd number}, \hspace{2mm}k=0,1,2,\cdots,2N-1
\end{array}
\right.
\end{equation}
$$

通过上述公式计算得到的极点具有以下特征:
- 在$s$平面内呈圆形分布
- 分布圆的半径为$\Omega_c$
- 极点间隔相等
- 总计有$2N$个极点

为确保IIR滤波器的稳定性，我们只能选择位于$S$平面左半平面的极点。基于所选定的稳定极点，模拟滤波器的传递函数可表示为:

$$
\begin{aligned}
H_a(s) = \prod _{Re[p_k] < 0} \frac{\Omega_c}{s-p_k}
\end{aligned}
$$




&nbsp;
### 巴特沃斯滤波器的实现

首先,我们需要先计算滤波器的阶数$N$。根据之前推导的公式:

$$
\begin{aligned}
N = \frac{1}{2} \frac{\log_{10} (10^{\frac{A_s}{10}} - 1)}{\log_{10} \left ( \frac{\Omega_s}{\Omega_c} \right )}
\end{aligned}
$$

```c++
N = Ceil(0.5*( log10 ( pow (10, Stopband_attenuation/10) - 1) / 
	 	            log10 (Stopband/Cotoff) ))
```


接下来需要计算滤波器的极点。由于涉及复数运算,我们需要定义复数结构体。为了简化计算,可以将自然指数形式转换为三角函数形式:


$$
\begin{aligned}
p_k = \left \{
\begin{array}{l}
\Omega_c \cos\frac{k + (1/2)}{N}\pi + j\Omega_c \sin\frac{k+(1/2)}{N}\pi&, \hspace{2mm}N: \text{even number}, \hspace{2mm}k=0,1,2,\cdots,2N-1 \\
\Omega_c \cos\frac{k}{N}\pi + j\Omega_c \sin\frac{k}{N}\pi &, \hspace{2mm}N: \text{odd number}, \hspace{2mm}k=0,1,2,\cdots,2N-1
\end{array}
\right.
\end{aligned}
$$


这样可以分别计算实部和虚部。其代码实现如下: 

```c++
typedef struct 
{
    double Real_part;
    double Imag_Part;
} COMPLEX;
 
COMPLEX poles[N];
 
for(k = 0;k <= ((2*N)-1) ; k++)
{
    if(Cotoff*cos((k+dk)*(pi/N)) < 0)
    {
        poles[count].Real_part = -Cotoff*cos((k+dk)*(pi/N));
	    poles[count].Imag_Part = -Cotoff*sin((k+dk)*(pi/N));	   
        count++;
	    if (count == N) break;
    }
}
```

计算出稳定极点后,就可以构造传递函数:

$$
\begin{aligned}
H_a(s) = \prod _{Re[p_k] < 0} \frac{\Omega_c}{s-p_k}
\end{aligned}
$$

这里，为了得到模拟滤波器的系数，需要将分母乘开。很显然，这里的极点不一定是整数，或者来说，这里的乘开需要做复数运算。其复数的乘法代码如下，


```c++
int Complex_Multiple(COMPLEX a,COMPLEX b,
	                 double *Res_Real,double *Res_Imag)
{
    *(Res_Real) = (a.Real_part)*(b.Real_part) - (a.Imag_Part)*(b.Imag_Part);
    *(Res_Imag) = (a.Imag_Part)*(b.Real_part) + (a.Real_part)*(b.Imag_Part);	   
	return (int)1; 
}
```

有了乘法代码之后，我们现在简单的情况下，看看其如何计算其滤波器系数。我们做如下假设

$$
\begin{aligned}
N=2, p_1 = a_1+ka_2, p_2 = b_1 + jb_2
\end{aligned}
$$

其传递函数为:

$$
\begin{aligned}
H_a(s) = \frac{1}{(s-(a_1+ka_2))(s-(b_1+kb_2))}
\end{aligned}
$$

展开计算过程如下图所示:

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Complex_Multiple.jpeg" width="450"></div>

高阶的情况下也一样，可以通过重复这个过程来计算。其代码实现为

```c++
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
    *(b+N) = *(a+N);
}
```

至此,我们就完成了巴特沃斯模拟低通滤波器的设计。


&nbsp;
### 双1次z变换的原理

双1次z变换是将模拟滤波器转换为数字滤波器的一种重要方法。通过建立s平面到z平面的映射关系，可以实现模拟滤波器到数字滤波器的转换。

让我们以一个简单的一阶模拟滤波器为例,其传递函数为:

$$
\begin{aligned}
H(s) = \frac{b}{s+a}
\end{aligned}
$$

对其进行拉普拉斯逆变换,可得到时域连续微分方程:

$$
\begin{aligned}
\frac{dy(t)}{dt} + ay(t) = bx(t)
\end{aligned}
$$

其中$x(t)$和$y(t)$分别表示输入和输出信号。假设采样周期为$T$,用差分方程近似替代微分方程:

$$
\begin{aligned}
\frac{1}{T} \left[ y(n) - y(n-1)\right] + \frac{a}{2} \left[ y(n) + y(n-1)\right] = \frac{b}{2} \left[ x(n) + x(n-1)\right] 
\end{aligned}
$$

对上式进行z变换并化简,可得:

$$
\begin{aligned}
H(z) = \frac{Y(z)}{X(z)} = \frac{b}{\frac{2}{T} \frac{1 - z^{(-1)}}{1 + z^{(-1)}} + a} = H(s) \Big|_{s=f(z)}
\end{aligned}
$$

由此可得s平面到z平面的映射关系:

$$
\begin{aligned}
s = f(z) = \frac{2}{T} \frac{1 - z^{(-1)}}{1 + z^{(-1)}}
\end{aligned}
$$

这个映射关系对高阶系统同样适用,因为高阶系统可以视为一阶系统的并联。

将$z=e^{j\omega}$和$s = \delta+j\Omega$代入上式:

$$
\begin{aligned}
s &= \frac{2}{T} \frac{1 - e^{-j\omega}}{1 + e^{-j\omega}} \\
  &= \frac{2}{T} \frac{1 - \cos \omega - j\sin\omega}{1 + \cos \omega + j\sin \omega} \\
  &= \frac{2}{T} \frac{1 - (\cos^{2}\omega + \sin^{2}\omega) - j2\sin\omega}{2+2\cos\omega} \\
  &= 0 + j\frac{2}{T} \tan\frac{\omega}{2} = \delta + j\Omega
\end{aligned}
$$

最终得到$\Omega$与$\omega$的对应关系:

$$
\begin{aligned}
\Omega = \frac{2}{T} \tan\frac{\omega}{2}
\end{aligned}
$$

这个对应关系在IIR滤波器设计中具有重要意义。由于我们的目标是设计数字滤波器,但采用的是间接设计法,因此需要先将数字滤波器的指标转换为模拟滤波器的指标,再基于转换后的指标设计模拟滤波器。值得注意的是,采样时间T的选择较为灵活,通常取1s即可简化计算。

&nbsp;
### 双1次z变换的实现（C语言）
我们设计好的巴特沃斯低通滤波器的传递函数如下所示。

$$
\begin{aligned}
H_a(s) = \frac{1}{a_{0}S^{n} + a_{1}S^{n-1} + \dotsc + a_{n-1}S + a_{n}} 
\end{aligned}
$$

我们将其进行双1次z变换，我们可以得到如下式子

$$
\begin{aligned}
H(z) &= H_a(s) \Big|_{s=\frac{2}{T}\frac{1 - z^{(-1)}}{1 + z^{(-1)}}} \\
     &= \frac{1}{a_{0}\left(\frac{2}{T}\right)^{n}(1-z^{-1})^{n} + 
                 a_{1}\left(\frac{2}{T}\right)^{n-1}(1-z^{-1})^{n-1}(1+z^{-1}) +
                 \dotsc + 
                 a_{n-m}\left(\frac{2}{T}\right)^{n-m}(1-z^{-1})^{n-m}(1+z^{-1})^{m}+
                 \dotsc + 
                 a_{n}(1+z^{-1})^{n}}  \\
\end{aligned}
$$

可以看出，我们还是需要将式子乘开，进行合并同类项，这个跟之前说的算法相差不大。其代码为。


```c++
for(Count = 0;Count<=N;Count++)
{    	
    for(Count_Z = 0;Count_Z <= N;Count_Z++)
	{
	    Res[Count_Z] = 0;
		Res_Save[Count_Z] = 0;	 
	}
    Res_Save [0] = 1;
	
    for(Count_1 = 0; Count_1 < N-Count;Count_1++)
	{
		for(Count_2 = 0; Count_2 <= Count_1+1;Count_2++)
	    {
	      	if(Count_2 == 0)
                Res[Count_2] += Res_Save[Count_2];	
            else if((Count_2 == (Count_1+1))&&(Count_1 != 0)) 
                Res[Count_2] += -Res_Save[Count_2 - 1]; 
            else  
                Res[Count_2] += Res_Save[Count_2] - Res_Save[Count_2 - 1];

    	    for(Count_Z = 0;Count_Z<= N;Count_Z++)
		    {
 		        Res_Save[Count_Z]  =  Res[Count_Z] ;
			    Res[Count_Z]  = 0;
		    }			
	    }

		for(Count_1 = (N-Count); Count_1 < N;Count_1++)
        {
            for(Count_2 = 0; Count_2 <= Count_1+1;Count_2++)
            {
                if(Count_2 == 0) 
                    Res[Count_2] += Res_Save[Count_2];	 
                else if((Count_2 == (Count_1+1))&&(Count_1 != 0)) 
                    Res[Count_2] += Res_Save[Count_2 - 1];
                else  
                    Res[Count_2] += Res_Save[Count_2] + Res_Save[Count_2 - 1];	
            }
                
            for(Count_Z = 0;Count_Z<= N;Count_Z++)
            {
                Res_Save[Count_Z]  =  Res[Count_Z] ;
                Res[Count_Z]  = 0;
            }
        }
	        
        for(Count_Z = 0;Count_Z<= N;Count_Z++)
		{
            *(az+Count_Z) += pow(2,N-Count) * (*(as+Count)) * Res_Save[Count_Z];
	        *(bz+Count_Z) += (*(bs+Count)) * Res_Save[Count_Z];		     
		}	
    }
}
```

到此，我们就已经实现了一个数字滤波器。

&nbsp;
### IIR滤波器的间接设计代码（C语言）

```c++
#include <stdio.h>
#include <math.h>
#include <malloc.h>
#include <string.h>
 
 
#define     pi     ((double)3.1415926)
 
 
struct DESIGN_SPECIFICATION
{
    double Cotoff;   
    double Stopband;
    double Stopband_attenuation;
};
 
typedef struct 
{
    double Real_part;
    double Imag_Part;
} COMPLEX;
 
int Ceil(double input)
{
    if(input != (int)input) return ((int)input) +1;
    else return ((int)input); 
}
 

int Complex_Multiple(COMPLEX a,COMPLEX b,
                     double *Res_Real, double *Res_Imag)	
{
    *(Res_Real) =  (a.Real_part)*(b.Real_part) - (a.Imag_Part)*(b.Imag_Part);
    *(Res_Imag)=  (a.Imag_Part)*(b.Real_part) + (a.Real_part)*(b.Imag_Part);	   
	return (int)1; 
}
 

int Buttord(double Cotoff,
	        double Stopband,
	        double Stopband_attenuation)
{
   int N;
 
   printf("Wc =  %lf  [rad/sec] \n" ,Cotoff);
   printf("Ws =  %lf  [rad/sec] \n" ,Stopband);
   printf("As  =  %lf  [dB] \n" ,Stopband_attenuation);
   printf("--------------------------------------------------------\n" );
	 
   N = Ceil(0.5*( log10 ( pow (10, Stopband_attenuation/10) - 1) / 
	 	          log10 (Stopband/Cotoff) ));
   
   
   return (int)N;
}
 
 
int Butter(int N, double Cotoff,
	       double *a, double *b)
{
    double dk = 0;
    int k = 0;
    int count = 0,count_1 = 0;
    COMPLEX poles[N];
    COMPLEX Res[N+1],Res_Save[N+1];
 
    if((N%2) == 0) dk = 0.5;
    else dk = 0;
 
    for(k = 0;k <= ((2*N)-1) ; k++)
    {
         if(Cotoff*cos((k+dk)*(pi/N)) < 0)
         {
            poles[count].Real_part = -Cotoff*cos((k+dk)*(pi/N));
		    poles[count].Imag_Part= -Cotoff*sin((k+dk)*(pi/N));	   
            count++;
	        if (count == N) break;
         }
    } 
 
    printf("Pk =   \n" );   
    for(count = 0;count < N ;count++)
    {
       printf("(%lf) + (%lf i) \n" ,-poles[count].Real_part
	                               ,-poles[count].Imag_Part);
    }
    printf("--------------------------------------------------------\n" );
	 
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
			   //printf( "Res_Save : (%lf) + (%lf i) \n" ,Res_Save[0].Real_part,Res_Save[0].Imag_Part);
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
 
                //printf( "Res          : (%lf) + (%lf i) \n" ,Res[count - 1].Real_part,Res[count - 1].Imag_Part);
			    //printf( "Res_Save : (%lf) + (%lf i) \n" ,Res_Save[count].Real_part,Res_Save[count].Imag_Part);
				
			    Res_Save[count].Real_part += Res[count - 1].Real_part;
			    Res_Save[count].Imag_Part += Res[count - 1].Imag_Part;
			
			    //printf( "Res_Save : (%lf) + (%lf i) \n" ,Res_Save[count].Real_part,Res_Save[count].Imag_Part);	
		    }
		    //printf("There \n" );
	     }
 
	     for(count = 0;count <= N;count++)
	     {
	        Res[count].Real_part = Res_Save[count].Real_part; 
            Res[count].Imag_Part= Res_Save[count].Imag_Part;	 
		    *(a + N - count) = Res[count].Real_part;
	     }
		 
	     //printf("There!! \n" );
		 
    	}
 
     *(b+N) = *(a+N);
 
     //------------------------display---------------------------------//
     printf("bs =  [" );   
     for(count = 0;count <= N ;count++)
     {
           printf("%lf ", *(b+count));
     }
     printf(" ] \n" );
 
     printf("as =  [" );   
     for(count = 0;count <= N ;count++)
     {
           printf("%lf ", *(a+count));
     }
     printf(" ] \n" );
 
     printf("--------------------------------------------------------\n" );
 
     return (int) 1;
}
 
 
int Bilinear(int N, 
	         double *as, double *bs,
	         double *az, double *bz)
{
    int Count = 0,Count_1 = 0,Count_2 = 0,Count_Z = 0;
    double Res[N+1];
	double Res_Save[N+1]; 
 
    for(Count_Z = 0;Count_Z <= N;Count_Z++)
	{
        *(az+Count_Z)  = 0;
		*(bz+Count_Z)  = 0;
	}
 
	
	for(Count = 0;Count<=N;Count++)
	{    	
	    for(Count_Z = 0;Count_Z <= N;Count_Z++)
	    {
	      	Res[Count_Z] = 0;
		    Res_Save[Count_Z] = 0;	 
	    }
        Res_Save [0] = 1;
	
	    for(Count_1 = 0; Count_1 < N-Count;Count_1++)
	    {
			for(Count_2 = 0; Count_2 <= Count_1+1;Count_2++)
	      	{
	      		if(Count_2 == 0)  
			    {
			        Res[Count_2] += Res_Save[Count_2];
				    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    } 	
 
			    else if((Count_2 == (Count_1+1))&&(Count_1 != 0))  
			    {
			        Res[Count_2] += -Res_Save[Count_2 - 1];   
                    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    } 
 
			    else  
			    {
			        Res[Count_2] += Res_Save[Count_2] - Res_Save[Count_2 - 1];
				    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    }				 
			}
 
            //printf( "Res : ");
		    for(Count_Z = 0;Count_Z<= N;Count_Z++)
		    {
 		        Res_Save[Count_Z]  =  Res[Count_Z] ;
			    Res[Count_Z]  = 0;
				//printf( "[%d]  %lf  " ,Count_Z, Res_Save[Count_Z]);     
		    }
		    //printf(" \n" );
	    }
 
		for(Count_1 = (N-Count); Count_1 < N;Count_1++)
	    {
            for(Count_2 = 0; Count_2 <= Count_1+1;Count_2++)
	      	{
	      		if(Count_2 == 0)  
			    {
			        Res[Count_2] += Res_Save[Count_2];
				    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    } 	
                else if((Count_2 == (Count_1+1))&&(Count_1 != 0))  
			    {
			        Res[Count_2] += Res_Save[Count_2 - 1];
                    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    } 
			    else  
			    {
			        Res[Count_2] += Res_Save[Count_2] + Res_Save[Count_2 - 1];
				    //printf( "Res[%d] %lf  \n" , Count_2 ,Res[Count_2]);
			    }				 
			}
 
            //	printf( "Res : ");
		    for(Count_Z = 0;Count_Z<= N;Count_Z++)
		    {
 		        Res_Save[Count_Z]  =  Res[Count_Z] ;
			    Res[Count_Z]  = 0;
				//printf( "[%d]  %lf  " ,Count_Z, Res_Save[Count_Z]);     
		    }
		    //printf(" \n" );
	    }
 
        //printf( "Res : ");
		for(Count_Z = 0;Count_Z<= N;Count_Z++)
		{
            *(az+Count_Z) +=  pow(2,N-Count)  *  (*(as+Count)) * Res_Save[Count_Z];
			*(bz+Count_Z) +=  (*(bs+Count)) * Res_Save[Count_Z];		
            //printf( "  %lf  " ,*(bz+Count_Z));         
		}	
		//printf(" \n" ); 
	}
 
    for(Count_Z = N;Count_Z >= 0;Count_Z--)
    {
        *(bz+Count_Z) =  (*(bz+Count_Z))/(*(az+0));
        *(az+Count_Z) =  (*(az+Count_Z))/(*(az+0));
    }

	//------------------------display---------------------------------//
    printf("bz =  [" );   
    for(Count_Z= 0;Count_Z <= N ;Count_Z++)
    {
        printf("%lf ", *(bz+Count_Z));
    }
    printf(" ] \n" );
    printf("az =  [" );   
    for(Count_Z= 0;Count_Z <= N ;Count_Z++)
    {
       printf("%lf ", *(az+Count_Z));
    }
    printf(" ] \n" );
    printf("--------------------------------------------------------\n" );
 
	return (int) 1;
}
 
 
 
  
int main(void)
{
    int count;
 
    struct DESIGN_SPECIFICATION IIR_Filter;
 
    IIR_Filter.Cotoff   = (double)(pi/2);         //[red]
    IIR_Filter.Stopband = (double)((pi*3)/4);   //[red]
    IIR_Filter.Stopband_attenuation = 30;        //[dB]
  
    int N;
 
    IIR_Filter.Cotoff = 2 * tan((IIR_Filter.Cotoff)/2);            //[red/sec]
    IIR_Filter.Stopband = 2 * tan((IIR_Filter.Stopband)/2);   //[red/sec]
 
    N = Buttord(IIR_Filter.Cotoff,
	 	        IIR_Filter.Stopband,
	 	        IIR_Filter.Stopband_attenuation);
    printf("N:  %d  \n" ,N);
    printf("--------------------------------------------------------\n" );
 
    double as[N+1] , bs[N+1];
    Butter(N, 
           IIR_Filter.Cotoff,
	       as,
	       bs); 
 
    double az[N+1] , bz[N+1];
     
    Bilinear(N, 
	         as,bs,
	         az,bz);
 
    printf("Finish \n" );
    
    return (int)0;
}
```

&nbsp;
### 间接设计实现的IIR滤波器的性能

&nbsp;
#### 设计指标
- 截止频率 $\omega_c=\frac{\pi}{2}[rad]$
- 阻带起始频率 $\omega_s=\frac{3}{4}\pi[rad]$
- 阻带衰减 $A_s=30[dB]$

&nbsp;
#### 程序执行结果
使用上述程序，gcc编译通过，执行结果如下。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Design-IIR-Filter-1-1.jpeg" width="450"></div>

&nbsp;
&nbsp;
其频率响应如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Design-IIR-Filter-1-2.jpeg" width="500"></div>

&nbsp;
&nbsp;
<div style="font-size:16px">
    <span style="float:right"> 
        <a href="{{ site.baseurl }}/2013/06/11/IIR-Filter-Design-2"> [下一回] >> </a>
    </span>　
        <a href="{{ site.baseurl }}/2013/06/05/IIR-Filter"> << [上一回] </a>
</div>