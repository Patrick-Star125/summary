## docker基础

### 基础概念



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



**添加容器端口映射**



**实时查看容器日志**

使用下面的命令

~~~shell
docker logs -f container_name
~~~

**查看docker容器内系统型号**

~~~bash
cat /etc/issue
~~~

> Debian GNU/Linux 11 \n \l 		//这表示系统是Debian 11

容器内系统的版本是很重要的，直接关系到服务能否正常运行，也关系到如何换源

[Debian 11 (bullseye) 国内软件源 - Guanglin - 博客园 (cnblogs.com)](https://www.cnblogs.com/liuguanglin/p/debian11_repo.html)

### 应用

* [使用Docker部署后端jar包](https://blog.csdn.net/f1443369600/article/details/102530828?utm_source=app&app_version=5.5.0&code=app_1562916241&uLinkId=usr1mkqgl919blen)



* [Docker容器化部署Python应用](https://zhuanlan.zhihu.com/p/71251233)



