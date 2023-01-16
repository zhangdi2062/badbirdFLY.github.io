---
title: CSS查漏补缺
date: 2019-12-01 11:26:36
tags:
- CSS
---

## 颜色继承

CSS3关键字：currentColor  // 当前文字的颜色
例如border会默认取得当前元素的color的颜色值或者继承父级color的色值

## 兄弟选择器～ +

注意两者都表示选择元素之后的兄弟节点

`~`表示元素后面所有的兄弟节点，多个

`+`表示元素后一个相邻的兄弟节点，一个

目前css还没有可选择前一个兄弟节点的方法，可以通过父元素选择，或js的previousElementSibling