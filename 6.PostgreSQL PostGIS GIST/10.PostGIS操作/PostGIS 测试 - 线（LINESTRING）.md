- [PostGIS 测试 - 线（LINESTRING）_万里归来少年心的博客-CSDN博客_postgis 线](https://blog.csdn.net/liyazhen2011/article/details/88995716)

### 1.建表

```sql
CREATE TABLE linetable ( 
  id SERIAL PRIMARY KEY,
  name VARCHAR(128),
   geom GEOMETRY(LINESTRING, 26910)
);
```

### 2.添加GIST索引

```sql
CREATE INDEX linetable_gix  ON linetable USING GIST (geom); 
```

### 3.插入数据

```sql
INSERT INTO linetable (name, geom) VALUES ('L1',
  ST_GeomFromText('LINESTRING(0 0,1 1,1 2)', 26910)
);
 
INSERT INTO linetable (name, geom) VALUES ('L2',
  ST_GeomFromText('LINESTRING(1 0,2 1,2 2)', 26910)
);
```

### 4.QGIS中显示几何数据

![img](https://img-blog.csdnimg.cn/20190403151354167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeWF6aGVuMjAxMQ==,size_16,color_FFFFFF,t_70)