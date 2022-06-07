## 配置Ubuntu虚拟机

我这里的环境是Windows，因此选择VMware作为虚拟机运行软件

首先下载VMware，下载破解软件建议到微信搜索，下面是随便找的一个例子

[VMware16中文版软件下载和安装教程|兼容WIN10 (qq.com)](https://mp.weixin.qq.com/s?src=11&timestamp=1651890498&ver=3783&signature=i21l8eLRVbnx8zusdtYtonycOJjI37T8DB0burRpespOr3cGU2D*EX6Il-G8-s9qEpSIRdBPfKB2*F6AjkrQl3-p9GwwBpEPYfA2BvAEa5WM6f8tAiCQlebrPlFaM-t*&new=1)

跟着流程下载后安装Ubuntu，我用的是20.04 LTS版本。

安装完成后如果发现虚拟机没有全屏，可以查看这个方法[VMware 设置虚拟机全屏](https://blog.csdn.net/nyist_zxp/article/details/108503580?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-108503580-blog-54409320.pc_relevant_paycolumn_v3&spm=1001.2101.3001.4242.2&utm_relevant_index=4)

这一段有很多教程了，因此这里不再赘述，最后的基础环境应该像这样

![](http://1.14.100.228:8002/images/2022/05/07/20220507170959.png)

然后我们换一下源，这里用阿里源

~~~bash
# 先将原本的源备份一下
sudo cp /etc/apt/sources.list /etc/apt/sources_init.list
# 然后打开源的配置文件
sudo nano /etc/apt/sources.list
#添加阿里源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
#添加清华源
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# 更新一下
sudo apt-get update
~~~

这样基础环境就配好了

## CMU15445

已经做了部分实验的同学会知道，CMU15445的实验环境就是下面几样加起来

* C++编辑器(clion)
* clang clang-format clang-tidy
* cmake
* doxygen
* g++
* valgrind

其中clion可以直接在软件商店下载，我是因为有教育账户所以直接用clion(原本收费)，如果条件不允许可以用其它的编辑器

![](http://1.14.100.228:8002/images/2022/05/07/20220507174312.png)

然后我们clone[课程项目](https://github.com/cmu-db/bustub)，如果直接git clone不成功的话可以退出虚拟机在外部clone再直接压缩拖拽到虚拟机中，十分方便

其它环境都可以通过运行工程中自带的package.sh文件安装

```bash
# 赋予文件执行的权限
sudo chmod u+x packages.sh
sudo ./build_support/packages.sh
```

安装完成后进入clion，会自动配置上g++和make，配置完成后就可以直接开始项目了

## XV6

同样在Ubuntu虚拟机的基础上搭建环境，具体的可以看这位老哥的博客[OS实验xv6 6.S081 开坑](https://blog.csdn.net/weixin_44465434/article/details/110522725)

在他的基础上可以做一些调整，我自己是用clion直接在虚拟机内开发。





















