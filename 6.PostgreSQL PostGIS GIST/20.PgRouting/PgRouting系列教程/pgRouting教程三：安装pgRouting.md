- [pgRouting教程三：安装pgRouting - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/82408769)

官方教程使用OSGeo Live这个软件集合，为了更贴近开发过程，我这里自己手动来安装pgRouting。

本文分别介绍pgRouting在Windows中的安装与在Linux（centos）中的安装。

推荐在Linux中安装pgRouting，因为生产环境中也大多是将pgRouting安装在Linux中。当然，在windows中安装pgRouting来学习本教程也是可以的。

### 一、在Windows中安装pgRouting

在Windows中安装pgRouting比较简单。因为PostGIS的Windows安装包里集成了pgRouting，只要安装了PostGIS也就同时安装了pgRouting。

要安装PostGIS，可以参考我的文章：

[不睡觉的怪叔叔：PostGIS教程二：PostGIS安装和创建空间数据库42 赞同 · 15 评论文章![img](https://pic1.zhimg.com/v2-3eb4080318a4f57d32a1614906d57b60_180x120.jpg)](https://zhuanlan.zhihu.com/p/62157728)

安装了PostGIS后（我这里安装的是2.5.3版的PostGIS），使用pgAdmin（我这里使用的pgAdmin4）连接PostgreSQL数据库，然后新建一个测试数据库test:

![img](https://pic3.zhimg.com/80/v2-ff37468d3ea530b6a570b9fcce15808e_720w.jpg)

在该数据库下执行SQL语句：**CREATE EXTENSION postgis**，创建PostGIS：

![img](https://pic2.zhimg.com/80/v2-3c60d5724ff320f752a330d416d2e1e1_720w.jpg)

成功创建postgis插件，接下来继续在test数据库中执行SQL语句：**CREATE EXTENSION pgRouting**，创建pgRouting:

![img](https://pic4.zhimg.com/80/v2-068e8313b157ad325384da626c91e16b_720w.png)

查看pgRouting版本：

![img](https://pic2.zhimg.com/80/v2-2ee348bb996b90b0a1d0b4b1f11879e5_720w.jpg)

至此，就在Windows操作系统中成功安装了pgRouting插件！