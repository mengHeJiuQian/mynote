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
		2）对于复合索引，如果跨列，则会出现using filesort（避免跨列和无序使用）。
	9.2：



