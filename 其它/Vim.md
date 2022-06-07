## Vim概述

现在最流行的编辑器是什么？ [Stack Overflow的调查](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools)(这个调查可能并不如我们想象的那样客观，因为Stack Overflow 的用户并不能代表所有程序员 )显示， [Visual Studio Code](https://code.visualstudio.com/)是目前最流行的代码编辑器。而[Vim](https://www.vim.org/)则是最流行的基于命令行的编辑器。

Vim有着悠久历史；它始于1976年的Vi编辑器，到现在还在 不断开发中。Vim有很多聪明的设计思想，所以很多其他工具也支持Vim模式（比如，140万人安装了 [Vim emulation for VS code](https://github.com/VSCodeVim/Vim)）。即使你最后使用 其他编辑器，Vim也值得学习。

作为程序员，我们大部分时间都花在代码编辑上，所以花点时间掌握某个适合自己的编辑器是非常值得的。通常学习使用一个新的编辑器包含以下步骤：

- 阅读教程（比如这节课以及我们为您提供的资源）
- 坚持使用它来完成你所有的编辑工作（即使一开始这会让你的工作效率降低）
- 随时查阅：如果某个操作看起来像是有更方便的实现方法，一般情况下真的会有。

## Vim思想

在编程的时候，你会把大量时间花在阅读/编辑而不是在写代码上。所以，Vim是一个`多模态`编辑器。

* Vim对于插入文字和操纵文字有不同的模式。
* Vim 既是可编程的 （可以使用 Vimscript 或者像 Python 一样的其他程序语言），Vim 的接口本身也是一个程序语言
* Vim不用鼠标， Vim 甚至避免用上下左右键因为那样需要太多的手指移动。

这样的设计哲学的结果是一个能跟上你思维速度的编辑器

## Vim模式

Vim的设计以大多数时间都花在阅读、浏览和进行少量编辑改动为基础，因此它具有多种操作模式：

- 正常模式：在文件中四处移动光标进行修改
- 插入模式：插入文本
- 替换模式：替换文本
- 可视化（一般，行，块）模式：选中文本块
- 命令模式：用于执行命令

在不同的操作模式下， 键盘敲击的含义也不同。在默认设置下，Vim会在左下角显示当前的模式。 Vim启动时的默认模式是正常模式。通常你会把大部分 时间花在正常模式和插入模式。

你可以按下 `<ESC>` （逃脱键） 从任何其他模式返回正常模式。 在正常模式，键入 `i` 进入插入 模式， `R` 进入替换模式， `v` 进入可视（一般）模式， `V` 进入可视（行）模式， `<C-v>` （Ctrl-V, 有时也写作 `^V`）进入可视（块）模式， `:` 进入命令模式。

因为你会在使用 Vim 时大量使用 `<ESC>` 键，可以考虑把大小写锁定键（或者其它）重定义成`<ESC>` 键

## Vim基础

### 插入文本

和正常的写文本一样

### 缓存、标签页、窗口

* Vim 会维护一系列打开的文件，称为 “缓存”。
* 一个 Vim 会话包含一系列标签页，每个标签页包含 一系列窗口 （分隔面板）。每个窗口显示一个缓存。 
* 缓存和窗口不是一一对应的关系，窗口只是视角。一个缓存可以在多个窗口打开，甚至在同一 个标签页内的多个窗口打开。
  * 这个功能其实很好用， 比如在查看同一个文件的不同部分的时候。
* Vim 默认打开一个标签页，这个标签也包含一个窗口。

* 使用命令 `:set mouse=a` 启用鼠标支持，就可以通过点击鼠标来进入不同的窗口

### 命令行

在正常模式下键入 `:` 进入命令行模式。 在键入 `:` 后，你的光标会立即跳到屏幕下方的命令行。 这个模式有很多功能， 包括打开， 保存， 关闭文件， 以及 [退出 Vim](https://twitter.com/iamdevloper/status/435555976687923200)。

- `:q` 退出 （关闭窗口）
- `:w` 保存 （写）
- `:wq` 保存然后退出
- `:e {文件名}` 打开要编辑的文件
- `:ls` 显示打开的缓存
- `:help {标题}`打开帮助文档
  - `:help :w` 打开 `:w` 命令的帮助文档
  - `:help w` 打开 `w` 移动的帮助文档

## Vim 接口

Vim 最重要的设计思想是 Vim 的界面本省是一个程序语言。 键入操作 （以及他们的助记名） 本身是命令， 这些命令可以组合使用。 这使得移动和编辑更加高效，特别是一旦形成肌肉记忆。

### 移动

多数时候你会在正常模式下，使用移动命令在缓存中导航。在 Vim 里面移动也被成为 “名词”， 因为它们指向文字块。

- 基本移动: `hjkl` （左， 下， 上， 右）
- 词： `w` （下一个词）， `b` （词初）， `e` （词尾）
- 行： `0` （行初）， `^` （第一个非空格字符）， `$` （行尾）
- 屏幕： `H` （屏幕首行）， `M` （屏幕中间）， `L` （屏幕底部）
- 翻页： `Ctrl-u` （上翻）， `Ctrl-d` （下翻）
- 文件： `gg` （文件头）， `G` （文件尾）
- 行数： `:{行数}<CR>` 或者 `{行数}G` ({行数}为行数)
- 杂项： `%` （找到配对，比如括号或者 /* */ 之类的注释对）
- 查找： f{字符}，t{字符}，F{字符}，T{字符}
  - 查找/到 向前/向后 在本行的{字符}
  - `,` / `;` 用于导航匹配
- 搜索: `/{正则表达式}`, `n` / `N` 用于导航匹配

### 选择

可视化模式:

- 可视化
- 可视化行
- 可视化块

可以用移动命令来选中。

### 编辑

所有你需要用鼠标做的事， 你现在都可以用键盘：采用编辑命令和移动命令的组合来完成。 这就是 Vim 的界面开始看起来像一个程序语言的时候。Vim 的编辑命令也被称为 “动词”， 因为动词可以施动于名词。

- `i`进入插入模式
  - 但是对于操纵/编辑文本，不单想用退格键完成
- `o` / `O` 在之上/之下插入行
- `d{移动命令}` 删除 {移动命令}
  - 例如， `dw` 删除词, `d$` 删除到行尾, `d0` 删除到行头。
- `c{移动命令}`改变 {移动命令}
  - 例如， `cw` 改变词
  - 比如 `d{移动命令}` 再 `i`
- `x` 删除字符（等同于 `dl`）
- `s` 替换字符（等同于 `xi`）
- 可视化模式 + 操作
  - 选中文字, `d` 删除 或者 `c` 改变
- `u` 撤销
- `y` 复制 / “yank” （其他一些命令比如 `d` 也会复制）
- `p` 粘贴
- 更多值得学习的: 比如 `~` 改变字符的大小写

## 计数

你可以用一个计数来结合“名词” 和 “动词”， 这会执行指定操作若干次。比如：

- `3w` 向前移动三个词
- `5j` 向下移动5行
- `7dw` 删除7个词

## 修饰词

你可以用修饰语改变 “名词” 的意义。修饰语有 `i`， 表示 “内部” 或者 “在内“， 和 `a`， 表示 ”周围“。

- `ci(` 改变当前括号内的内容
- `ci[` 改变当前方括号内的内容
- `da'` 删除一个单引号字符窗， 包括周围的单引号

## 自定义VIM

Vim 由一个位于 `~/.vimrc` 的文本配置文件 （包含 Vim 脚本命令）。MIT 6.null课程提供了一个基本设置可供我们直接使用，在[这儿](https://missing-semester-cn.github.io/2020/files/vimrc) 下载设置， 然后将它保存成 `~/.vimrc`。

Vim 能够被重度自定义， 花时间探索自定义选项是值得的。 你可以参考其他人的在 GitHub 上共享的设置文件， 比如， 你的授课人的 Vim 设置 ([Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc), [Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.vim) (uses [neovim](https://neovim.io/)), [Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc))。 有很多好的博客文章也聊到了这个话题。 尽量不要复制粘贴别人的整个设置文件， 而是阅读和理解它， 然后采用对你有用的部分。

## 扩展Vim

Vim 有很多扩展插件。自Vim 8.0开始，你可以使用内置的插件管理系统。 只需要创建一个 `~/.vim/pack/vendor/start/` 的文件家， 然后把插件放到这里 （比如通过 `git clone`）。

举例一些很好的插件：

- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): 模糊文件查找
- [ack.vim](https://github.com/mileszs/ack.vim): 代码搜索
- [nerdtree](https://github.com/scrooloose/nerdtree): 文件浏览器
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): 魔术操作

## 其它程序的Vim模式

很多工具提供了 Vim 模式。 这些 Vim 模式的质量参差不齐； 取决于具体工具， 有的提供了 很多酷炫的 Vim 功能， 但是大多数只对基本功能支持的很好。

### Shell

如果你是一个 Bash 用户， 用 `set -o vi`。 如果你用 Zsh： `bindkey -v`。 Fish 用 `fish_vi_key_bindings`。 另外， 不管利用什么 shell， 你可以 `export EDITOR=vim`。 这是一个用来决定当一个程序需要启动编辑时启动哪个的环境变量。 例如， `git` 会使用这个编辑器来编辑 commit 信息。

### Readline

很多程序使用 [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) 库来作为 它们的命令控制行界面。 Readline 也支持基本的 Vim 模式， 可以通过在 `~/.inputrc` 添加如下行开启：

```python
set editing-mode vi
```

比如， 在这个设置下， Python REPL 会支持 Vim 快捷键。

### 其它

甚至有 Vim 的网页浏览快捷键 [browsers](http://vim.wikia.com/wiki/Vim_key_bindings_for_web_browsers), 受欢迎的有 用于 Google Chrome 的 [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) 和用于 Firefox 的 [Tridactyl](https://github.com/tridactyl/tridactyl)。 你甚至可以在 [Jupyter notebooks](https://github.com/lambdalisue/jupyter-vim-binding) 中用 Vim 快捷键。

## Vim进阶

这里我们提供了一些展示这个编辑器能力的例子。你们应当在使用中学习。 一个好的对策是: 当你在使用你的编辑器的时候感觉 “一定有更好的方法来做这个”， 那么很可能真的有： 上网搜寻一下。

### 搜索和替换

`:s` （替换） 命令 （[文档](http://vim.wikia.com/wiki/Search_and_replace)）。

- ```
  %s/foo/bar/g
  ```

  - 在整个文件中将 foo 全局替换成 bar

- ```
  %s/\[.*\](\(.*\))/\1/g
  ```

  - 将有命名的 Markdown 链接替换成简单 URLs

### 多窗口

- 用 `:sp` / `:vsp` 来分割窗口
- 同一个缓存可以在多个窗口中显示。

### 宏（Macros）

- `q{字符}` 来开始在寄存器 `{字符}` 中录制宏
- `q` 停止录制
- `@{字符}` 重放宏
- 宏的执行遇错误会停止
- `{计数}@{字符}` 执行一个宏 {计数} 次
- 宏可以递归
  - 首先用 `q{字符}q` 清除宏
  - 录制该宏， 用 `@{字符}` 来递归调用该宏 （在录制完成之前不会有任何操作）
- 例子： 将 xml 转成 json ([file](https://missing-semester-cn.github.io/2020/files/example-data.xml))
  - 一个有 “name” / “email” 键对象的数组
  - 用一个 Python 程序？
  - 用 sed / 正则表达式
    - `g/people/d`
    - `%s/<person>/{/g`
    - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
    - …
  - Vim 命令 / 宏
    - `Gdd`, `ggdd` 删除第一行和最后一行
    - 格式化最后一个元素的宏 （寄存器 e）
      - 跳转到有 `<name>` 的行
      - `qe^r"f>s": "<ESC>f<C"<ESC>q`
    - 格式化一个人的宏
      - 跳转到有 `<person>` 的行
      - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
    - 格式化一个人然后转到另外一个人的宏
      - 跳转到有 `<person>` 的行
      - `qq@pjq`
    - 执行宏到文件尾
      - `999@q`
    - 手动移除最后的 `,` 然后加上 `[` 和 `]` 分隔符

## 扩展资料

- `vimtutor` 是一个 Vim 安装时自带的教程
- [Vim Adventures](https://vim-adventures.com/) 是一个学习使用 Vim 的游戏
- [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) 有很多 Vim 小技巧
- [Vim Golf](http://www.vimgolf.com/) 是用 Vim 的用户界面作为程序语言的 [code golf](https://en.wikipedia.org/wiki/Code_golf)
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/book/dnvim2/practical-vim-second-edition) （书）











































