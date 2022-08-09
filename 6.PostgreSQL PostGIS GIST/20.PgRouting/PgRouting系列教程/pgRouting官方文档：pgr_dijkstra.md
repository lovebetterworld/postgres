- [pgRouting官方文档：pgr_dijkstra - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/85905703)

原文地址：[pgr_dijkstra - pgRouting Manual (2.6)](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/2.6/en/pgr_dijkstra.html%23pgr-dijkstra)

**pgr_dijkstra** —— 使用dijkstra算法返回最短路径。基于Boost.Graph实现。

![img](https://pic4.zhimg.com/80/v2-e8d081eb32f80ecf8f0f36c61733951b_720w.png)

## 一、概述

Dijkstra算法，由荷兰计算机科学家**Edsger Dijkstra**（上面的照片就是这位大佬！）于1956年提出。它是一种图搜索算法，它解决了非负代价边路径图的最短路径问题，即从起始顶点（start_vid）到结束顶点（end_vid）的最短路径。此算法可以与**有向图**或**无向图**一起使用。

## 二、特征

pgr_dijkstra的主要特征是：

①过程只在成本为正的边路径上进行。

②值在有路径时返回：

- 当起始顶点和结束顶点相同时，就没有路径。此时，返回的agg_cost属性值为0。
- 当起始顶点和结束顶点不同且没有路径时，返回的agg_cost属性值为∞（无穷大）。

③出于优化的目的，start_vids或end*_*vids中的任何重复值都将被忽略。

④返回的值可以如下排序：

- 从start_vid上升排序
- 从end_vid上升排序

⑤运行时间：O(|start_vids|∗(VlogV+E))

## 三、函数的签名摘要

![img](https://pic4.zhimg.com/80/v2-ddb302be2633c82dec5c64ab3b44943b_720w.jpg)

## 四、函数签名的描述

## 4.1、edge_sql查询语句的描述

![img](https://pic1.zhimg.com/80/v2-c3bf16f60631dd11cfd55de061416c2c_720w.jpg)

上面的数据类型包括：

![img](https://pic1.zhimg.com/80/v2-855248258be8f62dd6d306cd857e1544_720w.png)

## 4.2、参数的描述

![img](https://pic4.zhimg.com/80/v2-c0f2eac2edef464f3c36f2838260757f_720w.jpg)

## 4.3、返回值的描述

![img](https://pic2.zhimg.com/80/v2-39a7150cd1dd36eecf1dba2a3ad84239_720w.jpg)

## 五、函数签名

## 5.1、最小签名

![img](https://pic2.zhimg.com/80/v2-e0e51dfd75ca438b0c36801729eb7279_720w.png)

最小签名用于求解在一个**有向图**中从一个start_vid到一个end_vid的最短路径。

**示例：**

![img](https://pic4.zhimg.com/80/v2-aa5a0e288ae5cd228e72da4b3cecbd83_720w.jpg)

## 5.2、pgr_dikstra一到一

![img](https://pic1.zhimg.com/80/v2-6341edd4ed28e81198fa604a554d7c88_720w.png)

此签名用于在**有向图**或**无向图**中找到从一个顶点到另一个顶点的最短路径：

- 如果不设置directed参数的值，或将该值设置为true，则计算过程考虑图的方向
- 如果设置directed参数的值为false，则计算过程不考虑图的方向

**示例：**

![img](https://pic3.zhimg.com/80/v2-63ba3ec6358618990bfe343d547b0f56_720w.jpg)

## 5.3、pgr_dijkstra一到多

![img](https://pic2.zhimg.com/80/v2-4b5fba2e6901160bb3d94edbc8b8e955_720w.png)

该签名用于找到从一个start_vid顶点到end_vids中的每一个end_vid顶点的最短路径：

- 如果不设置directed参数的值，或将该值设置为true，则计算过程考虑图的方向。
- 如果设置directed参数的值为false，则计算过程不考虑图的方向。

使用此签名，将加载一次图形，并在起始顶点固定的情况下之执行一到一的pgr_dijkstra，当到达所有end_vids时停止。

- 结果等同于一到一pgr_dijkstra的结果的集合。
- 结果中的end_vid字段用于区分它属于哪条路径。

**示例：**

![img](https://pic2.zhimg.com/80/v2-efb896da66c1b3af87f9fa6a42c712a5_720w.jpg)

## 5.4、pgr_dijkstra多到一

![img](https://pic1.zhimg.com/80/v2-158c27619308d41df221cd95d238f838_720w.png)

该签名用于找寻start_vids中的每一个start_vid顶点到一个end_vid的最短路径：

- 如果不设置directed参数的值，或将该值设置为true，则计算过程考虑图的方向。
- 如果设置directed参数的值为false，则计算过程不考虑图的方向。

使用此签名，将加载一次图形，并在结束顶点固定的情况下之执行一到一的pgr_dijkstra。

- 结果等同于一到一pgr_dijkstra的结果的集合。
- 结果中的start_vid字段用于区分它属于哪条路径。

**示例：**

![img](https://pic2.zhimg.com/80/v2-9fec60df4086cde551fbc961d45d3e91_720w.jpg)

## 六、额外的示例

## 6.1、在有向图中基于cost字段和reverse_cost字段的查询

![img](https://pic4.zhimg.com/80/v2-4cb473cb9cff727acf816659dd04b193_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-27a70bbc9b472d5065cbcceba4fcbb3d_720w.jpg)

![img](https://pic4.zhimg.com/80/v2-f86e306b091a57d2c50c23af6e538a4b_720w.jpg)

## 6.2、在无向图中基于cost字段和reverse_cost字段的查询

![img](https://pic4.zhimg.com/80/v2-b1b8130e659a266c21f3e6508aa791c7_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-e7d95deb21d4256ec36d7234bdfcd4ac_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-3d02a1dd2fb46e79f9c24bca58927fa4_720w.jpg)

## 6.3、在有向图上基于cost字段的查询

![img](https://pic4.zhimg.com/80/v2-a96777315d3bc4fc45e95a09a5b91e3f_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-15f7306960adf17907ff3fde7a019324_720w.jpg)

## 6.4、在无向图上基于cost字段的查询

![img](https://pic3.zhimg.com/80/v2-ee5093303e97fbb79b6df4d795fe19de_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-95b099700fbe28578d03dae9ba828f0e_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-e094ce75a706370e98b700ce9fe913fc_720w.jpg)