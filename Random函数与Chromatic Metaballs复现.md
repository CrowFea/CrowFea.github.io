---
title: Random函数与Chromatic Metaballs复现
date: 2019-11-08 17:29:12
categories:
    - 图形学
tags: 
    - 图形学
mathjax: true
---

### shdaertoy链接
[Chromatic Metaballs](https://www.shadertoy.com/view/MdjczK)

### 效果图
先上效果图
![img](https://s2.ax1x.com/2019/11/09/MeMDXQ.md.gif)


最近在读[the book of shaders](https://thebookofshaders.com/),进度还是比较慢的。书里有很多可以学习的地方，打算慢慢的把书里提到了例子都复现一下。不过现在还在参考shaderToy上面的代码进行学习。

这个shader是在看过random这一篇章之后编写的。下面大概讲一下思路和过程。

### 思路
首先这个图的运动并没有时间上规律的变化，因此应当有单独的random函数来提供数据。这里需要注意的是，图中的团子并不是一整块，而是多个小的小团子凑在一起。这样做的好处是，在一些“起泡”的动画里会十分好处理——直接让这一块团子出去就好了。

```c++
 for(int i=0;i<MAXSTEP;++i)
 ```

 MAXSTEP决定了产生多少个小团子。在之后的方法中我们决定了产生团子的位置，和i的大小也有关系，所有的团子会从中心产生像边缘扩散。

 ```c++
vec2 pos = random2(vec2(float(i),0.2)) * 0.2;
float angle = mix(0.0, TPI, random(float(i)));
float radius = mix(0.01, 0.2, random(float(i)));
float freq = 1.0;
if(random(float(i)) > 0.5)	freq = -1.0;
vec2 offset = radius * vec2(sin(freq*t+angle), cos(freq*t + angle));
```
pos表示当前团子的位置，这里在random函数生成坐标之后乘0.2的原因在于让团子尽可能的靠近画面中心。这个参数越大，团子越分散。接下来angle和radius表示了团子的角度和半径。接下来有一个有意思的处理，我们想达到一种随机的扩散效果，比如水中的微生物这样的效果，offset的目的在于让这个团子随机的动起来，可以想到必须用iTime，但如果所有的团子都按照同样的方程进行运动，实际上整体上就都是一个方向的运动，就不是很随机。因此我们强行设定一部分的团子按照另一种方式旋转，也即是在时间t前面加上参数freq，一部分为正，一部分为负。

```c++
m_dist.r += 1.0 / distance( p + vec2(0.00,0.00) , pos + offset);
m_dist.g += 1.0 / distance( p + vec2(0.01,0.01) , pos + offset);
m_dist.b += 1.0 / distance( p + vec2(0.05,0.00) , pos + offset);
```

在着色这一部分，我们想要达到的效果是，在团子的边缘有红绿蓝的散射感。因此我们计算团子中心到偏移位置的距离，距离越大，某一部分的值越大。因此在p中心的位置加上对不同颜色的偏离，这决定了不同颜色的中心位置。

同时distance的好处是，在团子融合和分离的过程中，上面方程的数据变化是很平滑的，因此可以实现非常好的实现融合和分离的效果。

```c++
col += smoothstep(0.49,0.50,m_dist * 0.01);
```
最后smoothstep的上下限也应该注意，这里有一个锐化的问题，如果上下限差距过大，边缘会十分模糊，差距越小分界线越明显。同时上下线取值的大小决定了色域的大小，这里我们希望图像大小大概是正中一半的感觉，0.5就比较合适。

### Random的设计
random函数的设计直接影响到整个shader的观感。我们接下来说一下random函数的设计。

```c++
float random(float val)
{
	return fract(sin(val * 12.9898) * 43756.5453123 );
}

float random(vec2 val)
{
	return fract(sin(dot(val.xy,vec2(12.9898,78.233))) * 43756.5453123 );
}

vec2 random2(float val)
{
	vec2 p = vec2(dot(vec2(val),vec2(127.1,311.7)),dot(vec2(val),vec2(269.5,183.3)) );
    return 2.0 * fract(sin(p) * 43758.5453123) - 1.0 ;
}

vec2 random2(vec2 val)
{
	vec2 p = vec2(dot(val,vec2(127.1,311.7)),dot(val,vec2(269.5,183.3)) );
    return 2.0 * fract(sin(p) * 43758.531545) - 1.0 ;
}
```
其实这几个函数可以直接作为轮子用，在设计的时候可以看到一些思路。生成float型的随机数都是使用fract去截取，如果输入时float，就直接带入三角函数；如果是vec，进行一次点乘转换成float带入三角函数。

有趣的一点是，这个三角函数我在不同的shader里都见过
```c++
return fract(sin(val * 12.9898) * 43756.5453123 );
```
不由得思考这两个参数是不是有什么神奇的意义，于是发现了这一篇回答[What's the origin of this GLSL rand() one-liner?](https://stackoverflow.com/questions/12964279/whats-the-origin-of-this-glsl-rand-one-liner)。其中一个回答是这么说的，在*On generating random numbers, with help of y= [(a+x)sin(bx)] mod 1", W.J.J. Rey, 22nd European Meeting of Statisticians and the 7th Vilnius Conference on Probability Theory and Mathematical Statistics, August 1998*这篇文章里第一次提到了这个方程，虽然没有给出为什么，但是唯一一个被推荐的一个数是$\frac{\sqrt{5}-1}{2}\pi$。所以大致猜想的是后来有人看见了这篇paper，首先写出了上面的公式，因为表现良好而被一直使用（用轮子真舒服）。

但是无论起源如何，43756.5453123的目的都在于实现一个简单的hash。大的数据长度保证了即使x的动量很小，在结果上的变动也十分巨大。牢记**random的精髓就在于冲突率小的hash**。