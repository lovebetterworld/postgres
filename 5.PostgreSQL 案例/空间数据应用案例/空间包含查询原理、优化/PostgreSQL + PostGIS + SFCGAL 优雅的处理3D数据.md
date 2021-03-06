# PostgreSQL + PostGIS + SFCGAL 优雅的处理3D数据

**作者**

digoal

**日期**

2017-10-18

**标签**

PostgreSQL , SFCGAL , 3D

# 背景

PostGIS 整合 SFCGAL，优雅的处理3D空间数据。

[![pic](https://github.com/digoal/blog/raw/master/201710/20171026_02_pic_001.jpg)](https://github.com/digoal/blog/blob/master/201710/20171026_02_pic_001.jpg)

[![pic](https://github.com/digoal/blog/raw/master/201710/20171026_02_pic_002.jpg)](https://github.com/digoal/blog/blob/master/201710/20171026_02_pic_002.jpg)

[![pic](https://github.com/digoal/blog/raw/master/201710/20171026_02_pic_003.jpg)](https://github.com/digoal/blog/blob/master/201710/20171026_02_pic_003.jpg)

# 例子

[PDF: 3D and exact geometries for PostGIS , FOSDEM PGDay](https://github.com/digoal/blog/blob/master/201710/20171026_02_pdf_001.pdf)

http://www.sfcgal.org/

https://www.tuicool.com/articles/jAjIBn

https://wiki.postgresql.org/images/3/36/Postgis_3d_pgday2013_hm.pdf

http://postgis.net/docs/manual-2.4/reference.html

So, for those of you who haven’t seen it, [SFCGAL](http://www.sfcgal.org/) , “a C++ wrapper library around [CGAL](http://www.cgal.org/) with the aim of supporting ISO 19107:2013 and [OGC Simple Features Access 1.2](http://www.opengeospatial.org/standards/sfa) for 3D operations” is now an optional include in PostGIS (I believe beginning with 2.1, forgive me if I’m wrong).

This was a quiet outcome of the Boston Code Sprint, after Paul Ramsey declared exact rational number representation would not make its way  into PostGIS.

(I promise, that’s the only animated gif I’ll ever do, hat tip James Fee who did it for years before it was cool).

What does this mean for a typical PostGIS user? Well, so far it adds a nice suite of new 2D and 3D functions :

http://postgis.net/docs/manual-2.4/reference.html

```plsql
postgis_sfcgal_version — Returns the version of SFCGAL in use  
  
ST_Extrude — Extrude a surface to a related volume  
  
ST_StraightSkeleton — Compute a straight skeleton from a geometry  
  
ST_ApproximateMedialAxis — Compute the approximate medial axis of an areal geometry.  
  
ST_IsPlanar — Check if a surface is or not planar  
  
ST_Orientation — Determine surface orientation  
  
ST_ForceLHR — Force LHR orientation  
  
ST_MinkowskiSum — Performs Minkowski sum  
  
ST_3DIntersection — Perform 3D intersection  
  
ST_3DDifference — Perform 3D difference  
  
ST_3DUnion — Perform 3D union  
  
ST_3DArea — Computes area of 3D surface geometries. Will return 0 for solids.  
  
ST_Tesselate — Perform surface Tesselation of a polygon or polyhedralsurface and returns as a TIN or collection of TINS  
  
ST_Volume — Computes the volume of a 3D solid. If applied to surface (even closed) geometries will return 0.  
  
ST_MakeSolid — Cast the geometry into a solid. No check is performed. To obtain a valid solid, the input geometry must be a closed Polyhedral Surface or a closed TIN.  
  
ST_IsSolid — Test if the geometry is a solid. No validity check is performed.  
```

ST_Extrude is fun，this is a function for doing things like this:

[![pic](https://github.com/digoal/blog/raw/master/201710/20171026_02_pic_004.jpg)](https://github.com/digoal/blog/blob/master/201710/20171026_02_pic_004.jpg)

Simulated extruded building footprints.

Extruded footprints from (ahem) City Engine. Ssssh. Don’t tell.

ST_StraightSkeleton does in one step the first phase of what I’ve  been going on about for a couple years re: Voronoi diagrams (and  bypasses Voronoi altogether):

[![pic](https://github.com/digoal/blog/raw/master/201710/20171026_02_pic_005.jpg)](https://github.com/digoal/blog/blob/master/201710/20171026_02_pic_005.jpg)

Approximation of straight skeleton / skeletonization of stream polygon。

Plus more! I’ve just started exploring.

BTW, in advance of there being an SFCGAL install guide for PostGIS, a good source for info on install can be gleaned from the [PostGIS Developers listserve](http://osgeo-org.1560.x6.nabble.com/SFCGAL-trouble-installing-td5083390.html) .