- [Geometry——PostgreSQL+PostGIS的使用 - 怪猫佐良 - 博客园 (cnblogs.com)](https://www.cnblogs.com/oddcat/articles/10722065.html)

## 一、PostGIS介绍

PostGIS是对象关系型数据库系统PostgreSQL的一个扩展，PostGIS提供如下空间信息服务功能:空间对象、空间索引、空间操作函数和空间操作符。

同时，PostGIS遵循OpenGIS的规范。

## 二、 PostGIS中的几何类型

PostGIS支持所有OGC(Open Geospatial Consortium) 规范的“Simple Features”类型，同时在此基础上扩展了对3DZ、3DM、4D坐标的支持。

### *1. OGC的WKB和WKT格式*

OGC定义了两种描述几何对象的格式，分别是WKB（Well-Known Binary）和WKT（Well-Known Text）。

在SQL语句中，用以下的方式可以使用WKT格式定义几何对象：

```
POINT(0 0)    　　　　　　//点
LINESTRING(0 0,1 1,1 2) //线
POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))　　//面
MULTIPOINT(0 0,1 2)　　  //多点
MULTILINESTRING((0 0,1 1,1 2),(2 3,3 2,5 4))　　　　　　　　 //多线
MULTIPOLYGON(((0 0,4 0,4 4,0 4,0 0),(1 1,2 1,2 2,1 2,1 1)), ((-1 -1,-1 -2,-2 -2,-2 -1,-1 -1)))//多面
GEOMETRYCOLLECTION(POINT(2 3),LINESTRING((2 3,3 4)))　　　　//几何集合
```

以下语句可以使用WKT格式插入一个点要素到一个表中，其中用到的GeomFromText等函数在后面会有详细介绍：

```
INSERT INTO table ( SHAPE, NAME )
VALUES ( GeomFromText('POINT(116.39 39.9)', 4326), '北京');
```

### *2. EWKT、EWKB和Canonical格式*

EWKT和EWKB相比OGC，WKT和WKB格式主要的扩展有3DZ、3DM、4D坐标和内嵌空间参考支持。

以下以EWKT语句定义了一些几何对象：

```
POINT(0 0 0)                     　　//3D点
SRID=32632;POINT(0 0)     　　　　　　//内嵌空间参考的点
POINTM(0 0 0)                    　　//带M值的点
POINT(0 0 0 0)                   　　//带M值的3D点
SRID=4326;MULTIPOINTM(0 0 0,1 2 1)  //内嵌空间参考的带M值的多点     
```

以下语句可以使用EWKT格式插入一个点要素到一个表中：

```
INSERT INTO table ( SHAPE, NAME )
VALUES ( GeomFromEWKT('SRID=4326;POINTM(116.39 39.9 10)'), '北京' )
```

Canonical格式是16进制编码的几何对象，直接用SQL语句查询出来的就是这种格式。

### *3. SQL-MM格式*

SQL-MM格式定义了一些插值曲线，这些插值曲线和EWKT有点类似，也支持3DZ、3DM、4D坐标，但是不支持嵌入空间参考。

以下以SQL-MM语句定义了一些插值几何对象：

```
CIRCULARSTRING(0 0, 1 1, 1 0) 　　　　　　　　　　　　　　　　//插值圆弧
COMPOUNDCURVE(CIRCULARSTRING(0 0, 1 1, 1 0),(1 0, 0 1))  //插值复合曲线
CURVEPOLYGON(CIRCULARSTRING(0 0, 4 0, 4 4, 0 4, 0 0),(1 1, 3 3, 3 1, 1 1)) //曲线多边形
MULTICURVE((0 0, 5 5),CIRCULARSTRING(4 0, 4 4, 8 4)) 　　 //多曲线
MULTISURFACE(CURVEPOLYGON(CIRCULARSTRING(0 0, 4 0, 4 4, 0 4, 0 0),(1 1, 3 3, 3 1, 1 1)),((10 10, 14 12, 11 10, 10 10),(11 11, 11.5 11, 11 11.5, 11 11))) //多曲面
```

------

## 三、 PostGIS中空间信息处理的实现

### *1. spatial_ref_sys表*

在基于PostGIS模板创建的数据库的public模式下，有一个spatial_ref_sys表，它存放的是OGC规范的空间参考。我们取我们最熟悉的4326参考看一下：

它的srid存放的就是空间参考的Well-Known ID，对这个空间参考的定义主要包括两个字段，srtext存放的是以字符串描述的空间参考，proj4text存放的则是以字符串描述的PROJ.4 投影定义（PostGIS使用PROJ.4实现投影）。

4326空间参考的srtext内容：

```
GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],TOWGS84[0,0,0,0,0,0,0],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],AUTHORITY["EPSG","4326"]]
```

4326空间参考的proj4text内容：

```
+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs
```

### *2. geometry_columns表*

geometry_columns表存放了当前数据库中所有几何字段的信息，比如我当前的库里面有两个空间表，在geometry_columns表中就可以找到这两个空间表中几何字段的定义：

其中f_table_schema字段表示的是空间表所在的模式，f_table_name字段表示的是空间表的表名，f_geometry_column字段表示的是该空间表中几何字段的名称，srid字段表示的是该空间表的空间参考。

### *3. 在PostGIS中创建一个空间表*

在PostGIS中创建一个包含几何字段的空间表分为2步：第一步创建一个一般表，第二步给这个表添加几何字段。

以下先在test模式下创建一个名为cities的一般表：

```
create table test.cities (id int4, name varchar(20))
```

再给cities添加一个名为shape的几何字段（二维点）：

```
select AddGeometryColumn('test', 'cities', 'shape', 4326, 'POINT', 2)
```

### *4. PostGIS对几何信息的检查*

PostGIS可以检查几何信息的正确性，这主要是通过IsValid函数实现的。
以下语句分辨检查了2个几何对象的正确性，显然，(0, 0)点和(1,1)点可以构成一条线，但是(0, 0)点和(0, 0)点则不能构成，这个语句执行以后的得出的结果是TRUE,FALSE。

```
select IsValid('LINESTRING(0 0, 1 1)'), IsValid('LINESTRING(0 0,0 0)')
```

默认PostGIS并不会使用IsValid函数检查用户插入的新数据，因为这会消耗较多的CPU资源（特别是复杂的几何对象）。当你需要使用这个功能的时候，你可以使用以下语句为表新建一个约束：

```
ALTER TABLE cities
ADD CONSTRAINT geometry_valid
CHECK (IsValid(shape))
```

这时当我们往这个表试图插入一个错误的空间对象的时候，会得到一个错误：

```
INSERT INTO test.cities ( shape, name )
VALUES ( GeomFromText('LINESTRING(0 0,0 0)', 4326), '北京');

ERROR: new row for relation "cities" violates check constraint "geometry_valid"
SQL 状态: 23514
```

### *5. PostGIS中的空间索引*

数据库对多维数据的存取有两种索引方案，R-Tree和GiST（Generalized Search Tree），在PostgreSQL中的GiST比R-Tree的健壮性更好，因此PostGIS对空间数据的索引一般采用GiST实现。

以下的语句给sde模式中的cities表添加了一个空间索引shape_index_cities，在pgAdmin中也可以通过图形界面完成相同的功能。

```
CREATE INDEX shape_index_cities
ON sde.cities
USING gist
(shape);
```

另外要注意的是，空间索引只有在进行基于边界范围的查询时才起作用，比如“&&”操作。

------

## 四、 PostGIS中的常用函数 


首先需要说明一下，这里许多函数是以ST_[X]yyy形式命名的，事实上很多函数也可以通过xyyy的形式访问，在PostGIS的函数库中我们可以看到这两种函数定义完全一样。

### *1. OGC标准函数* 

管理函数： 

```
添加几何字段 AddGeometryColumn(, , , , , )
删除几何字段 DropGeometryColumn(, , )
检查数据库几何字段并在geometry_columns中归档 Probe_Geometry_Columns()
给几何对象设置空间参考（在通过一个范围做空间查询时常用） ST_SetSRID(geometry, integer)
```

几何对象关系函数 ：

```
获取两个几何对象间的距离 ST_Distance(geometry, geometry)
如果两个几何对象间距离在给定值范围内，则返回TRUE ST_DWithin(geometry, geometry, float)
判断两个几何对象是否相等（比如LINESTRING(0 0, 2 2)和LINESTRING(0 0, 1 1, 2 2)是相同的几何对象） ST_Equals(geometry, geometry)
判断两个几何对象是否分离 ST_Disjoint(geometry, geometry)
判断两个几何对象是否相交 ST_Intersects(geometry, geometry)
判断两个几何对象的边缘是否接触 ST_Touches(geometry, geometry)
判断两个几何对象是否互相穿过 ST_Crosses(geometry, geometry)
判断A是否被B包含 ST_Within(geometry A, geometry B)
判断两个几何对象是否是重叠 ST_Overlaps(geometry, geometry)
判断A是否包含B ST_Contains(geometry A, geometry B)
判断A是否覆盖 B ST_Covers(geometry A, geometry B)
判断A是否被B所覆盖 ST_CoveredBy(geometry A, geometry B)
通过DE-9IM 矩阵判断两个几何对象的关系是否成立 ST_Relate(geometry, geometry, intersectionPatternMatrix)
获得两个几何对象的关系（DE-9IM矩阵） ST_Relate(geometry, geometry)
```

几何对象处理函数： 

```
获取几何对象的中心 ST_Centroid(geometry)
面积量测 ST_Area(geometry)
长度量测 ST_Length(geometry)
返回曲面上的一个点 ST_PointOnSurface(geometry)
获取边界 ST_Boundary(geometry)
获取缓冲后的几何对象 ST_Buffer(geometry, double, [integer])
获取多几何对象的外接对象 ST_ConvexHull(geometry)
获取两个几何对象相交的部分 ST_Intersection(geometry, geometry)
将经度小于0的值加360使所有经度值在0-360间 ST_Shift_Longitude(geometry)
获取两个几何对象不相交的部分（A、B可互换） ST_SymDifference(geometry A, geometry B)
从A去除和B相交的部分后返回 ST_Difference(geometry A, geometry B)
返回两个几何对象的合并结果 ST_Union(geometry, geometry)
返回一系列几何对象的合并结果 ST_Union(geometry set)
用较少的内存和较长的时间完成合并操作，结果和ST_Union相同 ST_MemUnion(geometry set)
几何对象存取函数： 
```

```
获取几何对象的WKT描述 ST_AsText(geometry)
获取几何对象的WKB描述 ST_AsBinary(geometry)
获取几何对象的空间参考ID ST_SRID(geometry)
获取几何对象的维数 ST_Dimension(geometry)
获取几何对象的边界范围 ST_Envelope(geometry)
判断几何对象是否为空 ST_IsEmpty(geometry)
判断几何对象是否不包含特殊点（比如自相交） ST_IsSimple(geometry)
判断几何对象是否闭合 ST_IsClosed(geometry)
判断曲线是否闭合并且不包含特殊点 ST_IsRing(geometry)
获取多几何对象中的对象个数 ST_NumGeometries(geometry)
获取多几何对象中第N个对象 ST_GeometryN(geometry,int)
获取几何对象中的点个数 ST_NumPoints(geometry)
获取几何对象的第N个点 ST_PointN(geometry,integer)
获取多边形的外边缘 ST_ExteriorRing(geometry)
获取多边形内边界个数 ST_NumInteriorRings(geometry)
同上 ST_NumInteriorRing(geometry)
获取多边形的第N个内边界 ST_InteriorRingN(geometry,integer)
获取线的终点 ST_EndPoint(geometry)
获取线的起始点 ST_StartPoint(geometry)
获取几何对象的类型 GeometryType(geometry)
类似上，但是不检查M值，即POINTM对象会被判断为point ST_GeometryType(geometry)
获取点的X坐标 ST_X(geometry)
获取点的Y坐标 ST_Y(geometry)
获取点的Z坐标 ST_Z(geometry)
获取点的M值 ST_M(geometry)
几何对象构造函数 ：
```

```
ST_PointFromText(text,[])
ST_LineFromText(text,[])
ST_LinestringFromText(text,[])
ST_PolyFromText(text,[])
ST_PolygonFromText(text,[])
ST_MPointFromText(text,[])
ST_MLineFromText(text,[])
ST_MPolyFromText(text,[])
ST_GeomCollFromText(text,[])
ST_GeomFromWKB(bytea,[])
ST_GeometryFromWKB(bytea,[])
ST_PointFromWKB(bytea,[])
ST_LineFromWKB(bytea,[])
ST_LinestringFromWKB(bytea,[])
ST_PolyFromWKB(bytea,[])
ST_PolygonFromWKB(bytea,[])
ST_MPointFromWKB(bytea,[])
ST_MLineFromWKB(bytea,[])
ST_MPolyFromWKB(bytea,[])
ST_GeomCollFromWKB(bytea,[])
ST_BdPolyFromText(text WKT, integer SRID)
ST_BdMPolyFromText(text WKT, integer SRID)
```

参考语义：

```
Text：WKT
WKB：WKB
Geom:Geometry
M:Multi
Bd:BuildArea
Coll:Collection ST_GeomFromText(text,[])
```

###  2. PostGIS扩展函数

管理函数：

```
删除一个空间表（包括geometry_columns中的记录） DropGeometryTable([], )
更新空间表的空间参考 UpdateGeometrySRID([], , , )
更新空间表的统计信息 update_geometry_stats([, ])
postgis_version()
postgis_lib_version()
postgis_lib_build_date()
postgis_script_build_date()
postgis_scripts_installed()
postgis_scripts_released()
postgis_geos_version()
postgis_jts_version()
postgis_proj_version()
postgis_uses_stats()
postgis_full_version()
```

参考语义：

```
Geos：GEOS库
Jts：JTS库
Proj：PROJ4库
```

几何操作符：

```
A范围=B范围 A = B
A范围覆盖B范围或A范围在B范围左侧 A &<> B
A范围在B范围左侧 A <<>> B
A范围覆盖B范围或A范围在B范围下方 A &<| B A范围覆盖B范围或A范围在B范围上方 A |&> B
A范围在B范围下方 A <<| B A范围在B范围上方 A |>> B
A=B A ~= B
A范围被B范围包含 A @ B
A范围包含B范围 A ~ B
A范围覆盖B范围 A && B
```

几何量测函数：

```
量测面积 ST_Area(geometry)
根据经纬度点计算在地球曲面上的距离，单位米，地球半径取值6370986米 ST_distance_sphere(point, point)
类似上，使用指定的地球椭球参数 ST_distance_spheroid(point, point, spheroid)
量测2D对象长度 ST_length2d(geometry)
量测3D对象长度 ST_length3d(geometry)
根据经纬度对象计算在地球曲面上的长度 ST_length_spheroid(geometry,spheroid)
ST_length3d_spheroid(geometry,spheroid)
量测两个对象间距离 ST_distance(geometry, geometry)
量测两条线之间的最大距离 ST_max_distance(linestring,linestring)
量测2D对象的周长 ST_perimeter(geometry)
ST_perimeter2d(geometry)
量测3D对象的周长 ST_perimeter3d(geometry)
量测两点构成的方位角，单位弧度 ST_azimuth(geometry, geometry)
```

几何对象输出： 

```
ST_AsBinary(geometry,{'NDR'|'XDR'})
ST_AsEWKT(geometry)
ST_AsEWKB(geometry, {'NDR'|'XDR'})
ST_AsHEXEWKB(geometry, {'NDR'|'XDR'})
ST_AsSVG(geometry, [rel], [precision])
ST_AsGML([version], geometry, [precision])
ST_AsKML([version], geometry, [precision])
ST_AsGeoJson([version], geometry, [precision], [options])
```

参考语义：

```
NDR：Little Endian
XDR：big-endian
HEXEWKB：Canonical
SVG：SVG 格式
GML：GML 格式
KML：KML 格式
GeoJson：GeoJson 格式
```

几何对象创建：

```
ST_GeomFromEWKB(bytea)
ST_MakePoint(, , [], [])
ST_MakePointM(, , )
ST_MakeBox2D(, )
ST_MakeBox3D(, )
ST_MakeLine(geometry set)
ST_MakeLine(geometry, geometry)
ST_LineFromMultiPoint(multipoint)
ST_MakePolygon(linestring, [linestring[]])
ST_BuildArea(geometry)
ST_Polygonize(geometry set)
ST_Collect(geometry set)
ST_Collect(geometry, geometry)
ST_Dump(geometry)
ST_DumpRings(geometry)
```

参考语义：

```
Dump：转储 ST_GeomFromEWKT(text)
```

几何对象编辑：

```
给几何对象添加一个边界，会使查询速度加快 ST_AddBBOX(geometry)
删除几何对象的边界 ST_DropBBOX(geometry)
添加、删除、设置点 ST_AddPoint(linestring, point, [])
ST_RemovePoint(linestring, offset)
ST_SetPoint(linestring, N, point)
几何对象类型转换 ST_Force_collection(geometry)
ST_Force_2d(geometry)
ST_Force_3dz(geometry), ST_Force_3d(geometry),
ST_Force_3dm(geometry)
ST_Force_4d(geometry)
ST_Multi(geometry)
将几何对象转化到指定空间参考 ST_Transform(geometry,integer)
对3D几何对象作仿射变化 ST_Affine(geometry, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8)
对2D几何对象作仿射变化 ST_Affine(geometry, float8, float8, float8, float8, float8, float8)
对几何对象作偏移 ST_Translate(geometry, float8, float8, float8)
对几何对象作缩放 ST_Scale(geometry, float8, float8, float8)
对3D几何对象作旋转 ST_RotateZ(geometry, float8)
ST_RotateX(geometry, float8)
ST_RotateY(geometry, float8)
对2D对象作偏移和缩放 ST_TransScale(geometry, float8, float8, float8, float8)
反转 ST_Reverse(geometry)
转化到右手定则 ST_ForceRHR(geometry)
参考IsSimple函数
使用Douglas-Peuker算法 ST_Simplify(geometry, tolerance)
ST_SimplifyPreserveTopology(geometry, tolerance)
讲几何对象顶点捕捉到网格 ST_SnapToGrid(geometry, originX, originY, sizeX, sizeY)
ST_SnapToGrid(geometry, sizeX, sizeY), ST_SnapToGrid(geometry, size)
第二个参数为点，指定原点坐标 ST_SnapToGrid(geometry, geometry, sizeX, sizeY, sizeZ, sizeM)
分段 ST_Segmentize(geometry, maxlength)
合并为线 ST_LineMerge(geometry)
```

线性参考：

```
根据location（0-1）获得该位置的点 ST_line_interpolate_point(linestring, location)
获取一段线 ST_line_substring(linestring, start, end)
根据点获取location（0-1） ST_line_locate_point(LineString, Point)
根据量测值获得几何对象 ST_locate_along_measure(geometry, float8)
根据量测值区间获得几何对象集合 ST_locate_between_measures(geometry, float8, float8)
```

杂项功能函数： 

```
几何对象的摘要 ST_Summary(geometry)
几何对象的边界 ST_box2d(geometry)
ST_box3d(geometry)
多个几何对象的边界 ST_extent(geometry set)
0=2d, 1=3dm, 2=3dz, 3=4d ST_zmflag(geometry)
是否包含Bounding Box ST_HasBBOX(geometry)
几何对象的维数：2、3、4 ST_ndims(geometry)
子对象的个数 ST_nrings(geometry)
ST_npoints(geometry)
对象是否验证成功 ST_isvalid(geometry)
扩大几何对象 ST_expand(geometry, float)
计算一个空间表的边界范围 ST_estimated_extent([schema], table, geocolumn)
获得空间参考 ST_find_srid(, , )
几何对象使用的内存大小，单位byte ST_mem_size(geometry)
点是否在圆上 ST_point_inside_circle(,,,)
获取边界的X、Y、Z ST_XMin(box3d)
ST_YMin(box3d)
ST_ZMin(box3d)
ST_XMax(box3d)
ST_YMax(box3d)
ST_ZMax(box3d)
构造一个几何对象的数组 ST_Accum(geometry set)
```

长事务支持： 

```
启用/关闭长事务支持，重复调用无副作用 EnableLongTransactions()
DisableLongTransactions()
检查对行的update和delete操作是否已授权 CheckAuth([], , )
锁定行 LockRow([], , , , [])
解锁行 UnlockRows()
在当前事务中添加授权ID AddAuth()
```

其它还有SQL-MM和ArcSDE样式的函数支持，可以参考http://postgis.refractions.net/documentation/manual-1.3/ch06.html#id2750611。

## 五、 PostGIS示例

下面我们通过一个简单的Flex应用示例来看一下PostGIS的用法：

假想现在发生了恐怖袭击，导致在一些城市有污染物出现，现在我们要根据污染物和当地风力、风向情况，计算污染扩散范围，针对这些区域及时进行警报和疏散。

首先我们希望获得所有发生污染的城市的当前风速、风向等信息，在我们的PostGIS数据库中有一个空间表保存着这些信息，我们构造这样的SQL语句进行查询：
select *,ST_AsGeoJson(shape) from sde.wind

这里会获取所有风相关的信息，并且附加了以JSON格式返回的几何信息，这有助于我们在Flex中进行解析。如下图是关于风的查询结果：

下面我们希望PostGIS帮助我们实现一些空间分析。我们以污染发生的城市为起点，当地风向为主方向，构造一个30度开角的范围；这个范围将是污染扩散的主要方向，扩散的范围主要和风的强度有关；在构造这个区域以后，为了保险起见，我们在对其进行一定范围的缓冲，最后得到每个污染源可能扩散的范围。我们构造的SQL语句如下：

```
select *,ST_AsGeoJson( ST_Buffer( ST_PolygonFromText( 'POLYGON((' ||ST_X(shape)||' '||ST_Y(shape)||',' ||ST_X(shape)+velocity*cos((direction+15)*PI()/180)/20||' '||ST_Y(shape)+velocity*sin((direction+15)*PI()/180)/20||',' ||ST_X(shape)+velocity*cos((direction-15)*PI()/180)/20||' '||ST_Y(shape)+velocity*sin((direction-15)*PI()/180)/20||',' ||ST_X(shape)||' '||ST_Y(shape)||'))' ) , velocity/50 ) ) from sde.wind
```

------

参考：

1. https://www.cnblogs.com/wuhenke/archive/2010/08/02/1790747.html