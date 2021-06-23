- [PostGIS使用2：如何利用PostGIS正确计算距离和面积](https://www.jianshu.com/p/be5049ad8884)
- [PostGIS使用3：PostGIS功能简介（1.概要+空间数据类型）](https://www.jianshu.com/p/9ed3b4dcd32a)



# 一、PostGIS是啥

> PostGIS是在对象关系型数据库PostgreSQL上增加了存储管理空间数据的能力的开源GIS数据库。

> GIS数据库，也叫空间数据库（Spatial database），是负责存储处理位置（空间）数据和非空间数据的数据库。

任何**真实世界的对象**（也叫物体）都能表示成数据库中的对象，其中一部分**真实世界的对象**会跟地理位置有关，我们管他们叫**“地物”（Features）**，对**地物**的存储和处理就是空间（GIS）数据库的基本功能。

在O2O、LBS、物联网、新零售等这些越来越倾向线下业务的互联网概念中，本质都绕不开对**“地物”**的处理，不管是动的对象（人、机器人、动物、汽车、自行车、无人机、地铁等）还是不动的对象（商店、公司、广场、水系、停车点等），这些都是**“地物”**，都需要做存储和运算，互联网开发人员如果掌握了PostGIS，就相当于掌握了处理**“地物”**的利器，多了一把在上述行业中披荆斩棘的斧子。

**PostGIS**是一群国外GISer在**PostgreSQL**实现的插件，**PostgreSQL**是著名的老牌数据库，特点是功能全，什么都能干（当数仓、做业务库、当MQ、做MPP等），社区活跃，业内对标的是oracle。开源后使用率年年上升，现压mongodb稳稳全球排名第四，而且还在上升中（mongo几乎没有增长）。

PostgresQL的标志是只大象。

**PostGIS**强大的空间数据库功能依托于**PostgreSQL**的两个**重要特性**：

## 1.1 Geometry对象

Geometry（几何对象类型）是PG的一个基本存储类型，PostGIS的空间数据都会以Geometry的形式存储在PostgreSQL里，本质是个二进制对象。

## 1.2 Gist索引

Gist(Generalized Search Tree)，通用搜索树，本身是索引框架，一种平衡树结构的访问方法，用户可以针对不同场景基于Gist定制自己的索引。
 Gist的索引建立依赖于聚合运算（聚合运算的实现接口PG也开放出来了，用户甚至可以自己写），适合多维数据类型和集合数据类型。
 PG针对Geometry对象已经写好了一套Gist索引，用于空间检索。

**PostGIS是“踩在巨人肩膀上”实现的空间数据存储与检索，自身又基于OGC（Open Geospatial Consortium，开放地理协会）的规范实现了专业的功能和合理的类型扩展，加上最终的开源，让他成为了现金最流行的空间数据库。**

# 二、PostGIS都能做啥

这个话题其实就是“一个好的空间数据库都能做啥？”
 答案是：能做
 **1.空间数据存储
 2.空间数据输出
 3.空间数据访问
 4.空间数据编辑
 5.空间数据处理
 6.空间数据关系判断和测量
 7.空间拓扑实现**

## 2.1 空间数据存储

PostGIS基于OGC的“Simple Feature for Specification for SQL”规范，在Geometry对象上实现了一系列的GIS Object（地物对象），使用了OGC推荐的WKT（Well-Known Text）和WKB（Well-Known Binary）格式进行描述，大幅增加了易用性，例如WKT的7个基本类型：

```plsql
点：
POINT(0 0)
线：
LINESTRING(0 0,1 1,1 2)
面：
POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))
多点：
MULTIPOINT((0 0),(1 2))
多线：
MULTILINESTRING((0 0,1 1,1 2),(2 3,3 2,5 4))
多面：
MULTIPOLYGON(((0 0,4 0,4 4,0 4,0 0),(1 1,2 1,2 2,1 2,1 1)), ((-1 -1,-1 -2,-2 -2,-2 -1,-1 -1)))
几何集合：
GEOMETRYCOLLECTION(POINT(2 3),LINESTRING(2 3,3 4))
```

WKB是WKT的二进制版本，还有一个TWKB是WKB的压缩版

PostGIS自身又在WKT和WKB基础上扩展实现了EWKT和EWKB来满足更复杂的场景需求，例如EWKT的类型：

```plsql
POINT(0 0 0) -- XYZ
SRID=32632;POINT(0 0) -- XY with SRID
POINTM(0 0 0) -- XYM
POINT(0 0 0 0) -- XYZM
SRID=4326;MULTIPOINTM(0 0 0,1 2 1) -- XYM with SRID
MULTILINESTRING((0 0 0,1 1 0,1 2 1),(2 3 1,3 2 1,5 4 1))
POLYGON((0 0 0,4 0 0,4 4 0,0 4 0,0 0 0),(1 1 0,2 1 0,2 2 0,1 2 0,1 1 0))
MULTIPOLYGON(((0 0 0,4 0 0,4 4 0,0 4 0,0 0 0),(1 1 0,2 1 0,2 2 0,1 2 0,1 1 0)),((-1 -1 0,-1 -2 0,-2 -2 0,-2 -1 0,-1 -1 0)))
GEOMETRYCOLLECTIONM( POINTM(2 3 9), LINESTRINGM(2 3 4, 3 4 5) )
MULTICURVE( (0 0, 5 5), CIRCULARSTRING(4 0, 4 4, 8 4) )
POLYHEDRALSURFACE( ((0 0 0, 0 0 1, 0 1 1, 0 1 0, 0 0 0)), ((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0)), ((0 0 0, 1 0 0, 1 0 1, 0 0 1, 0 0 0)), ((1 1 0, 1 1 1, 1 0 1, 1 0 0, 1 1 0)), ((0 1 0, 0 1 1, 1 1 1, 1 1 0, 0 1 0)), ((0 0 1, 1 0 1, 1 1 1, 0 1 1, 0 0 1)) )
TRIANGLE ((0 0, 0 9, 9 0, 0 0))
TIN( ((0 0 0, 0 0 1, 0 1 0, 0 0 0)), ((0 0 0, 0 1 0, 1 1 0, 0 0 0)) )
```

POINT(0 0 0)可以存三维坐标，POINTM(0 0 0) 可以存额外自定义维度，例如时间（很适合存轨迹点）等。

下面用个例子说明EWKT跟Geometry的关系：

```plsql
SELECT 'SRID=4;POINT(0 0)'::geometry;
```

::geometry的作用是把格式强制转换成geometry，EWKT的文字描述就被转成了Geometry格式的二进制对象，语句执行结果：

```plsql
geometry
----------------------------------------------------
01010000200400000000000000000000000000000000000000
(1 row)
```

上述的空间数据在一张表里的表现形式与其他类型数据没有差别，只是一个字段而已，字段类型是Geometry。



<font color='red'>PostGIS还有一种额外的空间数据类型**Geography（地理对象）**，跟Geometry的区别是Geography的空间运算都是基于EPSG:4326大地坐标系下的椭球面运算（Geometry的运算使用的都是笛卡尔平面直角坐标系），非常实用但会有额外开销，个人推荐存储时用Geometry，做距离面积等测量运算的时候在转换为Geography。</font>

可以看出PostGIS有着丰富的GIS Object（地物）描述类型，和拥抱OGC标准的数据结构，他也是系统化学习GIS的好工具。