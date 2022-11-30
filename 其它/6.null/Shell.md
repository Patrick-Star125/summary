# Shell

shell是一种负责人机交互的抽象，本身是一种脚本语言。而Bash是shell的一种实现，可以简单理解为bash命令激活自动化的shell脚本来对操作系统进行管理。在Linux系统中，bash最为常见，在Windows系统中，可以使用WSL或者GIT bash来使用bash命令

命令的参数如果只有一个字母用单横杠-，如果是一个单词用双横杠--，如果想知道有哪些命令可以用--help

## 认识Shell

在shell中，程序有两个主要的“流”：他们的输入流和输出流。 当程序尝试读取信息时，它们会从输入流中进行读取，当程序打印信息时，它们会将信息输出到输出流中。 通常，一个程序的输入输出流都是您的终端。也就是，您的键盘作为输入，显示器作为输出。 但是，我们也可以重定向这些流！

最简单的重定向是 `< file` 和 `> file`。这两个命令可以将程序的输入输出流分别重定向到文件：

```bash
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

您还可以使用 `>>` 来向一个文件追加内容。使用管道（ *pipes*），我们能够更好的利用文件重定向。 `|`操作符允许我们将一个程序的输出和另外一个程序的输入连接起来：

```bash
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

对于大多数的类Unix系统，有一类用户是非常特殊的，那就是：根用户（root用户）。 您应该已经注意到来，在上面的输出结果中，根用户几乎不受任何限制，他可以创建、读取、更新和删除系统中的任何文件。 通常在我们并不会以根用户的身份直接登陆系统，因为这样可能会因为某些错误的操作而破坏系统。 取而代之的是我们会在需要的时候使用 `sudo` 命令。顾名思义，它的作用是让您可以以su（super user 或 root的简写）的身份do一些事情。 当您遇到拒绝访问（permission denied）的错误时，通常是因为此时您必须是根用户才能操作。此时也请再次确认您是真的要执行此操作。

有一件事情是您必须作为根用户才能做的，那就是向`sysfs` 文件写入内容。系统被挂在在`/sys`下， `sysfs` 文件则暴露了一些内核（kernel）参数。 因此，您不需要借助任何专用的工具，就可以轻松地在运行期间配置系统内核。**注意 Windows or macOS没有这个文件**

例如，您笔记本电脑的屏幕亮度写在 `brightness` 文件中，它位于

```groovy
/sys/class/backlight
```

通过将数值写入该文件，我们可以改变屏幕的亮度。现在，蹦到您脑袋里的第一个想法可能是：

```ruby
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

出乎意料的是，我们还是得到了一个错误信息。毕竟，我们已经使用了 `sudo` 命令！关于shell，有件事我们必须要知道。`|`、`>`、和 `<` 是通过shell执行的，而不是被各个程序单独执行。 `echo` 等程序并不知道`|`的存在，它们只知道从自己的输入输出流中进行读写。 对于上面这种情况， *shell* (权限为您的当前用户) 在设置 `sudo echo` 前尝试打开 brightness 文件并写入，但是系统拒绝了shell的操作因为此时shell不是根用户。

明白这一点后，我们可以这样操作：

```bash
$ echo 3 | sudo tee brightness
```

因为打开`/sys` 文件的是`tee`这个程序，并且该程序以`root`权限在运行，因此操作可以进行。 这样您就可以在`/sys`中愉快地玩耍了，例如修改系统中各种LED的状态（路径可能会有所不同）：

```bash
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
```

## Shell脚本

shell脚本是深入使用shell的基础，关于shell有以下特点：

* 大多数shell都有自己的脚本语言，其中包含变量、控制流和自己的语法。

* 在shell脚本中，创建命令管道、将结果保存到文件中以及读取标准输入都是基本的操作

* 赋值：`foo=bar`可以将foo赋值为bar，并且可以用`$foo`访问foo变量

* 条件和函数：bash支持控制流技术，包括if、case、while和for，以及基本的函数

* 函数：以一个简单的函数为例

  ~~~bash
  mcd () {
      mkdir -p "$1"
      cd "$1"
  }
  ~~~

  上面的`$1`指第一个参数，bash使用各种特殊符号来表示参数，如下图所示

  ![](http://pic.netpunk.space/images/2022/05/29/20220529101124.png)

* 返回码

  * 命令行总是会返回结果到`STDOUT`，返回异常到`SEDERR`，返回码或退出状态是一种很好的标志脚本/命令执行状态的方式。例如，返回码为0通常表示一切正常；任何与0不同的内容都表示发生错误。
  * 返回码也可以用来作为判断下一条执行语句的条件变量，使用`&&`和`||`操作符把判断条件和执行命令联系起来，`;`操作符只是单纯的分隔符。

* 子命令：另一种常见的设计模式是将命令的输出当作参数

  * 例如，`for file in $(ls)`是一个很明显的命令替换模式
  * 在上面讲的输出重定向中，也有一种进程替换的方法，例如`diff <(ls foo) <(ls bar)`，将显示dirs`foo`和`bar`中文件之间的差异。

  ~~~bash
  #这里是一个综合了以上知识的例子
  #!/bin/bash
  
  echo "Starting program at $(date)" # Date will be substituted
  #在shell脚本里，“”和‘’含义不相同
  echo "Running program $0 with $# arguments with pid $$"
  
  for file in "$@"; do
      grep foobar "$file" > /dev/null 2> /dev/null
      # When pattern is not found, grep has exit status 1
      # We redirect STDOUT and STDERR to a null register since we do not care about them
      if [[ $? -ne 0 ]]; then
          echo "File $file does not have any foobar, adding one"
          echo "# foobar" >> "$file"
      fi
  done
  ~~~

* 参数通配符

  * 如果我们想在命令行参数里面用通配符，可以用`?`和`*`，其中`?`表示匹配1个任意字符，`*`表示匹配多个任意字符
  * 在shell脚本中，大括号表示匹配，例如`mv *{.py,.sh} folder`表示移动所有后缀为.py和.sh的文件
  
* shebang符号行

  * Shebang 通常在 Unix 系统脚本的第一行开头使用 指明执行这个脚本文件的执行程序，无论脚本在系统的什么位置，都会使用Shebang指定的应用程序（在环境变量里）来执行。

* Shell函数和Shell脚本的区别

  * 函数必须使用与shell相同的语言，而脚本可以使用任何语言编写。这就是为什么为脚本包含shebang很重要。
  * 函数在读取其定义时加载一次。每次执行脚本时都会加载脚本。这使得函数的加载速度略快，但无论何时更改它们，都必须重新加载其定义。
  * 函数在当前shell环境中执行，而脚本在其自己的进程中执行。因此，函数可以修改环境变量，例如更改当前目录，而脚本不能。脚本将通过使用export导出的值环境变量传递
  * 与任何编程语言一样，函数是实现模块化、代码重用和shell代码清晰性的强大构造。shell脚本通常会包含自己的函数定义。

## Shell工具

### 帮助手册

如果你没有具体的学习过一个命令的使用方法，那么在使用这个命令的时候我们可以有哪些可选的参数呢？答案是肯定的，系统内置就有

最常用的方法是为对应的命令行添加`-h` 或 `--help` 标记。另外一个更详细的方法则是使用`man` 命令。[`man`](http://man7.org/linux/man-pages/man1/man.1.html) 命令是手册（manual）的缩写，它提供了命令的用户手册。例如`man ls`可以打印出ls命令的详细信息，但有时候手册内容太过详实，让我们难以在其中查找哪些最常用的标记和语法。[TLDR pages](https://tldr.sh/) 是一个很不错的替代品，它提供了一些案例，可以帮助您快速找到正确的选项。

### 查找文件

程序员们面对的最常见的重复任务就是查找文件或目录。所有的类UNIX系统都包含一个名为 [`find`](http://man7.org/linux/man-pages/man1/find.1.html)的工具，它是shell上用于查找文件的绝佳工具。`find`命令会递归地搜索符合条件的文件，例如：

```python
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '**/test/**/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```

除了列出所寻找的文件之外，find还能对所有查找到的文件进行操作。这能极大地简化一些单调的任务。

```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {.}.jpg \;
```

尽管 `find` 用途广泛，它的语法却比较难以记忆。但shell的哲学之一便是寻找（更好用的）替代方案。 记住，shell最好的特性就是您只是在调用程序，因此您只要找到合适的替代程序即可（甚至自己编写）。

例如， [`fd`](https://github.com/sharkdp/fd) 就是一个更简单、更快速、更友好的程序，它可以用来作为`find`的替代品。它有很多不错的默认设置，例如输出着色、默认支持正则匹配、支持unicode并且我认为它的语法更符合直觉。以模式`PATTERN` 搜索的语法是 `fd PATTERN`。

### 查找代码

查找文件是很有用的技能，但是很多时候您的目标其实是查看文件的内容。一个最常见的场景是您希望查找具有某种模式的全部文件，并找它们的位置。

为了实现这一点，很多类UNIX的系统都提供了[`grep`](http://man7.org/linux/man-pages/man1/grep.1.html)命令，它是用于对输入文本进行匹配的通用工具。它是一个非常重要的shell工具，我们会在后续的数据清理课程中深入的探讨它。

`grep` 有很多选项，这也使它成为一个非常全能的工具。其中我经常使用的有 `-C` ：获取查找结果的上下文（Context）；`-v` 将对结果进行反选（Invert），也就是输出不匹配的结果。举例来说， `grep -C 5` 会输出匹配结果前后五行。当需要搜索大量文件的时候，使用 `-R` 会递归地进入子目录并搜索所有的文本文件。

但是，我们有很多办法可以对 `grep -R` 进行改进，例如使其忽略`.git` 文件夹，使用多CPU等等。

因此也出现了很多它的替代品，包括 [ack](https://beyondgrep.com/), [ag](https://github.com/ggreer/the_silver_searcher) 和 [rg](https://github.com/BurntSushi/ripgrep)。它们都特别好用，但是功能也都差不多，我比较常用的是 ripgrep (`rg`) ，因为它速度快，而且用法非常符合直觉。例子如下：

```python
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

与 `find`/`fd` 一样，重要的是你要知道有些问题使用合适的工具就会迎刃而解，而具体选择哪个工具则不是那么重要。

### 查找命令

目前为止，我们已经学习了如何查找文件和代码，但随着你使用shell的时间越来越久，您可能想要找到之前输入过的某条命令。首先，按向上的方向键会显示你使用过的上一条命令，继续按上键则会遍历整个历史记录。

`history` 命令允许您以程序员的方式来访问shell中输入的历史命令。这个命令会在标准输出中打印shell中的里面命令。如果我们要搜索历史记录，则可以利用管道将输出结果传递给 `grep` 进行模式搜索。 `history | grep find` 会打印包含find子串的命令。

对于大多数的shell来说，您可以使用 `Ctrl+R` 对命令历史记录进行回溯搜索。敲 `Ctrl+R` 后您可以输入子串来进行匹配，查找历史命令行。

反复按下就会在所有搜索结果中循环。在 [zsh](https://github.com/zsh-users/zsh-history-substring-search)中，使用方向键上或下也可以完成这项工作。

`Ctrl+R` 可以配合 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 使用。`fzf` 是一个通用对模糊查找工具，它可以和很多命令一起使用。这里我们可以对历史命令进行模糊查找并将结果以赏心悦目的格式输出。

另外一个和历史命令相关的技巧我喜欢称之为**基于历史的自动补全**。 这一特性最初是由 [fish](https://fishshell.com/) shell 创建的，它可以根据您最近使用过的开头相同的命令，动态地对当前对shell命令进行补全。这一功能在 [zsh](https://github.com/zsh-users/zsh-autosuggestions) 中也可以使用，它可以极大对提高用户体验。

最后，有一点值得注意，输入命令时，如果您在命令的开头加上一个空格，它就不会被加进shell记录中。当你输入包含密码或是其他敏感信息的命令时会用到这一特性。如果你不小心忘了在前面加空格，可以通过编辑。`bash_history`或 `.zhistory` 来手动地从历史记录中移除那一项。

## 常用工具

### fd

fd是Linux原生find命令的一个更快更易用的一个版本，常用的查找方式如下：

在当前目录里递归的找到符合模式的文件

```bash
fd {{pattern}}
```

找foo开头的文件

```bash
fd {{'^foo'}}
```

寻找后缀为txt的文件

```bash
fd --extension {{txt}}
```

在特定的目录内找符合模式的文件

```bash
fd {{pattern}} {{path/to/directory}}
```

查找时包括隐藏文件

```bash
fd --hidden --no-ignore {{pattern}}
```

对所有找到的文件执行命令，并返回结果

```bash
fd {{pattern}} --exec {{command}}
```

### fish

fish是一个功能齐舍，智能且对用户友好的Linux命令行Shell，它带有一些在大多数Shell中都不具备的方便功能。这些功能包括自动补全建议、Sane Scripting、手册页补全、基于Web的配置器和Glorious VGA Color。

### htop

htop是top的升级版，允许用户监视系统上运行的进程及其完整的命令行。

## 常用操作

### 硬件信息

**Free**

使用free命令来查看内存的使用情况，free命令产生内存使用表，利用man(manual操作面板)命令来查看free的使用描述和表头含义，利用man可以查看很多表的信息

关于free的基础信息可以看[Linux free命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-free.html)

还有一些有意思的东西[大佬们， Linux 内存情况看 free 还是看 available 呀-V2EX-非常论坛 (machbbs.com)](http://machbbs.com/v2ex/160267#:~:text=Linux 内存情况看 free 还是看 available 呀？ free 与,是应用程序认为可用内存数量，available %3D free %2B buffer %2B cache (注：只是大概的计算方法))

>available 是应用程序认为可用内存数量，available = free + buffer + cache (注：只是大概的计算方法)
>
>Linux 为了提升读写性能，会消耗一部分内存资源缓存磁盘数据，对于内核来说，buffer 和 cache 其实都属于已经被使用的内存。但当应用程序申请内存时，如果 free 内存不够，内核就会回收 buffer 和 cache 的内存来满足应用程序的请求。

**vmstat**

[Linux vmstat命令实战详解 - ggjucheng - 博客园 (cnblogs.com)](https://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html)

vmstat命令能够实时观看虚拟内存使用情况，非常有用的一个命令，可以看到机器整体使用的情况

> man vmstat  //查看表格各个参数的意思

**lshw**

lshw 这个命令是一个比较通用的工具，它可以详细的列出本机的硬件信息。但这个命令并非所有的发行版都有，比如 Fedora 就默认没有，a需要自己安装。

lshw 可以从各个 /proc 文件中提取出硬件信息，比如：CPU、内存、usb 控制器、硬盘等。如果不带选项的话，列出的信息将很长，加上 `-short` 选项时，将只列出概要信息。

~~~bash
[alvin@VM_0_16_centos ~]$ sudo lshw -short
#篇幅关系，以下结果有删减
H/W path            Device      Class          Description
==========================================================
                                system         Bochs
/0                              bus            Motherboard
/0/0                            memory         96KiB BIOS
/0/401                          processor      Intel(R) Xeon(R) CPU E5-26xx v4
/0/1000                         memory         2GiB System Memory
/0/1000/0                       memory         2GiB DIMM RAM
/0/100                          bridge         440FX - 82441FX PMC [Natoma]
/0/100/1                        bridge         82371SB PIIX3 ISA [Natoma/Triton II]
...
~~~

~~~bash
H/W path       Device     Class          Description
====================================================
                          system         30CBS0A400 (LENOVO_MT_30CB_BU_LENOVO_FM_ThinkStation P318)
/0/3e                     memory         16GiB System Memory
/0/48                     processor      Intel(R) Core(TM) i7-7700 CPU @ 3.60GHz
/0/100/1/0                display        GP106 [GeForce GTX 1060 6GB]
> uname -a
Linux BreezeShaneKXServer 5.17.9-arch1-1 #1 SMP PREEMPT Wed, 18 May 2022 17:30:11 +0000 x86_64 GNU/Linux
~~~



### 文件操作

**find**

[Linux find 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-find.html)

Linux find 命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。语法如下：

> find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;

find 根据下列规则判断 path 和 expression，在命令列上第一个 - ( ) , ! 之前的部份为 path，之后的是 expression。如果 path 是空字串则使用目前路径，如果 expression 是空字串则使用 -print 为预设 expression。下面记录一些常用的expression

-ipath p, -path p : 路径名称符合 p 的文件，ipath 会忽略大小写

-name name, -iname name : 文件名称符合 name 的文件。iname 会忽略大小写

-type c : 文件类型是 c 的文件。 

示例：

```shell
# 当前目录搜索所有文件，文件内容 包含 “140.206.111.111” 的内容
find . -type f -name "*" | xargs grep "140.206.111.111"
```

**根据文件或者正则表达式进行匹配**

**grep**

[Linux grep 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-grep.html)

grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 **-**，则 grep 指令会从标准输入设备读取数据。

grep命令十分复杂，功能也很强，用于查看各种文件和日志的信息，自带过滤功能。

**tail**

[Linux tail 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-tail.html)

tail 命令可用于查看文件的内容，有一个常用的参数 **-f** 常用于查阅正在改变的日志文件。

`tail -f filename` 会把 filename 文件里的最尾部的内容显示在屏幕上，并且不断刷新，只要 filename 更新就可以看到最新的文件内容。

**rm**

这个命令已经用了很久了，这里就记一下常用用法

`rm -rf /foldername`：删除文件夹

`rm -rf /foldername/*`：删除文件夹内所有文件

`rm -f /foldername/*.txt`：删除文件夹内所有txt文件

**mv**

这个命令已经用了很久了，这里就记一下常用用法

`mv webdata /bin/usr/`：移动文件夹

`mv /usr/lib/* /zone`：移动文件夹内所有文件

`mv /usr/lib/*.txt /zone`：移动文件夹内所有txt文件

**du**

[Linux du 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-du.html)

Linux du （英文全拼：disk usage）命令用于显示目录或文件的大小。

du 会显示指定的目录或文件所占用的磁盘空间。

- -a或-all 显示目录中个别文件的大小。
- -h或--human-readable 以K，M，G为单位，提高信息的可读性。
- -s或--summarize 仅显示总计。

`du -sh *`：统计当前文件夹下所有文件/文件夹的大小

### 进程相关

**Ps**

[Linux ps 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-ps.html)

ps（process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器，参数非常多，这里只记我用过的

> ps auxw	//查看当前所有进程按PID排序

**快捷键进程控制**

 shell 会使用 UNIX 提供的信号机制执行进程间通信。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。就这一点而言，信号是一种*软件中断*。

`Ctrl+C`：shell 会发送一个`SIGINT` 信号到进程，一般来讲这个信号会中断进程，但是如果代码有接收处理的话则不能中断

`Ctrl+\`：shell 会发送一个`SIGQUIT` 信号到进程，这个信号会直接中断进程

`Ctrl+Z`：shell 会发送一个`SIGTSTP` 信号到进程，这个信号会暂停进程，我们可以使用 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](http://man7.org/linux/man-pages/man1/bg.1p.html) 命令恢复暂停的工作。它们分别表示在前台继续或在后台继续。

**top/htop**

[linux下查询进程占用的内存方法总结 - wangmo - 博客园 (cnblogs.com)](https://www.cnblogs.com/wangmo/p/9486569.html)

**linux后台执行命令**

第一种方式：使用&和nohup

当在前台运行某个作业时，终端被该作业占据；可以在命令后面加上& 实现后台运行。例如：sh test.sh &，如果放在后台运行的作业会产生大量的输出，最好使用下面的方法把它的输出重定向到某个文件中：

~~~bash
command  >  out.file  2>&1  & #所有的标准输出和错误输出都将被重定向到一个叫做out.file 的文件中
~~~

使用&命令后，作业被提交到后台运行，当前控制台没有被占用，但是一但把当前控制台关掉(退出帐户时)，作业就会停止运行。nohup命令可以在你退出帐户之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)。

例如上面的作业就可以用这种方式来执行：

~~~bash
nohup command > myout.file 2>&1 &
~~~

其实这样有可能在当前账户非正常退出或者结束的时候，命令还是自己结束了。所以在使用nohup命令后台运行命令之后，需要使用exit正常退出当前账户，这样才能保证命令一直在后台运行。

第二种方式：bg, fg, job

简单的方法如下所示，这种bash退出时就会结束

> 假如你发现前天运行的一个程序需要很长的时间，但是需要干前天的事情，你就可以用ctrl-z挂起这个程序，然后可以看到系统的提示:
>
> [1]+ Stopped /root/bin/rsync.sh
>
> 然后我们可以吧程序调度到后台执行：（bg 作业号）
>
> bg 1
>
> [1]+ /root/bin/rsync.sh &
>
> 用jobs命令查看任务
>
> jobs
>
> [1]+ Running /root/bin/rsync.sh &
>
> 把它调回到控制台运行
>
> fg 1
>
> /root/bin/rsync.sh
>
> 这样，你这控制台上就只有等待这个任务完成了。



### 网络管理

[(32条消息) Linux中的网络管理——网络配置及命令_葡萄干是个程序员的博客-CSDN博客_linux网络配置](https://blog.csdn.net/qq_15096707/article/details/78420069)

**wget**

它支持HTTP、HTTPS，月以及FTP这三个常见的的TCP/IP协议下载。主要特点包括：

- 支持递归下载
- 恰当地转换页面中的连接
- 生成可在本地浏览的页面镜像
- 支持代理服务器

1. 使用wget下载单个文件：

~~~bash
wget https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.10/ubuntu-20.10-desktop-amd64.iso
~~~

会下载到当前文件夹（打开终端的所在文件夹）。在下载的过程中会显示进度条，包含（下载完成百分比，已经下载的字节，当前下载速度，剩余下载时间）。

2. 使用wget -O下载并以不同的文件名保存

wget默认会以最后一个符合'/'的后面的字符来命名，那么如果一个动态链接的下载通常会有问题。那么需要自己定义一个名字，下面是将上述的名字定义为ubuntu的下载：

```text
wget -O https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.10/ubuntu-20.10-desktop-amd64.iso
```

3. wget还可以限定速度下载

下面的命令是将下载速度限制在500k/s的速度：

```text
wget –limit-rate=500k https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.10/ubuntu-20.10-desktop-amd64.iso
```

4. 断续下载

在一般下在中，最让人头疼的是下载这网络不稳定，突然断网了，那么最好用下面的断续下在：

```text
wget -c https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.10/ubuntu-20.10-desktop-amd64.iso
```

5. 下载整个网站

如果你想下载某个不错的个人网站（当然该网站支持wget下载），而内容很多的情况下，不能逐个下载，那么就可以使用wget的递归功能来下载整个网站：

```text
wget -c -r -np -k -L -p https://***.org/path/
```

上述的https://***.org/path/是要下载的网址。

如果在该网址中，有一些需其它连接的数据， 就需要加一个-H参数，如下：

```text
wget -np -nH -R index https://***.org/path/
```

上述的https://***.org/path/是要下载的网址。下载完毕后，将会自动生成一个index.html的文件，打开文件就是需要下载的网址内容了。

- -r : 遍历所有子目录
- -np : 不到上一层子目录去
- -nH : 不要将文件保存到主机名文件夹
- -R index ： 不下载 index.html 文件，会自动生成index文件

### 其它

**swapon**

> enable/disable devices and files for paging and swapping

**journalctl**

journalctl命令来自于英文词组“journal control”的缩写，其功能是**用于查看指定的日志信息**。 在RHEL7/CentOS7及以后版本的Linux系统中，Systemd服务统一管理了所有服务的启动日志，带来的好处就是可以只用journalctl一个命令，查看到全部的日志信息了。

> journalctl //全部日志
>
> journalctl | grep sshd //有关ssh登录的日志
>
> journalctl | grep sshd | grep "Disconnected from"' | less //有关ssh登录失败的日志，且可翻页查看
>
> journalctl | grep sshd | grep "Disconnected from" | sed 's/.*Disconnected from //' //有关ssh登录失败的日志，且将Disconnected from内容删除

**unzip**

unzip 命令可以查看和解压缩 **zip** 文件。该命令的基本格式如下：

~~~shell
[root@localhost ~]# unzip [选项] zipfilename
~~~

此命令常用的选项以及各自的含义如表 1 所示。

| 选项      | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| -d 目录名 | 将压缩文件解压到指定目录下。                                 |
| -n        | 解压时并不覆盖已经存在的文件。                               |
| -o        | 解压时覆盖已经存在的文件，并且无需用户确认。                 |
| -v        | 查看压缩文件的详细信息，包括压缩文件中包含的文件大小、文件名以及压缩比等，但并不做解压操作。 |
| -t        | 测试压缩文件有无损坏，但并不解压。                           |

**校准系统时间**

有些Linux系统不知道为什么系统时间会和实际时间差几分钟，需要手动校准时间来正常使用代理等功能

~~~bash
sudo ntpdate cn.pool.ntp.org
~~~

**查看系统软件路径**

~~~bash
whereis program
~~~



# 增加磁盘和用户管理

