# PostGIS教程十九：维数扩展的9交集模型

# 一、什么是维数扩展的9交集模型

  "**维数扩展的9交集模型**-[Dimensionally Extended 9-Intersection Model](http://en.wikipedia.org/wiki/DE-9IM)"（**DE9IM**）是一个用于建模两个空间对象如何交互的框架。

  首先，每个空间对象都具有：

- 内部（interior)
- 边界（boundary）
- 外部（exterior）

![_images/de9im1.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im1.jpg)

  **内部**是以环为边界的里面的那一部分；**边界**是环本身；**外部**是边界外的一切。

  对于线性要素，**内部**、**边界**和**外部**不太为人所知:

![_images/de9im2.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im2.jpg)

  **内部**是以端点为界限的线的那一部分；**边界**是线性要素的端点；**外部**是平面中除**内部**和**边界**外的所有其他部分。

  对于点来说，事件更奇怪：**内部**是点，**边界**是空集，**外部**是平面上除点以外的所有其他部分。

  使用这些**内部**、**外部**和**边界**的定义，任何一对空间要素之间的关系都可以用一对要素的**内部**/**边界**/**外部**/之间九个可能的交集的维数来表征。

![_images/de9im3.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im3.jpg)

  对于上例中的多边形，**内部**的交集是二维区域，因此矩阵的对应部分用"2"填充。**边界**仅在零维点处相交，因此对应矩阵部分用"0"填充。

  当两个几何图形的这三个部分（**内部**，**边界**，**外部**）之间没有交集时，将用"**F**"填充矩阵中对应的部分。

  下面是另一个示例，关于线串的一部分和多边形相交的例子：

![_images/de9im4.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im4.jpg)

  关于它们的交集的**DE9IM矩阵**如下：

![_images/de9im5.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im5.jpg)

  请注意，以上两个要素的**边界**实际上根本不相交（线的端点与多边形的**内部**相交，而不是与多边形的**边界**相交，反之亦然），因此B/B单元用"F"填充。

  虽然让人从视觉上填写**DE9IM矩阵**很有趣，但如果计算机能够做到这一点就更好了，这就是**ST_Relate**函数的作用。

  前面的示例可以使用简单的矩形和直线进行简化，其空间关系与上面的多边形和线串的空间关系相同：

![_images/de9im6.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im6.jpg)

  我们可以使用SQL生成DE9IM信息：

```sql
SELECT ST_Relate(
         'LINESTRING(0 0, 2 0)',
         'POLYGON((1 -1, 1 1, 3 1, 3 -1, 1 -1))'
       );
```

![img](https://img-blog.csdnimg.cn/20190226134045512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  答案（**1010F0212**）与我们视觉上计算的结果相同，但以9个字符的字符串形式返回。将结果以三行的形式呈现：

![img](https://img-blog.csdnimg.cn/20190219152948831.png)

  但是，**DE9IM矩阵**的强大之处不在于生成它们，而在于使用它们作为匹配参数来查找彼此之间具有特定关系的几何图形。

# 二、查找具有特定关系的几何图形

  首先，在数据库中加入如下数据：

```sql
CREATE TABLE lakes ( id serial primary key, geom geometry );
CREATE TABLE docks ( id serial primary key, good boolean, geom geometry );
INSERT INTO lakes ( geom )
  VALUES ( 'POLYGON ((100 200, 140 230, 180 310, 280 310, 390 270, 400 210, 320 140, 215 141, 150 170, 100 200))');
INSERT INTO docks ( geom, good )
  VALUES
        ('LINESTRING (170 290, 205 272)',true),
        ('LINESTRING (120 215, 176 197)',true),
        ('LINESTRING (290 260, 340 250)',false),
        ('LINESTRING (350 300, 400 320)',false),
        ('LINESTRING (370 230, 420 240)',false),
        ('LINESTRING (370 180, 390 160)',false);
```

![img](https://img-blog.csdnimg.cn/20190226134939793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  假设我们有一个湖泊（Lakes）和码头（Docks）的数据模型，进一步假设码头必须位于湖泊内部，并且必须在一端接触到湖泊的边界。我们能在数据库中找到所有符合这一规则的码头吗？

![_images/de9im7.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im7.jpg)

  我们的合法码头具有以下特点：

- 它们的**内部**与湖泊**内部**有一个线性（一维）相交
- 它们的**边界**与湖泊**内部**有一个点（0维）相交
- 它们的**边界**与湖泊**边界**也有一个点（0维）相交
- 它们的**内部**与湖泊**外部**没有相交（F）

  所以它们的**DE9IM矩阵**看起来像这样：

![_images/de9im8.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im8.jpg)

  因此，要找到所有符合规则的码头，我们需要先找到所有与湖泊相交的码头，然后再从该集合中找到符合具体规则的所有码头。

```java
SELECT docks.*
FROM docks JOIN lakes ON ST_Intersects(docks.geom, lakes.geom)
WHERE ST_Relate(docks.geom, lakes.geom, '1FF00F212');
-- Answer: our two good docks
```

![img](https://img-blog.csdnimg.cn/2019022810312063.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  注意，ST_Relate的三参数版本的使用，如果前两个几何图形参数的关系与第三个**DE9IM模型**参数匹配，则返回ture；如果不匹配，则返回false。

  另外，对于更松散的匹配搜索，第三个参数允许DE9IM数据模型字符串使用通配符：

- "*"表示"此单元格中的任何值都可以接受"
- "T"表示"任何非假值（0、1或2）都可以接受"

  例如，我们在示例图形中添加一个与湖泊边界具有二维相交的码头：

```sql
INSERT INTO docks ( geom, good )
  VALUES ('LINESTRING (140 230, 150 250, 210 230)',true);
```

![img](https://img-blog.csdnimg.cn/2019022810431053.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/de9im9.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im9.jpg)

  如果要将这个新增的码头在ST_Relate函数检查中被视为符合规则，则需要更改ST_Relate函数的第三个参数。

  因为要使码头**内部**和湖泊**边界**的相交可以是1（我们的新情况）或F（我们的原始情况）。因此，我们使用"*"通配符覆盖所有情况。

![_images/de9im10.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im10.jpg)

  SQL语句如下所示：

```sql
SELECT docks.*
FROM docks JOIN lakes ON ST_Intersects(docks.geom, lakes.geom)
WHERE ST_Relate(docks.geom, lakes.geom, '1*F00F212');
-- Answer: our (now) three good docks
```

![img](https://img-blog.csdnimg.cn/20190228105242108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 三、数据质量测试

  **TIGER数据**在准备时经过仔细的质量控制，因此我们希望我们的数据也符合严格的标准。例如：任何**人口普查块**（census blocks）都不应与任何其他**人口普查块**重叠。我们能对我们的数据进行测试吗？

![_images/de9im11.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im11.jpg)

  当然！

```sql
SELECT a.gid, b.gid
FROM nyc_census_blocks a, nyc_census_blocks b
WHERE ST_Intersects(a.geom, b.geom)
  AND ST_Relate(a.geom, b.geom, '2********')
  AND a.gid != b.gid
LIMIT 10;
-- Answer: 10, there's some funny business
```

![img](https://img-blog.csdnimg.cn/20190228110437132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  同样，我们预计街道数据都是有尾节点的，也就是说，我们预计相交点只发生在街道直线的末端，而不是中点。

![_images/de9im12.jpg](https://postgis.net/workshops/postgis-intro/_images/de9im12.jpg)

  我们可以通过查找是否有相交但**边界**之间的交点不是零维的街道（也就是，线端点之间没有接触）来测试这一点：

```sql
SELECT a.gid, b.gid
FROM nyc_streets a, nyc_streets b
WHERE ST_Intersects(a.geom, b.geom)
  AND NOT ST_Relate(a.geom, b.geom, '****0****')
  AND a.gid != b.gid
LIMIT 10;
-- Answer: This happens, so the data is not end-noded.
```

![img](https://img-blog.csdnimg.cn/20190228111225228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 四、本文涉及的函数

![img](https://img-blog.csdnimg.cn/20190228111630712.png)