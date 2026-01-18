本文基于[mysql 5.7 官方参考手册](https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html)

# 14.1 InnoDB简介

InnoDB是一种兼顾高可靠性和高性能的通用存储引擎。在 MySQL5.7 中，InnoDB是默认的 MySQL 存储引擎。除非您配置了不同的默认存储引擎，否则发出[`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)不带`ENGINE` 子句的语句会创建一个InnoDB表。	

## InnoDB 的主要优势

- 它的 DML 操作遵循 ACID 模型，事务具有提交、回滚和崩溃恢复功能，以保护用户数据。请参阅[第 14.2 节，“InnoDB 和 ACID 模型”](https://dev.mysql.com/doc/refman/5.7/en/mysql-acid.html)。
- 行级锁和 Oracle 风格的一致读取提高了多用户的并发性和性能。请参阅 [第 14.7 节，“InnoDB 锁定和事务模型”](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-transaction-model.html)。
- InnoDB表将**您的数据以基于主键的方式排列在磁盘上**来优化查询。每个InnoDB表都有一个称为聚集索引（clustered index）的主键索引，用于组织数据，这样可以在主键查找时的最小化 I/O。请参阅[第 14.6.2.1 节，“聚集索引和二级索引”](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)。
- 为维护数据完整性，InnoDB支持 `FOREIGN KEY`约束。使用外键，检查插入、更新和删除以确保它们不会导致相关表之间的不一致。请参见 [第 13.1.18.5 节，“外键约束”](https://dev.mysql.com/doc/refman/5.7/en/create-table-foreign-keys.html)。

**表 14.1 InnoDB 存储引擎特性**

| 特征                                                        | 支持                                                         |
| :---------------------------------------------------------- | :----------------------------------------------------------- |
| **B树索引**                                                 | Yes                                                          |
| **备份/时间点恢复**（在服务器中实现，而不是在存储引擎中。） | Yes                                                          |
| 集群数据库支持                                              | 不                                                           |
| **Clustered indexes聚集索引**                               | Yes                                                          |
| 压缩数据                                                    | Yes                                                          |
| **数据缓存**                                                | Yes                                                          |
| 加密数据                                                    | Yes（通过加密函数在服务器中实现；在 MySQL 5.7 及更高版本中，支持静态数据加密。） |
| **外键支持**                                                | Yes                                                          |
| **全文检索索引**                                            | Yes（MySQL 5.6 及更高版本支持 FULLTEXT 索引。）              |
| **地理空间数据类型支持**                                    | Yes                                                          |
| **地理空间索引支持**                                        | Yes（MySQL 5.7 及更高版本支持地理空间索引。）                |
| **哈希索引**                                                | **NO（InnoDB 在内部利用哈希索引来实现其自适应哈希索引功能）** |
| **索引缓存**                                                | Yes                                                          |
| **锁定粒度**                                                | 行                                                           |
| **MVCC**                                                    | Yes                                                          |
| 复制支持（在服务器中实现，而不是在存储引擎中）              | Yes                                                          |
| 存储限制                                                    | 64TB                                                         |
| T-tree 索引                                                 | 不                                                           |
| Transactions事务                                            | Yes                                                          |
| 更新数据字典的统计信息                                      | Yes                                                          |

注意，**InnoDB不支持hash索引**，会根据表的使用情况自动为表生成哈希索引来提高查询性能，**不能人为干预是否在一张表中生成哈希索引**。

## 14.1.1 使用InnoDB表的好处

InnoDB 表有以下好处：

- 如果服务器由于硬件或软件问题而意外退出，无论当时数据库中发生了什么，您都无需在重新启动数据库后执行任何特殊操作。InnoDB崩溃恢复会自动完成崩溃之前提交的更改，并撤消正在进行但未提交的更改，允许您重新启动并从上次中断的地方继续。请参阅 [第 14.19.2 节，“InnoDB 恢复”](https://dev.mysql.com/doc/refman/5.7/en/innodb-recovery.html)。
- 该InnoDB存储引擎维护自己的缓冲池，在主内存缓存表和索引数据作为数据被访问。经常使用的数据直接从内存中处理。此缓存适用于多种类型的信息并加快处理速度。在专用数据库服务器上，多达 80% 的物理内存通常分配给缓冲池。请参见[第 14.5.1 节，“缓冲池”](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html)。
- 如果将相关数据拆分到不同的表中，则可以设置强制参照完整性的外键。请参见 [第 13.1.18.5 节，“外键约束”](https://dev.mysql.com/doc/refman/5.7/en/create-table-foreign-keys.html)。
- 如果磁盘或内存中的数据损坏，校验和机制会在您使用之前提醒您注意虚假数据。该 [`innodb_checksum_algorithm`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_checksum_algorithm) 变量定义了 使用的校验和算法InnoDB。
- 当您为每个表设计具有适当主键列的数据库时，会自动优化涉及这些列的操作。在[`WHERE`](https://dev.mysql.com/doc/refman/5.7/en/select.html) 子句、[`ORDER BY`](https://dev.mysql.com/doc/refman/5.7/en/select.html)子句、 [`GROUP BY`](https://dev.mysql.com/doc/refman/5.7/en/select.html) 子句和连接操作中引用主键列的速度非常快 。请参阅 [第 14.6.2.1 节，“聚集索引和二级索引”](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)。
- 插入、更新和删除通过称为更改缓冲的自动机制进行优化。InnoDB 不仅允许对同一表进行并发读写访问，它还缓存更改的数据以简化磁盘 I/O。请参阅 [第 14.5.2 节，“更改缓冲区”](https://dev.mysql.com/doc/refman/5.7/en/innodb-change-buffer.html)。
- 性能优势不仅限于具有长时间运行查询的大型表。当从表中一遍又一遍地访问相同的行时，自适应哈希索引会接管以使这些查找更快，就好像它们来自哈希表一样。请参阅[第 14.5.3 节，“自适应哈希索引”](https://dev.mysql.com/doc/refman/5.7/en/innodb-adaptive-hash.html)。
- 您可以压缩表和关联的索引。请参阅 [第 14.9 节，“InnoDB 表和页面压缩”](https://dev.mysql.com/doc/refman/5.7/en/innodb-compression.html)。
- 您可以加密您的数据。请参阅 [第 14.14 节，“InnoDB 静态数据加密”](https://dev.mysql.com/doc/refman/5.7/en/innodb-data-encryption.html)。
- 您可以创建和删除索引以及执行其他 DDL 操作，而对性能和可用性的影响要小得多。请参阅 [第 14.13.1 节，“在线 DDL 操作”](https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html)。
- 截断每个表的文件表空间非常快，可以释放磁盘空间供操作系统重用，而不仅仅是InnoDB. 请参阅 [第 14.6.3.2 节，“每个表的文件表空间”](https://dev.mysql.com/doc/refman/5.7/en/innodb-file-per-table-tablespaces.html)。
- 表数据的存储布局对于[`BLOB`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)长文本字段和`DYNAMIC`行格式更有效 。请参阅 [第 14.11 节，“InnoDB 行格式”](https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html)。
- 您可以通过查询`INFORMATION_SCHEMA`表来监控存储引擎的内部工作。请参阅 [第 14.16 节，“InnoDB INFORMATION_SCHEMA 表”](https://dev.mysql.com/doc/refman/5.7/en/innodb-information-schema.html)。
- 您可以通过查询 Performance Schema 表来监控存储引擎的性能详细信息。请参阅 [第 14.17 节，“InnoDB 与 MySQL Performance Schema 的集成”](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-schema.html)。
- 您可以将InnoDB表与来自其他 MySQL 存储引擎的表混合使用，即使在同一语句中也是如此。例如，您可以使用连接操作在单个查询中组合来自表InnoDB和 [`MEMORY`](https://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html)表的数据 。
- InnoDB 专为处理大量数据时的 CPU 效率和最大性能而设计。
- InnoDB 表可以处理大量数据，即使在文件大小限制为 2GB 的操作系统上也是如此。

对于InnoDB可以应用于MySQL服务器和应用程序代码的特定调整技术，请参阅 [第 8.5 节“优化 InnoDB 表”](https://dev.mysql.com/doc/refman/5.7/en/optimizing-innodb.html)。

## 14.1.2 InnoDB 表的最佳实践

本节介绍使用InnoDB表格时的最佳做法 

- 使用最常查询的一列或多列为每个表指定一个主键，如果没有明显的主键，则指定一个自增值。
- 在根据多个表中的相同 ID 值从多个表中提取数据的地方使用连接。为了快速连接性能，在连接列上定义外键，并在每个表中用相同的数据类型声明这些列。添加外键确保引用的列被索引，这可以提高性能。外键还会将删除和更新传播到所有受影响的表，并在父表中不存在相应 ID 时阻止在子表中插入数据。
- 关闭自动提交。每秒提交数百次会限制性能（受存储设备的写入速度限制）。
- 通过用`START TRANSACTION`和 `COMMIT`语句将相关的 DML 操作集分组到事务中。虽然您不想太频繁地提交，但您也不想发出大量运行数小时而不提交的 [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)、 UPDATE、 或 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)语句。
- 不要使用[`LOCK TABLES`](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html) 语句。InnoDB可以在不牺牲可靠性或高性能的情况下同时处理对同一个表的所有读取和写入的多个会话。要获得对一组行的独占写访问权限，请使用 [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)语法锁定您打算更新的行。
- 启用 [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) 变量或使用通用表空间将表的数据和索引放入单独的文件而不是系统表空间。[`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_file_per_table) 默认情况下启用该 变量。
- 评估您的数据和访问模式是否受益于InnoDB表或页面压缩功能。您可以在InnoDB不牺牲读/写能力的情况下压缩表。
- 使用[`--sql_mode=NO_ENGINE_SUBSTITUTION`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_sql_mode) 选项运行服务器 以防止使用您不想使用的存储引擎创建表。

## 14.1.3 验证InnoDB是否为默认的存储引擎

```sql
mysql> SHOW ENGINES;
```

在default中support列查找

# 14.3 InnoDB的MVCC

InnoDB是一个多版本存储引擎。它保留了关于改变行的旧版本的信息，以支持事务性功能，如并发和回滚。这些信息被存储在系统表空间或undo表空间中的一个叫做回滚段的数据结构中。InnoDB使用回滚段中的信息来执行事务回滚中需要的撤销操作。它还使用这些信息来建立一个行的早期版本，以便进行一致读取。参见章节14.7.2.3, "一致的无锁读取"。

在内部，InnoDB为存储在数据库中的每一条记录添加三个字段：

- 一个6字节的DB_TRX_ID字段表示插入或更新该行的最后一个事务的标识符。另外，一个删除在内部被视为一个更新，在行中的一个特殊位设置标记为删除。

- 一个7字节的DB_ROLL_PTR字段称为roll pointer。roll pointer指向写在回滚段上的undo日志记录。如果该行被更新了，undo日志记录包含了重建该行被更新前内容的必要信息。

- 一个6字节的DB_ROW_ID字段包含一个行的ID，这个ID随着新行的插入而单调地增加。如果InnoDB自动生成了一个聚集索引，那么该索引就包含了行ID值。否则，DB_ROW_ID列不会出现在任何索引中。

在回滚段的undo日志被分为insert和update undoLog。insert undoLog只在事务回滚中需要，并且在事务提交后可以立即丢弃。update undoLog也用于一致读取，但是它们只能在没有InnoDB为其分配快照的事务出现后被丢弃，在一致读取中可能需要update undoLog中的信息来建立数据库行的早期版本。

建议你定期提交事务，包括只发出一致读取的事务。否则，InnoDB不能丢弃update undoLog中的数据，并且回滚段可能会增长得太大，填满它所在的表空间。关于管理undoLog表空间的信息，请参见第14.6.3.4节，"undoLog表空间"。

回滚段中undoLog记录的物理大小通常比相应的插入或更新的行小。你可以使用这个信息来计算你的回滚段所需要的空间。

--------

在InnoDB的多版本模式中，当你用SQL语句删除一条记录时，它不会立即从数据库中被物理删除。InnoDB只有在丢弃为删除而写的update undoLog记录时，才会物理上删除相应的行和它的索引记录。这个删除操作被称为清除(purge)，它是相当快的，通常与进行删除的SQL语句的时间顺序相同。

如果你在表中以相同的速度小批量地插入和删除行，清除线程就会开始落后，并且由于所有的 "死 "行，表会越来越大，使得所有的东西都被磁盘束缚，非常缓慢。在这种情况下，要限制新行的操作，并通过调整innodb_max_purge_lag系统变量为清除线程分配更多资源。欲了解更多信息，请参见第14.8.10节，"清除配置"。

#### 多版本和二级索引
InnoDB多版本并发控制（MVCC）对待二级索引的方式与聚集索引不同。聚集索引中的记录是就地更新的，其隐藏的系统列指向undoLog条目，可以从中重建记录的早期版本。与聚集索引记录不同，二级索引记录不包含隐藏的系统列，也不在原地更新。

当一个二级索引列被更新时，旧的二级索引记录被标记删除，新的记录被插入，而被删除标记的记录最终被清除。当一个二级索引记录被标记删除或者二级索引页被一个较新的事务更新时，InnoDB在聚集索引中查找数据库记录。在聚集索引中，记录的DB_TRX_ID被检查，如果该记录在读取事务开始后被修改，则从undo log 中检索出该记录的正确版本。

如果一个二级索引记录被标记为删除，或者二级索引页被一个较新的事务更新，覆盖索引技术就不会被使用。InnoDB不会从索引结构中返回值，而是在聚集索引中查找记录。

然而，如果启用了索引条件下推（ICP）优化，并且WHERE条件的一部分可以只使用索引中的字段进行评估，MySQL服务器仍然将WHERE条件的这一部分下推到存储引擎，在那里使用索引对其进行评估。如果没有找到匹配的记录，就避免了聚集索引的查找。如果找到了匹配的记录，甚至在有删除标记的记录中，InnoDB在聚集索引中查找记录。

# 14.4 InnoDB的架构

下图显示了构成InnoDB存储引擎架构的内存和磁盘结构。有关每个结构的信息，请参阅 [第 14.5 节“InnoDB 内存结构”](https://dev.mysql.com/doc/refman/5.7/en/innodb-in-memory-structures.html)和 [第 14.6 节“InnoDB 磁盘结构”](https://dev.mysql.com/doc/refman/5.7/en/innodb-on-disk-structures.html)。

![innodb-architecture](images/innodb-architecture.png)



# 14.5 InnoDB的内存结构

## 14.5.1 Buffer Pool

缓冲池是主内存中的一个区域，用于在InnoDB访问时缓存表和索引数据。缓冲池允许直接从内存访问经常使用的数据，从而加快处理速度。在专用服务器上，多达 80% 的物理内存通常分配给缓冲池。

为了提高high-volume读取操作的效率，缓冲池被划分为可能包含多行的pages页。为了缓存管理的效率，缓冲池buffer pool被实现为 linked list of pages；很少使用的数据使用最近最少使用 (LRU) 算法的变体从缓存中老化。

了解如何利用缓冲池将经常访问的数据保存在内存中是 MySQL 调优的一个重要方面。

### 缓冲池 LRU 算法

缓冲池使用列表进行管理，使用 LRU 算法的变体。当向buffer pool,添加新页面需要空间时，最近最少使用的页面会被逐出，并将新页面添加到列表中间。此中点插入策略将列表视为两个子列表：

- 在头部，最近访问的新（“年轻”）页面 的子列表
- 在尾部，较少访问过的旧页面的子列表

![innodb-buffer-pool-list](images/innodb-buffer-pool-list.png)

该算法将经常使用的页面保留在新的子列表中。旧的子列表包含不太常用的页面；这些页面是[驱逐的](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_eviction)候选页面。

默认情况下，算法操作如下：

- 缓冲池的 3/8 专用于旧子列表。
- 列表的中点是新子列表尾部与旧子列表头部相交的边界。
- 当InnoDB将页面读入缓冲池时，它最初将它插入到中点（旧子列表的头部）。可以读取页面，因为它是用户启动的操作（例如 SQL 查询）所必需的，或者是由 自动执行的[预读](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_read_ahead)操作的一部分InnoDB。
- 访问旧子列表中的页面使其 “年轻”，将其移动到新子列表的头部。如果页面是因为用户启动的操作需要它而被读取，则第一次访问会立即发生，并且页面会变年轻。如果页面是由于预读操作而读取的，则第一次访问不会立即发生，并且在页面被逐出之前可能根本不会发生。
- 随着数据库的运行，缓冲池中未被访问的页面会通过向列表尾部移动来“老化”。新旧子列表中的页面随着其他页面的更新而老化。旧子列表中的页面也会随着页面插入中点而老化。最终，一个未使用的页面到达旧子列表的尾部并被驱逐。

默认情况下，查询读取的页面会立即移动到新的子列表中，这意味着它们在缓冲池中停留的时间更长。例如，[**mysqldump**](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)操作或没有WHERE子句的 SELECT语句 执行的表扫描可以将大量数据带入缓冲池并驱逐等量的旧数据，即使新数据不再使用。类似地，由预读后台线程加载且仅访问一次的页面被移动到新列表的头部。这些情况会将经常使用的页面推送到旧的子列表，在那里它们会被逐出。有关优化此行为的信息，请参阅 [第 14.8.3.3 节，“使缓冲池扫描具有抵抗性”](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-midpoint_insertion.html)和 [第 14.8.3.4 节，“配置 InnoDB 缓冲池预取（预读）”](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-read_ahead.html)。

InnoDB标准监视器输出在`BUFFER POOL AND MEMORY`有关缓冲池 LRU 算法操作的部分中包含多个字段。有关详细信息，请参阅[使用 InnoDB 标准监视器监视缓冲池](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html#innodb-buffer-pool-monitoring)。

### 缓冲池配置

您可以配置缓冲池的各个方面以提高性能

- 理想情况下，您将缓冲池的大小设置为尽可能大的值，从而为服务器上的其他进程留出足够的内存来运行而不会产生过多的分页。**缓冲池越大，就越InnoDB像内存数据库**，从磁盘读取数据一次，然后在后续读取期间从内存访问数据。请参阅 [第 14.8.3.1 节，“配置 InnoDB 缓冲池大小”](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool-resize.html)。
- 在具有足够内存的 64 位系统上，您可以将缓冲池拆分为多个部分，以最大程度地减少并发操作之间对内存结构的争用。有关详细信息，请参阅[第 14.8.3.2 节，“配置多个缓冲池实例”](https://dev.mysql.com/doc/refman/5.7/en/innodb-multiple-buffer-pools.html)。
- 您可以将经常访问的数据保留在内存中，而不管操作的活动突然激增，这些操作会将大量不常访问的数据带入缓冲池。有关详细信息，请参阅 [第 14.8.3.3 节，“使缓冲池扫描抵抗”](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-midpoint_insertion.html)。
- 您可以控制如何以及何时执行预读请求以异步地将页面预取到缓冲池中，以预期很快就会需要这些页面。有关详细信息，请参阅 [第 14.8.3.4 节，“配置 InnoDB 缓冲池预取（预读）”](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-read_ahead.html)。
- 您可以控制何时发生后台刷新以及是否根据工作负载动态调整刷新速率。有关详细信息，请参阅 [第 14.8.3.5 节，“配置缓冲池刷新”](https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool-flushing.html)。
- 您可以配置如何InnoDB保留当前缓冲池状态以避免服务器重新启动后的长时间预热。有关详细信息，请参阅 [第 14.8.3.6 节，“保存和恢复缓冲池状态”](https://dev.mysql.com/doc/refman/5.7/en/innodb-preload-buffer-pool.html)。

### 使用InnoDB标准监视器监视Buffer Pool

```sql
 SHOW ENGINE INNODB STATUS
```

可以查看Buffer Pool的各项指标和内存使用

```bash
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 2198863872
Dictionary memory allocated 776332
Buffer pool size   131072
Free buffers       124908
Database pages     5720
Old database pages 2071
Modified db pages  910
Pending reads 0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 4, not young 0
0.10 youngs/s, 0.00 non-youngs/s
Pages read 197, created 5523, written 5060
0.00 reads/s, 190.89 creates/s, 244.94 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not
0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read
ahead 0.00/s
LRU len: 5720, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
```

>  M数据库T服务器配置：
>
> BufferPool大小：20GB ，主机配置 16c 28G 600G PCIE-SSID
>
> 主从架构，一主二从

## 14.5.2 Change Buffer

Change Buffer是一个特殊的数据结构，**当二级索引页的改变不在buffer pool时，它可以缓存这些页的变更**。这些缓冲区的变更，可能来自于INSERT、UPDATE或DELETE操作（DML）,**当这些页被其他读操作加载到buffer pool，这些变更将被合并**。

![innodb-change-buffer](images/innodb-change-buffer.png)

与聚集索引不同，二级索引通常是非唯一的，而且对二级索引的插入是以相对随机的顺序进行的。同样地，删除和更新可能会影响到在索引树中不相邻的二级索引页。当受影响的页面被其他操作读入缓冲池时，在稍后的时间合并缓存的变化，避免了大量的随机访问I/O，而这是需要从磁盘上将二级索引页面读入缓冲池的。

当系统大部分时间处于空闲状态时，或在缓慢关机期间，定期运行清除操作，将更新的索引页写入磁盘。清除操作可以将**一系列的索引值写入磁盘块，比每个值立即写入磁盘更有效率**。

当有许多受影响的行和许多二级索引需要更新时，变更缓冲区的合并可能需要几个小时。在这段时间内，磁盘I/O会增加，这可能会导致对磁盘绑定的查询的显著减慢。在事务提交后，甚至在服务器关闭和重启后，变更缓冲区的合并也可能继续发生（更多信息请参见章节14.22.2，"强制InnoDB恢复"）。

在内存中，Change Buffer占据了缓冲池的一部分。在磁盘上，Change Buffer是系统表空间的一部分，当数据库服务器关闭时，索引变化被缓冲在这里。

缓存在Change Buffer的数据类型由innodb_change_buffering变量控制。更多信息，请参见配置变化缓冲。你也可以配置最大的变化缓冲区大小。更多信息，请参见配置变更缓冲区的最大尺寸。

如果索引包含一个降序索引列或者主键包含一个降序索引列，则不支持二级索引的变更缓冲。

### 配置Change Buffer
当对表进行INSERT、UPDATE和DELETE操作时，索引列的值（特别是次要键的值）通常是以未排序的顺序进行的，需要大量的I/O来使次要索引达到最新状态。当相关页面不在缓冲池中时，Change Buffer会对二级索引项的变化进行缓存，从而避免了昂贵的I/O操作，因为不会立即从磁盘中读入页面。当页面被加载到缓冲池中时，缓冲区的变化被合并，更新的页面随后被刷新到磁盘。InnoDB主线程在服务器接近空闲时，以及在缓慢关机时合并缓冲变化。

因为它可以导致更少的磁盘读写，所以Change Buffer对I/O绑定的工作负载最有价值；例如，有大量DML操作的应用，如批量插入，都会从Change Buffer中受益。

然而，Change Buffer占据了缓冲池的一部分，减少了可用于缓存数据页的内存。如果工作集几乎适合于缓冲池，或者如果你的表有相对较少的二级索引，禁用变化缓冲可能是有用的。如果工作数据集完全在缓冲池中，Change Buffer不会带来额外的开销，因为它只适用于不在缓冲池中的页面。

innodb_change_buffering变量控制InnoDB执行变化缓冲的程度。你可以为插入、删除操作（当索引记录最初被标记为删除时）和清除操作（当索引记录被物理删除时）启用或禁用缓冲。一个更新操作是插入和删除的组合。默认的innodb_change_buffering值是全部。

# 14.6 InnoDB磁盘结构

## 14.6.2 索引

### 14.6.2.1 聚集索引和二级索引

每个InnoDB表都有一个称为聚集索引的特殊索引，用于存储行数据。通常，聚集索引与主键同义。为了从查询、插入和其他数据库操作中获得最佳性能，了解如何InnoDB使用聚集索引来优化常见查找和 DML 操作非常重要。

- 在表上定义`PRIMARY KEY`a时，InnoDB将其用作聚集索引。应该为每个表定义一个主键。如果没有逻辑唯一且非空的列或列集使用主键，请添加自动增量列。自动递增列值是唯一的，并在插入新行时自动添加。
- 如果没有为表定义`PRIMARY KEY` a ，则InnoDB使用第一个 `UNIQUE`索引，并将所有键列定义为`NOT NULL`聚集索引。
- 如果表没有索引`PRIMARY KEY`或没有合适的 `UNIQUE`索引，则InnoDB 生成以`GEN_CLUST_INDEX`包含行 ID 值的合成列命名的隐藏聚集索引 。行按InnoDB分配的行 ID 排序。行 ID 是一个 6 字节的字段，随着插入新行而单调增加。因此，按行 ID 排序的行在物理上是按插入顺序排列的。

##### 聚集索引如何加快查询速度

通过聚集索引访问一行很快，因为索引搜索直接指向包含行数据的页面。如果表很大，与使用与索引记录不同的页面存储行数据的存储组织相比，聚簇索引体系结构通常可以节省磁盘 I/O 操作。

##### 二级索引与聚集索引的关系

聚集索引以外的索引称为二级索引。在InnoDB中，二级索引中的每条记录都包含该行的主键列，以及指定为二级索引的列。InnoDB使用此主键值去聚集索引中搜索该行。

如果主键很长，二级索引会占用更多的空间，所以主键短是有利的。

有关利用InnoDB 聚集索引和二级索引的指南，请参阅 [第 8.3 节 “优化和索引”](https://dev.mysql.com/doc/refman/5.7/en/optimization-indexes.html)。

### 14.6.2.2 InnoDB 索引的物理结构

除空间索引（spatial indexes）外，InnoDB 索引都是[B 树](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_b_tree)数据结构。空间索引使用 [R 树](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_r_tree)，它是用于索引多维数据的专用数据结构。索引记录存储在其 B 树或 R 树数据结构的叶页中。**索引页的默认大小为 16KB**。页大小由[`innodb_page_size`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_page_size)MySQL 实例初始化时的设置决定 。请参阅 [第 14.8.1 节，“InnoDB 启动配置”](https://dev.mysql.com/doc/refman/5.7/en/innodb-init-startup-configuration.html)。

当新记录插入到InnoDB 聚集索引中时，InnoDB尝试保留 1/16 的页面空闲空间以供将来插入和更新索引记录。如果按顺序（升序或降序）插入索引记录，则生成的索引页大约为 15/16。如果以随机顺序插入记录，则页面从 1/2 到 15/16 满。

InnoDB创建或重建 B 树索引时执行批量加载。这种创建索引的方法称为排序索引构建。该 [`innodb_fill_factor`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_fill_factor)变量定义了在排序索引构建期间填充的每个 B 树页面上的空间百分比，剩余空间保留用于将来的索引增长。空间索引不支持排序索引构建。有关更多信息，请参阅 [第 14.6.2.3 节，“排序索引构建”](https://dev.mysql.com/doc/refman/5.7/en/sorted-index-builds.html)。一个 [`innodb_fill_factor`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_fill_factor)100个叶子在聚簇索引页的空间1/16的设置免费为未来的指数增长。

如果InnoDB索引页面的填充因子低于`MERGE_THRESHOLD`，默认情况下为 50%，如果未指定，则InnoDB尝试收缩索引树以释放页面。该 `MERGE_THRESHOLD`设置适用于 B 树和 R 树索引。有关更多信息，请参阅 [第 14.8.12 节，“配置索引页面的合并阈值”](https://dev.mysql.com/doc/refman/5.7/en/index-page-merge-threshold.html)。

### 14.6.2.3 构建排序索引

InnoDB在创建或重建索引时执行批量加载而不是一次插入一个索引记录。这种索引创建方法也称为排序索引构建。空间索引不支持排序索引构建。

索引构建分为三个阶段。在第一阶段， 扫描[聚集索引](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_clustered_index)，生成索引条目并添加到排序缓冲区。当[排序缓冲区](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_sort_buffer)变满时，条目被排序并写出到临时中间文件。此过程也称为 “运行”。在第二阶段，将一次或多次运行写入临时中间文件，对文件中的所有条目执行归并排序。在第三个也是最后一个阶段，排序后的条目被插入到 [B 树中](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_b_tree)。

在引入排序索引构建之前，使用插入 API 将索引条目一次一条地插入到 B 树中。此方法涉及打开 B 树 [游标](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_cursor)以查找插入位置，然后使用[乐观](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_optimistic)插入将条目插入 B 树页面 。如果由于页面已满而导致插入失败， 则会执行[悲观](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_pessimistic)插入，这涉及打开 B 树游标并根据需要拆分和合并 B 树节点，以便为条目找到空间。这种“自上而下”的弊端 建立索引的方法是搜索插入位置的成本和不断分裂和合并 B 树节点。

排序索引构建使用“自底向上”建立索引的方法。使用这种方法，对最右侧叶页的引用保存在 B 树的所有级别。分配必要 B 树深度的最右侧叶页，并根据其排序顺序插入条目。一旦叶页已满，节点指针将附加到父页，并为下一次插入分配同级叶页。这个过程一直持续到所有条目都被插入，这可能会导致插入到根级别。分配同级页时，释放对先前固定的叶页的引用，新分配的叶页成为最右侧的叶页和新的默认插入位置。

##### 为未来的指数增长保留 B 树页面空间

要为将来的索引增长留出空间，您可以使用该 [`innodb_fill_factor`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_fill_factor)变量来保留 B 树页面空间的百分比。例如，设置 [`innodb_fill_factor`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_fill_factor)为 80 会在排序索引构建期间保留 B 树页面中 20% 的空间。此设置适用于 B 树叶页和非叶页。它不适用于用于[`TEXT`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)或 [`BLOB`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)条目的外部页面 。保留的空间量可能与配置的不完全相同，因为该 [`innodb_fill_factor`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_fill_factor)值被解释为提示而不是硬限制。

##### 排序索引构建和全文索引支持

[全文索引](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_fulltext_index) 支持排序索引构建 。以前，SQL 用于将条目插入到全文索引中。

##### 排序索引构建和压缩表

对于[压缩表](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_compression)，之前的索引创建方法将条目附加到压缩页和未压缩页。当修改日志（代表压缩页面上的可用空间）变满时，将重新压缩压缩页面。如果由于空间不足导致压缩失败，页面将被拆分。使用排序索引构建，条目仅附加到未压缩的页面。当一个未压缩的页面变满时，它就会被压缩。自适应填充用于确保在大多数情况下压缩成功，但如果压缩失败，页面将被拆分并再次尝试压缩。这个过程一直持续到压缩成功。有关 B-Tree 页面压缩的更多信息，请参阅 [第 14.9.1.5 节，“InnoDB 表的压缩如何工作”](https://dev.mysql.com/doc/refman/5.7/en/innodb-compression-internals.html)。

##### 排序索引构建和重做日志

在排序索引构建期间禁用[重做日志记录](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_redo_log)。相反，有一个 [检查点](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_checkpoint)来确保索引构建可以承受意外退出或失败。检查点强制将所有脏页写入磁盘。在排序索引构建期间，[页面清理器](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_page_cleaner)线程会定期收到信号以刷新 [脏页面，](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dirty_page)以确保可以快速处理检查点操作。通常，当干净页面的数量低于设置的阈值时，页面清理器线程会刷新脏页面。对于排序索引构建，脏页会立即刷新以减少检查点开销并并行化 I/O 和 CPU 活动。

##### 排序索引构建和优化器统计信息

排序索引构建可能会导致 [优化器](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_optimizer)统计信息与以前的索引创建方法生成的统计信息不同。统计数据的差异（预计不会影响工作负载性能）是由于用于填充索引的算法不同。

### 14.6.2.4 InnoDB全文索引

全文索引是在基于文本的列（[`CHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html)、 [`VARCHAR`](https://dev.mysql.com/doc/refman/5.7/en/char.html)或[`TEXT`](https://dev.mysql.com/doc/refman/5.7/en/blob.html)列）上创建的， 以加快对这些列中包含的数据的查询和 DML 操作。

全文索引被定义为[`CREATE TABLE`](https://dev.mysql.com/doc/refman/5.7/en/create-table.html)语句的一部分 或使用[`ALTER TABLE`](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html) 或添加到现有表中[`CREATE INDEX`](https://dev.mysql.com/doc/refman/5.7/en/create-index.html)。

全文搜索使用[`MATCH() ... AGAINST`](https://dev.mysql.com/doc/refman/5.7/en/fulltext-search.html#function_match)语法进行。有关使用信息，请参阅 [第 12.10 节，“全文搜索功能”](https://dev.mysql.com/doc/refman/5.7/en/fulltext-search.html)。

##### InnoDB全文索引设计

InnoDB全文索引采用倒排索引设计。倒排索引存储一个单词列表，对于每个单词，还有一个该单词出现的文档列表。为了支持邻近搜索，每个单词的位置信息也被存储为字节偏移量。

## 14.6.3 表空间



## 14.6.5 Doublewrite Buffer双写缓冲区

双写缓冲区是一个存储区域，InnoDB在将页面写到InnoDB数据文件的适当位置之前，**将buffer pool的页面先写入双写缓冲池中**。如果操作系统、存储子系统或意外的mysqld进程在页面写入过程中退出，InnoDB可以在**崩溃恢复期间从双写缓冲区中找到一个良好的页面副本**。

虽然数据被写了两次，但双写缓冲区并不要求两倍的I/O开销或两倍的I/O操作。数据是以一个大的顺序块写入双写缓冲区的，只需向操作系统调用一次fsync()（除了innodb_flush_method被设置为O_DIRECT_NO_FSYNC的情况）。

在大多数情况下，双写缓冲区是默认启用的。要禁用重写缓冲区，将innodb_doublewrite设置为0。

如果系统表空间文件（"ibdata文件"）位于支持原子写入的Fusion-io设备上，则自动禁用双写缓冲，Fusion-io的原子写入用于所有数据文件。因为双写缓冲设置是全局的，所以对于驻留在非Fusion-io硬件上的数据文件，双写缓冲也被禁用。这个功能只在Fusion-io硬件上支持，并且只在Linux上的Fusion-io NVMFS中启用。为了充分利用这个功能，建议将innodb_flush_method设置为O_DIRECT。

## 14.6.6 Redo日志

redo日志是一种基于磁盘的数据结构，**用于在崩溃恢复期间修复不完整事务写入的数据**。在正常操作期间，redo日志对由 SQL 语句或低级 API 调用产生的更改表数据的请求进行编码。在初始化期间和接受连接之前，会自动重播在意外关闭之前未完成更新数据文件的修改。有关redo日志在崩溃恢复中的作用的信息，请参阅 [第 14.19.2 节，“InnoDB 恢复”](https://dev.mysql.com/doc/refman/5.7/en/innodb-recovery.html)。

默认情况下，redo日志在磁盘上对应着两个名为`ib_logfile0`和`ib_logfile1`的物理文件。MySQL以**循环方式写入redo日志文件**。redo日志中的数据根据受影响的记录进行编码；这些数据统称为redo。通过redo日志的数据通道由不断增加的[LSN](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_lsn)值表示。

#### redo日志刷新的组提交

InnoDB像任何其他 [符合 ACID](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_acid)的数据库引擎一样，**在事务提交之前刷新事务的[redo日志](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_redo_log)**。InnoDB 使用[group commit](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_group_commit) 功能将多个刷新请求组合在一起，以避免每次提交都要刷新。使用组提交，InnoDB向日志文件发出一次写入，对应大约同时提交的多个用户的事务执行提交操作，从而显著提高吞吐量。

## 14.6.7 undo日志

undo日志是与单个读写事务相关联的撤消日志记录的集合。undo日志记录包含有关如何撤消事务对[聚集索引](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_clustered_index) 记录的最新更改的信息。如果另一个事务需要查看原始数据用于一致性读取操作的一部分，则将从undo Logs记录中检索未修改的数据。undo Log存在于 [撤消日志段中](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_undo_log_segment)，而[撤消日志段](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_undo_log_segment)包含在  [rollback segments](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_rollback_segment).。回滚段驻留在 [系统表空间](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_system_tablespace)、[undo表空间](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_undo_tablespace)和[临时表空间中](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_temporary_tablespace)。

驻留在临时表空间中的撤消日志用于修改用户定义临时表中数据的事务。这些撤消日志不会被redo日志记录，因为它们不是崩溃恢复需要的。它们仅用于在服务器运行时的回滚。这种类型的撤消日志通过避免重做日志记录 I/O 来提高性能。

InnoDB最多支持 128 个回滚段，其中 32 个分配给临时表空间。这留下了 96 个回滚段，可以分配给修改常规表中数据的事务。该 [`innodb_rollback_segments`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_rollback_segments)变量定义了 使用的回滚段数目InnoDB。

回滚段支持的事务数量取决于回滚段中的undo slot数量和每个事务所需的undo log数量。回滚段中的撤消槽数因InnoDB页面大小而异。

| InnoDB Page Size | Number of Undo Slots in a Rollback Segment (InnoDB Page Size / 16) |
| :--------------- | :----------------------------------------------------------- |
| `4096 (4KB)`     | `256`                                                        |
| `8192 (8KB)`     | `512`                                                        |
| `16384 (16KB)`   | `1024`                                                       |
| `32768 (32KB)`   | `2048`                                                       |
| `65536 (64KB)`   | `4096`                                                       |

一个事务最多分配四个撤消日志，一个用于以下每种操作类型：

1. [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html) 对用户定义表的操作
2. UPDATE以及 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)对用户定义表的操作
3. [`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html) 对用户定义的临时表的操作
4. UPDATE以及 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)对用户定义的临时表的操作

根据需要分配撤消日志。例如，对常规表和临时表执行[`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)、 UPDATE和 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)操作的事务需要完整分配四个撤消日志。仅对[`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)常规表执行操作的事务 需要单个撤消日志。

对常规表执行操作的事务从分配的系统表空间或撤消表空间回滚段分配撤消日志。对临时表执行操作的事务从分配的临时表空间回滚段分配撤消日志。

分配给事务的撤消日志在其持续时间内保持附加到事务。例如，为[`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html) 常规表上的操作分配给事务的撤消日志用于该事务[`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)在常规表上执行的所有 操作。

考虑到上述因素，可以使用以下公式来估计InnoDB能够支持的并发读写事务的数量：

# 14.7 InnoDB lock和事务模型

要实现大规模、繁忙或高度可靠的数据库应用程序，从不同的数据库系统移植大量代码，或调整 MySQL 性能，了解InnoDB锁和InnoDB 事务模型非常重要 。

## 14.7.1 InnoDB的锁

本节描述了InnoDB使用的锁类型

#### 共享锁和排他锁Shared and Exclusive Locks

InnoDB实现标准的行级锁，其中有两种类型的锁， [共享 ( `S`) 锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_shared_lock)和[排它 ( `X`) 锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_exclusive_lock)。

- [共享（`S`）锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_shared_lock)允许事务对读取的行持有锁。
- [独占（`X`）锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_exclusive_lock)允许事务在更新或删除行时持有锁。

如果事务T1在持有对行`r`的共享 ( `S`) 锁，那么来自某个不同事务`T2` 的对行`r`的锁请求将按如下方式处理：

- T2事务请求 `S`锁立即被授予。也就是`T1`与`T2` 持有行r的s锁。
- T2事务请求一个 `X`锁不能立即授予。

如果事务`T1`持有行r的排他(X)锁，则事务T2对该行r的任一类型锁的请求都不会被授予。相反，事务`T2`必须等待事务`T1`释放其对行r的锁。

#### 意向锁Intention Locks

InnoDB支持多粒度锁，允许行锁和表锁共存。例如，诸如LOCK TABLES ... WRITE这样的语句在指定的表上取得了一个独占锁（X锁）。为了使多个粒度级别的锁切实可行，InnoDB使用意图锁。意图锁是表级别的锁，它表明事务以后需要对表中的某一行进行哪种类型的锁（共享或独占）。有两种类型的意向锁。

- 意向共享锁（IS）表示事务打算对表中的个别行设置共享锁。

- 意向排他锁（IX）表示一个事务打算对表中的个别行设置排他锁。

例如，`SELECT ... LOCK IN SHARE MODE`设置一个IS锁，`SELECT ... FOR UPDATE`设置一个IX锁。

意向锁协议如下：

- 在一个事务可以获得表内某行的共享锁之前，它必须首先获得该表的IS锁或更强的锁。

- 在事务对表中的某一行取得独占锁之前，它必须首先取得该表的IX锁。

> 那可以理解为，想要获取共享锁，得先获取一个意向共享锁或排它锁；获取排他锁，就先获取意向排它锁。也就是要先表达出意向？

表级锁类型的兼容性总结在下面的矩阵中：

|      | `X`      | `IX`       | `S`        | `IS`       |
| :--- | :------- | :--------- | :--------- | :--------- |
| `X`  | Conflict | Conflict   | Conflict   | Conflict   |
| `IX` | Conflict | Compatible | Conflict   | Compatible |
| `S`  | Conflict | Conflict   | Compatible | Compatible |
| `IS` | Conflict | Compatible | Compatible | Compatible |

如果一个锁与现有的锁兼容，则授予请求事务；但如果与现有的锁冲突，则不授予，事务会等待，直到冲突的现有锁被释放。如果一个锁请求与现有的锁冲突，并且不能被授予，因为它会导致死锁，那么就会发生一个错误。

除了全表请求（例如，LOCK TABLES ... WRITE），意图锁不会阻止任何东西。意图锁的主要目的是显示有人正在锁定某一行，或将要锁定表中的某一行。

#### 记录锁

记录锁是对索引记录的锁。例如， `SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;` 可以防止从插入，更新或删除行，其中的值的任何其它交易`t.c1`是 `10`。

记录锁总是锁定索引记录，即使一个表没有定义索引。对于这种情况，InnoDB创建一个隐藏的聚集索引并使用该索引进行记录锁定。请参阅 [第 14.6.2.1 节，“聚集索引和二级索引”](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)。

用于在记录锁定事务数据出现类似于在以下[`SHOW ENGINE INNODB STATUS`](https://dev.mysql.com/doc/refman/5.7/en/show-engine.html)和 [InnoDB的监视器](https://dev.mysql.com/doc/refman/5.7/en/innodb-standard-monitor.html) 输出：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

#### 间隙锁

间隙锁是对索引记录之间的间隙的锁，或者是对第一个索引记录之前或最后一个索引记录之后的间隙的锁。例如，`SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE;`阻止其他事务将值`15`插入到列`t.c1`，无论列中是否已经存在任何此类值，因为该范围内所有现有值之间的间隙被锁定。

间隙可能跨越单个索引值、多个索引值，甚至是空的。

间隙锁是性能和并发性之间权衡的一部分，用于某些事务隔离级别而不是其他级别。

使用唯一索引锁定行搜索唯一行的语句不需要间隙锁。（这不包括搜索条件仅包含多列唯一索引的某些列的情况；在这种情况下，确实会发生间隙锁定。）。例如，如果该`id`列具有唯一索引，则以下语句仅使用`id`值为 100的行的索引记录锁定，其他会话是否在前面的间隙中插入行无关紧要：

```sql
SELECT * FROM child WHERE id = 100;
```

如果`id`未编入索引或具有非唯一索引，则该语句会锁定前面的间隙。

这里还值得注意的是，不同的事务可以在间隙上持有冲突的锁。例如，事务 A 可以在间隙上持有共享间隙锁（间隙 S 锁），而事务 B 在同一间隙上持有排他间隙锁（间隙 X 锁）。允许冲突间隙锁的原因是，如果从索引中清除记录，则必须合并不同事务在记录上持有的间隙锁。

间隙锁定InnoDB是“纯粹的抑制性”，这意味着它们的唯一目的是防止其他事务插入间隙。间隙锁可以共存。一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。共享和排他间隙锁之间没有区别。它们彼此不冲突，并且执行相同的功能。

可以明确禁用间隙锁定。如果您将事务隔离级别更改为[`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)或启用 [`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog) 系统变量（现已弃用），则会发生这种情况 。在这种情况下，对搜索和索引扫描禁用间隙锁定，仅用于外键约束检查和重复键检查。

使用 [`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)隔离级别或启用 [`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog). 在 MySQL 评估`WHERE`条件后释放不匹配行的记录锁。对于 `UPDATE`报表，InnoDB 做一个“半一致”读，这样它返回的最新提交版本到MySQL使MySQL能够确定该行是否比赛`WHERE` 的条件UPDATE。

#### Next-Key 锁

next-key 锁是索引记录上的记录锁和间隙锁的组合。

InnoDB执行行级锁定的方式是，当它搜索或扫描表索引时，它会在遇到的索引记录上设置共享锁或排他锁。因此，行级锁实际上是索引记录锁。索引记录上的 next-key 锁也会影响该索引记录之前的“间隙”。也就是说，next-key 锁是一个索引记录锁加上一个在索引记录间隙上的间隙锁。如果一个会话对`R`索引中的记录具有共享锁或排他锁 ，则另一个会话不能`R`在索引顺序中紧接在前的间隙中插入新的索引记录 。

假设一个索引包含值 10、11、13 和 20。该索引可能的 next-key 锁涵盖以下区间，其中圆括号表示排除区间端点，方括号表示包含端点：

```none
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```

对于最后一个时间间隔，next-key lock 锁定索引中最大值以上的间隙，并且“ supremum ” 伪记录的值高于索引中的任何实际值。supremum 不是真正的索引记录，因此，实际上，这个 next-key 锁只锁定最大索引值之后的间隙。

默认情况下，InnoDB在 [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)事务隔离级别运行。在这种情况下，**InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读**（请参阅[第 14.7.4 节，“幻像行”](https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html)）。

用于下一个键锁定事务数据出现类似于在下面[`SHOW ENGINE INNODB STATUS`](https://dev.mysql.com/doc/refman/5.7/en/show-engine.html)和 [InnoDB的监视器](https://dev.mysql.com/doc/refman/5.7/en/innodb-standard-monitor.html) 输出：

```sql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t`
trx id 10080 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

#### 插入意向锁Insert Intention Locks

插入意向锁是间隙锁的一种，由[`INSERT`](https://dev.mysql.com/doc/refman/5.7/en/insert.html)操作在插入行之前设置。这种锁发出了插入意图的信号，如果多个事务在同一索引间隙中插入的位置不一样，就不需要互相等待。假设有数值为4和7的索引记录。分别试图插入值为5和6的事务，在获得插入行的独占锁之前，各自用插入意图锁锁定了4和7之间的间隙，但是由于这些行是不冲突的，所以不会互相阻塞。

下面的例子演示了一个事务在获得被插入记录的独占锁之前，采取插入意向锁。这个例子涉及两个客户，A和B。

客户端A创建了一个包含两个索引记录（90和102）的表，然后启动一个事务，在ID大于100的索引记录上加一个独占锁。该独占锁包括102号记录前的间隙锁。：

```sql
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+
```

客户端 B 开始一个事务以在间隙中插入一条记录。事务在等待获得排他锁时使用插入意向锁。(注：此时肯定是插入不进去，要等待的)

```sql
mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);
```

用于插入意图锁定事务数据出现类似于在下面 [`SHOW ENGINE INNODB STATUS`](https://dev.mysql.com/doc/refman/5.7/en/show-engine.html)和 [InnoDB的监视器](https://dev.mysql.com/doc/refman/5.7/en/innodb-standard-monitor.html) 输出：

```sql
RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 80000066; asc    f;;
 1: len 6; hex 000000002215; asc     " ;;
 2: len 7; hex 9000000172011c; asc     r  ;;...
```

#### AUTO-INC 锁

`AUTO-INC`锁是一个特殊的表级锁，当事务插入到具有AUTO_INCREMENT列的表中。在最简单的情况下，如果一个事务正在向表中插入值，则任何其他事务都必须等待自己插入操作，以便第一个事务插入的行接收连续的主键值。

该[`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) 变量控制用于自动增量锁定的算法。它允许您选择如何在可预测的自动增量值序列和插入操作的最大并发之间进行权衡。

有关更多信息，请参阅 [第 14.6.1.6 节，“InnoDB 中的 AUTO_INCREMENT 处理”](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html)。

#### 空间索引的谓词锁

InnoDB支持`SPATIAL` 对包含空间数据的列进行索引（请参阅 [第 11.4.8 节，“优化空间分析”](https://dev.mysql.com/doc/refman/5.7/en/optimizing-spatial-analysis.html)）。

要把手锁止涉及操作 `SPATIAL`索引，下一键锁定不能很好地支持工作[`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)或 [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_serializable)事务隔离级别。多维数据中没有绝对排序的概念，所以不清楚哪个是 “ next ”键。

要为带`SPATIAL`索引的表启用隔离级别支持 ，请InnoDB 使用谓词锁。甲`SPATIAL`索引包含最小外接矩形（MBR）值，因此，InnoDB通过设置用于查询的MBR值的谓词锁强制上的索引一致的读取。其他事务无法插入或修改与查询条件匹配的行。

## 14.7.2 InnoDB的事务模型

InnoDB事务模型旨在将多版本数据库的最佳特性与传统的两阶段锁定相结合。InnoDB在行级别上执行锁，并且默认以非锁的一致读取方式运行查询，与Oracle的风格一致。InnoDB中的锁信息是以空间效率的方式存储的，因此不需要锁升级。通常情况下，允许几个用户锁定InnoDB表中的每一行，或者任何随机的行的子集，而不会导致InnoDB内存耗尽。

### 14.7.2.1 事务隔离级别

事务隔离是数据库处理的基础之一。隔离是缩写[ACID 中](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_acid)的 I ；隔离级别是在多个事务同时进行更改和执行查询时微调结果的性能和可靠性、一致性和可再现性之间的平衡的设置。

InnoDB提供了由SQL:1992标准描述的所有四个事务隔离级别： [`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted)， [`READ COMMITTED`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_read-committed)， [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)，和 [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_serializable)。InnoDB默认隔离级别是 [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)。

用户可以使用[`SET TRANSACTION`](https://dev.mysql.com/doc/refman/5.7/en/set-transaction.html)语句更改单个会话或所有后续连接的隔离级别。要为所有连接设置服务器的默认隔离级别，请使用 [`--transaction-isolation`](https://dev.mysql.com/doc/refman/5.7/en/server-options.html#option_mysqld_transaction-isolation)命令行或选项文件中的选项。有关隔离级别和级别设置语法的详细信息，请参阅 [第 13.3.6 节，“SET TRANSACTION 语句”](https://dev.mysql.com/doc/refman/5.7/en/set-transaction.html)。

InnoDB使用不同的[锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking)策略支持每个事务隔离级别 。你可以用默认的REPEATABLE READ级别强制执行高度的一致性，用于对关键数据的操作，在这种情况下，ACID符合性非常重要。或者你可以用READ COMMITTED甚至READ UNCOMMITTED来放松一致性规则，在诸如批量报告的情况下，精确的一致性和可重复的结果不如最小化锁的开销来的重要。SERIALIZABLE执行比REPEATABLE READ更严格的规则，主要用于特殊情况，如XA事务，以及排除并发和死锁的问题。

下面的列表描述了MySQL如何支持不同的事务级别。该列表从最常用的级别到最不常用的级别排列:

- `REPEATABLE READ`

  这是InnoDB默认的隔离级别。 同一事务内的[一致性读取](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_consistent_read)读取由第一次读取建立的 [快照](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_snapshot)。这意味着，如果您SELECT在同一事务中发出多个普通（非锁定）语句，则这些 [`SELECT`](https://dev.mysql.com/doc/refman/5.7/en/select.html)语句彼此之间也是一致的。请参阅 [第 14.7.2.3 节，“一致的非锁定读取”](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)。

  对于[锁定读取](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read) （ `SELECT with FOR UPDATE `或`LOCK IN SHARE MODE`）、 UPDATE和 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)语句，锁定取决于语句是使用具有唯一搜索条件的唯一索引还是范围类型搜索条件。

  - 对于具有唯一搜索条件的唯一索引，InnoDB只锁定找到的索引记录，而不锁定它之前的[间隙](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_gap)。
  - 对于其他搜索条件，InnoDB 锁定扫描的索引范围，使用 [间隙锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_gap_lock)或 [next-key 锁](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_next_key_lock) 阻止其他会话插入范围所覆盖的间隙。

- `READ COMMITTED`

  每个一致读取，即使在同一个事务中，也会设置和读取自己的新快照。

  对于锁定读取（SELECTwith`FOR UPDATE`或`LOCK IN SHARE MODE`）、UPDATE 语句和[`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html) 语句，InnoDB仅锁定索引记录，而不锁定它们之间的间隙，因此允许在锁定记录旁边自由插入新记录。间隙锁仅用于外键约束检查和重复键检查。

  由于间隙锁被禁用，可能会出现幻读问题，因为其他会话可以将新行插入间隙中。

  `READ COMMITTED`隔离级别 仅支持基于行的二进制日志记录 。如果使用`READ COMMITTED`with [`binlog_format=MIXED`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format)，服务器会自动使用基于行的日志记录。

  使用`READ COMMITTED`有额外的效果：

  - 对于 UPDATE或者 [`DELETE`](https://dev.mysql.com/doc/refman/5.7/en/delete.html)语句，InnoDB只为它更新或删除的行持有锁。在 MySQL 评估`WHERE`条件后释放不匹配行的记录锁 。这大大降低了死锁的可能性，但它们仍然可能发生。
  - 对于UPDATE语句，如果行已被锁定，InnoDB 执行一个“半一致”读，返回最新提交的版本到MySQL，以便MySQL能够确定该行是否匹配 `WHERE`的条件 UPDATE。如果行匹配（必须更新），MySQL 再次读取该行，这次InnoDB要么锁定它，要么等待锁定它。

  考虑以下示例，从该表开始：

  ```sql
  CREATE TABLE t (a INT NOT NULL, b INT) ENGINE = InnoDB;
  INSERT INTO t VALUES (1,2),(2,3),(3,2),(4,3),(5,2);
  COMMIT;
  ```

  在这种情况下，表没有索引，因此搜索和索引扫描使用隐藏的聚集索引进行记录锁定（而不是索引列。

  假设一个会话使用这些语句执行一个UPDATE ：

  ```sql
  # Session A
  START TRANSACTION;
  UPDATE t SET b = 5 WHERE b = 3;
  ```

  还假设第二个会话在第一个会话之后执行以下语句来执行UPDATE：

  ```sql
  # Session B
  UPDATE t SET b = 4 WHERE b = 2;
  ```

  在[InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)执行每个UPDATE，它首先为它读取的每一行获取一个排他锁，然后决定是否修改它。如果 [InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)不修改行，则释放锁。否则， [InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)保留锁直到事务结束。这会影响事务处理如下。

  使用默认`REPEATABLE READ` 隔离级别时，第一个UPDATE在它读取的每一行上获取一个 x 锁，并且不释放任何行：

  ```none
  x-lock(1,2); retain x-lock
  x-lock(2,3); update(2,3) to (2,5); retain x-lock
  x-lock(3,2); retain x-lock
  x-lock(4,3); update(4,3) to (4,5); retain x-lock
  x-lock(5,2); retain x-lock
  ```

  第二个UPDATE在尝试获取任何锁时立即阻塞（因为第一次更新已保留所有行上的锁），并且在第一次UPDATE提交或回滚之前不会继续：

  ```none
  x-lock(1,2); block and wait for first UPDATE to commit or roll back
  ```

  如果改为使用`READ COMMITTED`，则第一个UPDATE在它读取的每一行上获取一个 x 锁，并释放它不修改的那些行：

  ```none
  x-lock(1,2); unlock(1,2)
  x-lock(2,3); update(2,3) to (2,5); retain x-lock
  x-lock(3,2); unlock(3,2)
  x-lock(4,3); update(4,3) to (4,5); retain x-lock
  x-lock(5,2); unlock(5,2)
  ```

  对于第二个`UPDATE`，InnoDB执行 “半一致性”读取，将读取的每一行的最新提交版本返回给 MySQL，以便 MySQL 可以确定该行是否符合 UPDATE的 `WHERE`条件：

  ```none
  x-lock(1,2); update(1,2) to (1,4); retain x-lock
  x-lock(2,3); unlock(2,3)
  x-lock(3,2); update(3,2) to (3,4); retain x-lock
  x-lock(4,3); unlock(4,3)
  x-lock(5,2); update(5,2) to (5,4); retain x-lock
  ```

  但是，如果`WHERE`条件包含索引列并InnoDB使用索引，则在获取和保留记录锁时仅考虑索引列。在以下示例中，第 一个UPDATE在 b = 2 的每一行上获取并保留一个 x 锁。第二 个UPDATE在尝试获取相同记录上的 x 锁时会阻塞，因为它也使用在 b 列上定义的索引。

  ```sql
  CREATE TABLE t (a INT NOT NULL, b INT, c INT, INDEX (b)) ENGINE = InnoDB;
  INSERT INTO t VALUES (1,2,3),(2,2,4);
  COMMIT;
  
  # Session A
  START TRANSACTION;
  UPDATE t SET b = 3 WHERE b = 2 AND c = 3;
  
  # Session B
  UPDATE t SET b = 4 WHERE b = 2 AND c = 4;
  ```

  使用`READ COMMITTED` 隔离级别的效果与启用不推荐使用的[`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog) 变量相同 ，但有以下例外：

  - 启用 [`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog) 是全局设置并影响所有会话，而隔离级别可以为所有会话全局设置，也可以为每个会话单独设置。
  - [`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog) 只能在服务器启动时设置，而隔离级别可以在启动时设置或在运行时更改。

  `READ COMMITTED`因此提供比 更精细、更灵活的控制 [`innodb_locks_unsafe_for_binlog`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_locks_unsafe_for_binlog)。

- `READ UNCOMMITTED`

  SELECT语句是以非锁定的方式执行的，但是可能会使用某行的早期版本。因此，使用这种隔离级别，这种读取是不一致的。这也被称为脏读。否则，这种隔离级别的工作方式与READ COMMITTED类似。COMMITTED`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_read-committed).

- `SERIALIZABLE`

  这个级别和REPEATABLE READ一样，但是如果自动提交被禁用，InnoDB会隐含地将所有普通的SELECT语句转换为SELECT ... LOCK IN SHARE MODE。如果自动提交被启用，SELECT是它自己的事务。因此，它被认为是只读的，如果作为一个一致的（非锁定的）读来执行，可以被序列化，并且不需要为其他事务阻塞。(如果其他事务已经修改了所选的行，要强制一个普通的SELECT阻塞，请禁用自动提交）。)

### 14.7.2.2 自动提交，提交和回滚

在InnoDB中，所有的用户活动都发生在一个事务中。如果启用了自动提交模式，每个SQL语句就会形成一个单独的事务。默认情况下，MySQL为每个启用了自动提交功能的新连接启动会话，因此如果每个SQL语句没有返回错误，MySQL会在该语句之后进行提交。如果一个语句返回一个错误，提交或回滚行为取决于错误。

启用了自动提交功能的会话可以通过明确的START TRANSACTION或BEGIN语句开始，并用COMMIT或ROLLBACK语句结束来执行一个多语句事务。

如果在会话中用SET autocommit = 0禁用了自动提交模式，那么该会话总是有一个事务在打开。COMMIT或ROLLBACK语句会结束当前的事务并开始一个新的事务。

如果一个禁用了自动提交的会话在没有明确提交最终事务的情况下结束，MySQL将回滚该事务。

一些语句隐含地结束一个事务，就像你在执行语句前做了COMMIT一样。详情见 [Section 13.3.3, “Statements That Cause an Implicit Commit”](https://dev.mysql.com/doc/refman/5.7/en/implicit-commit.html).

COMMIT意味着在当前事务中所做的改变是永久性的，并且对其他会话是可见的。另一方面，ROLLBACK语句则取消了当前事务所做的所有修改。COMMIT和ROLLBACK都会释放所有在当前事务中设置的InnoDB锁。



默认情况下，与MySQL服务器的连接是在启用自动提交模式的情况下开始的，它在你执行每条SQL语句时自动提交。如果你有其他数据库系统的经验，可能会对这种操作模式感到陌生，因为其他数据库系统的标准做法是发出一连串的DML语句并提交它们或将它们全部回滚。

要使用多语句事务，请用SQL语句SET autocommit = 0关闭自动提交，并根据情况用COMMIT或ROLLBACK结束每个事务。要保持自动提交状态，用START TRANSACTION开始每个事务，用COMMIT或ROLLBACK结束。下面的例子显示了两个事务。第一个被提交，第二个被回滚。

```sql
mysql> CREATE TABLE customer (a INT, b CHAR (20), INDEX (a));
Query OK, 0 rows affected (0.00 sec)
mysql> -- Do a transaction with autocommit turned on.
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)
mysql> INSERT INTO customer VALUES (10, 'Heikki');
Query OK, 1 row affected (0.00 sec)
mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
mysql> -- Do another transaction with autocommit turned off.
mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)
mysql> INSERT INTO customer VALUES (15, 'John');
Query OK, 1 row affected (0.00 sec)
mysql> INSERT INTO customer VALUES (20, 'Paul');
Query OK, 1 row affected (0.00 sec)
mysql> DELETE FROM customer WHERE b = 'Heikki';
Query OK, 1 row affected (0.00 sec)
mysql> -- Now we undo those last 2 inserts and the delete.
mysql> ROLLBACK;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM customer;
+------+--------+
| a    | b      |
+------+--------+
|   10 | Heikki |
+------+--------+
1 row in set (0.00 sec)
mysql>
```

### 14.7.2.3 一致非锁定读

一致性读取是指InnoDB使用多版本来查询呈现数据库在某个时间点的快照。查询看到的是在该时间点之前提交的事务所做的改变，而没有看到后来或未提交的事务所做的改变。这个规则的例外是，查询可以看到同一事务中早期语句所做的改变。这个例外会导致以下异常：如果你更新了表中的一些行，SELECT会看到最新版本的更新行，但是它也可能看到任何行的旧版本。如果其他会话同时更新同一个表，该异常现象意味着你可能看到该表处于数据库中从未存在的状态。

如果事务隔离级别是REPEATABLE READ（默认级别），同一事务中的所有一致读取都会读取该事务中第一个此类读取所建立的快照。你可以通过提交当前事务并在之后发布新的查询来为你的查询获得一个更新鲜的快照。

在READ COMMITTED隔离级别下，事务中的每个一致读取都会设置并读取自己的新快照。

**一致性读取是InnoDB处理READ COMMITTED和REPEATABLE READ隔离级别中SELECT语句的默认模式。一致性读取不会在其访问的表上设置任何锁，因此在表上执行一致性读取的同时，其他会话可以自由修改这些表。**

假设你正运行在默认的REPEATABLE READ隔离级别中。当你发出一致读取（也就是一个普通的SELECT语句）时，InnoDB给你的事务一个时间点，你的查询会根据这个时间点来查看数据库。如果另一个事务删除了一条记录，并在你的时间点被分配后提交，你不会看到该记录被删除。插入和更新也是类似的处理方式。

> 注意
> 数据库状态的快照适用于事务中的SELECT语句，不一定适用于DML语句。如果你插入或修改了一些行，然后提交了该事务，那么从另一个并发的REPEATABLE READ事务发出的DELETE或UPDATE语句可能会影响那些刚刚提交的行，即使会话不能查询它们。如果一个事务确实更新或删除了由不同事务提交的行，这些变化对当前的事务是可见的。例如，你可能会遇到下面这种情况:
>
> ```sql
> SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
> -- Returns 0: no rows match.
> DELETE FROM t1 WHERE c1 = 'xyz';
> -- Deletes several rows recently committed by other transaction.
> 
> SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
> -- Returns 0: no rows match.
> UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
> -- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
> SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
> -- Returns 10: this txn can now see the rows it just updated.
> ```



你可以通过提交你的事务，然后做另一个SELECT或START TRANSACTION WITH CONSISTENT SNAPSHOT来推进你的时间点。

这被称为多版本的并发控制MVCC。

在下面的例子中，会话A只有在B提交了插入，A也提交了的情况下才能看到B插入的行，这样时间点就提前到了B的提交之后。

```sql
             Session A              Session B

           SET autocommit=0;      SET autocommit=0;
time
|          SELECT * FROM t;
|          empty set
|                                 INSERT INTO t VALUES (1, 2);
|
v          SELECT * FROM t;
           empty set
                                  COMMIT;

           SELECT * FROM t;
           empty set

           COMMIT;

           SELECT * FROM t;
           ---------------------
           |    1    |    2    |
           ---------------------
```

如果你想看到数据库的 "最新 "状态，可以使用READ COMMITTED隔离级别或锁定读取

```sql
SELECT * FROM t LOCK IN SHARE MODE;
```

在READ COMMITTED隔离级别下，事务中的每个一致读取都会设置并读取自己的新快照。在LOCK IN SHARE模式下，会发生一个锁定的读取。一个SELECT会阻塞，直到包含最新鲜记录的事务结束（参见第14.7.2.4节，"锁定读取"）。

- 一致性读取在某些DDL语句上不起作用。

- 一致性读取在DROP TABLE上不起作用，因为MySQL不能使用一个已经被丢弃的表，InnoDB会销毁该表。

一致性读取在ALTER TABLE操作上不起作用，ALTER TABLE操作是对原始表进行临时拷贝，并在临时拷贝建立后删除原始表。当你在一个事务中重新发出一致读取时，新表中的行是不可见的，因为当事务的快照被拍摄时，这些行并不存在。在这种情况下，该事务会返回一个错误。ER_TABLE_DEF_CHANGED, "表定义已经改变，请重试事务"。

对于INSERT INTO ...这样的语句中的选择，读取的类型是不同的。选择，更新... (SELECT)，和CREATE TABLE ... SELECT，这些子句没有指定FOR UPDATE或LOCK IN SHARE MODE。

- 默认情况下，InnoDB在这些语句中使用更强的锁，并且SELECT部分的行为就像READ COMMITTED，其中每个一致的读取，甚至在同一事务中，设置和读取自己的新快照。

- 要在这种情况下执行无锁读取，请启用innodb_locks_unsafe_for_binlog选项，并将事务的隔离级别设置为READ UNCOMMITTED、READ COMMITTED或REPEATABLE READ，以避免在从选定表读取的行上设置锁。

### 14.7.2.4 锁定读取

如果你查询数据，然后在同一个事务中插入或更新相关的数据，常规的SELECT语句并不能提供足够的保护。其他事务可以更新或删除你刚刚查询的同一行。InnoDB支持两种类型的锁读，提供额外的安全。

- [`SELECT ... LOCK IN SHARE MODE`](https://dev.mysql.com/doc/refman/5.7/en/select.html)

  在任何被读取的行上设置一个共享模式的锁。其他会话可以读取这些行，但是在你的事务提交之前不能修改它们。如果这些行被另一个尚未提交的事务所改变，你的查询会等待，直到该事务结束，然后使用最新的值。

- [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/5.7/en/select.html)

  对于搜索遇到的索引记录，锁住这些行和任何相关的索引条目，就像你对这些行执行了UPDATE语句一样。阻止其他事务更新这些行，阻止做SELECT ... LOCK IN SHARE MODE，或者阻止在某些事务隔离级别中读取数据。一致性读取会忽略在读取视图中存在的记录上设置的任何锁。(记录的旧版本不能被锁定；它们是通过在记录的内存副本上应用撤销日志来重构的。)


在处理树状结构或图状结构的数据时，这些条款主要是有用的，无论是在一个表中还是在多个表中分割。你从一个地方穿越边缘或树枝到另一个地方，同时保留回来改变任何这些 "指针 "值的权利。

所有由LOCK IN SHARE MODE和FOR UPDATE查询设置的锁在事务提交或回滚时被释放。

> 注意
> 只有在禁止自动提交的情况下才可以锁定读取（通过使用START TRANSACTION开始事务或将自动提交设置为0。

在外层语句中的锁定读取子句不会锁定嵌套子查询中的表的行，除非在子查询中也指定了锁定读取子句。例如，下面这个语句不会锁定表t2中的记录。

```sql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2) FOR UPDATE;
```

要锁定表t2中的记录，需要在子查询中添加一个锁定的读取子句。

```sql
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2 FOR UPDATE) FOR UPDATE;
```

#### 锁定读取的例子

假设你想在表child中插入一条新的记录，并确保该子记录在表parent中有一个父记录。你的应用程序代码可以在这一系列的操作中确保参考完整性。

首先，使用一个一致的读法来查询表PARENT，并验证父行是否存在。你能安全地插入子行到表CHILD中吗？不能，因为其他会话可能在你的SELECT和INSERT之间删除父行，而你并不知道。

为了避免这个潜在的问题，使用LOCK IN SHARE MODE来执行SELECT。

```sql
SELECT * FROM parent WHERE NAME = 'Jones' LOCK IN SHARE MODE;
```

在LOCK IN SHARE MODE查询返回父类 "Jones "后，你可以安全地将子类记录添加到CHILD表中并提交事务。任何试图在PARENT表的适用行中获取独占锁的事务都会等待，直到你完成，也就是说，直到所有表中的数据都处于一致状态。

另一个例子，考虑表CHILD_CODES中的一个整数计数器字段，用来给表CHILD中添加的每个孩子分配一个唯一的标识符。不要使用一致读取或共享模式读取计数器的当前值，因为数据库的两个用户可以看到计数器的相同值，如果两个事务试图向CHILD表添加具有相同标识符的行，会发生重复键错误。

在这里，LOCK IN SHARE MODE不是一个好的解决方案，因为如果两个用户同时读取计数器，当它试图更新计数器时，至少有一个用户会陷入死锁。

为了实现读取和增加计数器，首先使用FOR UPDATE对计数器进行锁定读取，然后再增加计数器。比如说

```sql
SELECT counter_field FROM child_codes FOR UPDATE;
UPDATE child_codes SET counter_field = counter_field + 1;
```

一个SELECT ... FOR UPDATE读取最新的可用数据，在它读取的每一行上设置独占锁。因此，它设置的锁与搜索的SQL UPDATE在行上设置的锁相同。

前面的描述只是一个关于SELECT ... FOR UPDATE如何工作的例子。在MySQL中，生成唯一标识符的具体任务实际上只需使用对表的一次访问就可以完成了。

```sql
UPDATE child_codes SET counter_field = LAST_INSERT_ID(counter_field + 1);
select last_insert_id();
```

SELECT语句只是检索标识符信息（特定于当前连接）。它并没有访问任何表。

> 注：从上面的例子来看，适用于以下情况？
>
> lock in share mode适用于两张表存在业务关系时的一致性要求，for  update适用于操作同一张表时的一致性要求。

### 14.7.3 InnoDB在不同SQL语句上设置的不同锁

锁定读取、UPDATE或者DELETE通常会对在处理SQL语句中被扫描的每一条索引记录设置记录锁。语句中是否有排除该行的WHERE条件并不重要。InnoDB不记得确切的WHERE条件，而只知道哪些索引范围被扫描了。这些锁通常是next-key 锁，也会阻止插入到紧邻记录的 "间隙 "中。然而，间隙锁可以明确地被禁用，这将导致不使用next-key 锁。

如果在搜索中使用了二级索引，并且要设置的索引记录锁是独占的，那么InnoDB也会检索相应的聚类索引记录并对它们设置锁。

如果你**没有适合你的语句的索引，并且MySQL必须扫描整个表来处理该语句，那么表的每一行都会被锁定，这反过来又会阻止其他用户对表的所有插入**。创建良好的索引是很重要的，这样你的查询就不会扫描超过需要的行。

InnoDB设置的特定类型的锁如下：

- SELECT ... FROM是一个一致的读取，读取数据库的一个快照，并且不设置锁，除非事务隔离级别被设置为SERIALIZABLE。对于SERIALIZABLE级别，搜索在它遇到的索引记录上设置共享的next-key 锁。然而，对于使用唯一索引锁定行的语句，只需要一个索引记录锁来搜索唯一行。

- 对于SELECT ... FOR UPDATE或者SELECT ... LOCK IN SHARE MODE，会获取要扫描的行的锁，并且期望会释放不符合列入结果集行的锁（例如，如果它们不符合WHERE子句中给出的标准）。然而，在某些情况下，行可能不会被立即解锁，因为在查询执行过程中，结果行和其原始来源之间的关系会丢失。例如，在UNION中，从表中扫描（和锁定）的行可能会在评估它们是否符合结果集的条件之前被插入到一个临时表中。在这种情况下，临时表中的记录与原始表中的记录的关系就会丢失，并且后者的记录直到查询执行结束时才会被解锁。

- SELECT ... LOCK IN SHARE MODE在搜索遇到的所有索引记录上设置共享的next-key 锁。然而，对于使用唯一索引锁定记录的语句，只需要一个索引记录锁来搜索唯一的记录。

- SELECT ... FOR UPDATE在搜索遇到的每一条记录上设置一个独占的next-key锁。然而，对于使用唯一索引锁定记录以搜索唯一记录的语句，只需要一个索引记录锁。

  对于搜索遇到的索引记录，SELECT ... FOR UPDATE会阻止其他会话进行SELECT ... LOCK IN SHARE MODE或在某些事务隔离级别中的读取。一致性读取会忽略在读取视图中存在的记录上设置的任何锁。

- UPDATE ... WHERE ... 在搜索遇到的每条记录上设置一个独占的next-key锁。然而，对于使用唯一索引锁定记录以搜索唯一记录的语句，只需要一个索引记录锁。

- 当UPDATE修改一个聚类索引记录时，会在受影响的二级索引记录上采取隐式锁。在插入新的二级索引记录之前执行重复检查扫描时，以及插入新的二级索引记录时，UPDATE操作也会在受影响的二级索引记录上取得共享锁。

- DELETE FROM ... WHERE ... 在搜索遇到的每条记录上设置一个独占的next-key锁。然而，对于使用唯一索引锁定记录的语句，只需要一个索引记录锁来搜索唯一记录。

- INSERT在插入的行上设置一个独占锁。这个锁是一个索引记录锁，而不是next-key锁（也就是说，没有间隙锁），并且不阻止其他会话在插入的行之前插入到间隙中。

  在插入行之前，会设置一种叫做插入意图间隙锁的间隙锁。这个锁发出了插入意图的信号，如果多个事务在同一索引间隙中插入的位置不一样，就不需要互相等待。假设有数值为4和7的索引记录。试图插入值为5和6的独立事务，在获得插入行的独占锁之前，各自用插入意图锁锁定了4和7之间的间隙，但是由于这些行是不冲突的，所以不会互相阻塞。

  如果发生重复键错误，就会在重复的索引记录上设置一个共享锁。如果有多个会话试图插入同一行，而另一个会话已经有了独占锁，那么这种共享锁的使用可能会导致死锁。

  ```sql
  CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;
  ```

  现在假设有三个事务按以下顺序执行

  ```sql
  # session1:
  START TRANSACTION;
  INSERT INTO t1 VALUES(1);
  
  # session2
  START TRANSACTION;
  INSERT INTO t1 VALUES(1);
  
  # session3
  START TRANSACTION;
  INSERT INTO t1 VALUES(1);
  
  # session1
  ROLLBACK;
  ```

  会话1的第一个操作获得了该行的独占锁。会话2和3的操作都导致了一个重复键的错误，他们都请求一个行的共享锁。当会话1回滚时，它释放了对该行的独占锁，会话2和3的排队共享锁请求被批准。在这一点上，会话2和3陷入僵局。由于另一个会话持有的共享锁，这两个会话都不能获得该行的独占锁。

  如果表已经包含一条键值为1的行，并且三个会话依次执行以下操作，也会出现类似的情况:

  ```sql
  # Session 1:
  START TRANSACTION;
  DELETE FROM t1 WHERE i = 1;
  
  # Session 2:
  START TRANSACTION;
  INSERT INTO t1 VALUES(1);
  
  # Session 3:
  START TRANSACTION;
  INSERT INTO t1 VALUES(1);
  
  # Session 1:
  COMMIT;
  ```

  会话1的第一个操作获得了该行的独占锁。会话2和3的操作都导致了一个重复键的错误，他们都请求一个行的共享锁。当会话1提交时，它释放了对该行的独占锁，会话2和3的排队共享锁请求被批准。在这一点上，会话2和3陷入僵局。由于另一个会话持有共享锁，两个会话都不能获得该行的独占锁。

- INSERT ... ON DUPLICATE KEY UPDATE与简单的INSERT不同，当发生重复键错误时，在要更新的行上放置一个独占锁而不是一个共享锁。对于一个重复的主键值，会有一个独占的索引记录锁。对于重复的唯一键值，会使用独占的next-key锁。

  如果在唯一键上没有碰撞，REPLACE会像INSERT一样进行。否则，一个排他性的next-key锁将被放在要替换的行上。

- INSERT INTO T SELECT ... FROM S WHERE ... 在插入到T中的每一条记录上设置一个独占索引记录锁（没有间隙锁）。如果事务隔离级别是READ COMMITTED，或者innodb_locks_unsafe_for_binlog被启用并且事务隔离级别不是SERIALIZABLE，InnoDB在S上进行搜索，作为一个一致读取（没有锁）。否则，InnoDB会在S的记录上设置共享的下一个键锁，在后一种情况下，InnoDB必须设置锁。在使用基于语句的二进制日志进行向前滚动恢复的过程中，每个SQL语句都必须以完全相同的方式执行。

  创建表... SELECT ... 使用共享的下一个键锁来执行SELECT，或者像INSERT ... SELECT一样，作为一个一致的读。

  当SELECT被用于构造REPLACE INTO t SELECT ... FROM s WHERE ... 或者UPDATE t ... WHERE col IN (SELECT ... FROM s ...)，InnoDB对s表的记录设置共享的下一个键锁。


- InnoDB在初始化表的一个先前指定的AUTO_INCREMENT列时，在与AUTO_INCREMENT列相关的索引的末端设置了一个独占锁。

  在innodb_autoinc_lock_mode=0的情况下，InnoDB使用一种特殊的AUTO-INC表锁模式，在访问自动增量计数器时，该锁被获得并保持到当前SQL语句的结束（而不是到整个事务的结束）。当AUTO-INC表锁被保持时，其他客户不能插入到该表。同样的行为发生在innodb_autoinc_lock_mode=1的 "批量插入 "中。在innodb_autoinc_lock_mode=2的情况下，表级的AUTO-INC锁不被使用。更多信息，请参见章节14.6.1.6, "InnoDB的AUTO_INCREMENT处理"。

  InnoDB获取先前初始化的AUTO_INCREMENT列的值而不设置任何锁。

- 如果在表上定义了一个FOREIGN KEY约束，任何需要检查约束条件的插入、更新或删除都会在检查约束的记录上设置共享记录级锁。InnoDB也会在约束条件失败的情况下设置这些锁。

- LOCK TABLES设置表锁，但设置这些锁的是InnoDB层以上的MySQL层。如果innodb_table_locks = 1（默认）和autocommit = 0，InnoDB就知道表锁，InnoDB上面的MySQL层知道行级锁。

  否则，InnoDB的自动死锁检测无法检测到涉及这种表锁的死锁。另外，因为在这种情况下，较高的MySQL层不知道行级锁，所以有可能在另一个会话当前拥有行级锁的表上获得一个表锁。然而，这并不危及事务的完整性，正如第14.7.5.2节 "死锁检测 "中所讨论的。

  如果innodb_table_locks=1（默认），LOCK TABLES在每个表上获得两个锁。除了在MySQL层上的表锁外，它还获取一个InnoDB表锁。要避免获取InnoDB表锁，请设置innodb_table_locks=0。 如果没有获取InnoDB表锁，即使表的一些记录被其他事务锁定，LOCK TABLES也会完成。

  在MySQL 5.7中，innodb_table_locks=0对用LOCK TABLES ...明确锁定的表没有影响。WRITE锁定的表没有影响。它对被LOCK TABLES ... WRITE锁定的读或写的表确实有影响。WRITE（例如，通过触发器）或通过LOCK TABLES ... 阅读。

- 所有由事务持有的InnoDB锁在事务提交或中止时被释放。因此，在自动提交=1模式下对InnoDB表调用LOCK TABLES没有多大意义，因为获得的InnoDB表锁将被立即释放。

- 你不能在事务中间锁定其他的表，因为LOCK TABLES会执行隐含的COMMIT和UNLOCK TABLES。

## 14.7.4 幻读

所谓的幻读问题就是，在一个事务中中同一个查询在不同的时间产生不同的记录集。例如，如果一个SELECT被执行了两次，但是第二次返回了一条第一次没有返回的记录，那么这条记录就是一个 "幻读 "记录。

假设在子表的id列上有一个索引，你想从表中读取并锁定所有标识符值大于100的记录，并打算以后更新所选记录中的某些列。

```sql
SELECT * FROM child WHERE id > 100 FOR UPDATE;
```

这个查询从id大于100的第一条记录开始扫描索引。该表包含id值为90和102的记录。如果在扫描范围内的索引记录上设置的锁没有锁定在空白处（在这种情况下，90和102之间的空白处）进行的插入，另一个会话可以向表中插入一条id为101的新行。如果你在同一个事务中执行同样的SELECT，你会在查询返回的结果集中看到一条id为101的新行（一个 "幻读"）。如果我们把一组行看作是一个数据项，那么新的幻读子就会违反事务的隔离原则，即一个事务应该能够运行，使它所读取的数据在事务中不发生变化。

为了防止幻读，InnoDB使用了一种叫做next-key的算法，它将索引-行锁和间隙锁结合起来。InnoDB执行行级锁的方式是，当它搜索或扫描一个表的索引时，它对它遇到的索引记录设置共享或排他锁。因此，行级锁实际上是索引-记录锁。此外，一个索引记录上的next-key 锁也会影响到索引记录之前的 "间隙"。也就是说，next-key 锁是一个索引记录锁加上索引记录之间的空隙锁。如果一个会话对索引中的记录R有一个共享或独占的锁，另一个会话就不能在索引顺序中紧挨着R的空隙中插入一个新的索引记录。

当InnoDB扫描一个索引时，它也可以锁定索引中最后一条记录之后的空隙。就在前面的例子中发生了这种情况。为了防止任何插入表的id大于100，InnoDB设置的锁包括对id值102之后的空隙的锁。

你可以在你的应用程序中使用next-key 锁来实现唯一性检查。如果你在共享模式下读取你的数据，并且没有看到你要插入的行的重复，那么你可以安全的插入你的行，并且知道在读取过程中对你的行的继承者设置的next-key 锁可以防止任何人同时插入重复的行。因此，next-key 锁使你能够 "锁定 "表中不存在的东西。

正如第14.7.1节 "InnoDB锁定 "中所讨论的，间隙锁可以被禁用。这可能会导致幽灵般的问题，因为当间隙锁被禁用时，其他会话可以在间隙中插入新的行。