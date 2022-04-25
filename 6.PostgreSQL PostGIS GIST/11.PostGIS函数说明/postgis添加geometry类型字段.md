## 1 postgis添加geometry类型字段

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

## 2 Postgresql存储Geometry对象类型

- [Postgresql存储Geometry对象类型](https://www.bianchengquan.com/article/366575.html)

### 2.1 查询Geometry

```sql
select ST_GeomFromText('Polygon((117.357442 30.231278,119.235188 30.231278,119.235188 32.614617,117.357442 32.614617,117.357442 30.231278))');
```

### 2.2 设计Geometry字段表

#### 2.2.1 政区表：Geometry=Polygon

```sql
drop table if EXISTS aggregate_state;
/***创建政区表***/
create table aggregate_state(  
  pk_uid serial primary key,  
	code CHARACTER VARYING(20) NOT NULL,
	"name" CHARACTER VARYING(100) NOT NULL,
	zoom INTEGER NOT NULL,
  geom geometry(Polygon,4326) NOT NULL ,/*geom public.geometry(Polygon,4490),*/
	longitude double precision ,
  latitude double precision
);
```

#### 2.2.2 空间业务表：Geometry=Point

```plsql
drop table if EXISTS aggregate_spatial;
/***创建空间业务表***/
create table aggregate_spatial(  
  pk_uid serial primary key,  
	sheng CHARACTER VARYING(20),
	shi CHARACTER VARYING(20),
	xian CHARACTER VARYING(20),
	xiang CHARACTER VARYING(20),
	cun CHARACTER VARYING(20),
	mzguid CHARACTER VARYING(255),
	zoom INTEGER,
  geom geometry(Point,4326) ,/*geom public.geometry(Point,4490),*/
	longitude double precision,
  latitude double precision
);
```

### 2.3 保存Geometry的类型

![img](https://cdn.bianchengquan.com/556f391937dfd4398cbac35e050a2177/blog/5ffd2c48b07aa.png)

用得比较多的就是point、path、Polygon、text。下面是保存示例：

```sql
insert into aggregate_state(code,name,zoom,geom,longitude,latitude) VALUES('21','上海',7,ST_GeomFromText('SRID=4326;Polygon((117.357442 30.231278,119.235188 30.231278,119.235188 32.614617,117.357442 32.614617,117.357442 30.231278))'),
ST_X(st_centroid(ST_GeomFromText('SRID=4326;Polygon((117.357442 30.231278,119.235188 30.231278,119.235188 32.614617,117.357442 32.614617,117.357442 30.231278))'))),
ST_Y(st_centroid(ST_GeomFromText('SRID=4326;Polygon((117.357442 30.231278,119.235188 30.231278,119.235188 32.614617,117.357442 32.614617,117.357442 30.231278))')))
);
```

注意：SRID必须与设计的表对应，且Geometry的类型要对应，insert into示例还计算了面的中心展示位置方便聚合数据输出到地图显示。

#### 2.3.1 Multipoint多点保存

```sql
CREATE TABLE xh_yw.xh_point_tb
(
    id bigint NOT NULL,
    track_point geometry(MultiPoint,4326),
    CONSTRAINT xh_point_tb_pkey PRIMARY KEY (id)
)

INSERT INTO xh_yw.xh_point_tb (id,track_point) VALUES (1,
  ST_GeomFromText('MULTIPOINT((0 0),(5 0),(0 10))', 4326)
);
```

