---
layout: post
title: "Histogtam-Processing : 直方图处理"
date: 2013-10-20 15:47:36 +0800
# categories: jekyll update
---

&nbsp;
### 直方图均衡 (Histogram Equalization)
图像的直方图，表示了其灰度分布的特性。对于数字图像来说，假设灰度值$k$出现了$n_k$次，那么其概率密度函数如下所示。

$$
\begin{aligned}
{P_{s} = \frac{n_k}{M * N}}
\end{aligned}
$$

在这里，$P_{s}$代表了像素的灰度值为$k$的概率。 其中，$M$与$N$为图像的尺寸。对于一幅动态范围较窄的图像，其归一化灰度直方图如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogtam-Show.jpeg" width="600"></div>

对于动态范围较窄的图像，应用灰度拉伸可以有效改善其动态范围并增强图像对比度。然而，灰度拉伸本质上是一种模糊的处理方法，它并没有明确的目的。灰度拉伸仅依赖某个函数进行灰度变换，从而扩大图像在灰度直方图中的分布范围。由于灰度拉伸对结果没有严格的要求，因此根据不同的变换函数，可以产生无数种结果。

与灰度拉伸不同，直方图均衡的目的则更加明确。它使用特定的函数来实现原图像灰度分布的平均化。为了达到这一目标，我们需要引入累积分布函数（Probability Density Function）的概念，即：

$$
\begin{aligned}
{r = T_{s} = \int_{0}^{r} P_{s}(w) dw, \hspace{3mm} s \in [0,1]}
\end{aligned}
$$

这里表述的平均化，从上式中，可得知直方图均衡的目的是将一幅图的累积分布的曲线（下图左），变为下图右的分布形式。

<table>
    <tr>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogtam-Original.jpeg" width="600">
            </div> 
        </td>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogtam-PDF-target.jpeg" width="600">
            </div> 
        </td>
    </tr>
</table> 

从右图中可以看出，为了保证直方图均衡后的累积分布曲线是一条倾角为45°的直线，即，变换后的函数的概率密度函数应与输入值无关，且保持一个常量。也就是，输入$s$与输出$r$有如下关系，

$$
\begin{aligned}
{P_{r}(r) = P_{s}(s)\left|\frac{ds}{dr}\right|, \hspace{3mm} s_{(x,y)} \in [0,1]}
\end{aligned}
$$

将 $T_{s}$ 带入可得到，

$$
\begin{aligned}
\frac{dr}{ds} = \frac{dT_s}{ds} = \frac{d}{ds}\left[ \int_{0}^{r} P_{s}(w) dw\right] = P_s(s)
\end{aligned}
$$

显然，这里呈现了互为倒数关系，即

$$
\begin{aligned}
P_{r}(r) =  P_{s}(s)\left|\frac{ds}{dr}\right| = 1
\end{aligned}
$$

到此，已经可以看出，输出的概率密度函数，与$r$的取值无关，很明显这个分布是绝对平均的。事实上，上面的式子均是在连续的情况下的。在数字图像的领域，我们必须将其离散化，才能使用。离散的情况下，其累计分布函数如下所示，

$$
\begin{aligned}
{r = T_{s} = \sum_{j=0}^{k} P_r(r_k) = \frac{ \sum_{j=0}^{k} n_j }{M*N} , \hspace{3mm} k \in [0, L-1]}
\end{aligned}
$$

### 实验
直方图均衡的实现，有如下 Matlab 代码：


{% highlight matlab %}
close all;
clear all;

%% -------------Histogram Equalization-----------------
close all;
clear all;

f = imread('washed_out_pollen_image.tif');
f = mat2gray(f,[0 255]);

[M,N] = size(f);
g = zeros(M,N);

r = imhist(f)/(M*N);  %(500*500 is size of image)
s = zeros(256,1);

for k = 1:1:256
    for j = 1:1:k
        s(k,1) = s(k,1) + r(j);
    end
end

for x = 1:1:M        %(500*500 is size of image)
    for y = 1:1:N
        g(x,y) = s(uint8(f(x,y)*255)+1,1);
    end
end 


figure();
subplot(2,2,1);
imshow(f);
xlabel('a).Original Image');

subplot(2,2,2);
h = imhist(f)/(M*N);
bar(0:1/255:1,h);
axis([0 1 0 0.1]),grid;
%axis square;
xlabel('b).The Histogram of a');
ylabel('Number of pixels');

subplot(2,2,3);
imshow(g);
xlabel('c).Histogram Equalization');

subplot(2,2,4);
h = imhist(g)/(M*N);
bar(0:1/255:1,h);
axis([0 1 0 0.1]),grid;
%axis square;
xlabel('d).The Histogram of c');
ylabel('Number of pixels');

figure();
x=0:1/255:1; 
plot(x,s);
axis([0,1,0,1]),grid;
axis square;
xlabel('intensity level of r');
ylabel('P_{r}(r)');

r = imhist(g)/(M*N);
s = zeros(256,1);
for k = 1:1:256
    for j = 1:1:k
        s(k,1) = s(k,1) + r(j);
    end
end
figure();
x=0:1/255:1; 
plot(x,s);
axis([0,1,0,1]),grid;
axis square;
xlabel('intensity level of s');
ylabel('P_{s}(s)');
{% endhighlight %}

其直方图均衡后的图像与灰度直方图如下所示。

<div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogram-Equalization-Res.jpeg" width="600"></div>

结果显示，经过直方图均衡处理后的直方图相比于原图，确实扩大了动态范围。然而，根据公式，直方图均衡算法应当生成一个“均衡”的、与$k$值无关的“白化”直方图。然而，从实际的直方图来看，这一条件显然没有得到满足。为了解释这一现象，我们将原图像的累积分布函数与经过直方图均衡后的累积分布函数进行对比，如下所示。

<table>
    <tr>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogtam-Original.jpeg" width="600">
            </div> 
        </td>
        <td> 
            <div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogram-Equalization-PDF.jpeg" width="600">
            </div> 
        </td>
    </tr>
</table> 

经过直方图均衡处理后，我们可以观察到累积分布曲线虽然呈现出一定的阶梯状，但总体上接近于一条倾角为45°的直线。这是因为在证明过程中，我们是基于连续情况进行思考的。然而，在实际的数字图像处理中，算法是在离散情况下执行的，这可能导致一定的误差。直方图均衡的主要目标是使处理后图像的累积分布接近于一条倾角为45°的直线，这也是它与其他灰度调整算法之间的最大区别。

&nbsp;
### 直方图匹配 (Histogram Matching)
直方图均衡是一种有效的图像处理技术，它能够生成一个灰度直方图分布均匀的图像。这意味着直方图均衡的结果是唯一且可重复的。当我们需要增强图像的对比度时，直方图均衡通常能够提供令人满意的效果。然而，如果对直方图均衡的结果不尽如人意，我们可能需要对其算法进行进一步的调整和优化。

例如，以下这张图像（来源于《Digital Image Processing》Rafael C. Gonzalez / Richard E. Woods）展示了使用直方图均衡处理后的结果。

<div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogram-Equalization-Res2.jpeg" width="600"></div>

如上图所示，直方图均衡的结果并未达到理想状态。与原图相比，尽管一些细节得以显现，但仍然存在改进的余地。通过适当的调整，我们可以实现更为理想的增强效果。

在讨论直方图均衡时，我们可以对图像 $s$ 进行均衡处理，以获得其概率分布函数。

$$
\begin{aligned}
{r = T_{s} = \sum_{j=0}^{k} P_r(r_k) = \frac{ \sum_{j=0}^{k} n_j }{M*N} , \hspace{3mm} k \in [0, L-1]}
\end{aligned}
$$

直方图匹配，是将图像对齐到了直方图的分布保持一个常量的“白化”分布上。如果我们有一个期待的概率分布是非“白化”的分布函数 $G(Z_q)$， 那么对于直方图匹配这个算法而言，其目的就是对执行完算法的图像 $s$ 其分布和 $G(Z_q)$ 一致。也就是

$$
\begin{aligned}
G(Z_q) = \sum_{i=0}^{q}(P_z)(Z_i)
\end{aligned}
$$

其中，$z$代表了期望的分布。我们假设如下关系成立。

$$
\begin{aligned}
G(Z_q) = S_k
\end{aligned}
$$

也就是，我们所期待的直方图分布的概率分布函数与原图像的直方图概率分布函数相等。那么，我们可以利用$G()$的反函数，我们也就可以得到我们所期待的直方图。

$$
\begin{aligned}
S_k = G^{-1}(S_k)
\end{aligned}
$$

为了使描述更加简洁明了，我们可以进行如下调整：

一. 对原始图像进行直方图均衡处理。

二. 假设期望的分布与直方图均衡的结果相符，利用反函数将其恢复为我们所期待的分布。

以下是实现的直方图均衡结果展示。

<div align=center><img src="{{ site.baseurl }}/assets/Histogtam-Processing/Histogram-Matching.jpeg" width="600"></div>

根据直方图匹配的结果，我们可以观察到，所得结果与我们期望的分布非常接近。这一结果也符合我们的初衷，使得原图的直方图略微向左移动。与直方均衡相比，所获得的结果显著更为优越。

{% highlight matlab %}
%% -------------Histogram Matching-----------------
close all;
clear all;

f = imread('mars_moon_phobos.tif');
f = mat2gray(f,[0 255]);

[M,N] = size(f);
g = zeros(M,N);

r = imhist(f)/(M*N); 
s = zeros(256,1);

for k = 1:1:256
    for j = 1:1:k
        s(k,1) = s(k,1) + r(j);
    end
end
 
for x = 1:1:M        
    for y = 1:1:N
        g(x,y) = s(uint8(f(x,y)*255)+1,1);
    end
end 

figure();
subplot(2,2,1);
imshow(f);
xlabel('a).Original Image');

subplot(2,2,2);
h = imhist(f)/(M*N);
bar(0:1/255:1,h);
axis([0 1 0 0.1]),grid;
xlabel('b).The Histogram of a');

subplot(2,2,3);
imshow(g);
xlabel('c).Histogram Equalization');

subplot(2,2,4);
h = imhist(g)/(M*N);
bar(0:1/255:1,h);
axis([0 1 0 0.1]),grid;
xlabel('d).The Histogram of c');

%---------------pz-----------------------
r = r';
p = [0:r(1,1)/15:r(1,1) 17*r(1,2:241)];
%p = [0:r(1,1)/15:r(1,1) r(1,2:241)];
%p = [0:55:255 , 255:-25:24 , 24:-0.14:0 , 0:1:15 , 15:-0.285:0];
p = (p/sum(p));
p = p';
r = r';

G = zeros(256,1);
for k = 1:1:256
    for j = 1:1:k
        G(k,1) = G(k,1) + p(j);
    end
end


G_Negatives = zeros(256,1);
Flag = 0;
for k = 1:1:256
    for j = 1:1:256
            if(uint8(G(j,1)*255) == k) 
                 G_Negatives(k,1) = (j-1)/255;
                 Flag = 1;
            end
    end
    
    if(Flag == 0)
        if(k == 1) G_Negatives(k,1) = 0;
        else G_Negatives(k,1) = G_Negatives(k-1,1); end   
    end
    Flag = 0;

end

z = zeros(M,N);
for x = 1:1:M       
    for y = 1:1:N
        z(x,y) =  G_Negatives(uint8(g(x,y)*255)+1,1);
    end
end 

figure();
subplot(2,2,1);
imshow(z);
xlabel('a).Histogram Matching');

subplot(2,2,2);
h = imhist(z)/(M*N);
bar(0:1/255:1,h);
axis([0 1 0 .1]),grid;
xlabel('b).The Histogram of a');

subplot(2,2,3);
x=0:1/255:1; 
plot(x,p);
axis([0,1,0,.1]),grid;
xlabel('c).Specifieds Histogram (Normalized)');
 
subplot(2,2,4);
x=0:1/255:1; 
plot(x,G,x,G_Negatives);
axis([0,1,0,1]),grid;
axis square;
xlabel('Input intensity level');
ylabel('Onput intensity level');
{% endhighlight %}






