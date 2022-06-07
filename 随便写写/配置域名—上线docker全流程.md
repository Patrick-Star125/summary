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
    -e WORDPRESS_DB_PASSWORD=@abc123456 \
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

## 购买域名-域名备案

## Nginx Proxy Manager配置

























