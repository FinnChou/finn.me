---
layout: post
title: "Intensity Transformations : 灰度变换"
date: 2013-10-14 16:54:36 +0800
tags: 数字图像处理
# categories: jekyll update
---

本文主要参考了《Digital Image Processing》一书的第三章，主要介绍了一些列的灰度变换（Intensity Transformations）的相关内容。灰度变换的算法，一般是对单通道的灰度图像进行处理。而对于彩色图像，无论是何种格式（RGB，RGBA，CMYK），都可以单独的将某一个通道单独取出来，也可以进行灰度变换算法的处理。所以，以灰度图像的逐像素处理的算法为基础进行描述，也可以比较方便的运用于彩色图像。

另外在图像数据处理的时候，一般都需要将图像的像素值缩放到一定的范围内，常用的缩放范围是$[0, 1]$ 或者 $[-1, 1]$。 这个过程通常被称为图像归一化(Image Normalization)。

当前，常用的数字图像大多数依旧是8bits的数据，其像素范围为$[0, 255]$。数字图像数据目前比较常用的压缩格式JPG，基本就是8bits图像。当然，也存在一些16bits的图像数据，比如TIFF格式，PNG格式等，都支持存储16bits的图像数据。还有一些Camera RAW格式图像，可能存储了12bits，14bits或16bits的图像数据。而这些Raww数据并非通用的编码格式，实际处理的时候需要特殊的渲染引擎。

如果直接使用8bits或者16bits的图像数据进行计算，那么计算结果会超出范围的时候，就需要频繁进行处理。因此，我对数学公式进行了调整，使其输入和输出均归一化到$[0, 1]$的范围内。

&nbsp;
### 图像负片 (Image Negativates)  
<hr style="border: 2px solid #ccc; margin: 20px 0;">
有地方翻译为图像反转，这个翻译不是很恰当。这里应该理解为负片变换，负片变换如下所示。
  
$$
\begin{aligned}
{res_{(x,y)} = 1.0 - r_{(x,y)}, \hspace{3mm} src_{(x,y)} \in [0,1]}
\end{aligned}
$$

负片变换，主要用于观察过黑的图片，负片变换之后，方便观察。很简单的变换。
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Image-Negativates.jpeg" width="600"></div>


&nbsp;
### 对数变换 (Log Transformations)
<hr style="border: 2px solid #ccc; margin: 20px 0;">
对数变换主要用于将图像的低灰度值部分扩展，将其高灰度值部分压缩，以达到强调图像低灰度部分的目的。变换方法由下式给出。

$$
\begin{aligned}
{res_{(x,y)} = c * log_{v+1}(1.0 + v * src_{(x,y)}) , \hspace{3mm} src_{(x,y)}\in [0,1]}
\end{aligned}
$$

这里的对数变换，底数为$v + 1$，实际计算的时候，需要用换底公式。其输入满足$src_{(x,y)} \in [0, 1]$，其输出亦满足$res_{(x,y)} \in [0, 1]$。对于不同的底数，其对应的变换曲线如下图所示。
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Log-Transformations.jpeg" width="300"></div>

底数越大，对低灰度部分的强调就越强，对高灰度部分的压缩也就越强。相反的，如果想强调高灰度部分，则用反对数函数就可以了。看下面的实验就可以很直观的理解，下图是某图像的二维傅里叶变换图像，其为了使其灰度部分较为明显，一般都会使用灰度变换处理一下。
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Log-Transformations-FFT.jpeg" width="600"></div>

实现对数变换的Matlab代码如下：

```matlab
close all;
clear all;

%% -------------Log Transformations-----------------
f = imread('DFT_no_log.tif');
f = mat2gray(f,[0 255]);

v = 10;
g_1 = log2(1 + v*f)/log2(v+1);

v = 30;
g_2 = log2(1 + v*f)/log2(v+1);

v = 200;
g_3 = log2(1 + v*f)/log2(v+1);

figure();
subplot(1,2,1);
imshow(f,[0 1]);
xlabel('a).Original Image');
subplot(1,2,2);
imshow(g_1,[0 1]);
xlabel('b).Log Transformations v=10');

figure();
subplot(1,2,1);
imshow(g_2,[0 1]);
xlabel('c).Log Transformations v=100');

subplot(1,2,2);
imshow(g_3,[0 1]);
xlabel('d).Log Transformations v=200');
```


&nbsp;
### 伽马变换 (Power-Law (Gamma) Transformations )
<hr style="border: 2px solid #ccc; margin: 20px 0;">
伽马变换主要用于图像的校正，将漂白的图片或者是过黑的图片，进行修正。伽马变换也常常用于显示屏的校正，这是一个非常常用的变换。其变化所用数学式如下所示，

$$
\begin{aligned}
{res_{(x,y)} = c * src_{(x,y)}^{\gamma} , \hspace{3mm} src_{(x,y)} \in [0,1]}
\end{aligned}
$$

其输入满足$src_{(x,y)} \in [0, 1]$，其输出亦满足$res_{(x,y)} \in [0, 1]$。 对于不同的伽马值，其对应的变换曲线如下图所示。
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Gamma-Transformations.jpeg" width="300"></div>


和对数变换一样，伽马变换可以强调图像的某个部分。根据下面两个实验，可以看出伽马变换的作用。

#### 实验1 :
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Gamma-Transformations-t1.jpeg" width="600"></div>

```matlab
close all;
clear all;

%% -------------Gamma Transformations-----------------
f = imread('fractured_spine.tif');
f = mat2gray(f,[0 255]);

C = 1;
Gamma = 0.4;
g2 = C*(f.^Gamma);

figure();
subplot(1,2,1);
imshow(f,[0 1]);
xlabel('a).Original Image');
∏
subplot(1,2,2);
imshow(g2,[0 1]);
xlabel('b).Gamma Transformations \gamma = 0.4');
```

#### 实验2 :
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Gamma-Transformations-t2.jpeg" width="600"></div>


&nbsp;
### 对比度拉伸 (Contrast Stretching)
<hr style="border: 2px solid #ccc; margin: 20px 0;">
对比度拉伸是一种用于增强图像特定部分的技术。与伽马变换和对数变换不同，对比度拉伸能够有效改善图像的动态范围。这种方法可以将原本低对比度的图像转换为高对比度图像。实现对比度拉伸的方法有很多，其中最简单的一种是线性拉伸。然而，本文将介绍一种稍微复杂一些的方法，如下所示。


$$
\begin{aligned}
{res_{(x,y)} = \frac{1}{1 + (m / r_{(x,y)})^{E}} , \hspace{3mm} src_{(x,y)} \in [0,1]}
\end{aligned}
$$

同样地，输入满足 $src_{(x,y)} \in [0, 1]$ 的情况下，输出也满足 $res_{(x,y)} \in [0, 1]$。这个公式非常熟悉，类似于巴特沃斯高通滤波器，其输入输出关系的形状也可以大致猜测。然而，这里出现了一个问题：当输入为0时，公式将变得无意义。因此，在实际执行算法时，我们会在分母上加上一个小数，从而将其转换为如下形式。

$$
\begin{aligned}
{res_{(x,y)} = \frac{1}{1 + (\frac{m}{src_{(x,y)} + eps})^{E}} , \hspace{3mm} src_{(x,y)} \in [0,1]}
\end{aligned}
$$

只是，其输入满足$src_{(x,y)} \in [0, 1]$的时候，其输出范围变为了 $res_{(x,y)} \in [\frac{1}{1 + (\frac{m}{eps})^{E}}, \frac{1}{1 + (\frac{m}{1 + eps})^{E}}]$，近似可以视为$res_{(x,y)} \in [0, 1]$。 为了精确起见，使用mat2gray函数将其扩展到精确的。调用格式如下。

```matlab
res = mat2gray(src, [1/(1+(m/eps)^E) 1/(1+(m/1+eps)^E)]);
```

输入输出问题已经解决，但仍然存在一个待处理的问题，即参数的确定。这里涉及两个参数：$m$（对于巴特沃斯高通滤波器而言，这是截止频率）和$E$（同样是针对巴特沃斯高通滤波器，这是滤波器的次数）。参数$m$可以调节变换曲线的重心，而$E$则影响曲线的斜率，如下图所示。

<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Contrast-Stretching.jpeg" width="600"></div>

$m$值的可取图像灰度分布的中央值，如下式所示，

$$
\begin{aligned}
m = \frac{1}{2}(Min(res_{(x,y)}) + Max(res_{(x,y)}))
\end{aligned}
$$

决定$m$之后，接下来只剩下$E$的计算。提升对比度的目的在于扩展图像的动态范围，我们希望将原本灰度范围为$[Min(res_{(x,y)}), Max(res_{(x,y)})]$的图像转换到$[0, 1]$之间。为此，我们可以直接使用最大值和最小值带入公式，解出$E$。

然而，如之前所述，我们所使用的公式的输出范围无法达到$[0, 1]$。直接取$[0, 1]$的范围会导致$E$的值非常大，从而使得变换曲线的斜率过于陡峭，最终的灰度扩展效果并不理想。因此，我们采取一种折中的方法，将输出范围设定为$[0.05, 0.95]$。接下来，$E$的取值如下所示。

$$
\begin{aligned}
E_{1} &= log_{\frac{m}{Min(res_{(x,y)})}}(\frac{1.0}{0.05} - 1.0) \\
E_{2} &= log_{\frac{m}{Max(res_{(x,y)})}}(\frac{1.0}{0.95} - 1.0) \\
E &= ceil(min(E_{1}, E_{2})
\end{aligned}
$$

#### 实验:
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Contrast-Stretching-t1.jpeg" width="600"></div>

从直方图看，原图的灰度范围确实被拉伸了。用上面所说的方法，确定的灰度拉伸的输入输出曲线如下图所示。
<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Contrast-Stretching-t1-curve.jpeg" width="300"></div>

其Matlab代码如下：

```matlab
close all;
clear all;

%% -------------Contrast Stretching-----------------
f = imread('washed_out_pollen_image.tif');
%f = imread('einstein_orig.tif');
f = mat2gray(f,[0 255]);

[M,N] = size(f);
g = zeros(M,N);

Min_f = min(min(f));
Max_f = max(max(f));
m = (Min_f + Max_f)/2;

Out_put_min = 0.05;
Out_put_max = 0.95;

E_1 = log(1/Out_put_min - 1)/log(m/(Min_f+eps));
E_2 = log(1/Out_put_max - 1)/log(m/(Max_f+eps));
E = ceil(min(E_1,E_2)-1);

g = 1 ./(1 + (m ./ (f+ eps)).^E);
g = mat2gray(g,[1/(1+(m/eps)^E) 1/(1+(m/1+eps)^E)]);

figure();
subplot(2,2,1);
imshow(f,[0 1]);
xlabel('a).Original Image');

subplot(2,2,2);
r = imhist(f)/(M*N);
bar(0:1/255:1,r);
axis([0 1 0 max(r)]);
xlabel('b).The Histogram of a');
ylabel('Number of pixels');

subplot(2,2,3);
imshow(g,[0 1]);
xlabel('c).Results of Contrast stretching');

subplot(2,2,4);
s = imhist(g)/(M*N);
bar(0:1/255:1,s);
axis([0 1 0 max(s)]);
xlabel('b).The Histogram of a');
ylabel('Number of pixels');

in_put = 0:1/255:1;
Out_put1 = 1 ./(1 + (m ./ (double(in_put)+ eps)).^E);
Out_put1 = mat2gray(Out_put1,[1/(1+(m/eps)^E) 1/(1+(m/1+eps)^E)]);

figure();
plot(in_put,Out_put1);
axis([0,1,0,1]),grid;
axis square;
xlabel('Input intensity level');
ylabel('Onput intensity level');
```



&nbsp;
#### 灰度切割 (Intensity-level Slicing)
<hr style="border: 2px solid #ccc; margin: 20px 0;">
灰度切割也是一个很简单，但也很实用的变换。灰度切割，主要用于强调图像的某一部份，将这个部分赋为一个较高的灰度值，其变换对应关系如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Intensity-level-Slicing.jpeg" width="600"></div>

灰度切割有以上两种方法，一种是特定灰度值的部分赋值为一个较高的灰度值，其余部分为一个较低的灰度值。这样的方法，得到的结果是一个二值化图像。另外一种方法，则是仅仅强调部分赋值为一个较高的灰度值，其余的部分不变。

<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Intensity-level-Slicing-t1.jpeg" width="600"></div>


&nbsp;
#### 位图切割 (Bit-plance Slicing)
<hr style="border: 2px solid #ccc; margin: 20px 0;">
位图切割，就是按照图像的位，将图像分层处理。若图像的某个像素，其bit7为1，则在位面7这个像素值为1，反之则为0。

<div align=center><img src="{{ site.baseurl }}/assets/basic-Intensity-Transformations-Functions/Bit-plance-Slicing.jpeg" width="600"></div>

由位图切割的结果，图像的主要信息包含在了高4位。仅仅靠高4位，还原的图像更原图基本差不多。由此可见，位图切割主要用于图像压缩。
