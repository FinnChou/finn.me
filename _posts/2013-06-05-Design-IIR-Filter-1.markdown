---
layout: post
title: "[数字信号处理]5.IIR滤波器的间接设计"
date: 2013-06-10 12:46:10 +0800
# categories: jekyll update
---

&nbsp;
## 模拟滤波器的设计

&nbsp;
### 巴特沃斯滤波器的次数
根据给定的参数设计模拟滤波器，然后进行变数变换，求取数字滤波器的方法，称为滤波器的间接设计。做为数字滤波器的设计基础的模拟滤波器，称之为原型滤波器。这里，我们首先介绍的是最简单最基础的原型滤波器，巴特沃斯低通滤波器。由于IIR滤波器不具有线性相位特性，因此不必考虑相位特性，直接考虑其振幅特性。

$$
\begin{aligned}
|H_a(j\Omega)| = \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }}  \Bigg | _{s=j \Omega} , \hspace{3mm} \Omega = - \infty \sim  + \infty 
\end{aligned}
$$

在这里，$N$是滤波器的次数，$\Omega_c$是截止频率。从上式的振幅特性可以看出，这个是单调递减的函数，其振幅特性是不存在纹波的。设计的时候，一般需要先计算跟所需要设计参数相符合的次数$N$。首先，就需要先由阻带频率，计算出阻带衰减

$$
\begin{aligned}
A_s = -20 \log_{10}|H_a(j\Omega_s)|
\end{aligned}
$$

将巴特沃斯低通滤波器的振幅特性，直接带入上式，则有

$$
\begin{aligned}
A_s &= -20 \log_{10} \left | \frac{1}{\sqrt{1+(-1)^{N} \left( \frac{S}{\Omega_c} \right)^{2N} }} \right | \\
    &= 10 \log_{10} \left [ 1 + \left ( \frac{\Omega_s}{\Omega_c} \right )^{2N}  \right ]
\end{aligned}
$$

可解得, 次数 $N$ 为

$$
\begin{aligned}
N = \frac{1}{2} \frac{\log_{10} (10^{\frac{A_s}{10}} - 1)}{\log_{10} \left ( \frac{\Omega_s}{\Omega_c} \right )}
\end{aligned}
$$

当然，这里的$N$只能为正数，因此，若结果为小数，则舍弃小数，向上取整.

&nbsp;
### 巴特沃斯滤波器的传递函数
巴特沃斯低通滤波器的传递函数，可由其振幅特性的分母多项式求得。其分母多项式

$$
\begin{aligned}
1 + (-1)^{N} \left( \frac{s}{\Omega_c}  \right)^{2N} = 0
\end{aligned}
$$

根据$S$解开，可以得到极点。这里，为了方便处理，我们分为两种情况去解这个方程。当$N$为偶数的时候，

$$
\begin{aligned}
1 + \left( \frac{s}{\Omega_c}  \right)^{2N} &= 0 \\
\left( \frac{s}{\Omega_c}  \right)^{2N} &= e^{j(2k + 1)\pi} \\
s &= \Omega_c e^{j \frac{2k+1}{2N}\pi}
\end{aligned}
$$

这里，使用了欧拉公式 $e^{j(2k + 1)\pi},\hspace{2mm}k=0,1,2,3,\dotsc,2N-1$。

同样的，当N为奇数的时候，

$$
\begin{aligned}
1 - \left( \frac{s}{\Omega_c}  \right)^{2N} &= 0 \\
\left( \frac{s}{\Omega_c}  \right)^{2N} &= e^{j2k\pi} \\
s &= \Omega_c e^{j \frac{k}{2N}\pi}
\end{aligned}
$$

同样的，这里也使用了欧拉公式。
归纳以上，极点的解为

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

上式所求得的极点，是在$s$平面内，在半径为$\Omega_c$的圆上等间距的点，其数量为$2N$个。为了使得其IIR滤波器稳定，那么，只能选取极点在$S$平面左半平面的点。选定了稳定的极点之后，其模拟滤波器的传递函数就可由下式求得。

$$
\begin{aligned}
H_a(s) = \prod _{Re[p_k] < 0} \frac{\Omega_c}{s-p_k}
\end{aligned}
$$

&nbsp;
### 巴特沃斯滤波器的实现（C语言）
首先，是次数的计算。次数的计算，我们可以由下式求得。

$$
\begin{aligned}
N = \frac{1}{2} \frac{\log_{10} (10^{\frac{A_s}{10}} - 1)}{\log_{10} \left ( \frac{\Omega_s}{\Omega_c} \right )}
\end{aligned}
$$

其对应的C代码为: 

```c++
N = Ceil(0.5*( log10 ( pow (10, Stopband_attenuation/10) - 1) / 
	 	            log10 (Stopband/Cotoff) ))
```

然后是极点的选择，这里由于涉及到复数的操作，我们就声明一个复数结构体就可以了。最重要的是，极点的计算含有自然指数函数，这点对于计算机来讲，不是太方便，所以，我们将其替换为三角函数，

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

这样的话，实部与虚部就还可以分开来计算。其代码实现为

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

计算出稳定的极点之后，就可以进行传递函数的计算了。传递的函数的计算，就像下式一样

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

这个时候，其传递函数为

$$
\begin{aligned}
H_a(s) = \frac{1}{(s-(a_1+ka_2))(s-(b_1+kb_2))}
\end{aligned}
$$

将其乘开，其大致的关系就像下图所示一样。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Complex_Multiple.jpeg" width="450"></div>

计算的关系一目了然，这样的话，实现就简单多了。高阶的情况下也一样，重复这种计算就可以了。其代码为

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

到此，我们就可以得到一个模拟滤波器巴特沃斯低通滤波器了。

&nbsp;
## 双1次z变换

&nbsp;
### 2.1双1次z变换的原理
我们为了将模拟滤波器转换为数字滤波器的，可以用的方法很多。这里着重说说双1次z变换。我们希望通过双1次z变换，建立一个s平面到z平面的映射关系，将模拟滤波器转换为数字滤波器。

和之前的例子一样，我们假设有如下模拟滤波器的传递函数。

$$
\begin{aligned}
H(s) = \frac{b}{s+a}
\end{aligned}
$$

将其做拉普拉斯逆变换，可得到其时间域内的连续微分方程式，

$$
\begin{aligned}
\frac{dy(t)}{dt} + ay(t) = bx(t)
\end{aligned}
$$

其中，$x(t)$表示输入，$y(t)$表示输出。然后我们需要将其离散化，假设其采样周期是$T$，用差分方程去近似的替代微分方程，可以得到下面结果:

$$
\begin{aligned}
\frac{1}{T} \left[ y(n) - y(n-1)\right] + \frac{a}{2} \left[ y(n) + y(n-1)\right] = \frac{b}{2} \left[ x(n) + x(n-1)\right] 
\end{aligned}
$$

然后使用$z$变换，再将其化简。可得到如下结果

$$
\begin{aligned}
H(z) = \frac{Y(z)}{X(z)} = \frac{b}{\frac{2}{T} \frac{1 - z^{(-1)}}{1 + z^{(-1)}} + a} = H(s) \Big|_{s=f(z)}
\end{aligned}
$$

从而，我们可以得到了$s$平面到$z$平面的映射关系，即

$$
\begin{aligned}
s = f(z) = \frac{2}{T} \frac{1 - z^{(-1)}}{1 + z^{(-1)}}
\end{aligned}
$$

由于所有的高阶系统都可以视为一阶系统的并联，所以，这个映射关系在高阶系统中，也是成立的。

然后，将关系式 $z=e^{j\omega}$ 和 $s = \delta+j\Omega$ 带入上式，可得

$$
\begin{aligned}
s &= \frac{2}{T} \frac{1 - e^{-j\omega}}{1 + e^{-j\omega}} \\
  &= \frac{2}{T} \frac{1 - \cos \omega - j\sin\omega}{1 + \cos \omega + j\sin \omega} \\
  &= \frac{2}{T} \frac{1 - (\cos^{2}\omega + \sin^{2}\omega) - j2\sin\omega}{2+2\cos\omega} \\
  &= 0 + j\frac{2}{T} \tan\frac{\omega}{2} = \delta + j\Omega
\end{aligned}
$$

到这里，我们可以就可以得到Ω与ω的对应关系了。

$$
\begin{aligned}
\Omega = \frac{2}{T} \tan\frac{\omega}{2}
\end{aligned}
$$

这里的$\Omega$与$\omega$的对应关系很重要。我们最终的目的设计的是数字滤波器，所以，设计时候给的参数必定是数字滤波器的指标。而我们通过间接设计设计IIR滤波器时候，首先是要设计模拟滤波器，再通过变换，得到数字滤波器。那么，我们首先需要做的，就是将数字滤波器的指标，转换为模拟滤波器的指标，基于这个指标去设计模拟滤波器。另外，这里的采样时间T的取值很随意，为了方便计算，一般取1s就可以。

&nbsp;
### 2.2双1次z变换的实现（C语言）
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
	             double *as,double *bs,
	             double *az,double *bz)
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
## 间接设计实现的IIR滤波器的性能


&nbsp;
### 设计指标
- 截止频率 $\omega_c=\frac{\pi}{2}[rad]$
- 阻带起始频率 $\omega_s=\frac{3}{4}\pi[rad]$
- 阻带衰减 $A_s=30[dB]$

&nbsp;
### 程序执行结果
使用上述程序，gcc编译通过，执行结果如下。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Design-IIR-Filter-1-1.jpeg" width="450"></div>


其频率响应如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Design-IIR-Filter-1/Design-IIR-Filter-1-2.jpeg" width="450"></div>
