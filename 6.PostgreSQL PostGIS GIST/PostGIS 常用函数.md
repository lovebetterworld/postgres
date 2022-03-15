- [PostGIS 常用函数](https://www.cnblogs.com/Jason1019/p/13474474.html)

## 一、OGC标准函数

### 1.1 管理函数

```plsql
--添加几何字段
AddGeometryColumn(, , , , , )

--删除几何字段
DropGeometryColumn(, , )

--检查数据库几何字段并在geometry_columns中归档
Probe_Geometry_Columns()

--给几何对象设置空间参考（在通过一个范围做空间查询时常用）
ST_SetSRID(geometry, integer)
```

### 1.2 几何对象关系函数

```plsql
--获取两个几何对象间的距离
ST_Distance(geometry, geometry)

--如果两个几何对象间距离在给定值范围内，则返回TRUE
ST_DWithin(geometry, geometry, float)

--判断两个几何对象是否相等（比如LINESTRING(0 0, 2 2)和LINESTRING(0 0, 1 1, 2 2)是相同的几何对象）
ST_Equals(geometry, geometry)

--判断两个几何对象是否分离 
ST_Disjoint(geometry, geometry)

--判断两个几何对象是否相交 
ST_Intersects(geometry, geometry)

--判断两个几何对象的边缘是否接触 
ST_Touches(geometry, geometry)

--判断两个几何对象是否互相穿过 
ST_Crosses(geometry, geometry)

--判断A是否被B包含 
ST_Within(geometry A, geometry B)

--判断两个几何对象是否是重叠
 ST_Overlaps(geometry, geometry)

--判断A是否包含B 
ST_Contains(geometry A, geometry B)

--判断A是否覆盖 B 
ST_Covers(geometry A, geometry B)

--判断A是否被B所覆盖 
ST_CoveredBy(geometry A, geometry B)

--通过DE-9IM 矩阵判断两个几何对象的关系是否成立 
ST_Relate(geometry, geometry, intersectionPatternMatrix)

--获得两个几何对象的关系（DE-9IM矩阵）
ST_Relate(geometry, geometry)
```

### 1.3 几何对象处理函数

```plsql
--修复错误图形 返回Geometry对象
ST_Buffer(geom, 0.0)
ST_MakeValid(geom)

--获取几何对象的中心
ST_Centroid(geometry)

--面积量测
ST_Area(geometry)

--长度量测
ST_Length(geometry)

--返回曲面上的一个点
ST_PointOnSurface(geometry)

--获取边界
ST_Boundary(geometry)

--获取缓冲后的几何对象
ST_Buffer(geometry, double, [integer])

--获取多几何对象的外接对象
ST_ConvexHull(geometry)

--获取两个几何对象相交的部分
ST_Intersection(geometry, geometry)

--将经度小于0的值加360使所有经度值在0-360间
ST_Shift_Longitude(geometry)

--获取两个几何对象不相交的部分（A、B可互换）
ST_SymDifference(geometry A, geometry B)

--从A去除和B相交的部分后返回
ST_Difference(geometry A, geometry B)

--返回两个几何对象的合并结果
ST_Union(geometry, geometry)

--返回一系列几何对象的合并结果
ST_Union(geometry set)

--用较少的内存和较长的时间完成合并操作，结果和ST_Union相同
ST_MemUnion(geometry set)
```

### 1.4 几何对象存取函数

```plsql
--获取几何对象的WKT描述 
ST_AsText(geometry)

--获取几何对象的WKB描述 
ST_AsBinary(geometry)

--获取几何对象的空间参考ID 
ST_SRID(geometry)

--获取几何对象的维数 
ST_Dimension(geometry)

--获取几何对象的边界范围 
ST_Envelope(geometry)

--判断几何对象是否为空 
ST_IsEmpty(geometry)

--判断几何对象是否不包含特殊点（比如自相交） 
ST_IsSimple(geometry)

--判断几何对象是否闭合 
ST_IsClosed(geometry)

--判断曲线是否闭合并且不包含特殊点 
ST_IsRing(geometry)

--获取多几何对象中的对象个数 
ST_NumGeometries(geometry)

--获取多几何对象中第N个对象 
ST_GeometryN(geometry,int)

--获取几何对象中的点个数 
ST_NumPoints(geometry)

--获取几何对象的第N个点 
ST_PointN(geometry,integer)

--获取多边形的外边缘 
ST_ExteriorRing(geometry)

--获取多边形内边界个数 
ST_NumInteriorRings(geometry)

--同上 
ST_NumInteriorRing(geometry)

--获取多边形的第N个内边界 
ST_InteriorRingN(geometry,integer)

--获取线的终点 
ST_EndPoint(geometry)

--获取线的起始点 
ST_StartPoint(geometry)

--获取几何对象的类型 
GeometryType(geometry)

--类似上，但是不检查M值，即POINTM对象会被判断为point 
ST_GeometryType(geometry)

--获取点的X坐标 
ST_X(geometry)

--获取点的Y坐标 
ST_Y(geometry)

--获取点的Z坐标 
ST_Z(geometry)

--获取点的M值 
ST_M(geometry)
```

### 1.5 几何对象构造函数

```plsql
ST_PointFromText(text,[])ST_GeomFromText(text)
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

## 二、扩展函数

### 2.1 管理函数

```plsql
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

### 2.2 几何操作符

```plsql
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

### 2.3 几何量测函数

```plsql
--量测面积 
ST_Area(geometry)
--根据经纬度点计算在地球曲面上的距离，单位米，地球半径取值6370986米 
ST_distance_sphere(point, point)
--类似上，使用指定的地球椭球参数 
ST_distance_spheroid(point, point, spheroid)
--量测2D对象长度 
ST_length2d(geometry)
--量测3D对象长度 
ST_length3d(geometry)
--根据经纬度对象计算在地球曲面上的长度 
ST_length_spheroid(geometry,spheroid) 
ST_length3d_spheroid(geometry,spheroid)
--量测两个对象间距离 
ST_distance(geometry, geometry)
--量测两条线之间的最大距离 
ST_max_distance(linestring,linestring)
--量测2D对象的周长 
ST_perimeter(geometry) 
ST_perimeter2d(geometry)
--量测3D对象的周长 
ST_perimeter3d(geometry)
--量测两点构成的方位角，单位弧度 
ST_azimuth(geometry, geometry)
```

### 2.4 几何对象输出

```plsql
ST_AsBinary(geometry,{'NDR'|'XDR'})
ST_AsEWKT(geometry)
ST_AsEWKB(geometry, {'NDR'|'XDR'})
ST_AsHEXEWKB(geometry, {'NDR'|'XDR'})
ST_AsSVG(geometry, [rel], [precision])
ST_AsGML([version], geometry, [precision])
ST_AsKML([version], geometry, [precision])
ST_AsGeoJson([version], geometry, [precision], [options])
```

### 2.5 几何对象创建

```plsql
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
ST_Dump(geometry) --可以将mutigeometry 分成单个geometry
ST_DumpRings(geometry)
```

### 2.6 几何对象编辑

```plsql
--给几何对象添加一个边界，会使查询速度加快 
ST_AddBBOX(geometry)

--删除几何对象的边界 
ST_DropBBOX(geometry)

--添加、删除、设置点 
ST_AddPoint(linestring, point, [])
ST_RemovePoint(linestring, offset)
ST_SetPoint(linestring, N, point)

--几何对象类型转换 
ST_Force_collection(geometry)
ST_Force_2d(geometry)
ST_Force_3dz(geometry), ST_Force_3d(geometry),
ST_Force_3dm(geometry)
ST_Force_4d(geometry)
ST_Multi(geometry)

--将几何对象转化到指定空间参考 
ST_Transform(geometry,integer)
--对3D几何对象作仿射变化 
ST_Affine(geometry, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8, float8)

--对2D几何对象作仿射变化 
ST_Affine(geometry, float8, float8, float8, float8, float8, float8)

--对几何对象作偏移 
ST_Translate(geometry, float8, float8, float8)

--对几何对象作缩放 
ST_Scale(geometry, float8, float8, float8)

--对3D几何对象作旋转 
ST_RotateZ(geometry, float8)
ST_RotateX(geometry, float8)
ST_RotateY(geometry, float8)

--对2D对象作偏移和缩放 
ST_TransScale(geometry, float8, float8, float8, float8)

--反转 
ST_Reverse(geometry)

--转化到右手定则 
ST_ForceRHR(geometry)
--参考IsSimple函数

--使用Douglas-Peuker算法 
ST_Simplify(geometry, tolerance)
ST_SimplifyPreserveTopology(geometry, tolerance)

--讲几何对象顶点捕捉到网格 
ST_SnapToGrid(geometry, originX, originY, sizeX, sizeY)
ST_SnapToGrid(geometry, sizeX, sizeY), ST_SnapToGrid(geometry, size)

--第二个参数为点，指定原点坐标 
ST_SnapToGrid(geometry, geometry, sizeX, sizeY, sizeZ, sizeM)

--分段 
ST_Segmentize(geometry, maxlength)

--合并为线 
ST_LineMerge(geometry)
```

### 2.7 线性参考

```plsql
--根据location（0-1）获得该位置的点 
ST_line_interpolate_point(linestring, location)

--获取一段线 
ST_line_substring(linestring, start, end)

--根据点获取location（0-1） 
ST_line_locate_point(LineString, Point)

--根据量测值获得几何对象 
ST_locate_along_measure(geometry, float8)

--根据量测值区间获得几何对象集合 
ST_locate_between_measures(geometry, float8, float8)
```

### 2.8 杂项功能函数

```plsql
--几何对象的摘要 
ST_Summary(geometry)

--几何对象的边界 
ST_box2d(geometry)
ST_box3d(geometry)

--多个几何对象的边界 
ST_extent(geometry set)

0=2d, 1=3dm, 2=3dz, 3=4d 
ST_zmflag(geometry)

--是否包含Bounding Box 
ST_HasBBOX(geometry)

--几何对象的维数：2、3、4 
ST_ndims(geometry)

--子对象的个数 
ST_nrings(geometry)
ST_npoints(geometry)

--对象是否验证成功 
ST_isvalid(geometry)

--扩大几何对象 
ST_expand(geometry, float)

--计算一个空间表的边界范围 
ST_estimated_extent([schema], table, geocolumn)

--获得空间参考 
ST_find_srid(, , )

--几何对象使用的内存大小，单位byte 
ST_mem_size(geometry)

--点是否在圆上 
ST_point_inside_circle(,,,)

--获取边界的X、Y、Z 
ST_XMin(box3d)
ST_YMin(box3d)
ST_ZMin(box3d)
ST_XMax(box3d)
ST_YMax(box3d)
ST_ZMax(box3d)

--构造一个几何对象的数组 
ST_Accum(geometry set)
```

### 2.9 长事务支持

```plsql
--启用/关闭长事务支持，重复调用无副作用 
EnableLongTransactions()
DisableLongTransactions()

--检查对行的update和delete操作是否已授权 
CheckAuth([],, )

--锁定行 
LockRow([],, , , [])

--解锁行 
UnlockRows()

--在当前事务中添加授权ID 
AddAuth()
```
