#  1 PostGIS 如何计算两个经纬度之间的距离

```sql
SELECT ST_Distance(ST_GeomFromText('POINT(120.30 30.06)')::geography, ST_GeomFromText('POINT(120.36 30.16)')::geography));
```

注意：如果POINT是3857坐标系的，就需要转一下，转成4326坐标系的再进行计算，如下所示

```sql
SELECT ST_Distance(ST_GeomFromText('POINT(lon1 lat1)', 4326)::geography, ST_GeomFromText('POINT(lon2 lat2)', 4326)::geography);
```

# 2 算两点距离的2种方法小结

postgresql计算两点距离

### 2.1 下面两种方法：

```plsql
select 
ST_Distance(
 ST_SetSRID(ST_MakePoint(115.97166453999147,28.716493914230423),4326)::geography,
 ST_SetSRID(ST_MakePoint(106.00231199774656,29.719258550486572),4326)::geography
),
ST_Length(
 ST_MakeLine(
 ST_MakePoint(115.97166453999147,28.716493914230423),
 ST_MakePoint(106.00231199774656,29.719258550486572)
 )::geography
)
```

备注：

```plsql
create or replace function getdistance
( 
 lon1 numeric,
 lat1 numeric, 
 lon2 numeric, 
 lat2 numeric 
) 
returns int 
as 
$body$ 
declare 
v_distance numeric;
v_earth_radius numeric;
radLat1 numeric;
radLat2 numeric;
v_radlatdiff numeric;
v_radlngdiff numeric;
begin 
 --地球半径
 v_earth_radius:=6378137;
  
 radLat1 := lat1 * pi()/180.0;
 radLat2 := lat2 * pi()/180.0;
 v_radlatdiff := radLat1 - radLat2;
 v_radlngdiff := lon1 * pi()/180.0 - lon2 * pi()/180.0; 
 v_distance := 2 * asin(sqrt(power(sin(v_radlatdiff / 2), 2) + cos(radLat1) * cos(radLat2) * power(sin(v_radlngdiff/2),2)));
 v_distance := round(v_distance * v_earth_radius);
 return v_distance; 
end;
$body$
language 'plpgsql' volatile;
```

**补充：postgresql计算两点欧式距离（经纬度地理位置）**

我就废话不多说了，大家还是直接看代码吧~

```plsql
create or replace function getdistance
( 
 i_lngbegin real,
 i_latbegin real, 
 i_lngend real, 
 i_latend real 
) 
returns float 
as 
$body$
/*
 * 
 * select getdistance_bygispoint(116.281524,39.957202,117.648673,38.42584) as distance;
 * */ 
declare 
v_distance real;
v_earth_radius real;
v_radlatbegin real;
v_radlatend real;
v_radlatdiff real;
v_radlngdiff real;
begin 
 --地球半径
 v_earth_radius:=6378.137;
  
 v_radlatbegin := i_latbegin * pi()/180.0;
 v_radlatend := i_latend * pi()/180.0;
 v_radlatdiff := v_radlatbegin - v_radlatend;
 v_radlngdiff := i_lngbegin * pi()/180.0 - i_lngend * pi()/180.0; 
 v_distance := 2 * asin(sqrt(power(sin(v_radlatdiff / 2), 2) + cos(v_radlatbegin) * cos(v_radlatend) * power(sin(v_radlngdiff/2),2)));
 v_distance := v_distance * v_earth_radius*1000; 
 return v_distance; 
end;
$body$ 
language 'plpgsql' volatile;
```