## Git

**git挂代理**

结合以上各位经验设置成功. 以下以macOS为准.

1. https访问（这个实测有用）
   仅为github.com设置socks5代理(推荐这种方式, 公司内网就不用设代理了, 多此一举):
   `git config --global http.https://github.com.proxy socks5://127.0.0.1:1086`
   其中1086是socks5的监听端口, 这个可以配置的, 每个人不同, 在macOS上一般为1086.
   设置完成后, ~/.gitconfig文件中会增加以下条目:

   ```json
   [http "https://github.com"]
       proxy = socks5://127.0.0.1:1086  \\这里改自己的端口
   ```

2. ssh访问
   需要修改~/.ssh/config文件, 没有的话新建一个. 同样仅为github.com设置代理:

   ```json
   Host github.com
       User git
       ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
   ```

   如果是在Windows下, 则需要个性%home%.ssh\config, 其中内容类似于:

   ```json
   Host github.com
       User git
       ProxyCommand connect -S 127.0.0.1:1086 %h %p
   ```

   这里-S表示使用socks5代理, 如果是http代理则为-H. connect工具git自带, 在\mingw64\bin\下面.

### 核心概念

**远程主机：**

**分支：**本地分支、远程分支

已经用过的所有指令

>git add --all
>
>git status
>
>git commit -m [message]
>
>git push 远程主机名 本地分支名
>
>git remote -v
>
>git remote add 远程主机名 -url
>
>git remote remove 远程主机名
>
>git branch
>
>git fetch 远程主机名
>
>git merge 远程主机名\远程分支

 





















## 服务器

基础构建

ssh登陆服务器

~~~bash
ssh -p 22016 root@10.12.0.87 -o ServerAliveInterval=60 #GPUservers'd
~~~

```bash
ssh root@1.14.100.228 -o ServerAliveInterval=60	#腾讯云
```

~~~bash
ssh root@122.9.145.200 -o ServerAliveInterval=60 #华为云1
ssh root@116.63.148.146 -o ServerAliveInterval=60 #华为云2
ssh root@122.9.162.62 -o ServerAliveInterval=60 #华为云3
~~~

