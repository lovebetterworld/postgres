# PostGIS空间数据类型的组织与表达（geography数据类型）

作者：不睡觉的怪叔叔

地址：https://zhuanlan.zhihu.com/p/105909086



在PostGIS中还有另一种数据类型**Geography**，它们的最大不同之处是对空间数据的计算方式不同。

​     例如，计算两个坐标点的距离，Geometry数据类型的数据是基于平面坐标来计算的，也就是将两个坐标点看作是处于同一个平面，计算它们之间的平面距离。而Geography数据类型的数据是基于球面坐标来计算的，也就是将两个坐标点看作地球参考椭球体上的两个坐标点，计算它们之间的[大圆航线](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E5%A4%A7%E5%9C%86%E8%88%AA%E7%BA%BF/8760252%3Ffr%3Daladdin)的距离。

​    我们都知道，在球面上的计算比在平面上的计算更复杂，代价更高，所以Geography数据类型支持的空间函数比Geometry数据类型支持的空间函数少。不过，Geography数据类型与Geometry数据类型可以相互转换。

​    另外，Geography数据类型早期只支持WGS84地理坐标系，但是现在Geography数据类型可以支持**spatial_ref_sys**表中定义的所有地理坐标系（注意！是所有地理坐标系，而不是投影坐标系，因为它是基于地球参考椭球体的）。

# 一、Geography基础

​    Geography数据类型除了不支持CURE、TIN、POLYHEDRALSURFACE类型，它支持所有Geometry中的类型：

- POINT
- LINESTRING
- POLYGON
- MULTIPOINT
- MULTILINESTRING
- MULTIPOLYGON
- GEOMETRYCOLLECTION
- 还有其他的多维数据类型。。。

​     可以采用以下语法创建一张基于Geography数据类型的POINT（点类型）的表：

```sql
CREATE TABLE global_points (
    id SERIAL PRIMARY KEY,
    name VARCHAR(64),
    geog GEOGRAPHY(POINT,4326)
);
```

​    然后就可以往该表插入数据：

```sql
INSERT INTO global_points (name, geog) VALUES ('Town', 'SRID=4326;POINT(-110 30)');
INSERT INTO global_points (name, geog) VALUES ('Forest', 'SRID=4326;POINT(-109 29)');
INSERT INTO global_points (name, geog) VALUES ('London', 'SRID=4326;POINT(0 49)');
```

​    还可以对geog列建立空间索引：

```sql
CREATE INDEX global_points_gix ON global_points USING GIST ( geog );
```

​    虽然语法和geometry类型建立索引一样，但是geometry类型建立的是基于平面索引，而这里建立的是基于球面的索引。

# 二、Geometry与Geography的比较

​    可以通过以下链接查看Geometry与Geography的详细比较：

[Geometry与Geography的详细比较](http://postgis.net/docs/manual-3.0/PostGIS_Special_Functions_Index.html#PostGIS_TypeFunctionMatrix)

# 三、Geography支持的空间函数

​      可以查看通过以下链接查看Geography支持的空间函数：

[Geography支持的空间函数](http://postgis.net/docs/manual-3.0/PostGIS_Special_Functions_Index.html#PostGIS_GeographyFunctions)