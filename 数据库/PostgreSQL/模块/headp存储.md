heap表引擎存储表数据和表索引

目录：/var/lib/postgresql/data/base

创建表，会做以下事情：

* 在目录中给表创建文件
* 给表创建oid
* 把逻辑表、oid、物理表文件都存储在系统表pg_class的一行数据

查看表实际文件

~~~sql
select pg_relation_filepath('d');
~~~

