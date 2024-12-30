---
layout: post
title:  "灰度变换—反转，对数变换，伽马变换，灰度拉伸，灰度切割，位图切割"
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
{src_{(x,y)} = 1.0 - r_{(x,y)}, \hspace{3mm} r_{(x,y)} \in [0,1]}
\end{aligned}
$$

负片变换，主要用于观察过黑的图片，负片变换之后，方便观察。很简单的变换。
![ImageNegativates](/assets/Image-Negativates.jpeg)
<!-- <div align=center><img src="../assets/Image-Negativates.jpeg" width="200"></div> -->


[Digital Image Processing homepage]: https://www.imageprocessingplace.com/



