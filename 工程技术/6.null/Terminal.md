学习如何同时**执行多个不同的进程**并**追踪它们的状态**、如何停止或**暂停某个进程**以及如何使进程在后台运行。学习一些能够**改善您的 shell 及其他工具的工作流的方法**，这主要是通过**定义别名**或**基于配置文件对其进行配置**来实现的。这些方法都可以帮您节省大量的时间。例如，仅需要执行一些简单的命令，我们就可以在所有的主机上使用相同的配置。我们还会学习如何使用 SSH 操作远端机器。

# 任务控制

## 结束进程

大多数情况下，我们可以使用 `Ctrl-C` 来停止命令的执行。但是它的工作原理是什么呢？为什么有的时候会无法结束进程？您的 shell 会使用 UNIX 提供的信号机制执行进程间通信。当一个进程接收到信号时，它会停止执行、处理该信号并基于信号传递的信息来改变其执行。就这一点而言，信号是一种`software interruption`。

当我们输入 `Ctrl-C` 时，shell 会发送一个`SIGINT` 信号到进程。一般来说，进程接收到`SIGINT` 信号就会停止执行，但是也可以在代码中定义捕获信号`SIGINT` 并忽略它。

~~~python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
~~~

此时为了停止这个程序，我们需要使用`SIGQUIT` 信号，通过输入`Ctrl-\`可以发送该信号。

尽管 `SIGINT` 和 `SIGQUIT` 都常常用来发出和终止程序相关的请求。`SIGTERM` 则是一个更加通用的、也更加优雅地退出信号。为了发出这个信号我们需要使用 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 命令, 它的语法是： `kill -TERM <PID>`。

## 暂停和后台执行进程

信号可以让进程做其他的事情，而不仅仅是终止它们。例如，`SIGSTOP` 会让进程暂停。在终端中，键入 `Ctrl-Z` 会让 shell 发送 `SIGSTOP` 信号，`SIGSTOP`是 Terminal Stop 的缩写（即`terminal`版本的SIGSTOP）。对于暂停的任务，有以下几种命令可以使用

`fg`：前台恢复暂停的工作

`bg`：后台恢复暂停的工作

`jobs`：列出当前终端会话中尚未完成的全部任务

命令中的 `&` 后缀可以让命令在直接在后台运行，这使得您可以直接在 shell 中继续做其他操作，不过它此时还是会使用 shell 的标准输出，这一点有时会比较恼人（这种情况可以使用 **shell 重定向**处理）。

让已经在运行的进程转到后台运行，您可以键入`Ctrl-Z` ，然后紧接着再输入`bg`。注意，后台的进程仍然是您的终端进程的子进程，一旦您关闭终端（会发送另外一个信号`SIGHUP`），这些后台的进程也会终止。为了防止这种情况发生，您可以使用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (一个用来忽略 `SIGHUP` 的封装) 来运行程序。针对已经运行的程序，可以使用`disown` 。

# 终端多路复用

用 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 这类的终端多路复用器，[Tmux使用手册 | louis blog (louiszhai.github.io)](http://louiszhai.github.io/2017/09/30/tmux/#关闭会话)

这里记录一些最基础的用法

```
创建并命名一个新会话：tmux new-session -s <会话名称>
重命名一个会话：tmux rename-session [-t current-name] [new-name]
显示所有会话：tmux ls
连接一个会话：tmux attach -a <会话名称>
退出会话：先按下组合键 [Ctrl+b] 然后再接着按 [d]
```

会话内操作

~~~
Ctrl+b %：划分左右两个窗格。
Ctrl+b "：划分上下两个窗格。
Ctrl+b <arrow key>：光标切换到其他窗格。<arrow key>是指向要切换到的窗格的方向键，比如切换到下方窗格，就按方向键↓。
Ctrl+b x：关闭当前窗格。
Ctrl+b !：将当前窗格拆分为一个独立窗口。
Ctrl+b z：当前窗格全屏显示，再使用一次会变回原来大小。
按完前缀ctrl+B后，再按冒号：进入命令行模式，
输入命令：set -g mouse on
~~~

# 别名

注意， `=`两边是没有空格的，因为 [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) 是一个 shell 命令，它只接受一个参数。

别名有许多很方便的特性:

```bash
# 创建常用命令的缩写
alias ll="ls -lh"

# 能够少输入很多
alias gs="git status"
alias gc="git commit"
alias v="vim"

# 手误打错命令也没关系
alias sl=ls

# 重新定义一些命令行的默认行为
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# 别名可以组合使用
alias la="ls -A"
alias lla="la -l"

# 在忽略某个别名
\ls
# 或者禁用别名
unalias la

# 获取别名的定义
alias ll
# 会打印 ll='ls -lh'
```

值得注意的是，在默认情况下 shell 并不会保存别名。为了让别名持续生效，您需要将配置放进 shell 的启动文件里，像是`.bashrc` 或 `.zshrc`

# 配置文件（Dotfiles）

shell 的配置也是通过这类文件完成的。在启动时，您的 shell 程序会读取很多文件以加载其配置项。根据 shell 本身的不同，您从登录开始还是以交互的方式完成这一过程可能会有很大的不同。关于这一话题，[这里](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) 有非常好的资源

对于 `bash`来说，在大多数系统下，您可以通过编辑 `.bashrc` 或 `.bash_profile` 来进行配置。在文件中您可以添加需要在启动时执行的命令，例如上文我们讲到过的别名，或者是您的环境变量。

实际上，很多程序都要求您在 shell 的配置文件中包含一行类似 `export PATH="$PATH:/path/to/program/bin"` 的命令，这样才能确保这些程序能够被 shell 找到。

还有一些其他的工具也可以通过*点文件*进行配置：

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` 和 `~/.vim` 目录
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

我们应该如何管理这些配置文件呢，它们应该在它们的文件夹下，并使用版本控制系统进行管理，然后通过脚本将其 **符号链接** 到需要的地方。这么做有如下好处：

- **安装简单**: 如果您登录了一台新的设备，在这台设备上应用您的配置只需要几分钟的时间；
- **可以执行**: 您的工具在任何地方都以相同的配置工作
- **同步**: 在一处更新配置文件，可以同步到其他所有地方
- **变更追踪**: 您可能要在整个程序员生涯中持续维护这些配置文件，而对于长期项目而言，版本历史是非常重要的

配置文件中需要放些什么？您可以通过在线文档和[帮助手册](https://en.wikipedia.org/wiki/Man_page)了解所使用工具的设置项。另一个方法是在网上搜索有关特定程序的文章，作者们在文章中会分享他们的配置。还有一种方法就是直接浏览其他人的配置文件：您可以在这里找到无数的[dotfiles 仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) —— 其中最受欢迎的那些可以在[这里](https://github.com/mathiasbynens/dotfiles)找到（我们建议您不要直接复制别人的配置）。[这里](https://dotfiles.github.io/) 也有一些非常有用的资源。

## 可移植性

配置文件的一个常见的痛点是它可能并不能在多种设备上生效。例如，如果您在不同设备上使用的操作系统或者 shell 是不同的，则配置文件是无法生效的。或者，有时您仅希望特定的配置只在某些设备上生效。

有一些技巧可以轻松达成这些目的。如果配置文件 if 语句，则您可以借助它针对不同的设备编写不同的配置。例如，您的 shell 可以这样做：

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# 使用和 shell 相关的配置时先检查当前 shell 类型
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# 您也可以针对特定的设备进行配置
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

如果配置文件支持 include 功能，您也可以多加利用。例如：`~/.gitconfig` 可以这样编写：

```bash
[include]
    path = ~/.gitconfig_local
```

然后我们可以在日常使用的设备上创建配置文件 `~/.gitconfig_local` 来包含与该设备相关的特定配置。您甚至应该创建一个单独的代码仓库来管理这些与设备相关的配置。

如果您希望在不同的程序之间共享某些配置，该方法也适用。例如，如果您想要在 `bash` 和 `zsh` 中同时启用一些别名，您可以把它们写在 `.aliases` 里，然后在这两个 shell 里应用：

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# SSH

## ssh连接服务器

和其他工具一样，SSH 也是可以高度定制的，也值得我们花时间学习它。

通过如下命令，您可以使用 `ssh` 连接到其他服务器：

```
ssh foo@bar.mit.edu
```

这里我们尝试以用户名 `foo` 登录服务器 `bar.mit.edu`。服务器可以通过 URL 指定（例如`bar.mit.edu`），也可以使用 IP 指定（例如`foobar@192.168.1.42`）。后面我们会介绍如何修改 ssh 配置文件使我们可以用类似 `ssh bar` 这样的命令来登录服务器。（利用别名）

## 执行命令

`ssh` 的一个经常被忽视的特性是它可以直接远程执行命令。 `ssh foobar@server ls` 可以直接在用foobar的命令下执行 `ls` 命令。 想要配合管道来使用也可以， `ssh foobar@server ls | grep PATTERN` 会在本地查询远端 `ls` 的输出而 `ls | ssh foobar@server grep PATTERN` 会在远端对本地 `ls` 输出的结果进行查询。

## SSH 密钥

基于密钥的验证机制使用了密码学中的公钥，我们只需要向服务器证明客户端持有对应的私钥，而不需要公开其私钥。这样您就可以避免每次登录都输入密码的麻烦了秘密就可以登录。不过，私钥(通常是 `~/.ssh/id_rsa` 或者 `~/.ssh/id_ed25519`) 等效于您的密码，所以一定要好好保存它。

### 基于密钥的认证机制

`ssh` 会查询 `.ssh/authorized_keys` 来确认那些用户可以被允许登录。您可以通过下面的命令将一个公钥拷贝到这里：

```
cat .ssh/id_ed25519 | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

如果支持 `ssh-copy-id` 的话，可以使用下面这种更简单的解决方案：

```
ssh-copy-id -i .ssh/id_ed25519.pub foobar@remote
```

## 通过 SSH 复制文件

使用 ssh 复制文件有很多方法：

- `ssh+tee`, 最简单的方法是执行 `ssh` 命令，然后通过这样的方法利用标准输入实现 `cat localfile | ssh remote_server tee serverfile`。回忆一下，[`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) 命令会将标准输出写入到一个文件；
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) ：当需要拷贝大量的文件或目录时，使用`scp` 命令则更加方便，因为它可以方便的遍历相关路径。语法如下：`scp path/to/local_file remote_host:path/to/remote_file`；
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 对 `scp` 进行了改进，它可以检测本地和远端的文件以防止重复拷贝。它还可以提供一些诸如符号连接、权限管理等精心打磨的功能。甚至还可以基于 `--partial`标记实现断点续传。`rsync` 的语法和`scp`类似；

## 端口转发

很多情况下我们都会遇到软件需要监听特定设备的端口。如果是在您的本机，可以使用 `localhost:PORT` 或 `127.0.0.1:PORT`。但是如果需要监听远程服务器的端口该如何操作呢？这种情况下远端的端口并不会直接通过网络暴露给您。

具体的应用场景、原理、方法在文章里：[SSH 端口转发-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/372995#:~:text=SSH 端口转发功能能够将其他 TCP 端口的网络数据通过 SSH 链接来转发，并且自动提供了相应的加密及解密服务。,其实这一技术就是我们常常听说的隧道 (tunnel)技术，原因是 SSH 为其他 TCP 链接提供了一个安全的通道来进行传输。)

## SSH 配置

我们已经介绍了很多参数。为它们创建一个别名是个好想法，我们可以这样做：

```
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

不过，更好的方法是使用 `~/.ssh/config`.

```
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# 在配置文件中也可以使用通配符
Host *.mit.edu
    User foobaz
```

这么做的好处是，使用 `~/.ssh/config` 文件来创建别名，类似 `scp`、`rsync`和`mosh`的这些命令都可以读取这个配置并将设置转换为对应的命令行选项。

注意，`~/.ssh/config` 文件也可以被当作配置文件，而且一般情况下也是可以被导入其他配置文件的。不过，如果您将其公开到互联网上，那么其他人都将会看到您的服务器地址、用户名、开放端口等等。这些信息可能会帮助到那些企图攻击您系统的黑客，所以请务必三思。

服务器侧的配置通常放在 `/etc/ssh/sshd_config`。您可以在这里配置免密认证、修改 ssh 端口、开启 X11 转发等等。 您也可以为每个用户单独指定配置。























