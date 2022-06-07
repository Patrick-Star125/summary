前几天发现，假如我要借给朋友自己的服务器来跑应用的话，自己没有时间给朋友配置服务，可以在容器里面配置ssh登录，交给别人自己去配置

这里以在腾讯云轻量级服务器上配置一个opengauss数据库容器为例，讲一讲我所做的全流程

## 创建并运行容器

首先是创建容器

> docker run --name dockername --privileged=true -d -e GS_PASSWORD=@Aa123456 -v /home/zhongyuhui:/var/lib/opengauss/data -p 6543:5432 -p 8500:22 enmotech/opengauss:latest

这里解释一下这句docker run的含义

* --name 是容器名字，自己填写
* -v 是挂载文件夹，`/var/lib/opengauss/data`是容器内opengauss数据库文件所在的位置
* 第一个-p是数据库挂载端口，前面无所谓，后面得是5432
* 第二个-p是ssh登录需要映射的端口，前面无所谓，后面得是22

然后进入容器

> docker exec -it dockername bash

## 安装并配置ssh

因为容器内核十分简洁，我们需要先安装一个nano，之后用得上

> apt-get update # 更新源
>
> apt-get install nano

之后我们换源，后面的下载会舒服很多

> nano /etc/apt/sources.list

删除里面所有的内容，然后将腾讯云自己的源放入

~~~bash
deb http://mirrors.tencentyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.tencentyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.tencentyun.com/ubuntu/ focal-updates main restricted universe multiverse
#deb http://mirrors.tencentyun.com/ubuntu/ focal-proposed main restricted universe multiverse
#deb http://mirrors.tencentyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.tencentyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.tencentyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.tencentyun.com/ubuntu/ focal-updates main restricted universe multiverse
#deb-src http://mirrors.tencentyun.com/ubuntu/ focal-proposed main restricted universe multiverse
#deb-src http://mirrors.tencentyun.com/ubuntu/ focal-backports main restricted universe multiverse
~~~

保存后退出，再更新源

> apt-get update

安装openssh-server并启动

~~~bash
apt-get install openssh-server
# 启动之前需手动创建/var/run/sshd，不然启动sshd的时候会报错
mkdir -p /var/run/sshd
# sshd以守护进程运行
/usr/sbin/sshd -D &
# 安装netstat，查看sshd是否监听22端口
apt-get install net-tools
netstat -apn | grep ssh
~~~

配置ssh登录

~~~bash
# 生成ssh key
ssh-keygen -t rsa
# 修改sshd-config允许root登陆
nano /etc/ssh/sshd_config
# 将下面这句
#PermitRootLogin prohibit-passwd
改为
PermitRootLogin yes
~~~

修改后退出，之后创建登录密码

> passwd # 运行命令后直接输入密码

修改完sshd-config之后需要重启sshd服务

~~~bash
// 找到pid
ps -aux | grep ssh
// s
kill -9 pid
// 再次运行ssh
/usr/sbin/sshd -D &
~~~

> ps -aux | grep ssh # 找到ssh的pid

![](http://1.14.100.228:8002/images/2022/04/04/20220404100239.png)

这里是5564，之后

> kill -9 5564
>
> /usr/sbin/sshd -D & # 再次运行ssh

这样就配置完了，我们尝试在其它机器上登录，我这里用powershell

> ssh -p 8500 root@ip # ip就是你自己服务器的弹性公网ip

输入密码，登录成功

![](http://1.14.100.228:8002/images/2022/04/04/20220404100853.png)

注意此时是**root用户**登录，如果是要公开访问的容器，最好配置**普通用户登录**













