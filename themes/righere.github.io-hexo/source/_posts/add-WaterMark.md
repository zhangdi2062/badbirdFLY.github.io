---
title: 使用七牛作为图床加上自己的水印
date: 2017-02-15 13:58:00
tags: Blog
---
有时候自己的写博客的图片，一方面不想被别人直接盗过去，另一方面又想宣传一下自己，达到和网友们交流的目的，所以一般都会自己加上水印，这个水印可以是自己的社交账号或者博客地址，推荐使用了七牛做为图床，然后想把自己的图片加上水印，这时候使用图床提供的图片处理接口就很容易进行图片处理了
<!--more-->
七牛提供的图片接口还是很多的，
![图床功能](http://of6x0sb2r.bkt.clouddn.com/qiniuPhotoProcess.png-WaterMark)

我们以加水印功能为例，

- 点击图片样式

![图片样式](http://of6x0sb2r.bkt.clouddn.com/picturestyle.png-WaterMark)

- 新建样式

![新建样式](http://of6x0sb2r.bkt.clouddn.com/newstyle.png-WaterMark)

- 设置水印参数

![设置图片样式](http://of6x0sb2r.bkt.clouddn.com/setStyle.png-WaterMark)

- 设置分隔符

![设置分隔符](http://of6x0sb2r.bkt.clouddn.com/separator.png-WaterMark)

## 下面我们就可以愉块的使用水印功能了：
例如，

我新建了一个图片样式名称为`WaterMark`,

样式的分隔符为中划线`-`，
    
现在我获取的图片的原始URL为`http://of6x0sb2r.bkt.clouddn.com/separator.png`

那么加水印的图片的URL就写成`http://of6x0sb2r.bkt.clouddn.com/separator.png-WaterMark`

图片就会自动加上我们新建的图片水印样式。