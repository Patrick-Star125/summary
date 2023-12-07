**安装Docker**

~~~bash
# ubuntu
apt  install docker.io
~~~

**docker换源**



### docker网络模式

网络是Docker中相对比较薄弱的部分，我们有必要了解Docker的网络知识，以满足更高的网络需求。

当主机安装Docker后，Docker会自动创建三个网络。你可以使用`docker network ls`命令列出这些网络：

~~~bash
NETWORK ID          NAME                DRIVER              SCOPE
594430d2d4bb        bridge              bridge              local
d855b34c5d51        host                host                local
b1ecee29ed5e        none                null                local
~~~

运行容器时，你可以使用`--network NAME`来指定容器应连接到哪些网络，Docker有以下4种网络模式：

host模式：使用 --net=host 指定。

none模式：使用 --net=none 指定。

bridge模式：使用 --net=bridge 指定，默认设置。

container模式：使用 --net=container:NAME_or_ID 指定。

**host模式**

Docker使用了Linux的Namespaces技术来进行资源隔离，如PID Namespace隔离进程，Mount Namespace隔离文件系统，Network Namespace隔离网络等。一个Network Namespace提供了一份独立的网络环境，包括**网卡、路由、Iptable规则**等都与其他的Network Namespace隔离。

host模式类似于Vmware的**桥接**模式，与宿主机在同一个网络中，但没有独立IP地址。一个Docker容器一般会分配一个独立的Network Namespace。但如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。

如下图所示：容器与主机在相同的网络命名空间下面，使用相同的网络协议栈，容器可以直接使用主机的所有网络接口

![](http://1.14.100.228:8002/images/2022/07/11/20220711103915.png)

host 模式简单并且性能高，host 模式下面的网络模型是**最简单**和**最低延迟**的模式，容器进程直接与主机网络接口通信，与物理机性能一致，host不利于**网络自定配置和管理**，并且所有主机的容器使用相同的IP。也不利于**主机资源的利用**。对网络性能要求比较高，当有特殊需求的时候可以使用该模式，否则应该使用其他模式

## docker进阶

### docker-compose

### Dockerfile

创建Dockerfile用于java后端

~~~bash
#指定基础镜像
From java:8
#复制文件到容器
COPY lizhishop-0.0.1-SNAPSHOT.jar /app.jar
#声明需要暴露的接口
EXPOSE 8088
#配置容器启动后执行的命令
ENTRYPOINT ["java","-jar","/app.jar"]
~~~

创建Dockerfile用于python后端

~~~bash
FROM python:3

LABEL Netpunk "netpunk125@gmail.com"

RUN rm /etc/apt/sources.list

COPY ./sources.list /etc/apt/sources.list

RUN apt-get update -y && apt-get install -y python3-pip

COPY ./requirements.txt /requirements.txt 

WORKDIR /

RUN pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

COPY . /

EXPOSE 80 5000

ENTRYPOINT ["python3", "app/app.py"]
~~~

- **FROM** - 所有Dockerfile的第一个指令都必须是 `FROM` ，用于指定一个构建镜像的基础源镜像，如果本地没有就会从公共库中拉取，没有指定镜像的标签会使用默认的latest标签，如果需要在一个Dockerfile中构建多个镜像，可以使用多次。
- **MAINTAINER** - 描述镜像的创建者，名称和邮箱。
- **RUN** - RUN命令是一个常用的命令，执行完成之后会成为一个新的镜像，通常用于运行安装任务从而向映像中添加额外的内容。在这里，我们需更新包，安装  `pip` 。在第二个 `RUN` 命令中使用 `pip` 来安装 `requirements.txt` 文件中的所有包。
- **COPY** - 复制本机文件或目录，添加到指定的容器目录, 本例中将 `requirements.txt` 复制到镜像中。
- **WORKDIR** - 为RUN、CMD、ENTRYPOINT指令配置工作目录。可以使用多个WORKDIR指令，后续参数如果是相对路径，则会基于之前命令指定的路径。
- **ENTRYPOINT** - 在启动容器的时候提供一个默认的命令项。
- **RUN** - 运行 `app` 目录中的 `app.py` 。

用Dockerfile打包镜像

~~~bash
docker build -t image-name:1.0 .
~~~

## docker技巧

**忘记mysql容器密码**



**忘记portainer密码**



**实时查看容器日志**

使用下面的命令

~~~shell
docker logs -f container_name
~~~

**查看docker容器内系统型号**

~~~bash
cat /etc/issue
或者 lsb_release -a
~~~

> Debian GNU/Linux 11 \n \l 		//这表示系统是Debian 11

容器内系统的版本是很重要的，直接关系到服务能否正常运行，也关系到如何换源

[Debian 11 (bullseye) 国内软件源 - Guanglin - 博客园 (cnblogs.com)](https://www.cnblogs.com/liuguanglin/p/debian11_repo.html)

**在没有nano/vim的容器编辑文件**

~~~bash
cat > hk.html << EOF
~~~

在Linux中，EOF指的是“End of File”，表示文件的结尾。当程序读取文件时，它会一直读取直到遇到EOF为止，因为在文件结尾后就没有更多的数据可以被读取了。

EOF在文件中是一个特殊的字符或标记，通常是ASCII码为-1。当读取器读取到EOF时，它会返回一个特殊值来表示文件结束，告诉调用者不再读取文件。

### 部署

一些文章

* [使用Docker部署后端jar包](https://blog.csdn.net/f1443369600/article/details/102530828?utm_source=app&app_version=5.5.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)

* [Docker容器化部署Python应用](https://zhuanlan.zhihu.com/p/71251233)

## 常用镜像

这些镜像基于的系统大多是Ubuntu、Arch、Centos。但是其版本和内置的系统生态各不相同

纯净系统：完全只有系统，没有vim、lsb_release等基础工具

基础系统：有vim、lsb_release等基础工具

~~~bash
openjdk:17.0.2-slim-buster # java17 纯净系统
williamyeh/java8 # java8 基础系统
mysql:5.7 # mysql5.7 纯净系统
mysql:8.0 # mysql8.0 纯净系统
mzz2017/v2raya # v2raya代理 系统不明
~~~

