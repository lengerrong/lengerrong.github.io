---
layout: post
title: VS2015 windows 编译WebKit 代码
tags: webkit build
categories: build
---

1. 安装开发工具，请参照：
http://www.webkit.org/building/tools.html
2.拿代码，请参照：

http://www.webkit.org/building/checkout.html

执行下面命令时 ./Tools/Scripts/update-webkit，可能会遇到一些pm文件找不到的错误，类似：

Can't locate HTTP/Date.pm in @INC

Can't locate LWP/Simple.pm in @INC
解决方法：使用Cygwin Setup安装必要的Perl扩展模块，如下图


下面的命令给遗漏了，请记得在编译之前执行，否则在编译时将会遇到<CoreFoundation/CoreFoundation.h>头文件找不到的错误

./Tools/Scripts/update-webkit-support-libs
3.开始编译，请参照：

http://www.webkit.org/building/build.html

由于webkit上面的文档大部分都过时了，实际编译过程中，产生了不少莫名的错误。

下面记录一下编译中遇到的错误及解决办法。


error 1: <internal:gem_prelude>:1:in `require': cannot load such file -- rubygems.rb

reason 1: cygwin里的ruby没有包括gems模块

solution 1:使用cygwin的安装文件重新安装一下ruby的gems模块，如下图



error 2 : 11>C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V140\Microsoft.CppCommon.targets(128,5): error MSB3073: 命令“@REM Do not edit from the Visual Studio IDE! Customize via a testRegExpLauncherPreLink.cmd file.
11>C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V140\Microsoft.CppCommon.targets(128,5): error MSB3073: @if not exist "C:\cygwin\home\lenger\webkit\Source\JavaScriptCore\JavaScriptCore.vcxproj\testRegExp\testRegExpLauncherPreLink.cmd" exit /b


reason: MSB3073 错误是由于依赖的项目的编译顺序不对或是脚本文件有错误之类的问题导致的。

这里是因为testRegExpLauncherPreLink.cmd 文件内容为空导致这个错误的。
solution:删除不必要的脚本文件，累计共删除了以下的xxx.cmd文件

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/JavaScriptCorePreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/jsc/jscPreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/jsc/jscLauncherPreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/testRegExp/testRegExpPreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/testRegExp/testRegExpLauncherPreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/testapi/testapiPreLink.cmd

rm Source/JavaScriptCore/JavaScriptCore.vcxproj/testapi/testapiLauncherPreLink.cmd

rm Source/WebKit/WebKit.vcxproj/WebKit/WebKitPreLink.cmd


WebKit for windows 如何开启Feature：

比如我想打开indexdb这一feature（默认是关着的）。

编辑文件WebKitLibraries\win\tools\vsprops\FeatureDefines.props，做如下修改：

<ENABLE_INDEXED_DATABASE />  ==> <ENABLE_INDEXED_DATABASE>ENABLE_INDEXED_DATABASE</ENABLE_INDEXED_DATABASE>

再次重新打开WebKit.sln工程，INDEXED_DATABASE这一feature就打开了。