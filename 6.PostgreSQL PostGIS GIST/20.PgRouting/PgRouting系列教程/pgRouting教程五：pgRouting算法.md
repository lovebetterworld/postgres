- [pgRouting教程五：pgRouting算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/121655755)

pgRouting最初被称为pgDijkstra，因为它只使用Dijkstra算法实现最短路径搜索的功能。后来，添加了其他函数，才将pgDijkstra改名为pgRouting。

## 一、pgr_dijkstra

Dijkstra算法是第一个在pgRouting中实现的算法。它不需要除id、source、target和cost之外的其他属性。而且可以明确指定将图视为**有向的**或**无向的**。

## **pgr_dijkstra函数的签名摘要：**

![img](https://pic4.zhimg.com/80/v2-e127e6c34d0ca3400727d2f0638cc37f_720w.jpg)

详细信息可以查看这篇文章：

[不睡觉的怪叔叔：pgRouting官方文档：pgr_dijkstra5 赞同 · 1 评论文章](https://zhuanlan.zhihu.com/p/85905703)

**注意：**

- 许多pgRouting函数都将sql :: text作为参数之一，虽然这开始看起来很混乱，但它使函数非常灵活。用户可以传递任何SELECT语句作为函数参数，只要该SELECT语句返回的结果包含所需数量的属性且包含正确的属性名。
- 多数pgRouting实现的算法不需要路网几何信息。
- 大多数pgRouting函数不返回几何信息，而只返回包含节点id或路径id的有序列表。

## 查询的标识符

source列和target列的顶点标识符（对应shenzhen_roads_vertices_pgr的id值）的分配可能不同，因为**pgr_createTopology函数**随机分配顶点标识符。所以每次运行pgr_createTopology函数建立路网拓扑，source列和target列的顶点标识符都是不同的。（因此仅以文章中的内容作为参考，然后使用你自己的数据进行测试）。

此教程将会使用深圳市的一些地点信息，这些地点的地名和对应的shenzhen_roads_vertices_pgr表中的顶点id值为：

- 世界之窗 —— 12089
- 锦绣中华民俗村 —— 10564
- 欢乐谷 —— 13019
- 华侨城体育场 —— 7304

下图中的三角形标识即为这四个地点：

![img](https://pic2.zhimg.com/80/v2-c20b4b3621c3a1e3dfdb307af5e0ed19_720w.jpg)

## 1.1、练习1——单条行人路线（一到一）

从锦绣民俗中华村步行到世界之窗

![img](https://pic1.zhimg.com/80/v2-13f89a6ead729887ab62a88e6f868dbc_720w.jpg)

- 行人想从顶点10564（锦绣民俗中华村）到顶点12089（世界之窗）（**再次强调，由于source列和target列的分配可能与文章中不同，所以请根据自己的数据进行练习**）
- 行人的步行的代价按路线长度来衡量的。
- 从行人的角度看，图是无向的，即行人在所有路段上都可以沿道路的两个方向移动。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	FROM shenzhen_roads',
	10564, 12089,
	directed := FALSE
);
```

结果（结果路径的可视化见上图）：

![img](https://pic2.zhimg.com/80/v2-ae73325bab97338ca3d2dae20cf2cb7d_720w.jpg)

以上的node列表示会经过的顶点，edge列表示会经过的路径，cost表示对应单条路径的成本，agg_cost表示路径的总成本。

**注意：**

- 返回的cost属性表示edges_sql参数中指定的cost属性。在这个例子中，cost是路径的长度。cost可以是时间、距离或任何其他属性以及自定义公式的组合。
- 结果中的node属性和edge属性取决于使用pgr_createTopology函数时对顶点标识符的分配。这个前面已经提过。

## 1.2、练习2——多个行人前往同一个目的地（多到一）

一个行人从锦绣民俗中华村步行到世界之窗，另外一个行人从欢乐谷步行到世界之窗。它们的起点不同，但终点一样。

![img](https://pic3.zhimg.com/80/v2-a07acd3db184aae88fa968bfd6f81e62_720w.jpg)

- 两个行人分别从顶点10564（锦绣民俗中华村）和13019（欢乐谷）出发。
- 两个行人都想去顶点12089（世界之窗）。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost, reverse_cost
	FROM shenzhen_roads',
	ARRAY[10564, 13019], 12089,
	directed := FALSE
);
```

结果：

![img](https://pic4.zhimg.com/80/v2-215231b13f73cde7fec5c4b537efe4bb_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-9d7f71219f6ce0085aa2f26d703551e0_720w.jpg)

这是两个行人所走的两条路径，start_vid为10564为一个行人的路径信息，start_vid为13019为另一个行人的路径信息。

## 1.3、练习3——许多行人从同一地点离开（一到多）

两个行人都从世界之窗离开，然后分别去锦绣中华民俗村和欢乐谷。

![img](https://pic3.zhimg.com/80/v2-a07acd3db184aae88fa968bfd6f81e62_720w.jpg)

- 两个行人都从顶点12089（世界之窗）出发。
- 两个行人分别要去顶点10564（锦绣民俗中华村）和13019（欢乐谷）。
- 使用行人的步行时间作为成本，假设行人的步行速度为s = 1.3 m/s，所以步行时间t = d / s。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost/1.3 AS cost, reverse_cost/1.3 AS reverse_cost
	FROM shenzhen_roads',
	12089, ARRAY[10564, 13019],
	directed := FALSE
);
```

结果：

![img](https://pic3.zhimg.com/80/v2-58baa7be44d9700ec8326f7424fd0cc2_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-69a9f09fb849961f4ad170583a8a0c36_720w.jpg)

## 1.4、练习4——多个行人到不同的目的地（多到多）

两个行人分别从世界之窗和欢乐谷出发，然后每个人都去两个地点锦绣民俗中华村和华侨城体育场 。

![img](https://pic3.zhimg.com/80/v2-462b9d8f060f3db4311009a58af81c06_720w.jpg)

- 两个行人分别从顶点12089（世界之窗）和顶点13019（欢乐谷）出发。
- 两个行人都会去两个目的地：10564（锦绣民俗中华村）和7304（华侨城体育场）。
- 也是使用行人的步行时间作为权重，不过单位是分钟。行人的步行速度为s = 1.3 m/s，所以步行时间t = d / s / 60。

```sql
SELECT * FROM pgr_dijkstra(
	'SELECT gid AS id,
		source, target,
		cost/1.3/60 AS cost, reverse_cost/1.3/60 AS reverse_cost
	FROM shenzhen_roads',
	ARRAY[12089, 13019], ARRAY[10564, 7304],
	directed := FALSE
);
```

结果：

![img](https://pic4.zhimg.com/80/v2-1f0893012f685d774941303fee7da97f_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-2db6fe27a53853e6da869b1472d85521_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-6a85a9bbdbef46958c785ae736b142ec_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-80561df4dd1b4df012fd4d5eca3c0af8_720w.jpg)

## 二、pgr_dijkstraCost

当主要目标是计算总代价（总成本）时，即在不关注pgr_dijkstra()的全部返回结果的情况下，使用pgr_dijkstraCost函数能返回更紧凑的结果。

**函数签名**

![img](https://pic1.zhimg.com/80/v2-633820c530b702c89fb3d5b412b39190_720w.jpg)

详细信息可以查看下面这篇文章：

[不睡觉的怪叔叔：pgRouting官方文档：pgr_dijkstraCost2 赞同 · 1 评论文章](https://zhuanlan.zhihu.com/p/86539197)

**2.1、练习5——多个行人到不同的目的地，返回总代价**

![img](https://pic4.zhimg.com/80/v2-2ec0c8e6e6dc37aacf1faf4aa8fb8b63_720w.jpg)

两个行人分别从世界之窗和欢乐谷出发，然后每个人都去两个地点锦绣民俗中华村和华侨城体育场 。

- 两个行人分别从顶点12089（世界之窗）和顶点13019（欢乐谷）出发。
- 两个行人都会去两个目的地：10564（锦绣民俗中华村）和7304（华侨城体育场）。
- 使用行人的步行时间作为权重，单位是分钟。行人的步行速度为s = 1.3 m/s，所以步行时间t = d / s / 60。
- 返回总代价值

```sql
SELECT * FROM pgr_dijkstraCost(
	'SELECT gid AS id,
		source, target,
		cost/1.3/60 AS cost, reverse_cost/1.3/60 AS reverse_cost
	FROM shenzhen_roads',
	ARRAY[12089, 13019], ARRAY[10564, 7304],
	directed := FALSE
);
```

结果：

![img](https://pic2.zhimg.com/80/v2-69d711423a5e5a29f86d9aee4c6d341d_720w.jpg)

**2.2、练习6——多个行人前往不同的目的地，将每个行人所花费的步行时间求和**

两个行人分别从世界之窗和欢乐谷出发，然后每个人都去两个地点锦绣民俗中华村和华侨城体育场 。

- 两个行人分别从顶点12089（世界之窗）和顶点13019（欢乐谷）出发。
- 两个行人都会去两个目的地：10564（锦绣民俗中华村）和7304（华侨城体育场）。
- 使用行人的步行时间作为权重，单位是分钟。行人的步行速度为s = 1.3 m/s，所以步行时间t = d / s / 60。
- 将每个行人的步行时间求和

```sql
SELECT start_vid, SUM(agg_cost) 
FROM pgr_dijkstraCost(
	'SELECT gid AS id,
		source, target,
		cost/1.3/60 AS cost, reverse_cost/1.3/60 AS reverse_cost
	FROM shenzhen_roads',
	ARRAY[12089, 13019], ARRAY[10564, 7304],
	directed := FALSE
)
GROUP BY start_vid
ORDER BY start_vid;
```

结果：

![img](https://pic2.zhimg.com/80/v2-58787bae7bbe2a032bba85f62766230d_720w.jpg)