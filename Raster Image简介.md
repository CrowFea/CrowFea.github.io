---
title: Raster Image简介
date: 2019-07-18 20:26:12
categories:
    - 图形学
tags: 
    - 图形学
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---

### Raster Image
这里我们主要说一下LCD和LED。虽然在现实生活中听了很多遍，但其实还是非常陌生的两个概念。

目前的显示器，包括电视和数字电影放映机以及计算机显示器和放映机，几乎都是基于固定的像素阵列。它们可以分为**发射式显示器**和**透射式显示器**，前者使用直接发射可控制数量的光的像素，后者使用像素本身不发射光，而是改变允许通过它们的光的数量。透射显示器需要光源来照亮它们:在直视显示器中，这是阵列后面的背光;在投影仪中，它是一种发出光的灯，光经过阵列后投射到屏幕上。发射显示器是它自己的光源。

LCD（液晶显示器）是典型的投射式显示器。液晶是一种材料，它的分子结构使它能够旋转通过它的光的偏振，并且旋转的程度可以通过施加的电压来调节。一个LCD像素（如下图）的后面有一层偏振光膜，所以它被偏振光照射。

![image](https://s2.ax1x.com/2019/07/18/ZXJ4KA.png)

LED（发光二极管）显示器是一种发射型显示器 每个像素由一个或多个LED组成,LED是半导体器件(基于无机或有机半导体),其发光强度取决于通过它们的电流(见图3.1) 彩色显示器中的像素分为三个独立控制的子像素,一个红色,一个绿色,一个蓝色,每个像素都有自己的LED,使用不同的材料制成,所以它们发出不同颜色的光(图3.2)

![image](https://s2.ax1x.com/2019/07/18/ZXY9aV.png)

### Pixel Format
```
Here are some pixel formats with typical applications:

*   1-bit grayscale—text and other images where intermediate grays are not desired (high resolution required);
*   8-bit RGB fixed-range color (24 bits total per pixel)—web and email applications, consumer photographs;
*   8- or 10-bit fixed-range RGB (24–30 bits/pixel)—digital interfaces to computer displays;
*   12- to 14-bit fixed-range RGB (36–42 bits/pixel)—raw camera images for professional photography;
*   16-bit fixed-rangeRGB (48 bits/pixel)—professional photographyand printing; intermediate format for image processing offixed-range images;
*   16-bit fixed-range grayscale (16 bits/pixel)—radiology and medical imaging;
*   16-bit “half-precision”floating-pointRGB—HDR images; intermediate format for real-time rendering;
*   32-bit floating-point RGB—general-purpose intermediate format for software rendering and processing ofHDR images.
```
### Monitor Intensities and Gamma

这里讨论的是显示强度和伽马。

所有的现代显示器都接受像素值的数字输入，并将其转换为强度级别。真正的显示器在关闭时会有一些非零强度，因为屏幕会反射一些光。出于我们的目的，我们可以认为这是黑色的，而显示器完全是白色的。我们假设像素颜色的数值描述范围从0到1。黑色是0，白色是1，介于黑色和白色之间的灰色是0.5。注意这里half指的是来自像素的物理光量，而不是外观。人类对强度的感知是非线性的，不会成为当前讨论的一部分。

要在监视器上生成正确的图像，必须理解两个关键问题。

首先，**监视器对于输入是非线性的**。例如，如果将监视器0、0.5和1.0作为三个像素的输入，则显示的强度可能是0、0.25和1.0(关闭、四分之一完全打开和完全打开)。这种非线性的近似描述,显示器通常表现为一个γ(γ)值。这个值就是公式中的自由度

![img](https://s2.ax1x.com/2019/07/18/ZXURot.png)

其中a为0到1之间的输入像素值。例如，如果监视器的gamma值为2.0，而我们输入的值为a =0.5，则显示的强度将是最大可能强度的四分之一，因为0.5^2 =0.25。请注意α= 0映射到零强度和α= 1映射到最大强度不管γ的值。描述一个显示年代使用γ只是一个近似非线性;我们不需要大量的精度估计γ的设备。测量非线性的一个很好的直观方法是找出α的值，再通过上式进行一个对数运算。

![img](https://s2.ax1x.com/2019/07/18/ZXaeSO.png)

找到阿尔法可以通过这种方式：显示一个黑白的棋盘图，旁边显示一个灰色像素的输入,然后要求用户调整(例如滑块之类的),直到双方平均亮度相匹配。当你从远处看这幅图像时(如果你是近视眼，或者不戴眼镜)，当α在黑白之间产生一种强度时，图像的两边看起来差不多。这是因为模糊的棋盘混合了偶数个黑白像素，所以整体效果是介于黑白之间的统一颜色。

显示器的另一个重要特性是它们接受量子化的输入值。因此，虽然我们可以在浮点范围[0,1]中操作强度，但是监视器的详细输入是一个固定大小的整数。这个整数最常见的范围是0-255，它可以保存在8位存储中。这意味着a的可能值不是[0,1]中的任何数字，而是

![img](https://s2.ax1x.com/2019/07/18/ZXaw0s.md.png)