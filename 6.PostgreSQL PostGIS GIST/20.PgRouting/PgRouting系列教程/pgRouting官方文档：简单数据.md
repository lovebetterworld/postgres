- [pgRouting官方文档：简单数据 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/85913987)

原文地址：[Sample Data - pgRouting Manual (2.6)](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/2.6/en/sampledata.html)

本文的数据是其他官方文档的实验数据。

文档提供了基于一个小示例道路网络数据的简单查询示例。要能够执行查询示例，请运行以下SQL命令来创建一个具有小网络数据集的表。

## 一、**创建表**

```sql
CREATE TABLE edge_table (
    id BIGSERIAL,
    dir character varying,
    source BIGINT,
    target BIGINT,
    cost FLOAT,
    reverse_cost FLOAT,
    capacity BIGINT,
    reverse_capacity BIGINT,
    category_id INTEGER,
    reverse_category_id INTEGER,
    x1 FLOAT,
    y1 FLOAT,
    x2 FLOAT,
    y2 FLOAT,
    the_geom geometry
);
```

## 二、**插入数据**

```sql
INSERT INTO edge_table (
    category_id, reverse_category_id,
    cost, reverse_cost,
    capacity, reverse_capacity,
    x1, y1,
    x2, y2) VALUES
(3, 1,    1,  1,  80, 130,   2,   0,    2, 1),
(3, 2,   -1,  1,  -1, 100,   2,   1,    3, 1),
(2, 1,   -1,  1,  -1, 130,   3,   1,    4, 1),
(2, 4,    1,  1, 100,  50,   2,   1,    2, 2),
(1, 4,    1, -1, 130,  -1,   3,   1,    3, 2),
(4, 2,    1,  1,  50, 100,   0,   2,    1, 2),
(4, 1,    1,  1,  50, 130,   1,   2,    2, 2),
(2, 1,    1,  1, 100, 130,   2,   2,    3, 2),
(1, 3,    1,  1, 130,  80,   3,   2,    4, 2),
(1, 4,    1,  1, 130,  50,   2,   2,    2, 3),
(1, 2,    1, -1, 130,  -1,   3,   2,    3, 3),
(2, 3,    1, -1, 100,  -1,   2,   3,    3, 3),
(2, 4,    1, -1, 100,  -1,   3,   3,    4, 3),
(3, 1,    1,  1,  80, 130,   2,   3,    2, 4),
(3, 4,    1,  1,  80,  50,   4,   2,    4, 3),
(3, 3,    1,  1,  80,  80,   4,   1,    4, 2),
(1, 2,    1,  1, 130, 100,   0.5, 3.5,  1.999999999999,3.5),
(4, 1,    1,  1,  50, 130,   3.5, 2.3,  3.5,4);

UPDATE edge_table SET the_geom = st_makeline(st_point(x1,y1),st_point(x2,y2)),
dir = CASE WHEN (cost>0 AND reverse_cost>0) THEN 'B'   -- both ways
           WHEN (cost>0 AND reverse_cost<0) THEN 'FT'  -- direction of the LINESSTRING
           WHEN (cost<0 AND reverse_cost>0) THEN 'TF'  -- reverse direction of the LINESTRING
           ELSE '' END;                                -- unknown
```

## 三、**拓扑**

在测试路由函数之前，请使用此查询创建拓扑（填充source和target列）：

```sql
SELECT pgr_createTopology('edge_table',0.001);
```

## 四、**兴趣点**

- 图形之外的点
- 与[withPoints](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/2.6/en/withPoints-family.html%23withpoints)函数族一起使用

```sql
CREATE TABLE pointsOfInterest(
    pid BIGSERIAL,
    x FLOAT,
    y FLOAT,
    edge_id BIGINT,
    side CHAR,
    fraction FLOAT,
    the_geom geometry,
    newPoint geometry
);

INSERT INTO pointsOfInterest (x, y, edge_id, side, fraction) VALUES
(1.8, 0.4,   1, 'l', 0.4),
(4.2, 2.4,  15, 'r', 0.4),
(2.6, 3.2,  12, 'l', 0.6),
(0.3, 1.8,   6, 'r', 0.3),
(2.9, 1.8,   5, 'l', 0.8),
(2.2, 1.7,   4, 'b', 0.7);
UPDATE pointsOfInterest SET the_geom = st_makePoint(x,y);

UPDATE pointsOfInterest
    SET newPoint = ST_LineInterpolatePoint(e.the_geom, fraction)
    FROM edge_table AS e WHERE edge_id = id;
```

## 五、**限制**

和[pgr_trsp - Turn Restriction Shortest Path（TRSP）](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/2.6/en/pgr_trsp.html%23trsp)函数一起使用。

```sql
CREATE TABLE restrictions (
    rid BIGINT NOT NULL,
    to_cost FLOAT,
    target_id BIGINT,
    from_edge BIGINT,
    via_path TEXT
);

INSERT INTO restrictions (rid, to_cost, target_id, from_edge, via_path) VALUES
(1, 100,  7,  4, NULL),
(1, 100, 11,  8, NULL),
(1, 100, 10,  7, NULL),
(2,   4,  8,  3, 5),
(3, 100,  9, 16, NULL);
```

## 六、**分类**

和[Flow函数族](https://link.zhihu.com/?target=http%3A//docs.pgrouting.org/2.6/en/flow-family.html%23maxflow)一起使用。

```sql
/*
CREATE TABLE categories (
    category_id INTEGER,
    category text,
    capacity BIGINT
);

INSERT INTO categories VALUES
(1, 'Category 1', 130),
(2, 'Category 2', 100),
(3, 'Category 3',  80),
(4, 'Category 4',  50);
*/
```

## 七、**顶点表**

被用于某些不推荐的函数

```sql
-- TODO check if this table is still used
CREATE TABLE vertex_table (
    id SERIAL,
    x FLOAT,
    y FLOAT
);
INSERT INTO vertex_table VALUES
(1,2,0), (2,2,1), (3,3,1), (4,4,1), (5,0,2), (6,1,2), (7,2,2),
(8,3,2), (9,4,2), (10,2,3), (11,3,3), (12,4,3), (13,2,4);
```

## 八、**图像**

- 红色箭头对应于edge_table中的cost > 0
- 蓝色箭头对应于edge_table中的reverse_cost > 0
- pointsOfInterest（兴趣点）在图之外

## **8.1、有向道路网，方向由cost和rever_cost决定**

在edge_table中用cost属性和reverse_cost属性决定表示道路方向（directed）的属性dir。

当使用城市道路网时，以下适合给交通工具使用（因为交通工具会受到道路方向的限制）：

![img](https://pic1.zhimg.com/80/v2-579e7a544ebf3086011b7f8f6710d000_720w.jpg)图1：有向图，由cost属性和reverse cost属性决定方向

## **8.2、无向道路网**

当使用城市道路网时，以下适合给行人使用（由于行人不会受到方向的限制，所以就不需要方向信息了）：

![img](https://pic2.zhimg.com/80/v2-ecfecc64a3efc246937c3f2ad7443e79_720w.jpg)图2：无向图

## **8.3、只使用cost属性的有向道路网**

![img](https://pic2.zhimg.com/80/v2-755c8faf14a5d8026c839fd6eafda5f1_720w.jpg)图3：有向图，只使用cost属性

## **8.4、只使用cost属性的无向道路网**

![img](https://pic3.zhimg.com/80/v2-cdc11f662d9c7730b08e656fe1847782_720w.jpg)图4：无向图，只使用cost属性

## 九、挑选&传送数据

```sql
DROP TABLE IF EXISTS customer CASCADE;
CREATE table customer (
    id BIGINT not null primary key,
    x DOUBLE PRECISION,
    y DOUBLE PRECISION,
    demand INTEGER,
    opentime INTEGER,
    closetime INTEGER,
    servicetime INTEGER,
    pindex BIGINT,
    dindex BIGINT
);


INSERT INTO customer(
  id,     x,    y, demand, opentime, closetime, servicetime, pindex, dindex) VALUES
(  0,    40,   50,     0,     0,  1236,    0,    0,    0),
(  1,    45,   68,   -10,   912,   967,   90,   11,    0),
(  2,    45,   70,   -20,   825,   870,   90,    6,    0),
(  3,    42,   66,    10,    65,   146,   90,    0,   75),
(  4,    42,   68,   -10,   727,   782,   90,    9,    0),
(  5,    42,   65,    10,    15,    67,   90,    0,    7),
(  6,    40,   69,    20,   621,   702,   90,    0,    2),
(  7,    40,   66,   -10,   170,   225,   90,    5,    0),
(  8,    38,   68,    20,   255,   324,   90,    0,   10),
(  9,    38,   70,    10,   534,   605,   90,    0,    4),
( 10,    35,   66,   -20,   357,   410,   90,    8,    0),
( 11,    35,   69,    10,   448,   505,   90,    0,    1),
( 12,    25,   85,   -20,   652,   721,   90,   18,    0),
( 13,    22,   75,    30,    30,    92,   90,    0,   17),
( 14,    22,   85,   -40,   567,   620,   90,   16,    0),
( 15,    20,   80,   -10,   384,   429,   90,   19,    0),
( 16,    20,   85,    40,   475,   528,   90,    0,   14),
( 17,    18,   75,   -30,    99,   148,   90,   13,    0),
( 18,    15,   75,    20,   179,   254,   90,    0,   12),
( 19,    15,   80,    10,   278,   345,   90,    0,   15),
( 20,    30,   50,    10,    10,    73,   90,    0,   24),
( 21,    30,   52,   -10,   914,   965,   90,   30,    0),
( 22,    28,   52,   -20,   812,   883,   90,   28,    0),
( 23,    28,   55,    10,   732,   777,    0,    0,  103),
( 24,    25,   50,   -10,    65,   144,   90,   20,    0),
( 25,    25,   52,    40,   169,   224,   90,    0,   27),
( 26,    25,   55,   -10,   622,   701,   90,   29,    0),
( 27,    23,   52,   -40,   261,   316,   90,   25,    0),
( 28,    23,   55,    20,   546,   593,   90,    0,   22),
( 29,    20,   50,    10,   358,   405,   90,    0,   26),
( 30,    20,   55,    10,   449,   504,   90,    0,   21),
( 31,    10,   35,   -30,   200,   237,   90,   32,    0),
( 32,    10,   40,    30,    31,   100,   90,    0,   31),
( 33,     8,   40,    40,    87,   158,   90,    0,   37),
( 34,     8,   45,   -30,   751,   816,   90,   38,    0),
( 35,     5,   35,    10,   283,   344,   90,    0,   39),
( 36,     5,   45,    10,   665,   716,    0,    0,  105),
( 37,     2,   40,   -40,   383,   434,   90,   33,    0),
( 38,     0,   40,    30,   479,   522,   90,    0,   34),
( 39,     0,   45,   -10,   567,   624,   90,   35,    0),
( 40,    35,   30,   -20,   264,   321,   90,   42,    0),
( 41,    35,   32,   -10,   166,   235,   90,   43,    0),
( 42,    33,   32,    20,    68,   149,   90,    0,   40),
( 43,    33,   35,    10,    16,    80,   90,    0,   41),
( 44,    32,   30,    10,   359,   412,   90,    0,   46),
( 45,    30,   30,    10,   541,   600,   90,    0,   48),
( 46,    30,   32,   -10,   448,   509,   90,   44,    0),
( 47,    30,   35,   -10,  1054,  1127,   90,   49,    0),
( 48,    28,   30,   -10,   632,   693,   90,   45,    0),
( 49,    28,   35,    10,  1001,  1066,   90,    0,   47),
( 50,    26,   32,    10,   815,   880,   90,    0,   52),
( 51,    25,   30,    10,   725,   786,    0,    0,  101),
( 52,    25,   35,   -10,   912,   969,   90,   50,    0),
( 53,    44,    5,    20,   286,   347,   90,    0,   58),
( 54,    42,   10,    40,   186,   257,   90,    0,   60),
( 55,    42,   15,   -40,    95,   158,   90,   57,    0),
( 56,    40,    5,    30,   385,   436,   90,    0,   59),
( 57,    40,   15,    40,    35,    87,   90,    0,   55),
( 58,    38,    5,   -20,   471,   534,   90,   53,    0),
( 59,    38,   15,   -30,   651,   740,   90,   56,    0),
( 60,    35,    5,   -40,   562,   629,   90,   54,    0),
( 61,    50,   30,   -10,   531,   610,   90,   67,    0),
( 62,    50,   35,    20,   262,   317,   90,    0,   68),
( 63,    50,   40,    50,   171,   218,   90,    0,   74),
( 64,    48,   30,    10,   632,   693,    0,    0,  102),
( 65,    48,   40,    10,    76,   129,   90,    0,   72),
( 66,    47,   35,    10,   826,   875,   90,    0,   69),
( 67,    47,   40,    10,    12,    77,   90,    0,   61),
( 68,    45,   30,   -20,   734,   777,   90,   62,    0),
( 69,    45,   35,   -10,   916,   969,   90,   66,    0),
( 70,    95,   30,   -30,   387,   456,   90,   81,    0),
( 71,    95,   35,    20,   293,   360,   90,    0,   77),
( 72,    53,   30,   -10,   450,   505,   90,   65,    0),
( 73,    92,   30,   -10,   478,   551,   90,   76,    0),
( 74,    53,   35,   -50,   353,   412,   90,   63,    0),
( 75,    45,   65,   -10,   997,  1068,   90,    3,    0),
( 76,    90,   35,    10,   203,   260,   90,    0,   73),
( 77,    88,   30,   -20,   574,   643,   90,   71,    0),
( 78,    88,   35,    20,   109,   170,    0,    0,  104),
( 79,    87,   30,    10,   668,   731,   90,    0,   80),
( 80,    85,   25,   -10,   769,   820,   90,   79,    0),
( 81,    85,   35,    30,    47,   124,   90,    0,   70),
( 82,    75,   55,    20,   369,   420,   90,    0,   85),
( 83,    72,   55,   -20,   265,   338,   90,   87,    0),
( 84,    70,   58,    20,   458,   523,   90,    0,   89),
( 85,    68,   60,   -20,   555,   612,   90,   82,    0),
( 86,    66,   55,    10,   173,   238,   90,    0,   91),
( 87,    65,   55,    20,    85,   144,   90,    0,   83),
( 88,    65,   60,   -10,   645,   708,   90,   90,    0),
( 89,    63,   58,   -20,   737,   802,   90,   84,    0),
( 90,    60,   55,    10,    20,    84,   90,    0,   88),
( 91,    60,   60,   -10,   836,   889,   90,   86,    0),
( 92,    67,   85,    20,   368,   441,   90,    0,   93),
( 93,    65,   85,   -20,   475,   518,   90,   92,    0),
( 94,    65,   82,   -10,   285,   336,   90,   96,    0),
( 95,    62,   80,   -20,   196,   239,   90,   98,    0),
( 96,    60,   80,    10,    95,   156,   90,    0,   94),
( 97,    60,   85,    30,   561,   622,    0,    0,  106),
( 98,    58,   75,    20,    30,    84,   90,    0,   95),
( 99,    55,   80,   -20,   743,   820,   90,  100,    0),
( 100,   55,   85,    20,   647,   726,   90,    0,   99),
( 101,   25,   30,   -10,   725,   786,   90,   51,    0),
( 102,   48,   30,   -10,   632,   693,   90,   64,    0),
( 103,   28,   55,   -10,   732,   777,   90,   23,    0),
( 104,   88,   35,   -20,   109,   170,   90,   78,    0),
( 105,    5,   45,   -10,   665,   716,   90,   36,    0),
( 106,   60,   85,   -30,   561,   622,   90,   97,    0);
```