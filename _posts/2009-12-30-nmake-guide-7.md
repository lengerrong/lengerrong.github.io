---
layout: post
title: NMAKE Guide (七)
tags: NMAKE
categories: build
---

指令
NMAKE提供了数种方法来控制NMAKE会话过程通过点指令和预处理指令。
指令是在makefile文件或是TOOLS.INI文件中给NMAKE的说明。
NMAKE处理点指令和预处理指令并将结果应用于makefile文件在其处理makefile文件中的依赖关系和命令之前。

点指令
点指令必须出现在描述块之外且出现在一行的开始位置。
点指令以.开头并且跟随一个冒号：。冒号的前面和后面都可以有空格或是制表符。
这些指令的名字都区分大小写且必须都大写。

.IGNORE:
忽略makefile文件中调用的程序的非零返回退出代码。
默认情况下，NMAKE将停止运行如果一个命令返回非零的退出代码。
该指令的影响从其在makefile文件中指定的位置开始到文件的结束。
想要关闭该指令，使用预处理指令!CMDSWITCHES。
忽略某个命令的退出代码，请使用命令修饰符—。
忽略整个makefile文件的退出代码，在调用NMAKE时使用/I选项。

.PRECIOUS:targets
告诉NMAKE不要删除目标如果生成他们的命令忽然中断了。
该指令将不会起作用如果一个处理删除文件的中断的命令被中断了。
多个目标之间使用空格或是制表符来分隔。
默认情况下，NMAKE目标如果生成过程被ctrl+c或是ctrl+break中断了。
多样的指定是可以积累的，每个.PRECIOUS都将应用于整个makefile文件。

.SILENT:
抑制执行命令行的显示。
默认情况下，NMAKE打印其调用的每条命令。
该指令的作用范围从其在文件中指定的位置到文件结束。
相要关闭该指令的作用，同样使用!CMDSWITCHES预定义指令。
抑制单个命令的打印输出，请使用命令修饰符@。
抑制整个makefile文件的命令的打印输出，请使用/S选项调用NMAKE程序。

.SUFFIXES:list
列出给NMAKE匹配的后缀名，当NMAKE尝试应用一条推导规则。
（更多的信息关于.SUFFIXES的使用，请查看563页的"推导规则“一节。）
预定义的名单如下：

.SUFFIXES : .exe .obj .asm .c .cpp .cxx .bas .cbl .for .pas .res .rc

想要在名单新增后缀名列表，请使用：

.SUFFIXES:suffixlist

如果suffixlist是一系列的后缀名集合，请用一个或是多个空格或是制表符分隔每个后缀名。
想要清除后缀名列表，请如下使用：

.SUFFIXES:

在冒号后不要添加任何后缀名。
想要改变列表中的顺序或是指定一个完整的全新的后缀名单集合，
您必须清除先前的名单然后重新指定一个新的设置。
想要知道当前的设置，请使用/P选项运行NMAKE程序。

预处理指令
NMAKE预处理指令和编译预处理指令很相似。
您可以使用多个预处理指令来有选择地处理您的makefile文件。
使用其他预处理指令，您可以打印错误信息、包含其他文件、取消一个宏定义和关闭特定的选项。
NMAKE读取和执行预处理指令在处理整个makefile文件之前。

预处理指令以感叹号!开始，且感叹号必须出现在一行的开始位置。
感叹号和指令的关键字之间可以有一个或多个空格或是制表符，这给样允许缩进以提高可读性。
这些指令（他们的关键词和操作）并不区分大小写。

!CMDWITCHES{+|-}opt...
开启或是关闭一个或多个选项。（选项的描述请看529页。）
指定一个操作，不论是使用一个+号来开启选项或是使用一个-号来关闭选项，
后面跟着代表着选项的一个或多个字母。这些字母并不区分大小写，不用指定反斜杠。
用一个或多个空格或是制表符来分开指令和操作。操作和选项之前不允许出现空格。
开启一些选项和关闭其他的选项，请使用多条独立的!CMDSWITCHES指令。

所有的选项除了/F,/HELP,/NOLOGO,/X和/?之外都可以出现在!CMDSWITCHES指令中在TOOLS.INI文件中。
在makefile文件中，只有字母D,I,N和S可以在!CMDSWITCHES指令中指定。
如果!CMDSWITCHES在一描述块中使用，该指令的修改要等下一个描述块中才起作用。
该指令将更新MAKEFLAGS宏，并且修改在递归调用中将被继承。

!ERROR text
显示文本text给错误U1050的标准错误信息，然后停止NMAKE会话。
该指令停止NMAKE的生成过程，即使有使用了/K、/I、.IGNORE、!CMDSWITCHES或是命令修饰符-。
在text之前的空格或是制表符会被忽略掉。

!MESSAGE text
打印文本text到标准输出，然后继续NMAKE会话。
在text之前的空格或是制表符会被忽略掉。

!INCLUDE[<filename>]
读取并将filename文件当做一份makefile文件来评价，在继续处理当前makefile文件之前。
NMAKE首先在当前目录中找寻filename文件，如果filename并没有指定路径。
如果指定了一个路径，NMAKE在指定的目录中找寻。
之后，如果!INCLUDE指令本身包含一个被包含的文件，NMAKE在包含该文件的文件所在目录寻找filename文件，这种搜索是递归的，结束于原始的makefile文件的目录。
最后，如果filename被一对尖括号包含，NMAKE将在宏INCLUDE定义的目录中寻找filename文件。
INCLUDE宏初始化为环境变量INCLUDE的值。

!IF constantexpression（常量表达式）
如果常量表达式的值不为零，NMAKE执行!IF和!ELSE或是!ENDIF之间的语句。

!IFDEF macroname
如果宏名macroname有定义过，那么执行!IFDEF和!ELSE或是!ENDIF之间的语句。
NMAKE认为一个值为空的宏是有定义的。

!IFNDEF macroname
如果宏名macroname没有被定义过，那么执行!IFNDEF和!ELSE或是!ENDIF之间的语句。

!ELSE[IF constantexpression|IFDEF macronmae|IFNDEF macronmae]
如果之前的!IF，!IFDEF或是!IFNDEF语句为零，那么执行!ELSE和!ELSE或是!ENDIF之间的语句。
可选的关键字给予了进一步的预处理的控制。

!ELSEIF
与!ELSE IF同一个意思。

!ELSEIFDEF
与!ELSE IFDEF同一个意思。

!ELSEIFNDEF
与!ELSE INNDEF同一个意思。

!ENDIF
标志着一个!IF，!IFDFE或是!IFNDEF块的结束。
在同一行上!ENDIF之后的任何字符都会被忽略掉。

!UNDEF macroname
取消一个宏的定义，将该宏名从NMAKE的符号表中删除。
（更多的信息，请看553页的"空宏和未定义的宏"一节。）

例子
如下一系列的指令：

!IF
!ELSE
! IF
! ENDIF
!ENDIF

等同于如下的指令集：
!IF
!ELSE IF
!ENDIF

预处理中的表达式
与!IF或是!ELSE IF指令一块使用的常量表达式可以是整数常量、字符常量或是程序调用。
您可以用括号将一组表达式括起来。
NMAKE将数字按十进制来处理除非他们以o（八进制）或是)0x（十六进制）开始。

NMAKE中的表达式使用C风格的有符号的长整型整数算法。
数字用32位的二进制补码的形式表示，其数值范围为-2147483648到2147483247。

两个一无操作符评估一个条件，返回逻辑数值，true(1)或是false(0)。

DEFINED(macroname)
表达式的值为true如果宏macroname有被定义。与!IF或是!ELSE IF指令一块使用时，该操作符等同于指令!IFDEF或是!ELSE IFDEF。
然而，与那些指令不同，DEFINED可以用在使用二进制逻辑操作的复杂表达式中。

EXIST(path)
表达式的值为true如果路径path存在。EXIST可以用在使用二进制逻辑操作的复杂表达式中。
如果路径path中包含空格（一些文件系统中允许有空格）,请将路径包含在一对双引号中。

整数常量可以用在一元操作符负号（-）、取反（~）和逻辑非（！）中。

常量表达式可以使用表16.2中任一的二进制操作。
比较两个字符串，请使用==或是!=操作符，将字符串包含在一对双引号中。

Table 16.2 Binary Operators for Preprocessing
-----------------------------------------------------------------------------
Operator   Description
-----------------------------------------------------------------------------
+      Addition（加）
–      Subtraction（减）
*      Multiplication（乘）
/      Division（除）
%       Modulus（取余）
&      Bitwise AND（按位与）
|      Bitwise OR（按位或）
^      Bitwise XOR（按位异或）
&&      Logical AND（逻辑与）
||      Logical OR（逻辑或）
<<         Left shift（左移）
>>         Right shift（右移)
==         Equality（等于）
!=         Inequality（不等于）
<          Less than（小于）
>          Greater than（大于）
<=         Less than or equal to（小于或等于）
>=         Greater than or equal to（大于或等于）
--------------------------------------------------------------------

例子
下面的例子演示了如何使用预处理指令来控制链接程序是否插入了调试信息给可执行程序（.exe）文件。

!INCLUDE <infrules.txt>
!CMDSWITCHES +D
winner.exe : winner.obj
!IF DEFINED(debug)
! IF "$(debug)"=="y"
LINK /CO winner.obj;
! ELSE
LINK winner.obj;
! ENDIF
!ELSE
! ERROR Macro named debug is not defined.
!ENDIF

在这个例子中，!INCLUDE指令将文件INFRULES.TXT插入到makefile文件中。
!CMDSWITCHES指令设置了/D选项，以打印文件的时间戳信息当检没文件的时间戳时。
!IF指令来检查宏debug是否有定义，如果有定义，那么接下来的!IF指令检测该宏是否被定义成值"y"。
如果确实是那样，那么NMAKE读取那条包含/CO选项的LINK命令，否则NMAKE读取那条没有包含/CO选项的LINK命令。
如果宏debug并没有定义，那么!ERROR指令打印指定的信息同时NMAKE停止运行。


 在预处理中执行程序
您可以从NMAKE中调用一个程序或是命令，然后在预处理中使用它的退出代码。
NMAKE在预处理过程中执行命令，然后用程序的返回值替代其在makefile文件中说明。
一个非零的返回值通常表明了一个错误。
您在表达式中使用程序的返回值来控制预处理。
指定的命令，包括方括号内的任何参数，（[]）。
您可以在命令中使用宏。
NMAKE会在命令执行前展开宏。

例子
下面是一个makefile文件的部分，它在执行NMAKE会话前先检测磁盘空间是否足够：

!IF [c:/util/checkdsk] != 0
! ERROR Not enough disk space; NMAKE terminating.
!ENDIF


NMAKE操作的顺序
当您在写一个复杂的makefile文件时，知道NMAKE中操作在平台中的执行顺序是非常有用的。
本节描述了这些操作和他们的顺序：

当您在命令行中调用NMAKE程序，NMAKE的第现一件事情就是找到makefile文件：

1.如果有使用/F选项，NMAKE查找在选项中指定的文件。如果NMAKE找不到该文件，NMAKE返回一个错误。

2.如果没有使用/F选项，NMAKE会在当前目录下寻找一个名为MAKEFILE的文件。
如果命令行有列出目标，NMAKE将根据MAKEFILE文件中的指令来生成他们。
如果命令行中没有目标，NMAKE只生成在MAKEFILE文件中找到的第一个目标文件。

3.如果NMAKE没有找到MAKEFILE文件，NMAKE寻找命令行中的目标并尝试使用推导规则来生成目标。
如果没有指定目标，NMAKE返回一个错误。

然后NMAKE使用如下的顺序给宏赋值（由高到低）：

1.命令参数行中定义的宏
2.在makefile文件中或是包含文件中定义的宏
3.继承的宏
4.定义在TOOLS.INI文件中的宏
5.预定义的宏（比如CC或是RFLAGS）

宏定义首先按优先级的顺序赋值，然后按NMAKE遇见宏的次序给其赋值。
举个例子，在包含文件中定义的宏会覆盖在TOOLS.INI文件中定义的同名宏。
注意makefile文件中宏可以重定义，一个宏的值从其被定义的点开始到其被重定义或是取消定义那刻是有效的。

NMAKE也同样给推导规则赋值，使用下面的优先顺序（由高到底）：

1.在makefile或是包含文件中的推导规则
2.在TOOLS.INI文件中定义的推导规则
3.预定义的推导规则（比如.asm.obj）

您可用命令参数行的选项来修改上面的优先级。

/E选项允许从环境变量继承的宏覆盖makefile文件中定义的宏。
/R选项告诉NMAKE忽略预定义的或是定义在TOOLS.INI文件中宏或是推导规则。

接下来，NMAKE处理任何预处理指令。
如果条件预处理中的表达式在方括号中包含了一个程序的调用，在预处理过程中会执行该程序，并在表达式中使用程序的返回值。
如果有!INCLUDE指令指定了一个文件，NMAKE在处理剩下的makefile文件之前首先处理包含的文件。
预处理过程决定了NMAKE最终读取的makefile文件内容。

现在NMAKE可以更新目标文件了。
如果您在命令参数行中指定目标，NMAKE只更新命令行指定的目标。
如果命令行中没有目标，NMAKE只生成在makefile文件中找到的第一个目标文件。

如果您指定了一个伪目标，NMAKE会永远更新目标。
如果您使用了/A选项，NMAKE也会永远更新目标，即使是文件并没有过期。

NMAKE根据目标文件与其依赖文件之前的时间戳先后顺序来决定是否更新目标。
如果目标的任一一个依赖文件有过最新的修改，目标文件则是过期的了。
如果指定了/B选项，那么目标也是过期的如果任一依赖文件是最新修改或是相同的时间戳。

如果目标的依赖项本身是过期的或是不存在，NMAKE首先更新他们。
如果一个目标并没有显示的指定依赖项，NMAKE会找一个与目标匹配的推导规则。
如果有多个推导规则与目标文件匹配，NMAKE根据.SUFFIXES名单中的优先顺序来决定使用那条推导规则。

NMAKE通常会停止运行makefile文件当一个命令返回一个非零的的退出值。
另外，如果NMAKE不知道目标是否有成功生成，NMAKE删除目标。
如果在命令参数行中使用了/I选项，或是使用.IGNORE或是!CMDSWITCHES或是-减号命令修饰符等，
这些将让NMAKE忽略错误返回值并尝试继续运行。
/K选项告诉NMAKE继续运行生成过程的其他无关部分如果命令执行返回错误。
.PRECIOUS指令阻止NMAKE删除部分创建的目标如果您通过CTRL+C或是CTRL+BREAK中断了NMAKE的运行。
您可以使用!ERROR指令记录错误并打印说明性文本。
!ERROR指令让NMAKE打印一些文本然后停止生成过程。

一份简单的NMAKE的makefile文件
下面的例子演示了很多的NMAKE特性。
该makefile用来编译一个C语言源文件以生成一个可执行程序文件。

# This makefile builds SAMPLE.EXE from SAMPLE.C,
# ONE.C, and TWO.C, then deletes intermediate files.
CFLAGS = /c /AL /Od $(CODEVIEW) # controls compiler options
LFLAGS = /CO # controls linker options
CODEVIEW = /Zi # controls debugging information
OBJS = sample.obj one.obj two.obj
all : sample.exe
sample.exe : $(OBJS)
link $(LFLAGS) @<<sample.lrf
$(OBJS: =+^
)
sample.exe
sample.map;
<<KEEP
sample.obj : sample.c sample.h common.h
CL $(CFLAGS) sample.c
one.obj : one.c one.h common.h
CL $(CFLAGS) one.c
two.obj : two.c two.h common.h
CL $(CFLAGS) two.c
clean :
-del *.obj
-del *.map
-del *.lrf

假设该makefile文件名为SAMPLE.MAK。
相要调用该makefile文件，输入

NMAKE /F SAMPLE.MAK all clean

NMAKE生成SAMPLE.EXE然后删除中间文件。

下面解释这个makefile文件怎么工作的。
宏CFLAGS、CODEVIEW和LFLAGS定义了默认的选项给编译器、链接器和调试信息的包容器。
您可以在命令参数行中重定义这些选项去修改或是删除他们。举个例子：

NMAKE /F SAMPLE.MAK CODEVIEW=CFLAGS=all clean

创建一个并不包含调试信息的可执行文件。

宏OBJS指定了生成SAMPLE.EXE的可重定位目标文件，所以他们可是重新使用而不需要重新录入。
他们的名字用一个空格精准地的隔开，这样空格可以在链接程序的响应文件中被加号+和回车符替换。
（这个有在560页中"宏中的替换“一节的第二个例子中有使用过。）

伪目标ALL指向了真正的目标，sample.exe。
如果您没有在命令参数行中指定其他的目标，NMAKE忽略伪目标clean但仍然生成目标all，因为all是makefile文件中第一个目标。

包含目标sample.exe的依赖行创建sample.exe的依赖文件，那些在宏OBJS中定义的可重定位目标文件。
命令段中只包含链接指令，并没有指定编译指令，因为他们在文件的后面被显式地声明了。
（您也可定义一个推导规则来指定怎样由一个C源文件生成一个object文件。

链接命令是异常的，因为链接参数和选项并没有直接传递给LINK程序。
相反，一个创建的内联文件包含了这些元素。
这减少了维护独立的链接响应文件的需要。

接下来的三个依赖关系定义了源文件和object文件之间的关系。
.H文件（头文件或是包含文件）同样也是依赖项，如果他们有任何修改也会需要重新编译目标。

伪目标clean用来删除生成过程之后的一些不需要的文件。
减号-命令修饰符告诉NMAKE忽略执行删除命令过程中的错误。
如果您想要保存这些文件，请不要在命令参数行中指定clean目标，NMAKE将忽略clean伪目标。

NMAKE的返回值
NMAKE运行结束，返回一个退出值给操作系统或是调用程序。
一个零值的返回值表示NMAKE的执行并没有产生错误。
警告返回零值。

Code                 Meaning
--------------------------------------------------------------------------------
0                    没有错误
1                    没完成的生成过程（只当/K选项使用时才会发布）
2                    程序错误，可能由以下原因造成：
		 makefile语法错误 
		 命令返回错误
		 用户中断
4           系统错误-内存不足
255                   目标不是最新的（只当/Q选项使用时才会发布）
---------------------------------------------------------------------------------