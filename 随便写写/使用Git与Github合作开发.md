因为一些需要，我想总结一下很多新手上手使用GitHub时不熟悉的合作部分的流程。

## 概述

使用Git在GitHub上进行合作的操作其实很简单，重要的是在合作开发时我们需要理解的一些核心概念，我总结为：

* 审核人与贡献者
* commit与merge
* 本地仓库、云端fork仓库、云端项目仓库

在合作开发项目时，代码总是要经历 push（提交pr）-review（审查代码）-merge（合并入库）三个阶段，贡献者编写代码push至云端仓库之后就会进入review阶段，比较规范的仓库的审核人会在这一阶段查看贡献者的代码是否有问题（一般是有的），如果存在问题则直接指出并提醒贡献者修改，这往往会让贡献者在coding-push之间反复横跳，调整代码，直至审核人觉得功能完善，没有冲突，很ok，就将其并入仓库，一次贡献就算是结束了。

![](http://1.14.100.228:8002/images/2022/05/20/20220520200136.png)

上述是在简化条件的情况下的贡献流程，实际中很多项目会有单元测试、文档补充等等流程，这些因项目不同而各异，因此这里就不多说了。

贡献者在贡献代码时，一次提交算一次commit，一次commit后贡献者提交的代码中所有对原仓库代码的更改都会被保留下来，这也是审核人检查代码的地方，他们凭其对项目的理解，直接在代码语句上标注需要修改的地方，贡献者修改后再次commit，直到审核人觉得可以了，就将最新的commit合入代码库中。

另外，如果贡献者觉得这次commit还不如上一次commit，就可以进行版本回退，放弃此次commit（减轻审核人的压力）

![](http://1.14.100.228:8002/images/2022/05/20/20220520200622.png)

贡献者想要贡献某一个项目时，需要先fork这个项目的所有代码，fork仓库会在贡献者的账号里面创建一个私有仓库，里面所有代码是贡献者fork时项目的最新代码，这相当于这个项目的最新版本，之后贡献者在本地更改代码，push到自己的fork仓库后，相当于fork仓库和项目仓库在此刻不一致了，导致不一致的更改就可以作为一次pr的开始，之后对fork仓库的push都算在本次pr上的修改，commit因此叠加起来，形成上图所示的结构。

## 流程

理论说再多不如实际操作一下，以下的git命令我不会写清除含义和作用，这里只演示过程。

[**复刻（Fork）**](https://help.github.com/articles/fork-a-repo/)

首先到项目的GitHub主页，然后单击 `Fork` 按钮，生成自己目录下的仓库，比如 https://github.com/USERNAME/ProjectName。

**克隆（Clone）**

将远程仓库 clone 到本地：

```bash
➜  git clone https://github.com/USERNAME/ProjectName
➜  cd ProjectName
```

**开始开发**

在本例中，我删除了项目中 README.md 中的一行，并创建了一个新文件。

通过 `git status` 查看当前状态，这会提示当前目录的一些变化，同时也可以通过 `git diff` 查看文件具体被修改的内容。

~~~bash
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
~~~

这里显示README.md被更改

**同步仓库**

在实际开发完成和第一次commit之间可能差了不少时间，因此此时云端fork仓库和项目仓库可能已经不同步了，需要先同步仓库到最新状态，例如此时有人已经对README.md进行了一些更改，GitHub上会显示有n个commit ahead于fork仓库，我们的同步过程为

~~~bash
➜  git remote add remotename https://github.com/Project/ProjectName.git # 链接远程仓库
➜  git remote -v # 查看已经连接的远程仓库
origin	https://github.com/USERNAME/ProjectName.git (fetch)
origin	https://github.com/USERNAME/ProjectName.git (push)
remote	https://github.com/Project/ProjectName.git (fetch)
remote	https://github.com/Project/ProjectName.git (push)
➜  git fetch remote # 将远程主机的更新与自己的本地仓库同步
➜  git merge remote/main # 将代码并入本地仓库
~~~

这样就完成了远程与本地的同步，如果用`git status`查看可以发现本地比fork仓库已经多了n个待提交的更改，将它们`push`上去之后就完成了三个仓库代码的同步。

**添加并提交更改**

~~~bash
➜  git add README.md
➜  git commit -m "change README.md"
~~~

这里`"change README.md"`是对本次提交代码的说明，说明应当简洁直观，不能每次commit都用相同的说明

commit后先查看自己分支名和远程仓库名

~~~bash
➜  git branch
* main
➜  git remote -v
origin  https://github.com/USERNAME/ProjectName.git (fetch)
origin  https://github.com/USERNAME/ProjectName.git (push)
~~~

提交代码

~~~bash
➜ git push origin main
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 307 bytes | 307.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/USERNAME/ProjectName.git
   e76a91e..944fa03  main -> main
~~~

这样一次push就完成了

## 创建PR

如果在push之前已经创建了PR，那么到提交之后一次commit就完成了，但如果没有创建，需要在push之后手动创建PR

进入fork仓库主页-Pull requests-New pull request，选择提交的那一次commit，点击New pull requests，添加描述后就成功创建了PR

进入项目仓库主页-Pull requests 搜索自己的PR描述就能够找到自己的PR，之后等待审查人检查代码即可，如果需要修改，则重复上述commit流程

**以上只是贡献项目的通用基础过程，仅供参考，实际项目贡献会复杂很多，这个要自己去探索**

## 其它

### 提交注意

这里有一些通用的提交代码的约定，比如

提交Pull Request前

- 请注意commit的数量：

原因：如果仅仅修改一个文件但提交了十几个commit，每个commit只做了少量的修改，这会给评审人带来很大困扰。评审人需要逐一查看每个commit才能知道做了哪些修改，且不排除commit之间的修改存在相互覆盖的情况。

建议：每次提交时，保持尽量少的commit，可以通过`git commit --amend`补充上次的commit。对已经Push到远程仓库的多个commit，可以参考[squash commits after push](http://stackoverflow.com/questions/5667884/how-to-squash-commits-in-git-after-they-have-been-pushed)。

- 请注意每个commit的名称：应能反映当前commit的内容，不能太随意。

此外，在回复评审人意见时，遵守以下约定：

1）评审人的每个意见都必须回复（这是开源社区的基本礼貌，别人帮了忙，应该说谢谢）：

- 对评审意见同意且按其修改完的，给个简单的`Done`即可；
- 对评审意见不同意的，请给出自己的反驳理由。

2）如果评审意见比较多：

- 请给出总体的修改情况。
- 请采用[start a review](https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/)进行回复，而非直接回复的方式。原因是每个回复都会发送一封邮件，会造成邮件灾难。

**注意：**这些并不是强制性的，不遵循这些约定大概不会被骂，但是遵循的话双方体验都会很好。

### 代理

如果因为网络原因无法直接push，那么你或许应该挂代理，有多种方法可以在Git挂代理

首先要找到代理流量走的是自己电脑的那个端口，然后

1. https访问（这个实测有用）
   仅为github.com设置socks5代理(推荐这种方式，公司内网就不用设代理了，多此一举：
   `git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`
   其中1086是socks5的监听端口, 这个可以配置的, 每个人不同, 在macOS上一般为1086.
   设置完成后, ~/.gitconfig文件中会增加以下条目:

   ```json
   [http "https://github.com"]
       proxy = socks5://127.0.0.1:1086  \\这里1086改自己的端口
   ```

2. ssh访问（参考方法）
   需要修改~/.ssh/config文件，没有的话新建一个。同样仅为github.com设置代理:

   ```json
   Host github.com
       User git
       ProxyCommand nc -v -x 127.0.0.1:1086 %h %p \\这里1086改自己的端口
   ```

   如果是在Windows下, 则需要修改%home%.ssh\config，其中内容类似于:

   ```json
   Host github.com
       User git
       ProxyCommand connect -S 127.0.0.1:1086 %h %p
   ```

   这里-S表示使用socks5代理, 如果是http代理则为-H. connect工具git自带, 在\mingw64\bin\下面.



