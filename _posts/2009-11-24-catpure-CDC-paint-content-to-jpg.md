---
layout: post
title: 捕获CScrollView的视窗CDC绘制内容于BMP文件或JPG文件中
tags: MFC
categories: 编程
---

最近做的一个东东，有个要求，需要将绘制在视窗中的内容保存成图片。而该视窗的类是继承于MFC中的CScrollView，即是滚动视图。滚动视图的文档内容往往都会比可见的视口要大许多。由于之前没做过这方面的玩意，所以随即百度、Google了一把。于是发现网上讲和较多的是屏幕截图或是只是捕获当前窗口可见区域的内容，滚动视图隐藏的部分要么截出来是黑的或是根本捕获不到。并没有找到符合自己要求的，但在网上查找这方面东东多了点，些许有了点想法。于是乎就立马code验证了，以下是验证过程。

由于是测试代码，就没有做太多的考虑，只是简单的验证了一下，测试代码很简单。首先准备测试工程。打开VS2008，新建立一个MDI工程，将View类的基类选择为CScrollView,工程名为TestScrollViewSavebmp。在doc类中添加一个成员变量和绘制方法函数。
~~~C++
#pragma once

class CTestScrollViewSavebmpDoc : public CDocument
{
protected: // create from serialization only
	CTestScrollViewSavebmpDoc();
	DECLARE_DYNCREATE(CTestScrollViewSavebmpDoc)

// Attributes
public:

// Operations
public:

// Overrides
public:
	virtual BOOL OnNewDocument();
	virtual void Serialize(CArchive& ar);

// Implementation
public:
	virtual ~CTestScrollViewSavebmpDoc();
#ifdef _DEBUG
	virtual void AssertValid() const;
	virtual void Dump(CDumpContext& dc) const;
#endif

protected:
	/**滚动视图的逻辑坐标轴最大范围*/
	CSize m_sztotalLog;

public:
	void SetTotalSize(CSize size)
	{
		m_sztotalLog = size;
	}

	CSize GetTotalSize();
	{
		return m_sztotalLog;
	}
        
        /**内容绘制函数*/
        void DrawDoc(CDC* pDC);

// Generated message map functions
protected:
	DECLARE_MESSAGE_MAP()

~~~
然后为初始化成员变量及简单实现内容绘制函数。
~~~C++
// TestScrollViewSavebmpDoc.cpp : implementation of the CTestScrollViewSavebmpDoc class
//

#include "stdafx.h"
#include "TestScrollViewSavebmp.h"

#include "TestScrollViewSavebmpDoc.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#endif


// CTestScrollViewSavebmpDoc

IMPLEMENT_DYNCREATE(CTestScrollViewSavebmpDoc, CDocument)

BEGIN_MESSAGE_MAP(CTestScrollViewSavebmpDoc, CDocument)
END_MESSAGE_MAP()


// CTestScrollViewSavebmpDoc construction/destruction

CTestScrollViewSavebmpDoc::CTestScrollViewSavebmpDoc()
{
	// TODO: add one-time construction code here
	m_sztotalLog.cx = 5000; /**x轴坐标最大值*/
	m_sztotalLog.cy = 5000; /**y轴坐标最大值*/
}

CTestScrollViewSavebmpDoc::~CTestScrollViewSavebmpDoc()
{
}

BOOL CTestScrollViewSavebmpDoc::OnNewDocument()
{
	if (!CDocument::OnNewDocument())
		return FALSE;

	// TODO: add reinitialization code here
	// (SDI documents will reuse this document)

	return TRUE;
}




// CTestScrollViewSavebmpDoc serialization

void CTestScrollViewSavebmpDoc::Serialize(CArchive& ar)
{
	if (ar.IsStoring())
	{
		// TODO: add storing code here
	}
	else
	{
		// TODO: add loading code here
	}
}


// CTestScrollViewSavebmpDoc diagnostics

#ifdef _DEBUG
void CTestScrollViewSavebmpDoc::AssertValid() const
{
	CDocument::AssertValid();
}

void CTestScrollViewSavebmpDoc::Dump(CDumpContext& dc) const
{
	CDocument::Dump(dc);
}
#endif //_DEBUG


// CTestScrollViewSavebmpDoc commands
void CTestScrollViewSavebmpDoc::DrawDoc(CDC* pDC)
{
	CRect rtbk;
	rtbk.SetRect(0,0,m_sztotalLog.cx,m_sztotalLog.cy);

	/**将背景刷成白色*/
	pDC->FillSolidRect(rtbk, RGB(255,255,255));

	/**画两个不可能一块出现在视窗口中有色方块*/
	CRect rt1, rt2;
	rt1.SetRect(100, 100, 400, 400);
	rt2.SetRect(1000, 1000, 4000, 4000);

	pDC->FillSolidRect(rt1, RGB(255,0,0));
	pDC->FillSolidRect(rt2, RGB(0, 255, 0));
}
~~~
然后在CScrollView的OnDraw函数中调用绘制内容的函数，同时记得在OnInitialUpdate()函数中设定滚动视窗的坐标大小范围。

代码如下。
~~~C++
void CTestScrollViewSavebmpView::OnInitialUpdate()
{
 CScrollView::OnInitialUpdate();

 CSize sizeTotal;
 // TODO: calculate the total size of this view
 /**从文档获得视窗大小*/
 CTestScrollViewSavebmpDoc* pDoc = GetDocument();
 ASSERT_VALID(pDoc);

 sizeTotal = pDoc->GetTotalSize();
 
 SetScrollSizes(MM_TEXT, sizeTotal);
}

...

void CTestScrollViewSavebmpView::OnDraw(CDC* pDC)
{
 CTestScrollViewSavebmpDoc* pDoc = GetDocument();
 ASSERT_VALID(pDoc);
 if (!pDoc)
  return;

 // TODO: add draw code for native data here
 pDoc->DrawDoc(pDC);
}
~~~
好了，到此为止，测试工程准备好了。现下就是来添加代码来验证想法了。

因为滚动视图的内容的绘制函数是由我们自己来控制的，将视图内容保存成图片就将会非常简单的事情。

我们只需要创建一块与视图窗口DC相适配的内存DC，然后为该内存DC关联一个适配的位图，之后再在内存DC上调用我们绘制视图内容的函数，绘制完毕，视图内容就保存在之前关联好的位图上了，接下来的事情，就是如果将一个位图保存文件的事情了，这个网上有很多方法的呢。具体见代码。

我们添加一个command来测试我们的代码。为CTestScrollViewSavebmpView添加ID_FILE_SAVE的command响应函数。然后在该响应函数中来完成我们的功能。

~~~C++
void CTestScrollViewSavebmpView::OnFileSave()
{
 // TODO: Add your command handler code here
 /**从文档获得视图大小*/
 CTestScrollViewSavebmpDoc* pDoc = GetDocument();
 ASSERT_VALID(pDoc);

 CSize size = pDoc->GetTotalSize();
 int   Width  = size.cx;
 int   Height = size.cy;

 /**获得当前DC*/
 CDC* pDC = GetDC();
    
 /**创建关联位图*/
 CBitmap   bm;   
 bm.CreateCompatibleBitmap(pDC,Width,Height);   
    
 /**创建与当前DC适配的内存DC*/
 CDC   memdc;   
 memdc.CreateCompatibleDC(pDC);

 /**为内存DC关联位图*/
 CBitmap   *pOld=memdc.SelectObject(&bm);

 /**在内存DC绘制文档内容*/
 pDoc->DrawDoc(&memdc);

 SYSTEMTIME sys;
 GetLocalTime(&sys);
 CString strtime;
 strtime.Format(_T("%04d%02d%02d%02d%02d%02d"),sys.wYear,sys.wMonth,sys.wDay,sys.wHour,sys.wMinute, sys.wSecond);
  
 CString strfilename = _T("D://") + strtime;
 strfilename += _T(".jpg");

 HRESULT hr;
   
 /**方法一，保存图片*/
#if 0       
 CImage img;
 int nBPP = memdc.GetDeviceCaps(BITSPIXEL) * memdc.GetDeviceCaps(PLANES);
 if(nBPP < 24) 
 {
  nBPP = 24;
 }

 BOOL bStat = img.Create(Width, Height, nBPP);
 ASSERT(bStat);

 CImageDC imageDC(img);

 ::BitBlt(imageDC, 0, 0, Width, Height, memdc, 0, 0, SRCCOPY);



 hr = img.Save(strfilename); 
#endif

 /**释放资源*/
 memdc.SelectObject(pOld);
 memdc.DeleteDC();   
 ReleaseDC(pDC);  

 /**保存成图片，后面找到一方便的方法*/
 CImage imgTemp;
 imgTemp.Attach(bm.operator HBITMAP());
 hr = imgTemp.Save(strfilename); 
}
~~~

好了，代码都写好了，来验证下吧。

编译运行，将看到如下界面：
![scrollview] [scrollview]

[scrollview]:  {{"/posts/capture_scrollview.jpg" | prepend: site.imgrepo }}

上面看到的时视图绘制的内容，滚动都不能看完整的视图视图。

点击File/Save，将调用我们写好的函数将当前视图保存成图片。执行过后，程序果然在D盘为我们生成了一张jpg图片，贴上：

到此，测试验证通过了呢。。

