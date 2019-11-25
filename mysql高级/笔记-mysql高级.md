[toc]

# 索引
## 描述一下索引的含义
> 索引是高效查询并获取数据的数据结构。索引是数据结构（B树、B+树、Hash表）。

## 索引的优缺点是什么
1. 优点
1.1 提高查询效率（降低IO使用率）。
1.2. 降低CPU使用率（order by age desc，B+树本身会对索引列数据进行排序）。

2. 缺点 
2.1 索引占用内存和磁盘空间。
2.2 会降低增删改的效率，因为数据的增删改也需要同部索引的增删改。

## 索引的分类
1. 单值索引 ：单列，一个表可以都多单值索引（age、name都可作为索引）。
2. 唯一索引：用作索引的列不能包含重复内容。
3. 复合索引：多个列构成的索引（相当于二级目录）。
4. 主键索引：主键索引和唯一索引类似，但是主键索引的索引列不能为null。

**索引操作示例**
```sql
-- 创建索引方式一
create 索引类型 索引名称 on 表名(字段名);

--单值索引
create index index_age on student(age);
--唯一索引
create unique index_student_id on student(id);
--复合索引
create index index_age_name on student(age,name);

-- 创建索引方式二
alter table 表名 add 索引类型 索引名称(索引字段);

-- 删除索引
drop index 索引名 on 表名;

-- 查看一张表的所有索引
show index from 表名;
show index from user;
```

**注意**
> 如果一个字段被primary key修饰，则该字段默认就是主键索引。

# 查询优化

## explain

**执行样例**
```sql
mysql> explain select tc.tcdesc from teacherCard tc where tc.tcid = 
    -> ( select t.tcid from teacher t where t.tid=(
    ->      select c.tid from course c where c.cname = 'sql'
    ->   ) 
    -> );
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | tc    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | t     | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  3 | SUBQUERY    | c     | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  |    4 |    25.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
```

**explain需要注意的地方**
1. id值不同，id值越大越优先查询；id值相同，从上往下顺序执行。
2. select_type
2.1 PRIMARY：包含子查询中的主查询（最外层）
2.2 SUBQUERY：包含子查询中的子查询（非最外层）
2.3 SIMPLE：简单查询
2.4 derived：衍生查询（使用到了临时表）
a) 在from子查询中只有一张表
b) 在from子查询中，如果有```table1 union table2```，则table1是derived，table2就是union。
3. type 查询类型
	3.1 system（忽略），只有一条数据的系统表，或者衍生表只有一条数据的主查询。
 	3.2 const，仅仅能查询到一条数据的查询且只能用于primary key或union索引。
	3.3 eq_ref，唯一性索引，对于每个索引列的查询，返回匹配唯一行数据（有且只有一个）。
	3.4 ref，非唯一性索引，对于每个所以列的查询，返回匹配的所有行（0个或多个）。
	3.5 range，检索指定范围的行。
	3.6 index，查询全部索引的数据。
	3.7 all，查询表全部的数据。
4. possible_keys：可能用到的索引
5. key：实际使用到的索引
6. key_len：索引的长度，可用于判断复合索引是否被完全使用。
7. ref：表示查询表时参照的字段，如果是参照常量，则为const。
8. row：被索引优化查询的数据个数。
9. extra：
	9.1：using filesort，性能消耗大，需要一次额外的排序（查询）。
   		1）对于单索引来说，如果索引字段和排序字段不一样，则会出现using filesort。
		2）对于复合索引，如果跨列，则会出现using filesort（where和order by使用时避免跨列和无序使用）。
	9.2：using temporary，用到了临时表，一般出现在group by中。
	9.3：using index，覆盖索引，只从索引文件中获取数据，不需要回表。
	9.4：using where，需要回表查询。
	9.5：impossible where：where子句逻辑永远为false。

**知识补充**
1. order by 中的 using filesort排序方式有两种，双路排序（MySQL4.1之前）和单路排序（MySQL4.1之后）。
2. 双路排序，先扫描排序字段，在```buffer```中排序，再扫描其他字段。
3. 单路排序，一次读取全部字段，有一个弊端是不一定是一次IO，可能多次IO，可以适当设置排序使用的buffer的大小。

## 链接查询
1. 小表驱动大表，表数据量小的放左边。

## 避免索引失效
1. 复合索引要符合最左匹配原则。
2. 不要在索引字段进行任何操作（计算，函数，类型转换，大于，小于，in语句），否则索引就会失效。
3. 复合索引不能使用“不等于”和 is null的判断，会导致索引失效。
4. like语句尽量使用做匹配。xx like "%aaa"。
5. 尽量不使用类型转换，显式或隐式。
6. 尽量不要使用or，否则索引失效（包括or左边的语句使用索引失效）。
7. exist和in
	1. 如果主查询数据集大，用in。
	2. 如果子查询数据集大，用exist。exist会将主查询的结果放在子查询结果中进行校验。

**注意**
> sql结构的优化是概率层面的，因为MySQL会对sql语句进行优化器处理，所以需要通过explain语句推测优化后结果。

# 慢查询日志
```sql
-- 查看慢查询是否开启
show variables like '%slow_query_log%';

-- 临时打开慢查询，重启后失效
set global show_query_log = 1;

-- 永久开启
-- 编辑MySQL配置文件/etc/my.cnf，增加如下配置：
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/localhost_show_query_log.log

-- 查看慢查询阈值。
show variables like '%long_query_time%';

-- 查看慢查询的记录条数
show variables like '%slow_queries%';
```
	
# mysqldumpslow工具查看慢sql

# 锁机制
## 锁分类
**按操作类型分**
1. 读锁（也称共享锁）：对于同一个数据，多个读操作可以同时进行，互不干扰。
2. 写锁（也称互斥锁）：对于同一个数据，如果数据的操作没有完成，其他线程不能操作该数据。

**按操作数据的范围分**
1. 表锁：一次性对一整张表加锁。开销小，加锁快，无死锁。
	1. 如果一个数据库的会话对一张表A执行的读锁，那么该会话对表A是可读不可写，对其他表不可读写（当前会话锁着表A的）。而其他会话只是对表A可读，但是写操作会被阻塞，直到可以获取表A相应的锁。
	2. 如果一个会话对一张表A加写锁，该会话对表A可读可写，对其他表不可读写（当前会话锁着表A的）。其他会话对表A可读写（但是会阻塞，直到获取到表A的锁）。
2. 行锁：一次性对一条记录加锁。开销大，加锁慢，可能会出现死锁。
	1. 如果会话a对记录data进行了DML操作，其他会话在操作记录data时需要等待会话a操作完成后才能操作。
	2. 如果不同会话操作不同数据，无影响。
	3. 间隙锁：行锁的一种特殊情况，值在范围内，但却不存在（例如，主键为自增id，更新语句的where条件是id>0 且id<10，但id=6的）。

3. 页锁

```sql
-- 增加锁
lock table 表名1 read/write, 表名1 read/write;
-- 查看加锁的表
show open tables;
-- 分析表锁定的严重性
show status like 'table%';
-- 释放锁
unlock tables;
-- 查看表的索引
show index from 表名
```

**小知识**
> MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新语句前，会自动给涉及的所有表加写锁。

>  对于Innodb，如果更改操作没有使用索引，则会使用表锁。




