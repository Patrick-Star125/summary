# Git

**git挂代理**

结合以上各位经验设置成功. 以下以macOS为准.

1. https访问（这个实测有用）
   仅为github.com设置socks5代理(推荐这种方式, 公司内网就不用设代理了, 多此一举):
   `git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`
   其中1086是socks5的监听端口, 这个可以配置的, 每个人不同, 在macOS上一般为1086.
   设置完成后, ~/.gitconfig文件中会增加以下条目:

   ```json
   [http "https://github.com"]
       proxy = socks5://127.0.0.1:1086  \\这里改自己的端口
   ```

2. ssh访问
   需要修改~/.ssh/config文件, 没有的话新建一个. 同样仅为github.com设置代理:

   ```json
   Host github.com
       User git
       ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
   ```

   如果是在Windows下, 则需要个性%home%.ssh\config, 其中内容类似于:

   ```json
   Host github.com
       User git
       ProxyCommand connect -S 127.0.0.1:1086 %h %p
   ```

   这里-S表示使用socks5代理, 如果是http代理则为-H. connect工具git自带, 在\mingw64\bin\下面.

## 核心概念

**远程主机：**

**分支：**本地分支、远程分支

## 指令详解

已经用过的所有指令

>git add --all
>
>git status
>
>git commit -m [message]
>
>git push 远程主机名 本地分支名
>
>git remote -v
>
>git remote add 远程主机名 -url
>
>git remote remove 远程主机名
>
>git branch
>
>git branch -a
>
>git fetch 远程主机名
>
>git merge 远程主机名\远程分支
>
>git pull
>
>git reset  --hard my/master
>
>git reset
>
>git log

###  branch

[git branch命令 - Git教程™ (yiibai.com)](https://www.yiibai.com/git/git_branch.html)

`git branch`命令用于列出，创建或删除分支。

如果给出了`--list`，或者如果没有非选项参数，则列出现有的分支; 当前分支将以星号突出显示。 选项`-r`导致远程跟踪分支被列出，而选项`-a`显示本地和远程分支。 如果给出了一个`<pattern>`，它将被用作一个shell通配符，将输出限制为匹配的分支。 如果给出多个模式，如果匹配任何模式，则显示分支。 请注意，提供`<pattern>`时，必须使用`--list`; 否则命令被解释为分支创建。

使用`--contains`，仅显示包含命名提交的分支(换句话说，提示提交的分支是指定的提交的后代)，`--no-contains`会反转它。 随着已经有了，只有分支合并到命名提交(即从提交提交可以提前提交的分支)将被列出。 使用`--no`合并只会将未合并到命名提交中的分支列出。 如果缺少`<commit>`参数，则默认为`HEAD`(即当前分支的提示)。

>remotes/origin/HEAD 是**远程命名 origin 的 default branch** 。 这可以让你简单地说是 origin ，而不是 origin/master 。

### reset

[【原】git如何撤销commit(未push) ](https://www.cnblogs.com/PeunZhang/p/11649910.html)

git reset用于撤销commit和add（用不同的参数）

**1.**使用参数--mixed(默认参数)，如git reset --mixed <commit ID>或git reset <commit ID>

撤销git commit，撤销git add，保留编辑器改动代码

**2.**使用参数--soft，如git reset --soft<commit ID> 

撤销git commit，不撤销git add，保留编辑器改动代码

**3.**使用参数--hard，如git reset --hard <commit ID>——此方式非常暴力，全部撤销，慎用

撤销git commit，撤销git add，删除编辑器改动代码

用git log打印出最近的push信息

~~~bash
git reset --hard HEAD^ #从上一次提交的版本退回
git reset --hard e09af7ae711e2a79c15144c1e792fb2e27d201ff #退回某一次版本
~~~

### 其它

[(35条消息) 【git】强制覆盖本地代码（与git远程仓库保持一致）_不才Jerry的博客-CSDN博客_git 远程覆盖本地](https://blog.csdn.net/sinat_36184075/article/details/80115000)

如果改了文件但是不知道改了哪些，add的时候可能会比较烦，可以用下面的强制覆盖代码

~~~git
git fetch --all
git reset --hard origin/master
~~~

## Git 本地开发流程

本文将指导您如何在本地进行代码开发

**代码要求**

- 代码注释请遵守 [Doxygen](http://www.doxygen.nl/) 的样式。
- 确保编译器选项 `WITH_STYLE_CHECK` 已打开，并且编译能通过代码样式检查。
- 所有代码必须具有单元测试。
- 通过所有单元测试。
- 请遵守[提交代码的一些约定](https://www.paddlepaddle.org.cn/documentation/docs/zh/guides/10_contribution/local_dev_guide_cn.html#提交代码的一些约定)。

以下教程将指导您提交代码。

[**复刻（Fork）**](https://help.github.com/articles/fork-a-repo/)

跳转到[PaddlePaddle](https://github.com/PaddlePaddle/Paddle) GitHub首页，然后单击 `Fork` 按钮，生成自己目录下的仓库，比如 https://github.com/USERNAME/Paddle。

**克隆（Clone）**

将远程仓库 clone 到本地：

```
➜  git clone https://github.com/USERNAME/Paddle
➜  cd Paddle
```

**创建本地分支**

Paddle 目前使用[Git流分支模型](http://nvie.com/posts/a-successful-git-branching-model/)进行开发，测试，发行和维护，具体请参考 [Paddle 分支规范](https://github.com/PaddlePaddle/FluidDoc/blob/develop/doc/fluid/design/others/releasing_process.md)。

所有的 feature 和 bug fix 的开发工作都应该在一个新的分支上完成，一般从 `develop` 分支上创建新分支。

使用 `git checkout -b` 创建并切换到新分支。

```
➜  git checkout -b my-cool-stuff
```

值得注意的是，在 checkout 之前，需要保持当前分支目录 clean，否则会把 untracked 的文件也带到新分支上，这可以通过 `git status` 查看。

**使用 `pre-commit` 钩子**

Paddle 开发人员使用 [pre-commit](http://pre-commit.com/) 工具来管理 Git 预提交钩子。 它可以帮助我们格式化源代码（C++，Python），在提交（commit）前自动检查一些基本事宜（如每个文件只有一个 EOL，Git 中不要添加大文件等）。

`pre-commit`测试是 Travis-CI 中单元测试的一部分，不满足钩子的 PR 不能被提交到 Paddle，首先安装并在当前目录运行它：

```
➜  pip install pre-commit
➜  pre-commit install
```

Paddle 使用 `clang-format` 来调整 C/C++ 源代码格式，请确保 `clang-format` 版本在 3.8 以上。

注：通过`pip install pre-commit`和`conda install -c conda-forge pre-commit`安装的`yapf`稍有不同的，Paddle 开发人员使用的是`pip install pre-commit`。

**开始开发**

在本例中，我删除了 README.md 中的一行，并创建了一个新文件。

通过 `git status` 查看当前状态，这会提示当前目录的一些变化，同时也可以通过 `git diff` 查看文件具体被修改的内容。

```
➜  git status
On branch test
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	test

no changes added to commit (use "git add" and/or "git commit -a")
```

**编译和单元测试**

关于编译 PaddlePaddle 的源码，请参见[从源码编译](https://www.paddlepaddle.org.cn/documentation/docs/install/compile/fromsource.html) 选择对应的操作系统。 关于单元测试，可参考[Op单元测试](https://www.paddlepaddle.org.cn/documentation/docs/zh/guides/07_new_op/new_op.html#id7)的运行方法。

**提交（commit）**

接下来我们取消对 README.md 文件的改变，然后提交新添加的 test 文件。

```
➜  git checkout -- README.md
➜  git status
On branch test
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	test

nothing added to commit but untracked files present (use "git add" to track)
➜  git add test
```

Git 每次提交代码，都需要写提交说明，这可以让其他人知道这次提交做了哪些改变，这可以通过`git commit` 完成。

```
➜  git commit
CRLF end-lines remover...............................(no files to check)Skipped
yapf.................................................(no files to check)Skipped
Check for added large files..............................................Passed
Check for merge conflicts................................................Passed
Check for broken symlinks................................................Passed
Detect Private Key...................................(no files to check)Skipped
Fix End of Files.....................................(no files to check)Skipped
clang-formater.......................................(no files to check)Skipped
[my-cool-stuff c703c041] add test file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 233
```

**保持本地仓库最新**

在准备发起 Pull Request 之前，需要同步原仓库（https://github.com/PaddlePaddle/Paddle）最新的代码。

首先通过 `git remote` 查看当前远程仓库的名字。

```
➜  git remote
origin
➜  git remote -v
origin	https://github.com/USERNAME/Paddle (fetch)
origin	https://github.com/USERNAME/Paddle (push)
```

这里 origin 是我们 clone 的远程仓库的名字，也就是自己用户名下的 Paddle，接下来我们创建一个原始 Paddle 仓库的远程主机，命名为 upstream。

```
➜  git remote add upstream https://github.com/PaddlePaddle/Paddle
➜  git remote
origin
upstream
```

获取 upstream 的最新代码并更新当前分支。

```
➜  git fetch upstream
➜  git pull upstream develop
```

**Push 到远程仓库**

将本地的修改推送到 GitHub 上，也就是 https://github.com/USERNAME/Paddle。

```
# 推送到远程仓库 origin 的 my-cool-stuff 分支上
➜  git push origin my-cool-stuff
```

**通过单元测试**

您在Pull Request中每提交一次新的commit后，会触发CI单元测试，请确认您的commit message中已加入必要的说明，请见[提交（commit）](https://www.paddlepaddle.org.cn/documentation/docs/zh/guides/10_contribution/local_dev_guide.html#permalink-8--commit-)

请您关注您Pull Request中的CI单元测试进程，它将会在几个小时内完成

您仅需要关注和自己提交的分支相关的CI项目，例如您向develop分支提交代码，则无需关注release/1.1一栏是否通过测试

当所需的测试后都出现了绿色的对勾，表示您本次commit通过了各项单元测试

如果所需的测试后出现了红色叉号，代表您本次的commit未通过某项单元测试，在这种情况下，请您点击detail查看报错详情，并将报错原因截图，以评论的方式添加在您的Pull Request中，我们的工作人员将帮您查看

**删除远程分支**

在 PR 被 merge 进主仓库后，我们可以在 PR 的页面删除远程仓库的分支。

![img](https://githubraw.cdn.bcebos.com/PaddlePaddle/FluidDoc/develop/doc/paddle/guides/08_contribution/img/delete_branch.png?raw=true)

也可以使用 `git push origin :分支名` 删除远程分支，如：

```
➜  git push origin :my-cool-stuff
```

**删除本地分支**

最后，删除本地分支。

```
# 切换到 develop 分支
➜  git checkout develop

# 删除 my-cool-stuff 分支
➜  git branch -D my-cool-stuff
```

至此，我们就完成了一次代码贡献的过程。

## PR提交注意

**提交代码的一些约定**

为了使评审人在评审代码时更好地专注于代码本身，请您每次提交代码时，遵守以下约定：

1）请保证Travis-CI 中单元测试能顺利通过。如果没过，说明提交的代码存在问题，评审人一般不做评审。

2）提交PUll Request前：

- 请注意commit的数量：

原因：如果仅仅修改一个文件但提交了十几个commit，每个commit只做了少量的修改，这会给评审人带来很大困扰。评审人需要逐一查看每个commit才能知道做了哪些修改，且不排除commit之间的修改存在相互覆盖的情况。

建议：每次提交时，保持尽量少的commit，可以通过`git commit --amend`补充上次的commit。对已经Push到远程仓库的多个commit，可以参考[squash commits after push](http://stackoverflow.com/questions/5667884/how-to-squash-commits-in-git-after-they-have-been-pushed)。

- 请注意每个commit的名称：应能反映当前commit的内容，不能太随意。

3）如果解决了某个Issue的问题，请在该PUll Request的**第一个**评论框中加上：`fix #issue_number`，这样当该PUll Request被合并后，会自动关闭对应的Issue。关键词包括：close, closes, closed, fix, fixes, fixed, resolve, resolves, resolved，请选择合适的词汇。详细可参考[Closing issues via commit messages](https://help.github.com/articles/closing-issues-via-commit-messages)。

此外，在回复评审人意见时，请您遵守以下约定：

1）评审人的每个意见都必须回复（这是开源社区的基本礼貌，别人帮了忙，应该说谢谢）：

- 对评审意见同意且按其修改完的，给个简单的`Done`即可；
- 对评审意见不同意的，请给出您自己的反驳理由。

2）如果评审意见比较多：

- 请给出总体的修改情况。
- 请采用[start a review](https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/)进行回复，而非直接回复的方式。原因是每个回复都会发送一封邮件，会造成邮件灾难。



















































**服务器**

基础构建

Windows power shell ssh登陆服务器

~~~bash
ssh -p 22016 root@10.12.0.87 -o ServerAliveInterval=60 #GPUservers'd
~~~

```bash
ssh root@1.14.100.228 -o ServerAliveInterval=60	#腾讯云
```

~~~bash
ssh root@122.9.145.200 -o ServerAliveInterval=60 #华为云1
ssh root@116.63.148.146 -o ServerAliveInterval=60 #华为云2
ssh root@122.9.162.62 -o ServerAliveInterval=60 #华为云3
~~~

