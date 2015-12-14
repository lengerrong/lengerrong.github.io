---
layout: post
title: windows下如何将cygwin中所有的dll重新rebase
tags: cygwin windows
categories: 环境搭建
---

在cygwin的安装目录（通常是c:\cygwin\bin）下提供了一个rebase的脚本程序。


在同一目录下有个ash.exe程序是用来执行这个脚本的。



在windows系统上，按如下步骤做即可：

1.打开CMD窗口（按下Win+R键启动运行窗口，输入CMD，最后按下回车键即可）

2.切换到c:\cygwin\bin目录（命令行：cd /d c:\cygwin\bin）

3.运行ash.exe程序（命令行：ash.exe）

4.启动rebase脚本（命令：./bin/rebaseall -v）


这么做主要是为了解决在Vista或是Windows7系统下，Cygwin的fork出不来的实现问题。

