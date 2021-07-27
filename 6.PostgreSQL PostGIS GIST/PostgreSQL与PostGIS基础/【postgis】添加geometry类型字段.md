# postgis添加geometry类型字段

创建一张测试表

```plsql
CREATE TABLE test1(
  id int4,
  name varchar(255)
)

> NOTICE:  Table doesn't have 'DISTRIBUTED BY' clause -- Using column named 'id' as the Greenplum Database data distribution key for this table.
> HINT:  The 'DISTRIBUTED BY' clause determines the distribution of data. Make sure column(s) chosen are the optimal data distribution key to minimize skew.


> 时间: 0.246s
```

增加geometry类型字段

```plsql
SELECT AddGeometryColumn ('test1', 'the_geom', 4326, 'POINT', 2);
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200703153136472.png)

插入数据

```plsql
INSERT INTO test1 (id, the_geom, name) VALUES (1,ST_GeomFromText('POINT(-0.1257 51.508)',4326),'London, England');
INSERT INTO test1 (id, the_geom, name) VALUES (2,ST_GeomFromText('POINT(-81.233 42.983)',4326),'London, Ontario');
INSERT INTO test1 (id, the_geom, name) VALUES (3,ST_GeomFromText('POINT(27.91162491 -33.01529)',4326),'East London,SA');
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200703152928560.png)