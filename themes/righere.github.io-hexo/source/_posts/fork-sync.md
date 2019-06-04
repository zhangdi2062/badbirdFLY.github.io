---
title: 使用代码的方式同步Fork的仓库
date: 2017-02-15 17:50:23
tags: git
---
在访问开源社区的时候，有自己想关注的项目的时候，Fork到自己账户仓库是个很不错的方式

# 添加要fork的远程版本仓库
```
git remote add upstream `要fork的远程仓库地址`
```
<!--more-->

# fetch到要fork的远程版本仓库
此时会获取到远程版本仓库到本地
```
git fetch upstream
```

# checkout 到本地的版本仓库
```
git checkout master
```

# 将fetch的远程版本仓库和本地版本仓库合并
```
git merge upstream/master
```

# 将合并的本地仓库同步到自己的远程版本仓库
```
git push origin master
```