---
layout: post
title: 一个可以记住上一次的选择的选择目录的对话框的实现代码段
tags: MFC
categories: 编辑
---

//下面的代码将调出一个选择目录的对话框

~~~c
BROWSEINFO bi;

char Buffer[MAX_PATH];

bi.hwndOwner = m_hWnd;

bi.pidlRoot = NULL;

bi.pszDisplayName = Buffer;

bi.lpszTitle = "Select drectory";

bi.ulFlags = BIF_RETURNONLYFSDIRS | BIF_NEWDIALOGSTYLE | BIF_VALIDATE;

bi.iImage = 0;

 

  //注册一个callback，来设定目录选择对话框的初始选择目录

  //这里假设lastpath是用来记录初始选择目录的字串指针

  //即打开选择目录对话框时，默认选择上一回选择的目录

 

  if (lastpath != NULL)   //该值可以存放于全局变量或存储于配置文件中

  {

  	bi.lParam = lastpath; //传给callback的参数，即初始选择的目录路径

  }

  else

  {

      bi.lParam = NULL;

  }

 

  bi.lpfn = BrowseCallbackProc;   //注册回调函数 

 

LPITEMIDLIST pIDList = SHBrowseForFolder(&bi);  //显示目录选择对话框 

 

if(pIDList)

{

	SHGetPathFromIDList(pIDList, Buffer);   //获取用户选择的目录

	Global_Free(pIDList);//记得释放pIDList，这里函数名记不太清了，大伙可以查下MSDN

}



 

 

 

//回调函数的实现，主要响应两个消息

inline int CALLBACK BrowseCallbackProc(HWND   hwnd,   UINT   uMsg,   LPARAM   lParam,   LPARAM   lpData)   

{   

    lParam = lParam;

    switch(uMsg)

    {   

    case BFFM_INITIALIZED:  //初始化时，选择我们记录的路径

        {

            ::SendMessage(hwnd,BFFM_SETSELECTION,TRUE,(LPARAM)lpData);   

        }

        break;   

    case BFFM_SELCHANGED:   //目录选择改变时，记录选择的目录

        {

            ::SendMessage(hwnd,BFFM_SETSTATUSTEXT,0,lpData);    

        }

        break;   

    }   

    return 0;   

}
~~~