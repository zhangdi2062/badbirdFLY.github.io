---
title: 搭建hexo博客并简单的实现多终端同步
date: 2016-10-10 23:33:26
tags: hexo
---

主要是熟悉下hexo配合github创建自己的博客，首先按照hexo的官网进行配置，简单地创建自己的博客，然后配置_config.yml,其中可以配置自己的博客主题，部署到github.io中，因为本人用的windows和ubuntu的双系统，顺便解决下多终端下怎么同步写作的问题。

<!--more-->
## 安装Hexo

参照官网：[Hexo官网](https://hexo.io/zh-cn/docs/)

新建一个文件夹，这里就是我们本地的Hexo目录了，比如新建`Myblogs`文件夹,然后进入这个目录，
> cd Myblogs

然后用执行安装Hexo的命令:
> npm install hexo-cli -g

此时会出现一些WARN，不过不会影响hexo使用
> npm install hexo --save

初始化Hexo
> hexo init

安装hexo相关组件
> npm install

<!--more-->

## 开始使用hexo

先生成一个helloworld的demo网页
> hexo g

本地测试生成的网页
> hexo d

打开浏览器输入 http://localhost:4000/
可以看到生成的demo网页

新建自己的blog
> hexo n "MyBlog"

这样就新建了一个自己的blog，页面编辑` /source/_posts/ `目录下面会生成markdown文件

新建远程github仓库，仓库命名规则：`你的名字.github.io`

修改全局配置文件

配置主题：网上可以找到很多好看的主题，推荐next，很简洁，配置简单，[next项目地址](https://github.com/iissnan/hexo-theme-next)

部署到github，修改_config.yml文件中的Deployment，
```
    deploy:
        type: git
        repo: git@github.com:yourname/yourname.github.io.git
        branch: master
```
使用git的方式部署，需要执行
> npm install hexo-deployer-git --save

执行部署
> hexo d

打开自己的写的blog,打开网页
> https：//yourname.github.io

---
## 使用git进行版本控制，实现不同终端同步

如果在其他电脑上或者系统上编辑自己的博客也很简单：

- 先在远程仓库新建一个branch，比如hexo

<center>
![new branch](http://of6x0sb2r.bkt.clouddn.com/newHexoBranch.png "新建branch")
</center>

接着将hexo的分支设为默认，

<center>
![default branch](http://of6x0sb2r.bkt.clouddn.com/SetDefaultBranch.png "设置默认branch")
</center>

- 将我们的源文件上传到hexo分支

>注意这里有个巨大的坑！！！如果你用的是第三方的主题theme，是使用git clone下来的话，要把主题文件夹下面把.git文件夹删除掉，不然主题无法push到远程仓库，导致你发布的博客是一片空白

初始化本地仓库： `git init`

添加本地所有文件到仓库：`git add -A`

添加commit：`git commit -m "blog源文件"`

添加本地仓库分支hexo：`git branch hexo`

添加远程仓库：`git remote add origin git@github.com:yourname/yourname.github.io.git`

将本地仓库的源文件分支hexo强制推送到远程仓库hexo分支：`git push origin hexo -f`

<center>
![hexo branch](http://of6x0sb2r.bkt.clouddn.com/hexo_srcfiles.png "上传完成")
</center>

上传完成之后，我们就拥有了两个远程的分支：master和hexo，其中master是部署成博客的分支；hexo是我们可以clone到其他电脑或其他系统的hexo源文件的分支，而且我们已经将它设置成默认仓库；

- 在其他电脑设备上执行clone远程仓库的hexo分支clone到本地，

> git clone -b hexo git@github.com:yourname/yourname.github.io.git

进入本地仓库执行hexo安装：`npm install`

- 编辑本地blog之后，编辑发布博客

依次执行`git add .` ,`git commit -m "改了啥"`， `git push origin hexo`,同步本地仓库到远程

部署发布博客: `hexo g`，`hexo d`这样就生成静态网页部署到了github中

## 遇到的一些问题

关于第三方主题next的问题以及解决办法

症状： 今天使用github将next的主题文件进行了更新，使用`hexo g`生成静态网页，然后`hexo s`本地调试成功，也没什么问题，当我`hexo d`发布到远程仓库的之后，在线浏览的时候发现一片空白，这时候使用其他主题都没问题，网上找到了答案。

解决办法: 将主题文件夹下面的`source/vendors`，全部改成`source/lib`
    
    1. 先把主题文件夹下面`source/`下的`vendors`改成`lib`；
    2. 将主题配置文件中`_internal: vendors`改成`_internal: lib`
    3. 将主题文件夹下面其他文件中的`source/vendors`改成`source/lib`
<center>
![where're libs](http://of6x0sb2r.bkt.clouddn.com/hexo_next.png)
</center>