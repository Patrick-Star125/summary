# MapReduce

## 概念

2004年谷歌提出了MapReduce，这是一种用于处理大规模数据集的编程模型，在MapReduce出现之前，谷歌程序员面对的大规模数据集，通常需要完全手动编程实现，例如：

1. 统计某个关键词的现的频率,计算pageRank
2. 对大规模数据按词频排序
3. 对多台机器上的文件进行grep等

这些工作不可能在一台机器上完成(否则也不能称之为大规模)，因此谷歌的程序员每次编写代码都需要处理,**多机并行协同**，**网络通信，处理错误，提高执行效率**等问题。MapReduce就是为了提高开发效率而产生的。

来自于对Lisp语言中map/reduce原语的借鉴，经过谷歌大量重复的编写数据处理类的程序，发现所有数据处理的程序都有类似的过程：

> 将一组输入的数据应用map函数返回一个k/v对的结构作为中间数据集,并将具有相同key的数据输入到一个reduce函数中执行,最终返回处理后的结果。

![](http://1.14.100.228:8002/images/2022/09/07/20220907163909.png)

左图和右图合起来一起看，假如按map函数规则来看数据能够分为n类，那么会产生n个key，reduce对相同key的数据集进行收集整理得到计算结果。以一个统计文档单词数的java代码为例，理解一下MapReduce的作用方式。

~~~java
map(String key,String value):
    // key: 文档名
    // value: 文档内容
    for word w in value:
        EmitIntermediate(w, wordcount);
reduce(String key, Iterator values):
    // key: 一个单词
    // value: 计数值列表
    int result = 0;
    for each v in values:
        result += ParseInt(v);
    Emit(AsString(result));
~~~

这一计算模型的优势在于非常利于并行化，map的过程可以在多台机器上而机器之间不需要相互协调，reduce的执行同样不需要协调，没有相互依赖意味着可并行执行。

 所有的依赖性和协调过程都被隐藏在map与reduce函数的数据分发（shuffle）之间。因此这里有一个很重要的细节就是 **map和reduce都是可以各自并行执行,但reduce执行的前提条件是所有的map函数执行完毕**。

作为一个编程模型，mapreduce如此工作 但是这一过程想要落地就必须要考虑工程上的永恒问题 **可靠性** 与 **性能**。

大规模的机器产生局部性的失败是一种必然现象，因此如果mapreduce分散在多台机器上执行,其中一个map 任务失败导致整个处理过程都需要重新计算这是不可接受的。 因此对于大规模分布式程序来说能够应对局部性失败的容错性与性能同等重要。这是一个必要的问题,**分布式的容错性本质上就是如何在不可靠的硬件之上构建可靠的软件。**

所以 作为一个成熟的工业级实现 MapReduce 就是一个 **利用普通机器组成的大规模计算集群进行并行的,高容错,高性能的数据处理函数框架。**

### 应用场景

论文中列举了一些Google在实际业务中遇到的大数据场景，它们都可以很轻松的用MapReduce模型表达。

**分布式Grep：**map函数在匹配到给定的pattern时输出一行。reduce函数只是将给定的中间数据复制到输出上。

**URL访问频次统计：**map函数处理网页请求的日志，对每个URL输出〈URL, 1〉。reduce函数将相同URL的所有值相加并输出〈URL, 总次数〉对。

**倒转Web链接图：**map函数在source页面中针对每个指向target的链接都输出一个〈target, source〉对。reduce函数将与某个给定的target相关联的所有source链接合并为一个列表，并输出〈target, list(source)〉对。

**每个主机的关键词向量：**关键词向量是对出现在一个文档或一组文档中的最重要的单词的概要，其形式为〈单词, 频率〉对。map函数针对每个输入文档（其主机名可从文档URL中提取到）输出一个〈主机名, 关键词向量〉对。给定主机的所有文档的关键词向量都被传递给reduce函数。reduce函数将这些关键词向量相加，去掉其中频率最低的关键词，然后输出最终的〈主机名, 关键词向量〉对。

**倒排索引：**map函数解析每个文档，并输出一系列〈单词, 文档ID〉对。reduce函数接受给定单词的所有中间对，将它们按文档ID排序，再输出〈单词, list(文档ID)〉对。所有输出对的集合组成了一个简单的倒排索引。用户可以很轻松的扩展这个过程来跟踪单词的位置。

**分布式排序：**map函数从每条记录中提取出key，并输出〈key, 记录〉对。reduce函数不改变这些中间对，直接输出。

## 计算过程

![](http://1.14.100.228:8002/images/2022/09/07/20220907165841.png)

1. 用户编写map函数和reduce函数，并且指定有m个map任务和R个reduce任务可以同时执行
2. 用户程序中的MapReduce库首先将输入文件切分为M块，每块的大小从16MB到64MB（用户可通过一个可选参数控制此大小）
3.  MapReduce库会在一个集群的若干台机器上启动程序的多个副本（下发任务脚本）
4. Master节点将M个map任务和R个reduce任务分配给空闲的工作节点，Worker节点一项任务。
5. 进行Map过程的第一阶段，按配置好的方式读取KV，也就是对应的输入区块内容，它从输入数据中解析出key/value对，然后将每个key/value对传递给用户定义的map函数。由map函数产生的中间key/value对都缓存在内存中。
6. 然后进行Map过程的第二阶段，映射R个内存映射区将kv存储R个中间文件中文件名为: Map任务编号-Reduce任务编号
7. R个复制reduce的worker各自获取M个中间文件（Shuffle），各自进行外部归并排序（或者其它排序）
8. 创建一个迭代器，聚合所有相同key的value组成一个`list<value>`，调用reduce函数传递进去，得到计算结果并且存储在最终的文件里
9. 将结果文件通过RPC传递回main函数

> worker始终和master节点保持通讯，将任务处理进度上报更新

## 容错

一个1000台机器的集群都没出错概率是很低的，一个简单的正态分布就可以说明大规模集群下容错的必要性。

### 工作节点的容错

1. **执行 map 任务的节点** 通过周期性的ping-pong心跳机制 main节点感知到map节点的状态，如果心跳超时认为节点失败。 重新将当前worker上的map task 传递给其他worker节点完成。主节点要维护一个**任务队列**来记录哪些任务还未完成，同时记录已经完成的任务的状态，来感知到当前mapreduce任务处于一个怎样的生命周期。
2. **执行reduce任务的节点** reduce产出的文件是一个持久性的文件，存在副作用，因此每次reduce被重新分配后要重命名一个新的文件，防止与损坏的文件冲突，或者在完成计算后直接覆盖掉损坏的文件。
3. **主节点**  如果主节点失败了怎么办？对于任务执行的元数据产生的中间态可以保持在一个**恢复点**文件中，当节点崩溃重启后可以从最近的一个恢复点重新执行mapreduce任务。
4. **副作用**  对于所有持久化的操作，不可避免的会产生副作用。导致这种副作用的根本原因在于不能原子的提交状态。 因此解决方案就是保证map与reduce产生的文件在执行过程中先存储在**临时文件中**(以**时间戳**命名) 等到提交文件时 将其原子的重命名为最终文件(linux内核中 重命名操作是有原子性的保证的)。

### **利用局部性**

在分布式系统中，网络带宽通常是稀缺资源，很容易成为系统瓶颈。 在MapReduce的任务中 至少需要**M*R次**的网络传输，才能将中间文件发送给reduce所在的worker节点上。同时把输入文件发送给map任务所在的worker 也是非常消耗网络带宽的事情。但是我们可以通过仅将map任务分配给本来就有所有输入文件的节点上，来减少一次网络调用使得性能得到提升。同时还可以使用一些流处理的思路优化shuffle的过程，那就是当一个map任务完成后通知main进程后，main进程立即通知reduce任务拉取其中一份文件，而不必等到所有map任务全部执行完毕后进行网络传输而提高了并行性。

### **任务的粒度**

应该配置多少个任务执行map，多少个任务执行reduce？任务拆分过多加剧网络传输的负担。而任务拆分的过少又会导致并行度不够而降低整体的执行效率。

为此 一些经验性的配置是map任务通常为输入文件总大小除以64M的值(这源于底层的分布式文件系统是以64m为一个chuck进行存储的)，reduce的数量通常是map任务的一半。同时为了发挥机器本身的多核特性，一台机器上可以指定多个mapreduce任务来执行通常是任务总数的百分之一。

### **备用任务**

如果最后的几个任务执行时间过长怎么办?存在这种case,10个任务用5分钟完成了其中9个,但最后一个任务因为当前机器的负载过高花费了20分钟执行完毕,这么整个任务的执行周期就是20分钟。 如何能应对这一问题呢? 当仅剩下1%的任务时,可以启动**备用任务**,即同时在两个节点上执行相同的任务。这样只要其中一个先返回即可结束整个任务,同时释放未完成的任务所占用的资源。

## 技巧

### **划分函数**

为了解决数据倾斜的问题（例如：100亿个相同数据，10万个不同数据），按map函数输出的key决定当前的kv被划分到哪一个中间文件中。 工程经验上看,我们有时不仅仅要求其平衡划分，我们还可能要求其可以按照某种kv的元数据信息（例如UUID）进行分区。 比如将一个主机下的文件划分到一起等定制的能力。

### **顺序保证**

有时我们需要支持对数据文件按key进行随机访问,或者有序输出，并且为了**减少reduce任务的负担**。 输出的每个map任务的中间文件都是保证按key递增有序的。

### **合并函数**

某些情况中，不同的map任务产生的中间key重复率非常高，而且用户指定的reduce函数可进行交换组合。一个典型的例子就是2.1节中的单词统计。单词频率符合齐夫分布，因此每个map任务都会产生非常多的<the, 1>这样的记录。所有这些记录都会通过网络被发送给一个reduce任务，然后再通过reduce函数将它们相加，产生结果。我们允许用户指定一个可选的合并函数，在数据被发送之前进行局部合并。（简单说就是在map之前就提前reduce一下，大幅减少RPC压力）

合并函数由每个执行map任务的机器调用。通常合并函数与reduce函数的实现代码是相同的。它们唯一的区别就是MapReduce库处理函数输出的方式。reduce函数的输出会写到最终的输出文件中，而合并函数的输出会写到中间文件中，并在随后发送给一个reduce任务。

### **输入和输出类型**

MapReduce库支持多种不同格式输入数据的读取。例如，“text”模式将每行输入当作一个key/value对：key是文件中的偏移，而value则是该行的内容。另一种支持的常见格式将key/value对按key排序后连续存储在一起。每种输入类型的实现都知道如何将它本身分成有意义的区间从而能分成独立的map任务进行处理（例如text模式的区间划分保证了划分点只出现在行边界处）。用户也可以通过实现一个简单的reader接口来提供对新的输入类型的支持，尽管大多数用户只是使用为数不多的几种预定义输入类型中的一种。

reader的实现不一定要提供从文件中读数据的功能。例如，可以很容易的定义一个从数据库读记录的reader，或是从映射在内存中的某种数据结构中。（也就是现代分布式计算系统的数据接口灵感来源）

类似的，我们也支持一组输出类型来产生不同格式的数据，而用户提供代码去支持新的输出类型也不难。

### **确定性问题**

 如果mapreduce执行过程是不确定那就会产生问题,也就说 mapreduce执行的前提假设是输入文件是静态的,是有界的,不能在执行过程中发生改变。如果发生改变那么mapreduce的计算结果并不保证其正确性。

### **略过坏记录**

数据集并不能保证完全是正确的,如果有一行记录是错误的导致map任务崩溃,不断的重试最终使得整个程序不能结束。因此必然需要跳过这一段错误的记录。 如何跳过呢？

每个map任务要捕获异常,通过安装信号的方式,在程序退出之前执行安装的信息函数把执行到的文件的行号offset等信息发送给主节点。 主节点在下次调度的时候 将这些offset处的记录作为黑名单列表传递给新的map任务，执行时会对此处的记录跳过执行。

### **本地执行**

如何调试分布式的程序?MapReduce通常在几千台机器上执行,这其中如果存在逻辑问题非常难以调试,因此为MapReduce提供单机多进程的实现可以有效的进行本地debug发现问题。

### **状态信息**

分布式系统的可观测性尤为重要,  通过在ping-pong过程中将每个worker节点的工作状态进行上报,存储在main进程中，并提供可访问的网页来展示系统运行的信息。即可实现可解释性。

###  **计数器**

如何在map reduce的处理进行埋点统计？以实现用户自定义的指标监控?  需要创建一个原子计数器的对象,由用户在map 和 reduce的函数中 在发生某些事件时 对其进行累加。 并通过ping-pong的心跳中将数据携带回 main进程 累加进去，但要注意每次消息的幂等性来保证不会导致重复累加或者少累加了计数的情况。

## 性能

如何对MapReduce的计算性能进行测试，一个计算则是排序1TB的数据，另一个计算是在差不多1TB的数据中查找指定的模式。这两个程序能代表大量真实的MapReduce程序——一类程序是将数据从一种表示形式转换为另一种，另一类程序则是从很大数量的数据集中提取出一小部分感兴趣的数据。

### 集群配置

所有的程序都运行在由1800台机器组成的集群上。每台机器配有：2GHz的Intel Xeon处理器且支持超线程，4GB内存，2块160GB的IDE硬盘，和1GB以太网连接。所有机器都安排在2层树结构的网关内，根节点的可用带宽加起来有100-200Gbps。所有的机器都处于相同的托管设施下，因此任意两台机器间的往返通信时间都少于1ms。

除了4GB的内存，还有1-1.5GB的内存保留给了集群上运行的其它任务。程序运行的时间是一个周末的下午，此时大多数CPU、磁盘和网络资源都空闲中。

### Grep

grep程序要扫描1010个长度为100字节的记录来寻找一个相当罕见的三字符模式（这个模式出现于92337个记录中）。输入被切分成64MB大小的部分（M = 15000），整个输出为一个文件（R = 1）。

![img](https://hardcore.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2Q3NTM5MmMzMTE2YzFjYTA2NGE0MGZkODYzODAwZmZfbUxUa005YXMyY2h6ZU1HZTVtbFNOeGlVTDE1TmlkWk5fVG9rZW46Ym94Y25zWGdtZEdhcUw0VzFiV291UGtDQTZEXzE2NjI5NzI4MTk6MTY2Mjk3NjQxOV9WNA)

图2显示了随时间变化的计算进度。Y轴是输入数据扫描的速率。这个速率逐渐提升代表更多的机器被分配给MapReduce计算，并在分配的机器数达到1764台时超过了30GB/s。随着map任务的结束，这个速率开始下降，并在计算开始后80s时降至0。整个计算过程共花费约150s。这包括了大约1分钟的启动开销。启动开销是由于要把程序传播到所有工作机器上，还包括GFS要打开1000个输入文件和获取局部优化所需信息而导致的延迟。

### 排序

排序程序要对10100个100字节的记录进行排序（差不多1TB数据）。这个程序模仿了TeraSort测试程序。

排序程序只包括不到50行的用户代码。map函数一共3行，它从一个文本行中提取出10字节的排序key，再将key和原始文本行输出为中间key/value对。我们使用了一个内置的Identity函数作为reduce函数。这个函数会将未更改的中间key/value输出为结果。最终的排序后输出被出到一组2路复制的GFS文件（例如，这个程序的输出要写2TB的数据）。

如前所述，输入数据被分成若干个64MB大小的部分（M = 15000）。我们将已排序的输出分成4000个文件（R = 4000）。划分函数使用key的首字节来将它分到R个文件中的一个。

此次测试中我们的划分函数使用了key分布的内建知识。在一个一般的排序程序中，我们会预先加一轮MapReduce操作，收集key的样本，并使用key抽样的分布来计算最终输出文件的划分点。

![img](https://hardcore.feishu.cn/space/api/box/stream/download/asynccode/?code=MmZjNTY5ZWZhYjVjYWRmMWMwMzU3NWJkYjA0OGZiODFfVjlxbGpYbGd6ZHBtcFpMTVdYMHRRYVExZHV4Y0ZVOGhfVG9rZW46Ym94Y25ianJKSkR0dTd3OGw2NnNGSHpwTmVmXzE2NjI5NzI4MTk6MTY2Mjk3NjQxOV9WNA)

图3(a)是排序程序的正常执行过程。上边的图显示了输入读取的速度。输入速度的峰值为13GB/s，并在200秒后所有的map任务都完成时迅速下降至0。可以注意到这个速度要比grep的速度小。这是因为排序的map任务要在向本地磁盘写中间文件上花费大约一半的时间和I/O带宽。而grep中相应的中间输出则可以忽略不计。

中间的图显示了数据从map节点通过网络向reduce节点发送的速度。这个移动开始于第一个map任务完成。图中的第一个驼峰出现在reduce任务第一次达到1700个时（整个MapReduce共分配到1700台机器，每台机器同时最多只运行一个reduce任务）。计算开始差不多300秒时，第一批reduce任务已经有部分完成，我们开始向剩余的reduce任务传送数据。所有传送过程在计算开始600秒后结束。

下面的图是reduce任务向最终的输出文件写入已排序数据的速度。在第一个传输周期的结束与写入周期的开始之间有一个延迟，因为此时机器正在排序中间数据。写入速度在2-4GB/s下持续了一段时间。所有的写操作在计算开始后850秒左右结束。包括启动开销的话，整个计算花费了891秒。这与TeraSort上目前公布的最佳记录1057秒很接近。

一些要注意的事：因为我们的局部性优化，输入速度要比传播速度和输出速度都快——大多数数据都读自本地磁盘，绕开了我们带宽相当有限的网络。传播速度要比输出速度高，因为输出阶段要写两份已排序的数据（因为可靠性和实用性的考虑）。我们写两份输出是因为这是我们的底层文件系统针对可靠性和实用性提供的机制。如果底层文件系统使用擦除代码而不是复制，那么写数据需要的网络带宽就会减少。

### 备用任务的影响

在图3(b)中显示的是禁用了备用任务情况下排序程序的执行过程。这种情况下的执行过程与图3(a)中的很相似，但在末尾处有很长一段时间几乎没有任何的写操作发生。在960秒后，只有5个reduce任务还没有完成。但这最后的几个任务直到300秒后才结束。整个计算花费了1283秒，比正常情况多花费44%的时间。

### 机器失败

在图3(c)中显示了有机器失败情况下的排序程序执行过程。在计算开始几分钟后，我们有意的杀掉了1746个工作节点中的200个。底层集群调度器立即在这些机器上重启了新的工作进程（只有进程被杀掉了，机器还在正常运行中）。

工作节点的死亡显示为一个负的输入速度，因为一些已经完成的map任务消失了（因为对应的map节点被杀掉了）需要重新完成。这些map任务的重执行很快就会发生。整个计算过程在933秒后结果，包括了启动开销（相比正常执行时间只增加了5%）。

## HDFS的基本使用

~~~
hadoop fs -mkdir test //创建test文件夹
hadoop fs -ls //查看/user/hadoop目录中所有文件
hadoop fs -put .bashrc test //上传到test文件夹中
hadoop fs -get test /usr/local/hadoop //复制文件
~~~



