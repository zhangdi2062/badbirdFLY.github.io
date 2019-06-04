---
title: D3DXCreateTexture And LockedRect
date: 2017-01-18 11:07:02
tags: DirectX9
---
最近使用Directx对图像进行显示，出现图像乱码的现象，研究发现创建Texture的时候指定的图片宽度和锁定的纹理表面的宽度(LockedRect.Pitch)不一致，导致图片纹理是乱码。下面通过两种方式创建纹理分析LockedRect和最终纹理宽高之间的关系，一是通过图片的数据创建纹理，二是直接通过已知图片的像素点数据来创建数据。
<!--more-->

<center>
![乱码图片](http://of6x0sb2r.bkt.clouddn.com/wrongTexture.png-WaterMark "乱码图片")
![正确图片](http://of6x0sb2r.bkt.clouddn.com/rightTexture.png-WaterMark "正确图片")
</center>
# LockedRect数据结构

```
typedef struct _D3DLOCKED_RECT
{
    INT                 Pitch;
    void*               pBits;
} D3DLOCKED_RECT;
```
`Pitch`表示纹理表面一行所包含的byte，也就是一行纹理的长度，在使用的过程当中我们会发现这个长度必须是位宽的整数倍，也就保证和显存位宽对齐，这个和显卡有关系，常见的有64bit、128bit和256bit等；
`pBits`表示一个指针，指向我们锁定表面纹理的数据；

如果不考虑图片的宽和LockedRect.Pitch之间的关系，会很容易出现纹理最终显示出来的图片乱码

# 通过RGB数据生成纹理
首先通过一个例子，已知我们有图像的RGB数据，存储在imgData数组里面，我们要将RGB数据生成图片，该如何做？

> 解题流程：创建纹理 ---> 锁定纹理表面 ---> 对表面进行填充 ---> 解锁表面 ---> 生成纹理

```
//新建一个纹理
if (FAILED(D3DXCreateTexture(pd3dDevice, imgWidth, imgHeight, 1, D3DUSAGE_DYNAMIC, D3DFMT_R8G8B8, D3DPOOL_DEFAULT, &m_pTex1)))
{
    printf("D3DXCreateTexture failed\n");
    return FALSE;
}

D3DLOCKED_RECT LockedRect;

if (m_pTex1->LockRect(0, &LockedRect, NULL, 0) != D3D_OK)
{
    printf("LockedRect failed\n");
    return FALSE;	
}

BYTE *pSource = (BYTE*)LockedRect.pBits;

for (int i = 0; i < imgHeight; i++)
{
	for (int j = 0; j < imgWidth; j++)
	{
		int iDest = i * imgWidth * 4 + j * 4;
		int iSrc = i * imgWidth * 3 + j * 3;
		for (int k = 0; k < 3; k++)
		{
			pSource[x + k] = imgData[x + k];  //依次将内存中的RGB数据写入
		}
		pSource[x + 3] = 0xff;  //alpha通道的数据，如果没有可以取0-255之间的值，也可以不需要这一句
	}
}

m_pTex1->UnlockRect(0);

D3DXSaveTextureToFile("D:\\1.bmp", D3DXIFF_BMP, m_pTex1, NULL); //保存图片到D盘
```
**其实以上代码是有错误的**，
- 1、以二维图片的存储的思维，好像是没有错的，但是实际上计算机内存里面比不是二维储存的，而是一维存储方式；
- 2、将数据从内存拷贝到纹理当中又会不一样，在显存中纹理会有一个固定的位宽，这个位宽由电脑硬件决定，一般有128bit，64bit等;

所以我们要将上面for循环里面的代码进行修改：
```
int iDest = i * LockedRect.Pitch + j * 4;
int iSrc = i * imgWidth * 3 + j * 3;
for (int k = 0; k < 3; k++)
{
    pSource[x + k] = imgData[x + k];
}
    pSource[x + 3] = 0xff;
```

这里要注意几点，
- **注意一**： 在创建纹理的时候，我们明明使用的纹理格式是R8G8B8，但是最后生成纹理的时候，每个像素点仍然占用4个字节，所以我们如果有alpha通道的数值可以填充进去，因此代码可以再进行修改如下；
```
int iDest = i * LockedRect.Pitch + j * 4;
int iSrc = i * imgWidth * 4 + j * 4;
for (int k = 0; k < 4; k++)
{
    pSource[x + k] = imgData[x + k];
}
```
- **注意二**： 不能错误地把创建纹理表面的宽度和图片的宽度划等号，他们并不相等,他们之间的关系**使用公式** `LockedRect.Pitch = 128*ceil(imgWidth*4/128)`。
以我电脑显卡为例，创建表面的宽度是128的倍数，即`LockedRect.Pitch`是128的倍数，
那么问题来了，假如我现在的图片宽度是2160，按正常的4通道图片来算应该是`2160 * 4 = 8640`,但是我们锁定的纹理表面的宽度`LockedRect.Pitch = 8704 = 128 * ceil(2160*4/128)`，
这样就多出了64字节的空间，怎么办？其实不用管，不用进行任何操作。

# 使用已知图片的RGB数据创建纹理
下面是通过图片创建纹理，这个例子更简单,只要保证每次拷贝一行数据到纹理对应的一行就行了，
```
//新建一个纹理
if (FAILED(D3DXCreateTexture(pd3dDevice, imgWidth, imgHeight, 1, D3DUSAGE_DYNAMIC, D3DFMT_R8G8B8, D3DPOOL_DEFAULT, &m_pTex1)))
{
    printf("D3DXCreateTexture failed\n");
    return FALSE;
}

D3DLOCKED_RECT LockedRect;

if (m_pTex1->LockRect(0, &LockedRect, NULL, 0) != D3D_OK)
{
    printf("LockedRect failed\n");
    return FALSE;	
}

pSource = (BYTE*)LockedRect.pBits;
for (m = 0; m < imgHeight; m++)
{
	iDest = m*LockedRect.Pitch;
	iScr = m*imgWidthStep;     //imgWidthStep = imgWidth * 4, 4通道图片一行的宽度
	memcpy(&pSource[iDest], &(pSourceImg[iScr]), imgWidthstep);
}

m_pTexture[iDx]->UnlockRect(0);
```