---
title: ShaderToy丨圆
date: 2019-11-19 14:25:12
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

### 颜色变换
再来实现一个稍微复杂一点的shader，这次我们在里面加入颜色和大小的变换。效果图如下：

![img](https://media.giphy.com/media/SXxufjP5dWzcn8Pv8F/giphy.gif)

可以看到是这样颜色渐变的波纹形状，波纹还是比较有意思的，很多更好看的shader都使用到了波纹，之后或许会发一篇关于波纹的文章（咕咕）

#### 实现
这个实际上要注意的就是大小的扩展，也就是波纹的效果。但是在此之前，我们首先把颜色的部分搞定
```c++
vec2 uv = fragCoord.xy / iResolution.xy;		
vec3 col = vec3(uv,0.5+0.5*sin(iTime));
```
常规操作，就是对颜色进行一个时间和空间上的渐变。接下来我们认真看一下大小的扩散，这个地方还是需要理解的。先上代码：
```c++
vec2 center = vec2(0.5,0.5);
float speed = 0.035;
float invAr = iResolution.y / iResolution.x;

float x = (center.x-uv.x);
float y = (center.y-uv.y) *invAr;	
float r = -sqrt(x*x + y*y); //uncoment this line to symmetric ripples
//float r = -(x*x + y*y);

float z = 1.0 + 0.5*sin((r+iTime*speed)/0.01);

fragColor = vec4(col*z,1.0);
```
首先既然是圆，计算的方法一定是平方和为半径平方。用自定义的圆心位置减掉当前坐标的位置，注意y坐标要进行一个映射，否则出来的是椭圆，最后求出当前坐标所在圆的半径。接下来一步是相当重要的，即是求出当前这个半径圆的颜色参数。这个可以当作轮子使用，基本的方程是这样
```c++
float z = a + b*sin((r+iTime*speed)/c);
```
a参数表示光环的大小与宽度，同时因为最后整个参数会添加到col中，所以该参数也会提高亮度。b参数用于对光的diffuse进行调整，同时该参数越大，颜色r和g的占比越大。c参数代表着分界的大小，参数越小，分出来的圈越多。最后我们将z参数添加到col中。
```c++
fragColor = vec4(col*z,1.0);
```

关于圆的话一开始本来以为能写很多，写着写着发现其实很多方法都是噪声、随机、或者是颜色的方法。造型函数的部分不是非常多。先到此为止，之后会整理一篇所有的造型函数的文章用作参考。
