---
title: Ray Tracing简介
date: 2019-07-26 22:26:45
categories:
    - 图形学
tags: 
    - 图形学
mp3: http://domain.com/awesome.mp3
cover: http://domain.com/awesome.jpg
---

### Rendering
图形学研究的一个基本问题是，如何将具有三维特性的物体以二维的image呈现出来。从某种层面上讲，建筑师、画家都或多或少的从事这方面的工作。在计算机中，这个过程叫**渲染**</p>
> Fundamentally, rendering is a process that takes as its input a set ofobjects and produces as its output an array of pixels

渲染有两种方式：

*   object-order rendering

    遍历每个object，计算object对于每个像素的贡献。
*   image-order rendering

    遍历每个像素，计算当前像素有哪些object的贡献。

    简单来说就是，两种方法都有一个for_each pixels的方法，object-order中这个循环在里面；image-order中这个循环在外面。
<!--more-->
两种渲染方式都可以得到结果，一般采用image-order，他在渲染时更容易出结果，也更灵活，耗时更久。

**Ray tracing is an image-order algorithm for making renderings of 3D scenes**

### Ray Tracer

一个基本的Ray tracer有三个部分:

*   ray generation，根据相机的几何形状计算出每个像素的观看射线的来源和方向;
*   ray intersection，找到与观察射线相交最近的物体;
*   shading，根据光线相交的结果计算像素的颜色

生成一个Ray Tracer的基本步骤是

```c++
for each pixel 
do 
    compute viewing ray 
    find first object hit by ray and its surface normal n 
    set pixel color to value computed from hit point, light, and n
```

#### Coordinates

首先要定义什么是光线。他可以解析的定义为以下的式子：

![img](https://s2.ax1x.com/2019/07/26/eueho8.png)

e代表发出光线的点，在这里指我们的观察者的眼睛（虽然光线最终进入眼睛，不过在这里我们认为眼睛发出ray）。s代表光线打到成像平面上的点。这里的值域所代表的就是光线传播方向上点的向量。当t是0时，这个点就是e，当t是1时，这个点就是s。

研究光线以及相应的成像我们一般在Camera Frame这个坐标系下完成。

![img](https://s2.ax1x.com/2019/07/26/eumwXn.png)

### Computint Viewing Rays

光线分两种情况讨论，正交试图和透视视图。

![img](https://s2.ax1x.com/2019/07/26/euuPVx.png)

对正交视图来说，光线的方向都是一样的，即坐标系的w方向。唯一不同的是光线的起点。正交视图我们可以认为，发出光线的平面和呈现的平面是一个平面，因为成像的结果不会因为这两个平面的距离变化而变化。

![img](https://s2.ax1x.com/2019/07/26/euuprR.png)

透视视图存在一个距离问题，即成像的平面和发出光线的平面中间有一个距离d。这个我们称之为焦距。在透视试图下，所有的光线生成点是一个，但是所有光线的方向不同。

![img](https://s2.ax1x.com/2019/07/26/euuSM9.png)

### Ray-Object Intersection

在这里书里面说的很花哨，但看下来实际上就是解析几何里的，直线和平面相交的解析求法，有两个特殊的情况需要考虑。

#### Ray-Triangle Intersection

当我们需要处理的平面是三角形时，有一个需要注意的判断条件,下面这个是我们需要处理的等式：

![img](https://s2.ax1x.com/2019/07/26/euMTrd.png)

为了保证相交的点在该三角形内，需要满足β&gt;0,γ&gt;0且β+γ&lt;1。在处理等式时可以化成矩阵，用克莱姆规则求解。

![img](https://s2.ax1x.com/2019/07/26/euM7qA.md.png)

#### Intersecting a Group of Objects

实际上，我们更可能碰见的情况就是一个场景中有多个物体。对于这种情况要实现如下的算法：

![img](https://s2.ax1x.com/2019/07/26/euQhoq.png)

### Shading

大多数的Shading Model，设计来捕捉光的反射过程，即表面被光源照亮和反射光线到相机。Shading就是如何给当前的物体加上阴影的过程。在这里有几个重要的向量。l向量表示光照方向的向量，v表示视点的向量，n是成像平面的法向量。

![img](https://s2.ax1x.com/2019/07/26/eu1Ybd.png)

接下来介绍几个简单的模型

![img](https://s2.ax1x.com/2019/07/26/eu3nsg.png)

#### Lambertian Shading

![img](https://s2.ax1x.com/2019/07/26/eu16bj.png)

其中kd是一个扩散系数，I是当前方向的光照强度。后面的n·l应当注意，在这里因为n，l都是单位向量，所以n·l=|n|×|l|×cos&lt;n,l&gt;=cos&lt;n,l&gt;。所以这里指的是n和l向量的cos值。

Lambertian Shading十分简单，但缺陷在于物体呈现的阴影与视角是没有关系的，因此阴影是不具有高光。就是我们看到的镜面效果。

#### Blinn-Phong Shading

![img](https://s2.ax1x.com/2019/07/26/eu1XPx.png)

BP的模型中加入了高光元素的修正。当人的视角与光线照射的反射角相等时，就会出现镜面反射的效果。这个时候应该非常亮。因此我们使用一个向量h来表示l和v向量相加方向的单位向量。这个向量和平面的法向量夹角越大，证明越不容易镜面反射，也越暗淡。同理，因为都是单位向量，在表示cos值的时候可以使用点乘来表示。

![img](https://s2.ax1x.com/2019/07/26/eu3uLQ.png)

#### Ambient Shading

![img](https://s2.ax1x.com/2019/07/26/eu3deJ.md.png)

当我们不考虑表面角度，只考虑每一个物体本身的颜色时，就需要引入环境光。因为就算角度再变，红色的还是红色，只不过可能是暗一点或者亮一点的红色。因此在上面加入了一个环境修正，作为常数。

以上三个模型结合在一起，基本上完成了一个简单但是有用的shading modle。