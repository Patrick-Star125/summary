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

docker run -d --name sqlite \
    -p 3306:3306 \
    nouchka/sqlite3

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

之后我们配置一些开机启动项，注意有的人可能上来就想装fish，但是实测fish的安装如果不慎可能会直接破坏系统terminal，具体原因未知，但是不建议安装fish

## 上线openMLDB

以实际案例为引导，这里我们安装一个openMLDB，并启动运行一个能够自动化feature process的线上ML应用

首先安装docker

> sudo apt install docker.io

因为docker源在国外，我们再进行换源

```bash
cd /etc/docker
sudo n daemon.json
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

如果这时候查看容器会发现由镜像创建的容器处于Exited(0)的状态，具体原因不详，目前没有解决，所以不知道该怎样上线
