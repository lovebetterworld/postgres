# PostGIS教程十七：相等

在处理几何图形时确定相等可能很困难。PostGIS支持三种不同的函数，可以用来确定不同级别的相等。为了说明这些函数，我们将使用以下多边形。

![_images/polygon-table.png](https://postgis.net/workshops/postgis-intro/_images/polygon-table.png)

  使用以下命令加载这些多边形：

```sql
CREATE TABLE polygons (id integer, name varchar, poly geometry);
INSERT INTO polygons VALUES
  (1, 'Polygon 1', 'POLYGON((-1 1.732,1 1.732,2 0,1 -1.732,
      -1 -1.732,-2 0,-1 1.732))'),
  (2, 'Polygon 2', 'POLYGON((-1 1.732,-2 0,-1 -1.732,1 -1.732,
      2 0,1 1.732,-1 1.732))'),
  (3, 'Polygon 3', 'POLYGON((1 -1.732,2 0,1 1.732,-1 1.732,
      -2 0,-1 -1.732,1 -1.732))'),
  (4, 'Polygon 4', 'POLYGON((-1 1.732,0 1.732, 1 1.732,1.5 0.866,
      2 0,1.5 -0.866,1 -1.732,0 -1.732,-1 -1.732,-1.5 -0.866,
      -2 0,-1.5 0.866,-1 1.732))'),
  (5, 'Polygon 5', 'POLYGON((-2 -1.732,2 -1.732,2 1.732,
      -2 1.732,-2 -1.732))');
```

![img](https://img-blog.csdnimg.cn/20190215101530469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 一、精确相等

  精确相等是通过按顺序逐个比较两个几何图形的顶点来确定的，以确保它们在位置上是相同的。下面的例子说明了这种方法的有效性是如何受到限制的。

```sql
SELECT a.name, b.name, CASE WHEN ST_OrderingEquals(a.poly, b.poly)
    THEN 'Exactly Equal' ELSE 'Not Exactly Equal' end
FROM polygons as a, polygons as b;
```

![img](https://img-blog.csdnimg.cn/20190215102744901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  在此示例中，多边形仅与自身相等，而不等于其他看似相等的多边形（如在多边形1到3的情况下）。对于多边形1、2和3，顶点处于相同的位置，但定义顺序不同。多边形4在六边形边上有共线（因此是冗余的）顶点，导致与多边形1不相等。

 

# 二、空间相等

  正如我们在上面看到的，精确的相等并没有考虑到几何图形的空间性质。有一个名为**ST_Equals**的函数，可用于测试几何图形的空间相等性或等价性。

```sql
SELECT a.name, b.name, CASE WHEN ST_Equals(a.poly, b.poly)
    THEN 'Spatially Equal' ELSE 'Not Equal' end
  FROM polygons as a, polygons as b;
```

![img](https://img-blog.csdnimg.cn/20190215103521620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  这些结果更符合我们对相等的直觉理解。多边形1到4被认为是相等的，因为它们包含相同的区域。请注意，无论是绘制多边形的方向、定义多边形的起点，还是使用的点的个数的差异在这里都不重要。重要的是多边形包含相同的空间区域。

 

# 三、等边界框

  在最坏的情况下，需要精确相等来比较几何图形中的每个顶点以确定相等。这可能会比较慢，并且可能不适合数量大的几何图形。为了更快地进行比较，提供了等边界运算符 ' **=** ' 。这仅在边界框（矩形）上操作，确保几何图形占用相同的二维范围，但不一定占用相同的空间。

```sql
SELECT a.name, b.name, CASE WHEN a.poly = b.poly
    THEN 'Equal Bounds' ELSE 'Non-equal Bounds' end
FROM polygons as a, polygons as b;
```

![img](https://img-blog.csdnimg.cn/20190215110205932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  如你所见，我们所有在空间上相等的几何图形也有相等的边界框。不幸的是，在此测试中返回的多边形5也是相等的，因为它与其他几何图形一样具有相同的边界框。那这有什么用呢？虽然这将在稍后详细介绍，但简短的答案是，这允许使用空间索引，在连接或筛选数据时，可以将大量的比较集快速减少为更易于处理的子集。