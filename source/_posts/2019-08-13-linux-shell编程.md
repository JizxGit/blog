---
title: linux shell编程

tags:
  - shell
  - linux
categories:
  - linux
date: 2019-08-13 11:00:08
---

<Excerpt in index | 首页摘要> 

<!-- more -->
<The rest of contents | 余下全文>

##  基本知识

### Shell 进程

#### 父子进程

在CLI提示符后输入/bin/bash命令或其他等效的bash命令时，会创建一个新的shell程序。 这个shell程序被称为子shell（child shell）。

在生成子shell进程时，只有部分父进程的环境被复制到子shell环境中。这会对包括变量在内的一些东西造成影响。



#### 进程列表

可以在一行内指定依次运行的一系列指令，通过**命令列表**来实现：

```bash
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls
```

命令列表要想成为**进程列表**，这些命令必须包含在括号里。括号的加入使命令列表变成了进程列表，生成了一个子shell来执行对应的命令。

>   进程列表是一种命令分组（command grouping） 。另一种命令分组是将命令放入花括号中， 并在命令列表尾部加上分号（;）。语法为 `{ command; }`但是不会像进程列表那样创建出子shell。

要想知道是否生成了子shell，得借助一个使用了环境变量的命令`echo $BASH_SUBSHELL`。如果该命令返回0，就表明没有子shell。如果返回 1或者其他更大的数字，就表明存在子shell。

```bash
$ pwd ;  echo $BASH_SUBSHELL
/Users/admin
0

$ { pwd ; echo $BASH_SUBSHELL; }
/Users/admin
0

$ (pwd ; echo $BASH_SUBSHELL)
/Users/admin
1

$ ( pwd ; (echo $BASH_SUBSHELL))
/Users/admin
2
```

>   在shell脚本中，经常使用子shell进行多进程处理。但是采用子shell的成本不菲，会明显拖慢处理速度。
>
>   在交互式的CLI shell会话中，子shell同样存在问题。它并**非真正的多进程**处理，因为终端控制着子shell的I/O。
>
>   在CLI中运用子shell的创造性方法之一就是将进程列表置入**后台模式**。你既可以在子shell中 进行繁重的处理工作，同时也不会让子shell的I/O受制于终端。



#### 高效子进程

##### 后台模式

```
$ sleep 3000& 
[1] 2396
```

在命令末尾加上字符`&`，在shell CLI提示符返回之前，会出现一条信息，代表后台作业（background job）号（1）与后台作业的进程ID（2396）。

可以通过`ps -f` 或者 `jobs -l` 来查看



##### 协程

协程 协程可以同时做两件事。它在后台生成一个子shell，并在这个子shell中执行命令。 要进行协程处理，得使用coproc命令，还有要在子shell中执行的命令。

```
coproc sleep 10
coproc My_Job { sleep 10; } # 指定协程名
```

>   通过使用扩展语法，协程的名字被设置成My_Job。这里要注意的是，扩展语法写起来有点 麻烦。必须确保在第一个花括号`{`和命令名之间有一个空格。还必须保证命令以分号`;`结 尾。另外，分号和闭花括号`}`之间也得有一个空格。



### shell 内建命令

#### 外部命令

外部命令，是存在于bash shell之外的程序。外部命令程序通常位于`/bin`、`/usr/bin`、`/sbin`或`/usr/sbin`中。

当外部命令执行时，会创建出一个子进程。这种操作被称为**衍生**（forking）。

当进程必须执行衍生操作时，它需**要花费时间和精力来设置新子进程的环境**。所以说，外部命令多少还是有代价的。

#### 内建命令

内建命令和外部命令的区别在于前者**不需要使用子进程来执行**。它们已经和shell编译成了一体，作为shell工具的组成部分存在。比如`cd`和`exit`

利用`type`命令来了解某个命令是否是内建的

```
$ type cd
cd is a shell builtin
```

有些命令有多种实现。例如`echo`和`pwd`既有内建命令也有外部命令，要查看命令的不同实现，使用type命令的-a选项。

```ba&#39;sh
$ type -a echo 
echo is a shell builtin 
echo is /bin/echo
```

要使用外部命令 `pwd` ，可以输入`/bin/pwd` 



##### 内建命令之 history

设置保存在bash历史记录中的命令数。要想实现这一点，你需要修改名为 `HISTSIZE` 的环境变量

输入`!!`可以重新执行上一条命令，`!编号`即可执行历史列表中的对应编号的命令

bash命令的历史记录是先存放在内存中，当shell退出时才被写入到历史文件`~/.bash_history`中。

`~/.bash_history`文件只有在打开首个终端会话时才会被读取。

可以在退出shell会话之前强制将命令历史记录写入`.bash_history`文件。要实现强制写入，需要使用`history -a`

要想强制重新读 取.bash_history文件，更新终端会话的历史记录，可以使用 `history -n` 命令。

##### 内建命令之 alias

要查看当前可用 的别名，使用`alias`或者`alias -p`。

使用alias命令创建属于自己的别名。

`alias li='ls -li'`



### shell 配置文件(TODO)

http://ddrv.cn/a/173848/

`Shell` 启动方式（TODO 到底是几种）

-   交互式登录
-   交互式非登录
-   非交互式登录
-   非交互式非登录



启动bash shell有3种方式：

-   登录时作为默认登录shell
-   作为非登录shell的交互式shell 
-   作为运行脚本的非交互shell

>   `交互式`：一个个地输入命令并及时查看它们的输出结果，整个过程都在跟 Shell 不停地互动。
>   `非交互式`：运行一个 `Shell 脚本` 文件，让所有命令批量化、一次性地执行。
>   `登录式`：需要输入用户名和密码才能使用。
>   `非登录式`：直接可以使用。



#### 判断 shell 类型

**如何判断是否为交互式 Shell? 有两种方式**

1.  查看特殊变量 `-` ，如果值包含 `i`，则是交互式，否则是非交互式

```
echo $-
```

1.  查看变量 `PS1` 是否为空，如果不为空，则是交互式，否则为非交互式

```
echo $PS1
```

**如何判断是否为登录式 Shell ?**
执行命令 `shopt login_shell`，如果 `login_shell` 的值为 `on`表示登录式，为 `off`表示非登录式。



##### 登录 shell

当你登录Linux系统时，bash shell会作为登录shell启动。（对于没有图形化界面来说）

登录shell会从5个不同的启动文件里 读取命令：

-   `/etc/profile` ——>会去读取`/etc/profile.d`目录下的配置文件
-   `$HOME/.bash_profile `
-   `$HOME/.bashrc `
-   `$HOME/.bash_login `
-   `$HOME/.profile`

`$HOME`目录下的启动文件 

剩下的启动文件都起着同一个作用：提供一个用户专属的启动文件来定义该用户所用到的环 境变量。大多数Linux发行版只用这四个启动文件中的1、2个：

-   `$HOME/.bash_profile`
-   `$HOME/.bashrc`
-   `$HOME/.bash_login`
-   `$HOME/.profile`

shell会按照按照下列顺序，运行第一个被找到的文件，忽略其他文件：

-   `$HOME/.bash_profile`
-   `$HOME/.bash_login`
-   `$HOME/.profile`

这个列表中没有`$HOME/.bashrc`文件是因为该文件**通常通过其他文件运行**的。比如 `.bash_profile`会先去检查HOME目录中是不是还有一个叫`.bashrc`的启动文件。如果有的话，会先执行启动文件里面的命令。因此`.bashrc`顺序最先，但是并不是优先级最高，因为`.bash_profile`设置的变量会覆盖 `.bashrc` 中的变量

```bash
# .bash_profile 
# Get the aliases and functions 
if [ -f ~/.bashrc ]; then 
	. ~/.bashrc 
fi 
# User specific environment and startup programs 
PATH=$PATH:$HOME/bin 
export PATH 
$
```



##### 交互式 shell 进程

如果你的bash shell不是登录系统时启动的（比如是在命令行提示符下敲入bash时启动），那 么你启动的shell叫作交互式shell。

作为交互式shell启动的，就**不会访问`/etc/profile`文件**，只会检查用户HOME目录中的`.bashrc`文件。

```bash
# .bashrc # Source global definitions 
if [ -f /etc/bashrc ]; then 
	. /etc/bashrc 
fi

# User specific aliases and functions
```

`.bashrc`文件有两个作用：

1.  查看/etc目录下通用的bashrc文件
2.  为用户提供一个定制自 己的命令别名和私有脚本函数的地方。



##### 非交互式 shell

系统执行shell脚本时用的就是这种shell。不同的地方在于它没有命令行提示符。

>   脚本能以不同的方式执行。只有其中的某一些方式能够启动子shell。

bash shell提供了BASH_ENV环境变量。当shell启动一个非交互式shell进 程时，它会检查BASH_ENV来查看要执行的启动文件。如果有指定的文件，shell会执行该文件里的命令，这通常包括shell脚本变量设置。

但是 CentoS 与 Ubuntu 都没有该变量，shell脚本到哪里去获得它们的环境变量呢？

-   有些 shell脚本是通过启动一个子shell来执行的。子shell可以继承父shell导出过的变量。
-   对于那些不启动子shell的脚本， 变量已经存在于当前shell中了。 所以就算没有设置 BASH_ENV，也可以使用当前shell的局部变量和全局变量。



#### 配置文件加载顺序

[参考网站](https://blog.csdn.net/bjnihao/article/details/51775854)

正常启动

![](./shell配置文件加载顺序1.png)

su 切换用户

![](./shell配置文件加载顺序2.png)



###  环境变量

全局环境变量对于shell会话和所有生成的子shell都是可见的。局部变量则只对创建它们的 shell可见。

查看环境变量的命令有`set`、`env`、`printenv`

>   它们的区别：
>
>   set 命令会显示出全局变量、局部变量以 及用户定义变量。它还会按照字母顺序对结果进行排序。
>
>   env 和 printenv 命令不会对变量排序，也不会输出局部变量和用户定义变量。
>
>   TODO env 与 printenv

要显示个别环境变量的值，可以使用`printenv` 或者 `echo`

```bash
$ printenv HOME 
/home/Christine

$ echo $HOME 
/home/Christine
```



#### 设置局部变量

变量名区分大小写。所有的环境变量名均使用大写字母，自己创建的局部变量或是shell脚本，请使用小写字母。

记住，变量名、等号和值之间**没有空格**：如果在赋值表达式中加上了空格， bash shell就会把值当成一个单独的命令：

```bash
$ my_variable = "Hello World" 
-bash: my_variable: command not found
```



#### 设置全局变量

创建全局环境变量的方法是先创建一个局部环境变量，然后通过`export`命令把它导出到全局环境中。变量名前面不需要加$。

```bash
$ my_variable="I am Global now" 
$ my_variable2="I am Global now" 
$ export my_variable my_variable2  # 可以同时导出多个变量
```



修改*子shell*中全局环境变量并**不会影响到父shell中该变量的值**。这种改变仅在子shell中有效，并不会被反映到父shell中。甚至无法使用export命令改变父shell中全局环境变量的值。

```bash
$ my_variable="I am Global now" 
$ export my_variable 
$ 
$ echo $my_variable I am Global now 
$ 
$bash 
$ 
$ echo $my_variable 
I am Global now 
$ 
$ my_variable="Null" 
$ 
$ export my_variable  # 导出变量也没用
$ 
$ echo $my_variable
Null 
$ 
$ exit 
exit 
$ 
$ echo $my_variable 
I am Global now 
$
```



## shell 编程

###  创建脚本

在创建shell脚本文件时，必须在文件的第一行指定要使用的shell。其格式为：

`#!/bin/bash`

在通常的shell脚本中，井号（#）用作注释行。然而， shell脚本文件的第一行是个例外。



###  打印

默认情况下，不需要使用引号将要显示的文本字符串划定出来

可用单引号或双引号来划定文本字符串。如果在字符串中用到了它们，你需要在 文本中使用其中一种引号，而用另外一种来将字符串划定起来。

不换行`echo -n "The time and date are: "`

>   反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符串。
>
>   单引号（''）：全局转义，转义其中所有的变量为单纯的字符串。
>
>   双引号（""）：保留其中的变量属性，不进行转义处理。
>
>   反引号（``）：把其中的命令执行后返回结果，等价于$(命令)

### 别名

通常别名定义在 **$HOME/.bashrc** 或者 **$HOME/bash_aliases** (在 **$HOME/.bashrc**被加载).

```shell
if [ -e $HOME/.bash_aliases ]; then
    source $HOME/.bash_aliases
fi
```

推荐的别名

```shell
alias ls='ls -F'   # 目录名后面加上/
alias ll='ls -lh'  # 人类可读的方式显示容量，KB、GB
alias gh='history|grep' # 查找历史命令
alias count='find . -type f | wc -l' # 计算当前目录下文件总数

# 自定义函数最好保存在 bash_functions 文件中
if [ -e $HOME/.bash_functions ]; then
    source $HOME/.bash_functions
fi
# 切换目录同时展示目录下的内容
function cl() {
    DIR="$*";
        # if no DIR given, go current dir
        if [ $# -lt 1 ]; then
                DIR=".";
    fi;
    builtin cd "${DIR}" && \
    # use your preferred ls command
        ls -F --color=auto
}
```



###  变量

#### 环境变量

在 shell 中`set`命令来显示一份完整的当前环境变量列表

在脚本中，你可以在环境变量名称之前加上美元符`$`来使用这些环境变量，或者`${variable}` 形式引用的变量。变量名两侧额外的花括号通常用来帮助识别美元符后的变量名

#### 用户变量

-   由字母、数字或下划线组成的文本字符串，长度不超过20个

-   用户变量 区分大小写
-   使用等号将值赋给用户变量。在变量、等号和值之间**不能出现空格**
-   在shell脚本结束时会被删除掉
-   用户变量可通过美元符引用

#### 只读变量

使用 **readonly** 下面的例子尝试更改只读变量，结果报错：

```
#!/bin/bash
myUrl="http://c.biancheng.net/shell/"
readonly myUrl
```



#### 字符串

##### 字符串的3种形式

1.  由单引号`' '`包围的字符串：
    任何字符都会原样输出，在其中使用**变量是无效的**。
    字符串中**不能出现单引号**，即使对单引号进行转义也不行。

2.  由双引号`" "`包围的字符串：
    如果其中包含了某个变量，那么该**变量会被解析**（得到该变量的值），而不是原样输出。
    字符串中**可以出现双引号**，只要它被转义了就行。

3. 不被引号包围的字符串
不被引号包围的字符串中出现变量时也会被解析，这一点和双引号" "包围的字符串一样。
字符串中**不能出现空格**，否则空格后边的字符串会作为其他变量或者命令解析。

```shell
n=74
str1=c.biancheng.net$n 
str2="shell \"script\" $n"
str3='C语言中文网 $n'

echo $str1
echo $str2
echo $str3

运行结果：
c.biancheng.net74
shell "script" 74
C语言中文网 $n
```



##### 变量(字符串)拼接

直接将两个变量写在一起就是拼接

```bash
str1=$name$url      #中间不能有空格，遇到空格就认为字符串结束了，空格后边的内容会作为其他变量或者命令解析，
str2="$name $url"   #如果被双引号包围，那么中间可以有空格
str3=$name": "$url  #中间可以出现别的字符串
str4="$name: $url"  #这样写也可以
str5="${name}Script: ${url}index.html"  #这个时候需要给变量名加上大括号 加{ }是为了帮助解释器识别变量的边界
```

##### 字符串的长度

```shell
${#string_name}
```

##### 字符串的截取 

**1.`#`：截取右边字符，删除左边字符**

变量 `var=http://www.aaa.com/123.htm`

-   单`#`：`echo ${var#*//}`，结果：保留 `www.aaa.com/123.htm`

    其中 var 是变量名，`#`是运算符，`*`是通配符，`*//` 表示从左边开始删除第一个 `//` 号及左其边的所有字符，即删除`http://`，保留 `www.aaa.com/123.htm`

- 双`##`（贪心地删除）：`echo ${var##*/}`，结果：保留 `123.htm`

    `##*/`表示从左边开始删除最后一个`/`号及左边的所有字符，即删除 `http://www.aaa.com/`，保留 `123.htm`



**2. `%`：截取左边字符，删除右边字符**

变量 `var=http://www.aaa.com/123.htm`

-   单`%`：`echo ${var%/*}`，结果：保留`http://www.aaa.com`

     `%/*` 表示从右边开始，删除第一个`/`号及右边的字符，保留`http://www.aaa.com`

*   双`%%`（贪心地删除）:`echo ${var%%/*}`，结果：保留`http:`

     `%%/*` 表示从右边开始，删除最后（最左边）一个`/`号及右边的字符，保留`http:`



#### 删除变量

记住不要使用`$`

```shell
unset my_variable
```

在处理**全局环境变量**时，如果你是在子进程中删除了一个全局环境变量， 这只对子进程有效。该全局环境变量在**父进程中依然可用**。

#### 变量默认值

参数设置默认值${var:-default}

`var=${var1:-default}` 当var1不存在时，使用default作为默认值

```bash
serverparm=$1
servername=${serverparm:-"/home/test/tomcat-server1"}
servernum=${serverparm:-34}   # 默认值是34哦
echo $servername
echo $servernum
```



#### 变量前的 $ 符号

记住一点就行了：如果要用到变量，使用`$`；如果要操作变量，不使用`$`。这条规则的一个例外就是使用 `printenv` 显示某个变量的值。



#### `$*` 与`$@`

`$*` 和`$@` 都表示传递给函数或脚本的所有参数， 当 `$*` 和 `$@` 不被**双引号`""`**包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。

但是当它们被双引号`" "`包含时，就会**有区别**了：

-   `"$*"`会将所有的参数从**整体上看做一份数据**，而不是把每个参数都看做一份数据。
-   `"$@"`仍然将每个参数都看作一份数据，彼此之间是独立的。

比如传递了 5 个参数，那么对于`$*`来说，这 5 个参数会合并到一起形成一份数据，它们之间是无法分割的；而对于`$@`来说，这 5 个参数是相互独立的，它们是 5 份数据。

```shell
echo "脚本的名字是："$0
n=1
echo "使用\$@的参数列表为："$@
for temstr in "$@"
do
  echo "第$n个参数是：" $temstr
  let n+=1
done

n=1
echo "使用\$*的参数列表为："$*
for temstr in "$*"
do
  echo "第$n个参数是：" $temstr
  let n+=1
done

##### 结果 #####

脚本的名字是：test.sh
使用$@的参数列表为：1 2 3 4 5
第1个参数是： 1
第2个参数是： 2
第3个参数是： 3
第4个参数是： 4
第5个参数是： 5
使用$*的参数列表为：1 2 3 4 5
第1个参数是： 1 2 3 4 5
```



#### `$0`的含义

第一种情况：直接命令调用一个shell，比如bash，会打开一个新的bash子shell，这时`echo $0`显示 shell的名称，比如`sh`，或者`bash`

```
# bash
# echo '$0' is $0
# $0 is bash
```

第二种情况：shell 调用脚本文件，那么在脚本文件中`echo $0`就是脚本的文件名

```
# bash  main.sh
# $0 is main.sh
```



#### 环境变量持久化

对全局环境变量来说**不要将变量、设置放在`/etc/profile`**文件中，因为在你升级了所用的发行版后， 这个文件也会跟着更新，那所有定制过的变量设置可就都没有了。

**最好**是在`/etc/profile.d`目录中创建一个以`.sh`结尾的文件。把所有新的或修改过的全局环境变 量设置放在这个文件中。

在大多数发行版中，存储个人用户永久性bash shell变量的地方是`~/.bashrc`文件。这一点适用于所有类型的shell进程。 但如果设置了 BASH_ENV 变量， 那么记住， 除非它指向的是`~/.bashrc`，否则你应该将非交互式shell的用户变量放在别的地方。



#### 命令输出赋给变量、命令替换

-   反引号字符（`）
-   `$()`格式

>    原理：命令替换会创建一个子shell来运行对应的命令。子shell（ subshell）是由运行该脚本的shell 所创建出来的一个独立的子shell（child shell） 。正因如此，由该子shell所执行命令是无法 使用脚本中所创建的变量的。



TODO 没明白

在命令行提示符下使用路径 ./ 运行命令的话，也会创建出子shell；要是运行命令的时候 不加入路径，就不会创建子shell。如果你使用的是内建的shell命令，并不会涉及子shell。 在命令行提示符下运行脚本时一定要留心！



#### 变量截取

假设我们定义了一个变量为：

```jsx
代码如下:
file=/dir1/dir2/dir3/my.file.txt
```

可以用`${ }`分别替换得到不同的值：

```ruby
${file#*/}：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/}：删掉最后一个 /  及其左边的字符串：my.file.txt
${file#*.}：删掉第一个 .  及其左边的字符串：file.txt
${file##*.}：删掉最后一个 .  及其左边的字符串：txt
${file%/*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3
${file%%/*}：删掉第一个 /  及其右边的字符串：(空值)
${file%.*}：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*}：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my
```

记忆的方法为：

```ruby
代码如下:
# 是 去掉左边（键盘上#在 $ 的左边）
%是去掉右边（键盘上% 在$ 的右边）
单一符号是最小匹配；两个符号是最大匹配
${file:0:5}：提取最左边的 5 个字节：/dir1
${file:5:5}：提取第 5 个字节右边的连续5个字节：/dir2
```

也可以对变量值里的字符串作替换：

```ruby
代码如下:
${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt
${file//dir/path}：将全部dir 替换为 path：/path1/path2/path3/my.file.txt

利用 ${ } 还可针对不同的变数状态赋值(沒设定、空值、非空值)：

${file-my.file.txt} ：假如 $file 沒有设定，則使用 my.file.txt 作传回值。(空值及非空值時不作处理) 
${file:-my.file.txt} ：假如 $file 沒有設定或為空值，則使用 my.file.txt 作傳回值。 (非空值時不作处理)
${file+my.file.txt} ：假如 $file 設為空值或非空值，均使用 my.file.txt 作傳回值。(沒設定時不作处理)
${file:+my.file.txt} ：若 $file 為非空值，則使用 my.file.txt 作傳回值。 (沒設定及空值時不作处理)
${file=my.file.txt} ：若 $file 沒設定，則使用 my.file.txt 作傳回值，同時將 $file 賦值為 my.file.txt 。 (空值及非空值時不作处理)
${file:=my.file.txt} ：若 $file 沒設定或為空值，則使用 my.file.txt 作傳回值，同時將 $file 賦值為my.file.txt 。 (非空值時不作处理)
${file?my.file.txt} ：若 $file 沒設定，則將 my.file.txt 輸出至 STDERR。 (空值及非空值時不作处理)

${file:?my.file.txt} ：若 $file 没设定或为空值，则将 my.file.txt 输出至 STDERR。 (非空值時不作处理)
${#var} 可计算出变量值的长度：

${#file} 可得到 27 ，因为/dir1/dir2/dir3/my.file.txt 是27个字节
```

[参考来源](https://www.jb51.net/article/64804.htm)

#### let

et 命令是 BASH 中用于计算的工具，用于执行一个或多个表达式，变量计算中不需要加上`$`来表示变量。如果表达式中包含了空格或其他特殊字符，则必须引起来。

实例：

```bash
# !/bin/bash

let no++
let no--
let no+=10，let no-=20 # 分别等同于 let no=no+10，let no=no-20
let a=5+4
let b=9-3 
echo $a $b
```



### 数组变量

-   Bash Shell 只支持一维数组（不支持多维数组）。
-   初始化时不需要定义数组大小（与 PHP 类似）。
-   数组元素的下标由0开始。

#### 数组定义

Shell 数组用括号来表示，元素用”空格”符号分割开，语法格式如下：

```
array_name=(value1 value2 … valuen)
```

例如：my_array=(A B “C” D)

我们也可以使用下标来定义数组：

```shell
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```



#### 读取数组

这样是行不通的

```shell
$ echo $mytest      # 打印数组只会显示第一个值
one
```

一般格式是：`${array_name[index]}`

```shell
my_array=(A B "C" D)
echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"
```

**获取数组中的所有元素**

使用`@` 或 `*` 可以获取数组中的所有元素，例如：

```shell
my_array=(A B "C" D)
echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 数组的元素为: A B C D 数组的元素为: A B C D
```

**获取数组的长度**

获取数组长度的方法与获取字符串长度的方法相同，例如：

```shell
my_array=(A B "C" D)
echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"

# 数组元素个数为: 4 数组元素个数为: 4
```

#### 删除某个值

unset命令删除数组中的某个值，但是要小心，这可能会有点复杂。看下面的例子。

```bash
$ unset mytest[2]  

$ echo {mytest[*]}  # 遍历时会跳过被删除的索引位置
one two four five 
 
$ echo ${mytest[2]} # 但是该索引位置还占用着

$ echo {mytest[3]} 
four 
```

这个例子用unset命令删除在索引值为2的位置上的值。显示整个数组时，看起来像是索引 里面已经没这个索引了。但当专门显示索引值为2的位置上的值时，就能看到这个位置是空的。 最后，可以在unset命令后跟上数组名来删除整个数组。

```shell
$ unset mytest   
$ echo {mytest[*]}
```



#### 数组遍历

首先创建一个数组 array=( A B C D 1 2 3 4)

##### **标准的for循环**

```shell
for(( i=0;i<${#array[@]};i++)) #${#array[@]}获取数组长度用于循环
do    
	echo ${array[i]}
done
```



##### **for … in**

不带数组下标

```shell
for element in ${array[@]}    #也可以写成for element in ${array[*]}
do
    echo $element
done
```

带数组下标

```shell
for i in ${!array[@]}  
do   
   echo $i ${array[$i]}
done 
```



##### **while循环法**

```shell
i=0  
while [ $i -lt ${#array[@]} ]    #当变量（下标）小于数组长度时进入循环体
do  
    echo ${array[$i]}   #按下标打印数组元素
    let i++  
done 
```



### shell 展开

Bash 有七种扩展格式。本文只介绍其中五种：`~` 扩展、算术扩展、路径名称扩展、大括号扩展和命令替换。

![shell 展开](./shell展开.png)



#### `{}`花括号

从一个包含花括号的模式中创建多个文本字符串。

```shell
# 逗号
$ echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back

# 序列
$ echo Num_{1..5}
Num_1 Num_2 Num_3 Num_4 Num_5

$ echo Num_{1..10..2}  # 间隔
Num_1 Num_3 Num_5 Num_7 Num_9

$ echo {Z..A}          # 倒序
Z Y X W V U T S R Q P O N M L K J I H G F E D C B A

$ echo a{A{1,2},B{3,4}}b #嵌套
aA1b aA2b aB3b aB4b
```



#### `~`扩展 

Bash shell 把这个快捷方式展开成用户的完整的家目录。

```shell
$ echo ~
/home/student
```



#### 路径名称扩展

路径名称扩展是展开文件通配模式为匹配该模式的完整路径名称的另一种说法，匹配字符使用 `?` 和 `*`

-   `?` — 匹配字符串中特定位置的一个任意字符

-   `*` — 匹配字符串中特定位置的 0 个或多个任意字符



#### `$`展开

'$' 符号引入了三种 shell 展开，包括 “参数展开”，“命令替换” 和 “算术表达式”。

##### 参数展开

```shell
$ echo $USER
```

参数展开的基本的形式是 `${PARAMETER}`，整体被替换为 PARAMETER 的值。花括号如果 PARAMETER 是位置参数，而且由两个及以上的数字表示，这时必须使用花括号：`${10}`。另外当 PARAMETER 与其它字符相邻连接时，也必须使用花括号：`${Var}lala`。



##### 命令替换

命令替换是让一个命令的标准输出数据流被当做参数传给另一个命令的扩展形式。

命令替换有两种格式：``command`` 和 `$(command)`。

```shell
$ echo "Todays date is $(date)"
Todays date is Sun Apr  7 14:42:59 EDT 2019
```



##### 算术扩展

数字扩展的语法是 `$((arithmetic-expression))` ，分别用两个括号来打开和关闭表达式。算术扩展在 shell 程序或脚本中**类似命令替换**；表达式结算后的结果替换了表达式，用于 shell 后续的计算。

```shell
$ Var1=5 ; Var2=7 ; Var3=$((Var1*Var2)) ; echo "Var 3 = $Var3"
Var 3 = 35
```



#### 禁用展开

##### 双引号

把文本放在双引号中后，shell 使用的特殊字符，除了 `$`，`\` ，和 `（倒引号）之外， 则失去它们的特殊含义，被当作普通字符来看待。这意味着单词分割、路径名展开、波浪线展开、花括号展开都被禁止，**然而参数展开，算术展开，命令替换仍然有效。**

```shell
# 多余的空格会被压缩
$ echo this is a    test
this is a test

# 双引号关闭了单词分割功能
$ echo "this is a    test"
this is a    test
```

案例2

```shell
# 1. 没有引用的命令替换导致命令行包含 38 个参数。
$ echo $(cal)
January 2019 Su Mo Tu We Th Fr Sa 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31

# 2. 命令行只有一个参数，参数中包括嵌入的空格和换行符。
$ echo "$(cal)"
    January 2019
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31
```

##### 单引号

如果需要禁止所有的展开，需要使用单引号，包括转义符号`\`

##### 转义字符

在文件名中可能使用一些对于 shell 来说，有特殊含义的字符。这些字符包括 "$", "!", " " 等字符。在文件名中包含特殊字符，你可以这样做：

```
mv bad\&filename good_filename
```

注意在**单引号**中，反斜杠**失去它的特殊含义**，它会被看作普通字符。



### 重定向

#### 输出

覆盖原本文件

```
> 正常信息写入
2> 错误信息写入
&> 不论是正确还是错误信息，都写入文件中
```

追加文本文件，多一个`>`

```
>> 正常信息写入
2>> 错误信息写入
&>> 不论是正确还是错误信息，都写入文件中
```

#### 输入

```
<   # 输入重定向
<<  # 内联输入重定向
```

内联输入重定向符号是远小于号（<<）。除了这个符号，你必须指定一个文本标记来划分输 入数据的开始和结尾。它的用途请看 数学运算那一章节的 bc 计算器

```
$ wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
$ 3  9  42
```



#### 重定向绑定`>& `

**原理**：linux在执行shell命令之前，就**会确定好所有的输入输出位置**，并且**从左到右依次执行重定向的命令**



`command 2>&1`

这条命令用到了**重定向绑定**，采用`&`可以将两个输出绑定在一起，即错误输出将和标准输出共用一个文件描述符。

可以这样记住这条写法（虽然并不完全正确）首先，`2>1` 会被解释成将`stderr` 重定向到一个名为 `1`的文件中。 因此加入`&`来表示紧跟着的是文件描述符（file descriptor)而不是文件名，因此最终就是这样的形式： `2>&1`



理解上面的原理后，就可以明白下面的例子了：

**1、标准输出和标准错误重定向到不同log文件中**

```shell
sh mr_add_test.sh >log.log 2>log_err.log
```



**2、将标准输出和标准错误重定向到同一log文件**

```
foo >file 2>&1 
```

解读：`>file` 将标准输出重定向到文件中，`2>&1`  将错误绑定到标准输出上，此时标准输出已经重定向到file了



**3、输出标准输出和标准错误，同时保存到文件logfile**

方法一： `<command> 2>&1 | tee <logfile>`

管道符号把一个进程的标准输出作为另一个进程的标准输入。`2>&1`是把标准错误重定向到标准输出的副本一起输出。上面的命令，把标准输出和标准错误都输出作为tee命令的标准输入，tee的作用为把标准输入的内容拷贝到文件，并输出。



方法二：`<command>  2> logfile | cat - logfile`

cat可以带多个文件参数，同时显示多个文件的内容。 `-`代表标准输入，logfile是管道前保存的标准错误文本。



**4、只输标准出错误，并保存错误信息到文件中**

```
<command> 2>&1 >/dev/null | tee logfile
```

这条命令其实分为两命令来看：

1.  `2>&1`  将标准错误重定向到标准输出，注意，**此时标准输出还没有被重定向**，其输出仍然是屏幕。
2.   `/dev/null`文件是一个空设备，类似于windows内的回收站，使用`>/dev/null`将标准输出重定向到`/dev/null`，即不显示标准输出的内容。所以这时的标准输出就仅变为重定向过来的标准错误了。

相反，如果两者颠倒顺序，那标准输出连同它的副本都会被重定向到/dev/null，这是一个逻辑问题。

```
make 2>&1 >/dev/null     # 顺序1，错误还是输出到屏幕
make: *** No targets specified and no makefile found.  Stop.

make >/dev/null 2>&1     # 顺序2，不输出错误
```



**`>/dev/null 2>&1` 与 `>/dev/null 2>/dev/null`**

```
ls a.txt b.txt >out 2>out
cat out
# a.txt
# txt: No such file or directory
```

`out`中出现了丢失。采用这种写法，标准输出和错误输出会抢**占往out文件的管道**，所以可能会导致输出内容的时候出现缺失、覆盖等情况。有时候也有可能出现只有error信息或者只有正常信息的情况。不管怎么说，采用这种写法，最后的情况是无法预估的。

而且，由于out文件被打开了两次，两个文件描述符会抢占性的往文件中输出内容，所以整体IO效率不如`>/dev/null 2>&1`来得高。



**5、`&>`与`2>&1`的区别**

**&>file** 意思是把**标准输出** 和 **标准错误输出** 都重定向到文件file中

**2>&1**  意思是把 标准错误输出 重定向到 标准输出，标准输出并不一定输出到文件中哦

> Bash's man page mentions there's two ways to redirect [stderr and stdout](https://en.wikipedia.org/wiki/Standard_streams): `&> file` and `>& file`. Now, notice that it says both stderr and stdout.
>
> In case of this `>file 2>&1` we are doing redirection of stdout (1) to file, but then also telling stderr(2) to be redirected to the same place as stdout ! So the purpose may be the same, but the idea slightly different. In other words "John, go to school; Suzzie go where John goes".
>
> 在这个`> file 2>＆1`的情况下，我们正在将stdout（1）重定向到文件，然后还要告诉stderr（2）重定向到与stdout相同的位置！ 因此目的可能是相同的，但想法略有不同。 换句话说，“约翰，去上学；苏奇去约翰去的地方”。
>
> What about preference ? `&>` is a `bash` thing. So if you're porting a script, that won't do it. But if you're 100% certain your script will be only working on system with bash - then there's no preference.
>
>  “＆>”是专属于“bash”的东西。 因此，如果你有移植脚本的打算，就不会用这种方法。 但是，如果您100％确定您的脚本只在具有bash的系统上运行。那就没有什么问题。
>
> Here's an example with `dash` , the Debian Amquist Shell which is Ubuntu's default.
>
> ```bsh
> $ grep "YOLO" * &> /dev/null
> $ grep: Desktop: Is a directory
> grep: Documents: Is a directory
> grep: Downloads: Is a directory
> grep: Music: Is a directory
> grep: Pictures: Is a directory
> grep: Public: Is a directory
> grep: Templates: Is a directory
> grep: Videos: Is a directory
> grep: YOLO: Is a directory
> grep: bin: Is a directory
> ```
>
> As you can see, stderr is not being redirected。你可以看到标准错误没有被重定向【失败了】
>
> To address your edits in the question, you can use if statement to check $SHELL variable and change redirects accordingly
>
> But for most cases `> file 2>&1` should work。而`> file 2>&1` 大多数情况都是OK的
>
> ------
>
> In more technical terms, the form `[integer]>&word` is called [Duplicating Output File Descriptor](http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html#tag_02_07_06), and is a feature specified by POSIX Shell Command Language standard, which is supported by most POSIX-compliant and Brourne-like shells.
>
> See also [What does & mean exactly in output redirection?](https://askubuntu.com/a/1031663/295286)

来自：[What is the differences between &> and 2>&1](https://askubuntu.com/questions/635065/what-is-the-differences-between-and-21)

### 数学运算



#### expr (不推荐)

标准操作符在expr命令中工作得很好，但在脚本或命令行上使用它们时仍有问题出现。 许多expr命令操作符在shell中另有含义（比如星号）。

```
$ expr 5 * 2 
expr: syntax error 

# 需要进行转义
$ expr 5 \* 2
```



#### 方括号(只支持整数)

用`$[ operation ]`将数学表达式围起来

```
$ var1=$[1 + 5]
$ echo $var1 
# 6
```

 **注意**：bash shell数学运算符**只支持整数运算**。若要进行任何实际的数学计算，这是一个巨大的限制。z shell（zsh）提供了完整的浮点数算术操作。

```
var1=100
var2=45
var3=$[$var1 / $var2]
echo $var3
# 2
```



#### bc计算器(浮点数计算)

**命令行**

bash计算器支持变量print语句允许你打印 变量和数字，`-q` 命令行选项可以不显示bash计算器冗长的欢迎信息。

```bash
$ bc  -q
res=12 * 5.4 # 变量
print res
64.8
1+res
65.8    # 计算结果
quit	# 退出
```

浮点运算是由内建变量scale控制的。必须将这个值设置为你希望在计算结果中保留的小数 位数，否则无法得到期望的结果

```bash
$ bc -q
3.44 / 5 
0 
scale=4
3.44 / 5 
.6880 
quit 
$
```



**脚本**

基本格式如下：

`variable=$(echo "options; expression" | bc)`

-    options 允许你设置变量。 如果你需要不止一个变量， 可以用分号将其分开

-   expression参数定义了通过bc执行的数学表达式

```bash
$ cat test10 
#!/bin/bash 
var1=100 
var2=45 
var3=$(echo "scale=4; $var1 / $var2" | bc) 
echo The answer for this is $var3 
$
```

 **大量运算**

最好的办法是使用内联输入重定向，当然，必须用命令替换符号标识出用来给变量赋值的命令。

```bash
$ cat test12
#!/bin/bash
var1=10.46
var2=43.67
var3=33.2
var4=71
var5=$(bc << EOF 
scale = 4 
a1 = ( $var1 * $var2) 
b1 = ($var3 * $var4) 
a1 + b1 
EOF 
)

echo The final answer for this mess is $var5 
s$
```

你可以在bash计算器中赋值给变量。这一点很重要：在bash 计算器中创建的变量只在bash计算器中有效，**不能在shell脚本中使用**。

#### 圆括号

| 运算操作符/运算命令                              | 说明                                                         |
| :----------------------------------------------- | :----------------------------------------------------------- |
| `((a=10+66) ((b=a-15)) ((c=a+b))`                | 在 (( )) 中使用变量无需加上`$`前缀，(( )) 会自动解析变量名   |
| `a=$((10+66)`<br> `b=$((a-15))`<br> `c=$((a+b))` | 使用`$`获取 (( )) 命令的结果                                 |
| `((a>7 && b==c))`                                | (( )) 也可以进行逻辑运算，在 if 语句中常会使用逻辑运算。     |
| `echo $((a+10))`                                 | 需要立即输出表达式的运算结果时，可以在 (( )) 前面加`$`符号。 |
| `c=$((a=3+5, b=a+10))`                           | 多个表达式之间以逗号`,`分隔。以最后一个表达式的值作为整个 (( )) 命令的执行结果。 |

**数字高级比较** （双括号）

```
(( expression ))
```

双括号命令允许你在比较过程中使用高级数学表达式。test命令只能在比较中使用简单的算术操作。但还是**只支持整数**

```
val++     # 后增 
val--     # 后减
++val     # 先增
--val     # 先减
!         # 逻辑求反
~         # 位求反
**        # 幂运算
<<        # 左位移 
>>        # 右位移
&         # 位布尔和
|         # 位布尔或 
&&        # 逻辑和 
||        # 逻辑或
```

注意，**不需要将双括号中表达式里的大于号转义**。这是双括号命令提供的另一个高级特性。

```shell
#!/bin/bash 
# using double parenthesis 
# 
val1=10 
# 
if (( $val1 ** 2 > 90 )) 
then
    (( val2 = $val1 ** 2 ))
    echo "The square of $val1 is $val2" 
fi
```

#### let

et 命令是 BASH 中用于计算的工具，用于执行一个或多个表达式，变量计算中不需要加上`$`来表示变量。如果表达式中包含了空格或其他特殊字符，则必须引起来。

实例：

```bash
# !/bin/bash

let no++
let no--
let no+=10，let no-=20 # 分别等同于 let no=no+10，let no=no-20
let a=5+4
let b=9-3 
echo $a $b
```



### 退出脚本

shell中运行的每个命令都使用退出状态码（exit status）告诉shell它已经运行完毕。退出状态码是一个**0～255**的整数值，在命令结束运行时执行`exit code`传给shell。

| 状态码 | 说明                           |
| ------ | ------------------------------ |
| 0      | 成功结束                       |
| 1      | 一般性未知错误                 |
| 2      | 不合适的 shell 命令            |
| 126    | 命令不可执行，比如没有权限     |
| 127    | 没有找到命令                   |
| 128    | 无效的退出参数                 |
| 128+x  | 与 Linux 信号 x 相关的严重错误 |
| 130    | 通过 Ctrl+C 终止的命令         |
| 255    | 正常范围之外的退出状态码       |



### 脚本的健壮性

要写出健壮性更好的脚本，下面是一些相关的技巧：

- 使用 `-e` 参数，如：`set -e` 或是 `#!/bin/sh -e`，这样设置会让你的脚本出错就会停止运行，这样一来可以防止你的脚本在出错的情况下还在拼拿地干活停不下来。
- 使用 `-u` 参数，如： `set -eu`，这意味着，如果你代码中有变量没有定义，就会退出。
- 你可以使用 `set -x` 来debug输出。
- 考虑使用 `set -o pipefail` 来限制错误。还可以使用trap来截获信号（如截获ctrl+c）。
- 对一些变理，你可以使用默认值。如：`${FOO:-'default'}`
- 处理你代码的退出码。这样方便你的脚本跟别的命令行或脚本集成。
- 尽量使用 `&&` 来执行多个命令，而不是 `;`，这样会在出错的时候停止运行后续的命令。
- 对于一些字符串变量，使用引号括起，避免其中有空格或是别的什么诡异字符。
- 如果你的脚有参数，你需要检查脚本运行是否带了你想要的参数，或是，你的脚本可以在没有参数的情况下安全的运行。
- 为你的脚本设置 `-h` 和 `--help` 来显示帮助信息。千万不要把这两个参数用做为的功能。
- 使用 `$()` 而不是 “ 来获得命令行的输出，主要原因是易读。
- 小心不同的平台，尤其是 MacOS 和 Linux 的跨平台。
- 对于 `rm -rf` 这样的高危操作，需要检查后面的变量名是否为空，比如：`rm -rf $MYDIDR/*` 如果 `$MYDIR`为空，结果是灾难性的。
- 考虑使用 `find/while`而不是 `for/find`。如：`for F in $(find . -type f) ; do echo $F; done` 写成 `find . -type f | while read F ; do echo $F ; done` 不但可以容忍空格，而且还更快。
- 防御式编程，在正式执行命令前，把相关的东西都检查好，比如，文件目录有没有存在。

你还可以使用网站[ShellCheck](https://www.shellcheck.net/)来帮助你检查你的脚本。

转至：https://coolshell.cn/articles/19219.html



### 流程控制

#### 逻辑符号

`command1 && command2` ：如果 command1 执行成功，那么接着执行 command2。如果 command1 失败，就跳过 command2。

`command1 || command2`：如果 command1 失败，执行 command2。隐藏的逻辑是，如果 command1 成功，跳过 command2。

`&&` 和 `||` 两种运算符结合起来才能发挥它们的最大功效

```shell
前置 commands ; command1 && command2 || command3 ; 跟随 commands
```

语法解释：假如 command1 退出时返回码为零，就执行 command2，否则执行 command3。**类似于 if-else的逻辑**。

用具体代码试试，在 root 的 home 目录下测试：

```shell
[student@studentvm1 ~]$ Dir=/root/testdir ; mkdir $Dir && cd $Dir || echo "$Dir was not created."mkdir: cannot create directory '/root/testdir': Permission denied/root/testdir was not created.
```

现在在你的家目录执行，你将会有权限创建这个目录了：

```shell
[student@studentvm1 ~]$ Dir=~/testdir ; mkdir $Dir && cd $Dir || echo "$Dir was not created."
```

#### if else

```
if command 
then
    commands 
fi
```

或者

```
# 推荐
if command; then 
    commands
fi
```

>   通过把分号放在待求值的命令尾部，就可以将 then 语句放在同一行上了，这样看起来更 像其他编程语言中的 if-then 语句。

bash shell的if语句会运行`if`后面的那个命令。如果该命令的**退出状态码是0**，就执行`then`后的命令。在其他编程语言 中，if语句之后的对象是一个等式，这个等式的求值结果为TRUE或FALSE。

**if else**

```
if command
then
    commands 
else
    commands 
fi
```

**if elif** 

```
if command1 
then
    commands 
elif command2
then
    more commands 
fi
```

>   注意：记住， 在 elif 语句中， 紧跟其后的 else 语句属于 elif 代码块。 它们并不属于之前的 if-then 代码块。



#### test 命令

if-then语句是否能测试命令退出状态码之外的条件。答案是不能。

`test`命令提供了在`if-then`语句中测试不同条件的途径。如果test命令中列出的条件成立， `test`命令就会退出并返回退出状态码`0`。

```
if test condition 
then 
    commands
fi
```

##### 推荐写法

bash shell提供了另一种条件测试方法，无需在if-then语句中声明test命令。

```
if [ condition ] 
then
    commands
fi
```

方括号定义了测试条件。注意，

-   **第一个方括号之后和第二个方括号之前必须加上一个空格**， 否则就会报错。
-   括号内的大于号、小于号，需要进行转义，后面的双括号才不需要。



**否定写法(感叹号)**

```
if [ ! -f $FILE ]; then  # !后有空格
    echo "source company doesn\'t exist"
    exit
fi
```



##### 条件测试

test命令可以判断三类条件：

1.  数值比较 
2.  字符串比较
3.  文件比较

###### 整数比较

我们不能在 test命令中使用浮点值

```
n1 -eq n2    # 检查n1是否=n2 
n1 -ge n2    # 检查n1是否>=n2 
n1 -gt n2    # 检查n1是否>n2 
n1 -le n2    # 检查n1是否<=n2 
n1 -lt n2    # 检查n1是否<n2 
n1 -ne n2    # 检查n1是否!=n2
```

test命令只能在比较中使用简单的 算术操作。双括号命令提供了更多的数学符号。请看后面内容。



###### 字符串比较(有坑)

```
str1 = str2   # 检查str1是否和str2相同 
str1 != str2  # 检查str1是否和str2不同 
str1 \< str2   # 检查str1是否比str2小，注意一定要转义
str1 \> str2   # 检查str1是否比str2大，注意一定要转义
-n str1       # 检查str1的长度是否非0
-z str1       # 检查str1的长度是否为0，未在shell脚本中定义过，所以它的字符串长度仍然 为0
```

**字符串长度比较**

计算字符串的长度没有简单的方法。有很多种方法可以计算，但是我认为使用 `expr`（求值表达式）命令是相对最简单的一种。

```shell
MyVar="How long is this?" ; expr length "$MyVar"    # 注意 mac 中并没有 length 这个命令
```



这里会出现经常困扰shell程序员的问题：

-   大于号和小于号必须转义，否则shell会把它们当作**重定向符号**，把字符串值当作文件名；

-   大于和小于顺序和sort命令所采用的不同

    >   这是因为 比较测试中使用的是标准的ASCII顺序，根据每个字符的ASCII数值来决定排序结果。sort 命令使用的是系统的本地化语言设置中定义的排序顺序。对于英语，本地化设置指定了在排序顺 序中小写字母出现在大写字母前。

-   未在shell脚本中定义过，`-z`认为它的字符串长度为0



**字符串中有空格的情况处理**

有些人从在文件名或者命令行参数中使用空格，你需要在编写脚本时时刻记得这件事。你需要时刻记得用引号包围变量。

`if [ $filename = "foo" ];`

当$filename变量包含空格时就会挂掉。可以这样解决：

`if [ "$filename" = "foo" ];`

使用`$@`变量时，你也需要使用引号，因为空格隔开的两个参数会被解释成两个独立的部分。



###### 文件比较

| 操作符            | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `-a filename`     | 如果文件存在，返回真值；文件可以为空也可以有内容，但是只要它存在，就返回真值 |
| `-b filename`     | 如果文件存在且是一个块设备，如 `/dev/sda` 或 `/dev/sda1`，则返回真值 |
| `-c filename`     | 如果文件存在且是一个字符设备，如 `/dev/TTY1`，则返回真值     |
| `-d filename`     | 如果文件存在且是一个目录，返回真值                           |
| `-e filename`     | 如果文件存在，返回真值；与上面的 `-a` 相同，可用于文件和目录 |
| `-f filename`     | 如果文件存在且是一个一般文件，不是目录、设备文件或链接等的其他的文件，则返回 真值 |
| `-g filename`     | 如果文件存在且 `SETGID` 标记被设置在其上，返回真值           |
| `-h filename`     | 如果文件存在且是一个符号链接，则返回真值                     |
| `-k filename`     | 如果文件存在且粘滞位已设置，则返回真值                       |
| `-p filename`     | 如果文件存在且是一个命名的管道（FIFO），返回真值             |
| `-r filename`     | 如果文件存在且有可读权限（它的可读位被设置），返回真值       |
| `-s filename`     | 如果文件存在且大小大于 0，返回真值（存在并非空）；如果一个文件存在但大小为 0，则返回假值 |
| `-t fd`           | 如果文件描述符 `fd` 被打开且被关联到一个终端设备上，返回真值 |
| `-u filename`     | 如果文件存在且它的 `SETUID` 位被设置，返回真值               |
| `-w filename`     | 如果文件存在且有可写权限，返回真值                           |
| `-x filename`     | 如果文件存在且有可执行权限，返回真值                         |
| `-G filename`     | 如果文件存在且文件的组 ID 与当前用户相同，返回真值           |
| `-L filename`     | 如果文件存在且是一个符号链接，返回真值（同 `-h`）            |
| `-N filename`     | 如果文件存在且从文件上一次被读取后文件被修改过，返回真值     |
| `-O filename`     | 如果文件存在且你是文件的拥有者，返回真值                     |
| `-S filename`     | 如果文件存在且文件是套接字，返回真值                         |
|                   | 在你尝试使用`ef`、`-nt`或 `-ot`比较文件之前，必须先确认文件是存在的。 |
| `file1 -ef file2` | 如果文件 `file1` 和文件 `file2` 指向同一设备的同一 INODE 号，返回真值（即硬链接） |
| `file1 -nt file2` | 如果文件 `file1` 比 `file2` 新（根据修改日期），或 `file1` 存在而 `file2` 不存在，返回真值 |
| `file1 -ot file2` | 如果文件 `file1` 比 `file2` 旧（根据修改日期），或 `file1` 不存在而 `file2` 存在 |

判断**文件存在并不为空**的脚本

```shell
File="TestFile1"
echo "This is $File" > $File
if [ -s $File ]
   then
   echo "$File exists and contains data."
elif [ -e $File ]
   then
   echo "$File exists and is empty."
else
   echo "$File does not exist."
fi
```

###### 其他杂项

这些杂项操作符展示一个 shell 选项是否被设置，或一个 shell 变量是否有值，但是它不显示变量的值，只显示它是否有值。

| 操作符       | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `-o optname` | 如果一个 shell 选项 `optname` 是启用的（查看内建在 Bash 手册页中的 set `-o` 选项描述下面的选项列表），则返回真值 |
| `-v varname` | 如果 shell 变量 `varname` 被设置了值（被赋予了值），则返回真值 |
| `-R varname` | 如果一个 shell 变量 `varname` 被设置了值且是一个名字引用，则返回真值 |

https://linux.cn/article-11687-1.html



##### 布尔逻辑

```
if-then语句允许你使用布尔逻辑来组合测试。有两种布尔运算符可用：  
[ condition1 ] && [ condition2 ] 
[ condition1 ] || [ condition2 ]
```

例子

```bash
#!/bin/bash 
# testing compound comparisons 
# if [ -d $HOME ] && [ -w $HOME/testing ] 
then
    echo "The file exists and you can write to it" 
else
    echo "I cannot write to the file" 
fi
```

#### 双括号与双中括号(进阶)

##### 数字高级比较 （双括号）

```
(( expression ))
```

双括号命令允许你在比较过程中使用高级数学表达式。test命令只能在比较中使用简单的算术操作。但还是**只支持整数**

```
val++     # 后增 
val--     # 后减
++val     # 先增
--val     # 先减
!         # 逻辑求反
~         # 位求反
**        # 幂运算
<<        # 左位移 
>>        # 右位移
&         # 位布尔和
|         # 位布尔或 
&&        # 逻辑和 
||        # 逻辑或
```

注意，**不需要将双括号中表达式里的大于号转义**。这是双括号命令提供的另一个高级特性。

```shell
#!/bin/bash 
# using double parenthesis 
# 
val1=10 
# 
if (( $val1 ** 2 > 90 )) 
then
    (( val2 = $val1 ** 2 ))
    echo "The square of $val1 is $val2" 
fi
```



##### 字符串高级比较（双中括号）

```
[[ expression ]]
```

-   不需要把变量名用双引号`""`包围起来，即使变量是空值，也不会出错。
-   不需要、也不能对 >、< 进行转义，转义后反而会出错。

>   双方括号在bashshell中工作良好。不过要小心，不是所有的shell都支持双方括号。

##### 逻辑运算

对多个表达式进行逻辑运算时，可以使用逻辑运算符将多个 test 命令连接起来，例如：

```
[ -z "$str1" ] || [ -z "$str2" ]
```

你也可以借助选项把多个表达式写在一个 test 命令中，例如：

```
[ -z "$str1" -o -z "$str2" ]
```

但是，这两种写法都有点“别扭”，[[ ]]  支持在括号内直接使用 &&、|| 和 ! 三种逻辑运算符。 使用 [[ ]] 对上面的语句进行改进：

```
[[ -z $str1 || -z $str2 ]]
```

这种写法就比较简洁漂亮了。注意，[[ ]] 剔除了 test 命令的`-o`和`-a`选项，你只能使用 || 和 &&。这意味着，你不能写成下面的形式：

```
[[ -z $str1 -o -z $str2 ]] ×
```

当然，使用逻辑运算符将多个 [[ ]] 连接起来依然是可以的，因为这是 Shell 本身提供的功能，跟 [[ ]] 或者 test 没有关系，如下所示：

```
[[ -z $str1 ]] || [[ -z $str2 ]]
```



##### [[ ]] 支持正则表达式

在 Shell [[ ]] 中，可以使用`=~`

```
[[ str =~ regex ]]
if [[ $tel =~ ^1[0-9]{10}$ ]]
```

有了 `[[ ]]`，你还有什么理由使用 test 或者 `[ ]`，`[[ ]]` 完全可以替代之，而且更加方便，更加强大。

但是 `[[ ]]` 对数字的比较仍然不友好，所以我建议，以后大家使用 `if` 判断条件时，用 `(()) `来处理整型数字，用` [[ ]]` 来处理字符串或者文件。  



#### swtich case

```bash
case variable in 
pattern1 | pattern2) 
    commands1;; 
pattern3) 
    commands2;; 
*) 
    default_commands;; 
esac
```

例子：

```bash
case $USER in 
rich | barbara)
    echo "Welcome, $USER"
    echo "Please enjoy your visit";; 
testing)
    echo "Special testing account";; 
jessica)
    echo "Do not forget to log off when you're done";; 
*)
    echo "Sorry, you are not allowed here";; 
esac
```



#### for

Python 风格

```
for var in list 
do 
    $var commands 
done
```

或者

```bash
for var in list; do 
    $var commands
done
```

类似于 python，执行到 `for`的时候，已经将数据加载到 list 中了，并不是每次加载一行，然后赋值给变量。list 的内容其实已经全部通过 `IFS` 分割然后加载进来了。

**list是以空格分割的**

例子：

```bash
for test in Alabama Alaska Arizona Arkansas California Colorado do 
    echo "The next state is $test" 
done 
echo "The last state we visited was $test"
```

使用`$var` 获取列表中的值，for循环假定每个值都是用空格分割的。 如果有包含空格的数据值，就必须用双引号将这些值圈起来。或者查看【字段分隔符】章节的解决方案



**list 中有引号的情况**

```bash
for test in I don't know if this'll work 
do 
    echo "word:$test" 
done 

$ ./badtest1 
word:I 
word:dont know if thisll 
word:work
```

shell看到了列表值中的单引号并尝试使用它们来定义一个单独的数据值，这真是把事情搞得一团糟。 有两种办法可解决这个问题：

-   使用转义字符（反斜线）来将单引号转义； 
-   使用双引号来定义用到单引号的值:`"this'll"`

```bash
for test in I don\'t know if "this'll" work 
do 
    echo "word:$test" 
done
```



**list添加**

用`"`进行拼接

```
list="Alabama Alaska Arizona Arkansas Colorado" 
list=$list" Connecticut" # 拼接
```



**循环命令结果**

使用命令替换符号可以对命令的结果进行循环

```shell
for RPM in `rpm -qa | sort | uniq` ; do rpm -qi $RPM ; done
```



##### 字段分隔符

特殊的环境变量`IFS`，叫作内部字段分隔符（internal field separator）。 IFS环境变量定义了bash shell用作字段分隔符的一系列字符。默认情况下，bash shell会将下列字 符当作字段分隔符： 

-    空格 
-   制表符 
-    换行符

如果你想修改`IFS`的值，使其只能识别换行符，那就必须这么做：`IFS=$'\n'`

指定多个IFS字符，只要将它们在赋值行串起来就行。`IFS=$'\n':;"`  ，这个赋值会将换行符、冒号、分号和双引号作为字段分隔符。

>   一个可参考的安全实践是在改变 IFS 之前保存原 来的 IFS 值，之后再恢复它。 这种技术可以这样实现：
>
>   `IFS.OLD=$IFS `
>
>   `IFS=$'\n' `
>
>   <在代码中使用新的IFS值>
>
>   ` IFS=$IFS.OLD`
>
>   这就保证了在脚本的后续操作中使用的是 IFS 的默认值。



##### 遍历目录

目录名和文件名中包含空格当然是合法的。要适应这种情况，一种方法是将`$file`变量用双引号圈起来。

```shell
for file in /home/rich/test/* 
do
    if [ -d "$file" ] 
    then 
        echo "$file is a directory" 
    elif [ -f "$file" ] 
    then 
        echo "$file is a file" 
    fi 
done
```



另一种方法是指定`IFS`

典型的例子是处理/etc/passwd文件中的数据。这要求你逐行遍历/etc/passwd文件，并将IFS 变量的值改成冒号，这样就能分隔开每行中的各个数据段了。

```bash
#!/bin/bash # changing the IFS value

IFS.OLD=$IFS 
IFS=$'\n' # 指定分隔符为换行
for entry in $(cat /etc/passwd) # 这时数据已经通过\n加载在list中了
do
    echo "Values in $entry –"
    IFS=: # 指定为冒号，后面不用恢复成\n，原因如上面的注释
    for value in $entry
    do
        echo " $value"
    done 
done
```



##### c语言风格的 for

```
for (( variable assignment ; condition ; iteration process ))
```

这与之前的 bash shell 标准有些不同

-   变量赋值可以有空格； 
-   条件中的变量不以美元符开头； 
-   迭代过程的算式未用expr命令格式；

```
#!/bin/bash 
# multiple variables

for ((i=1; i<=100; i ++))
do
	echo $i
done


for (( a=1, b=10; a <= 10 && b>=5; a++, b-- )) 
do 
    echo "$a - $b" 
done
```



#### while 与 until

while命令的格式是：

```
while test_command 
do 
    other commands 
done
```

例子1

```
while [ $var1 -gt 0 ]
do
    echo $var1
    var1=$[ $var1 - 1 ] 
done
```



`while`命令允许你在while语句行定义多个测试命令：

-   每个测试命令都出现在**单独的一行**上。
-   只有**最后一个**测试命令的退出状态码会被用来决定什么时候结束循环。
-   在每次迭代中所有的测试命令都会被执行，包括测试命令失败的最后一次迭代。要留心这种用法。

```bash
#!/bin/bash
# multiple variables

var1=10

while echo $var1 echo “minglin2” # 可以有多条命令，但是测试命令只能一行一条
    [ $var1 -ge 0 ]
do
    echo "This is inside the loop"
    var1=$[ $var1 - 1 ]
done
```



until命令和while命令工作的方式完全相反，只有测试命令的退出状态码不为0，bash shell才会执行循环中列出的命令。其他的与 `while` 相同

#### break 与 continue

默认 break 只跳出所在的最内层的循环。

有时你在内部循环，但需要停止外部循环。break命令接受单个命令行参数值：

`break n`其中n指定了要跳出的循环层级。

默认情况下，n为1，表明跳出的是当前的循环。如果你将 n设为2，break命令就会停止下一级的外部循环。



和break命令一样，continue命令也允许通过命令行参数指定要继续执行哪一级循环：

`continue n`其中n定义了要继续的循环层级。

默认情况下，n为1，表明当继续执行下一次循环。

例子：当 2<a<4时跳过循环

```shell
for (( a = 1; a <= 5; a++ )) 
do
    echo "Iteration $a:"
    for (( b = 1; b < 3; b++ )) 
    do
        if [ $a -gt 2 ] && [ $a -lt 4 ]
        then
            continue 2 # 当 2<a<4时跳过循环
        fi 
        var3=$[ $a * $b ] 
        echo " The result of $a * $b is $var3" 
    done
done

$ ./test22 
Iteration 1:
    The result of 1 * 1 is 1 
    The result of 1 * 2 is 2 
Iteration 2:
    The result of 2 * 1 is 2 
    The result of 2 * 2 is 4 
Iteration 3: # 跳过了第 3 次
Iteration 4:
    The result of 4 * 1 is 4 
    The result of 4 * 2 is 8 
Iteration 5:
    The result of 5 * 1 is 5 
    The result of 5 * 2 is 10
```





#### 循环的重定向

你可以对循环的输出使用管道或进行重定向。这可以通过在关键字`done`之后添加`>`或 `|`命令来实现。

重定向到文件

```bash
for file in /home/rich/* 
do 
    if [ -d "$file" ] 
    then 
        echo "$file is a directory" 
    elif 
        echo "$file is a file" 
    fi 
done > output.txt  # 重定向到文件
```

从文件中循环读取

```bash
#!/bin/bash 
# process new user accounts

input="users.csv" 
while IFS=',' read -r userid name 
do
    echo "adding $userid"
    useradd -c "$name" -m $userid 
done < "$input"      
```

​          

### 函数

所有函数**在使用前必须定义**。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

```shell
# [ ] 中括号表示可选

[ function ] funname [()]
{

    action;
    [return int;]
}
```

调用函数时可以向其传递参数。在函数体内部，通过 `$n` 的形式来获取参数的值

```shell
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```



### Tips

#### 命令行参数

位置参数变量是标准的数字：`$0`是程序名，`$1`是第 一个参数，`$2`是第二个参数，依次类推，直到第九个参数`$9`。



#### 常用的转义字符

4个最常用的转义字符如下所示。

>   反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符串。
>
>   单引号（''）：全局转义，转义其中所有的变量为单纯的字符串。
>
>   双引号（""）：保留其中的变量属性，不进行转义处理。
>
>   反引号（``）：把其中的命令执行后返回结果，等价于$(命令)



#### 获取文件的绝对路径

**误区一**

是使用 **pwd** 命令，print name of current/working directory

你可以试试 `bash shell/a.sh`，`a.sh` 内容是 `pwd`，你会发现，显示的是执行命令的路径 `/home/june`，并不是`a.sh` 所在路径：`/home/june/shell/a.sh`

**误区二**

 **$0**，这个也是不对的，这个$0是Bash环境下的特殊变量，其真实含义是：

这个`$0`有可能是好几种值，跟调用的方式有关系：

-   使用一个文件调用bash，那$0的值，是那个文件的名字(没说是绝对路径噢)
-   使用-c选项启动bash的话，真正执行的命令会从一个字符串中读取，字符串后面如果还有别的参数的话，使用从$0开始的特殊变量引用(跟路径无关了)
-   除此以外，$0会被设置成调用bash的那个文件的名字(没说是绝对路径)



**正确方法：dirname 配合 readlink**

`dirname` ：可以获取所在目录，输出已经去除了尾部的“/”字符部分的名称；如果名称中不包含“/”，则显示“.”(表示当前目录)。

>   取决于你传递给它的是不是绝对路径

`readlink` 可以获取文件的完整路径

最终形式：

```shell
echo $(dirname $(readlink -f "$0"))
```



例子

下面对比下正确答案：

```
Jun@VAIO 192.168.1.216 23:52:54 ~ >
cat shell/a.sh
#!/bin/bash
echo '$0: '$0
echo "pwd: "`pwd`
echo "============================="
echo "scriptPath1: "$(cd `dirname $0`; pwd)
echo "scriptPath2: "$(pwd)
echo "scriptPath3: "$(dirname $(readlink -f $0))
echo "scriptPath4: "$(cd $(dirname ${BASH_SOURCE:-$0});pwd)
echo -n "scriptPath5: " && dirname $(readlink -f ${BASH_SOURCE[0]})

Jun@VAIO 192.168.1.216 23:53:17 ~ >
bash shell/a.sh
$0: shell/a.sh
pwd: /home/Jun
=============================
scriptPath1: /home/Jun/shell
scriptPath2: /home/Jun
scriptPath3: /home/Jun/shell
scriptPath4: /home/Jun/shell
scriptPath5: /home/Jun/shell
```

在此解释下 scriptPath1 ：

-   `dirname $0`，取得当前执行的脚本文件的父目录
-   `cd dirname $0`，进入这个目录(切换当前工作目录)
-   `pwd`，显示当前工作目录(cd执行后的)
-   由此，我们获得了当前正在执行的脚本的存放路径。



####  检查命令是否存在

用 `which` 吗？最好不要用，因为很多操作系统的 `which` 命令没有设置退出状态码，这样你不知道是否是有那个命令。

```shell
# POSIX 兼容:
command -v [the_command]
 
# bash 环境:
hash [the_command]
type [the_command]
 
# 示例：
gnudate() {
    if hash gdate 2> /dev/null; then
        gdate "$@"
    else
        date "$@"
    fi
}
```



#### 路径中获取文件

```shell
chengdan_file="dir/file.txt"
echo $chengdan_file
file_name=${chengdan_file##*/}   # 去除目录
file_name=${file_name%%.*}       # 去除后缀
```



## 最佳实践

```shell
#!/usr/bin/env bash
# Bash3 Boilerplate. Copyright (c) 2014, kvz.io

set -o errexit
set -o pipefail
set -o nounset
# set -o xtrace

# Set magic variables for current file & dir
__dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
__file="${__dir}/$(basename "${BASH_SOURCE[0]}")"
__base="$(basename ${__file} .sh)"
__root="$(cd "$(dirname "${__dir}")" && pwd)" # <-- change this as it depends on your app

arg1="${1:-}"

# 显示执行进展
echo -e '##################################【第1步:xxx:开始】###################################################'
echo -e '##################################【第1步:xxx:完成】###################################################\n'

# 对于耗时的任务，最好复用本地文件
if [[ !-f ${file} ]];then
	sh xxx.sh
else
	echo "复用已存在的${file}"
if
```



1.  使用长的参数名，以便增加可读写，除非在命令行中简短的参数更快速 (`logger --priority` vs `logger -p`).
2.  `set -o errexit` (a.k.a `set -e`) 让脚本在**运行出错**时退出，而不会继续执行下去
3.  `set -o nounset` (a.k.a. `set -u`)让脚本在使用了**未声明变量**时退出.
4.  `set -o xtrace` (a.k.a `set -x`) 用于 debug
5.  `set -o pipefail` 用于捕获管道命令的出错，比如捕获 mysqldump 的在`mysqldump |gzip`中出现的错误
6.  `#!/usr/bin/env bash` 比 `#!/bin/bash`更有兼容性。
7.  用`{}`括起你的变量
8.  You don't need two equal signs when checking `if [ "${NAME}" = "Kevin" ]`. TODO
9.  在脚本的头部定义魔法变量、basename、目录等等。

参考:https://kvz.io/bash-best-practices.html