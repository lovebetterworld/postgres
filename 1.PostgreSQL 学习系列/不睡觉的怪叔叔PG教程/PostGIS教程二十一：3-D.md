# PostGIS教程二十一：3-D

 注意：本节介绍许多PostGIS2.0及更高版本才支持的功能。

# 一、3-D几何图形

  到目前为止，我们一直在处理2-D几何图形（二维几何图形），只有X和Y坐标。但是PostGIS支持所有几何图形类型额外的维度，对于每个坐标，另外还能支持用于表示高度信息的"**Z**"维度以及用于添加额外附加信息的"**M**"维度（通常为时间、道路英里或距离信息）。

  对于3-D和4-D几何图形，额外的维度将作为几何图形中每个顶点的额外坐标添加，并且几何图形类型将得到增强，以指示如何解译额外的维度。添加额外维度会为每个基本几何图形额外添加三种几何图形类型：

- 点（二维类型）  ——  PointZ、PointM和PointZM
- 线串（二维类型）  ——  LinestringZ、LinestringM和LinestringZM
- 多边形（二维类型）  ——  PolygonZ、PolygonM和PolygonZM
- 等等

  对于well-known text（[WKT](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkt)）的表示，高维几何图形由**ISO SQL/MM**规范提供。附加维度信息仅添加到类型名称后的文本字符串中，额外坐标添加在X/Y信息之后，例如：

- POINT ZM(1 2 3 4)
- LINESTRING M(1 1 0, 1 2 0, 1 3 1, 2 2 0)
- POLYGON Z((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0))

  在处理3-D和4-D几何图形时，ST_AsText()函数将返回上述表示。

  对于well-known binary（[WKB](https://postgis.net/workshops/postgis-intro/glossary.html#term-wkb)）表示，高维几何图形的格式由ISO SQL/MM规范提供。该格式的BNF表可在http://svn.osgeo.org/postgis/trunk/doc/bnf-wkb.txt 查询。

  除了标准类型的高维形式外，PostGIS还包括一些在三维空间中有意义的新类型：

- TIN（不规则三角网）类型允许将三角形网格建模为数据库中的行。
- 使用POLYHEDRALSURFACE可以在数据库中对体积对象进行建模。

  由于这两种类型都用于对三维对象建模，因此使用Z变量才是真正有意义的。POLYHEDRALSURFACE Z的一个示例是1单位立方体：

```sql
POLYHEDRALSURFACE Z (
  ((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0)),
  ((0 0 0, 0 1 0, 0 1 1, 0 0 1, 0 0 0)),
  ((0 0 0, 1 0 0, 1 0 1, 0 0 1, 0 0 0)),
  ((1 1 1, 1 0 1, 0 0 1, 0 1 1, 1 1 1)),
  ((1 1 1, 1 0 1, 1 0 0, 1 1 0, 1 1 1)),
  ((1 1 1, 1 1 0, 0 1 0, 0 1 1, 1 1 1))
)
```

 

# 二、3-D函数

  有许多函数可用于计算三维对象之间的关系：

![img](https://img-blog.csdnimg.cn/2019030509121125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  例如，我们可以使用**ST_3DDistance**函数计算单位立方体与点之间的距离：

```sql
-- This is really the distance between the top corner
-- and the point.
SELECT ST_3DDistance(
  'POLYHEDRALSURFACE Z (
    ((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0)),
    ((0 0 0, 0 1 0, 0 1 1, 0 0 1, 0 0 0)),
    ((0 0 0, 1 0 0, 1 0 1, 0 0 1, 0 0 0)),
    ((1 1 1, 1 0 1, 0 0 1, 0 1 1, 1 1 1)),
    ((1 1 1, 1 0 1, 1 0 0, 1 1 0, 1 1 1)),
    ((1 1 1, 1 1 0, 0 1 0, 0 1 1, 1 1 1))
  )'::geometry,
  'POINT Z (2 2 2)'::geometry
);
-- So here's a shorter form.
SELECT ST_3DDistance(
  'POINT Z (1 1 1)'::geometry,
  'POINT Z (2 2 2)'::geometry
);
-- Both return 1.73205080756888 == sqrt(3) as expected
```

 

# 三、N-D索引

  一旦有了更高维度的数据，对其进行索引可能是有意义的。但是，在应用**多维索引**（multi-dimensional index）之前，应该仔细考虑数据在所有维度中的分布情况。

  索引仅在允许数据库中WHERE条件查询中大幅减少返回行数时才有用。要使更高维度的索引有用，数据必须分布在该维度的广泛范围中（相对于你正在构造的查询）

- 一组DEM（数字高程模型）点可能并不适合构建3-D索引，因为涉及到的查询通常是提取一个2-D点框，而很少尝试选择Z维度上的信息。
- 如果X/Y/T空间中的一组GPS轨迹在所有维度上频繁重叠，这可能适合构建3-D索引，因为数据集在所有维度都会有很大的差异。

  你可以为任何维度（甚至混合维度）的数据创建多维索引。例如，要在nyc_streets表上创建多维索引。

```sql
CREATE INDEX nyc_streets_gix_nd ON nyc_streets
USING GIST (geom gist_geometry_ops_nd);
```

![img](https://img-blog.csdnimg.cn/20190305094835341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **gist_geometry_ops_nd**参数告诉PostGIS使用**N-D索引**而不是标准的**2-D索引**。

  构建索引后，可以在查询中使用**&&&**索引操作符，&&&和&&是相同的语义——边界框相交——区别在于，&&&使用几何图形的所有维度来应用这个语义。维数不匹配的几何图形不会相交。

```sql
-- Returns true (both 3-D on the zero plane)
SELECT 'POINT Z (1 1 0)'::geometry &&&
       'POLYGON ((0 0 0, 0 2 0, 2 2 0, 2 0 0, 0 0 0))'::geometry;
-- Returns false (one 2-D one 3-D)
SELECT 'POINT Z (1 1 1)'::geometry &&&
       'POLYGON ((0 0, 0 2, 2 2, 2 0, 0 0))'::geometry;
-- Returns true (the volume around the linestring interacts with the point)
SELECT 'LINESTRING Z(0 0 0, 1 1 1)'::geometry &&&
       'POINT(0 1 1)'::geometry;
```

  要使用**N-D索引**搜索nyc_streets表，只需将&&2-D索引运算符替换为&&&3-D索引运算符。

```sql
-- N-D index operator
SELECT gid, name
FROM nyc_streets
WHERE geom &&&
      ST_SetSRID('LINESTRING(586785 4492901,587561 4493037)' :: geometry,26918);
-- 2-D index operator
SELECT gid, name
FROM nyc_streets
WHERE geom &&
      ST_SetSRID('LINESTRING(586785 4492901,587561 4493037)' :: geometry,26918);
```

  结果应该是一样的。一般来说，**N-D索引**只比**2-D索引**执行速度稍慢一些，所以只使用**N-D索引**，因为**N-D查询**将提高查询的多维度选择性。