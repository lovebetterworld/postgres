- [pgRouting教程四：准备数据 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/121602526)

本文将介绍如何准备实验数据并将实验数据导入数据库。

思考再三，决定使用OpenStreetMap的深圳范围数据作为实验数据。

## 一、准备实验数据

首先要下载OpenStreetMap中国地区数据，下载地址：[Geofabrik Download Server](https://link.zhihu.com/?target=http%3A//download.geofabrik.de/asia.html)

![img](https://pic1.zhimg.com/80/v2-a843dda072974a7543ad7656e964b174_720w.jpg)

下载shapefile格式的数据，然后裁剪出深圳范围的路网数据。

由于OSM数据是WGS84坐标系（EPSG:4326）的，所以还需将其转换为Web墨卡托坐标系（EPSG:3857)。

另外还需要使用ArcGIS中的"**要素转线**"这个工具将折线数据在相交点处打断：

![img](https://pic1.zhimg.com/80/v2-21e9ab333937883b0b5b9d8ca8ec2170_720w.png)

这样实验数据就准备好了，如下所示：

![img](https://pic3.zhimg.com/80/v2-5e99f883400376ccf7f9451d7af3989e_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-6a90e31afd578d711178e12a04155a4d_720w.jpg)

实验数据我已经放到网盘：

链接：[https://pan.baidu.com/s/1UWxwtiKVB9zvYnpyb-hpog](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1UWxwtiKVB9zvYnpyb-hpog)

提取码：xudh

## 二、将数据导入数据库

**2.1、创建数据库**

直接使用postgres用户创建一个名为**city_routing**的数据库：

![img](https://pic2.zhimg.com/80/v2-86a925c2ed53aed80470084b53a1dee9_720w.jpg)

在city_routing数据库中分别添加PostGIS插件和pgRouting插件：

![img](https://pic2.zhimg.com/80/v2-962f100542a5fe174e7808368718cd5d_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-db2e21a8d67ba3853d234769b0ff7086_720w.jpg)

通过查询版本来测试以上两个插件是否添加成功：

![img](https://pic3.zhimg.com/80/v2-63d2c5095984ccc65383e631c75f704e_720w.png)

![img](https://pic2.zhimg.com/80/v2-9a5284db62499231189df50083099f41_720w.jpg)

添加成功，数据库创建完成！

### **2.2、使用PostGIS的数据导入工具导入数据**

在Windows中安装PostGIS会自动安装Shapefile数据导入工具：

![img](https://pic3.zhimg.com/80/v2-33086dbf59e6b2a0ed0182b53efcfbc2_720w.jpg)

那么就可以使用这个工具将深圳路网数据导入刚刚创建的city_routing数据库。

打开PostGIS 2.0 Shapefile and DBF Exporter Manager工具，连接city_routing数据库：

![img](https://pic1.zhimg.com/80/v2-3abaad504a28200e6802d40bfbec9c24_720w.jpg)

然后点击**Add File**，添加shenzhen_roads.shp文件：

![img](https://pic4.zhimg.com/80/v2-b8d921eb3c4746b2583441992d116267_720w.jpg)

修改SRID：

![img](https://pic3.zhimg.com/80/v2-59bdd86b6775a782b31d73df51f262a2_720w.jpg)

接着务必点击**Options**选项，然后勾选**Import Options**中的最后一个选项，使数据以单几何图形的形式导入到数据库：

![img](https://pic2.zhimg.com/80/v2-4992176953069dd83d70ee5de903be4d_720w.jpg)

最后点击**Import**导入数据，查看log window看数据是否导入成功:

![img](https://pic3.zhimg.com/80/v2-19c10bab8aa56de1eec90bca864cb98e_720w.jpg)

数据导入成功！

## 三、设置路径成本并建立路网拓扑

**3.1、设置路径成本**

首先为shenzhen_roads表添加四个字段（后面涉及到对应字段时再详细解释）：

- source —— 用于保存路径起始顶点的id
- target —— 用于保存路径终止顶点的id
- cost —— 用于保存路径正向的成本（或者代价）
- reverse_cost —— 用于保存路径反向的成本（或者代价）

SQL语句：

```sql
ALTER TABLE shenzhen_roads
ADD COLUMN source INTEGER,
ADD COLUMN target INTEGER,
ADD COLUMN cost DOUBLE PRECISION,
ADD COLUMN reverse_cost DOUBLE PRECISION;
```

OSM数据包含一个oneway列，它的值可能是以下三个值之一：

- '**F**' —— 表示该路径是单向的，且路径方向是正向的。
- '**T**' —— 表示该路径是单向的，且路径方向是反向的。
- '**B**' —— 表示该路径是双向的。

现在就根据路径的方向来计算各个路径的成本。

将路径的长度作为成本，如果路径是正向的，则cost值为路径的长度，reverse_cost值为-1；如果路径是反向的，则reverse_cost值为路径的长度，cost值为-1；如果路径是双向的，则cost和reverse_cost的值都为路径的长度。

①正向路径的成本：

```sql
UPDATE shenzhen_roads
SET cost = ST_Length(geom), reverse_cost = -1
WHERE oneway = 'F';
```

②反向路径的成本：

```sql
UPDATE shenzhen_roads
SET reverse_cost = ST_Length(geom), cost = -1
WHERE oneway = 'T';
```

③双向路径的成本：

```sql
UPDATE shenzhen_roads
SET cost = ST_Length(geom), reverse_cost = ST_Length(geom)
WHERE oneway = 'B';
```

**3.2、创建路网拓扑**

创建路网拓扑需要调用[pgr_createTopology](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/latest/en/pgr_createTopology.html)函数：

```sql
SELECT pgr_createTopology(
	'shenzhen_roads', 
	0.001,
	'geom',
	'gid',
	'source',
	'target'
); 
```

上面的六个参数分别表示：

- 路网表
- 路径之间的容差，两条路径的距离大于这个容差值，就表示它们不相交，否则就是相交。
- 路网表中包含空间信息的列
- 路网表的主码列
- 保存路径起始顶点的id的列
- 保存路径终止顶点的id的列

拓扑路径创建完成后，数据库中会自动多出一张表**shenzhen_roads_vertices_pgr：**

![img](https://pic3.zhimg.com/80/v2-47d2fdec42fb88b6230ab8c7d4277c22_720w.jpg)

这张表保存了路径的起始、终止顶点数据：

![img](https://pic4.zhimg.com/80/v2-0da6ffe906bebc5f7188574a36f23d67_720w.jpg)

与路网一起可视化显示：

![img](https://pic1.zhimg.com/80/v2-f3ae5a41204805a0098b1d9efd11ed40_720w.jpg)

我们再查询路网表shenzhen_roads，可以发现source列和target列都被填满了，值就是shenzhen_roads_vertices_pgr这张表中对应的的id值：

![img](https://pic2.zhimg.com/80/v2-3564b19da534d90fe0162830b3c25de1_720w.jpg)

这样，实验数据就准备好了！