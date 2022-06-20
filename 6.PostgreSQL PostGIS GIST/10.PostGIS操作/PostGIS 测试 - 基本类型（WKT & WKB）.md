- [PostGIS 测试 - 基本类型（WKT & WKB）_万里归来少年心的博客-CSDN博客_postgis wkt](https://blog.csdn.net/liyazhen2011/article/details/89165052)

OGC定义了两种描述几何对象的格式，分别是WKB（Well-Known Binary）和WKT（Well-Known Text）。 在[SQL语句](https://so.csdn.net/so/search?q=SQL语句&spm=1001.2101.3001.7020)中，用以下的方式可以使用WKT格式定义几何对象：

| 几何类型 | WKT格式                                                      |
| :------- | :----------------------------------------------------------- |
| 点       | POINT(0 0)                                                   |
| 线       | LINESTRING(0 0,1 1,1 2)                                      |
| 面       | POLYGON((0 0,4 0,4 4,0 4,0 0),(1 1, 2 1, 2 2, 1 2,1 1))      |
| 多点     | MULTIPOINT(0 0,1 2)                                          |
| 多线     | MULTILINESTRING((0 0,1 1,1 2),(2 3,3 2,5 4))                 |
| 多面     | MULTIPOLYGON(((0 0,4 0,4 4,0 4,0 0),(1 1,2 1,2 2,1 2,1 1)), ((-1 -1,-1 -2,-2 -2,-2 -1,-1 -1))) |
| 几何集合 | GEOMETRYCOLLECTION(POINT(2 3),LINESTRING((2 3,3 4)))         |

  本文通过实例演示几何对象的定义。

## 1.建表

```sql
INSERT INTO postgis2d (name, geom) VALUES ('p1',
  ST_GeomFromText('POINT(0 0)', 26910)
);
INSERT INTO postgis2d (name,geom) VALUES ('p2',
  ST_GeomFromText('POINT(5 0)', 26910)
);
INSERT INTO postgis2d (name,geom) VALUES ('p3',
  ST_GeomFromText('POINT(0 10)', 26910)
);
```

## 2.添加GIST索引

```sql
CREATE INDEX postgis2d_gix ON postgis2d USING GIST (geom); 
```

## 3.插入数据

### 3.1 点（POINT）

```sql
INSERT INTO postgis2d (name, geom) VALUES ('p1',
  ST_GeomFromText('POINT(0 0)', 26910)
);
INSERT INTO postgis2d (name,geom) VALUES ('p2',
  ST_GeomFromText('POINT(5 0)', 26910)
);
INSERT INTO postgis2d (name,geom) VALUES ('p3',
  ST_GeomFromText('POINT(0 10)', 26910)
);
```

  或使用多点（MULTIPOINT）

```sql
INSERT INTO postgis2d (name,geom) VALUES ('p3',
  ST_GeomFromText('MULTIPOINT((0 0),(5 0),(0 10))', 26910)
);
```

   QGIS中显示几何如下：

 ![img](https://img-blog.csdnimg.cn/20190409225154824.png)

### 3.2 线（LINESTRING）

```sql
INSERT INTO postgis2d (name, geom) VALUES ('L1',
  ST_GeomFromText('LINESTRING(0 0,1 1,1 2)', 26910)
);
 
INSERT INTO postgis2d (name, geom) VALUES ('L2',
  ST_GeomFromText('LINESTRING(1 0,2 1,2 2)', 26910)
);
```

  或使用多线（MULTILINESTRING）

```sql
INSERT INTO postgis2d (name, geom) VALUES ('L1',
  ST_GeomFromText('MULTILINESTRING((0 0,1 1,1 2),(1 0,2 1,2 2))', 26910)
);
```

  QGIS中显示几何如下：

  ![img](https://img-blog.csdnimg.cn/20190409225845906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWF6aGVuMjAxMQ==,size_16,color_FFFFFF,t_70)

### 3.3 多边形（POLYGON）

```sql
INSERT INTO postgis2d (name, geom) VALUES ('p1',
  ST_GeomFromText('POLYGON((4 0,8 0,8 4,4 0))', 26910)
);
 
INSERT INTO postgis2d (name, geom) VALUES ('p2',
  ST_GeomFromText('POLYGON((1 1, 2 1, 2 2, 1 2,1 1))', 26910)
);
```

  或使用多边形集合（MULTIPOLYGON）

```sql
INSERT INTO postgis2d (name, geom) VALUES ('p2',
  ST_GeomFromText('MULTIPOLYGON(((4 0,8 0,8 4,4 0),(1 1, 2 1, 2 2, 1 2,1 1)))', 26910)
);
```

  QGIS中显示几何如下：

   ![img](https://img-blog.csdnimg.cn/20190409230405942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWF6aGVuMjAxMQ==,size_16,color_FFFFFF,t_70)

### 3.4 几何集合

```sql
INSERT INTO postgis2d (name, geom) VALUES ('c1',
  ST_GeomFromText('GEOMETRYCOLLECTION(POLYGON((4 0,8 0,8 4,4 0)),LINESTRING(2 3,3 4))',26910)
);
```

 