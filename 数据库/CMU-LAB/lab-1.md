# Lab-缓存池

实现一个在bustub中可用的缓存池，支持**多线程**，因此需要使用锁来构建所有可访问的数据结构。有以下要注意的地方

* 不要更改代码中API的标识符，否则编译错误
* 不要增加额外的类(辅助类)，但是可以增加辅助函数
* 可以使用C++17的原生容器，但是如果它们是线程不安全的，则要加额外的锁
* 不能用那些牛逼的包（比如boost）

## Task1-LRU Replacement Policy

**背景**

和操作系统中的LRU概念一样，LRU会维护每个页面上次访问的时间戳。该时间戳可以存储在单独的数据结构中，例如队列，以便进行排序并提高效率。DBMS选择退出具有最早时间戳的页面。此外，页面按排序顺序保存，以减少逐出时的排序时间。但是实现上这种算法开销大到无法负担。

在`src/include/buffer/replacer.h`里面定义了`Replacer`抽象类，需要在`src/buffer/lru_replacer.cpp`中进行实现`LRUReplacer`，这个数据结构包含即将要被替换的page，对它的说明如下：

* LRUReplacer的最大页数与缓冲池的大小相同，因为它包含BufferPoolManager中所有帧的占位符
* 在任何给定时刻，并不是所有的帧都在LRUReplacer中
* LRUReplacer初始化为没有帧
* 只有被标记的帧才会被视为在LRUReplacer

你需要实现在课上讲过的LRU policy，具体的讲，要实现以下方法：

* `Victim(frame_id_t*)`：移除由`Replacer`追踪的所有其他元素相比最近访问最少的对象，存储对象内容到输出参数中并且返回**True**，如果`Replacer`不追踪任何对象，返回**False**。
* `Pin(frame_id_t)`：当页被标记到`BufferPoolManager`中的帧时，调用这个方法移除在`LRUReplacer`中对应的帧。
* `Unpin(frame_id_t)`：当页中的pin_count变为0时，调用这个方法将包含这个页的帧添加到`LRUReplacer`中。
* `Size()`：此方法返回`LRUReplacer`中当前的帧数。

你可以假设内存是无限的，并且你只需实现LRU更换策略。（虽然有有相应的文件，不必实现时钟替换策略）

## Task2-BUFFER POOL MANAGER INSTANCE

**背景**

需要在系统中实现缓冲池管理器（BufferPoolManagerInstance）。BufferPoolManagerInstance负责从DiskManager获取数据库页并将其存储在内存中。BufferPoolManagerInstance可以在两种情况下将脏页写入磁盘。

* 显式指示BufferPoolManagerInstance将脏页写入磁盘时
* 或者当它需要逐出页以为新页腾出空间时

为了确保您的实现与系统的其余部分正确配合，我们将为您提供一些已经填写的功能。您也不需要实现实际将数据读写到磁盘的代码（在我们的实现中称为DiskManager）。我们将为您提供该功能。

您的BufferPoolManagerInstance实现将使用您在此分配的前面步骤中创建的LRUReplacer类。它将使用LRUReplacer跟踪何时访问页面对象，以便在必须释放一个框架以腾出空间从磁盘复制新的物理页面时，决定退出哪个对象。

系统中的所有内存页都由页对象表示。要点如下：

* 页面对象的标识符（Page_id）跟踪它包含的物理页面；如果页面对象不包含物理页面，则其Page_id必须设置为INVALID_Page_id。
* BufferPoolManagerInstance将重用相同的页面对象来存储来回移动到磁盘的数据。对应的方法是`Flush`相关函数。
  * 每个页面对象还跟踪它是否脏。您的工作是记录页面在取消固定之前检查数据是否已修改。
  * BufferPoolManagerInstance必须将脏页的内容写回磁盘，然后才能重用该对象。
* 每个页面对象还维护一个计数器，用于“固定”该页面的线程数。不允许BufferPoolManagerInstance释放已固定的页面。

需要实现的源文件为`src/buffer/buffer_pool_manager_instance.cpp`，其头文件定义在`src/include/buffer/buffer_pool_manager_instance.h`

需要实现的函数为：

- `FetchPgImp(page_id)`：从物理磁盘中获取page到内存中
- `UnpinPgImp(page_id, is_dirty)`：`is_dirty`参数跟踪页面在固定时是否被修改。
- `FlushPgImp(page_id)`：刷新page，无论其pin状态如何
- `NewPgImp(page_id)`：
- `DeletePgImp(page_id)`：
- `FlushAllPagesImpl()`：刷新所有page，无论其pin状态如何

注意：LRUReplacer和BufferPoolManagerInstance上下文中的Pin和Unpin具有相反的含义。在LRUReplacer的上下文中，固定页面意味着我们不应该因为页面正在使用而逐出该页面。这意味着我们应该将其从LRUReplacer中删除。另一方面，将页面固定在BufferPoolManagerInstance中意味着我们想要使用页面，并且不应该将其从缓冲池中删除。

## Task3-PARALLEL BUFFER POOL MANAGER

**背景**

正如您在上一个任务中可能注意到的那样，单缓冲池管理器实例需要使用锁存器才能保证线程安全。这可能会导致大量争用，因为每个线程在与缓冲池交互时都会争夺单个锁存器。一种可能的解决方案是在系统中有多个缓冲池，每个缓冲池都有自己的线程锁。

* ParallelBufferPoolManager是一个包含多个BufferPoolManagerInstance的类。对于每个操作，ParallelBufferPoolManager都会选择一个BufferPoolManagerInstance并委托给该实例。

* 使用模运算符，将page_id映射到正确的范围，进而确定使用哪个特定的BufferPoolManagerInstance。

* 首次实例化ParallelBufferPoolManager时，它的起始索引应为0。每次创建新页面时，您都将尝试每个BufferPoolManagerInstance，从起始索引开始，直到一个成功。然后将起始索引增加1。

* 确保在创建单个BufferPoolManagerInstances时使用接受uint32_t num_instances和uint32_t instance_index索引的构造函数，以便正确创建page_id。

需要实现的源文件为`src/buffer/parallel_buffer_pool_manager.cpp`，其头文件定义在`src/include/buffer/parallel_buffer_pool_manager.h`

需要实现的函数为：

- `ParallelBufferPoolManager(num_instances, pool_size, disk_manager, log_manager)`：初始化线程数量，池大小、磁盘管理器、日志管理器
- `~ParallelBufferPoolManager()`：析构函数
- `GetPoolSize()`：
- `GetBufferPoolManager(page_id)`：
- `FetchPgImp(page_id)`：
- `UnpinPgImp(page_id, is_dirty)`：
- `FlushPgImp(page_id)`：
- `NewPgImp(page_id)`：
- `DeletePgImp(page_id)`：
- `FlushAllPagesImpl()`：

# 我的实现

上面说了很多项目相关的要求，简单总结就是实现一个可用的缓冲池

缓冲池是数据库的缓存的抽象，缓存是数据库内存空间在磁盘上的拓展

































