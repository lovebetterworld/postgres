- [PostgreSql 索引简明教程](https://blog.csdn.net/neweastsun/article/details/119205148)

# PostgreSql 索引简明教程

> 索引是数据库引擎加速获取数据的查找表。简言之，索引是指向表数据的指针，类似书的目录。
>
> PostgreSql 提供了Btree、Hash、GiST、SP-GiST、GIN、BRIN等多种索引类型，每种索引类型使用不同的算法来适应不同类型的查询。在默认情况下，创建的索引类型为B-tree索引。

## 1. 索引概述

索引可以加速查询，但会降低插入或更新速度。创建或删除索引不会影响数据。

### 创建索引

创建索引语法：

```sql
CREATE INDEX index_name ON table_name [USING method]
(
    column_name [ASC | DESC] [NULLS {FIRST | LAST }],
    ...
);
```

- 首先在 CREATE INDEX 后面指定索引名称，名称应该有意义方便记忆和理解。
- 其次指定表名，表示在那个表上建索引。
- 第三，指定索引方法，比如：Btree、Hash、GiST、SP-GiST、GIN、BRIN等，默认为Btree。
- 第四，指定一个或多个列。后面是排序方式及null值排序。缺省为DESC 和 NULLS LAST。

### 删除索引

```sql
DROP INDEX index_name;
```

使用drop 命令删除索引。删除索引时要小心，会对性能有负面或正面影响。

## 2. 索引类型

### 单列索引

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

### 多列索引

```sql
CREATE INDEX index_name
ON table_name (column1_name, column2_name);
```

无论是单列索引或多列索引，应考虑这些列应该在where条件子句中作为过滤条件频繁使用。

### 唯一索引

```sql
CREATE UNIQUE INDEX index_name
on table_name (column_name);
```

唯一索引不仅提升性能，也是为实现数据完整性，它不允许插入重复值。

### 局部索引

```sql
CREATE INDEX idx_customer_inactive
ON customer(active)               -- 指定表和字段
WHERE active = 0;                 -- 指定过滤条件
```

可以参考[这篇博文](https://blog.csdn.net/neweastsun/article/details/113096327?ops_request_misc=%7B%22request%5Fid%22%3A%22162752297816780269854510%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=162752297816780269854510&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-113096327.pc_v2_rank_blog_default&utm_term=部分索引&spm=1018.2226.3001.4450)，上面示例我们仅对 active=0 的记录感兴趣。

### 表达式索引

```sql
CREATE INDEX index_name 
ON table_name (expression);
```

当对应 expression 在where或order  by子句中出现时，数据库会使用该索引。注意，维护表达式上的索引是相当昂贵的，因为当插入或更新每一行时，数据库系统必须对表达式求值，并使用结果进行索引。因此，当检索速度比插入和更新速度更重要时，才使用表达式上索引。

举例：

```sql
CREATE INDEX idx_ic_last_name
ON customer(LOWER(last_name));
```

对 last_name 的 值转为小写的索引。

### 隐式索引

是数据库自动创建的索引，当我们创建主键和约束和唯一约束时，数据库系统自动帮我们创建的索引。

## 3. 索引方法

### Btree索引

Btree索引使用Btree数据结构来存储索引数据，可用于处理等值查询和范围查询，包括＜、＜=、=、＞=、＞等运算符，以及BETWEEN、IN、IS NULL、IS NOT NULL 等条件。

```sql
<
<=
=
>=
BETWEEN
IN
IS NULL
IS NOT NULL
```

Btree索引可用于模式匹配查询，如“col LIKE′foo%′”或“col～′^foo′”，但是不能用于“col LIKE′%bar′”之类的后缀模糊匹配查询。Btree索引还可以用于查询结果集排序，如ORDER BY排序。

```sql
column_name LIKE 'foo%' 
column_name LKE 'bar%' 
column_name  ~ '^foo'
```

### Hash索引

Hash索引根据每一行数据的索引字段计算哈希码，并维护哈希码、记录指针对应关系。对于哈希码相同的数据来说，可以采用链表来解决冲突。Hash索引的查询速度很快。

```sql
CREATE INDEX index_name 
ON table_name USING HASH (indexed_column);
```

### GiST索引

GiST（Generalized Search  Tree）是一种平衡的树型结构访问方法，可作为一种基础模板来实现任意索引模式。B-tree索引和许多其他的索引模式都可以用GiST索引来实现。GiST 索引适用于多维数据类型和集合数据类型。GiST  多列索引支持在查询条件中包含索引字段的子集。PostgreSQL包含了全文检索、几何数据类型等多个用GiST实现的索引方法。

### SP-GiST索引

SP-GiST索引与GiST索引类似，可作为一种基础模板来实现多种搜索方法。SP-GiST索引主要实现非平衡的基于硬盘的数据结构，如四叉树、k-d树和radix树。

### GIN索引

GIN索引是一种通用倒排索引（GIN stands for **g**eneralized **in**verted indexes），可以处理包含多个键值，如 hstore, array，jsonb, 和 range 类型。用它来全文搜索或JSON键值的效率很高。GIN 允许用户开发自定义访问方法的数据类型索引，可以支持多种不同用户定义的索引策略。

### BRIN索引

BRIN索引BRIN表示块范围索引。BRIN索引存储连续相邻的数据块统计信息，可以大大缩小索引占用空间。对于数据量比较大的表来说，BRIN索引比B-Tree索引插入数据的速度要快，两者的范围查询效率相当。BRIN索引通常用于线性顺序排列的列，如订单表的创建日期。

### 布隆过滤索引

Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。Bloom  Filter的这种高效是有一定代价的：在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false  positive）。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom  Filter通过极少的错误换取了存储空间的极大节省。

读者可以查找相关资源更深入了解，这里简单示例如何使用布隆过滤索引：

```sql
CREATE EXTENSION bloom;
CREATE INDEX SomeIndex ON SomeTable USING bloom (c1, c2, c3, c4);
-- a query which may take advantage of the Bloom index
SELECT * FROM SomeTable WHERE c2 = 5432 AND c4 = 1234;
```

## 4. 总结

虽然索引用于提升数据库性能，但有时我们应该避免使用索引。下面列举几条供参考：

- 数据量小的表不要使用索引
- 频繁大批量插入、更新操作的表不要索引
- 列上有大量null值，不应该建索引
- 列值被频繁维护，不应该建索引