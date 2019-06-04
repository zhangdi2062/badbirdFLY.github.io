---
title: DirectShow播放视频流程
date: 2017-03-22 18:05:54
tags: DirectShow
---
# DirectShow框架简介
DirectShow框架是多媒体播放框架上一个非常经典的框架，现在已经十多年了，在Windows平台上依然无法替代，非常值得去学习研究。个人觉得从设计模式的角度上看，directshow框架的灵活性、复用性、可维护性、可拓展性这些方面做得非常不错，也是它经久不衰历久弥新的一个原因，现在的很多第三方的decoder和filter都基于directshow框架开发，可以很灵活的移植到directshow视频框架中，例如[视骏](http://www.strongene.com/cn/hevc/hevcProduct.jsp)开发的`HEVC/H.265`解码器，都可以直接挂载在directshow框架中进行视频解码。

---  
<!--more-->
# 图形化理解DirectShow  
推荐一款工具[GraphStudio](https://en.wikipedia.org/wiki/GraphStudio),了解DirectShow框架必备工具，软件截图如下：
![GraphStudio](http://of6x0sb2r.bkt.clouddn.com/graphstudio.png-WaterMark)  
我们点击`Graph`可以插入我们在电脑系统中注册的`Filter` `Render`，默认情况下，我们将播放的视频加到GraphStudio中，会自动生成`directshow`的整个播放流程,然后就可以播放视频了。一般的播放效果流程如下：  
![directshow框图](http://of6x0sb2r.bkt.clouddn.com/direcshow%E6%B5%81%E7%A8%8B%E5%9B%BE.png-WaterMark)

`GraphStudio`会自动采用系统默认的一套`Filter`和`Render`,如果安装了[K-Lite Codec Pack](https://www.codecguide.com/download_kl.htm)，就可以修改系统默认的这一套，如下图:  
![K-Lite](http://of6x0sb2r.bkt.clouddn.com/K_Lite_kit.png-WaterMark)  
我们想测试我们自己的`Filter`和`Render`，都可以自定义插入，下面就以DirectShow中植入视骏的`HEVC`解码器为例子，了解DirecShow的整个播放流程，如下图所示：  
![Hevc-VideoPlay](http://of6x0sb2r.bkt.clouddn.com/HEVCDecoder.png-WaterMark)  
# DirectShow播放HEVC视频
可以参考雷老师关于`DirectShow`的介绍，地址：http://blog.csdn.net/leixiaohua1020/article/details/42372419  
播放的流程如下：

![direcshow](http://of6x0sb2r.bkt.clouddn.com/directshowplay.png-WaterMark)

整个播放我们可以抽象出三个步骤：

- 注册：播放HEVC视频需要的`Strongene Mpeg-4 Demultiplexor`,`Lentoid HEVC Decoder`,`Video Renderer`
- 链接：相当于上面`GraphStudio`图中的链接箭头
- 播放：将视频导入播放链路中，开始播放

## 注册Filter和Render

### 解码器属性

首先获取到这些Filter的`Object name`、`CLSID`、`Filename`和`FilePath`
![FilterInfo](http://of6x0sb2r.bkt.clouddn.com/filterInfo.png-WaterMark)  
如果已经将filter注册进Windows的系统中,就只需要用到`Object Name`,为了避免重新注册导致冲突；
```
//Name:File Source (Async.) {E436EBB5-524F-11CE-9F53-0020AF0BA770}
/**
Windows自带File Source (Async.)，dll路径：C:\Windows\SysWOW64\quartz.dll
CLSID  = 
*/

//MP4 Splitter {61F47056-E400-43D3-AF1E-AB7DFFD4C4AD} Name: Strongene Mpeg-4 Demultiplexor
static const GUID CLSID_MP4Splitter = {0x61f47056,0xe400,0x43d3,{0xaf,0x1e,0xab,0x7d,0xff,0xd4,0xc4,0xad}};

//h265-HEVC Decoder {658C5E1C-58E1-43CA-9D10-13735D465576} Name:Lentoid HEVC Decoder
static const GUID CLSID_HEVCDecoder = {0x658C5E1C,0x58E1,0x43CA,{0x9D,0x10,0x13,0x73,0x5D,0x46,0x55,0x76}};

//Name:Video Mixing Renderer {B87BEB7B-8D29-423F-AE4D-6582C10175AC}
/**
Windows自带VideoRenderer，dll路径：C:\Windows\SysWOW64\quartz.dll
*/
```
---

### 声明DirectShow播放需要的类和变量

```
IGraphBuilder * mGraph;//创建一个Filter Graph Manager组件
IMediaControl *	mMediaControl;//提供控制过滤器图表中多媒体数据流的方法，包括运行、暂停和停止
IMediaEventEx *	mEvent;//继承自IMediaEvent接口，处理过滤器图表的事件
IBasicVideo *	mBasicVideo;//用于设置视频特性，如视频显示的目的区域和源区域
IBasicAudio *	mBasicAudio;//用于控制音频流的音量和平衡
IVideoWindow  *	mVideoWindow;//定义一个视频窗口的控制对象
IMediaSeeking *	mSeeking;//提供搜索数据流位置和设置播放速率的方法

IBaseFilter *m_pSourceFilter = NULL;    
IBaseFilter *m_pSplitter = NULL;
IBaseFilter *m_pSplitter = NULL; 
IBaseFilter *m_pVideoHEVCDecoder = NULL;
IBaseFilter *m_pVideoRender = NULL; 

DWORD mObjectTableEntry = 0;
```
---
### 注册Filter：

- 加载解码器并获取DllGetClassObject指针,loadFilter()
```
bool loadFilter(LPCSTR chAx)
{
	//加载Filter所在的dll文件
	m_hInst = ::LoadLibrary(chAx);
	if (NULL == m_hInst)
	{
		return false;
	} 

	//获取DllGetClassObject函数指针
	m_pDllGetClassObject = ( DLL_GET_CLASS_OBJECT )::GetProcAddress(m_hInst, "DllGetClassObject");
	if (NULL == m_pDllGetClassObject)
	{
		return false;
	}
	return true;
}
```
---

- 动态创建解码器对象,createFilter()
```
bool createFilter(GUID clId, IBaseFilter** pBaseFilter)
{
	if (NULL == m_pDllGetClassObject)
	{
		return false;
	}

	//获取类工厂接口
	IClassFactory *p_IClassFactory = NULL;
	if (FAILED( m_pDllGetClassObject(clId, IID_IClassFactory, (void **)&p_IClassFactory) ))
	{
		return false;
	}

	//创建与类厂相关联的COM对象（Filter），并获取其IBaseFilter接口
	p_IClassFactory->CreateInstance(NULL, IID_IBaseFilter, (void **)pBaseFilter);
	p_IClassFactory->Release();
	p_IClassFactory = NULL;

	if ((NULL == pBaseFilter) || (NULL == *pBaseFilter))
	{
		return false;
	}
	return true;	
}
```
---
- 申明一个方法函数InitRegister()，
```
bool InitRegister()
{
    mGraph=NULL;
    mMediaControl=NULL;
    mEvent=NULL;
    mBasicVideo=NULL;
	mBasicAudio=NULL;
	mVideoWindow=NULL;
	mSeeking=NULL;
	
    if(!mGraph)
    {
        //建立 filter graph manager
        HRESULT hr = CoCreateInstance(CLSID_FilterGraph, NULL, CLSCTX_INPROC_SERVER,IID_IGraphBuilder, (void **)&mGraph);
        if(mGraph != NULL)
        {
            LPCSTR  splitterDll, HEVCDecoderDll;
        	LPCWSTR splitterName, HEVCDecoderName;
        	GUID	splitterGuid, HEVCDecoderGuid;
        	
            //Source Filter
            hr = CoCreateInstance(CLSID_AsyncReader, NULL, CLSCTX_INPROC,IID_IBaseFilter, (void **)&m_pSourceFilter);
            if(hr == S_OK)
            {
                mGraph->AddFilter(m_pSourceFilter, L"File Source Filter");
            }
            else
            {
                cout << "load source filter failed" << endl;
                return false;
            }
            
            //Splitter 
            splitterDll = ".\\Codec\\mp4demux.dll";
            splitterName = L"Strongene Mpeg-4 Demultiplexor";
            splitterGuid = CLSID_SMp4Demultiplexor;
            
            //HEVCDecoder
            HEVCDecoderDll = ".\\Codec\\hevcdecfltr.dll";
            HEVCDecoderName = L"Lentoid HEVC Decoder";
            HEVCDecoderGuid = CLSID_HEVCDecoder;

             //Splitter
            if (loadFilter(splitterDll))
            {
            	if (createFilter(splitterGuid, &m_pSplitter))
            	{
            		mGraph->AddFilter(m_pSplitter, splitterName);
            	}
            	else
            	{
            	    cout << "add spliter failed" << endl;
            		return false;
            	}
            }
            else
            {
                cout << "load splitter faild" << endl;
            	return false;
            }
            
            //HEVC Decoder
            if (loadFilter(HEVCDecoderDll))
            {
            	if (createFilter(HEVCDecoderGuid, &m_pVideoHEVCDecoder))
            	{
            		mGraph->AddFilter(m_pVideoHEVCDecoder, HEVCDecoderName);
            	}
            	else
            	{
            	    cout << "add decoder failed" << endl
            		return false;
            	}
            }
            else
            {
                cout << "load decoder failed!" << endl;
            	return false;
            }
            
    		//Video Renderer
			hr = CoCreateInstance(CLSID_VideoMixingRenderer9, NULL, CLSCTX_INPROC, 
				IID_IBaseFilter, (void **)&m_pVideoRender);
			if(hr == S_OK)
			{
			    mGraph->AddFilter(m_pVideoRender, L"Video Mixing Renderer 9");
			}
			else
			{
			    cout << "load video renderer failed" << endl;
				return false;
			}
    	}
    }
}

```
## 链接Filter和Renderer

- 查找空闲的filter的接口
```
HRESULT GetUnconnectedPin( IBaseFilter *pFilter, PIN_DIRECTION PinDir, IPin **ppPin, int iNum )
{
	*ppPin = NULL;
	IEnumPins* pEnum = NULL;
	IPin* pPin = NULL;
	int nIndex = 0;
	
	HRESULT hr = pFilter->EnumPins(&pEnum);
	if (FAILED(hr))
    {
        return hr;
    }

	while(pEnum->Next(1, &pPin, NULL) == S_OK)
	{
		PIN_DIRECTION ThisPinDir;
		pPin->QueryDirection(&ThisPinDir);
		if (ThisPinDir == PinDir)  //如果out/in类型符合
		{
			nIndex ++;
			IPin *pTmp = NULL;
			hr = pPin->ConnectedTo(&pTmp);  //如果已经被连接
			if (SUCCEEDED(hr))  
			{
				pTmp->Release();
			}
			else  
			{
				if(nIndex == iNum)  //若端口号与传入端口号同
				{
					pEnum->Release();
					*ppPin = pPin;
					pPin->Release();
					return S_OK;
				}
			}
		}
		pPin->Release();
	}
	pEnum->Release();
	cout << "GetUnconnectedPin ----> Next is null" << endl;
	return false;
}
```
- 查找空闲的splitter的空闲指针接口

```
HRESULT GetUnconnectedMajortypePin( IBaseFilter *pFilter, PIN_DIRECTION PinDir, IPin **ppPin, GUID myMajortype)
{
	*ppPin = NULL;
	IEnumPins* pEnum = NULL;
	IPin* pPin = NULL;
	int nIndex = 0;
	IEnumMediaTypes *pType;  
	AM_MEDIA_TYPE  *pMType;  

	HRESULT hr = pFilter->EnumPins(&pEnum);
	if (FAILED(hr))
	{
		return hr;
	}

	while(pEnum->Next(1, &pPin, NULL) == S_OK)
	{
		PIN_DIRECTION ThisPinDir;
		pPin->QueryDirection(&ThisPinDir);
		if (ThisPinDir == PinDir)  //如果out/in类型符合
		{
			nIndex ++;
			IPin *pTmp = NULL;
			hr = pPin->ConnectedTo(&pTmp);
			if (SUCCEEDED(hr))    //如果已经被连接
			{
				pTmp->Release();
			}
			else  
			{
				pPin->EnumMediaTypes(&pType);
				pType->Next(1, &pMType, 0);
				if(myMajortype == pMType->majortype)  //类型匹配
				{
					pEnum->Release();
					pType->Release();
					*ppPin = pPin;
					pPin->Release();
					return S_OK;
				}

				pType->Release();
			}
		}
		pPin->Release();
	}
	pEnum->Release();
	cout << "GetUnconnectedMajortypePin failed" << endl;
	return false;
}
```
- DirectShow提供自动搜寻Filter，我们只需要将他们链接起来

```
bool LinkFilter()
{
	//定义pin对象
	IPin *ppinOut = NULL;
	IPin *ppinIn = NULL;
	IPin* Pin;
	//sourceOut -> Splitter
	if(m_pSourceFilter != NULL && m_pSplitter != NULL)
	{
		hr = GetUnconnectedPin(m_pSourceFilter, PINDIR_OUTPUT, &Pin, 1); //查找Filter空闲的pin
		if (SUCCEEDED(hr))
		{
			ppinOut = Pin;			
		}
		hr = GetUnconnectedPin(m_pSplitter, PINDIR_INPUT, &Pin, 1);
		if (SUCCEEDED(hr))
		{
			ppinIn = Pin;			
		}
		if (FAILED(mGraph->Connect(ppinOut, ppinIn)))
		{
		    cout << "link source filter and spliter failed!" <<endl;
			return false;
		}
	}
	
	//Spliter -> hevcdecoder//解码HEVC
	if (m_pSplitter != NULL && m_pVideoHEVCDecoder != NULL)
	{
		hr = GetUnconnectedMajortypePin(m_pSplitter, PINDIR_OUTPUT, &Pin, MEDIATYPE_Video);//查找splitter空闲的video outpin
		if (SUCCEEDED(hr))
		{
			ppinOut = Pin;
		}
		hr = GetUnconnectedPin(m_pVideoHEVCDecoder, PINDIR_INPUT, &Pin, 1);
		if (SUCCEEDED(hr))
		{
			ppinIn = Pin;
		}
		if (FAILED(mGraph->Connect(ppinOut, ppinIn)))
		{
			return false;
			cout << "link splitter and decoder failed!" << endl;
		}
	}
	//hevcdecoder -> videorender
	if (m_pVideoDecoder != NULL && m_pVideoRender != NULL)
	{
		hr = GetUnconnectedPin(m_pVideoHEVCDecoder, PINDIR_OUTPUT, &Pin, 1); //查找Filter空闲的pin
		if (SUCCEEDED(hr))
		{
			ppinOut = Pin;
		}
		hr = GetUnconnectedPin(m_pVideoRender, PINDIR_INPUT, &Pin, 1);
		if (SUCCEEDED(hr))
		{
			ppinIn = Pin;
		}
		if (FAILED(mGraph->Connect(ppinOut, ppinIn)))
		{
			return false;
			cout<<"link decoder and video renderer failed!"<<endl;
		}
	}
	return true;
}						
```
## 播放HEVC视频

```
bool RenderFile(const *char MediaUri)
{
	if (mGraph)
	{
		HRESULT hr;
		WCHAR    szFilePath[MAX_PATH];
		//该函数映射一个字符串到一个宽字符的字符串
		MultiByteToWideChar(CP_ACP, 0, MediaUri, -1, szFilePath, MAX_PATH);
		
		//source filter与视频文件关联
		IFileSourceFilter* pFSFilter = NULL; 
		
		//pFSFilter仅需要局部变量，用完释放
		m_pSourceFilter->QueryInterface(&pFSFilter);
		pFSFilter->Load(szFilePath, NULL);   //FilePath;
		pFSFilter->Release();
		pFSFilter = NULL;
		if(!LinkFilter())
		{
		    cout << "link filter failed" << endl;
		}
	}
	else
	{
	    cout << "render file failded" << endl;
	    return false;
	}
}
```