


# SQL优化

你都做过哪些SQL优化？调优知道吗？怎么处理慢SQL？等等

- 业务层面

  比如可以加限制条数和限制时间范围：查老师最近登录的记录，只允许你查最近一个月的，再多就不允许了。

- **SQL层面**

  SQL优化和索引优化

- **数据库架构**

  分库分表：

  读写分离：课程报告表数据量很多，做了读写分离。我是想引入shardingJDBC，但是和leader沟通后说时间紧有风险，目前也是只实现读写不同的库，所以自己通过注解实现了一套，每个SQL上走不同的数据库。

- 数据库服务器层面

  服务器端也可以做些调优，调整buffer poll ，磁盘IO等参数，一般由公司的运维/DBA负责。

## 慢SQL怎么处理？

> 注意要从三个步骤来考虑，先是是否加载多余数据，再考虑是否走索引，最后考虑分表等

开启慢查询日志，通过skywalking或者数据库的一些监控软件来监控慢SQl，

- 分析SQL是否加载了多余的数据，是否需要改写SQL缩小查询范围；

  insert时批量插入数据，避免网络IO

- 查看索引的使用情况，分析SQL的查询计划，修改SQl或者增加索引，避免全表扫描。

  可以建立适当的联合索引和覆盖索引，索引如果不能包含null值就提前声明not null

- 最后如果实在表数据量大，是否要读写分离，分库分表。

## SQL优化案例

[zm优化案例](https://mp.weixin.qq.com/s/GpRX1uEqUz4eRAQpEU3-eg)

- 不要轻易分组排序，排序操作极耗资源

- 不要使用 select * 写法，只查询需要的列

- 通过 limit 方式分页会导致后续分页越来越慢，可取前一次分页的最大 ID 作为下一页参数输入，进行分页。

  分页去除总页数，只显示上一页下一页。

  ```sql
  -- 虽然 limit 100 和limit 1000000,100 的解释计划相同，但是执行时间差异非常大。
  -- limit m,n 中的 m 数越大，则查询越慢。limit 100 的执行时间 0.05s；limit 1000000,100 的执行时间 35s
  select students.id,students.user_id,students.stu_city
   from students
   join students_seller on students.id=students_seller.student_id
   where students.created_at>='2021-01-01'
    and students_seller.state=0
    limit 100;
    
  -- 例如假设第 10000 页取到的 100 行数据中最大的 students.ID 为 54978976，现在需要取得第 10001 页的 100 行数据
  -- 那么可以添加 students.id>54978976,然后取 100 行数据，此方法取得的即为第 10001 页的数据。
  -- 通过 id 字段的范围限制，比简单的 limit m,n 更加高效，即使是大的数据分页也不会导致效率变低。该方法执行时间稳定为 0.05s
  select students.id,students.user_id,students.stu_city
   from students
   join students_seller on students.id=students_seller.student_id
   where students.created_at>='2021-01-01'
    and students_seller.state=0  
    and students.id>54978976 -- 取上一次分页的最大ID
    limit 100;
  ```

- **使用union/union all 替换 where 子句中的 or 连接条件**

  ```sql
  -- 需求：查询18年后和有对应销售人员的学生
  -- 1.因为条件里有 >, >=，所以使用or不走索引，扫描全表。
  select count(0)
   from students
   where students.created_at >= '2021-05-01'
   or students.referrer_user_id > 0;
   
   -- 2.改成union，会走索引，而且会走覆盖索引using index，效率很高
   select count(0) from 
   (select id
   from students
   where students.created_at>='2021-05-01'
   union
   select id
   from students
   where students.referrer_user_id>0) as t;
  ```

- 使join关联查询替换子查询，in，exists写法

  在 Mysql 的写法中，非常不建议子查询写法。子查询是一个独立的查询，不能参与到驱动表的数据过滤，而且子查询的结果数据会被放到临时表中存放，然后与驱动表进行关联，效率非常低。

  ```sql
  -- 说明：payments 表使用单独的子查询，type 为 ALL 需要扫描整个 payments 表记录，将返回的结果存放在临时表，然后与 students 表进行匹配。查询耗时 22s
  select sum(t.money)
  from students
  join 
  (select payments.stu_id,payments.money
  from payments
  where payments.is_paid=1
    and payments.is_canceled=0
    and payments.money>0) as t 
    on students.id=t.stu_id
      where students.created_at>='2021-05-01'
      and students.created_at< '2021-06-01';
  
  -- 说明：不使用子查询，改为 join 写法后。查询首先根据 students.created_at 走索引扫描，然后根据查询的结果与 payments 表的 stu_id 进行关联。查询耗时 1.85s
  select sum(payments.money)
  from students
  join payments on students.id=payments.stu_id
  where payments.is_paid=1
    and payments.is_canceled=0
      and payments.money>0
      and students.created_at>='2021-05-01'
      and students.created_at< '2021-06-01';
  ```

  **join还可以用来替代not in**

  ```sql
  -- 查询是统计 users 表创建时间大于 '2021-05-20 12:00:00'，且不在 users_account_number 表中的用户 ID
  -- 请注意 id=2 的 select_typ 类型为 DEPENDENT SUBQUERY，表示外层 select 结果需要依赖于子查询的结果。效率会非常差
  select u.id 
   from users u 
   where u.updated_at > '2021-05-20 12:00:00'
   and u.id not in(select a.user_id from users_account_number a);
   
   -- 直接用 join/left join 的写法替代，效率将会有大幅提升
  select u.id 
   from users u 
   left join users_account_number a on u.id=a.user_id
   where u.updated_at > '2021-05-20 12:00:00'
   and a.user_id is null;
  ```

-    索引字段类型错误或函数运算导致索引失效

```sql
-- users 表的 mobile 字段是 varchar 类型，并且创建有索引，但是输入参数是整形，导致查询无法走索引扫描，type 为 ALL 全表扫描
select *
 from users
 where mobile=13999999999;
 
-- mobile 为 varchar 类型，则在参数上添加引号''标识为字符类型 直接走唯一索引
select *
 from users
 where mobile='13999999999';
```

created_at字段函数运算导致索引失效

```sql
-- 由于在 students.created_at 字段上使用 date 函数，导致无法通过 created_at 列匹配满足时间要求的数据，通过 rows 列可以看到是全表扫描 5285w 数据量。
# 或许有人会疑问既然是全表扫描，为什么 type 是 index 而不是 ALL。这是因为 students 表只用到 created_at 和 id 列，这两列是直接包含在索引中的，查询是走的覆盖索引。
select count(0)
 from students
 join students_seller on students.id=students_seller.student_id
 where date(students.created_at)='2021-05-05'
  and students_seller.state=0;
  
-- 由于是查询注册时间为 2021-05-05 日的学生，那么可以替代为使用 created_at>= ... and <= 的写法
-- 优化后查询根据 students.created_at 走索引范围查询，匹配行数 rows 为 134092 条，然后与 students_seller 表关联，效率非常高。
select count(0)
 from students
 join students_seller on students.id=students_seller.student_id
 where students.created_at>='2021-05-05 00:00:00'
   and students.created_at<='2021-05-05 23:59:59'
  and students_seller.state=0;
```

- where + order by + limit 查询陷阱

  在程序设计中，经常需要用到 where+order by+limit 写法获取满足条件的、排序后的 N 条数据结果。这种写法本身并没有问题，但是在实际使用中，这条语句却常常引起严重的性能问题，需要我们重点关注。

  ```sql
  # 查询是想取得学生注册时间在 2021-04-05 的，并且 state=0 的，按照更新时间取得最近 100 条学生记录。
  # 请注意解释计划中的 key 为 updated_at 字段，我们明明是对 created_at 字段做范围限制，为啥 MySQL 选择走 updated_at 列索引？
  select students.id,students.user_id,students_seller.seller_id,students.stu_city
   from students
   join students_seller on students.id=students_seller.student_id
   where students.created_at>='2021-04-05'
    and students.created_at<='2021-04-05 23:59:59'
    and students_seller.state=0
   order by students.updated_at desc 
   limit 100;
  ```

  当查询语句中包含 where+order by+limit 时

  由于 MySQL 优化器会认为排序 order by 是非常耗时的操作，如果能在一开始就将结果做排序返回，那是最好的。而且 limit 100 会让优化器认为这 100 条记录是很容易满足的，此时优化器会走如下执行计划：

    1. 由于 students.updated_at 字段本身就有索引（已排序），按照 updated_at 字段每次取 100 条结果，然后走 where 匹配，将满足的结果返回

    2. 重复以上操作，直到有 100 条结果满足要求，循环结束

    3. 如果查询按照 students.updated_at 排序的数据经过 where 条件过滤后能快速满足 100 条结果输出，那么查询或许会非常快

    4. 但如果一致没有取到满足 where 条件的 100 条结果，会一直循环操作直至遍历 students 整个表！那这个代价是非常恐怖的

  可以通过构建子查询的方式，强制先导执行

  ```sql
  select t.*
   from
   (select students.id, students.user_id, students_seller.seller_id,s tudents.stu_city, students.updated_at
   from students
   join students_seller on students.id=students_seller.student_id
   where students.created_at>='2021-04-05'
    and students.created_at<='2021-04-05 23:59:59'
    and students_seller.state=0) as t
  order by t.updated_at desc 
  limit 100;
  ```

# 数据库规范

## 建表规范

- 数据库表字符集统一设置为 utf8mb4 ，排序规则为 utf8mb4_general_ci

  当字符集或者排序规则不一致时，会导致表无法关联查询；如果进行字符转换，会导致索引失效，而且也会额外消耗数据库性能；统一的字符集和排序规则设置，能减少不必要的字符集问题

- 必须包含 deleted ，create_time ，update_time 数据列（zm业务规范而已）

  逻辑删除，create_time ，update_time可以交由数据库自动维护；方便进行增量数据同步（BI部门）

- 尽量不要使用blog和text等大字段

  数据库在存储大字段的数据时，数据列上只存储指向外部存储的指针，查询时需要通过指针找到外部存储，然后再读取大字段的内容，增加了额外的磁盘IO操作，效率低下！

  课程报告表有个字段（知识点内容），长度很大，单独拆分出来。

- 字段类型

  定长字符串尽量使用 char，不要使用 varchar（注意 char 最大定义为 255 ）；

  字段尽量设置为非空（ not null ），并设置 default 属性；

  字段长度满足业务需求下尽可能短，

  > 常见字段类型的长度（以下字段长度均为不为空时的字段长度）
  >
  > - tinyint 1 字节；smallint 2 字节；int 4 字节；bigint 8 字节；
  >
  > - date 3 字节；datetime 8 字节；timestamp 4 字节；
  >
  > 字符类型括号中定义的数字为字符数，即 varchar(32) 可以存放 32 个数字或者汉字
  >
  > 如果字段为 varchar 变长类型，当定义小于 255 字节时，需要 1 字节存放字段长度；当定义超过 255 字节时，需要 2 字节存放字段长度
  >
  > 字符类型长度跟字符集有关（其中 utf8 占用 3 字节，utf8mb4 占用 4 字节）
  >
  >   varchar(10) 类型（utf8mb4）字节数： 10*4B+1B=41B
  >
  >   varchar(100) 类型（utf8mb4）字节数：100*4B+2B=402B
  >
  >   char(10) 类型（utf8mb4）字节数：10*4B=40B

# 数据库常见问题

## 如何平滑的给表添加字段？

如果数据量很大，会锁表。影响线上业务

- 在凌晨业务不繁忙的时候添加字段，不影响线上业务。
- 提前预留字段，可以预留很多个
- 新建一张表包含新字段的，然后DTS迁移过去，然后查新表。把老表改成别的名字，新表改成正确的表名。
- 提前设计，使用key/value方法存储到redis里，优雅！

# 问题案例

## 1. 大量同时更新表导致连接池不够用

```properties
GlobalExceptionHandler(Exception)  error:nested exception is org.apache.ibatis.exceptions.PersistenceException:  ### Error  querying database. Cause:  org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain  JDBC Connection; nested exception is  java.sql.SQLTransientConnectionException: hikari - Connection is not  available, request timed out after 10000ms.  ### The error  may exist in URL  
```

**排查过程**：

线上短时间大量报错，看日志是数据库连接池不够了。

然后去查看公司的慢SQL日志记录，发现有个SQL是慢SQL，而且时间是不断累加的，也就是开始1秒，然后3秒，4秒。

分析这个SQL很简单就是update操作，条件是where device_id ='cccc'。虽然表记录只有2万多行，但是这个条件列没有建立索引，查看执行计划，是ALL全表扫描。当业务同时发生大批量更新，因为没有索引导致了锁全表，很多SQL占用了连接，慢慢的连接池不够用了

**解决方法**：

就是条件字段建立索引。

**扩展**：

即便有二级索引，当MySQL预估扫描行数超过全表总数约 20% ～ 30% 时，也会直接升级为全表扫描。这种情况一般不考虑，因为实际的查询行数一般不会超过全表的30%。

Each table index is queried, and the best index is used unless the optimizer believes that it is more efficient to use a table scan. At one time, a scan was used based on whether the best index spanned more than 30% of the table, but a fixed percentage no longer determines the choice between using an index or a scan. The optimizer now is more complex and bases its estimate on additional factors such as table size, number of rows, and I/O block size.

[https://blog.csdn.net/n88Lpo/article/details/78099094](https://blog.csdn.net/n88Lpo/article/details/78099094)写的不错，有实践证明

## 2. 优化器选择了错误的索引

背景：名片的申请订单表有几十万条数据，建有（shop_id，order_token，status)的索引，也有time的索引。根据这些条件来查询时，mysql优化器非要走shop_id的索引（区分度不高，大概十几个不同的值），但经测试（force index）发现走time索引肯定更快。

做法：因为shop_id索引区分度不高，且命名也不规范，于是删除之。再测试就走了正确的time索引。

原理：

优化器会根据**扫描行数**，还有**是否使用临时表**，**是否排序**，是否要回表等因素综合判断。

优化器并不知道精确的记录条数，只能根据统计信息进行估算，根据的是索引的区分度。一个索引上不同的值越多，区分度就越好。一个索引上不同的值个数，称为基数，基数越大，索引区分度越好。`show index from t` 可以查看索引的基数。

优化器是采用采样统计的方式来得到索引的基数，如选10个数据页，统计这些页上的平均值，乘以索引的页数，得到这个索引的基数。

很明显，这个统计信息很可能不准，导致优化器估算的扫描行数也会不准，从而导致选择错误的索引。我们可以用analyze table t来重新统计索引信息，


索引选择异常的处理

1. 采用force index强行选择一个索引。但是这种方式不够便捷，也不优美
2. 修改SQL。比如子查询，
3. 新建合适的索引，甚至删除错误的索引


# 8.8 理解查询执行计划

根据你的表、列、索引的细节以及你的WHERE子句中的条件，MySQL优化器考虑了许多技术来有效地执行SQL查询中涉及的查找。对一个巨大的表的查询可以在不读取所有行的情况下进行；涉及几个表的连接可以在不比较每个行的组合的情况下进行。优化器为执行最有效的查询而选择的一组操作被称为 "查询执行计划query execution plan"，也被称为EXPLAIN计划。你的目标是识别EXPLAIN计划中表明查询被很好优化的方面，并学习SQL语法和索引技术，以便在看到一些低效操作时改进计划。

## 8.8.1使用EXPLAIN优化查询

EXPLAIN语句提供关于MySQL如何执行语句的信息：

- EXPLAIN与SELECT、DELETE、INSERT、REPLACE和UPDATE语句一起工作。

- 当EXPLAIN与可解释语句一起使用时，MySQL显示来自优化器的关于语句执行计划的信息。也就是说，MySQL解释它将如何处理该语句，包括关于如何连接表和以何种顺序连接的信息。关于使用EXPLAIN获得执行计划信息的信息，见第8.8.2节 "EXPLAIN输出格式"。

- 当EXPLAIN与FOR CONNECTION connection_id一起使用，而不是与可解释的语句一起使用时，会显示在指定连接中执行的语句的执行计划。参见章节8.8.4, "获取命名连接的执行计划信息"。

- 对于SELECT语句，EXPLAIN产生额外的执行计划信息，可以使用SHOW WARNINGS来显示。参见章节8.8.3, "扩展的EXPLAIN输出格式"。

- EXPLAIN对于检查涉及分区表的查询非常有用。参见章节22.3.5, "获取关于分区的信息"。

- FORMAT选项可以用来选择输出格式。TRADITIONAL是以表格格式输出的。如果没有FORMAT选项，这是默认的。JSON格式以JSON格式显示信息。

在EXPLAIN的帮助下，你可以看到你应该在哪里给表添加索引，以便通过使用索引来查找行，使语句执行得更快。你还可以使用EXPLAIN来检查优化器是否以最佳顺序连接表。为了提示优化器使用与表在SELECT语句中的命名顺序相对应的连接顺序，可以用SELECT STRAIGHT_JOIN开始语句，而不是仅仅用SELECT。(参见章节13.2.9, "SELECT语句"。)然而，STRAIGHT_JOIN可能会阻止索引的使用，因为它禁用了半连接的转换。参见章节8.2.2.1, "用半连接转换优化子查询、派生表和视图引用"。

优化器跟踪有时可以提供与EXPLAIN互补的信息。然而，优化器跟踪的格式和内容在不同的版本中会有变化。详情请见MySQL内部。追踪优化器。

如果你有一个问题，即当你认为应该使用索引时却没有使用，那么运行ANALYZE TABLE来更新表的统计数据，如键的cardinality，这可以影响优化器的选择。参见第13.7.2.1节，"ANALYZE TABLE语句"。

注意
EXPLAIN也可以用来获得关于表中列的信息。EXPLAIN tbl_name与DESCRIBE tbl_name和SHOW COLUMNS FROM tbl_name同义。更多信息请参见章节13.8.1, "DESCRIBE语句 "和章节13.7.5.5, "SHOW COLUMNS语句"。

## 8.8.2 EXPLAIN输出格式

EXPLAIN语句提供关于MySQL如何执行语句的信息。EXPLAIN与SELECT、DELETE、INSERT、REPLACE和UPDATE语句一起工作。

EXPLAIN为SELECT语句中使用的每个表返回一行信息。它在输出中按照MySQL在处理语句时读取它们的顺序列出表。MySQL使用嵌套循环连接方法解决所有连接。这意味着MySQL从第一个表中读取一条记录，然后在第二个表、第三个表中找到匹配的记录，以此类推。当所有的表都被处理后，MySQL输出所选择的列，并在表列表中回溯，直到找到一个有更多匹配行的表。下一条记录将从这个表中读取，然后继续处理下一个表。

EXPLAIN输出包括分区信息。另外，对于SELECT语句，EXPLAIN产生的扩展信息可以在EXPLAIN之后用SHOW WARNINGS来显示（参见章节8.8.3, "扩展的EXPLAIN输出格式"）。

>  注意
> 在较早的MySQL版本中，分区和扩展信息是使用EXPLAIN PARTITIONS和EXPLAIN EXTENDED产生的。为了向后兼容，这些语法仍然被认可，但是分区和扩展输出现在是默认启用的，所以PARTITIONS和EXTENDED关键字是多余的，并且被废弃。使用它们会导致一个警告；预计它们会在未来的MySQL版本中从EXPLAIN语法中删除。
>
> 你不能在同一个EXPLAIN语句中同时使用已废弃的PARTITIONS和EXTENDED关键字。此外，这些关键字都不能与FORMAT选项一起使用。
>
> 注意
> MySQL Workbench有一个可视化解释功能，提供EXPLAIN输出的可视化表示。见教程。使用解释来提高查询性能。

### EXPLAIN 输出列

本节描述了由EXPLAIN产生的输出列。后面的章节提供了关于type和Extra列的其他信息。

EXPLAIN的每一条输出行都提供了关于一个表的信息。每一行都包含了表8.1 "EXPLAIN输出列 "中总结的数值，并且在表的后面有更详细的描述。列名显示在表的第一列中；第二列提供了使用FORMAT=JSON时输出中显示的等效属性名称。

**Table 8.1 EXPLAIN Output Columns**

| Column                                                       | JSON Name       | Meaning                                        |
| :----------------------------------------------------------- | :-------------- | :--------------------------------------------- |
| [`id`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_id) | `select_id`     | The `SELECT` identifier                        |
| [`select_type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_select_type) | None            | The `SELECT` type                              |
| [`table`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_table) | `table_name`    | The table for the output row                   |
| [`partitions`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_partitions) | `partitions`    | The matching partitions                        |
| [`type`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_type) | `access_type`   | The join type                                  |
| [`possible_keys`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_possible_keys) | `possible_keys` | The possible indexes to choose                 |
| [`key`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key) | `key`           | The index actually chosen                      |
| [`key_len`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_key_len) | `key_length`    | The length of the chosen key                   |
| [`ref`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_ref) | `ref`           | The columns compared to the index              |
| [`rows`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_rows) | `rows`          | 扫描行数的预估                                 |
| [`filtered`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_filtered) | `filtered`      | Percentage of rows filtered by table condition |
| [`Extra`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain_extra) | None            | Additional information                         |

- `Extra` (JSON name: none)

  这一列包含关于MySQL如何解决查询的额外信息。查看 [`EXPLAIN` Extra Information](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-extra-information).

  没有与`Extra`列相对应的单一JSON属性；但是，在这一列中可能出现的值被暴露为JSON属性，或作为`message`属性的文本。


## 表设计性能原则

- 数据类型：比如能用smallInt就不要用int。能用整数类型的就不要用char类型，因为他们占用的存储空间是不一样的

- 使用非空约束。不仅避免业务上非空的判断，也容易创建索引。甚至节省存储空间。

- 合理增加冗余字段，减少关联查询

- 适当拆分表。把过长的字段和不常用的字段拆分成单独的表