- [基于pgrouting的路径规划处理 - 开放GIS - 博客园 (cnblogs.com)](https://www.cnblogs.com/share-gis/p/16148019.html)

对于GIS业务来说，路径规划是非常基础的一个业务，一般公司如果处理，都会直接选择调用已经成熟的第三方的接口，比如高德、百度等。当然其实路径规划的算法非常多，像比较著名的Dijkstra、A*算法等。当然本篇文章不是介绍算法的，本文作者会根据pgrouting已经集成的Dijkstra算法来，结合postgresql数据库来处理最短路径。

## 一、数据处理

   路径规划的核心是数据，数据是一般的路网数据，但是我们拿到路网数据之后，需要对数据进行处理，由于算法的思想是基于有向图的原理，因此首先需要对数据做topo处理，通过topo我们其实就建立了路网中各条道路的顶点关系，下面是主要命令：   

```plsql
--开启执行路网topo的插件
create extension postgis;
create extension postgis_topology;
--数据创建拓扑
ALTER TABLE test_road ADD COLUMN source integer;
ALTER TABLE test_road ADD COLUMN target integer;
SELECT pgr_createTopology('test_road',0.00001, 'geom', 'gid');
```

其中test_road是将路网数据导入到postgresql中的表名。

处理完topo之后，基本就够用了，我们就可以借助pgrouting自带的函数，其实有很多，我们选择pgr_dijkstra

```plsql
CREATE OR REPLACE FUNCTION public.pgr_dijkstra(
    IN edges_sql text,
    IN start_vid bigint,
    IN end_vid bigint,
    IN directed boolean,
    OUT seq integer,
    OUT path_seq integer,
    OUT node bigint,
    OUT edge bigint,
    OUT cost double precision,
    OUT agg_cost double precision)
  RETURNS SETOF record AS
$BODY$
DECLARE
BEGIN
    RETURN query SELECT *
    FROM _pgr_dijkstra(_pgr_get_statement($1), start_vid, end_vid, directed, false);
  END
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
  ROWS 1000;
ALTER FUNCTION public.pgr_dijkstra(text, bigint, bigint, boolean)
  OWNER TO postgres;
```

从函数输入参数可以看到，我们需要一个查询sql,一个起始点、一个结束点、以及是否考虑方向，好了了解到调用函数输入参数，我们就来写这个函数。

## 二、原理分析

　　一般路径规划，基本都是输入一个起点位置、一个终点位置然后直接规划，那么对于我们来说，要想套用上面的函数，必须找出起点位置target ，以及终点位置的source，然后规划根据找出的这两个topo点，调用上面的函数，来返回自己所需要的结果。

　　如何根据起始点找到对应的target呢，其实就是找离起点最近线的target，同理终点的source，其实就是找离终点最近线的source,当然将这两个点规划规划好之后，基本就可以了，但是最后还需要将起点到起点最近先的target连接起来，终点到终点最近线的source连接起来，这样整个路径规划就算完成了。

　　下面我们来看具体的实现存储过程：

```plsql
CREATE OR REPLACE FUNCTION public.pgr_shortest_road(
    IN startx double precision,
    IN starty double precision,
    IN endx double precision,
    IN endy double precision,
    OUT road_name character varying,
    OUT v_shpath character varying,
    OUT cost double precision)
RETURNS SETOF record AS
$BODY$ 
declare 
v_startLine geometry;--离起点最近的线 
v_endLine geometry;--离终点最近的线 
v_startTarget integer;--距离起点最近线的终点 
v_endSource integer;--距离终点最近线的起点 
v_statpoint geometry;--在v_startLine上距离起点最近的点 
v_endpoint geometry;--在v_endLine上距离终点最近的点 
v_res geometry;--最短路径分析结果 
v_perStart float;--v_statpoint在v_res上的百分比 
v_perEnd float;--v_endpoint在v_res上的百分比 
v_rec record; 
first_name varchar;
end_name varchar;
first_cost double precision;
end_cost double precision;
begin 
--查询离起点最近的线 
execute 'select geom,target,name from china_road where 
ST_DWithin(geom,ST_Geometryfromtext(''point('|| startx ||' ' || starty||')''),0.01) 
order by ST_Distance(geom,ST_GeometryFromText(''point('|| startx ||' '|| starty ||')'')) limit 1' 
into v_startLine ,v_startTarget,first_name; 
--查询离终点最近的线 
execute 'select geom,source,name from china_road
where ST_DWithin(geom,ST_Geometryfromtext(''point('|| endx || ' ' || endy ||')''),0.01) 
order by ST_Distance(geom,ST_GeometryFromText(''point('|| endx ||' ' || endy ||')'')) limit 1' 
into v_endLine,v_endSource,end_name; 
--如果没找到最近的线，就返回null 
if (v_startLine is null) or (v_endLine is null) then 
return; 
end if ; 
select ST_ClosestPoint(v_startLine, ST_Geometryfromtext('point('|| startx ||' ' || starty ||')')) into v_statpoint; 
select ST_ClosestPoint(v_endLine, ST_GeometryFromText('point('|| endx ||' ' || endy ||')')) into v_endpoint; 

--计算距离起点最近线上的点在该线中的位置
select ST_Line_Locate_Point(st_linemerge(v_startLine), v_statpoint) into v_perStart;

select ST_Line_Locate_Point(st_linemerge(v_endLine), v_endpoint) into v_perEnd;

select ST_Distance_Sphere(v_statpoint,ST_PointN(ST_GeometryN(v_startLine,1), ST_NumPoints(ST_GeometryN(v_startLine,1)))) into first_cost;

select ST_Distance_Sphere(ST_PointN(ST_GeometryN(v_endLine,1),1),v_endpoint) into end_cost; 

if (ST_Intersects(st_geomfromtext('point('|| startx ||' '|| starty ||') '), v_startLine) and ST_Intersects(st_geomfromtext('point('|| endx ||' '|| endy ||') '), v_startLine)) then 
select ST_Distance_Sphere(v_statpoint, v_endpoint) into first_cost;

select ST_Line_Locate_Point(st_linemerge(v_startLine), v_endpoint) into v_perEnd;
for v_rec in 
select ST_Line_SubString(st_linemerge(v_startLine), v_perStart,v_perEnd) as point,COALESCE(end_name,'无名路') as name,end_cost as cost loop
v_shPath:= ST_AsGeoJSON(v_rec.point);
cost:= v_rec.cost;
road_name:= v_rec.name;
return next;
end loop;
return;
end if;
--最短路径 
for v_rec in 
(select ST_Line_SubString(st_linemerge(v_startLine),v_perStart,1) as point,COALESCE(first_name,'无名路') as name,first_cost as cost
 union all
 SELECT st_linemerge(b.geom) as point,COALESCE(b.name,'无名路') as name,b.length as cost
 FROM pgr_dijkstra(
     'SELECT gid as id, source, target, length as cost FROM china_road
     where st_intersects(geom,st_buffer(st_linefromtext(''linestring('||startx||' ' || starty ||','|| endx ||' ' || endy ||')''),0.05))', 
     v_startTarget, v_endSource , false 
 ) a, china_road b 
 WHERE a.edge = b.gid
 union all
 select ST_Line_SubString(st_linemerge(v_endLine),0,v_perEnd) as point,COALESCE(end_name,'无名路') as name,end_cost as cost)
loop
v_shPath:= ST_AsGeoJSON(v_rec.point);
cost:= v_rec.cost;
road_name:= v_rec.name;
return next;
end loop; 
end; 
$BODY$
LANGUAGE plpgsql VOLATILE STRICT;
```

上面这种实现，是将所有查询道路返回一个集合，然后客户端来将各个线路进行合并，这种方式对最终效率影响比较大，所以一般会在函数中将道路何合并为一条道路，我们可以使用postgis的st_union函数来处理，小编经过长时间的试验，在保证效率和准确性的情况下，对上面的存储过程做了很多优化，最终得出了如下：

```plsql
CREATE OR REPLACE FUNCTION public.pgr_shortest_road(
    startx double precision,
    starty double precision,
    endx double precision,
    endy double precision)
  RETURNS geometry AS
$BODY$   
declare  
    v_startLine geometry;--离起点最近的线  
    v_endLine geometry;--离终点最近的线 
    v_perStart float;--v_statpoint在v_res上的百分比  
    v_perEnd float;--v_endpoint在v_res上的百分比  
    v_shpath geometry;
    distance double precision;
    bufferInstance double precision;
    bufferArray double precision[];  
begin  
    execute 'select geom,
        case china_road.direction
        when ''3'' then
        source
        else
        target
        end 
        from china_road where  
        ST_DWithin(geom,ST_Geometryfromtext(''point('|| startx ||' ' || starty||')'',4326),0.05) 
        AND width::double precision >= '||roadWidth||'
        order by ST_Distance(geom,ST_GeometryFromText(''point('|| startx ||' '|| starty ||')'',4326))  limit 1'  
    into v_startLine; 

    execute 'select geom,
        case china_road.direction
        when ''3'' then
        target
        else
        source
        end 
        from china_road
        where ST_DWithin(geom,ST_Geometryfromtext(''point('|| endx || ' ' || endy ||')'',4326),0.05)  
        AND width::double precision >= '||roadWidth||'
        order by ST_Distance(geom,ST_GeometryFromText(''point('|| endx ||' ' || endy ||')'',4326))  limit 1'  
    into v_endLine;  

    if (v_startLine is null) or (v_endLine is null) then  
        return null;
    end if; 

    if (ST_equals(v_startLine,v_endLine)) then 
        select ST_LineLocatePoint(st_linemerge(v_startLine), ST_Geometryfromtext('point('|| startx ||' ' || starty ||')',4326)) into v_perStart;
        select ST_LineLocatePoint(st_linemerge(v_endLine), ST_Geometryfromtext('point('|| endx ||' ' || endy ||')',4326)) into v_perEnd;
        select ST_LineSubstring(st_linemerge(v_startLine),v_perStart,v_perEnd) into v_shPath;
        return v_shPath;
    end if;

    select ST_DistanceSphere(st_geomfromtext('point('|| startx ||' ' || starty ||')',4326),st_geomfromtext('point('|| endx ||' ' || endy ||')',4326)) into distance;
    if ((distance / 1000) > 50) then
        bufferArray := ARRAY[0.1,0.2,0.3,0.5,0.8];    
    else
        bufferArray := ARRAY[0.02,0.05,0.08,0.1];
    end if;

    forEACH bufferInstance IN ARRAY bufferArray
    LOOP
    select _pgr_shortest_road(startx,starty,endx,endy,bufferInstance) into v_shPath;
    if (v_shPath is not null) then    
        return v_shPath;
    end if;
    end loop;

    end;  
    $BODY$
  LANGUAGE plpgsql VOLATILE STRICT
  COST 100;
ALTER FUNCTION public.pgr_shortest_road(double precision, double precision, double precision, double precision )
  OWNER TO postgres;

DROP FUNCTION public._pgr_shortest_road(double precision, double precision, double precision, double precision, double precision);
```

上面的函数，其实对于大部分情况下的操作，基本可以满足了。

## 三、效率优化

　　其实在数据查询方面，我们使用的是起点和终点之间的线性缓冲来提高效率，如下：

```plsql
SELECT gid as id, source, target, cost,rev_cost as reverse_cost FROM china_road
      where geom && st_buffer(st_linefromtext(''linestring('||startx||' ' || starty ||','|| endx ||' ' || endy ||')'',4326),'||bufferDistance||')
```

　　当然这在大部分情况下，依旧是不错的，然后在有些情况下，并不能起到很好的作用，因为如果起点和终点之间道路偏移较大（比如直线上的山脉较多的时候，路就会比较绕），这个时候，可能会增大缓冲距离，而增加缓冲距离就会导致，部分区域的查询量增大，继而影响效率，因此其实我们可以考虑使用mapid这个参数，这个参数从哪来呢，一般我们拿到的路网数据都会这个字段，我们只需要生成一个区域表，而这个区域表就俩个字段，一个是mapid，一个是这个mapid的polygon范围，这样子，上面的查询条件，就可以换成如下：　　

```plsql
SELECT gid as id, source, target, cost,rev_cost as reverse_cost FROM china_road
      where mapid in (select mapid from maps where geom && st_buffer(st_linefromtext(''linestring('||startx||' ' || starty ||','|| endx ||' ' || endy ||')''),'||bufferDistance||'))
```

这样就可以在很大程度上提高效率。

## 四、数据bug处理

　　其实有时候我们拿到的路网数据，并不是非常的准确，或者说是录入的有瑕疵，我自己遇到的就是生成的topo数据，本来一条路的target应该和它相邻路的source的点重合，然后实际却是不一样，这就导致最终规划处的有问题，因此，简单写了一个处理这种问题的函数　

```plsql
CREATE OR REPLACE FUNCTION public.modity_road_data()
RETURNS void AS
$BODY$   
declare  
n integer;
begin  
    for n IN (select distinct(source) from china_road )  loop
        update china_road
        set geom = st_multi(st_addpoint(ST_geometryN(geom,1),
        (select st_pointn(ST_geometryN(geom,1),1) from china_road where source = n
        limit 1),
        st_numpoints(ST_geometryN(geom,1))))
        where target = n;
    end loop;
    end;  
    $BODY$
  LANGUAGE plpgsql VOLATILE STRICT
  COST 100;
ALTER FUNCTION public.modity_road_data()
OWNER TO postgres;
```

## 五、后续规划

　　上面的函数已在百万数据中做过验证，后续还会验证千万级别的路网数据，当然这种级别，肯定要在策略上做一些调整了，比如最近测试的全国路网中，先规划起点至起点最近的高速入口，在规划终点至终点最近的高速出口，然后再高速路网上规划高速入口到高速出口的路径，这样发现效率提升不少，当然，这里面还有很多逻辑和业务，等所有东西都验证完毕，会再出一版，千万级别路径规划的文章。