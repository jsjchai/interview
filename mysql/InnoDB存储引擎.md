## InnoDB简介
> InnoDB是一种兼顾了高可靠性和高性能的通用存储引擎。在MySQL 5.6中，InnoDB是默认的MySQL存储引擎。除非您配置了其他默认存储引擎，否则发出CREATE TABLE不带ENGINE 子句的语句将创建一个InnoDB表

## 主要优势
* 其DML操作遵循ACID模型，并具有具有提交，回滚和崩溃恢复功能的事务，以保护用户数据
* 行级锁定和Oracle风格的一致读取可提高多用户并发性和性能
* InnoDB表将您的数据安排在磁盘上，以基于主键优化查询。每个InnoDB表都有一个称为聚集索引的主键索引，该索引组织数据以最小化主键查找的I/O
* 为了保持数据完整性，InnoDB支持FOREIGN KEY约束。使用外键检查插入，更新和删除操作，以确保它们不会导致相关表之间的不一致

## 使用InnoDB表的好处
* 如果服务器由于硬件或软件问题而意外退出，无论当时数据库中发生了什么，重新启动数据库后都无需执行任何特殊操作。InnoDB崩溃恢复会自动完成在崩溃之前提交的更改，并撤消正在处理但尚未提交的更改，从而使您可以从上次中断的地方重新启动并继续
* 该InnoDB存储引擎维护自己的缓冲池，在主内存缓存表和索引数据作为数据被访问
* 如果数据在磁盘或内存中损坏，则校验和机制会在使用前提醒您注意虚假数据
* InnoDB 为处理大量数据时的CPU效率和最佳性能而设计

## ACID模型
* 原子性
  * autocommit
  * commit
  * rollback
* 一致性
  * 双InnoDB写缓冲区
  * InnoDB崩溃恢复 
* 隔离性
  * autocommit
  * 事务隔离级别和SET TRANSACTION语句
  * InnoDB 锁定 的底层细节。可以在INFORMATION_SCHEMA表中查看详细信息 
  * MySQL 通过锁机制来保证事务的隔离性
* 持久性
  * 双InnoDB写缓冲区
  *  innodb_flush_log_at_trx_commit
  *  sync_binlog
  *  innodb_file_per_table
  *  存储设备（例如磁盘驱动器，SSD或RAID阵列）中的写缓冲区
  *  存储设备中由电池供电的高速缓存
  *  用来运行MySQL的操作系统，特别是它对fsync()系统调用的支持
  *  不间断电源（UPS）保护运行MySQL服务器并存储MySQL数据的所有计算机服务器和存储设备的电源
  *  备份策略，例如备份的频率和类型以及备份保留期
  *  对于分布式或托管数据应用程序，MySQL服务器的硬件所在的数据中心的特定特性，以及数据中心之间的网络连接
  *  MySQL 使用 redo log 来保证事务的持久性

## InnoDB多版本
> InnoDB是一个多版本的存储引擎。它保留有关已更改行的旧版本的信息，以支持事务功能，例如并发和回滚。此信息以称为回滚段的数据结构存储在系统表空间或撤消表空间中。
* 回滚段中的撤消日志分为插入和更新撤消日志。插入撤消日志仅在事务回滚时才需要，并且在事务提交后可以立即将其丢弃
* InnoDB多版本并发控制（MVCC）对次级索引的处理方式与对聚簇索引的处理方式不同。聚簇索引中的记录将就地更新，其隐藏的系统列指向撤消日志条目，可以从中重建记录的早期版本。与聚簇索引记录不同，辅助索引记录不包含隐藏的系统列，也不会就地更新
* 更新二级索引列时，将对旧的二级索引记录进行删除标记，将新记录插入，并最终清除带有删除标记的记录
* 如果二级索引记录被标记为删除，或者二级索引页被更新的事务更新， 则不使用覆盖索引技术。而不是从索引结构中返回值，而是InnoDB在聚集索引中查找记录

## InnoDB架构
![InnoDB架构](https://dev.mysql.com/doc/refman/5.6/en/images/innodb-architecture.png)

## InnoDB内存结构
* 缓冲池
  * 缓冲池是主内存中的一个区域，在InnoDB访问表和索引数据时会在其中进行 高速缓存。缓冲池允许直接从内存访问经常使用的数据，从而加快了处理速度。在专用服务器上，通常将多达80％的物理内存分配给缓冲池
  * 为了提高大容量读取操作的效率，缓冲池被划分为多个页面，这些页面可以潜在地容纳多行。为了提高缓存管理的效率，缓冲池被实现为页面的链接列表。使用最近最少使用（LRU）算法的变体，将很少使用的数据从缓存中老化掉
* 更改缓冲区
   * 更改缓冲区是一种特殊的数据结构，当二级索引页不在缓冲池中时，这些 更改将缓存这些更改 。当页面通过其他读取操作加载到缓冲池中时，可能由INSERT， UPDATE或 DELETE操作（DML）导致的缓冲更改 稍后合并
* 自适应哈希索引
  * 自适应哈希索引可以InnoDB在不牺牲事务功能或可靠性的情况下，在工作负载和缓冲池有足够内存的适当组合的系统上，更像是内存数据库。自适应哈希索引由innodb_adaptive_hash_index 变量启用 ，或在服务器启动时由禁用 --skip-innodb-adaptive-hash-index
* 日志缓冲区
  * 日志缓冲区是存储区域，用于保存要写入磁盘上的日志文件的数据。日志缓冲区的大小由innodb_log_buffer_size变量定义 。默认大小为16MB。日志缓冲区的内容会定期刷新到磁盘。较大的日志缓冲区使大型事务可以运行，而无需在事务提交之前将重做日志数据写入磁盘。因此，如果您有更新，插入或删除许多行的事务，则增加日志缓冲区的大小可以节省磁盘I/O

## 事务的隔离级别
* 可重复读 REPEATABLE READ
  * InnoDB默认隔离级别 
  *  同一事务中的一致读取将读取由第一次读取建立的快照
* 读提交 READ COMMITTED
  * 即使在同一事务中，每个一致的读取都将设置并读取其自己的新快照
* 读未提交 READ UNCOMMITTED
  * SELECT语句以非锁定方式执行，但是可能会使用行的早期版本。因此，使用此隔离级别，此类读取不一致。这也称为脏读 
* SERIALIZABLE 可串行化
  *  此级别类似于REPEATABLE READ，但是InnoDB将所有普通SELECT 语句隐式转换为SELECT ... LOCK IN SHARE MODEif autocommit禁用。如果 autocommit启用，则 SELECT是其自身的事务。因此，它被认为是只读的，如果作为一致（非锁定）读取执行并且不需要阻塞其他事务，则可以序列化。（SELECT如果其他事务已修改所选行，则要强制平原 阻止，请禁用 autocommit。）

| 隔离级别 | 脏读 |不可重复读|幻读|
| :----:| :----: |:----: |:----: |
| 读未提交| 可以出现 |可以出现 | 可以出现 |
| 读提交| 不允许出现 |可以出现 | 可以出现 |
| 可重复读| 不允许出现 |不允许出现 | 可以出现 |
| 可串行化| 不允许出现 |不允许出现 | 不允许出现 |

## InnoDB锁
* Shared and Exclusive Locks 共享锁和排他锁
  * 共享（S）锁允许持有锁读取行的事务
  * 独占（X）锁允许持有锁，更新或删除行的事务
* Intention Locks 意向锁
* Record Locks 记录锁
* Gap Locks 间隙锁
* Next-Key Locks 下一键锁
* Insert Intention Locks 插入意图锁
* AUTO-INC Locks 自动上锁

## 参考文档
* [事务及其特性](https://developer.ibm.com/zh/technologies/databases/articles/os-mysql-transaction-isolation-levels-and-locks/)
* [The InnoDB Storage Engine](https://dev.mysql.com/doc/refman/5.6/en/innodb-storage-engine.html)
* [InnoDB Locking](https://dev.mysql.com/doc/refman/5.6/en/innodb-locking.html)
