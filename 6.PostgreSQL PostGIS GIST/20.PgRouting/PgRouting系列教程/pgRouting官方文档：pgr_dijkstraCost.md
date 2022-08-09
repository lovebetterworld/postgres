- [pgRouting官方文档：pgr_dijkstraCost - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/86539197)

原文地址： [pgr_dijkstraCost - pgRouting Manual (3.0-alpha)](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/latest/en/pgr_dijkstraCost.html)

pgr_dijkstraCost是基于Boost Graph实现的Dijkstra算法，它只提取找出的最短路径消耗的总代价（Cost）。

## 一、描述

pgr_dijkstraCost函数是用于计算图中节点对最短路径的代价之和。它的时间复杂度为：O(VlogV+E)。

该函数的特征为：

①它不返回最短路径，它返回的是找到的最短路径的代价之和。

②只有路径边的cost为正值，才能将该路径边包含在计算之中。

③值在有路径时返回：

- 返回的值是一组（start_vid, end_vid, agg_cost）的形式*。*
- 当起始节点和结束节点是相同的，则最短路径不存在。此时agg_cost为0。
- 当起始节点和结束节点是不同的，但是找不到最短路径，则agg_cost为∞。

④假设返回的值存储在表中，因此唯一的主键索引将是属性对: （start_vid, end_vid）。

⑤对于无向图来说，结果是对称的。即顶点对(u, v)和顶点对(v, u)将得到相同的结果。

⑥start_vid或end_vid中的任何重复值都将被忽略。

⑦返回的值可以如下排序：

- 从start_vid上升排序
- 从end_vid上升排序

⑧运行时间：O(|start_vids|∗(VlogV+E))

## 二、函数签名

**签名摘要**

![img](https://pic3.zhimg.com/80/v2-38d870f0b05e644ddf1ca6d8e685307e_720w.jpg)

**使用默认值**

![img](https://pic1.zhimg.com/80/v2-7bf1ef8b1d9cd0e3f1601fcd03f91abc_720w.jpg)

示例：在**有向图**中从顶点2到顶点3

![img](https://pic1.zhimg.com/80/v2-ab2a995c4cca21ae2fd6c9480e160348_720w.jpg)

## 2.1、一到一

![img](https://pic4.zhimg.com/80/v2-2fd0037eabe4c9575f8fc859278f62a7_720w.jpg)

示例：在**无向图**中从顶点2到顶点3

![img](https://pic4.zhimg.com/80/v2-ca058dd40472091e211ef3ee2ceb5c7b_720w.jpg)

## 2.2、一到多

![img](https://pic3.zhimg.com/80/v2-47266444a93cf57598299d875924c9d2_720w.jpg)

示例：在**有向图**中从顶点2到两个顶点{3, 11}

![img](https://pic2.zhimg.com/80/v2-6a8adc40b6893eec5fe9a2e9e142e509_720w.jpg)

## 2.3、多到一

![img](https://pic3.zhimg.com/80/v2-17c0c7ca5f77d976fd401f0a55a1abca_720w.jpg)

示例：在**有向图**中从两个顶点{2, 7}到顶点3

![img](https://pic1.zhimg.com/80/v2-5f29fb1b589156a6be44227542513c24_720w.jpg)

## 2.4、多到多

![img](https://pic3.zhimg.com/80/v2-cbe53884848b3f0ad463d0c58d0d8f9a_720w.jpg)

示例：在有向图中从两个顶点{2, 7}到两个顶点{3, 11}

![img](https://pic2.zhimg.com/80/v2-916aaefc139a4f401b2e3598647c87cd_720w.jpg)

## 三、其他示例

示例1：忽略重复值，并对结果进行排序

![img](https://pic1.zhimg.com/80/v2-43b45bc6231a65ef850f10d94f109a88_720w.jpg)

示例2：使start_vids与end_vids相同

![img](https://pic3.zhimg.com/80/v2-430da5f5e02ba94af302ed6d561117b6_720w.jpg)