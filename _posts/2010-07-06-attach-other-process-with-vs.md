---
layout: post
title: 通过写代码，attach程序中创建的其他进程，将其加入VS的Debugger，以方便调试
tags: MFC
categories: 编程
---


主要功能函数:AttachProcess，DettachProcess

AttachProcess将进程加入到debugger以方便调试，

DettachProcess杀掉进程。

 

使用方法：在需要调试时调用AttachProcess，在中断调试时调用DettachProcess杀掉进程。

 

函数实现：

/**

把进程attch给debugger

lpWndName，进程的窗口名字

*/
DWORD AttachProcess(LPCWSTR lpWndName)
{
 HWND  hWnd = NULL;
 DWORD hInstance = -1;
 hWnd = FindWindow(NULL, lpWndName);  //根据进程窗口名字找到进程窗口句柄
 if (NULL != hWnd)
 {
  GetWindowThreadProcessId(hWnd, &hInstance);  //根据进程窗口句柄获取进程的PID
  if (hInstance > 32)
  {
   if (0 == DebugActiveProcess((DWORD)hInstance))   //将进程attach给debugger
   {
    printf("debug active process failed!");
   }
   else
   {
    printf("debug active process success!");
   }
  }
 }

 return hInstance;   // 返回进程PID
}

 

/**

根据进程的PID，杀掉进程

*/

void DettachProcess(DWORD hInstance)
{
 HANDLE hProcess = NULL;
 hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, hInstance);  //根据进程PID获取进程句柄
 TerminateProcess(hProcess, 0);  //根据进程句柄杀掉进程，这样与debugger就分开了
}

 

测试工程，请到我的资源列表中下载，《一个简单的可以Debug用CreateProcess创建的进程例子》。

 

例子工程源码地址：http://download.csdn.net/source/2519155

 

现贴出主要测试代码mainfrm.cpp

 

// MainFrm.cpp : implementation of the CMainFrame class
//

#include "stdafx.h"
#include "test.h"

#include "MainFrm.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#endif

/**

把进程attch给debugger

lpWndName，进程的窗口名字

*/
DWORD AttachProcess(LPCWSTR lpWndName)
{
 HWND  hWnd = NULL;
 DWORD hInstance = -1;
 hWnd = FindWindow(NULL, lpWndName);  //根据进程窗口名字找到进程窗口句柄
 if (NULL != hWnd)
 {
  GetWindowThreadProcessId(hWnd, &hInstance);  //根据进程窗口句柄获取进程的PID
  if (hInstance > 32)
  {
   if (0 == DebugActiveProcess((DWORD)hInstance))   //将进程attach给debugger
   {
    printf("debug active process failed!");
   }
   else
   {
    printf("debug active process success!");
   }
  }
 }

 return hInstance;   // 返回进程PID
}

/**

根据进程的PID，杀掉进程

*/

void DettachProcess(DWORD hInstance)
{
 HANDLE hProcess = NULL;
 hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, hInstance);  //根据进程PID获取进程句柄
 TerminateProcess(hProcess, 0);  //根据进程句柄杀掉进程，这样与debugger就分开了
}


// CMainFrame

IMPLEMENT_DYNAMIC(CMainFrame, CMDIFrameWnd)

BEGIN_MESSAGE_MAP(CMainFrame, CMDIFrameWnd)
 ON_WM_CREATE()
END_MESSAGE_MAP()

static UINT indicators[] =
{
 ID_SEPARATOR,           // status line indicator
 ID_INDICATOR_CAPS,
 ID_INDICATOR_NUM,
 ID_INDICATOR_SCRL,
};

static DWORD hInstance = -1;

// CMainFrame construction/destruction

CMainFrame::CMainFrame()
{
 // TODO: add member initialization code here
 ::ShellExecute(NULL, L"open", L"F://projects//test//debug//test3.exe", NULL, NULL, SW_SHOW);  // 启动test3.exe

 //等待test3.exe启动完毕
 while(1)
 {
  HWND hWnd = ::FindWindow(NULL, L"test3");
  if (hWnd != NULL)
  {
   break;
  }
 }
}

CMainFrame::~CMainFrame()
{

//如果调用了DebugSetProcessKillOnExit(FALSE)，则调用下面的函数将attatch上的进程干掉
 DettachProcess(hInstance);

}


int CMainFrame::OnCreate(LPCREATESTRUCT lpCreateStruct)
{
 if (CMDIFrameWnd::OnCreate(lpCreateStruct) == -1)
  return -1;
 
 if (!m_wndToolBar.CreateEx(this, TBSTYLE_FLAT, WS_CHILD | WS_VISIBLE | CBRS_TOP
  | CBRS_GRIPPER | CBRS_TOOLTIPS | CBRS_FLYBY | CBRS_SIZE_DYNAMIC) ||
  !m_wndToolBar.LoadToolBar(IDR_MAINFRAME))
 {
  TRACE0("Failed to create toolbar/n");
  return -1;      // fail to create
 }

 if (!m_wndStatusBar.Create(this) ||
  !m_wndStatusBar.SetIndicators(indicators,
    sizeof(indicators)/sizeof(UINT)))
 {
  TRACE0("Failed to create status bar/n");
  return -1;      // fail to create
 }

 // TODO: Delete these three lines if you don't want the toolbar to be dockable
 m_wndToolBar.EnableDocking(CBRS_ALIGN_ANY);
 EnableDocking(CBRS_ALIGN_ANY);
 DockControlBar(&m_wndToolBar);

 hInstance = AttachProcess(L"test3");   // 将test3.exe加入调试

 //DebugSetProcessKillOnExit(FALSE); // 调试结束或是中断，并不关闭test3.exe，VS默认调试结束时会关闭其attach上的进程

 return 0;
}

BOOL CMainFrame::PreCreateWindow(CREATESTRUCT& cs)
{
 if( !CMDIFrameWnd::PreCreateWindow(cs) )
  return FALSE;
 // TODO: Modify the Window class or styles here by modifying
 //  the CREATESTRUCT cs

 return TRUE;
}


// CMainFrame diagnostics

#ifdef _DEBUG
void CMainFrame::AssertValid() const
{
 CMDIFrameWnd::AssertValid();
}

void CMainFrame::Dump(CDumpContext& dc) const
{
 CMDIFrameWnd::Dump(dc);
}

#endif //_DEBUG


// CMainFrame message handlers

 

 

 

以下测试结果截图，test3.exe已经被我们写的code加入到了VS调试器中，如果test3.exe程序的调试也存在，就可以进入到test3.exe进行调试了。