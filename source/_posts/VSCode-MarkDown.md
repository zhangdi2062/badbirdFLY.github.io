---
title: VSCode MarkDown
date: 2019-05-24 22:58:08
tags: 
- markdown目录
---

vscode中实现markdown自动生成目录

---

<!--more-->

## VSCode安装markdown扩展生成目录

### 1.安装扩展

- 搜索Markdown TOC / 命令行  npm install markdown-toc

### 2.可能出现的问题

- 目录出现排序不规范，如下状况，解决方式如下
![img](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3cswepjxxj31iu09y43i.jpg)
![solve](https://cdn.sinaimg.cn.52ecy.cn/large/005BYqpgly1g3ct1d4cjoj31i00u0jzn.jpg)

---

正则表达式写一丢丢

[1-9]\.  表示查找 “数字.”的情况，如“1.”

\### $0  表示替换上述为“### 1.”