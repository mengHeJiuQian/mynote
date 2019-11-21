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





1. 1. 1. 1. 1. 
