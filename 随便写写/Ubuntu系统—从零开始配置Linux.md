## Multipass使用

我使用的是multipass虚拟机来测试和使用Ubuntu环境，这里介绍一些multipass的基础操作

首先最重要的是创建虚拟机，这里创建一个2核1G内存10G硬盘空间的最新发行版Ubuntu虚拟机

> multipass launch -n test -c 2 -m 1G -d 10G

查看有哪些虚拟机

> multipass list

查看虚拟机的信息

> multipass info [name]

两种在虚拟机内使用bash的方法

> multipass exec [name] -- [bash]
>
> multipass shell [name]

虚拟机的重启，关机，开机，(彻底)删除，恢复删除

> multipass restart [name]
>
> multipass stop [name]
>
> multipass start [name]
>
> multipass delete [name] --[purge]
>
> multipass recover [name]

## 基础配置

我们以一个Ubuntu-server系统为例，这也是我用的最多的系统

注意我遇到过sudo命令依然权限不够的情况，如果你觉得有必要的话可以使用

> sudo passwd root

来设置root用户的密码，之后

> su root

输入密码，就可以进入root用户执行命令了，需要退出root使用ctrl+D即可

首先apt换源，换源之前将原来的apt源配置文件复制保存一下，以防万一嘛

> sudo cp /etc/apt/sources.list /etc/apt/sources_init.list

然后打开源的配置文件

> sudo nano /etc/apt/sources.list

然后替换源的配置文件中的内容，并保存

> 腾讯源
>
> deb http://mirrors.tencentyun.com/ubuntu/ focal main restricted universe multiverse
> deb http://mirrors.tencentyun.com/ubuntu/ focal-security main restricted universe multiverse
> deb http://mirrors.tencentyun.com/ubuntu/ focal-updates main restricted universe multiverse
> #deb http://mirrors.tencentyun.com/ubuntu/ focal-proposed main restricted universe multiverse
> #deb http://mirrors.tencentyun.com/ubuntu/ focal-backports main restricted universe multiverse
> deb-src http://mirrors.tencentyun.com/ubuntu/ focal main restricted universe multiverse
> deb-src http://mirrors.tencentyun.com/ubuntu/ focal-security main restricted universe multiverse
> deb-src http://mirrors.tencentyun.com/ubuntu/ focal-updates main restricted universe multiverse
> #deb-src http://mirrors.tencentyun.com/ubuntu/ focal-proposed main restricted universe multiverse
> #deb-src http://mirrors.tencentyun.com/ubuntu/ focal-backports main restricted universe multiverse

注意如果是买的国内服务器，一般apt源是已经配置好的，比如腾讯云服务器是腾讯源，如果是使用原生的Ubuntu，则需要换源

之后更新源的配置文件

> sudo apt-get update

再更新换源之后的各种软件

> sudo apt-get upgrade

这一步如果是装的最新版Ubuntu实测不会有什么影响

之后我们配置一些开机启动项，注意有的人可能上来就想装fish，但是实测fish的安装如果不慎可能会直接破坏系统terminal，建议装之前考虑一下系统环境是否适合。

## 上线openMLDB

以实际案例为引导，这里我们安装一个openMLDB，并启动运行一个能够自动化feature process的线上ML应用

首先安装docker

> sudo apt install docker.io

因为docker源在国外，我们再进行换源

```bash
cd /etc/docker
sudo nano daemon.json
//然后添加源
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://ustc-edu-cn.mirror.aliyuncs.com",
    "https://ghcr.io",
    "https://mirror.baidubce.com"
  ]
}
//最后重启docker
sudo service docker restart
```

然后我们创建openMLDB的docker容器

~~~bash
//拉取并创建docker容器，镜像下载大小大约 1GB，解压后约 1.7 GB
sudo docker pull 4pdosc/openmldb:0.4.0
//然后运行openMLDB的容器
sudo docker run -it 4pdosc/openmldb:0.4.0 bash
~~~

因为openMLDB并不是挂起即上线的数据库镜像，因此需要进入容器中进行配置后才能上线服务，具体可以看[官方案例](https://openmldb.ai/docs/zh/main/use_case/taxi_tour_duration_prediction.html)。

## 系统总览

如果在整个系统文件目录的最上层可以看到系统目录的结构如下图所示

![](http://1.14.100.228:8002/images/2022/05/15/20220515105557.png)

这里有一些文章解释了这些目录的作用，因为实在是太多了，所以不在这里一一列举。

* [Linux /dev目录详解和Linux系统各个目录的作用_maopig的博客-CSDN博客_linux各个目录的作用](https://blog.csdn.net/maopig/article/details/7195048)

上面文章简单讲解了root、bin、etc、dev、home、tmp、usr、opt、media的功能，详细讲解了dev目录和proc目录。

* [ubuntu中 /usr、/var、/opt目录解析_echo_________的博客-CSDN博客_ubuntu usr目录在哪](https://blog.csdn.net/weixin_40822665/article/details/115057223)

上面文章详细讲解了var、usr目录。





















