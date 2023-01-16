---
title: svg画饼图
date: 2019-11-02 10:15:39
tags: 
- svg 
---

先来看图

单环图
![KqYDjH.png](https://s2.ax1x.com/2019/11/02/KqYDjH.png)

多环图
![KqY1c4.png](https://s2.ax1x.com/2019/11/02/KqY1c4.png)

通常遇到这种饼图都会用echarts实现，但是echarts不能实现两端圆角，最多通过border和shadow实现两端近似圆角的平角，如图
[![KqtE8O.md.png](https://s2.ax1x.com/2019/11/02/KqtE8O.md.png)](https://imgchr.com/i/KqtE8O)

那开始上代码，实现方式是用svg，主要用到三个属性 `stroke-dasharray`、`stroke-dashoffset`、`stroke-linecap`

属性的解释可以参考：
[SVG：理解stroke-dasharray和stroke-dashoffset属性](https://juejin.im/post/5c8db3175188257e252a49da)

单环图的实现请参考：
[使用svg实现一个半圆圆角进度条](https://blog.csdn.net/breavo_raw/article/details/96159252)
[使用 SVG，实现带动画效果的圆环进度条](https://juejin.im/entry/58f95efda0bb9f0065a8c263)


首先，我们用svg画出两个圆，一个用作背景，一个表示进度，将进度的圆利用`stroke-dasharray`、`stroke-dashoffset`两个属性做出进度条，再用`rotate`旋转角度，修改进度开始的角度

多环图的实现同样是上述原理，只是需要动态计算旋转的角度。

后面的升级版就是带有label标签的环状图了，大家有更好的做法要留言哦！
![KjwlKP.png](https://s2.ax1x.com/2019/11/03/KjwlKP.png)







