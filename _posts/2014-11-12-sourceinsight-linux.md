---
layout: post
title: vim + ctags + taglist + cscope, linux版本的sourceinsight
tags: vim ctags cscope taglist
categories: 文本编辑
---

vim是一般linux系统的自带工具，无须下载。

sudo apt-get install vim



安装ctags,cscope

sudo apt-get install exuberant-ctags

cscope



安装taglist

taglist配合ctags使用，本身是vim的插件，在vim官网有下载
http://www.vim.org/scripts/script.php?script_id=273


下载后解压，检查是否有~/.vim目录，如果没有就建立一个：mkdir
 ~/.vim

解压出来的2个文件夹doc和plugin拷贝到~/.vim目录下


配置cscope的vim插件

wget http://cscope.sourceforge.net/cscope_maps.vim
将下载的cscope_map.vim文件放在~/.vim/plugin目录中

vim的配置文件

set tabstop=4

set shiftwidth=4
set expandtab
set nu
set hlsearch
set incsearch
set ruler
set softtabstop=4
"set autochdir
set autoread
set autoindent
set smartindent
set ruler
"set list
set showmatch
set paste
syntax on
filetype plugin indent on  


nnoremap <silent> <F8> : TlistToggle<CR>
let Tlist_Use_Right_Window = 1 
let Tlist_Exit_OnlyWindow = 1 
let Tlist_Show_One_File = 1 
let Tlist_File_Fold_Auto_Close = 1 
let Tlist_Display_Prototype = 1 


set tags=tags


function! UpdateCtags()
    let curdir=getcwd()
    while !filereadable("./tags")
        cd ..
        if getcwd() == "/" 
            break
        endif
    endwhile
    if filewritable("./tags")
        !ctags -R --file-scope=yes --langmap=c:+.h --languages=c,c++ --links=yes --c-kinds=+p --c++-kinds=+p --fields=+iaS --extra=+q
        TlistUpdate
    endif
    execute ":cd " . curdir
endfunction


nnoremap <silent> <F11> : call UpdateCtags()<CR>


至此，工具和配置已经完毕，接下来就可以直接使用了。

将以下脚本添加到~/.bashrc文件

ctags_cscope_func() {
    ctags -R --file-scope=yes --langmap=c:+.h --languages=c,c++ --links=yes --c-kinds=+p --c++-kinds=+p --fields=+iaS --extra=+q
    find . -iname '*.c' -o -iname '*.cpp' -o -iname '*.h' -o -iname '*.hpp' -o -iname "*.cc" > cscope.files
    cscope -Rbkq -i cscope.files
}
alias ccsi='ctags_cscope_func'
添加完毕，执行source ~/.bashrc，这样我们就可以使用ccsi命令来创建ctags和cscope的数据库了。


进入你想建立符号库的工程的源码根目录，执行ccsi命令

errong.leng@VDBS1444:~/google_blink/src$ ccsi 

命令执行完毕，应该会生成如下几个文件，表示创建成功了。

cscope.files  cscope.in.out  cscope.out  cscope.po.out tags


接下来就可以像使用sourceinsight一样的，用VIM来编辑你的代码了。

打开VIM，按下F8，可以打开tag list，按下F11可以更新符号库。

注意事项：务必要在执行ccsi命令的地方，即你的工程的源码的根目录使用VIM。


How to use ctags

vi −t tag
Start vi and position the cursor at the file and line where "tag" is defined.
:ta tag
Find a tag.
Ctrl-]
Find the tag under the cursor.
Ctrl-T
Return to previous location before jump to tag (not widely implemented).
more details please refer to http://ctags.sourceforge.net/ctags.html


How to use taglist:

   " <enter> : Jump to tag definition                                                                                                                                              
   " o : Jump to tag definition in new window
   " p : Preview the tag definition
   " <space> : Display tag prototype
   " u : Update tag list
   " s : Select sort field
   " d : Remove file from taglist
   " x : Zoom-out/Zoom-in taglist window
   " + : Open a fold
   " - : Close a fold
   " * : Open all folds
   " = : Close all folds
   " [[ : Move to the start of previous file
   " ]] : Move to the start of next file
   " q : Close the taglist window
   " <F1> : Remove help text
more details please refer to http://vim-taglist.sourceforge.net/manual.html


How to use cscope:

打开VIM，输入:cs f c|d|e|f|g|i|s|t

cscope commands:
add  : Add a new database             (Usage: add file|dir [pre-path] [flags])
find : Query for a pattern            (Usage: find c|d|e|f|g|i|s|t name)
       c: Find functions calling this function
       d: Find functions called by this function
       e: Find this egrep pattern
       f: Find this file
       g: Find this definition
       i: Find files #including this file
       s: Find this C symbol
       t: Find this text string
help : Show this message              (Usage: help)
kill : Kill a connection              (Usage: kill #)
reset: Reinit all connections         (Usage: reset)
show : Show connections               (Usage: show)
more details please refer to http://cscope.sourceforge.net/cscope_vim_tutorial.html