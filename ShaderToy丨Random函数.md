---
title: ShaderToy丨Random函数
date: 2019-11-10 17:29:12
categories:
    - 图形学
tags: 
    - shader
mathjax: true
---

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

值得看一下random函数的图像
![img](https://s2.ax1x.com/2019/11/13/MJCKbQ.md.png)
可以看出取值十分随机，并且值域在0-1。

有趣的一点是，这个三角函数我在不同的shader里都见过
```c++
return fract(sin(val * 12.9898) * 43756.5453123 );
```
不由得思考这两个参数是不是有什么神奇的意义，于是发现了这一篇回答[What's the origin of this GLSL rand() one-liner?](https://stackoverflow.com/questions/12964279/whats-the-origin-of-this-glsl-rand-one-liner)。其中一个回答是这么说的，在*On generating random numbers, with help of y= [(a+x)sin(bx)] mod 1", W.J.J. Rey, 22nd European Meeting of Statisticians and the 7th Vilnius Conference on Probability Theory and Mathematical Statistics, August 1998*这篇文章里第一次提到了这个方程，虽然没有给出为什么，但是唯一一个被推荐的一个数是$\frac{\sqrt{5}-1}{2}\pi$。所以大致猜想的是后来有人看见了这篇paper，首先写出了上面的公式，因为表现良好而被一直使用（用轮子真舒服）。

但是无论起源如何，43756.5453123的目的都在于实现一个简单的hash。大的数据长度保证了即使x的动量很小，在结果上的变动也十分巨大。牢记**random的精髓就在于冲突率小的hash**。