# 概念和结构

heap表引擎存储表数据和表索引

目录：/var/lib/postgresql/data/base

创建表，会做以下事情：

* 在目录中给表创建文件（空文件）
* 给表创建oid
* 把逻辑表、oid、物理表文件都存储在系统表pg_class的一行数据

查看表实际文件路径

~~~sql
select pg_relation_filepath('tablename');
~~~

PG默认单表文件大小（--with-segsize）上限是1G，超过1G就会创建新文件，导致文件查询IO增大

表文件内部由PG的page组成（--with-blocksize），page内部由tuple组成，一个page number和tuple offset就能锁定一个tuple

page结构，在page中，page需要提供给外界的信息有这些（page header）：

* lsn：在日志中上一次访问这个page的记录id
* checksum：校验page是否完整
* flags：标识是否有空余的tuple空间，tuple是否对外可见
* lower：空余空间的最低地址
* upper：空余空间的最高地址
* special：特殊空间的地址
* pagesize_version：page大小和版本
* prune_xid：可以覆盖的tuple ID
* line_pointers：各个tuple的地址

tuple结构

* heapTupleData：整条tuple的信息（自身位置、所属的表）

* heapTupleHeaderData：支持事务

* dataArea：存储实际数据，16进制表示数据列

  ~~~sql
  SELECT lp,lp_off, t_xmin, t_ctid, t_attrs FROM heap_page_item_attrs(get_raw_page('c', 0), 'c');
  ~~~

> append-only模式下的update也是insert，从这一点上讲，lsm-tree和heap tuple是类似的

# 写入

heap表的写入并不是将元组构造好插入到page之后直接落盘，而是将插入元组的page标记为 dirty，由专门的 checkpointer 进程进行 周期性 buffer 的刷盘

核心方法，heapam_tuple_insert(\backend\access\heap\heapam_handler.c)、heap_insert(src\backend\access\heap\heapam.c)

步骤如下：

1. 初始化元组头
2. 从 Relation cache 中获取一个 可用的page block-number
3. 做事务上的冲突检测（rw/ww）
4. 将元组信息添加到这个 page中 
5. 标记 page 为dirty 
6. 写 WAL 
7. 标记 relation-cache 中旧的 tuple 所在的buffer 失效

**初始化元组头**

~~~c
// xid 是当前操作元组的 transaction id，这里是更新到了我们前面说的 t_xmin之中。
HeapTupleHeaderSetXmin(tup->t_data, xid);
// 更新 t_cid，即 commid-id，用来标识当前事务操作之前有多少个 command.
HeapTupleHeaderSetCmin(tup->t_data, cid);
// 更新 t_xmax，即 对当前元组发生update 或者 delete 的事务id，默认是0.
HeapTupleHeaderSetXmax(tup->t_data, 0); /* for cleanliness */
// 更新 t_tableOid 为 表 oid.
tup->t_tableOid = RelationGetRelid(relation);
~~~

不知道这和事务有具体什么关系

**获取一个可用的page number**

1. 从relation cache中获取可以用的page number
2. 从`_fsm`文件读取
3. 从`smgr`拿取一块磁盘空间（新建一个page block）
4. 将page读取到buffer中
5. 标记获取到的page为dirty

> 关系缓存（relation cache）是用于存储已经打开的数据库关系（表、索引等）的内存结构。它是一个高效的数据结构，可以减少对磁盘的访问，提高查询性能。
>
> 1. 关系元数据：包括表名、列名、数据类型、约束条件等。
> 2. 关系的物理位置：关系在磁盘上的文件路径和块号。
> 3. 关系的统计信息：包括行数、页数、索引信息等。
> 4. 内部缓存数据结构：用于查询优化和执行的数据结构，如查询计划树、索引扫描信息等。

**写入之前的冲突检测**

直接调用函数`CheckForSerializableConflictIn`

**写入元组到page**

1. 将tuple 的 t_data 数据部分 插入到page中，更新page header信息
2. 将offnum 和 buffer 更新到 tuple 的 t_self中

~~~c
void
RelationPutHeapTuple(Relation relation,
                     Buffer buffer,
                     HeapTuple tuple,
                     bool token)
{
    ...
    // 拿到上文中获取到 buffer ，也就是 block-number,拿到对应的page.
    /* Add the tuple to the page */
    pageHeader = BufferGetPage(buffer);
    // 在宏  PageAddItem 调用 PageAddItemExtended 函数 将 tup 的 data数据添加到page中。
    // 其中 PageAddItemExtended 函数中，会更新 page的 header 信息：
    // 比如 pd_lower, pd_upper, line-pointer 也就是 pd_linp 等
    // 更新完成这一些信息之后会将 tuple 传入的 t_data 信息 memcpy 到page中。
    offnum = PageAddItem(pageHeader, (Item) tuple->t_data,
                         tuple->t_len, InvalidOffsetNumber, false, true);
    ...
    // 将offnum 和 buffer 更新到 tuple 的 t_self中.
    ItemPointerSet(&(tuple->t_self), BufferGetBlockNumber(buffer), offnum);
    ...
~~~

**标记page 为dirty 以及 checkpointer 进程异步刷脏页**

通过 `MarkBufferDirty(buffer);` 对该page 写入 dirty标记。

元组所在page 的实际写盘是通过额外的 checkpoint 进程进行写入的。

checkpointer 进程 是PG 众多子进程中的一个，定期对内存中缓存的数据进行落盘，包括 CLOG(事务状体数据)， 子事务，Relation map, snapshot , buffer pool 脏页（淘汰页）等。

![img](https://pic3.zhimg.com/80/v2-6e4a78819dfb443111b5937749bfe90a_720w.webp)

checkpointer 子进程是在 postgresql 启动时在PostmasterMain 初始化的子进程。 其调用栈如下：

```text
CheckpointerMain
  CreateCheckPoint
    CheckPointGuts
        CheckPointBuffers
```

在函数 `CheckPointBuffers` 中，会先通过 `BufferSync` 将 buffer 中的数据统一通过 posix write 写入到磁盘；再通过`ProcessSyncRequests` 对 buffer写入的page 所处的路径调用 fsync。

函数路径如下

> CheckpointerMain: src\backend\postmaster\checkpointer.c
>
> BufferSync: src\backend\storage\buffer\bufmgr.c

**写入WAL**

如果用户在 `postgresql.conf` 文件中设置了 `wal_level=0`, 则会关闭 WAL 功能。同样，因为 wal 的源码逻辑 是 和事务体系相关联的，所以本篇也暂时不会描述。

代码 还是在 `heap_insert`之中

```c
/* XLOG stuff */
if (!(options & HEAP_INSERT_SKIP_WAL) && RelationNeedsWAL(relation))
{
    ...
}
```

应该是9系版本及以前，PG的WAL 还是同步落盘的，后来为了分离IO调度，又搞了一个wal-writer 子进程，来单独调度写 WAL的逻辑。

**刷新relation cache**

这里是在对元组更新的场景下， 如果这个元组所在的page 存在于 relation-cache 之中， 则需要告诉relation-cache 让这个page失效，因为数据已经发生了变更。 

invalid 的代码在：

```c
void
CacheInvalidateHeapTuple(Relation relation,
                         HeapTuple tuple,
                         HeapTuple newtuple)
{
    ...
}
```

> CacheInvalidateHeapTuple: src\backend\access\heap\heapam.c
>
> 关于 relation-cache 和 catalog（系统表）后续会详细介绍，这里也是拥有非常多的细节。

# 读取

以seqscan为例，函数调用栈如下

~~~
main
 PostmasterMain
  ServerLoop
   BackendStartup
    BackendRun
     PostgresMain
      exec_simple_query # 词法解析/语法解析/优化器
       PortalRun # 已经生产执行计划，开始执行
        PortalRunSelect # 执行计划是 查询，这里和 insert的执行计划是不一样的
         standard_ExecutorRun
          ExecutePlan
           ExecProcNode
            ExecScan
             ExecScanFetch
              SeqNext
               table_scan_getnextslot
                heap_getnextslot
~~~

主体逻辑是在 `heap_getnextslot` 函数中，该函数内部逻辑分为两部分： 1. 从磁盘上读取对应的 tuple 坐在的page 数据到内存中。 2. 将读取上来的tuple 填充到 可以被用户读取到的 `TupleTableSlot` 数据结构中。

第一部分的核心函数`heap_getnextslot`还是在heapam.c中，而第二部分已经是执行器的内容了，和本节没有关系

# 总结

先记一些别的：

1. 可以看到 PG 为了将IO 调度和内核逻辑处理分离，设计了很多子进程进行异步IO调度（所有的IO 调度都会通过 smgr 存储管理器进行），比如堆表文件中的 page落盘，其在 INSERT 的主链路上并不会直接落盘，而是插入到 内存中对应的 pg page区域中，对 page 做一个标记，后续通过 checkpointer 进行 脏页的异步落盘(调用fsync)。
2. 从heap 表引擎设计来看， pg将应用逻辑和io调度分割开，PG 主进程只需要专注于DML 等语句的CPU 逻辑调度，而 IO 层面交给专门的后台进程，对性能更为友好。（**逻辑和性能分开**）

3. 因为IO 调度对于线程来说过于繁重，且 不利于 PG 的架构节藕，所以PG 设计了多进程的应用调度架构，每一个 I/O 繁重且功能集中的 组件交给专门的子进程进行调度，资源利用率更高。当然，多进程架构可扩展性就弱了不少
4. 对DML语句来说，性能影响最大的是WAL写入的性能















