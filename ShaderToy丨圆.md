---
title: ShaderToy丨圆
date: 2019-11-15 12:17:12
categories:
    - 图形学
tags: 
    - 图形学
    - shader
mathjax: true
---

### github地址
[ShaderToy](https://github.com/CrowFea/ShaderToy)

### 关于圆
造型函数！这个名称莫名的让我想起妖精的尾巴里的那个冰之造型魔法。造型函数是非常基本而且重要的，相当于搭房子的砖头。这篇文章主要探索一下仅仅是画圆可以画出什么样的效果。

注意区分一下**圆**和**球**的区别，虽然在不加光照的情况下两个看起来一样，但实际上是不同的，我们这里讨论的是二维的圆。

### 基本的圆
先画一个基本的圆。很容易理解

```c++
float sdCircle( vec2 pos , float r )
{
	return length(pos)-r;
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
   	vec2 p = (2.0 * fragCoord - iResolution.xy) / min(iResolution.x,iResolution.y);
    
    vec3 col = vec3(0.0);
    float t = sdCircle(p,0.5);
    if(t<0.0001)	col = vec3(1.0);
    
    fragColor = vec4(col,1.0);
}
```
画出来如图
![img](https://s2.ax1x.com/2019/11/15/MaKwZQ.png)

很丑，而且边缘锯齿感也比较强烈。实际上，对于实现一个基本的圆形，我们稍加改动就可以呈现出一个观感更好的图形。我们把边缘稍微变得圆滑一点，这里过渡函数理应首先想到smoothstep，他的实际效果是非常棒的。值得注意的是，smoothstep的第二个参数，他的增量越小，过渡量越小，整体的边缘锯齿就越重；但如果过渡量过大，就会产生晕染的效果。因此这个参数要实际情况实际考虑。

```c++
float sdCircle( vec2 pos , float r )
{
    float d = length(pos);
	return smoothstep(d,d+0.01,r);
}
```
产生的圆就十分的平滑。我们可以让这个图形更好看一点，更改一下前后景的颜色就可以轻松做到。在另一个shader中我看到了使用mix函数进行着色的写法，看起来十分不错[Simple Circle](https://www.shadertoy.com/view/XsjGDt)。最终可以看到以下的图形：
![img](https://s2.ax1x.com/2019/11/15/Ma8NMq.png)

在这里额外推荐一个配色网站[Hello Color](https://jxnblk.github.io/hello-color/?c=dbcabf)

