- [pgr_createTopology — pgRouting Manual (2.2)](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_createTopology.html)

## 1 pgr_createTopology 概述

函数作用：根据几何信息构建网络拓扑。

函数返回:

- 在网络拓扑图和顶点表建立好之后就OK了。
- 由于错误导致网络拓扑构建失败。

## 2 pgr_createTopology 参数说明

```sql
varchar pgr_createTopology(text edge_table, double precision tolerance,
                   text the_geom:='the_geom', text id:='id',
                   text source:='source',text target:='target',
                   text rows_where:='true', boolean clean:=false)
```

拓扑创建函数接受以下参数：

**edge_table**：text，表名

**tolerance**：float8，误差缓冲值，两个点的距离在这个距离内，就算重合为一点。这个距离使用st_length计算

**the_geom**：text，该表的空间坐标字段 

**id**：text，该表的主键

**source**：text，空间起点编号

**target**：text，空间终点编号

**rows_where**：text，条件选择子集或行。默认值为true，表示源或目标具有空值的所有行，否则将使用条件。

**clean**：text，每次执行都重建拓扑图，默认false

说明：

- 执行SQL后，数据库中会创建或更新soure、target字段。
- 如果索引不存在，将创建一个索引，以加快以下列的进程：id、the_geom、source、target。

函数返回：

当创建拓扑结果成功时：

- 则会自动创建出来一个新表，存储了节点信息，表名：<edge_table>_vertices_pgr.

- 填充顶点表的id和the_geom列。新表会自动生成id和the_geom列， 并填充数据。
- 引用顶点表的id填充边缘表的源列和目标列。即关联新表和旧表，新表的ID关联旧表的source和target。

当创建拓扑失败时：

- 找不到表中所需的列，或者该列的类型不合适。
- 生成的数据构造的条件不合适，貌似就是数据结构不合理，构造出的数据不对。
- source、target或者id相同。
- SRID无法确定。

## 3 The Vertices Table

新创建出来的点集表包涵函数[*pgr_analyzeGraph*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeGraph.html#pgr-analyze-graph) and [pgr_analyzeOneway](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeOneWay.html#pgr-analyze-oneway) 。

新表结构为：

| id:       | `bigint` 顶点的标识符。                                      |
| :-------- | ------------------------------------------------------------ |
| cnt:      | `integer` edge_table中引用此顶点的顶点数。See [*pgr_analyzeGraph*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeGraph.html#pgr-analyze-graph). |
| chk:      | `integer` 指示顶点可能有问题。See [*pgr_analyzeGraph*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeGraph.html#pgr-analyze-graph). |
| ein:      | `integer` edge_table中引用这个顶点的顶点数。See [*pgr_analyzeOneway*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeOneWay.html#pgr-analyze-oneway). |
| eout:     | `integer` edge_table中引用该顶点作为输出的顶点数。See [*pgr_analyzeOneway*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeOneWay.html#pgr-analyze-oneway). |
| the_geom: | `geometry` Point geometry of the vertex：顶点的点几何。      |

## 4 SQL注意事项

### 4.1 参数使用顺序

当参数按形参中描述的顺序给出时，我们得到的结果与使用函数的最简单方法相同。

```sql
SELECT  pgr_createTopology('edge_table', 0.001,
        'the_geom', 'id', 'source', 'target');
NOTICE:  PROCESSING:
NOTICE:  pgr_createTopology('edge_table', 0.001, 'the_geom', 'id', 'source', 'target', rows_where := 'true', clean := f)
NOTICE:  Performing checks, please wait .....
NOTICE:  Creating Topology, Please wait...
NOTICE:  -------------> TOPOLOGY CREATED FOR  18 edges
NOTICE:  Rows with NULL geometry or NULL id: 0
NOTICE:  Vertices table for table public.edge_table is: public.edge_table_vertices_pgr
NOTICE:  ----------------------------------------------
 pgr_createtopology 
--------------------
 OK
(1 row)
```

警告:

当参数没有按适当的顺序给出时，将会发生错误。

在这个例子中，表ege_table的列id作为几何列传递给函数，而几何列the_geom作为id列传递给函数。

```sql
SELECT  pgr_createTopology('edge_table', 0.001,
        'id', 'the_geom');
NOTICE:  PROCESSING:
NOTICE:  pgr_createTopology('edge_table', 0.001, 'id', 'the_geom', 'source', 'target', rows_where := 'true', clean := f)
NOTICE:  Performing checks, please wait .....
NOTICE:  ----> PGR ERROR in pgr_createTopology: Wrong type of Column id:the_geom
NOTICE:  Unexpected error raise_exception
 pgr_createtopology 
--------------------
 FAIL
(1 row)
```

### 4.2 命名表示法

当使用命名表示法时，用默认值定义的参数可以省略，只要值与默认值匹配，并且参数的顺序不重要。

```sql
SELECT  pgr_createTopology('edge_table', 0.001,
                           the_geom:='the_geom', id:='id', source:='source', target:='target');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

```sql
SELECT  pgr_createTopology('edge_table', 0.001,
                           source:='source', id:='id', target:='target', the_geom:='the_geom');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

```sql
SELECT  pgr_createTopology('edge_table', 0.001, source:='source');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

### 4.3 选择指定行

使用rows_where参数选择行，根据id选择行。

```sql
SELECT  pgr_createTopology('edge_table', 0.001, rows_where:='id < 10');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

选择与id = 5行的几何图形相邻的行。

```sql
SELECT  pgr_createTopology('edge_table', 0.001,
        rows_where:='the_geom && (SELECT st_buffer(the_geom, 0.05) FROM edge_table WHERE id=5)');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

Selecting the rows where the geometry is near the geometry of the row with `gid` =100 of the table `othertable`.

```sql
CREATE TABLE otherTable AS  (SELECT 100 AS gid,  st_point(2.5, 2.5) AS other_geom);
SELECT 1
SELECT  pgr_createTopology('edge_table', 0.001,
                           rows_where:='the_geom && (SELECT st_buffer(other_geom, 1) FROM otherTable WHERE gid=100)');
 pgr_createtopology 
--------------------
 OK
(1 row)
```

## 5 示例

这个例子开始一个干净的拓扑，有5条边，然后增加到其余的边。

```sql
SELECT pgr_createTopology('edge_table',  0.001, rows_where:='id < 6', clean := true);
NOTICE:  PROCESSING:
NOTICE:  pgr_createTopology('edge_table', 0.001, 'the_geom', 'id', 'source', 'target', rows_where := 'id < 6', clean := t)
NOTICE:  Performing checks, please wait .....
NOTICE:  Creating Topology, Please wait...
NOTICE:  -------------> TOPOLOGY CREATED FOR  5 edges
NOTICE:  Rows with NULL geometry or NULL id: 0
NOTICE:  Vertices table for table public.edge_table is: public.edge_table_vertices_pgr
NOTICE:  ----------------------------------------------
 pgr_createtopology 
--------------------
 OK
(1 row)

SELECT pgr_createTopology('edge_table',  0.001);
NOTICE:  PROCESSING:
NOTICE:  pgr_createTopology('edge_table', 0.001, 'the_geom', 'id', 'source', 'target', rows_where := 'true', clean := f)
NOTICE:  Performing checks, please wait .....
NOTICE:  Creating Topology, Please wait...
NOTICE:  -------------> TOPOLOGY CREATED FOR  13 edges
NOTICE:  Rows with NULL geometry or NULL id: 0
NOTICE:  Vertices table for table public.edge_table is: public.edge_table_vertices_pgr
NOTICE:  ----------------------------------------------
 pgr_createtopology 
--------------------
 OK
(1 row)
```

