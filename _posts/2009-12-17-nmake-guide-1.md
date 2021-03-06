---
layout: post
title: NMAKE Guide (一)
tags: NMAKE
categories: build
---

本文介绍了微软程序维护实用工具（NMAKE）1.20版本。NMAKE是一个复杂的命令处理器，节省时间并简化项目管理。
一旦您指定了项目文件之间的依赖关系，NMAKE即可自动编译项目而不需要重新自编译上一次编译后并没有改变的文件。

如果您使用程序员工作台（PWB）构建您的项目，PWB自动创建一个makefile文件并调用NMAKE运行该文件来帮您编译项目。
如果您打算构建您的项目而不使用PWB，或者如果你想理解或修改PWB创建的makefile文件，又或是您想在PWB中使用另一版本的makefile文件（非PWB创建的，那么您或许会想要阅读下本文。



NMAKE可以交换自己以扩充内存，扩展内存或磁盘以预留空间给它产生的命令来运行。（更多的信息可以查看/M选项的详细介绍在531页。）

新的功能



NMAKE1.20版本新增以下特性，对于每个功能的详情，请参阅本文中对应的引用部分。
新选项：/B, /K, /M, /V
指令：!MESSAGE
两个预处理操作：DEFINED, EXIST
三个关键字同!ELSE指令的一块使用：IF, IFDEF,IFNDEF
新增指令：!ELSEIF, !ELSEIFDEF, !ELSEIFNDEF
增加.cpp和.cxx文件后缀到.SUFFIXES（后缀）列表
C++程序的预定义宏：CPP, CXX, CPPFLAGS, CXXFLAGS
C++程序预定义推导规则（IR）

概述

NMAKE靠检查文件的时间戳来工作。时间戳记录了文件最后一次修改时的时间和日期。  大多数操作系统会在文件被修改的二秒间隔内为文件打上时时戳。NMAKE对比目标文件的时间戳与其依赖的文件的时间戳。  目标通常是一个你想创建的文件，比如一个可执行文件，但它可能是一个你希望执行的一系列命令的标签。依赖通常是目标文件创建的源头，比如一份源文件。  一个目标将会被称为过期的，如果任意一个其依赖文件具有比目标时间戳以后的时间戳或是目标不存在。（更多的信息关于2秒的间隔可影响你的项目的编译，可以查看/B选项的详细介绍在530页。）

警告：为使NMAKE正常的工作，您的系统上的日期和时间设置必须与于以前的设置一致。如果每次启动系统时你都会设置时间和日期，须仔细并正确地的设置。  如果你的系统保存一项设置，确保电源是在工作的。

当你运行NMAKE，NMAKE将读取你提供的makefile文件，makefile文件（有时称为描述文件）是一份包含NMAKE用来编译您的项目的一系列指令的文本文件。  这些命令由描述块、宏、指令和推导规则组成。每个描述块通常会列出一个目标（或多个目标），目标的依赖，及那些建立目标的命令。  NMAKE对比目标文件的时间戳和依赖文件的时间戳。如果任意一个依赖文件的时间戳与目标文件的时间戳一致或是比目标文件的时间戳要晚，那么NMAKE将执行列在描述块中的命令以重新生成目标文件。

运行NMAKE而不提供makefile文件，这是可行的。在这种情况下，NMAKE将使用命令行参数中给定或是TOOLS.INI配置文件中提供的预定义宏和推导规则指令。（更多关于TOOLS.INI文件的信息，请参阅534页。）

NMAKE的主要目的是帮助您快捷方便地的生成程序。然而，它并不局限于编译和链接；NMAKE可以运行其他类型的程序 和执行操作系统的命令。  您可以使用NMAKE备份、移动文件和运行那些您常在操作系统提示下执行的其他的工程管理任务。

本文使用术语“生成“来表示目标文件的编译过程，意思是评估目标和其依赖文件的时间戳，如果目标是过时的，则执行与目标相关的命令。术语"生成”有这么一个意思，这些命令有实际创建或改变目标文件或者是没有。

运行NMAKE

您调用下面的句法即可运行NMAKE：
NMAKE [options] [macros] [targets]
选项字段列出NMAKE选项，NMAKE选项将在章节“命令行选项”进行详细介绍。
宏字段列出宏的定义，它允许你改变makefile文件的内容。对于宏的语法在551页“用户自定义宏”章节有描述。
目标字段列出了那些需要生成的目标文件。NMAKE只生成列在命令行参数目标字段的目标文件。如果你不指定一个目标，NMAKE只生成makefile文件中第一个依赖的第一个目标。  （您可以使用伪目标让NMAKE来生成多个目标文件。详细请参阅540页”伪目标“一节。）

NMAKE使用如下优先级去决定怎样管理生成过程。
1.如果使用了/F选项，NMAKE于当前目录或指定目录搜索指定的makefile文件，如果makefile文件不存在，那么NMAKE暂停并显示错误信息。
2.如果/F选项没有被使用，NMAKE于当前目录搜索一个名叫MAKEFILE的文件作为makefile文件。
3.如果在当前目录并不存在名为MAKEFILE的文件，NMAKE检查命令行中的目标文件并尝试去生成目标文件使用推导规则（预定义的或是定义在TOOLS.INI文件的）。
这一特性使得您可以运行NMAKE而无须提供一份makefile文件只要NMAKE存在一份适用于目标文件的推导规则。
4.如果并没有提供makefile文件并且命令行参数中没有指定目标，那么NMAKE暂停并显示错误信息。

例子

下面这条命令指定了/S选项和一个宏定义（”program=sample"），让NMAKE生成两个目标文件（sort.exe和search.exe）。
该命令并没有指定一份makefile文件，所以NMAKE在当前目录查找MAKEFILE文件或是使用默认推导规则。

NMAKE /S "program=sample" sort.exe search.exe

关于NMAKE的宏定义信息，请参阅550页。

命令行选项

NMAKE接收选择项以控制NMAKE会话。选择项并不区分大小写且其前面可以是斜线（/）或是破折号（-）。
您可以指定一些选择项在makefile文件中或是TOOLS.INI文件中。

/A
强制NMAKE去生成所有检测到的目标文件，甚至目标文件相比于其依赖文件并没有过期。此选项并不强制NMAKE产生无关的目标文件。

/B
告诉NMAKE去执行依赖性检查即使时间戳是一样的。大多数操作系统在2秒种内分配一个时间戳。如果您的命令执行地很快，NMAKE也许可能认为一个文件是最新的但其实不是。
此选项可能会导致一些不必要的生成步骤，但建议在速度非常快系统上运行NMAKE时加上该选项。

/C 
禁止默认NMAKE输出，包括非致命NMAKE错误或警告讯息，时间戳，以及NMAKE版权信息。如果同时指定/C和/K选项，那么/C将会禁止来自/K选项的警告输出。

/D
打印信息NMAKE执行期间的信息。这些信息散布于NMAKE的默认输出中打印到屏幕。NMAKE打印在生成过程检测到的每个目标文件和依赖文件的时间戳，并报告一个信息当一个目标文件不存在时。
目标文件的依赖文件的打印信息在目标文件之前且是交错的。/D和/P选项在调试一个makefile时将会非常有用。
设置或清除部分makefile文件的/D选项，可以使用!CMDSWITCHES指令，具体请参阅572页“预处理指令”章节。

/E
引用环境变量来覆盖makefile文件中的宏定义。请参阅“宏定义”一节于550页。

/F filename
指定makefile文件的文件名。文件名前可以加空格或制表符。如果你指定破折号（-）于文件名前，NMAKE将从标准输入设备获得makefile的输入。
（按下F6或是CTRL+Z来结束键盘输入。）NMAKE可以接受多个makefile文件，每个makefile文件之间用/F隔开。
如果你忽略了/F选项，NMAKE搜索当前目录中文件名为MAKEFILE（没有后缀）的文件并将其当做makefile输入。
如果MAKEFILE文件并不存在，NMAKE使用默认推导规则来生成命令行中的目标。

/HELF
调用快速帮助功能。如果NMAKE不能定位帮助文档可是快速帮助，那么它显示一份NMAKE命令行语句的简要总结。

/I
忽略列在makefile文件中的所有命令的退出代码。NMAKE执行整个makefile文件即使有错误产生。如果只想忽略某些命令或是某部分makefile的退出代码，可以在命令之前加破折号（-）或是使用.IGNORE指令。
详情请参阅544页的”命令修饰符“节和570页的”点指令“节。想要设置或清除部分makefile的/I选项，可以使用!CMDSITCHES指令。具体请看572页”预处理指令“一节。
想要忽略生成过程无关部分的错误，可以使用/K选项。/I覆盖/K选项，如果两选择项同时被设置。

/K
继续关系依赖树其余部分的生成过程如果一条命令由于错误而中断。默认情况下，NMAKE停止如果有命令返回任一非零的退出值。
如果该选择项被指定，那么当一个命令返回一个非为的退出值，NMAKE停止执行包含该命令的块。NMAKE并不尝试去生成那些取决于该命令结果的目标文件；相反，NMAKE报告一个警告，然后继续生成其他的目标。
如果/K选择项被指定同时生成过程并没有结束，NMAKE返回一个退出值1。这与/I选项不同，/I选项完全忽略退出代码。/I覆盖/K选项如果它们同时被指定。/C选项禁止/K选项报告的警告信息。

/M
交换NMAKE到磁盘以保存延长或扩充的内存在MS-DOS。默认情况下，NMAKE在低地址内存留下空间给命令执行通过交换它自己到扩展的内存（如果有足够的空间存在）或是扩充的内存（如果没有足够的扩展的内存，但有足够的扩充内存）或磁盘。
/M选择项让NMAKE忽略任何的可扩展的内存或是可扩充内存，而只交换自己到磁盘。

/N
显示但并不执行那些在生成过程中会被执行的命令。该选择项对于调试makefile和查看那个目标过期时非常有用。
想要设置或清除makefile文件的某部分/N选择项，可使用!CMDSWITCHES指令。详情见572页”预处理指令“节。

/NOLOGO
禁止输出NMAKE版权信息。

/P
在开始NMAKE会话前，打印NMAKE的信息到标准输出设备，包括所有的宏定义、推导规则、目标描述块和.SUFFIXES列表。
如果/P选择项被指定但没有提供makefile文件或是命令行目标，那么NMAKE打印信息但并不报告错误。/P和/D选择项对于高度makefile文件非常有用。

/Q
检查在生成过程中可能会更新的目标文件的时间戳但并不执行生成过程。如果目标文件都是最新的，那么NMAKE返回一个零值；如果任一目标文件过期了，那么NMAKE返回一个非零值。
只有makefile文件中的预处理命令将会被执行。该选择项在批处理文件中调用NMAKE时非常有用。

/R
清空.SUFFIXES列表且忽略那些定义在TOOLS.INT文件中或是预定义的推导规则和宏定义。

/S
禁止所有的执行命令的输出。为了禁止某一执行命令的输出，使用@命令修饰符或是使用.SILENT指令。详情请参阅544页的”命令修饰符“节和570页的”点指令“节。
想要设置或清除makefile文件的某部分/S选择项，可使用!CMDSWITCHES指令。详情见572页”预处理指令“节。

/T
修改命令行目标（或是makefile文件中第一个目标文件如果命令行中并没指定任何目标）的时间戳为当前时间并且执行预处理命令但并不执行生成过程。
目标文件的内容并没有改变。

/V

引起递归时所有的宏被继承。默认情况下，只有定义在命令行和环境变量中的宏定义是被继承的当NMAKE调用递归时。该选择项使所有的宏定义在NMAKE调用递归期间都有效。详情见563页”继承的宏定义“节。

/X filename
发送所有的NMAKE错误输出到filename，filename可以是文件或是设备。
filename之前可以加空格或制表符。如果您在filename之前加破折号（-）,NMAKE发送它的错误输出到标准的输出设备。
默认情况下，NMAKE发送错误输出到标准错误流。 该选择项并不影响那些由makefile中的命令产生的发送到标准错误流的错误输出。

/?
打印一份NMAKE命令行句法的简要总结并退出到操作系统。

例子


下面的命令行指定了两个NMAKE选择项：
NMAKE /F sample.mak /c targ1 targ2

/F选择项告诉NMAKE去读取sample.mak文件。/C选择项告诉NMAKE不要打钱非致命的错误和警告信息。
该命令指定了两个目标去更新。




