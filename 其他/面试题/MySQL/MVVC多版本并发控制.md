# MVVC 多版本并发控制

MVCC(Multi-Version Concurrency Control多版本并发控制)：

- MVCC每次更新操作都会复制一条新的记录，新纪录的创建时间为当前事务id
- 优势为读不加锁，读写不冲突
- InnoDb存储引擎中，每行数据包含了一些隐藏字段 DATA_TRX_ID，DATA_ROLL_PTR，DB_ROW_ID，DELETE BIT
- DATA_TRX_ID 字段记录了数据的创建和删除时间，这个时间指的是对数据进行操作的事务的id
- DATA_ROLL_PTR 指向当前数据的undo log记录，回滚数据就是通过这个指针
- DELETE BIT位用于标识该记录是否被删除，这里的不是真正的删除数据，而是标志出来的删除。真正意义的删除是在mysql进行数据的GC，清理历史版本数据的时候。

具体的DML：

- INSERT：创建一条新数据，DB_TRX_ID中的创建时间为当前事务id，DB_ROLL_PT为NULL
- UPDATE：将当前行的DB_TRX_ID中的删除时间设置为当前事务id，DELETE BIT设置为1
- DELETE：复制了一行，新行的DB_TRX_ID中的创建时间为当前事务id，删除时间为空，DB_ROLL_PT指向了上一个版本的记录，事务提交后DB_ROLL_PT置为NULL

为了提高并发度，InnoDb提供了这个「非锁定读」，即不需要等待访问行上的锁释放，读取行的一个快照即可。 既然是多版本读，那么肯定读不到隔壁事务的新插入数据了，所以解决了幻读。



## 快照读与当前读

在MVCC并发控制中，读操作可以分成两类：快照读 (snapshot read)与当前读 (current read)。

快照读，读取的是记录的可见版本 (有可能是历史版本)，不用加锁。

当前读，读取的是记录的最新版本，并且，**当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录**。 

在一个支持MVCC并发控制的系统中，哪些读操作是快照读？哪些操作又是当前读呢？以MySQL InnoDB为例： 

- **快照读：**简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)

  - select * from table where ?; 

- **当前读：**特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。

  - select * from table where ? lock in share mode;
  - select * from table where ? for update;
  - insert into table values (…);
  - update table set ? where ?;
  - delete from table where ?;

  所有以上的语句，都属于当前读，读取记录的最新版本。并且，读取之后，还需要保证其他并发事务不能修改当前记录，对读取记录加锁。其中，除了第一条语句，对读取记录加S锁 (共享锁)外，其他的操作，都加的是X锁 (排它锁)。 



## MVVC 与间隙锁

MySQL 中在可重复读的隔离级别中解决了幻读。主要使用两种技术互相配合：MVVC 和 间隙锁。

MVVC 的意思是多版本并发控制，是一种乐观锁，间隙锁是悲观锁。

行锁和间隙锁组合起来就叫Next-Key Lock。

MVVC 中有快照读和当前读两种概念。

对于快照读来说，你随便读我不管你，反正你读的是历史数据，绝对不会出现幻读。

对于当前读来说，需要使用 Next-Key Lock，以实现对写数据进行阻止，防止出现幻读。

通过利用这两种方式，平衡了并发性能和数据安全。