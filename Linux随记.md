# Shell

shell是一种负责人机交互的抽象，本身是一种脚本语言。而Bash是shell的一种实现，可以简单理解为bash命令激活自动化的shell脚本来对操作系统进行管理。在Linux系统中，bash最为常见，在Windows系统中，可以使用WSL或者GIT bash来使用bash命令

命令的参数如果只有一个字母用单横杠-，如果是一个单词用双横杠--，如果想知道有哪些命令可以用--help

## File operation

- ls
- pwd
- cd
- mkdir
- rmdir
- touch
- cp
- mv
- rm
- cat
- head
- tail
- grep
- |(pipe)

#### Free

使用free命令来查看内存的使用情况，free命令产生内存使用表，利用man(manual操作面板)命令来查看free的使用描述和表头含义，利用man可以查看很多表的信息

关于free的基础信息可以看[Linux free命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-free.html)

还有一些有意思的东西[大佬们， Linux 内存情况看 free 还是看 available 呀-V2EX-非常论坛 (machbbs.com)](http://machbbs.com/v2ex/160267#:~:text=Linux 内存情况看 free 还是看 available 呀？ free 与,是应用程序认为可用内存数量，available %3D free %2B buffer %2B cache (注：只是大概的计算方法))

>available 是应用程序认为可用内存数量，available = free + buffer + cache (注：只是大概的计算方法)
>
>Linux 为了提升读写性能，会消耗一部分内存资源缓存磁盘数据，对于内核来说，buffer 和 cache 其实都属于已经被使用的内存。但当应用程序申请内存时，如果 free 内存不够，内核就会回收 buffer 和 cache 的内存来满足应用程序的请求。

#### Ps

[Linux ps 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-ps.html)

ps（process status）命令用于显示当前进程的状态，类似于 windows 的任务管理器，参数非常多，这里只记我用过的

> ps auxw	//查看当前所有进程按PID排序

#### vmstat

[Linux vmstat命令实战详解 - ggjucheng - 博客园 (cnblogs.com)](https://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html)

vmstat命令能够实时观看虚拟内存使用情况，非常有用的一个命令，可以看到机器整体使用的情况

> man vmstat  //查看表格各个参数的意思

#### swapon

> enable/disable devices and files for paging and swapping

#### find

[Linux find 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-find.html)

Linux find 命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。语法如下：

> find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;

find 根据下列规则判断 path 和 expression，在命令列上第一个 - ( ) , ! 之前的部份为 path，之后的是 expression。如果 path 是空字串则使用目前路径，如果 expression 是空字串则使用 -print 为预设 expression。下面记录一些常用的expression

-ipath p, -path p : 路径名称符合 p 的文件，ipath 会忽略大小写

-name name, -iname name : 文件名称符合 name 的文件。iname 会忽略大小写

-type c : 文件类型是 c 的文件。

#### grep

[Linux grep 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-grep.html)

grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 **-**，则 grep 指令会从标准输入设备读取数据。

grep命令十分复杂，功能也很强，用于查看各种文件和日志的信息，自带过滤功能。

### wc

[Linux wc命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-comm-wc.html)



# tools

#### objdump

#### gdb

#### valgrind

#### purify

