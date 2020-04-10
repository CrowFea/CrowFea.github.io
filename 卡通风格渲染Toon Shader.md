---
title: 卡通风格渲染
date: 2019-12-5 18:25:12
categories:
    - 图形学
tags: 
    - 图形学
mathjax: true
---

最近尝试做了些卡通风格渲染的shader，还是比较有趣的，将其中一个过程和心得分享在下面。

### 效果图
![img](https://s2.ax1x.com/2019/12/05/Q8x9s0.png)

### 怎样卡通
卡通风格渲染实际上是有一定的规律。在实现上面这个shader的时候，使用的方法实际上还是之前用过的Phong-Blion光照模型，但是展现出来的效果就完全不同。卡通风格有几个明显的特点：
- 低色阶

    这点在日式的风格中十分明显，基本是大块大块的色域，并不像现实物体那样有着渐变的颜色，颜色之间的转折十分尖锐。
- 描边

    这点或多或少都会有，描边有着多种方法，其生成的边也不尽相同，这里可以看这篇[文章](https://blog.uwa4d.com/archives/usparkle_cartoonshading.html)
    ![img](https://s2.ax1x.com/2019/12/05/Q8YHJO.jpg)
    *无主之地3的描边非常明显*

- 高光聚焦

    这一点是我自己总结的（大概率不对），在卡通渲染中高光并不像现实那样，是一片由亮到暗，而是很明显的一条或者一个点。
    ![img](https://s2.ax1x.com/2019/12/05/Q8NQEt.jpg)
    *好想玩塞尔达*

    可以看到林克的头发上，很明显的一块高光，身上被照亮的地方也是，不存在一个渐变的情况。

我们在接下来看一下如何实际操作这两项。

### 降低色阶
既然是冯氏着色，就是有典型的三种光：环境、高光、物体光。我们以环境光为例。（在此例中我们使用的只有一个光源，使用多个光源会更复杂，比如塞尔达荒野之息使用了两个光源）
```c#
float NdotL = dot(_WorldSpaceLightPos0, normal);

float lightIntensity = smoothstep(0, 0.01, NdotL);// * shadow);	
float4 light = lightIntensity * _LightColor0;
```
点乘的目的是为了求环境光和顶点法线的cos，这个是冯氏光照的内容。接下来一行值得注意：我们为了让色阶降低，很容易想到的方法是，减少颜色的渐变。为此，我们使用smoothstep，限制cos的大小，实际上就是限制法线和光线的角度。当这个角度大于一个值时强行设定为1，我们故意将这个值设定的极小，（在这里我们设置的是0.01）。对于下限也一样。也就是说，在本例中，只有夹角的cos值在0-0.01间，他才有可能根据不同的角度，显现出“自己”反射的颜色。这样一来就有效的减少了渐变。

同时，使用smoothstep还有一个原因，就是当他的定义域在一定范围内的的时候，值域的变化很小。我们可以看一下smoothstep用的函数：

![img](https://s2.ax1x.com/2019/12/05/Q8aO4P.png)

可见在0-1内的小区间内。值域的变化很小。这样即使颜色落在了设定的上下限之间，他根据自己定义域的变化实际上也不大。我们可以将smoothstep变成clamp观察一下，就会发现使用clamp函数会有更强的真实感，这也是我们不希望产生的。

降低色阶还有一种方法是采用幂运算，将一个数不断的进行小数次幂的运算也能有效降低变化。
```c#
float specularIntensity = pow(NdotH * lightIntensity, _Glossiness * _Glossiness);
```
我们通过这种方法处理了高光，因为高光本身就需要一个pow运算。同样对于高光聚焦的问题，也是通过这样的方式解决。无论如何，应当发现，**降低色阶的主要手段就是减少任何因素对颜色的影响。一般来说是当前顶点和光线的角度，通过高阶运算的方式，将定义域的改变对值域的影响减小，这就是降低色阶的方法。**

### 描边
在本例中我们对描边讨论的并不是很深，实现的是一个边缘的高亮效果。

值得一提的是，确定边缘的方法有很多，我们在后续会讨论。本例中采用的方法是基于视角的，通过计算视线和顶点法线的夹角cos，如果为0，那么为相切，这个地方就是边缘。为了方便计算，往往会用1去减，获得一个正相关的数。
```c#
float rimDot = 1 - dot(viewDir, normal);
float rimIntensity = rimDot * pow(NdotL, _RimThreshold);
rimIntensity = smoothstep(_RimAmount - 0.01, _RimAmount + 0.01, rimIntensity);
float4 rim = rimIntensity * _RimColor;
```
边缘的高亮效果可以理解为只在边缘的高光，只不过视角的影响是通过rimDot体现的，而非在幂函数内部。同样的，通过smoothstep方法，我们可以设定边沿的大小，也可以将这个转折变得尖锐。

唯一的问题是，由于最后我们将颜色整合的方式是全部加起来，这样就没有办法画出黑色的描边（因为黑色是0），后续会想办法解决这个问题。

### 阴影
最后一点是阴影，一开始在做完后我们的物体没有办法投射阴影，在这个地方踩了一些坑花了很久。

阴影可以直接在最后调用Legacy Shader中的Pass来解决。
```c#
UsePass "Legacy Shaders/VertexLit/SHADOWCASTER"
```
这个Pass可以投射阴影，同时在前面的若干地方要添加几项宏定义：
```c#
// Files below include macros and functions to assist
// with lighting and shadows.
#include "Lighting.cginc"
#include "AutoLight.cginc"

//------in struct v2f
// Macro found in Autolight.cginc. Declares a vector4
// into the TEXCOORD2 semantic with varying precision 
// depending on platform target.
SHADOW_COORDS(2)

//------in vertex 
// Defined in Autolight.cginc. Assigns the above shadow coordinate
// by transforming the vertex from world space to shadow-map space.
TRANSFER_SHADOW(o)

//------in frag
// Samples the shadow map, returning a value in the 0...1 range,
// where 0 is in the shadow, and 1 is not.
float shadow = SHADOW_ATTENUATION(i);

```
注释已经编写在内，不再赘述。最后需要额外注意的一点是，有可能这些都齐活了也看不到阴影，这可能有两个情况：
- Inspector中没有开cast shadow
- 物体太大，摄像机太远

对于我而言就是第二个问题，将物体scale缩小，摄像机移近后就都可以看见了。