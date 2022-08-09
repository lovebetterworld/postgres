- [pgrouting最优路径规划之一_evomap的博客-CSDN博客_pgrouting](https://blog.csdn.net/u014529917/article/details/72866436)

## 一、pgrouting安装和配置

需要根据系统版本、PostgreSQL版本、PostGIS版本来选择合适的pgrouting版本

下载包以后解压缩，将lib目录下文件复制到PostgreSQL的lib目录下，再在PostgreSQL数据库中执行share/extension目录下的sql脚本，这样就完成了整个环境的配置

配置完成后，使用命令行创建数据库，使数据库支持PostGIS和pgRouting的函数和基础表

```sql
CREATE EXTENSION postgis;
CREATE EXTENSION pgrouting;
CREATE EXTENSION postgis_topology;
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION address_standardizer;
```

## 二、导入shp数据

我使用的shp数据是自己手工绘制的，并通过拓扑打断相交线
我这里使用的是PostgreSQL自带的pgadmin，将shp数据导入到PostgreSQL数据库:

打开postgis安装目录下的PostGISShapefile Import/Export Manager

![img](https://img-blog.csdn.net/20170605133728000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

点击view connection details,设置数据库的连接

![img](https://img-blog.csdn.net/20170605133807578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在Import选项卡中，点击Add File，并选择进行路径规划的路网shp

![img](https://img-blog.csdn.net/20170605133955160?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

导入成功后就能在postgresql中看到导入的数据表

![img](https://img-blog.csdn.net/20170605134021505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



## 三、创建拓扑

//添加起点id

```sql
ALTER TABLE crossroad ADD COLUMN source integer;
```

//添加终点id

```sql
ALTER TABLE crossroad ADD COLUMN target integer;
```

//添加道路权重值

```sql
ALTER TABLE crossroad ADD COLUMN length double precision;
```

//为sampledata表创建拓扑布局，即为source和target字段赋值，生成crossroad_vertices_pgr

```sql
SELECT pgr_createTopology('crossroad',0.00001, 'geom', 'gid');
```

//为source和target字段创建索引

```sql
CREATE INDEX source_idx ON crossroad ("source");

CREATE INDEX target_idx ON crossroad ("target");
```

//为length赋值

```sql
update crossroad set length =st_length(geom);
```

//或者用已有的字段长度赋值，下面shape_length为shp中已有的长度属性

```sql
UPDATE crossroad SET length = shape_length;
```

//为crossroad表添加reverse_cost字段并用length的值赋值

```sql
ALTER TABLE crossroad ADD COLUMN reverse_cost double precision;
UPDATE crossroad SET reverse_cost =length;
```

## 四、查询

使用pgr_dijkstra算法查询

```sql
SELECT seq, id1 AS node, id2 AS edge, cost FROM pgr_dijkstra('
SELECT gid AS id,                   
source::integer,                       
target::integer,                      
length::double precision AS cost
FROM crossroad,
1, 9, false, false);
```

查询结果如图所示：

![img](https://img-blog.csdn.net/20170605134152897?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

其中node代表路径的节点、edge代表边、cost就是length字段的成本，从1号点到9号点的最短路径可视化结果：1-8-9

## 五、geoserver发布图层

a.生成路网基础图

先发布路网shp数据，再发布crossroad_vertices_pgr，再将两图层叠加到一起，形成路径规划的基础图。（红色标注为节点编号）

![img](https://img-blog.csdn.net/20170605134254680?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

b.生成路径规划图

新建图层，选择数据存储为postGIS的存储，再选择“配置新的SQL视图”，在SQl语句中添加以下语句，创建视图

```sql
SELECT seq, id1 AS node, id2 AS edge, cost,geom  FROM pgr_dijkstra('
SELECT gid AS id,                    
source::integer,                       
target::integer,                      
length::double precision AS cost
FROM crossroad',
1, 9, false, false) as di
join crossroad pt
on di.id2 = pt.gid;
```

计算好边界之后，再将1和9替换为带参的%a%和%b%

## **六、展示查询结果**

在Openlayers中，我们通过TileWMS的方式来加载路网基础和路径规划

路网基础服务：

```javascript
var roadLayer = new ol.layer.Tile({
          source: new ol.source.TileWMS({
            url: 'http://localhost:8080/geoserver/test/wms',
            params: {'LAYERS': 'test:cross', 'TILED': true},
            serverType: 'geoserver'
          })
        })
map.addLayer(roadLayer);
```

路径规划服务：（带参数）

```javascript
var routeLayer = new ol.layer.Tile({
	source: new ol.source.TileWMS({
		url: 'http://localhost:8080/geoserver/test/wms',
		params: {'LAYERS': 'test:cross_route', 'TILED': true,'viewparams':'a:'+val1+';b:'+val2},
		serverType: 'geoserver'
	})
})
map.addLayer(routeLayer);
```

最终结果：

![img](https://img-blog.csdn.net/20170605135523104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDUyOTkxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)