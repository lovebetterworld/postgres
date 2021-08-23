- [PostGIS管网连通性分析 ](https://juejin.cn/post/6955670546038128677)



GIS在管网数据中的很重要的一个应用方向就是”管网空间分析“，其中包括连通性分析、上下游分析、爆管分析等等。下面是我使用`postgis`来实现该“管网连通性分析”的解决方案，分享给大家，以便相互学习。

使用该分析之前确保已添加扩展`postgis`、`pgrouting`

```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;
```

# 导入数据

将数据导入到`postgreSQL`数据库，我是从`ArcGIS`直接导入的，导入方式参考[blog.csdn.net/eternity_xy…](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Feternity_xyf%2Farticle%2Fdetails%2F80168029)

# 创建拓扑

这里我用的管网数据表名为`pipe`

创建拓扑，生成`pipe_vertices_pgr`，该操作类似于`ArcGIS`中创建路网数据。

为`pipe`添加管段起始编号`pgr_source`、结束编号`pgr_target`、管段长度`pgr_length`三个字段，其中管段长度是用于分析的权重值。

```sql
--添加起点id
ALTER TABLE postgres.pipe ADD COLUMN IF NOT EXISTS pgr_source integer;

--添加终点id
ALTER TABLE postgres.pipe ADD COLUMN IF NOT EXISTS pgr_target integer;

--添加权重值
ALTER TABLE postgres.pipe ADD COLUMN IF NOT EXISTS pgr_length double precision;
```

为`pgr_source`、`pgr_target`创建索引

```sql
--为pgr_source字段创建索引
CREATE INDEX IF NOT EXISTS pgr_source_idx ON postgres.pipe("pgr_source")

--为pgr_target字段创建索引
CREATE INDEX IF NOT EXISTS pgr_target_idx ON postgres.pipe("pgr_target")
```

为权重字段`pgr_length`赋值

```sql
--为pgr_length赋值，shape为几何类型的字段，可能为shape、the_geom，通过ArcGIS导入的时候字段为"shape"，其他方式导入时一般为"the_geom"
update postgres.pipe set pgr_length = public.st_length(shape)
```

调用`pgr_createTopology`方法，创建拓扑，这个步骤会为`pgr_source`和`pgr_target`字段赋值，同时生成节点表`pipe_vertices_pgr`

```sql
--为目标表创建拓扑布局，即为pgr_source和pgr_target字段赋值
select public.pgr_createTopology('postgres.pipe',0.000001,'shape','objectid','pgr_source','pgr_target')
```

# 计算联通性

根据起点坐标、终点坐标从`pipe_vertices_pgr`查询最近的起点、终点标识

![image-20210426182158735](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed55469791044b0ab105c6b20a6bb1ff~tplv-k3u1fbpfcp-watermark.image)

调用`pgr_kdijkstraPath`函数，查询出起点、终点联通的线。

![image-20210315104505557](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38dc881e5ae34a0587d9aa4b6217f3ed~tplv-k3u1fbpfcp-watermark.image)

通过这里我们可以看出，该分析的核心是调用了`pgrouting`扩展中的求最短路径的函数`pgr_kdijkstraPath`，该函数用的是是[Dijkstra](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Flbperfect123%2Farticle%2Fdetails%2F84281300)算法，通过已添加的索引`pgr_source`和`pgr_target`以及权重值`pgr_length`，计算出两点之间的最短路径，如果有最短路径，证明两点联通。

该分析可用于计算给水管网、排水管网、输油管道等管网数据的两节点的连通性，当然也可用于路网的最短路径分析。

# 函数脚本

上面为整体分析思路，现在将上述思路整理成函数，方便使用

1. 创建拓扑函数：[analysis_updatetopology()](https://link.juejin.cn?target=http%3A%2F%2Fgisarmory.xyz%2Fblog%2Findex.html%3Fsource%3DPostGISUpdateTopology)
2. 计算连通性函数：[analysis_connect()](https://link.juejin.cn?target=http%3A%2F%2Fgisarmory.xyz%2Fblog%2Findex.html%3Fsource%3DPostGISConnect)

# 如何使用

1. 调用`analysis_updatetopology()`函数，完成拓扑创建

   ```sql
   -- 传入表名pipe，创建拓扑
   select * from analysis_updatetopology('pipe')
   ```

2. 从地图选择起点、终点，然后调用`analysis_connect()`函数，得到分析结果

   ```sql
   -- 传入表名、起点坐标、终点坐标、容差值
   select * from analysis_connect('pipe',103.90893393,30.789659886,103.911700936,30.787850094,0.00001)
   ```

------

原文地址：[gisarmory.xyz/blog/index.…](https://link.juejin.cn?target=http%3A%2F%2Fgisarmory.xyz%2Fblog%2Findex.html%3Fblog%3DPostGISConnect)