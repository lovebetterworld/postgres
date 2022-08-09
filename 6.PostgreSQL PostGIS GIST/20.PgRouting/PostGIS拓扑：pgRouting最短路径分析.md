- [PostGIS拓扑：pgRouting最短路径分析 - GoodGF - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaofan/p/11531429.html)

前提：在PostgreSQL中建立PostGIS数据库，安装pgRouting插件，导入现有的线表shp数据（示例使用的是管线pipesectionmain,其他的线表数据均可）。

## 1、pgRouting在edge表中添加字段

线表中必须有id,source,target,cost,the_geom 5个字段，其中现有空间数据表中的gid可作为id,shape_leng可作为cost,geom可作为the_geom。还需要额外增加source和target字段

新增souce和target字段并加上索引

```sql
alter table waterdataset.pipesectionmain add column source int;

alter table waterdataset.pipesectionmain add column target int;

create index road_source_idx on waterdataset.pipesectionmain("source");

create index road_target_idx on waterdataset.pipesectionmain("target");
```

如现有空间表中没有长度字段，可通过以下语句初始化

```sql
ALTER TABLE waterdataset.pipesectionmain  ADD COLUMN length double precision; 
update waterdataset.pipesectionmain set length =st_length(geom);
```

## 2、建立拓扑

```sql
SELECT pgr_createTopology('waterdataset.pipesectionmain',0.001, 'geom', 'gid'); 
```

执行后会在相应的架构下创建pipesectionmain_vertices_pgr表

注：边表pipesectionmain生成的节点表，路径分析时的起止点编号均来源于此表；

对现有topo进行几何分析，检查现有几何错误（非必要步骤）

```sql
SELECT pgr_analyzegraph('waterdataset.pipesectionmain', 0.001,'geom', 'gid');
```

修正topo并输出修正过的边数据到新表（非必要步骤）

```sql
SELECT pgr_nodeNetwork('waterdataset.pipesectionmain', 0.001,'gid','geom');
```

## 3、调用pgr_dijkstra进行最短路径分析

pgr_dijkstra函数使用有以下几种方式

```sql
//起止点均为单点（一对一）

pgr_dijkstra(edges_sql, start_vid, end_vid)

pgr_dijkstra(edges_sql, start_vid, end_vid, directed:=true)

//起点为单点，终点为多点（一对多）

pgr_dijkstra(edges_sql, start_vid, end_vids, directed:=true)

//起点为多点，终点为单点（多对一）

pgr_dijkstra(edges_sql, start_vids, end_vid, directed:=true)

//起点终点均为多点（多对多）

pgr_dijkstra(edges_sql, start_vids, end_vids, directed:=true)
```

参数解析

| **参数**          | **类型**      | **默认** | **描述**                                          |
| ----------------- | ------------- | -------- | ------------------------------------------------- |
| **edges_****sql** | TEXT          |          | 边表查询语句，查询结果需包含id,source,target,cost |
| **start_vid**     | BIGINT        |          | 起点id                                            |
| **start_vids**    | ARRAY[BIGINT] |          | 起点id数组                                        |
| **end_vid**       | BIGINT        |          | 终点id                                            |
| **end_vids**      | ARRAY[BIGINT] |          | 终点id数组                                        |
| **directed**      | BOOLEAN       | true     | 默认是 true，设置为有向图 false ，设置为无向图    |

我们下面示例为一对一方式：

 从建立拓扑生成的节点表pipesectionmain_vertices_pgr中选择起点4093，终点2350（可在QGIS中加载线表pipesectionmain和点表pipesectionmain_vertices_pgr，方便查看与选择）。

由于我们没有创建topo所需要的所有字段，有部分是用现有字段替代的，因此，在调用最短路径分析函数时，需在sql中显示指定这些字段。

```sql
select pgr_dijkstra('SELECT gid AS id,                     

source::integer,                        

target::integer,                       

shape_leng::double precision AS cost 

FROM  waterdataset.pipesectionmain', 4093, 2350,false)
```

##  4、查询结果

```sql
select * from pgr_dijkstra('SELECT gid AS id,                     

source::integer,                        

target::integer,                       

length::double precision AS cost 

FROM  waterdataset.pipesectionmain', 4093, 2350,false);
```

1）可查看返回结果

![img](https://img2018.cnblogs.com/blog/76794/201909/76794-20190917085905436-26849618.png)

结果解析

| **列**       | **类型** | **描述**                                                |
| ------------ | -------- | ------------------------------------------------------- |
| **seq**      | INT      | 从1开始的序号                                           |
| **path_seq** | INT      | 路径上的相对位置，从1开始的序号                         |
| **node**     | BIGINT   | 节点id                                                  |
| **edge**     | BIGINT   | 边id（上述节点关联的下一条边）. -1表示最后一个边不存在. |
| **cost**     | FLOAT    | 当前路径花费                                            |
| **agg_cost** | FLOAT    | 到目前为止路径花费累加                                  |

2）查看图形结果

![img](https://img2018.cnblogs.com/blog/76794/201909/76794-20190917083235228-1921723810.png)

3）在QGIS中用pgroutinglayer插件查看结果

![img](https://img2018.cnblogs.com/blog/76794/201909/76794-20190917083227563-1581884788.png)

