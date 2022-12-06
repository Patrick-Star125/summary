[Nginx Proxy Manager](https://nginxproxymanager.com/)安装docker语句(yml)

**run**

~~~bash
docker run -d --name proxy \
	-v /home/proxy/data:/data \
	-v /home/proxy/letsencrypt:/etc/letsencrypt \
	-p 80:80 -p 81:81 -p 443:443 \
	--restart=unless-stopped \
	--link mysql:mysql \
    -e WORDPRESS_DB_HOST=mysql:3306 \
    -e WORDPRESS_DB_NAME=proxy \
    -e WORDPRESS_DB_USER=root \
    -e WORDPRESS_DB_PASSWORD=password \
	jc21/nginx-proxy-manager:latest
~~~

**yml**

~~~bash
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - /home/proxy/data:/data
      - /home/proxy/letsencrypt:/etc/letsencrypt
~~~

[Portainer](https://www.portainer.io/)安装docker语句(yml)

~~~bash
docker run -d -p 9000:9000 -p 9443:9443 \
	--name portainer \
	--restart=unless-stopped \
	-v /var/run/docker.sock:/var/run/docker.sock \
	-v /home/portainer_data:/data \
	--link mysql:mysql \
    -e WORDPRESS_DB_HOST=mysql:3306 \
    -e WORDPRESS_DB_NAME=portainer \
    -e WORDPRESS_DB_USER=root \
    -e WORDPRESS_DB_PASSWORD=@abc123456 \
	portainer/portainer-ce:2.11.1
~~~

上面的两个端口一个是http服务端口(9000)，另一个是https服务端口(9443)

域名（Domain Name）是由一串字符组成的，域名指向某一个IP地址。例如xxx.xxx.cn。

使用域名主要有两个好处：

1. ip是不好记的，而域名是好记的，用域名更好管理ip
2. ip如果一直暴露，有可能有安全风险（不过这个可能性不大）

## 购买域名

有关域名的购买，有两点要注意

1. 域名的价值
2. 域名的价格

## 域名备案



## 配置Nginx Proxy Manager



## 安装Portainer(optional)





























