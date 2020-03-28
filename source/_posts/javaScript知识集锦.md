---
title: javaScript知识集锦
date: 2019-05-23 22:06:47
tags: 
- 前端
- js
categories:
- JavaScript
---

小生不才，正处在初级学习阶段，如有不正确之处请多指教。

---  
<!--more-->

## 防抖

## 数组

- 

## 事件冒泡 / 捕获
 
事件冒泡： dom结构上（非视觉上）嵌套关系的元素，存在事件冒泡功能，即同一事件，子元素冒泡向父元素（自底向上）

事件捕获：dom结构上（非视觉上）嵌套关系的元素，存在事件冒泡功能，即同一事件，父元素捕获至子元素（事件源元素）（自顶向下），IE上没有事件捕获

触发顺序，先捕获后冒泡

单个元素本身事件按顺序执行

focus/blur/select/change/submit/reset 没有事件冒泡

例如
```html
<div class="wrapper">
    <div class="content">
        <div class="box"></div>
    </div>
</div>
<!-- 上面三个元素都有click事件，那么点击box时，事件执行顺序是子元素到父元素，为冒泡 -->
<!-- 上面三个元素都有click事件，那么点击box时，事件执行顺序是父元素到子元素，为捕获 -->
<!-- 设置方式为addEventListener(type, fn, false) false为冒泡，true为捕获-->
```

## 事件委托

利用事件冒泡和事件源对象进行处理

优点： 

性能 不需要循环所有子元素绑定事件

灵活 当有新的子元素时不需要重新绑定事件

用途：

10000个li，点击每一个li，打印li的内容，
正常写法：for循环，给每个li添加click事件
事件委托：
```js
ul.onclick = function(e) {
    let event = e || window.event;
    let target = event.target || window.srcTarget
    let text = target.innerText
    console.log(text)
  }
```