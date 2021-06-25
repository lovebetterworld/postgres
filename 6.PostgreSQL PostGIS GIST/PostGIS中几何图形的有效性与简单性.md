# PostGIS中几何图形的有效性与简单性

作者：不睡觉的怪叔叔

地址：https://zhuanlan.zhihu.com/p/117267292



# 一、什么是几何图形的简单性与有效性？

​    关于几何图形的**有效性**可以查看这篇文章：

[PostGIS教程十五：几何图形的有效性](https://zhuanlan.zhihu.com/p/63526306)

​    几何图形的**简单性**可以理解为几何图形比较简单整齐，不会自己与自己重叠，不繁杂。

​    PostGIS遵循OGC的OpenGIS规范，它要求被各类方法操作的几何图形要既是简单的又是有效的，但是我通过实际操作发现**其实只要几何图形是有效的就能被操作，非简单的几何图形也能被操作处理。**

​    在PostGIS中可以通过**[ST_IsSimple()](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/ST_IsSimple.html)**和**[ST_IsValid()](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/ST_IsValid.html)**这两个方法分别判断几何图形的简单性和有效性。

# 二、点的简单性与有效性

## 2.1、单点（POINT）的简单性与有效性

​    单个点（**Point**）肯定是简单的且有效的，因为一个点孤零零的肯定是简单、有效的。

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'POINT(0 0)'
)), ST_IsValid(ST_GeometryFromText(
         'POINT(0 0)'
))
```

![img](https://pic3.zhimg.com/80/v2-acfa85d2fae38fcaa9b2a6a05b2b94c6_720w.jpg)

## 2.2、多点（MULTIPOINT）的简单性与有效性

   多个点（**MultiPoint**）肯定是有效的，但不一定是简单的。

​    如果多点中有两个或两个以上的点重合（也就是坐标一致），那么它就不是简单的，但是确是有效的。

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'MULTIPOINT((0 0),(0 0))'
)), ST_IsValid(ST_GeometryFromText(
         'MULTIPOINT((0 0),(0 0))'
))
```

# 三、线串的简单性与有效性

## 3.1、单线串（LINESTRING）的简单性与有效性

​    单线串（LINESTRING）如果有重叠、相交就不是简单的（除了端点相交，端点相交就说明这条线串是闭合的，但它是简单的）：

![img](https://pic1.zhimg.com/80/v2-c65483fe45b7dc2842a4a05f34b2542c_720w.jpg)

​    另外，单线串都是有效的。

​    比如构建一个如下所示的线串：

![img](https://pic3.zhimg.com/80/v2-d19e96b0f3fc559182d3107e8065f3ce_720w.jpg)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'LINESTRING(0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0)'
)), ST_IsValid(ST_GeometryFromText(
         'LINESTRING(0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0)'
))
```

![img](https://pic4.zhimg.com/80/v2-6985bc0786d863339e504997356c5dab_720w.jpg)

​    可以发现它是有效的，但不是简单的！

## 3.2、多线串（MULTILINESTRING）的简单性与有效性

多线串（MULTILINESTRING）只要它的元素（LINESTRING）都是简单的，且两个元素只在某个点相切，那么它就是简单。

![img](https://pic1.zhimg.com/80/v2-eb610f397c10641b6e377592a37e276c_720w.jpg)

​        上图中（f）多线串的两个子元素只在一个点相切，所以它是简单的，而（g）多线串的两个子元素在一个点相交，因此它不是简单的。

​    和单线串一样，多线串总是有效的。

​    示例1（多线串的两个子元素相切于一点）：

![img](https://pic3.zhimg.com/80/v2-c73e494c9e2bd8cc3b179959ff4b0f2a_720w.jpg)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'MULTILINESTRING((0 0, 50 50), (20 0, 50 50))'
)), ST_IsValid(ST_GeometryFromText(
         'MULTILINESTRING((0 0, 50 50), (20 0, 50 50))'
));
```

​     结果：

![img](https://pic1.zhimg.com/80/v2-97f134fedbe02efd1a0e658b246df344_720w.jpg)

​    示例2（多线串的两个子元素相交于一点）：

![image-20210625155943599](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210625155943599.png)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'MULTILINESTRING((0 0, 50 50), (20 0, 0 50))'
)), ST_IsValid(ST_GeometryFromText(
         'MULTILINESTRING((0 0, 50 50), (20 0, 0 50))'
));
```

​    结果：

![img](https://pic4.zhimg.com/80/v2-28d10736a2f70b1d89ab09f51e2e5ff3_720w.jpg)

# 四、多边形的简单性与有效性

## 4.1、单多边形（POLYGON）的简单性与有效性

​    如果违反以下规则，那么对应的单多边形就不是有效的。

- 多边形的环必须闭合
- 内环应该处于外环的内部
- 环不能自相交（它们不能相互接触，也不能交叉）
- 环不能与其他环接触，除非在某个点相切（只能有一个在一个点相切）

​    另外，多边形的环只要不自相交，则该多边形就是简单的。

![image-20210625155953321](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210625155953321.png)

​    上图中（i）的内环和外环在一个点相切，所以它也是简单和有效的；（j）的内环和外环在两个点相切所以是简单的，但不是有效的；（I）的外环自相交了，所以它不是简单的，也不是有效的；（m）的内环都跑到外环的外面了，所以是简单的，但不是有效的。

​    示例1（多边形内环和外环在一个点相切）：

![img](https://pic1.zhimg.com/80/v2-253c666034e9b175c9864238cbec7bc8_720w.jpg)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (2 2, 5 2, 20 20, 7 10, 2 2))'
)), ST_IsValid(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (2 2, 5 2, 20 20, 7 10, 2 2))'
));
```

​    结果：

![img](https://pic3.zhimg.com/80/v2-a5d05b26515f716b7c59b46da4b4133a_720w.jpg)

​    示例2（多边形内环和外环互相接触）：

![img](https://pic4.zhimg.com/80/v2-cc86fb4a3e2b35e6df528091a5944e57_720w.jpg)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (2 2, 5 2, 20 20, 5 20, 7 10, 2 2))'
)), ST_IsValid(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (2 2, 5 2, 20 20, 5 20, 7 10, 2 2))'
));
```

​    结果：

![img](https://pic4.zhimg.com/80/v2-1fc651a2b63af609c367990e78723b87_720w.jpg)

​    示例3（内环位于外环的外部）：

![img](https://pic1.zhimg.com/80/v2-2c9a6a7bdfc4b3b6674d2ef1300d4974_720w.jpg)

```sql
 SELECT ST_IsSimple(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (30 0, 40 0, 40 10, 30 10, 30 0))'
)), ST_IsValid(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 0 20, 0 0), (30 0, 40 0, 40 10, 30 10, 30 0))'
));
```

![img](https://pic1.zhimg.com/80/v2-f1fffc68b721736de348d9317816fdb0_720w.jpg)

​    示例4（外环自相交，内环位于外环的外部）：

![img](https://pic4.zhimg.com/80/v2-63411d82b5d7894d304ac09b6b0ec12f_720w.jpg)

```sql
SELECT ST_IsSimple(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 10 20, 10 30, 10 20, 0 20, 0 0), (30 0, 40 0, 40 10, 30 10, 30 0))'
)), ST_IsValid(ST_GeometryFromText(
         'POLYGON((0 0, 20 0, 20 20, 10 20, 10 30, 10 20, 0 20, 0 0), (30 0, 40 0, 40 10, 30 10, 30 0))'
));
```

​    结果：

![img](https://pic3.zhimg.com/80/v2-c153151eb8e306a2729164e09de72c42_720w.jpg)

## 4.2、多多边形（MULTIPOLYGON）的简单性与有效性

​    **多多边形里**只要各个子元素（单多边形）是简单的、有效的，而且子元素之间只在有限的点上接触，那么它就是简单的、有效的。

![img](https://pic4.zhimg.com/80/v2-a64b3a85ebcc63898d3e851dcc31048f_720w.jpg)

上图中**（p)**中的两个子多边形因为是切点相交，且相交点是有限个，所以是有效的。

# 五、预防与修复无效几何图形

​    因为只要几何图形是有效的，那么PostGIS中的方法就能对该几何图形进行操作处理，所以我们只要保证几何图形是有效的（不用太关心简单性）那么就可以进行我们的业务工作。

​    前面提到**[ST_IsValid()](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/ST_IsValid.html)**方法能够检测几何图形的有效性。但是默认情况下，PostGIS不会在几何图形输入PostGIS时应用这种有效性检查，因为对于复杂的几何图形，尤其是多边形，有效性测试需要大量的CPU时间。如果不信任数据源，则可以通过添加**CHECK约束**（即用户定义的完整性约束）来手动对表强制执行这样的有效性检查：

```sql
ALTER TABLE mytable ADD CONSTRAINT geometry_valid_check CHECK (ST_IsValid(the_geom));
```

​    OGC规范的某些方法也不允许几何图形有z值和m值，因此有z值或m值的几何图形数据可能会导致某些方法执行失败。另外，[AddGeometryColumn()](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/AddGeometryColumn.html)方法允许我们指定表中空间信息的维度。

​    关于无效几何图形的修复，下面这篇文章可以参考下面这篇文章：

[PostGIS教程十五：几何图形的有效性](https://zhuanlan.zhihu.com/p/63526306)

# 六、本文参考资料

- [Chapter 4. Using PostGIS: Data Management and Queries](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/using_postgis_dbmanagement.html%23OGC_Validity) 
- [ST_IsValid](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/ST_IsValid.html) 
- [ST_IsSimple](https://link.zhihu.com/?target=http%3A//postgis.net/docs/manual-3.0/ST_IsSimple.html) 