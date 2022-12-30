## CUDA基础概念

* CUDA编程模型是一个异构模型，需要CPU和GPU协同工作。
* CUDA程序中既包含host程序，又包含device程序，它们分别在CPU和GPU上运行。
* 典型的CUDA程序的执行流程如下：
  1. 分配host内存，并进行数据初始化；
  2. 分配device内存，并从host将数据拷贝到device上；
  3. 调用CUDA的核函数在device上完成指定的运算；
  4. 将device上的运算结果拷贝到host上；
  5. 释放device和host上分配的内存。
* 最重要的是调用CUDA的核函数来执行并行计算
  * kernel是在device上线程中并行执行的函数（核函数）
  * 用`__global__`符号声明核函数
  * 用`<<<grid, block>>>`来指定核函数要执行的线程数量及结构
  * 每一个线程都要执行核函数，并且每个线程会分配一个唯一的线程号thread ID
    * 这个ID值可以通过核函数的内置变量`threadIdx`来获得
* 由于GPU实际上是异构模型，所以需要区分host和device上的代码
  * 通过函数类型限定词开区别host和device上的函数
    * `__global__`：在device上执行，从host中调用，返回类型必须是`void`，不支持可变参数参数，不能成为类成员函数。用`__global__`定义的kernel是异步的，这意味着host不会等待kernel执行完就执行下一步。
    * `__device__`：在device上执行，单仅可以从device中调用，不可以和`__global__`同时用。
    * `__host__`：在host上执行，仅可以从host上调用，一般省略不写，不可以和`__global__`同时用，但可和`__device__`一起用，此时函数会在device和host都编译。

* 要对kernel的线程层次结构有一个清晰的认识

  * 第一层：grid

    * Host调用kernel，一个kernel所启动的所有线程称为一个**网格**（grid）
    * 同一个网格上的线程共享相同的全局内存空间

  * 第二层：block

    * 网格又可以分为很多**线程块**（block）
    * 一个线程块里面包含很多线程

  * 两层组织结构如下图所示

    * ![img](https://pic1.zhimg.com/80/v2-aa6aa453ff39aa7078dde59b59b512d8_720w.jpg)

    * 图中gird和block均为2-dim的结构

      * grid和block都是定义为`dim3`类型的变量
      * `dim3`可以看成是包含三个无符号整数（x，y，z）成员的结构体变量
      * `dim3`可以灵活的定义为1-dim，2-dim以及3-dim结构

    * 上图kernel的定义语句如下

      ~~~c++
      dim3 grid(3, 2);
      dim3 block(5, 3);
      kernel_fun<<< grid, block >>>(prams...);
      ~~~

    * 因此一个线程需要两个内置的坐标变量（blockIdx，threadIdx）来唯一标识

      * blockIdx指明线程所在grid中的位置，是`dim3`类型
      * threaIdx指明线程所在block中的位置，是`dim3`类型































