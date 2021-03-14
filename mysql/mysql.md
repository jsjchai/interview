
## 日志

| 日志类型 | 描述 |
| :----:| :----: |
| Error log | 启动，运行或停止mysqld遇到的问题 | 
| General query log | 建立的客户连接和从客户那里收到的陈述 |
| Binary log | 更改数据的语句（也用于复制） |
| Relay log | 从复制源服务器收到的数据更改 |
|Slow query log|long_query_time执行耗时超过几秒钟的查询|
|DDL log (metadata log)|DDL语句执行的元数据操作|

* [binlog](https://dev.mysql.com/doc/refman/5.6/en/binary-log.html) 归档日志
  * 二进制日志包含描述数据库更改（例如表创建操作或表数据更改）的“事件”。它还包含针对可能进行了更改的语句的事件（例如， DELETE不匹配任何行的a），除非使用基于行的日志记录。二进制日志还包含有关每个语句花费该更新数据多长时间的信息。 
  * 它记录了所有的 DDL 和 DML 语句（除了数据查询语句select、show等），以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。binlog 的主要目的是复制和恢复 
  * 二阶段提交
* [redolog](https://dev.mysql.com/doc/refman/5.6/en/innodb-redo-log.html) 重做日志 InnoDB
  * 重做日志是基于磁盘的数据结构，在崩溃恢复期间用于纠正不完整事务写入的数据 
  * InnoDB存储引擎层的日志，又称重做日志文件，用于记录事务操作的变化，记录的是数据修改之后的值，不管事务是否提交都会记录下来
  * 二阶段提交
* [undolog](https://dev.mysql.com/doc/refman/5.6/en/innodb-undo-logs.html) 回滚日志 InnoDB
  * 撤消日志是与单个读写事务关联的撤消日志记录的集合。撤消日志记录包含有关如何撤消事务对聚集索引记录的最新更改的信息。如果另一个事务需要将原始数据作为一致读操作的一部分来查看，则从undo日志记录中检索未修改的数据。Undo日志存在于Undo日志段中，Undo日志段包含在回滚段中。默认情况下，回滚段在物理上是system表空间的一部分，但是它们也可以位于undo表空间中 
  * 保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读
