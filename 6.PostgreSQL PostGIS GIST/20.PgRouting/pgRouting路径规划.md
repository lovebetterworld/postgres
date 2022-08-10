- [pgRouting路径规划_贲_WM的博客-CSDN博客](https://blog.csdn.net/wm6752062/article/details/102592487?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-5-102592487-blog-83351481.pc_relevant_multi_platform_whitelistv2eslanding&spm=1001.2101.3001.4242.4&utm_relevant_index=8)

pgRouting扩展了[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020)/PostGIS地理空间数据库，以提供地理空间路由功能。在项目中用到，此处简单记录一下过程。

## 1 数据准备

​    路网数据是关键，在保持道路完整性的同时，道路在相交的路口要打断，路口的各道路起点或终点最好是同一个点。同时做好拓扑检查（ArcMap），防止道路自相交、覆盖或被覆盖等拓扑问题。

## 2 路网入库

​    使用pgAdmin在PostgreSQ新建数据库pgroute，并添加 postgis和pgrouting扩展。使用PostGIS工具将路网数据（road，坐标系为4326）导入pgroute数据库。

## 3 创建路网拓扑结构

```sql
--对road表添加起点(source)和终点(target)字段
alter table road add column source integer;
alter table road add column target integer;
 
--为了提升查询效率，需要对这source和target添加索引
create index road_source_idx on road("source");
create index road_target_idx on road("target");
 
--添加道理权重length和rev_length
alter table road add column length numeric;
alter table road add column rev_length numeric;
 
--创建权重
update road set length=ST_Length(geom,true);
update road set rev_length=length;
 
--创建拓扑结构，即为source和target字段赋值，生成road_vertices_pgr
SELECT pgr_createTopology('road',0.000002,'geom','gid');
```

## 4 创建最短路径函数

```sql
--DROP FUNCTION public.pgr_fromatob(tbl varchar,startx float,starty float,endx float,endy float);
 
CREATE OR REPLACE FUNCTION "public"."pgr_fromatob"(tbl varchar, startx float8, starty float8, endx float8, endy float8)
    RETURNS "public"."geometry" AS $BODY$ 
declare
    v_startLine geometry;--离起点最近的线 
	v_endLine geometry;--离终点最近的线 
 
	v_startTarget integer;--距离起点最近线的终点
	v_startSource integer;
	v_endSource integer;--距离终点最近线的起点
	v_endTarget integer;
	v_statpoint geometry;--在v_startLine上距离起点最近的点 
	v_endpoint geometry;--在v_endLine上距离终点最近的点 
 
	v_res geometry;--最短路径分析结果
	v_res_a geometry;
	v_res_b geometry;
	v_res_c geometry;
    v_res_d geometry; 
 
	v_perStart float;--v_statpoint在v_res上的百分比 
	v_perEnd float;--v_endpoint在v_res上的百分比 
 
	v_shPath_se geometry;--开始到结束
	v_shPath_es geometry;--结束到开始
	v_shPath geometry;--最终结果
	tempnode float; 
begin
	--查询离起点最近的线 
    execute 'select geom, source, target  from ' ||tbl||
                          ' where ST_DWithin(geom,ST_Geomfromtext(''POINT('|| startx ||' ' || starty||')'',4326),15)
                            order by ST_Distance(geom,ST_GeomFromText(''POINT('|| startx ||' '|| starty ||')'',4326))  limit 1'
                            into v_startLine, v_startSource ,v_startTarget; 
 
	--查询离终点最近的线  
	execute 'select geom, source, target from ' ||tbl||
                           ' where ST_DWithin(geom,ST_Geomfromtext(''point('|| endx || ' ' || endy ||')'',4326),15)
                            order by ST_Distance(geom,ST_GeomFromText(''point('|| endx ||' ' || endy ||')'',4326))  limit 1'
                            into v_endLine, v_endSource,v_endTarget; 
	--如果没找到，返回null
    if (v_startLine is null) or (v_endLine is null) then 
        return null; 
    end if ; 
 
    -- 查询起点和起点最近的先的点
    select  ST_ClosestPoint(v_startLine, ST_Geomfromtext('point('|| startx ||' ' || starty ||')',4326)) into v_statpoint; 
    raise notice 'v_statpoint %',  v_statpoint;
 
    -- 查询终点和终点最近的先的点
    select  ST_ClosestPoint(v_endLine, ST_GeomFromText('point('|| endx ||' ' || endy ||')',4326)) into v_endpoint; 
	raise notice 'v_endpoint %',  v_endpoint;
 
    -- ST_Distance 
    --从开始的起点到结束的起点最短路径 
	execute 'SELECT st_linemerge(st_union(b.geom)) ' ||
    'FROM pgr_kdijkstraPath ( 
    ''SELECT gid as id, source::integer, target::integer, length::double precision as cost FROM ' || tbl ||''',' 
    ||v_startSource || ', '||'array['|| v_endSource||'] , false, false 
    ) a, ' 
    || tbl || ' b 
    WHERE a.id3=b.gid   
    GROUP by id1   
    ORDER by id1' into v_res ;
 
    --从开始的终点到结束的起点最短路径
    execute 'SELECT st_linemerge(st_union(b.geom)) ' ||
    'FROM pgr_kdijkstraPath( 
    ''SELECT gid as id, source::integer, target::integer, length::double precision as cost FROM ' || tbl ||''',' 
    ||v_startTarget || ', ' ||'array['|| v_endSource||'] , false, false 
     ) a, ' 
    || tbl || ' b 
    WHERE a.id3=b.gid   
    GROUP by id1   
    ORDER by id1' into v_res_b ;
 
    --从开始的起点到结束的终点最短路径
    execute 'SELECT st_linemerge(st_union(b.geom)) ' ||
    'FROM pgr_kdijkstraPath( 
    ''SELECT gid as id, source::integer, target::integer, length::double precision as cost FROM ' || tbl ||''',' 
    ||v_startSource || ', ' ||'array['|| v_endTarget||'] , false, false 
    ) a, ' 
    || tbl || ' b 
    WHERE a.id3=b.gid   
    GROUP by id1   
    ORDER by id1' into v_res_c ;
 
    --从开始的终点到结束的终点最短路径
    execute 'SELECT st_linemerge(st_union(b.geom)) ' ||
    'FROM pgr_kdijkstraPath( 
    ''SELECT gid as id, source::integer, target::integer, length::double precision as cost FROM ' || tbl ||''',' 
    ||v_startTarget || ', ' ||'array['|| v_endTarget||'], false, false 
     ) a, ' 
    || tbl || ' b 
    WHERE a.id3=b.gid   
    GROUP by id1  
    ORDER by id1' into v_res_d ;
 
	if(ST_Length(v_res) > ST_Length(v_res_b)) then
        v_res = v_res_b;
    end if;
 
    if(ST_Length(v_res) > ST_Length(v_res_c)) then
        v_res = v_res_c;
    end if;
 
    if(ST_Length(v_res) > ST_Length(v_res_d)) then
        v_res = v_res_d;
    end if;
 
	raise notice 'v_res %',  v_res;
	raise notice 'v_startLine %',  v_startLine;
	raise notice 'v_endLine %', v_endLine;
    
    --如果没找到，返回null
	if(v_res is null) then   
        return null;   
    end if;
    
   --将v_res,v_startLine,v_endLine进行拼接 
   select  st_linemerge(st_union(array[v_res,v_startLine,v_endLine])) into v_res;
   
   select  ST_LineLocatePoint(v_res, v_statpoint) into v_perStart; 
   select  ST_LineLocatePoint(v_res, v_endpoint) into v_perEnd; 
 
   if(v_perStart > v_perEnd) then 
       tempnode =  v_perStart;
       v_perStart = v_perEnd;
       v_perEnd = tempnode;
   end if;
 
    --截取v_res 
    SELECT ST_Line_SubString(v_res,v_perStart, v_perEnd) into v_shPath;
    return v_shPath; 
end;
 
 
$BODY$
  LANGUAGE 'plpgsql' VOLATILE STRICT  COST 100
;
 
ALTER FUNCTION "public"."pgr_fromatob"(tbl varchar, startx float8, starty float8, endx float8, endy float8) OWNER TO "postgres";
```

## 5 路网测试

   输入起始经纬度坐标，执行sql语句，获取geojson格式最短路径。

```sql
 SELECT ST_AsGeoJSON(pgr_fromAtoB('road',108.253264, 40.075626, 108.254826,40.076253));
```

 