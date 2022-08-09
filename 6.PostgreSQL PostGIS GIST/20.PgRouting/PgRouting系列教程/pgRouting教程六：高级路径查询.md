- [pgRouting教程六：高级路径查询 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/86960640)

不仅行人要路径规划，而且交通工具更需要路径规划，也更频繁使用到路径规划。

## 一、车辆路径

车辆路径规划一般与行人路径规划不同：

①车辆道路是有方向的

②成本（代价）可以是：

- 距离
- 时间
- 货币 —— 例如人民币，欧元
- 二氧化碳排放量
- 车辆耗损等

③在双向道路上必须同时考虑cost和reverse_cost属性

- 成本应该和cost属性相同或和reverse_cost属性相同
- 同一条道路的cost和reverse_cost可以是不同的

这是因为有些道路是单向的。

单向道路的特征是：

- (source, target) 路段中，cost >= 0 且 reverse_cost < 0
- (target, source)路段中，cost < 0 且 reverse_cost >= 0

所以，用cost属性或reverse_cost属性的**负值**表示对应方向是无效的。

对于双向道路，cost >= 0和reverse_cost >= 0，它们的值可能不同。例如，在一条倾斜的道路中，下山肯定比上山容易（或者更快）。成本可以是长度、时间、坡度、路面、道路类型等，也可以是多个参数的组合。

**以下查询显示单向路段的数量：**

①（target, source)单向路段（cost < 0）的数量

```sql
SELECT count(*)
FROM shenzhen_roads
WHERE cost < 0;
```

![img](https://pic4.zhimg.com/80/v2-ce61255c958b88531b26841bda7a8647_720w.jpg)

②（source, target)单向路段（reverse_cost < 0）的数量

```sql
SELECT count(*)
FROM shenzhen_roads
WHERE reverse_cost < 0;
```

![img](https://pic2.zhimg.com/80/v2-47601c75f28fb15ff49644e567ce6df9_720w.png)

**1.1、练习1 —— 行车路径-出发**

乘车从深圳北站出发到深圳野生动物园。

![img](https://pic2.zhimg.com/80/v2-4a888b0592d8df920e43e6f5317239e5_720w.jpg)

- 车辆将从深圳北站（50013）到深圳野生动物园（19688）。
- 使用cost和reverse_cost列，它们的单位为米。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	 FROM shenzhen_roads
	',
	50013, 19688,
	directed := true
);
```

![img](https://pic4.zhimg.com/80/v2-1443264d0ce8d35f66ff105a0ebce44b_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-8226749a0a80c64b6062972bda8d31d9_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-dcc55bb2bf179eec929e50d7747a36c8_720w.jpg)

以下红色路线就是查询的结果路线：

![img](https://pic2.zhimg.com/80/v2-12ff94c92983d4fa6b147033252e2ced_720w.jpg)

**1.2、练习2 —— 行车路径-返回**

乘车从深圳野生动物园返回到深圳北站。

- 车辆将从深圳野生动物园（19688）到深圳北站（50013）。
- 使用cost和reverse_cost列，它们的单位为米。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	 FROM shenzhen_roads
	',
	19688, 50013, 
	directed := true
);
```

![img](https://pic4.zhimg.com/80/v2-778dd86f9fe768019cc9c43ce8734b07_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-b7b1d54a09722393e0e070560b2fd59e_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-bc15db727e1468489afac59fe2a4e8fc_720w.jpg)

![img](https://pic4.zhimg.com/80/v2-bf8168f025c1f0258c155592f7265fd7_720w.jpg)

以下绿色路径就是查询的结果路径：

![img](https://pic3.zhimg.com/80/v2-3cf46f696155360e126d23eee1945026_720w.jpg)

可以发现返回的路径和出发的路径在有些范围不是同一条路径（说明对应路径是单向路径）：

![img](https://pic4.zhimg.com/80/v2-7ed0d500005181caff41c4154642ee87_720w.jpg)

注意：在有向图上，往返路径的成本是不同的。

## 二、操作路径的成本

在处理数据时，了解所使用的数据类型可以顾及更多因素，从而得到更理想的结果。

例如，如果我们知道某条路径是行人路径（pedestrian），就应该让车辆尽量避免或禁止在该路径上行驶。

**2.1、练习3 —— 没有优先级的行车路径**

OSM的路网数据中具有包含道路类型信息的fclass列，基于该列查询道路的类型信息：

```sql
SELECT DISTINCT fclass FROM shenzhen_roads ORDER BY fclass;
```

![img](https://pic4.zhimg.com/80/v2-e08e63c05679983ef39d85bb41740b93_720w.jpg)

现在添加一个penalty列用于保存每条路径的优先级信息（penalty值越小，优先级越高）：

```sql
ALTER TABLE shenzhen_roads ADD COLUMN penalty DOUBLE PRECISION;
```

把每条路径的penalty的值都设为1.0，表示所有路径都没有优先级之分（penalty值都为1.0）。

```sql
UPDATE shenzhen_roads SET penalty = 1.0;
```

基于**无优先级路径**查询乘车从深圳北站（50013）出发到深圳野生动物园（19688）的路径（将cost * penalty作为正向路径成本）：

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost * penalty AS cost, 
	        reverse_cost
	 FROM shenzhen_roads
	',
	50013, 19688, 
	directed := true
);
```

查询的结果路径和练习1的结果路径一致：

![img](https://pic1.zhimg.com/80/v2-71964ea1c50910d16ce0fb52499d9284_720w.jpg)

**2.2、练习4 —— 具有优先级的行车路径**

因为业务场景是查找车辆的行驶路径，所以可以让路径的优先级符合以下规则：

- 禁止在pedestrian、footway、steps、cycleway、track这些路径上行驶。
- 不鼓励在rsidential路径上行驶。
- 鼓励在primary、primary_link路径上行驶。

修改路径的优先级具体如下：

```sql
UPDATE shenzhen_roads SET penalty = -1.0
WHERE fclass IN ('steps','footway','pedestrian', 'cycleway', 'track', 'track_grade2');

UPDATE shenzhen_roads SET penalty = 5.0
WHERE fclass IN ('residential');

UPDATE shenzhen_roads SET penalty = 0.8
WHERE fclass IN ('tertiary', 'tertiary_link', 'motorway', 'motorway_link', 'living_street');
UPDATE shenzhen_roads SET penalty = 0.5
WHERE fclass IN ('secondary', 'secondary_link');

UPDATE shenzhen_roads SET penalty = 0.3
WHERE fclass IN ('primary', 'primary_link', 'trunk', 'trunk_link');
```

现在基于**具有优先级的路径**再次查询乘车从深圳北站（50013）出发到深圳野生动物园（19688）的路径（将cost * penalty作为正向路径成本）：

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost * penalty AS cost, 
	    reverse_cost
	 FROM shenzhen_roads
	',
	50013, 19688, 
	directed := true
);
```

![img](https://pic4.zhimg.com/80/v2-40b15a3080df8b711dc741e38fe00aff_720w.jpg)

![img](https://pic4.zhimg.com/80/v2-c019c9c348f461787b41344c525d83bb_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-3ed5bfa34e6db55c3e161a342a6489d8_720w.jpg)

查询的结果路径如下蓝色路径所示：

![img](https://pic1.zhimg.com/80/v2-07475894e99939fc681e66a77e5bea68_720w.jpg)

可以发现基于具有优先级的路径的查询结果与基于不具有优先级的路径的查询结果不同，原因正是我们为路网设置的优先级起的作用，即查询兼顾了路网优先级这一影响因素。

通过为路网设置优先级，我们可以让数据满足更多的业务场景。