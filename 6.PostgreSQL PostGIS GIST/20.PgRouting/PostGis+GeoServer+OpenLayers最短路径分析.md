- [PostGis+GeoServer+OpenLayers最短路径分析_qgbihc的博客-CSDN博客_openlayers路径分析](https://blog.csdn.net/qgbihc/article/details/108635912)

使用PostGis存储路线数据，在GeoServer中发布路网图层和[最短路径](https://so.csdn.net/so/search?q=最短路径&spm=1001.2101.3001.7020)图层，通过OpenLayers建立路网和展示图层。

1、PostGis：[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020) 12.4,
2、GeoServer：geoserver-2.17.2
3、[OpenLayers](https://so.csdn.net/so/search?q=OpenLayers&spm=1001.2101.3001.7020)：6.4.3

## 一、做成路网数据

### 1、在PostGis建立路网数据表，主键使用[sequence](https://so.csdn.net/so/search?q=sequence&spm=1001.2101.3001.7020)，路网geom 使用geometry(LineString,4326)

```sql
CREATE SEQUENCE public.my_roads_id_seq
    INCREMENT 1
    START 1
    MINVALUE 1
    MAXVALUE 2147483647
    CACHE 1;
    
ALTER SEQUENCE public.my_roads_id_seq OWNER TO postgres;

CREATE TABLE public.my_roads
(
    id integer NOT NULL DEFAULT nextval('my_roads_id_seq'::regclass),
    name character varying(32) COLLATE pg_catalog."default",
    geom geometry(LineString,4326),
    CONSTRAINT my_roads_pkey PRIMARY KEY (id)
)TABLESPACE pg_default;

ALTER TABLE public.my_roads OWNER to postgres;
```

### 2、在GeoServer中发布路网图层

建立工作区，建立postgis的数据存储，发布my_roads图层。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917094730682.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
EPSG使用4326，边框及经纬度为：【-180，-90 】【180，90】即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917094845290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

### 3、OpenLayers中建立路网

使用vue开发路网编辑画面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917103321188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
实现方式，请参考：[OpenLayers 在Vue中增删改](https://blog.csdn.net/qgbihc/article/details/108547898)

注意点：新增的线不再是newFeature.setGeometry(new MultiLineString([geometry.getCoordinates()]))

```javascript
// 转换坐标
 let geometry = this.drewFeature.getGeometry().clone();
 geometry.applyTransform(function (flatCoordinates, flatCoordinates2, stride) {
     for (let j = 0; j < flatCoordinates.length; j += stride) {
         let y = flatCoordinates[j];
         let x = flatCoordinates[j + 1];
         flatCoordinates[j] = x;
         flatCoordinates[j + 1] = y;
     }
 });

 // 设置feature对应的属性，这些属性是根据数据源的字段来设置的
 let newFeature = new Feature();
 newFeature.setId('nyc_roads.new.' + this.newId);
 newFeature.setGeometryName('the_geom');
 newFeature.set('the_geom', null);
 newFeature.set('name', newFeature.getId());
 newFeature.setGeometry(new MultiLineString([geometry.getCoordinates()]));
```

而是newFeature.setGeometry(geometry);

```javascript
// 转换坐标
let geometry = this.drewFeature.getGeometry().clone();
geometry.applyTransform(function (flatCoordinates, flatCoordinates2, stride) {
    for (let j = 0; j < flatCoordinates.length; j += stride) {
        let y = flatCoordinates[j];
        let x = flatCoordinates[j + 1];
        flatCoordinates[j] = x;
        flatCoordinates[j + 1] = y;
    }
});

// 设置feature对应的属性，这些属性是根据数据源的字段来设置的
let newFeature = new Feature();
newFeature.setId('am_roads.new.' + this.newId);
newFeature.setGeometryName('geom');
newFeature.set('geom', null);
newFeature.set('name', newFeature.getId());
newFeature.setGeometry(geometry);
```

编辑好的路网如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917105637113.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

## 二、路径分析基础数据

### 1、路网拓扑生成

```sql
-- 创建拓扑布局，即为source和target字段赋值
select pgr_createTopology('my_roads',0.001, 'geom', 'id');

--为cost 赋值
update my_roads set cost =st_length(geom);
--为双向reverse_cost 赋值
UPDATE my_roads SET reverse_cost = st_length(geom);

--为线的起点和终点坐标赋值 
UPDATE my_roads SET x1 =ST_x(ST_PointN(geom, 1));	
UPDATE my_roads SET y1 =ST_y(ST_PointN(geom, 1));	
UPDATE my_roads SET x2 =ST_x(ST_PointN(geom, ST_NumPoints(geom)));	
UPDATE my_roads SET y2 =ST_y(ST_PointN(geom, ST_NumPoints(geom)));
```

### 2、最短路径函数

此函数参考网上路径分析函数，解决了计算不准确问题和MultiLineString问题。

- 建立根据拓扑后节点，计算最短路径函数pgr_shortestpath_edge。

```sql
CREATE  OR REPLACE FUNCTION pgr_shortestpath_edge (
	tbl CHARACTER VARYING,
	fromNode INTEGER,
	toNode INTEGER,
	startLine geometry,
	endLine geometry)
RETURNS geometry
AS $BODY$ 
DECLARE
	v_res geometry;--最短路径分析结果
	v_linemerge geometry;--合并线后结果
	v_shPath geometry;--最终结果
BEGIN
	--从开始的节点到结束的节点最短路径
	EXECUTE'SELECT st_linemerge(st_union(b.geom)) ' || 'FROM 
	pgr_dijkstra( ''SELECT id, source, target, cost FROM ' || tbl || ''',' || fromNode || ', ' || toNode || ', false ) a
	LEFT JOIN  ' || tbl || ' b  ON a.edge = b.id ' INTO v_res;
	
	-- 合并线
	select  st_linemerge(ST_Union(array[startLine, v_res, endLine])) into v_linemerge; 
	
 	RETURN v_linemerge;
END; 

$BODY$ LANGUAGE plpgsql VOLATILE STRICT COST 100;
ALTER FUNCTION pgr_shortestpath_edge ( CHARACTER VARYING, INTEGER, INTEGER, geometry, geometry ) OWNER TO postgres;
```

- 建立最短路径函数

```sql
CREATE  OR REPLACE FUNCTION pgr_shortestpath ( tbl CHARACTER VARYING, startx DOUBLE PRECISION, starty DOUBLE PRECISION, endx DOUBLE PRECISION, endy DOUBLE PRECISION ) RETURNS geometry
AS $BODY$ DECLARE
	v_startLine geometry;--离起点最近的线
	v_endLine geometry;--离终点最近的线
	v_startPoint geometry;--在v_startLine上距离起点最近的点
	v_endPoint geometry;--在v_endLine上距离终点最近的点
	
	v_startLine_pre geometry; --起点到v_startPoint的线  
	v_endLine_next geometry; --终点到v_endPoint的线  
	v_startPoint_pre geometry;  --起点
	v_endPoint_pre geometry;  --终点
	
	v_startTarget INTEGER;--距离起点最近线的目标节点
	v_startSource INTEGER;--距离起点最近线的源节点
	v_endSource INTEGER;--距离终点最近线的源节点
	v_endTarget INTEGER;--距离起点最近线的目标节点
	v_start_x1 double precision;
	v_start_y1 double precision;
	v_start_x2 double precision;
	v_start_y2 double precision;
	v_end_x1 double precision;
	v_end_y1 double precision;
	v_end_x2 double precision;
	v_end_y2 double precision;
	
	v_startLine_Source geometry;
	v_startLine_Target geometry;
	v_endLine_Source geometry;
	v_endLine_Target geometry;
	
	v_res geometry;--最短路径分析结果
	v_res_a geometry;
	v_res_b geometry;
	v_res_c geometry;
	v_res_d geometry;
	
	v_shPath geometry;--最终结果

BEGIN
	--查询离0.01度(大约1000米)范围内起点的最近线
	EXECUTE'select geom, source, target,x1,y1,x2,y2  from ' || tbl || ' where ST_DWithin(geom,ST_Geometryfromtext(''point(' || startx || ' ' || starty || ')'',4326),0.01)
	order by ST_Distance(geom,ST_GeometryFromText(''point(' || startx || ' ' || starty || ')'',4326))  limit 1'
	INTO v_startLine,v_startSource,v_startTarget,v_start_x1,v_start_y1,v_start_x2,v_start_y2;
	
	--查询离终点0.01度(大约1000米)范围内的最近线
	EXECUTE'select geom, source, target,x1,y1,x2,y2 from ' || tbl || ' where ST_DWithin(geom,ST_Geometryfromtext(''point(' || endx || ' ' || endy || ')'',4326),0.01)
	order by ST_Distance(geom,ST_GeometryFromText(''point(' || endx || ' ' || endy || ')'',4326))  limit 1' 
	INTO v_endLine,v_endSource,v_endTarget,v_end_x1,v_end_y1,v_end_x2,v_end_y2;
	
	--如果没找到最近的线，就返回null
	IF ( v_startLine IS NULL )  OR ( v_endLine IS NULL ) THEN
		RETURN NULL;
	END IF;
	
	 --创建查询的起点和终点  
	 SELECT ST_SetSRID( ST_MakePoint(startx , starty),4326 )INTO v_startPoint_pre;  
	 SELECT ST_SetSRID( ST_MakePoint(endx , endy),4326 )INTO v_endPoint_pre;  
	 
	--查询线上最接近某点的点（此点在线上）
	SELECT ST_ClosestPoint ( v_startLine, v_startPoint_pre) INTO v_startPoint;
	SELECT ST_ClosestPoint ( v_endLine, v_endPoint_pre) INTO v_endPoint;

	--根据最近点，把最近线分为source和target两段
	SELECT ST_MakeLine( v_startPoint,ST_SetSRID( ST_MakePoint(v_start_x1 , v_start_y1),4326 )) INTO v_startLine_Source;
	SELECT ST_MakeLine( v_startPoint,ST_SetSRID( ST_MakePoint(v_start_x2 , v_start_y2),4326 )) INTO v_startLine_Target;
	SELECT ST_MakeLine( v_endPoint,ST_SetSRID( ST_MakePoint(v_end_x1 , v_end_y1),4326 )) INTO v_endLine_Source;
	SELECT ST_MakeLine( v_endPoint,ST_SetSRID( ST_MakePoint(v_end_x2 , v_end_y2),4326 )) INTO v_endLine_Target;
	
	--计算最短路径（开始的起点/终点 --- 结束的起点/终点）
	SELECT pgr_shortestpath_edge (tbl, v_startSource, v_endSource, v_startLine_Source, v_endLine_Source) INTO v_res_a;
	SELECT pgr_shortestpath_edge (tbl, v_startTarget, v_endSource, v_startLine_Target, v_endLine_Source) INTO v_res_b;
	SELECT pgr_shortestpath_edge (tbl, v_startSource, v_endTarget, v_startLine_Source, v_endLine_Target) INTO v_res_c;
	SELECT pgr_shortestpath_edge (tbl, v_startTarget, v_endTarget, v_startLine_Target, v_endLine_Target) INTO v_res_d;
	
	v_res = v_res_a;
	IF( ST_Length ( v_res ) > ST_Length ( v_res_b )) THEN
		v_res = v_res_b;
	END IF;
	IF(ST_Length ( v_res ) > ST_Length ( v_res_c )) THEN
		v_res = v_res_c;
	END IF;
	IF(ST_Length ( v_res ) > ST_Length ( v_res_d )) THEN
		v_res = v_res_d;
	END IF;
	
	--创建查询起点和终点到最近的线之间的线
	SELECT ST_MakeLine( v_startPoint,v_startPoint_pre) INTO v_startLine_pre;  
	SELECT ST_MakeLine( v_endPoint,v_endPoint_pre) INTO v_endLine_next;  

	select st_union(array[v_endLine_next, v_res, v_startLine_pre]) into v_shPath; 
	
    return v_shPath;
END; 

$BODY$ LANGUAGE plpgsql VOLATILE STRICT COST 100;
ALTER FUNCTION pgr_shortestpath ( CHARACTER VARYING, DOUBLE PRECISION, DOUBLE PRECISION, DOUBLE PRECISION, DOUBLE PRECISION ) OWNER TO postgres;
```

## 三、显示最短路径

### 1、GeoServer建立创建sql视图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917111914331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917112252440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)
D的默认值全部为0，
验证的正则表达式全部为^-?[\d.]+$
类型选为 Geometry类型，SRD为4326
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917112541714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)

### 2、OpenLayer中传参并加载最短路径图层

VUE中增加加载最短路径图层方法：

```javascript
querypath(){
    let startCoord = [116.43442149206543,39.927703206481944]
    let destCoord = [116.410648020996,39.9069321798706]
    let params = {
        LAYERS: 'postgis:shortestpath',
        FORMAT: 'image/png',
    };
    let viewparams = [
        'x1:' + startCoord[0], 'y1:' + startCoord[1],
        'x2:' + destCoord[0], 'y2:' + destCoord[1]
    ];
    params.viewparams = viewparams .join(';');

    let imageLayer = new ImageLayer({
        source: new ImageWMS(
            {
                url: 'http://localhost:8088/geoserver/postgis/wms',
                params: params
            }),
    });
    this.map.addLayer(imageLayer);
},
```

### 3、最终结果显示

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917113226221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FnYmloYw==,size_16,color_FFFFFF,t_70#pic_center)