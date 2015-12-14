---
layout: post
title: windows平台获取Chromium源代码
tags: chromium windows
categories: chromium
---

The Chromium Projects是使用gclient工具来维护代码。
在Windows上想要运行gclient，用cygwin会更好点，虽然gclient有win32版本的可执行文件。
获取源代码之前的准备工作：安装cygwin执行环境，安装depot_tools工具。
参考文档：http://dev.chromium.org/developers/how-tos/get-the-code#TOC-Check-out-the-sources

咱只是把它翻译下哈，抽取其中Windows平台下需要的东西然后写成一个日记吧。


一、安装cygwin
参考文档：http://dev.chromium.org/developers/how-tos/cygwin

从这里http://cygwin.org/setup.exe下载cygwin的安装程序，然后执行，按默认选项进行安装。

注意安装过程有一项，记得安装以下的包：
"apache",
"bc",
"bison",
"curl",
"diffutils",
"e2fsprogs",
"emacs",
"flex",
"gcc",
"gperf",
"keychain",
"make",
"nano",
"openssh",
"patch",
"perl",
"perl-libwin32",
"python",
"rebase",
"rsync",
"ruby",
"subversion",
"unzip",
"vim",
"zip"

cygwin安装完成后，rebase的工作可以参考我之前的博客进行。

http://blog.csdn.net/sunshineboyleng/article/details/6725656

二、安装depot_tools工具

参考文档：http://dev.chromium.org/developers/how-tos/install-depot-tools
1.下载depot_tools工具
打开一个cygwin的shell窗口，在命令行中输入：
svn co http://src.chromium.org/svn/trunk/tools/depot_tools
命令执行完毕，在home目录将会看到depot_tools目录。
2.将depot_tools工具目录添加到路径环境变量里
在这里，我的本本的目录为/home/Admin/depot_tools。
vi ~/.bashrc
添加内容：export PATH="$PATH":/home/Admin/depot_tools到文件末尾。
然后保存文件并退出。
3.退出当前cygwin的shell窗口，重新开启新的cygwin的shell窗口，准备开始下载chromium源代码。


三、使用gclient下载chromium源代码

参考文档：http://dev.chromium.org/developers/how-tos/get-the-code#TOC-Windows
1.建立源代码目录：mkdir chromiumtrunk 
2.进入刚建立的目录：cd chromiumtrunk
3.生成gclient配置文件：gclient config http://src.chromium.org/svn/trunk/src
这样会在当前目录生成一个.gclient文件，在这里我的文件路径为/home/Admin/chromiumtrunk/.gclient
4.编辑生成的.gclient文件，添加如下的内容，这样可以不用下载一些测试的代码
  "custom_deps": {      
    "src/third_party/WebKit/LayoutTests": None,
    "src/chrome/tools/test/reference_build/chrome": None,
    "src/chrome_frame/tools/test/reference_build/chrome": None,
    "src/chrome/tools/test/reference_build/chrome_linux": None,
    "src/chrome/tools/test/reference_build/chrome_mac": None,
    "src/third_party/hunspell_dictionaries": None,
  }
5.更新到最新的代码
在准备更新到最新代码之前，打开下面的网页来确认最新代码树是否可以成功编译http://build.chromium.org/p/chromium/console。

这个Tree is open字样可能在天朝的路由下无法打开，访问这个网点里大伙最好用下代理，翻墙才能看到。
打开的网页的顶端看到tree is open字样表示当前最新的代码树是可以成功编译的，可以更新到最新的代码。
如果看到tree is close则当前最新代码可能无法成功编译，这时需要再等些时候来更新代码。
输入命令gclient sync来将代码更新到latest版本。
当然你也可以输入gclient sync --revision src@####来将代码更新到指定的revision，其中####代表revision的版本号。
在http://build.chromium.org/p/chromium/console同时可以看到最近的几个版本所做的修改的记录。


上图中的111703就是revision的版本号。


chromium工程的代码量很多，估计要下载需要好几个小时吧，这个拼的就是网速啦~现在代码正在sync，希望明早醒来代码已经下载好哈~
这样明天就可以编译版本了~
困了，晚安！