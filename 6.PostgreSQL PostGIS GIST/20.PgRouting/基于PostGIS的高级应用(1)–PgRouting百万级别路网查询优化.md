- [GIS开发者 GIS开发 (giserdqy.com)](https://www.giserdqy.com/test/37833/)

pgrouting是postgis的插件，主要做网络分析等业务使用，一般一个地区，一个城市几万级别的路网，查询是非常快速的，但是全国路网动辄几百万，几千万的路网规模，默认查询就非常的慢了。于是，本文主要以dijkstra算法，安装pg的单机默认配置，重点阐述如何“动脑经”加速路径查询速度，而不是单纯依靠机器配置（毕竟再牛逼的机器也架不住无脑的大量运算啊），当然本文的方法并不是非常规范和标准，但提供了一个解决问题的思路，即大量路网的复杂查询优化一定要避免全表查询，尽量减少计算！

# 一 全表路径分析查询

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-50337c501ebe7803199b47b0594c8f36.png?x-oss-process=image%2Fformat,webp)

北京路网.png



查询路网的线数据规模：

```
network=# select count(*) from ways;
  count
---------
 1250371
(1 row)
```

以dijkstra算法示例查询：

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-dd77fc49b6329a377388e8f2a548ce4b.png?x-oss-process=image%2Fformat,webp)

规划A_B的路径查询.png



如上图：A点坐标[115.2,39.8]，B点坐标[115.4,40]，A点的附近对应道路的gid是487371，B点附近对应道路的gid是62553（gid事先查询好的，测试就不写如何获取坐标附近的道路gid），考虑到道路有单行道的关系，所以有通行权重cost和反向权重reverse_cost ，查询语句如下：

```
SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways
',487371,62553,true);
```

返回93行记录，平均耗时5.8s，但是如图可知，其实AB两点比较近，而大部分路网其实对他们的计算根本就没关系，于是我们考虑在一开始查询时就规避无效路网。

# 二 矩形范围过滤

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-11c15ceec3384d562f9608009c3837ff.png?x-oss-process=image%2Fformat,webp)

矩形过滤



我们发现，AB之间范围及其附近的路网就足够分析出路径，而大量其他数据是没有任何影响作用的，我们设AB两点构成一个矩形，然后缓冲2km作为备用参与分析道路（即红色斜线部分），语句如下：

```
SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways
where st_intersects(geom,st_buffer(ST_PolygonFromText(''POLYGON((115.2 39.8,115.4 39.8,115.4 40,115.2 40,115.2 39.8))'',4326),0.02))
',487371,62553,true);
```

返回93行记录，平均耗时150 ms。
实验结果证明：与全表查询的分析结果一致，但全表查询是矩形查询耗时的 5800/150约49倍，明显优化速度还是很明显的。

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-2ee874a9c0037a1b83625506ed1b4690.png?x-oss-process=image%2Fformat,webp)

矩形问题.png

但是矩形查询也存在一个问题，当AB两点的经度接近（极端情况就是一致），那么两点只能构成一个 水平或者垂直的 线段（无法构造矩形区域），绝大部分形成一个细长的面区域如上图，这种情况下，因不能构造矩形筛选区域，或者说构造的区域过于狭窄无法满足路径查询的要求，采用“线性过滤”会更恰当些。

# 三 线性范围过滤

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-b48a0df7b512adf36744ca7cca27fe1e.png?x-oss-process=image%2Fformat,webp)

线性过滤示意图.png

AB两点坐标接近垂直或水平时，可选用线性查询。举例：AB两点不变，根据AB两点坐标构成线，缓冲5公里（线比矩形那个要大，尽量将可能的道路加入分析），查询语句如下：

```
SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways
where st_intersects(geom,st_buffer(ST_LineFromText(''LineString(115.2 39.8,115.4 40)'',4326),0.05))
',487371,62553,true);
```

耗时：180ms，效果仍然比较明显。

# 四 网格筛选过滤

三四节作者猜想了以矩形和线性做查询筛选（线性更通用，不推荐矩形查询），但是他们只能处理两点比较近的时候，筛选出一小部分区域作分析标本，随着两点朝相反对角线拉大，以上图形构成的查询区域也随之变得很大，即使有索引，但是pg的查询优化器发现查询的数量非常大而不是小部分时，可能不走索引，也就是说，随着两点距离变大，越来越接近于全表查询（甚至比全表查询还慢）。这种情况下，作者采用网格对路段分组，如下图：



![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-f3b5b5c5711c82f110344f7a9cccc1bd.png?x-oss-process=image%2Fformat,webp)

城市路网网格化.png



图形可视化，形象生动可见路网和格网的关系，但我们还是客观具体的看下表中数据的关系如下。
查询网格：

```
network=# d maps
          数据表 "public.maps"
 栏位  |          类型          | 修饰词
-------+------------------------+--------
 mapid | integer                | 非空
 geom  | geometry(Polygon,4326) |
索引：
    "maps_pkey" PRIMARY KEY, btree (mapid)
    "maps_geom_index" gist (geom)
network=# select * from maps limit 10;
 mapid  |                                                                                                geom                                                                                
--------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 595756 | 0103000020E6100000010000000500000065BDBD10976F5D402753A278D7DF434065BDBD10976F5D407D776D6786EA4340C85C19549B775D407D776D6786EA4340C85C19549B775D402753A278D7DF434065BDBD10976F5D402753A278D7DF4340
 555571 | 0103000020E61000000100000005000000463150849AC75C4025A93394E69F4240463150849AC75C40AB6B40C694AA42406270E6BA9DCF5C40AB6B40C694AA42406270E6BA9DCF5C4025A93394E69F4240463150849AC75C4025A93394E69F4240
 615867 | 0103000020E6100000010000000500000074D6E1C79CB75D40CA83DB771895444074D6E1C79CB75D4024EAEC01C69F44407C61536399BF5D4024EAEC01C69F44407C61536399BF5D40CA83DB771895444074D6E1C79CB75D40CA83DB7718954440
 615707 | 0103000020E61000000100000005000000E144B24F99775D4091A8275E2B554440E144B24F99775D403459CC9DD35F4440EE76FF50977F5D403459CC9DD35F4440EE76FF50977F5D4091A8275E2B554440E144B24F99775D4091A8275E2B554440
 605752 | 0103000020E610000001000000050000007EEB1E34964F5D40911DA72A253544407EEB1E34964F5D40D20A5FA1873E4440A3C87B5192575D40D20A5FA1873E4440A3C87B5192575D40911DA72A253544407EEB1E34964F5D40911DA72A25354440
 555451 | 0103000020E61000000100000005000000A71F798C97875C4093814DE7948A4240A71F798C97875C40BA5D58CC429542406E34FC7E9C8F5C40BA5D58CC429542406E34FC7E9C8F5C4093814DE7948A4240A71F798C97875C4093814DE7948A4240
 595632 | 0103000020E610000001000000050000009238F1F69C0F5D40308FCA877FCA43409238F1F69C0F5D40FE13F9812DD5434081CFEE149B175D40FE13F9812DD5434081CFEE149B175D40308FCA877FCA43409238F1F69C0F5D40308FCA877FCA4340
 625533 | 0103000020E6100000010000000500000098C4D5D890D75C40E3B5BF7161CA444098C4D5D890D75C40F392BDAD0DD54440F0826F3794DF5C40F392BDAD0DD54440F0826F3794DF5C40E3B5BF7161CA444098C4D5D890D75C40E3B5BF7161CA4440
 615633 | 0103000020E610000001000000050000008947A0C997175D406F0490771A7544408947A0C997175D406A69B0A1C27F44407B9CEDFA9A1F5D406A69B0A1C27F44407B9CEDFA9A1F5D406F0490771A7544408947A0C997175D406F0490771A754440
 615663 | 0103000020E610000001000000050000007DAF424697175D407346393D149544407DAF424697175D407CCE61E7BB9F44400418F9699A1F5D407CCE61E7BB9F44400418F9699A1F5D407346393D149544407DAF424697175D407346393D14954440
(10 行记录)
```

查询路网

```
network=# d ways
                                 数据表 "public.ways"
    栏位    |           类型            |                   修饰词
------------+---------------------------+---------------------------------------------
 gid        | integer                   | 非空 默认 nextval('ways_gid_seq'::regclass)
 name       | character varying(128)    |
 pyname     | character varying(128)    |
 mapid      | integer                   |
 id         | character varying(13)     |
 kind_num   | character varying(2)      |
 kind       | character varying(30)     |
 width      | character varying(3)      |
 direction  | character varying(1)      |
 toll       | character varying(1)      |
 const_st   | character varying(1)      |
 undconcrid | character varying(13)     |
 snodeid    | integer                   |
 enodeid    | integer                   |
 funcclass  | character varying(2)      |
 detailcity | character varying(1)      |
 through    | character varying(1)      |
 unthrucrid | character varying(13)     |
 ownership  | character varying(1)      |
 road_cond  | character varying(1)      |
 special    | character varying(1)      |
 admincodel | character varying(6)      |
 admincoder | character varying(6)      |
 uflag      | character varying(1)      |
 onewaycrid | character varying(13)     |
 accesscrid | character varying(13)     |
 speedclass | character varying(1)      |
 lanenums2e | character varying(2)      |
 lanenume2s | character varying(2)      |
 lanenum    | character varying(1)      |
 vehcl_type | character varying(32)     |
 elevated   | character varying(1)      |
 structure  | character varying(1)      |
 usefeecrid | character varying(13)     |
 usefeetype | character varying(1)      |
 spdlmts2e  | character varying(4)      |
 spdlmte2s  | character varying(4)      |
 spdsrcs2e  | character varying(1)      |
 spdsrce2s  | character varying(1)      |
 dc_type    | character varying(1)      |
 nopasscrid | character varying(13)     |
 geom       | geometry(LineString,4326) |
 length     | double precision          |
 rev_length | double precision          |
 x1         | double precision          |
 y1         | double precision          |
 x2         | double precision          |
 y2         | double precision          |
索引：
    "ways_pkey" PRIMARY KEY, btree (gid)
    "mapid_index" btree (mapid)
    "ways_enodeid_idx" btree (enodeid)
    "ways_geom_idx" gist (geom)
    "ways_snodeid_idx" btree (snodeid)
network=# select a.gid,a.mapid from ways a,(select mapid from maps limit 1) b where a.mapid=b.mapid limit 10;
   gid   | mapid
---------+--------
  112434 | 595756
  112440 | 595756
  117555 | 595756
   23611 | 595756
 1041239 | 595756
 1193746 | 595756
  694218 | 595756
  735844 | 595756
  739260 | 595756
  740230 | 595756
(10 行记录)
```

网格和路网之间已经根据mapid建立了关系，路网中mapid建立了索引，格网中mapid是主键。
根据图形之前的关系，我们的思路是：根据AB两点建立直线，对该直线建立一定范围内的缓冲面，缓冲面查询与哪些网格有相交关系（相交意味着这些网格是有效的分析网格，其他网格就没任何关系了），直接把这些有效的网格中的路段，作为分析的样本。

![img](https://mtr-1.oss-cn-beijing.aliyuncs.com/qyblog/2022/01/frc-47450fd48367b4c6276549e2b20abc20.png?x-oss-process=image%2Fformat,webp)

测试范围.png



全表查询AB路径如下：

```
--测试样本
network=# select count(*) from ways;
  count
---------
 1250371
(1 行记录)
network=# SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways'
,647331,856772,true);
--耗时8.1s
```

线性查询AB路径如下：

```
network=# select count(*) from ways where geom&&st_buffer(ST_LineFromText('LineString(114.53247 37.34692,118.125 39.82983)',4326),0.08);
 count
--------
 678892
(1 行记录)
network=# SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways
where geom&&st_buffer(ST_LineFromText(''LineString(114.53247 37.34692,118.125 39.82983)'',4326),0.08)
',647331,856772,true);
--耗时4.5s
```

网格查询AB路径如下：

```
network=# select count(*) from ways where mapid in ( select mapid from maps where st_intersects(geom,st_buffer(ST_LineFromText('LineStr
ing(114.53247 37.34692,118.125 39.82983)',4326),0.08)));
 count
--------
 127914
(1 行记录)
network=# SELECT * FROM pgr_dijkstra('SELECT gid as id,snodeid as source,enodeid as target,length::float as cost,rev_length::float as reverse_cost FROM ways
where mapid in (select mapid from maps where st_intersects(geom,st_buffer(ST_LineFromText(''LineString(114.53247 37.34692,118.125 39.82983)'',4326),0.08)))
',647331,856772,true);
--耗时1.8s
```

表格对比如下：

| 查询方式 | 查询数据量 | 查询时间 |
| :------: | :--------: | :------: |
| 全表查询 |  1250371   |  8.1 s   |
| 线性查询 |   678892   |  4.5 s   |
| 网格查询 |   127914   |  1.8 s   |

虽然不是太满意结果，但是结果的确体现了“智慧的结晶”了，还算欣慰。

# 五 道路属性过滤

以全国路网分析简介（未测试），虽然几百万路网，但是 高速，快速路这种道路并不是很多，在做远距离路径分析时，优先选择起点最近的高速A1，终点附近的高速B1，那么先规划AA1，再A1B1，再B1B，将全国路网全表查询细分成高速路路径分析，高速和城市道路分析，细节部分也可采用范围叠加规划。同样的，对完整路网拆分成高速国道的一级查询，省道县道的二级查询，乡道的三级查询等等，尽可能将路网从整体分析中抽离出去。

# 五 总结

在近距离路径分析时，可考虑范围进行过滤，然后查询过滤后的路网。在远距离路径分析时，可以实际的条件做过滤。在城市道路时，远距离规划车辆路径，应优先规划起点到绕城，绕城出口到终点。
  当然，以上测试都是单机测试，全部思路是减少查询路网数量，是动脑经的结果，没有在数据库的配置，硬件的优化上下功夫。作者将在适当时机，移入pgxl进行规划分析测试，看集群是否有利于查询速度提升。