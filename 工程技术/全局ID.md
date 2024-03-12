# 雪花算法

雪花算法是由Twitter开源的分布式ID生成算法，以划分命名空间的方式将 64-bit位分割成多个部分，每个部分代表不同的含义。而 Java中64bit的整数是Long类型，所以在 Java 中 SnowFlake 算法生成的 ID 就是 long 来存储的。

- **第1位**占用1bit，其值始终是0，可看做是符号位不使用。
- **第2位**开始的41位是时间戳，41-bit位可表示2^41个数，每个数代表毫秒，那么雪花算法可用的时间年限是`(1L<<41)/(1000L360024*365)`=69 年的时间。
- **中间的10-bit位**可表示机器数，即2^10 = 1024台机器，但是一般情况下我们不会部署这么台机器。如果我们对IDC（互联网数据中心）有需求，还可以将 10-bit 分 5-bit 给 IDC，分5-bit给工作机器。这样就可以表示32个IDC，每个IDC下可以有32台机器，具体的划分可以根据自身需求定义。
- **最后12-bit位**是自增序列，可表示2^12 = 4096个数。

这样的划分之后相当于**在一毫秒一个数据中心的一台机器上可产生4096个有序的不重复的ID**。但是我们 IDC 和机器数肯定不止一个，所以毫秒内能生成的有序ID数是翻倍的。

**雪花算法提供了一个很好的设计思想，雪花算法生成的ID是趋势递增，不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的，而且可以根据自身业务特性分配bit位，非常灵活**。

但是不同IDC、甚至是相同IDC中的不同机器的系统时间都会有差异，而雪花算法强**依赖机器时钟**，如果发现时钟回拨，会导致发号重复或者服务会处于不可用状态。如果恰巧回退前生成过一些ID，而时间回退后，生成的ID就有可能重复。官方对于此并没有给出解决方案，而是简单的抛错处理，这样会造成在时间被追回之前的这段时间服务不可用。

# 百度开源UidGenerator

基于Snowflake算法的唯一ID生成器。而且，它非常适合虚拟环境，比如：Docker。另外，它通过消费未来时间克服了雪花算法的并发限制。UidGenerator提前生成ID并缓存在RingBuffer中。 压测结果显示，单个实例的QPS能超过6000,000。运行环境：JDK8+、MySQL/PG

CachedUidGenerator方式主要通过采取如下一些措施和方案规避了时钟回拨问题和增强唯一性：

**自增列**：UidGenerator的workerId在实例每次重启时初始化，且就是[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)的自增ID，从而完美的实现每个实例获取到的workerId不会有任何冲突。

**RingBuffer**：UidGenerator不再在每次取ID时都[实时计算](https://cloud.tencent.com/product/oceanus?from_column=20065&from=20065)分布式ID，而是利用RingBuffer数据结构预先生成若干个分布式ID并保存。

**时间递增**：传统的雪花算法实现都是通过System.currentTimeMillis()来获取时间并与上一次时间进行比较，这样的实现严重依赖[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)的时间。而UidGenerator的时间类型是AtomicLong，且通过incrementAndGet()方法获取下一次的时间，从而脱离了对服务器时间的依赖，也就不会有时钟回拨的问题（这种做法也有一个**小问题**，即分布式ID中的时间信息可能并不是这个ID真正产生的时间点，例如：获取的某分布式ID的值为3200169789968523265，它的反解析结果为{"timestamp":"2019-05-02 23:26:39","workerId":"21","sequence":"1"}，但是这个ID可能并不是在"2019-05-02 23:26:39"这个时间产生的）。

![image-20240312112610672](C:\Users\NetPunk\AppData\Roaming\Typora\typora-user-images\image-20240312112610672.png)

# 美团开源—Leaf

大部分在线服务的分布式ID都是公司内部单独做的，所以这些开源方案只能是参考，这里美团的只贴一下链接

[Leaf：美团分布式ID生成服务开源 - 美团技术团队 (meituan.com)](https://tech.meituan.com/2019/03/07/open-source-project-leaf.html)

