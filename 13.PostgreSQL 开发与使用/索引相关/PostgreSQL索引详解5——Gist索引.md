- [PostgreSQL索引详解5——Gist索引](https://blog.csdn.net/weixin_39540651/article/details/105998418)

## 1、概述

Gist(Generalized Search Tree)，即通用搜索树。和btree一样，也是平衡的搜索树。
 和btree不同的是，btree索引常常用来进行例如大于、小于、等于这些操作中，而在实际生活中很多数据其实不适用这种场景，例如地理数据、图像等等。如果我们想要查询在某个地方是否存在某一点，即判断地理位置的"包含"那么我们就可以使用gist索引了。
  因为gist索引允许定义规则来将任意类型的数据分布到一个平衡的树中，并且允许定义一个方法使用此表示形式来让某些运算符访问。例如，对于空间数据，GiST索引可以使用 R树，以支持相对位置运算符（位于左侧，右侧，包含等），而对于树形图，R树可以支持相交或包含运算符。

## 2、结构

Gist索引的结构可以简单概括为：

- 树结构，深度一致。Tree-structure
- 索引PAGE内的数据不按KEY的顺序排序。No order within pages
- 不同的索引PAGE的内容可能存在交叉，例如RANGE类型，值交叉。Key ranges of pages can overlap
- 由于第三条，所以搜索一条TUPLE时，可能会找到多个满足条件的PAGE。No single “correct” location for
   a particular tuple

**2.1、单个gist索引page的结构：**
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508152819251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTU0MDY1MQ==,size_16,color_FFFFFF,t_70)
 索引page中存放的是key+tid，例如上图中[100,150]表示100到150这个范围，(1,10)表示行的ctid（第几个HEAP PAGE，里面的第几条记录）。同时，这些存放的值在索引中是无序的。

**2.2、Gist两级索引结构：**
 GiST两级索引长这样，上一级代表下一级中单个INDEX PAGE的大范围。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508152837842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTU0MDY1MQ==,size_16,color_FFFFFF,t_70)
 例如我们现在查询[55,60]这个范围，在索引中是如何工作的呢？
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508152855150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTU0MDY1MQ==,size_16,color_FFFFFF,t_70)

## 3、使用实例

在使用之前，我们先来看看Gist索引的使用场景有哪些。
 因为gist是一个通用的索引接口，所以可以使用GiST实现b-tree, r-tree等索引结构。
 不同的类型，支持的索引检索也各不一样。例如：
 1、几何类型，支持位置搜索（包含、相交、在上下左右等），按距离排序。
 2、范围类型，支持位置搜索（包含、相交、在左右等）。
 3、IP类型，支持位置搜索（包含、相交、在左右等）。
 4、空间类型（PostGIS），支持位置搜索（包含、相交、在上下左右等），按距离排序。
 5、标量类型，支持按距离排序。

**3.1、几何类型检索**
 创建一个存放几何数据的表：

```sql
bill@bill=>create table t_gist (id int, pos point);    
CREATE TABLE
bill@bill=>insert into t_gist select generate_series(1,100000), point(round((random()*1000)::numeric, 2), round((random()*1000)::numeric, 2));    
INSERT 0 100000
bill@bill=>select * from t_gist limit 5;
 id |       pos       
----+-----------------
  1 | (614.87,412.97)
  2 | (248.71,779.61)
  3 | (976.17,826.79)
  4 | (999.39,126.9)
  5 | (651.61,364.49)
(5 rows)
```

在pos列上创建gist索引：

```sql
bill@bill=>create index idx_t_gist_1 on t_gist using gist (pos);  
CREATE INDEX
```

查询：
 可以发现gist索引支持bitmap scan和index scan。

```sql
bill@bill=>explain (analyze,verbose,timing,costs,buffers) select * from t_gist where circle '((100,100) 10)'  @> pos;    
                                                       QUERY PLAN                                                       
------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on bill.t_gist  (cost=2.35..113.84 rows=100 width=20) (actual time=0.107..0.157 rows=35 loops=1)
   Output: id, pos
   Recheck Cond: ('<(100,100),10>'::circle @> t_gist.pos)
   Heap Blocks: exact=32
   Buffers: shared hit=38
   ->  Bitmap Index Scan on idx_t_gist_1  (cost=0.00..2.33 rows=100 width=0) (actual time=0.093..0.093 rows=35 loops=1)
         Index Cond: (t_gist.pos <@ '<(100,100),10>'::circle)
         Buffers: shared hit=6
 Planning Time: 0.188 ms
 Execution Time: 0.268 ms
(10 rows)

bill@bill=>explain (analyze,verbose,timing,costs,buffers) select * from t_gist where circle '((100,100) 1)' @> pos order by pos <-> '(100,100)' limit 10; 
                                                             QUERY PLAN                                                              
-------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.28..12.72 rows=10 width=28) (actual time=0.084..0.085 rows=1 loops=1)
   Output: id, pos, ((pos <-> '(100,100)'::point))
   Buffers: shared hit=4
   ->  Index Scan using idx_t_gist_1 on bill.t_gist  (cost=0.28..124.73 rows=100 width=28) (actual time=0.082..0.083 rows=1 loops=1)
         Output: id, pos, (pos <-> '(100,100)'::point)
         Index Cond: (t_gist.pos <@ '<(100,100),1>'::circle)
         Order By: (t_gist.pos <-> '(100,100)'::point)
         Buffers: shared hit=4
 Planning Time: 0.175 ms
 Execution Time: 0.129 ms
(10 rows)
```

**3.2、标量类型**
 因为gist可以实现btree的索引结构，所以我们也可以在例如数字这种标量类型上使用gist索引(虽然一般都不如btree索引效果好)，不过我们还需要使用btree_gist索引插件。

```sql
bill@bill=>create extension btree_gist ;
CREATE EXTENSION
bill@bill=>create table t_btree(id int,info text);
CREATE TABLE
bill@bill=>insert into t_btree select generate_series(1,100000),md5(random()::text);
INSERT 0 100000
```

创建gist索引：

```sql
bill@bill=>create index idx_t_btree on t_btree using gist(id);
CREATE INDEX
```

查询：

```sql
bill@bill=>explain (analyze,verbose,timing,costs,buffers) select * from t_btree order by id <-> 100 limit 1;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=0.28..0.32 rows=1 width=41) (actual time=0.072..0.073 rows=1 loops=1)
   Output: id, info, ((id <-> 100))
   Buffers: shared hit=4
   ->  Index Scan using idx_t_btree on bill.t_btree  (cost=0.28..3798.18 rows=100000 width=41) (actual time=0.071..0.071 rows=1 loops=1)
         Output: id, info, (id <-> 100)
         Order By: (t_btree.id <-> 100)
         Buffers: shared hit=4
 Planning Time: 0.334 ms
 Execution Time: 0.118 ms
(9 rows)
```

## **4、总结**

Gist索引实现首先要将数据聚集，聚集后，在单个组内包含的KEY+HEAP行号会放到单个INDEX PAGE中。
 聚集的范围作为一级结构，存储在GiST的entry 中，便于检索。
 既然灵魂是聚集，那么GiST的性能就和他的聚集算法息息相关，PostgreSQL把这个接口留给了用户，用户在自定义数据类型时，如果要自己实现对应的GIST索引，那么就好好考虑这个类型聚集怎么做吧。
 PostgreSQL内置的range, geometry等类型的GIST已经帮你做好了，你只需要做新增的类型，比如你新增了一个存储人体结构的类型，存储图片的类型，或者存储X光片的类型，怎么快速检索它们，那就是你要实现的GIST索引聚集部分了。

## 参考链接：

https://habr.com/ru/company/postgrespro/blog/444742/

https://www.cnblogs.com/cmi-sh-love/p/kong-jian-shud-ju-suo-yinRTree-wan-quan-jie-xi-jiJa.html

https://github.com/digoal/blog/blob/master/201906/20190604_03.md

