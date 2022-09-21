## 环境搭建

### 1.安装go

没什么好说的，`yay -S go`直接下载安装最新版，如果需要的话记得配置一下国内源，不然包下载太慢

### 2.获取实验代码

```bash
$ git clone git://g.csail.mit.edu/6.824-golabs-2022 6.824
$ cd 6.824
$ ls
Makefile src
```

### 3.运行教学示例

每个项目不一样，过程都比较简单

### 4.运行测试脚本

每个项目不一样，过程都比较简单，会告诉你成功的实验项目会有什么功能产出。

### 5.提交测试

> 在提交之前最后运行一次测试

在 https://6824.scripts.mit.edu/2022/handin.py/ 里面输入邮箱就能收到api.key， 然后使用key提交，过程如下

```bash
$ cd ~/6.824
$ echo XXX > api.key
$ make lab1
```

## Lab简述

### Lab1

实现分布式mr，一个coordinator，一个worker（启动多个），在这次实验都在一个机器上运行。worker通过rpc和coordinator交互。worker请求任务，进行运算，写结果到文件。coordinator需要关心worker的任务是否完成，在超时情况下将任务重新分配给别的worker。

在这里看规则和提升：[送餐员小李/mit6.824 (gitee.com)](https://gitee.com/songcanyuan/mit6.824)