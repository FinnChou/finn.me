---
layout: post
title:  "灰度变换—负片，对数变换，伽马变换，灰度拉伸，灰度切割，位图切割"
date:   2013-10-14 16:54:36 +0800
# categories: jekyll update
---

## 前言
灰度变换，属于一个非常重要的概念。这里主要参考《Digital Image Processing》 Rafael C. Gonzalez / Richard E. Woods 的第三章。书中所有的实验与数学式都采用了8-bit 图像的灰度范围，也就是0到255这样一个范围，这是本书不合理的一个地方。首先，这样做并不泛用，图片不一定是8-bit的。其次，在做某些变换的时候，可能会导致溢出。比如，伽马变化，假设伽马值为2，那么灰度为255的像素点，其变换之后值为65025，这里就溢出了。当然，要是使用Matlab计算，肯定会处理的非常好，直接使用mat2gray函数就能将其压缩回0到255。但是要是其他嵌入式平台处理的时候，直接套用不方便不说，直接按照8-bit的图来理解很不直观。因此，我将数学式做了改变，让其输入为0到1的浮点数，其输出也是0到1的浮点数，这样方便理解。

本文所使用的图片，均来源于《Digital Image Processing》的主页：
- [Digital Image Processing homepage]

## 图像负片 (Image Negativates)
有地方翻译为图像反转，这个翻译不是很恰当。这里应该理解为负片变换，负片变换如下所示。
  
$$
\begin{aligned}
{res_{(x,y)} = 1.0 - r_{(x,y)}, \hspace{3mm} src_{(x,y)} \in [0,1]}
\end{aligned}
$$

负片变换，主要用于观察过黑的图片，负片变换之后，方便观察。很简单的变换。
![ImageNegativates](/assets/Image-Negativates.jpeg)
<!-- <div align=center><img src="../assets/Image-Negativates.jpeg" width="200"></div> -->

## 对数变换 (Log Transformations)
对数变换主要用于将图像的低灰度值部分扩展，将其高灰度值部分压缩，以达到强调图像低灰度部分的目的。变换方法由下式给出。
$$
\begin{aligned}
{res_{(x,y)} = c * log_{v+1}(1.0 + v * src_{(x,y)}) , \hspace{3mm} r_{(x,y)} \in [0,1]}
\end{aligned}
$$

这里的对数变换，底数为$v + 1$，实际计算的时候，需要用换底公式。其输入满足$src_{(x,y)} \in [0, 1]$，其输出亦满足$res_{(x,y)} \in [0, 1]$。对于不同的底数，其对应的变换曲线如下图所示。
![LogTransformations](/assets/Log-Transformations.jpeg)

底数越大，对低灰度部分的强调就越强，对高灰度部分的压缩也就越强。相反的，如果想强调高灰度部分，则用反对数函数就可以了。看下面的实验就可以很直观的理解，下图是某图像的二维傅里叶变换图像，其为了使其灰度部分较为明显，一般都会使用灰度变换处理一下。
![LogTransformationsFFT](/assets/Log-Transformations-FFT.jpeg)

实现对数变换的Matlab代码如下：

{% highlight matlab %}
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
{% endhighlight %}

## 伽马变换 (Power-Law (Gamma) Transformations )
伽马变换主要用于图像的校正，将漂白的图片或者是过黑的图片，进行修正。伽马变换也常常用于显示屏的校正，这是一个非常常用的变换。其变化所用数学式如下所示，
$$
\begin{aligned}
{res_{(x,y)} = c * src_{(x,y)}^{\gamma} , \hspace{3mm} r_{(x,y)} \in [0,1]}
\end{aligned}
$$
其输入满足$src_{(x,y)} \in [0, 1]$，其输出亦满足$res_{(x,y)} \in [0, 1]$。 对于不同的伽马值，其对应的变换曲线如下图所示。

![GammaTransformations](/assets/Gamma-Transformations.jpeg)

和对数变换一样，伽马变换可以强调图像的某个部分。根据下面两个实验，可以看出伽马变换的作用。
#### 实验1 :
![GammaTransformationsTest1](/assets/Gamma-Transformations-t1.jpeg)
{% highlight matlab %}
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

subplot(1,2,2);
imshow(g2,[0 1]);
xlabel('b).Gamma Transformations \gamma = 0.4');
{% endhighlight %}

#### 实验2 :
![GammaTransformationsTest2](/assets/Gamma-Transformations-t2.jpeg)


[Digital Image Processing homepage]: https://www.imageprocessingplace.com/



