最近有在服务器的docker中编译一个大型项目的需求，但是发现编译过程中要在线clone许多其它的项目，因此需要配置代理到到git上。

## Linux配置代理

首先是给系统配置代理，我的情况如下：

系统：Arch Linux 5.17.9

代理：自建v2ray vps 使用的协议为VMESS(websocket+TLS)

客户端：v2raya 用这个是因为有web控制台，比较直观

其它的环境都在docker里面了，没什么参考意义

### v2raya安装

这里默认你已经知道v2ray代理如何配置，直接到安装的环节

**安装v2ray core**

v2raya基于v2ray core提供服务，因此首先要安装v2ray core。直接按照系统安装v2ray即可，安装之前首先更新一下源

~~~bash
sudo pacman -Sy # arch linux
sudo apt update # ubuntu
~~~

然后下载v2ray core

~~~bash
sudo yay -S v2ray # arch linux
sudo apt install v2ray # ubuntu
~~~

**安装启动v2raya**

直接安装即可

~~~bash
sudo yay -S v2raya # arch linux
sudo apt install v2raya # ubuntu
~~~

如果提示找不到，可以尝试一下更新源，如果还是找不到，则可以尝试一下增加v2raya软件源

首先添加软件源公钥

```text
wget -qO - https://apt.v2raya.mzz.pub/key/public-key.asc | sudo apt-key add -
```

然后添加 v2raya软件源，并更新

```text
echo "deb https://apt.v2raya.mzz.pub/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
```

然后启动 v2raya

```text
sudo systemctl start v2raya.service
```

如果有需要则设置开机自动启动，不然主机每一次重启都要重新启动v2raya

```text
sudo systemctl enable v2raya.service
```

### 配置v2raya

启动v2raya后，系统会自动在2017端口启动控制台，可以在内网的另一台主机上访问该控制台 http://xxx.xxx.xxx.xxx:2017

访问控制台，新建管理员账号后，就能够看到控制台界面了，直接点击创建，v2ray相信配置过的人也都会配，创建完节点之后界面如下图所示

![](http://1.14.100.228:8002/images/2022/07/11/20220711111913.png)

比较关键的部分我用红框标出，配过代理的人点进去看一下也就知道什么意思了，都配置完毕后，我们点击运行，主机的代理就算是配好了

> 我在启动的遇到了系统时间和实际时间对不上的错误，这里也一同附上解决办法
>
> 1.安装ntpdate：`yum install -y ntpdate`
>
> 2.同步时间：`ntpdate 0.asia.pool.ntp.org`

如果是Arch系统，那么到这里就算配好了，但如果是Ubuntu系统，还要手动配置一下系统的代理，如果想要暂时的代理

~~~bash
export http_proxy='http://127.0.0.1:20171'
export socks5_proxy='socks5://127.0.0.1:20170'
export no_proxy='localhost,127.0.0.1'
~~~

如果想要持久的代理，到用户目录下，将上面的语句添加到.bashrc文件中，然后重新载入文件即可。

~~~bash
vim .bashrc
source .bashrc
~~~

我们尝试一下git clone

![](http://1.14.100.228:8002/images/2022/07/11/20220711112214.png)

可以看到速度还是不错的

## Docker容器代理

回到我一开始的目的，我要在docker容器内配置代理，这牵扯到另一个概念，docker网络模式，这里不细究原理，直接说结果：docker容器的网络和主机的网络默认是两套东西，如果我们想在docker容器内使用代理，有两种方式

### 一、直接让容器和主机用同一套网络

因为我只是想在docker中编译一个大型项目，并不需要高性能的网络通信，因此我直接采用更简单的思路，让容器和主机用同一套网络很简单，只要在`docker run`命令加上一个参数`--network=host`即可让容器和主机网络同步，具体的说，容器与主机在相同的网络命名空间下面，使用相同的网络协议栈，容器可以直接使用主机的所有网络接口。

~~~bash
sudo docker run --network host
~~~

### 二、单独给docker容器配置代理

这方面我也查阅了一些资料，但是没有实操，有需要的自己动手去做吧。

资料一：[一文详解Docker 代理脱坑 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/106968269)

资料二：[如何优雅的给 Docker 配置网络代理 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1806455)

