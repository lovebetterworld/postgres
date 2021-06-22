 在90%的情况下，"为什么我的查询给了我一个'TopologyException'错误"的问题的答案是"一个或多个输入的几何图形是无效的"，这就引出了这样一个问题:几何图形"**无效**"是什么意思？我们为什么要关心它?

# 一、什么是有效性

  对于定义有界区域并需要大量结构的多边形来说，它的几何图形**有效性**是最重要的。线串非常简单，不会无效，点也不会无效。

  多边形**有效性**的一些规则很明显，而另一些规则是任意的（事实上，是任意的）。

- 多边形的环必须闭合
- 定义孔洞的环应该是在外部边界的环内
- 环不能自相交（它们不能相互接触，也不能交叉）
- 环不能与其他环接触，除非在某个点接触

  最后两条规则属于任意类别。定义多边形的其他规则也是自洽合理的，但是上面的规则是PostGIS所遵循的[OGC](https://postgis.net/workshops/postgis-intro/glossary.html#term-ogc) [SFSQL](https://postgis.net/workshops/postgis-intro/glossary.html#term-sfsql)标准所使用的多边形有效性的规则。

  规则之所以重要，是因为几何图形的计算依赖于输入的几何图形的结构。可以构建没有结构假设的算法，但这些程序往往非常慢，因为任何无结构程序的第一步都是分析输入并在其中构建结构。

  这里有一个解释为什么几何图形的结构重要的例子。首先这个多边形是**无效的**：

```sql
POLYGON((0 0, 0 1, 2 1, 2 2, 1 2, 1 0, 0 0));
```

  在此图中，你可以更清楚地看到无效的原因：

![_images/figure_eight.png](https://postgis.net/workshops/postgis-intro/_images/figure_eight.png)

  这个多边形的外环实际上是一个数字8的形状，中间有一个**自交点**。图形程序成功地渲染了多边形填充，使其在视觉上看起来是一个"区域"：两个一个单位的正方形，因此多边形总面积为两个单位的面积。

  让我们看看PostGIS数据库认为多边形的面积是多少：

```sql
SELECT ST_Area(ST_GeometryFromText(
         'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
       ));
```

![img](https://img-blog.csdnimg.cn/20190124095148150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190124095201814.png)

  这里发生了什么？计算面积的算法假设环不自相交。程序始终计算位于边界线的一侧的区域的面积。

然而，在我们的（表现不佳）的数字8中，对于一个叶部分，图形区域位于边界线的右侧，而对于另一个叶部分，图形区域在边界线的左侧。这将导致为每个叶部分计算的面积互相抵消（一个为1，另一个为-1），因此结果为"0面积"。

# 二、检测有效性

  在前面的示例中，我们可以轻易发现一个多边形是无效的。然而我们如何在一个包含数百万个几何图形的表中检测无效？答案是使用**ST_IsValid(geometry)**函数：

```sql
SELECT ST_IsValid(ST_GeometryFromText(
         'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
       ));
```

![img](https://img-blog.csdnimg.cn/20190124100738900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190124100751599.png)

   现在我们知道这个图形是无效的，但是我们不知道为什么无效。我们可以使用**ST_IsValidReason(geometry)**函数来查找无效的原因：

```sql
SELECT ST_IsValidReason(ST_GeometryFromText(
         'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
       ));
```

![img](https://img-blog.csdnimg.cn/20190124101516189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190124101400937.png)

  请注意，除了原因（**自相交**），图形自相交的坐标（coordinate(1 1))也被返回了。

  我们也可以使用ST_IsValid(geometry)函数来测试数据表：

```sql
-- Find all the invalid polygons and what their problem is
SELECT name, boroname, ST_IsValidReason(geom)
FROM nyc_neighborhoods
WHERE NOT ST_IsValid(geom);
```

![img](https://img-blog.csdnimg.cn/20190124102004795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190124102015909.png)

 

# 三、修复无效的图形

  首先，坏消息是：没有100%确定的方法来修复无效的几何图形。最坏的情况是使用ST_IsValid(geometry)函数识别它们，然后将它们移动另一张表，导出该表，然后在外部修复它们。

  下面是SQL的一个示例，它将无效的几何图形从原表转移到另一张表中（注意，同时要：

```sql
-- Side table of invalids
CREATE TABLE nyc_neighborhoods_invalid AS
SELECT * FROM nyc_neighborhoods
WHERE NOT ST_IsValid(geom);
-- Remove them from the main table
DELETE FROM nyc_neighborhoods
WHERE NOT ST_IsValid(geom);
```

![img](https://img-blog.csdnimg.cn/20190125153536970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  在视觉上修复无效几何图形的一个好工具是**OpenJump**([http://openjump.org](http://openjump.org/))，它在**Tools->QA->Validate Selected Layers**.下包含一个验证程序。

  现在好消息是：可以使用以下任何一种方法在数据库中修复很大一部分的缺陷：

- ST_MakeValid函数
- ST_Buffer函数

## 3.1、ST_MakeValid

  **ST_MakeValid**尝试在不对输入几何图形进行更改的情况下修复缺陷。不会删除或移动任何顶点，只需重新排列对象的结构即可。对于清晰但无效的数据来说，这个函数非常适用，对于杂乱无章且无效的数据来说，这个函数可能并不适用。

```sql
-- Fix the invalid figure-8 polygon
SELECT ST_AsText(ST_MakeValid(
         'POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'
       ));
```

![img](https://img-blog.csdnimg.cn/20190125155646510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190125155658133.png)

  ST_MakeValid成功地将几何图形"8"转换为表示相同面积的multi-polygon。

## 3.2、ST_Buffer

  使用缓冲区技巧清理时，可以利用缓冲区的生成方式来达到修复几何图形的目的：缓冲区几何图形是全新的几何图形，由关于原始图形的偏移线构建。如果不偏移原始线（零），则新几何图形在结构上将与原始几何图形相同，但由于它是使用[OGC](https://postgis.net/workshops/postgis-intro/glossary.html#term-ogc)拓扑规则构建的，因此它将是有效的。

  例如，这里有一个典型的无效现象——"**香蕉多边形**" —— 一个环，它包围着一个区域，但弯曲着接触自己，留下一个"孔洞（hole）"，实际上并不是一个孔洞。

```sql
POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))
```

![_images/banana.png](https://postgis.net/workshops/postgis-intro/_images/banana.png)

  在多边形上计算零偏移缓冲区将返回有效的[OGC](https://postgis.net/workshops/postgis-intro/glossary.html#term-ogc)多边形，该多边形由在一点接触的外环和内环组成。

```sql
SELECT ST_AsText(
         ST_Buffer(
           ST_GeometryFromText('POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))'),
           0.0
         )
       );
```

![img](https://img-blog.csdnimg.cn/20190125162034777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190125162059735.png)

 