---
layout: post
title: NMAKE Guide (六)
tags: NMAKE
categories: build
---

推导规则
推导规则是种模板，该模板定义了如何由一种后缀文件生成另一种后缀文件。
NMAKE使用推导规则为更新目标提供命令和为目标推导规则。
在依赖树中，推导规则为目标提供推导的依赖项就像在依赖行中显示指定的依赖一样，请查阅569页的"推导依赖项“一节。

.SUFFIXES名单决定了推导规则应用时的优先级，请查看570页的”点指令“一节。

推导规则提为常用操作提供了一种便捷的方式。
举个例子，可以用推导规则来避免在多个描述块中重复写同样的命令。
您也可以定义自己的推导规则或是使用预定义的推导规则。
推导规则可以在makefile文件中或是TOOLS.INI文件中指定。

推导规则可以用在以下的场景中：

如果NMAKE遇到没有命令的描述块，NMAKE将检查.SUFFIXES名单和当前或是指定目录下的文件，
然后搜索一条和目标文件后缀匹配的推导规则和在.SUFFIXES名单中有最高优先级的已经存在的依赖项文件。

如果一个依赖文件不存在但并没有作为目标文件列在其他描述块中，NMAKE寻找一条演示了如何由另一个同名的文件创建缺失的依赖项的推导规则。

如果一个目标没有依赖项且它的描述块并没有命令，NMAKE可以使用一条推导规则来创建目标。

如果一个在命令行参数列表中列出的目标且没有指定makefile文件（或是在makefile文件中没有提及该目标文件），
NMAKE将使用推导规则来创建目标文件。

如果一个目标在多个单分号的依赖关系中使用，推导规则或许并不如预期的一样适用，详情请参阅539页的”依赖关系中目标的积累“一节。

推导规则语法
使用如下的语法来定义一条推导规则：

.fromext.toext:
commands

第一行列出两个后缀：fromext代表了依赖文件的后缀；toext代表了目标文件的后缀。
后缀并不区分大小写。可以调用宏来代表fromext和toext，宏在处理时展开。

fromext前的.号必须出现在行的开始位置。冒号前可以放置一个或多个空格或是制表符，它后面只能有空格或是制表符。
；分号指定一条命令，#描述了一条注释或是换行符。不允许有其他的空格。

推导规则只能在目标和依赖有着相同的文件名时适用。
一条推导规则不可能适应多种目标或是依赖文件。
举个例子，您不能定义一个推导规则来替代一个库文件中的多个模块，因为从一个目标库在出来的所有模块必须有且不同的文件名。

推导规则只适用于拥有在.SUFFIXES名单中后缀的依赖文件。
（更多信息关于.SUFFIXES，请查看570页的”点指令“一节。）
如果一个过时的依赖并没有命令块，如果同时在.SUFFIXES名单中包含有该依赖项的后缀，
那么NMAKE寻找一条和目标后缀匹配的推导规则和当前或指定的目录中存在的文件来生成目标。
如果多个推导规则和存在的依赖文件匹配，NMAKE使用.SUFFIXES名单中后缀的顺序来决定使用那条推导规则。
名单中的先后顺序由左到右。NMAKE也许会调用一条推导规则即使有显示地的指定依赖项，更多的信息请参阅569页的”推导规则"一节。

推导规则告知NMAKE如何去生成在命令参数行中指定的目标，如果没有提供makefile文件或是在makefile文件中并没有关于指定目标的依赖关系。
如果NMAKE不知道如何去生成命令中指定的目标，NMAKE会找寻一条推导规则来生成它。
如果您在预定义的推导规则和在TOOLS.INI定义的推导规则足够满足您的目标生成需要，那么您可以运行NMAKE程序而无须指定makefile文件。

推导规则搜索路径
先前描述的推导规则定义语法告诉NMAKE在当前目录寻找指定的文件来生成目标。
当然您也可以告知NMAKE搜索目录。指定搜索路径的推导规则的语法如下 ：

{frompath}.fromext{topath}.toext:
commands

中间不允许有空格。frompath目录必须和指定的依赖文件的目录匹配；同样的，topath目录必须和目标文件目录匹配。
为让NMAKE为一依赖关系应用一条推导规则，在依赖行中指定的目录必须和推导规则中指定的目录精确的一致。
举个例子，如果当前目录为PROJ，那么推导规则

{../proj}.exe{../proj}.obj:

并不应用于下面的依赖关系

project1.exe : project1.obj

如果您在推导规则中为一个后缀使用了目录，您必须为所有的后缀都使用目录。
您可以用.或是一对空的大括号来表示当前目录。

您在推导规则中只能为每种后缀指定一条路径。
如果需要指定多个路径，您必须为每个路径实现独立的推导规则。

宏调用可以用来代表frompath和path，宏在处理里展开。

用户自定义推导规则
下面的例子演示了写推导规则的几种方式：

例子1
下面的makefile文件包含了一条推导规则和一个最小的描述块：

.c.obj:
cl /c $<

sample.obj

推导规则告诉NMAKE如果由a.c文件来生成a.obj文件。
预定义宏$<代表比目标文件有更新时间戳的依赖文件。
描述块只列出了目标SAMPLE.OBJ文件，但并没有依赖项或是命令。
但是，根据目标文件名和后缀再加上推导规则，NMAKE有足够的信息来生成目标。

在检查到.c后缀有列在.SUFFIXES名单中，NMAKE寻找一个与目标同名的以.c为后缀的文件来生成目标。
如果SAMPLE.C文件存在（没有更高的优先级的后缀的同名文件的存在），NMAKE比较该文件和sample.obj文件的时间戳的先后。
如果SAMPLE.C文件有新的修改，NMAKE编译sample.c通过列在推导规则中的CL命令来生成sample.obj文件：cl /c sample.c

例子2
下面的推导规则当前目录下的.c文件和在其他目录下的对应的.obj文件：

{.}.c{c:/objects}.obj:
cl /c $<;

为.c文件指定的路径是.，代表着当前目录。
依赖文件的后缀也必须指定搜索路径，因为目标的后缀指定了搜索路径。

这条推导规则适用于包含同样的路径的依赖关系，就像下面的：

c:/objects/test.obj : test.c

那条规则没不适用于下面的依赖关系:

test.obj : test.c 

例子3
下面的推导规则调用宏来指定推导规则中的路径：

C_DIR = proj1src
OBJ_DIR = proj1obj
{$(C_DIR)}.c{$(OBJ_DIR)}.obj:
cl /c $

如果宏重新定义了，NMAKE使用在处理时有效的那个定义。
要重用一条推导规则用另一个宏定义，您必须重复推导规则使用新的定义：

C_DIR = proj1src
OBJ_DIR = proj1obj
{$(C_DIR)}.c{$(OBJ_DIR)}.obj:
cl /c $<
C_DIR = proj2src
OBJ_DIR = proj2obj
{$(C_DIR)}.c{$(OBJ_DIR)}.obj:
cl /c $<

预定义推导规则
NMAKE提供了预定义推导规则包含命令来创建对象、可执行文件和资源文件。
表16.1描述了预定义推导规则：

Table 16.1 Predefined Inference Rules
------------------------------------------------------------
Rule     Command            Default Action
------------------------------------------------------------
.asm.exe  $(AS)$(AFLAGS)$*.asm             ML $*.ASM
.asm.obj  $(AS)$(AFLAGS)/c $*.asm          ML /c $*.ASM
.c.exe    $(CC)$(CFLAGS)$*.c            CL $*.C
.c.obj    $(CC)$(CFLAGS)/c$*.c            CL /c $*.C
.cpp.exe  $(CPP)$(CPPFLAGS)$*.cpp         CL $*.CPP
.cpp.obj  $(CPP)$(CPPFLAGS)/c$*.cpp        CL /c $*.CPP
.cxx.exe  $(CXX)$(CXXFLAGS)$*.cxx          CL $*.CXX
.cxx.obj  $(CXX)$(CXXFLAGS)/c$*.cxx        CL /c $*.CXX
.bas.obj  $(BC)$(BFLAGS)$*.bas;            BC $*.BAS;
.cbl.exe  $(COBOL)$(COBFLAGS)$*.cbl,$*.exe;   COBOL $*.CBL,$*.EXE;
.cbl.obj  $(COBOL)$(COBFLAGS)$*.cbl;       COBOL $*.CBL;
.for.exe  $(FOR)$(FFLAGS)$*.for          FL $*.FOR
.for.obj  $(FOR)/c$(FFLAGS)$*.for         FL /c $*.FOR
.pas.exe  $(PASCAL)$(PFLAGS)$*.pas         PL $*.PAS
.pas.obj  $(PASCAL)/c$(PFLAGS) $*.pas       PL /c $*.PAS
.rc.res   $(RC)$(RFLAGS)/r$*            RC /r $*
------------------------------------------------------------------------

举个例子，假设您有下面的makefile文件：

sample.exe : 

上面的描述块列出了一个目标但并没有依赖项和命令。
NMAKE查看目标文件的后缀（.exe）然后搜索一条描述如何生成一个.exe文件推导规则。
由表16.1中可以看出，有多条推导规则可以生成.exe文件。
NMAKE根据在.SUFFIXES名单中的后缀名顺序来决定使用那条推导规则来生成目标文件。
然后NMAKE在当前目录或是指定目录在搜索与目标同名的且具有.SUFFIXES名单中的后缀名的文件，
它依个检查.SUFFIXEX名单中后缀名直到在目录中找到一个匹配的依赖文件。

举个例子，如果一个名为sample.asm的文件存在，NMAKE使用.asm.exe推导规则。
如果sample.c和sample.asm同时存在，如果在.SUFFIXES名单中.c后缀出现在.asm前，
那么NMAKE使用.c.exe推导规则来编译SAMPLE.C然后链接生成文件sample.obj文件来生成SAMPLE.EXE。

-----------------------------------------------------------------------------------------------------------
注意：默认情况下，选项宏（AFLAGS,CFLAGS，等等）是没有定义的。
按照在554页中“宏的使用”一节中的解释，这并不会引起任何问题。
NMAKE使用空串来替换一个未定义的宏。
由于这些预定义宏有在推导规则中使用，您可以重定义这些宏然后将给其分配的值自动传递到这些预定义的推导规则中。
-----------------------------------------------------------------------------------------------------------

推导的依赖项
NMAKE可以为目标假想一些推导的依赖项如果存在一条适当的推导规则。
一条适当的推导规则符合如下条件：

推导规则中toext后缀名和检测的目标文件的后缀一致。

推导规则中fromext后缀名和当前或是指定目录下的一个与目标文件同名的文件的后缀名一样。

fromext后缀名有列出在.SUFFIXES名单中。

在匹配的推导规则中没有其他的fromext后缀名列出在.SUFFIXES名单中具有更高的优先级。

没有显示地指定更高优先级后缀的依赖项。

如果一个存在的依赖项和一个推导规则匹配并且有一个后缀名拥有更高的.SUFFIXES优先级，
那么NMAKE并不推导出一个依赖。

NMAKE并不需要执行推导规则中的命令块为一个推导的依赖项。
如果一个目标的描述块中包含有命令，NMAKE执行描述块中的命令而不是匹配的推导规则的命令块。
下面的例子演示了推导的依赖项的作用:

project.obj : 
cl /Zi /c project.c

如果一个makefile文件包含上面的描述块，并且当前目录中包含一个名为PROJECT.C的文件，没有其他的文件，
那么NMAKE使用预定义推导规则.c.obj推导出依赖项project.c。
NMAKE并不执行预定义推导规则中的命令，而是执行在makefile文件中指定的命令：cl /c project.c。

推导的依赖项可能会引起预料之外的影响。
在下面的例子中，假设PROJECT.ASM和PROJECT.C同时存在同时.SUFFIXES包含默认的设置。
如果makefile包含下面的描述块：

project.obj : project.c

NMAKE推导的依赖项project.asm优先于project.c，因为在.SUFFIXES中.asm中.c之前且存在.asm.obj的推导规则。
如果project.asm或是project.c过期了，NMAKE将执行推导规则.asm.obj中命令。

但是，之前例子中的依赖关系中添加命令块，NMAKE执行命令块中的命令，而不是提供推导的依赖项的推导规则中的命令。

另一副作用产生了，因为NMAKE生成目标如果任一一个依赖文件过期了，不管是显示指定的还是推导的依赖项。
举个例子，如果PROJECT.OBJ相对于PROJECT.C文件是最新的，但是对于PROJECT.ASM却是过期的 ，如果makefile包含：

project.obj : project.c
cl /Zi /c project.c

NMAKE推导出依赖项为PROJECT.ASM，于是乎使用描述块的指定的命令来更新目标文件。

推导规则的优先级
如果同一个推导规则在多处定义，NMAKE使用具有最高优先级的推导规则。
优先级从高到低如下：

1.makefile文件中定义的推导的规则。如果定义了多个，那么应用最后的那一个。

2.在TOOLS.INI文件中定义的推导规则。如果定义了多个，那么应用最后一个。

3.预定义的推导规则。

用户自定义了推导规则总是会覆盖预定义的推导规则。
NMAKE使用预定义推导规则如果对于给定的目标和依赖并没有存在用户自定义的推导规则。

如果两个推导规则匹配一个目标文件的后缀且没有指定依赖，NMAKE使用推导规则中的那个依赖项的后缀出现在.SUFFIXES名单中较前的那个。