对于大多数系统来说，不论其是否包含代码，都会包含一个“构建过程”。有时，您需要执行一系列操作。通常，这一过程包含了很多步骤，很多分支。执行一些命令来生成图表，然后执行另外的一些命令生成结果，然后再执行其他的命令来生成最终的论文。有很多事情需要我们完成，您并不是第一个因此感到苦恼的人，幸运的是，有很多工具可以帮助我们完成这些操作。

## Make

`make` 是最常用的构建系统之一，您会发现它通常被安装到了几乎所有基于UNIX的系统中。`make`并不完美，但是对于中小型项目来说，它已经足够好了。当您执行 `make` 时，它会去参考当前目录下名为 `Makefile` 的文件。所有构建目标、相关依赖和规则都需要在该文件中定义，它看上去是这样的：

```makefile
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

这个文件中的指令，即如何使用右侧文件构建左侧文件的规则。或者，换句话说，冒号左侧的是**构建目标**，冒号右侧的是构建它**所需的依赖**。缩进的部分是从依赖构建目标时需要用到的一段程序。在 `make` 中，第一条指令还指明了构建的目的，如果您使用不带参数的 `make`，这便是我们最终的构建结果。或者，您可以使用这样的命令来构建其他目标：`make plot-data.png`。

规则中的 `%` 是一种模式，它会匹配其**左右两侧相同**的字符串。例如，如果目标是 `plot-foo.png`， `make` 会去寻找 `foo.dat` 和 `plot.py` 作为依赖。现在，让我们看看如果在一个空的源码目录中执行`make` 会发生什么？

```bash
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```

`make` 会告诉我们，为了构建出`paper.pdf`，它需要 `paper.tex`，但是并没有一条规则能够告诉它如何构建该文件。让我们构建它吧！

```bash
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

哟，有意思，我们是**有**构建 `plot-data.png` 的规则的，但是这是一条模式规则。因为源文件`data.dat` 并不存在，因此 `make` 就会告诉您它不能构建 `plot-data.png`，让我们创建这些文件：

```bash
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

当我们执行 `make` 时会发生什么？

```
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

看！PDF ！

如果再次执行 `make` 会怎样？

```
$ make
make: 'paper.pdf' is up to date.
```

什么事情都没做！为什么？好吧，因为它什么都不需要做。make回去检查之前的构建是因其依赖改变而需要被更新。让我们试试修改 `paper.tex` 在重新执行 `make`：

```bash
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

注意 `make` 并**没有**重新构建 `plot.py`，因为没必要；`plot-data.png` 的所有依赖都没有发生改变。

## 包版本管理

很多语言的包管理工具的版本管理都是写在一个文件中，然后通过下载源码的方式进行自动化依赖构建，例如：

1. python的`requirements.txt`
2. go的`go.mod`和`go.sum`
3. rust的`Cargo.toml`

它们的版本指定都有一些特性，例如在`Cargo.toml`中`dependencies`的版本号指定是一个区间，而不是一个指定版本的包，例如：

```toml
[dependencies]
time = "0.1.12"
```

字符串`0.1.12`是版本要求。虽然它看起来像时间箱的特定版本，但它实际上指定了一系列版本，并允许SemVer兼容更新。如果新版本号未修改主要、次要、修补程序分组中最左边的非零位，则允许进行更新。在这种情况下，如果我们运行货物更新-p时间，如果它是最新的0.1.z版本，货物应该将我们更新到版本0.1.13，但不会将我们更新到0.2.0。如果我们将版本字符串指定为 1.0，则 cargo 应更新为 1.1（如果它是最新的 1.y 版本），而不是 2.0。版本 0.0.x 不被视为与任何其他版本兼容。

有一些例子可供参考，如下

~~~
1.2.3  :=  >=1.2.3, <2.0.0
1.2    :=  >=1.2.0, <2.0.0
1      :=  >=1.0.0, <2.0.0
0.2.3  :=  >=0.2.3, <0.3.0
0.2    :=  >=0.2.0, <0.3.0
0.0.3  :=  >=0.0.3, <0.0.4
~~~

## Git hooks

其实Git可以作为一个简单的 CI 系统来使用，在任何 git 仓库中的 `.git/hooks` 目录中，可以找到一些文件，它们的作用和脚本一样，当某些事件发生时便可以自动执行。

**pre-commit**

编写文件逻辑能够在commit时触发并进行自动化处理，例如编写一个`pre-commit`钩子，当执行make命令失败后，它会执行 make paper.pdf 并拒绝commit，这样做可以避免产生包含不可构建版本的提交信息；

做法就是修改`.git/hooks` 目录下面的`pre-commit.sample`文件并将其命名为`pre-commit`，然后在文件中编写

```
if  ! make ; then
     echo "build failed, commit rejected"
     exit 1
fi
```

就可以检查commit的合法性。















