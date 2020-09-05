---
title: ShaderToy丨着色与时间-Virus的实现
date: 2019-11-15 14:09:12
categories:
    - 图形学
tags: 
    - shader
mathjax: true
---

链接[ShaderToy](https://github.com/CrowFea/ShaderToy)
### 效果图
之前一直做的是随机和噪声，今天做一点鲜艳的，搞一个着色和时间函数的作品。

![img](https://media.giphy.com/media/cLSYRNd51uXatonc4a/giphy.gif)

这次实现的是一个三维的效果，球体上的滚动效果，看起来就像病毒或者是充满魔力的灵魂之球（嗯
<!--more-->
### 思路

这个shader实际上是一个冯氏光照下的球体，球体的表面随着时间起伏。这个起伏的实现方式是球体表面加上一个大小不同增量。冯氏光照的模型不再赘述，直接看一下病毒的造型函数。
```c++
float map(vec3 pos)
{
	float d = length(pos)-1.5;
    float peak = sin(5.0*pos.x+iTime*2.0)*sin(5.0*pos.y+iTime*2.0)*sin(5.0*pos.z+iTime*2.0)*0.15;
    return d+peak;
}
```
peak即球体上的增量值，注意要实现这样的效果，必须在xyz三个方向上都进行计算，pos.xyz前面的参数代表着peak的峰值，iTime后面的参数代表着时间的变化速度。如果只计算某几个方向的值，也会有一些很有意思的效果。

![img](https://media.giphy.com/media/UQgUIlUbvbSx3ZZ3AA/giphy.gif)

*计算x方向*

![img](https://media.giphy.com/media/S5QeWkdHkjNYlqwKQW/giphy.gif)

*计算xy方向*

### 光照
这里要对冯氏模型进行一点小改造，我们不需要镜面反射，只要将漫反射的参数调整一下。
```c++
float flu(vec3 pos)
{
    vec3 lightPos = vec3(4.0*sin(iTime),2.0,4.0*cos(iTime));
    
	vec3 N = calcNormal(pos);
    vec3 L = normalize(lightPos-pos);
    float LN = dot(L,N);
   
    return (LN+1.0)/2.0;
}
```
LN后的参数是漫反射的亮度。最后进行着色。
```c++
vec3 pos = ro + t*rd;
vec3 nor = calcNormal(pos);
        
float k_d = flu(pos);
col = 0.5 + 0.5*sin(iTime+pos.xyx+vec3(0,2,4));
col = k_d * col;
```