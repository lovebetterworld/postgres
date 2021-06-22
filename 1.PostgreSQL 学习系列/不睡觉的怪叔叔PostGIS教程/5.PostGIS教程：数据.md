# PostGIS教程五：数据

教程的数据是有关**纽约市**的四个shapefile文件和一个包含社会人口经济数据的数据表。在前面一节我们已经将shapefile加载为PostGIS表，在后面我们将添加社会人口经济数据。

  下面描述了每个数据集的记录数量和表属性。这些属性值和关系是我们以后分析的基础。

   要在pgAdmin中浏览表的性质或属性，请在高亮显示的表上单击鼠标右键，然后选择**属性**（property）。

# 一、nyc_census_blocks

  **人口普查块**（census block）是报告人口普查数据的最小地理单位。

  所有更高级别的人口普查的地理单位（块组-block groups、区域-tracts、市区-metro areas、县-counties等）都可以通过对人口普查块的联合（Union）而建立起来。

  我们已经附加了一些人口统计数据到我们的nyc_census_blocks表里。

  记录的数目：36592

![img](https://img-blog.csdnimg.cn/20181225085628764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

![_images/nyc_census_blocks.png](https://postgis.net/workshops/postgis-intro/_images/nyc_census_blocks.png)

​             黑人人口占总人口的百分比

  **注意**：要制作这样的GIS数据，需要连接两部分信息：实际社会人口数据（text）和几何信息数据（spatial）。获取数据的方法很多，比如可以从美国人口普查局的[American FactFinder](http://factfinder.census.gov/)下载数据。

# 二、nyc_neighborhoods

  纽约有丰富的社区名称和社区范围变更的历史。社区是不遵循政府规定的界限的的社会地理结构。

  例如，**布鲁克林区**（Brooklyn）的**卡罗尔花园**（Carroll Gardens）、**红钩区**（Red Hook）和**克布尔山**（Cobble Hill）曾被统称为“**南布鲁克林**（South Brooklyn）"。

  而现在，以前被同时称为红钩的四个街区现在可以被分别称为**哥伦比亚高地**（Columbia Heights）、**西卡罗尔花园**（Carroll Gardens）或**红钩**（Red Hook）。

   记录数目：129

![img](https://img-blog.csdnimg.cn/20181225091251480.png)

![_images/nyc_neighborhoods.png](https://img-blog.csdnimg.cn/20181225092800878)

​                      纽约市的街区分布

# 三、nyc_streets

  街道线构成了这个城市的交通网络。为了区分主干道、高速公路和较小的街道，这些街道被标记为不同的类型（type）。适宜居住的地方可能位于住宅区街道附近，而不是在高速公路旁。

  记录数量：19091

![img](https://img-blog.csdnimg.cn/20190413215923226.png)

![_images/nyc_streets.png](https://postgis.net/workshops/postgis-intro/_images/nyc_streets.png)

​              纽约市街道分布

# 四、nyc_subway_stations

  地铁站将人们居住的地铁世界与地下看不见的地铁网络连接起来。

  作为公共交通系统的入口，车站位置决定了处于不同位置的市民使用地铁交通的难易程度。

  记录数目：491

![img](https://img-blog.csdnimg.cn/20181225091726455.png)

![_images/nyc_subway_stations.png](https://img-blog.csdnimg.cn/20181226090334197)

​                  纽约市地铁站的位置分布

# 五、nyc_census_sociodata

  在普查过程中收集了大量的社会人口经济数据，但仅限于普查区域较大的地理层面。

  **人口普查块**（census blocks）组合在一起形成**人口普查区域**（census tracts）（和**块组**-block groups）。

  我们收集了一些与**人口普查区域**对应的社会人口经济数据，来回答更多关于纽约市的有趣问题。

  注意：nyc_census_sociodata是一个**数据表**。在进行任何空间分析之前，我们需要将其与人口普查的地理信息联系起来。

![img](https://img-blog.csdnimg.cn/20181225092552520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)