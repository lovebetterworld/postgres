- [pgRouting教程一：介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/82225790)

## 一、概述

**pgRouting**向PostGIS添加了路由功能，这个教程将向你展示：使用[OpenStreetMap](https://link.zhihu.com/?target=http%3A//www.openstreetmap.org/)道路网数据的示例，包括如何准备数据，进行路径查询，编写一个自定义的**'PL/PGSQL'（PostgreSQL的过程化语言-Process Language）**函数来在web地图应用程序中绘制你的路径。换句话说，将pgRouting和其他FOSS4G（Free and Open Source Software For GIS）工具集成到一起。

道路网的导航需要复杂的图论算法，这些算法支持考虑**转弯限制（Turn Restriction）**，甚至支持考虑**依赖时间的属性**。pgRouting是一个可扩展的开源库，它作为PostgreSQL和PostGIS的扩展为最短路径搜索提供了多种工具。

本教程的重点是在实际道路网络中使用pgRouting进行最短路径搜索。它将涵盖以下主题：

- 安装pgRouting
- 创建路由拓扑（topology）
- 使用pgRouting算法
- 导入Open Street Map道路网数据
- 编写高级查询
- 使用PL/PGSQL编写自定义PostgreSQL存储过程（函数）
- 构建一个简单的浏览器应用程序
- 使用OpenLayers构建基本的地图界面

## 二、先决条件

- 教程等级：中级
- 需要的预备知识：SQL（PostgreSQL，PostGIS），JavaScript，HTML
- 设备：原教程使用[OSGeo Live](https://link.zhihu.com/?target=http%3A//live.osgeo.org/)（Version 12.0），我这里不用OSGeo Live这个软件集合，而是直接一步步在Linux中安装相关软件