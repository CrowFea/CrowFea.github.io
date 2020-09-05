---
title: ShaderToy丨随机-DataPath的实现
date: 2019-11-13 10:26:12
categories:
    - 图形学
tags: 
    - shader
mathjax: true
---

### 效果图

![img](https://media.giphy.com/media/jmx3p9w9hMfC9zvXG2/giphy.gif)

这次实现的还是关于随机的一个例子，实现一个雪花的效果。
<!--more-->
### 思路
对于这种生成很多点的shader，后面还有一个做了还没写到的也类似，都是先生成一个网格来放这些点
```c++
vec2 grid = vec2(100.0,50.0);
p *= grid;
```
这个网格决定了由多少个可能出现的点，我们要将每一个像素坐标映射进去。
```c++
vec2 ipos = floor(p);  // integer
vec2 fpos = fract(p);  // fraction
```
这个做法我在其他的shader中也都有看到，算是一种常用的方法。整数部分的ipos可以用来进行随机化的处理，fpos可以在随机化处理完成后进行一定程度的偏移，来达到一种更随机的效果。
```c++
vec2 vel = vec2(iTime*2.0*max(grid.x,grid.y)); // time
vel *= vec2(-1.0,0.0) * random(1.0+ipos.y); // direction
```
接下来要定义一下雪花块运动的速度，包含速率和方向。我们把基础速率定义成一个和时间有关的数.但是如果只有iTime就显得太慢，因此我们把它扩大一点。接下来就是确定vel的方向，-1向右，+1向左，最后我们希望不同的行会按照不同的速率运动，因此这里对ipos.y做一个随机。值得注意的是，因为random函数中使用的是sin函数，如果是最低的一行，输入就是0。这个就相当于没有随机，因此要给所有的y加上一个正数保证足够大。
![gif](https://media.giphy.com/media/ZG0YwSFS35sR2jSh1A/giphy.gif)

```c++
vec2 offset = vec2(0.1,0.0);

vec3 col = vec3(0.0);
col.r = pattern(p+offset,vel,0.5);
col.g = pattern(p,vel,0.5);
col.b = pattern(p-offset,vel,0.5);
```
![img](https://s2.ax1x.com/2019/11/13/M8IUpt.png)

接下来就是正式的画图，首先定义一个offset量，我们仔细看这个图，发现其实在一些边缘是有不同的色块的，这个就是通过偏移量实现。对颜色我们做这样的处理：传入三个参数，一个是当前坐标+/-offset，一个是当前的速率，一个是标准量。pattern函数中对坐标+速率进行一个随机处理，如果处理后的数大于标准量，就返回1；否则返回0。这里的标准量建议都一样，这样才会出现各种各样的颜色，如果某一个颜色不同，比如变成0.4，0.5，0.5就会都变成红色。

接下来我们仔细看一下pattern函数
```c++
float pattern(vec2 st, vec2 v, float t) {
    vec2 p = floor(st+v);
    return step(t, random(100.0+p*0.000001)+random(p.x)*0.5 );
}
```
首先是进行了一个floor操作，这在生成随机的时候其实相当有必要。因为往往我们放进随机函数中的数据是坐标，这个数据是十分小并且每一个都有差别。因此当我们把这个数直接放进random时就会产生十分随机的随机数。虽然这在数学上可能是好的，但是在shader中，我们要实现的不是真正的随机，而是冲突率小的hash。因此如果数据太过随机，最后生成出来的数就会十分恐怖，比如下面这样
![gif](https://media.giphy.com/media/ZG0YwSFS35sR2jSh1A/giphy.gif)

因此要将一部分的随机数变成一个数，这样生成的图像才会连续。接下来对p进行随机化处理。这里random函数可以参考之前的一篇博客，最后生成的数为0-1的随机数。0和1的结果会随机的生成。

![gif](https://media.giphy.com/media/ZG0YwSFS35sR2jSh1A/giphy.gif)