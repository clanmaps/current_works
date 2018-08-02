配置文件
-------
**config.h**
定义一系列全局配置相关的宏。这些宏的具体含义见```README.txt```。
**config-std.h**
和```config.h```内容相同。
**config.cpp**
*设置tpcc事务类型和TestCases类型。*
**makefile**
编译使用文件。
**LICENSE**  

system
------
**global.h**  
声明各全局变量。
**global.cpp**
主要将```global.h```中声明的全局变量赋值，赋的是```config.h```中对应的宏的值。另外定义一些全局管理器，指定负载中表的大小等。
**parser.cpp**
解析命令行参数。命令行参数可用来配置一些全局变量，例如同时运行线程个数、系统逻辑分区数、每个线程涉及的逻辑分区数等等。-h可查看具体用法。 
**mem_aloc.h**  & **mem_aloc.cpp**
声明并定义内存分配器类型mem_alloc、内存单位类型Arena。  
分配规则：每个线程分配4个arena，这4个arena中每个块的大小分别为32字节、64字节、256字节、1024字节，  
**stats.h** & **stats.cpp** 
声明并定义统计信息类型Stats，在Stats中有全局统计信息（死锁检测时间、死锁等待时间、环路检测次数、死锁次数），还为每个线程分配一个Stats_thd类型记录该线程的统计信息。
**manager.h** & **manager.cpp**  
声明并定义全局管理器类型manager。全局管理器管理着几个协议需要的参数，如timestamp、epoch，同时还包含中央锁管理机制、事务管理器(txn_man)。其中，锁机制是将记录哈希分布到BUCKET_CNT=31个组中，按组加锁；每个线程分配一个事务管理器。
**wl.h** & **wl.cpp**
声明并定义所有负载的基类workload，实现各种负载相同的初始化和索引插入操作。
**helper.h** & **helper.cpp**
声明并定义一些整个系统中都常用的函数如get_sys_clock()。此外，还定义了两个类，itemid_t类是数据项类型，指明某种type（可能是table/page/row）的数据是否有效、位置在哪；myrand类产生随机数。
**query.h** & **query.cpp**
Query_queue类：这是一个总的队列对象，为避免冲突，其内部的all_queries数组为每个线程分配一个查询队列Query_thd*。
Query_thd类：管理具体的查询队列如tpcc_query*,记录数组中当前查询的位置q_idx。
base_query类：query基类，由具体负载的query继承。
**thread.h** & **thread.cpp**


**txn.h** & **txn.cpp**  






benchmarks
----------
**TPCC_short_schema.txt**  
tpcc较小版负载schema，减少了一些表的列数和**索引**以适应内存不足的情况。  
**TPCC_full_schema.txt**  
tpcc完整版负载schema。  
**tpcc.h**  
tpcc_wl类：继承自workload，包含tpcc需要的9张表以及对应索引。
tpcc_txn_man类：继承自txn_man，负责跑tpcc的5种事务（实际只实现了两种）。  
**tpcc_const.h**   
定义了9个枚举类，分别对应tpcc中9张表的各列。每个枚举类枚举出一张表的每个列。
**tpcc_helper.h** & **tpcc_helper.cpp**  
声明并定义随机产生TPCC负载所需的各种函数，例如生成随机数、随机字符串、用多个字段值生成索引键值。
**tpcc_wl.cpp**  
定义tpcc_wl类。初始化9张表的模式和索引，并为其中7张表插入随机生成的数据，表ORDER和ORDER-LINE中没有数据。  
**tpcc_query.h** & **tpcc_query.cpp**  
tpcc_query类：继承base_query类，包含TPCC事务需要的输入，并能随机产生一条查询需要的各个字段的数据（即输入）。调用它的init()函数,随机产生payment或new_order类型事务的所需字段值。
**tpcc_txn.cpp**  
定义tpcc_txn_man类，
 



 

concurrency_control
-------------------

libs
----
**libjemalloc.a**  
makefile中编译使用到的静态链接库，没发现有什么用，删掉之后也能正常编译运行。  

storage
-------
**catalog.h** & **catalog.cpp**  
Catalog类：记录表的模式信息。每张表一个Catalog，包含每一列对应的Column类，可方便获得每列的模式信息。  
Column类：记录每个列模式信息。包含列名、列类型、列大小、列偏移、列id。
**row.h** & **row.cpp**
声明并定义row_t类存储每行数据以及改变这些数据的方法。针对不同的并发控制协议，row.h中声明不同的行管理器(manager)。协议x的行管理器声明在row_x.h中。*注：不是每行都有manager。*
**table.h** & **table.cpp** 
table_t类很简单，只要包含表的表名、当前表大小、Catalog。同时实现向表中插入新行的方法。连delete_row()都尚未支持。直接从一个table_t实例中找不到它拥有的元组，
**index_base.h**
声明索引的基类index_base，btree和hash索引都继承自它。
**index_hash.h** & **index_hash.cpp**
声明并定义哈希索引类IndexHash。每个分区对应一个完整的哈希表，哈希表是共享的数据结构，写需要按桶加锁，读不加。  
![index_hash](https://i.loli.net/2018/07/23/5b55cd85f416e.png)
**index_btree.h** & **index_btree.cpp**

