---
title: vim常用按键总结
date: 2017-11-08 17:09:53
author:
    nick: jizx
tags:
- linux
- vim
categories:
- linux
- vim
---

<Excerpt in index | 首页摘要> 
通过对linux中自带的vimtutor命令学习，然后对常用的vim命令进行总结，方便以后忘记了常看
<!-- more -->
<The rest of contents | 余下全文>

```
vim file +n ：打开文件并跳到第N行

默认显示行号：编辑或新建~/.vimrc   添加一行 set nu 即可，或者在/etc/vimrc 内编辑
```

## 命令模式

命令的基本模式 `operation [number] range`

### 定位移动
```shell
        k上
    h左       l右
         j下
    
4h 左4个字符     4k 上4行     4j 下4行     4l 右4个字符
    
2gg                             "定位到第2行代码，等同于2G
2G                              "定位到第2行代码
gg                              "定位到整个代码的第1行
G                               "定位到整个代码的最后一行

0                               "行首
$                               "行尾

e                               "向后跳到该单词结束处
w                               "向后跳一个单词的长度，即调到下一个单词的开始处
b                               "向前跳一个单词的长度，即调到上一个单词的开始处
2e,2w,2b表示移动2个单位

H                               "当前屏幕可见的第一行
M                               "当前屏幕可见的中间
L                               "当前屏幕可见的最后一行
```

### 编辑

```shell
i                               "光标处插入
a                               "光标后一个字符处插入
A                               "光标所在行末尾插入
o                               "光标下方新建一行
O                               "光标上方新建一行

p                               "粘贴
y                               "复制选择的文本
yw或ye                          "复制一个单词
yy                              "复制 光标所在的这一行
4yy                             "复制 光标所在行开始向下的4行

dd                              "剪切 光标所在的这一行
2dd                             "剪切 光标所在行 向下 2行
d0                              "从当前的光标开始剪切，一直到行首
D                               "从当前的光标开始剪切，一直到行末

dw                              "删除光标后的一个单词
d2w：删除光标后2个单词
x                               "删除当前的光标，每次只会删除一个
X                               "删除当前光标前面的那个，每次只会删除一个

r                               "替换光标后的一个字符
R                               "连续替换光标后的字符

ce或cw                          "修改该单词到单词结尾
c$                              "修改光标后该行的全部内容

.                               "重复执行上一次的命令

u                               "撤销最后的操作
U：撤销对整行的修改
ctrl+r                          "反撤销
```

### 翻页

```shell
ctrl+f                          "向下翻一页代码
ctrl+b                          "向上翻一页代码

ctrl+d                          "向下翻半页代码
ctrl+u                          "向上翻半页代码

" 可视化模式，可以配合d,x,y等进行修改，也可以进来末行模式:w filename 进行另存
v                               "字符可视化模式（Characterwise visual mode），文本选择是以字符为单位的，
V                               "行可视化模式（Linewise visual mode)，文本选择是以行为单位的。
ctrl-V                          "块可视化模式（Blockwise visual mode），可以选择一个矩形内的文本。

>>                              "向右移动代码
<<                              "向左移动代码

shift+zz                        "相当于wq
```

### 查找与替换

```shell
/xxx                            "向下查找关键字xxx
/xxx\c                          "忽略大小写
?xxx                            "向上查找关键字xxx
?xxx\c                          "忽略大小写
输入n则继续查找下一个匹配
ctrl+o                          "光标跳转到查询到的上一个位置
ctrl+i                          "光标跳转到查询到的下一个位置

%                               "快速查找光标所在处的括号所匹配的另一个括号 (),[],{}

:s/old/new                      "替换该行第一个匹配串
:s/old/new/g                    "替换该行全部匹配串
:#,#s/old/new/g                 "替换#到"行之间的词
:%s/old/new/gc                  "替换全文匹配串，并逐个询问是否替换
```



## 末行模式

```shell
:w                              "保存
:w filename                     "保存为指定的文件
:q                              "退出
:wq                             "保存并且推出
:x                              "等同于wq
:r filename                     "读取外部文件粘贴到光标处
:r !ls                          "获取命令输出内容粘贴到光标处
```



### 常用设置

比较全面的内容参考全局配置那一节

```shell
:set ic                         "查询关键字时忽略大小写
:set is                         "查询关键字时显示部分匹配
:set hls                        "高亮显示匹配到的关键字‘hlsearch’
:set cp                         "兼容模式
:set noxxx                      "关闭上面对应的设置

:set encoding=utf-8             "设置编码
:set number                     "显示行号
:set ruler                      "右下角显示光标当前位置信息
:set tabstop=4                  "设置缩进宽度
:set shiftwidth=4               "设置缩进宽度
"可以将这些设置偏好设置到 ~/.vimrc文件中
```



## 全局配置

- 单次启用某个配置项，在命令模式下，先输入一个冒号，再输入配置
- 用户个人的配置在`~/.vimrc`

- Vim 的全局配置一般在`/etc/vim/vimrc`或者`/etc/vimrc`，对所有用户生效



### 查看配置状态

查询某个配置项是打开还是关闭，可以在命令模式下，输入该配置，并在后面加上问号`?`。

```
:set nu?
:set number?
```

上面的命令会返回`number`或者`nonumber`

### 打开与关闭某项设置

配置项一般都有"打开"和"关闭"两个设置。"关闭"就是在"打开"前面加上前缀"no"。

```shell
" 打开
set nu
set number

" 关闭
set nonu
set nonumber
```

### 末行命令自动补全

```shell
" 末行命令自动补全
set wildmenu " (跟下面组合使用？没看出效果)
set wildmode=longest:list,full    "命令模式下，底部操作指令按下 Tab 键自动补全。第一次按下 Tab，会显示所有匹配的操作指令的清单；第二次按下 Tab，会依次选择各个指令。
```



### 推荐配置文件内容

http://www.ruanyifeng.com/blog/2018/09/vimrc.html

#### 外观、代码风格

```shell
" 颜色相关

syntax on                       "打开语法高亮，自动识别代码
set hlsearch                    "搜索时，高亮显示匹配结果
set showmatch                   "光标遇到圆括号、方括号、大括号时，自动高亮对应的另一个圆括号、方括号和大括号
set t_Co=256                    "启用256色

" 换行相关
set tabstop=4                   "按下Tab 键时，Vim 显示的空格数
set wrap                        "自动折行，即太长的行分成几行显示。
set linebreak                   "不会在单词内部折行。只有遇到指定的符号（比如空格、连词号和其他标点符号），才发生折行

" 光标位置相关
set scrolloff=5                 "垂直滚动时，光标距离顶部/底部的行数，方便看到后面的内容
set sidescrolloff=15            "水平滚动时，光标距离行首或行尾的位置（单位：字符）。该配置在不折行时比较有用

" 状态栏相关
set ruler                       "在状态栏显示光标的当前位置（位于哪一行哪一列）
set laststatus=2                "是否显示状态栏。0:不显示，1:只在多窗口时显示，2:显示
set showmode                    "在底部显示，当前处于命令模式还是插入模式
set showcmd                     "命令模式下，在底部显示，当前键入的指令。比如，键入的指令是2y3d，那么底部就会显示2y3，当键入d的时候，操作完成，显示消失。

" 辅助显示
set nu
set number                      "显示行号

set list                        "如果行尾有多余的空格（包括 Tab 键），该配置将让这些空格显示成可见的小方块。与listchars配合使用
set listchars=tab:»■,trail:■ ,eol:$


"###"""""""""""""" 没效果或没用的😁
" set cursorline                光标所在的当前行用下划线高亮(没什么用)
" set relativenumber            显示光标所在的当前行的行号，其他行都为相对于该行的相对行号。(没什么用)
" set spell spelllang=en_us     打开英语单词的拼写检查(不推荐，太亮了)
" set textwidth=80              设置行宽，即一行显示多少个字符。(没效果？)
" set wrapmargin=2              指定折行处与编辑窗口的右边缘之间空出的字符数
```



#### 编辑相关

```shell
set mouse=a                     "支持使用鼠标
set encoding=utf-8              "使用 utf-8 编码。

" 缩进相关
set autoindent                  "按下回车键后，下一行的缩进会自动跟上一行的缩进保持一致
set expandtab                   "tab转为空格,而不是\t。
set softtabstop=4               "Tab 转为多少个空格
filetype indent on              "开启文件类型检查，并且载入与该类型对应的缩进规则。比如，如果编辑的是.py文件，Vim 就是会找 Python 的缩进规则~/.vim/indent/python.vim

set history=1000                "Vim 需要记住多少次历史操作。
set nocompatible                "不与 Vi 兼容（采用 Vim 自己的操作命令）。

" 文件备份、历史相关
set nobackup                    "不创建备份文件。默认情况下，文件保存时，会额外创建一个备份文件，它的文件名是在原文件名的末尾，再添加一个波浪号（〜）。
set swapfile                    "创建交换文件。交换文件主要用于系统崩溃时恢复文件，文件名的开头是.、结尾是.swp。
set undofile                    "关闭文件后仍保留撤销历史。Vim 会在编辑时保存操作历史，用来供用户撤消更改。默认情况下，操作记录只在本次编辑时有效，一旦编辑结束、文件关闭，操作历史就消失了。打开这个设置，可以在文件关闭后，操作记录保留在一个文件里面，继续存在。这意味着，重新打开一个文件，可以撤销上一次编辑时的操作。撤消文件是跟原文件保存在一起的隐藏文件，文件名以.un~开头。
set autochdir                   "自动切换工作目录。这主要用在一个 Vim 会话之中打开多个文件的情况，默认的工作目录是打开的第一个文件的目录。该配置可以将工作目录自动切换到，正在编辑的文件的目录。
set backupdir=~/.vim/.backup//  "要新建这个文件夹
set directory=~/.vim/.swp//     "要新建这个文件夹
set undodir=~/.vim/.undo//      "要新建这个文件夹
" 设置备份文件、交换文件、操作历史文件的保存位置。结尾的//表示生成的文件名带有绝对路径，路径中用%替换目录分隔符，这样可以防止文件重名。

set autoread                    "打开文件监视。如果在编辑过程中文件发生外部改变（比如被别的编辑器编辑了），就会发出提示。


" 末行命令自动补全
set wildmenu                    "(跟下面组合使用？没看出效果)
set wildmode=longest:list,full  "命令模式下，底部操作指令按下 Tab 键自动补全。第一次按下 Tab，会显示所有匹配的操作指令的清单；第二次按下 Tab，会依次选择各个指令。

""""""" 暂时不知道有什么用的
set shiftwidth=4                "在文本上按下>>（增加一级缩进）、<<（取消一级缩进）或者==（取消全部缩进）时，每一级的字符数
```



#### 搜索相关

```shell
set incsearch 
set is                          "输入搜索模式时，每输入一个字符，就自动跳到第一个匹配的结果
set ignorecase 
set ic                          "搜索时忽略大小写
set smartcase                   "如果同时打开了ignorecase，那么对于只有一个大写字母的搜索词，将大小写敏感；其他情况都是大小写不敏感。比如，搜索Test时，将不匹配test；搜索test时，将匹配Test
```



### 关于粘贴缩进问题

结论：使用`:set paste `



在Vim中粘贴Python代码后，缩进就全乱了。仔细研究了以下，原来是自动缩进的缘故，于是做如下设置：

```
:set noai nosi 
```

取消了自动缩进和智能缩进，这样粘贴就不会错行了。但在有的vim中不行，还是排版错乱。

后来发现了**更好用**的设置：

```
:set paste 
```

进入paste模式以后，可以在插入模式下粘贴内容，不会有任何变形。这个真是灰常好用，情不自禁看了一下帮助，发现它做了这么多事：

```
textwidth设置为0
wrapmargin设置为0
set noai
set nosi
softtabstop设置为0
revins重置
ruler重置
showmatch重置
formatoptions使用空值

下面的选项值不变，但却被禁用：

lisp
indentexpr
cindent
```

怪不得之前只设置noai和nosi不行，原来与这么多因素有关！

#### 快捷键设置

但这样还是比较麻烦的，每次要粘贴的话，先set paste，然后粘贴，然后再set nopaste。**更方便的方法**是使用键盘映射：

```
:map <F10> :set paste<CR> 
:map <F11> :set nopaste<CR> 
```

这样在粘贴前按F10键启动paste模式，粘贴后按F11取消paste模式即可。其实，paste有一个切换paste开关的选项，这就是pastetoggle。通过它可以绑定快捷键来激活/取消 paste模式。比如（暂时没试成功）

```
:set pastetoggle=<F11> 
```

这样减少了一个快捷键的占用，使用起来也更方便一些。

以上这些设置只是打开这次回话才有效，想要永久有效，需要在`~/.vimrc`中添加上面的命令

```
" 不需要冒号
set pastetoggle=<F11> 
```



