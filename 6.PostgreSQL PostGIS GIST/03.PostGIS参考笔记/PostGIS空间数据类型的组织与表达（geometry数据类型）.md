# PostGIS空间数据类型的组织与表达（geometry数据类型）

作者：不睡觉的怪叔叔

地址：https://zhuanlan.zhihu.com/p/103694713



# 一、Simple Features Interface Standard（SFS）

 SFS是OGC用于描述地理要素几何对象的模型的一个规范。

## 1.1、SFS定义的几何对象模型

![image-20210625155410910](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210625155410910.png)

## 1.2、SFS定义的几何对象的方法

![image-20210625155418671](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210625155418671.png)

# 二、SQL Multimedia and Application Packages（SQL/MM）

 SQL/MM是ISO定义的用于描述地理要素几何对象模型的规范。

## 2.1、SFS和SQL/MM几何类型定义的对应关系

![image-20210625155217086](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210625155217086.png)

SQL/MM采用“ST_”作为前缀进行命名。

## 2.2、SQL/MM定义的几何对象模型

![img](https://pic3.zhimg.com/80/v2-b53b8c69fa10946150aeea0c0caf8fd2_720w.jpg)

​    SQL/MM的几何对象模型与SFS的几何对象模型非常类似。

# 三、PostGIS的二维几何对象模型

## 3.1、PostGIS二维几何对象模型的组织

​    PostGIS在二维几何对象方面支持OGC"**Simple Features for SQL**"规范中指定的所有对象和函数，同时也支持"**SQL/MM**"规范的几何对象。

## 3.2、PostGIS二维几何对象模型的表达

​    PostGIS在内部将二维几何对象数据编码为二进制数据（也就是geometry类型）进行存储。 

​    PostGIS同时也最经常使用以下两种表达方式对二维几何对象模型进行表达：

- **Well-Known Text (WKT)**
- **Well-Known Binary (WKB)**

 WKT形式的数据通常是这样的：

- POINT(0 0)
- LINESTRING(0 0,1 1,1 2)
- POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))
- MULTIPOINT((0 0),(1 2))
- MULTILINESTRING((0 0,1 1,1 2),(2 3,3 2,5 4))
- MULTIPOLYGON(((0 0,4 0,4 4,0 4,0 0),(1 1,2 1,2 2,1 2,1 1)), ((-1 -1,-1 -2,-2 -2,-2 -1,-1 -1)))
- GEOMETRYCOLLECTION(POINT(2 3),LINESTRING(2 3,3 4))

   WKB形式的数据就是一串二进制位串。

   可以通过以下函数将WKT和WKB的数据转换为geometry类型数据：

- **ST_GeometryFromText(text WKT, SRID)**
- **ST_GeomFromWKB(bytea WKB, SRID)**

​    可以通过以下函数将geometry类型数据转换为WKT和WKB的数据：

- **ST_AsBinary(geometry)**
- **ST_AsText(geometry)**

​    WKB形式的数据可以借助encode()函数将其转换为十六进制字符串的格式（方便观察）：

```sql
SELECT encode(
  	ST_AsBinary(ST_GeometryFromText('POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))')),
 	'hex');
```

​    结果：

```text
"01030000000200000005000000000000000000000000000000000000000000000000001040000000000000000000000000000010400000000000001040000000000000000000000000000010400000000000000000000000000000000005000000000000000000f03f000000000000f03f0000000000000040000000000000f03f00000000000000400000000000000040000000000000f03f0000000000000040000000000000f03f0000（00000000f03f" 
```

# 四、PostGIS的三维几何对象模型

## 4.1、PostGIS三维几何对象模型的组织

 由于SFS和SQL/MM规范都只定义了二维几何对象模型，所以PostGIS自行扩展了以下三维、四维几何类型：

- 点    ——    **PointZ**、**PointM**和**PointZM**
- 线串    ——    **LinestringZ**、**LinestringM**和**LinestringZM**
- 多边形    ——    **PolygonZ**、**PolygonM**和**PolygonZM**
- 等等

## 4.2、PostGIS三维几何对象模型的表达

  PostGIS同样在内部将三、四维几何对象数据编码为二进制数据（也就是geometry类型）进行存储。 

​     PostGIS使用以下两种表达方式对三、四维几何对象模型进行表达：

- **Extended Well-Known Text (EWKT)**
- **Extended Well-Known Binary (EWKB)**

  EWKT形式的数据通常是这样的：

- POINT(0 0 0) -- XYZ
- SRID=32632;POINT(0 0) -- XY with SRID
- POINTM(0 0 0) -- XYM
- POINT(0 0 0 0) -- XYZM
- SRID=4326;MULTIPOINTM(0 0 0,1 2 1) -- XYM with SRID
- MULTILINESTRING((0 0 0,1 1 0,1 2 1),(2 3 1,3 2 1,5 4 1))
- POLYGON((0 0 0,4 0 0,4 4 0,0 4 0,0 0 0),(1 1 0,2 1 0,2 2 0,1 2 0,1 1 0))
- MULTIPOLYGON(((0 0 0,4 0 0,4 4 0,0 4 0,0 0 0),(1 1 0,2 1 0,2 2 0,1 2 0,1 1 0)),((-1 -1 0,-1 -2 0,-2 -2 0,-2 -1 0,-1 -1 0)))
- GEOMETRYCOLLECTIONM( POINTM(2 3 9), LINESTRINGM(2 3 4, 3 4 5) )
- MULTICURVE( (0 0, 5 5), CIRCULARSTRING(4 0, 4 4, 8 4) )
- POLYHEDRALSURFACE( ((0 0 0, 0 0 1, 0 1 1, 0 1 0, 0 0 0)), ((0 0 0, 0 1 0, 1 1 0, 1 0 0, 0 0 0)), ((0 0 0, 1 0 0, 1 0 1, 0 0 1, 0 0 0)), ((1 1 0, 1 1 1, 1 0 1, 1 0  0, 1 1 0)), ((0 1 0, 0 1 1, 1 1 1, 1 1 0, 0 1 0)), ((0 0 1, 1 0 1, 1 1  1, 0 1 1, 0 0 1)) )
- TRIANGLE ((0 0, 0 9, 9 0, 0 0))
- TIN( ((0 0 0, 0 0 1, 0 1 0, 0 0 0)), ((0 0 0, 0 1 0, 1 1 0, 0 0 0)) )

 EWKT形式的数据与WKT形式的数据相比除了多了维度信息以外，还多了空间参考信息（比如"SRID=32632;POINT(0 0)"）。也就是说PostGIS的多维数据类型的数据可以内嵌空间参考信息。

  可以通过以下函数将WKT和WKB的数据转换为geometry类型数据：

- **ST_GeomFromEWKB(bytea EWKB)** 
- **ST_GeomFromEWKT(text EWKT)**

​    可以通过以下函数将geometry类型数据转换为WKT和WKB的数据：

- **ST_AsEWKB(geometry)**
- **ST_AsEWKT(geometry)**