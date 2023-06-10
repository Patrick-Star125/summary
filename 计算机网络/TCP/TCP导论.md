# TCP

![](http://1.14.100.228:8002/images/2022/11/10/20221110163950.png)

## TCP概述

TCP 是**面向连接的、可靠的、基于字节流**的传输层通信协议，它能确保接收端接收的网络包是**无损坏、无间隔、非冗余和按序的。**

接下来所有

- **面向连接**：一定是「一对一」才能连接，不能像 UDP 协议可以一个主机同时向多个主机发送消息，也就是一对多是无法做到的；
- **可靠的**：无论的网络链路中出现了怎样的链路变化，TCP 都可以保证一个报文一定能够到达接收端；
- **字节流**：用户消息通过 TCP 协议传输时，消息可能会被操作系统「分组」成多个的 TCP 报文，如果接收方的程序如果不知道「消息的边界」，是无法读出一个有效的用户消息的。并且 TCP 报文是「有序的」，当「前一个」TCP 报文没有收到的时候，即使它先收到了后面的 TCP 报文，那么也不能扔给应用层去处理，同时对「重复」的 TCP 报文会自动丢弃。
- **无损坏：**
- **无间隔：**
- **非冗余：**
- **按顺序：**



**两个方面理解TCP连接**

建立一个 TCP 连接是需要客户端与服务端端达成**Socket、序列号、窗口大小**三个信息的一致

- **Socket**：由 IP 地址和端口号组成
- **序列号**：用来解决乱序问题等
- **窗口大小**：用来做流量控制

TCP 四元组可以唯一的确定一个连接，四元组包括

<img src="http://1.14.100.228:8002/images/2022/11/10/20221110150547.png" style="zoom:67%;" />

理论上，一个源地址的源端口提供的服务可以连接**目标地址x目标端口**数量的连接，具体的说，对 IPv4，客户端的 IP 数最多为 `2` 的 `32` 次方，客户端的端口数最多为 `2` 的 `16` 次方，也就是服务端单机最大 TCP 连接数，约为 `2` 的 `48` 次方。

## TCP与主机系统

当服务端与客户端建立TCP连接时，会收到系统的限制，包括：

- 文件描述符限制，每个 TCP 连接都是一个文件，如果文件描述符被占满了，会发生 too many open files。Linux 对可打开的文件描述符的数量分别作了三个方面的限制：
  - **系统级**：当前系统可打开的最大数量，通过 cat /proc/sys/fs/file-max 查看；
  - **用户级**：指定用户可打开的最大数量，通过 cat /etc/security/limits.conf 查看；
  - **进程级**：单个进程可打开的最大数量，通过 cat /proc/sys/fs/nr_open 查看；
- **内存限制**，每个 TCP 连接都要占用一定内存，操作系统的内存是有限的，如果内存资源被占满后，会发生 OOM。



## TCP与UDP

相比于TCP，UDP 不提供复杂的控制机制，利用 IP 提供面向「无连接」的通信服务。

|          |                        TCP                         |                   UDP                    |
| :------: | :------------------------------------------------: | :--------------------------------------: |
|   连接   | TCP 是面向连接的传输层协议，传输数据前先要建立连接 |      UDP 是不需要连接，即刻传输数据      |
| 服务对象 |   TCP 是一对一的两点服务，即一条连接只有两个端点   | UDP 支持一对一、一对多、多对多的交互通信 |
|          |                                                    |                                          |
|          |                                                    |                                          |
|          |                                                    |                                          |
|          |                                                    |                                          |
|          |                                                    |                                          |

由于 TCP 是面向连接，能保证数据的可靠性交付，因此经常用于：

- `FTP` 文件传输；
- HTTP / HTTPS；

由于 UDP 面向无连接，它可以随时发送数据，再加上 UDP 本身的处理既简单又高效，因此经常用于：

- 包总量较少的通信，如 `DNS` 、`SNMP` 等；
- 视频、音频等多媒体通信；
- 广播通信；

TCP和UDP在系统层面的处理是分离的，所以一个应用程序可以同时使用TCP和UDP协议监听相同的端口



## TCP与应用程序

几乎所有语言都实现了Socket，程序范式如下图所示：

![基于 TCP 协议的客户端和服务端工作](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzM0LmpwZw?x-oss-process=image/format,png)

这里需要注意的是，服务端调用 `accept` 时，连接成功了会返回一个已完成连接的 socket，后续用来传输数据。

所以，监听的 socket 和真正用来传送数据的 socket，是「两个」 socket，一个叫作**监听 socket**，一个叫作**已完成连接 socket**。

成功连接建立之后，双方开始通过 read 和 write 函数来读写数据，就像往一个文件流里面写东西一样。

**socket示例-python服务端**

~~~python
def handle_accept(c, addr):
    with c:
        print(addr, "connect: ")

        while True:
            data = c.recv(1024) #接收最大1024字节数据
            if not data:
                break
            c.sendall(data) #全部返回
    
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind(("0.0.0.0", 1234)) #端口绑定
    s.listen() #端口监听
    while True:
        c, addr = s.accept() #接收连接

        t = threading.Thread(target=handle_accept, args=(c, addr)) #建立线程处理
        t.start()
~~~

