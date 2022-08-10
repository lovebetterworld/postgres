- [PostgreSQL 实时位置跟踪+轨迹分析系统实践 - 单机顶千亿轨迹/天_zxfBdd的博客-CSDN博客_postgis 轨迹](https://blog.csdn.net/u011250186/article/details/103712799?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-103712799-blog-99733230.pc_relevant_default&spm=1001.2101.3001.4242.2&utm_relevant_index=4)

## 标签

[PostgreSQL](https://so.csdn.net/so/search?q=PostgreSQL&spm=1001.2101.3001.7020) , PostGIS , 动态更新位置 , 轨迹跟踪 , 空间分析 , 时空分析

------

## 背景

随着移动设备的普及，越来越多的业务具备了时空属性，例如快递，试试跟踪包裹、快递员位置。例如实体，具备了空间属性。

例如餐饮配送，送货员位置属性。例如车辆，实时位置。等等。

其中两大需求包括：

1、对象位置实时跟踪，例如实时查询某个位点附近、或某个多边形区域内的送货员。

2、对象位置轨迹记录和分析。结合地图，分析轨迹，结合路由算法，预测、生成最佳路径等。

## DEMO

以快递配送为例，GPS设备实时上报快递员轨迹，写入位置跟踪系统，同时将轨迹记录永久保存到轨迹分析系统。

由于快递员可能在配送过程中停留时间较长（比如在某个小区配送时），上报的多条位置可能变化并不大，同时考虑到数据库更新消耗，以及位置的时效性，可以避免一些点的更新（打个比方，上一次位置和当前位置变化量在50米时，不更新）。

动态更新可以减少数据库的更新量，提高整体吞吐能力。

### 设计

![img](https://img-blog.csdnimg.cn/20191226114559800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEyNTAxODY=,size_16,color_FFFFFF,t_70)

 

### 实时位置更新

1、建表

```sql
create table t_pos (  
  uid int primary key,   -- 传感器、快递员、车辆、。。。对象ID  
  pos point,             -- 位置  
  mod_time timestamp     -- 最后修改时间  
);  
  
create index idx_t_pos_1 on t_pos using gist (pos);  
```

真实环境中，我们可以使用PostGIS空间数据库插件，使用geometry数据类型来存储经纬度点。

```sql
create extension postgis;  
  
create table t_pos (  
  uid int primary key,   -- 传感器、快递员、车辆、。。。对象ID  
  pos geometry,          -- 位置  
  mod_time timestamp     -- 最后修改时间  
);  
  
create index idx_t_pos_1 on t_pos using gist (pos);  
```

2、上报位置，自动根据移动范围，更新位置。

例如，移动距离50米以内，不更新。

```csharp
insert into t_pos values (?, st_setsrid(st_makepoint($lat, $lon), 4326), now())  
on conflict (uid)  
do update set pos=excluded.pos, mod_time=excluded.mod_time  
where st_distancespheroid(t_pos.pos, excluded.pos, 'SPHEROID["WGS84",6378137,298.257223563]') > ?;  -- 超过多少米不更新  
```

### 历史轨迹保存

通常终端会批量上报数据，例如每隔10秒上报10秒内采集的点，一次上报的数据可能包含多个点，在PostgreSQL中可以以数组存储。

```sql
create table t_pos_hist (  
  uid int,                  -- 传感器、快递员、车辆、。。。对象ID  
  pos point[],              -- 批量上报的位置  
  crt_time timestamp[]      -- 批量上报的时间点  
);   
  
create index idx_t_pos_hist_uid on t_pos_hist (uid);                 -- 对象ID  
create index idx_t_pos_hist_crt_time on t_pos_hist ((crt_time[1]));    -- 对每批数据的起始时间创建索引  
```

有必要的话，可以多存一个时间字段，用于分区。

### 历史轨迹分析

## 动态位置变更压测

写入并合并，同时判断当距离大于50时，才更新，否则不更新。

（测试）如果使用point类型，则使用如下SQL

```csharp
insert into t_pos values (1, point(1,1), now())  
on conflict (uid)  
do update set pos=excluded.pos, mod_time=excluded.mod_time  
where t_pos.pos <-> excluded.pos > 50;  
```

（实际生产）如果使用PostGIS的geometry类型，则使用如下SQL

```csharp
insert into t_pos values (1, st_setsrid(st_makepoint(120, 71), 4326), now())  
on conflict (uid)  
do update set pos=excluded.pos, mod_time=excluded.mod_time  
where st_distancespheroid(t_pos.pos, excluded.pos, 'SPHEROID["WGS84",6378137,298.257223563]') > 50;  
```

### 压测

首先生成1亿随机空间对象数据。

```sql
postgres=# insert into t_pos select generate_series(1,100000000), point(random()*10000, random()*10000), now();  
INSERT 0 100000000  
Time: 250039.193 ms (04:10.039)  
```

压测脚本如下，1亿空间对象，测试动态更新性能（距离50以内，不更新）。

```csharp
vi test.sql    
  
\set uid random(1,100000000)    
insert into t_pos    
select uid, point(pos[0]+random()*100-50, pos[1]+random()*100-50), now() from t_pos where uid=:uid   
on conflict (uid)   
do update set pos=excluded.pos, mod_time=excluded.mod_time   
where t_pos.pos <-> excluded.pos > 50;   
```

压测结果，动态更新 21.6万点/s，187亿点/天。

```java
pgbench -M prepared -n -r -P 1 -f ./test.sql -c 64 -j 64 -T 120   
  
number of transactions actually processed: 26014936
latency average = 0.295 ms
latency stddev = 0.163 ms
tps = 216767.645838 (including connections establishing)
tps = 216786.403543 (excluding connections establishing)
```

## 轨迹写入压测

每个UID，每批写入50条：写入速度约 467.5万点/s，4039亿点/天。

压测时，写多表，压测使用动态SQL。

```ruby
do language plpgsql $$  
declare  
begin  
  for i in 0..127 loop  
    execute 'create table t_pos_hist'||i||' (like t_pos_hist including all)';  
  end loop;  
end;  
$$;  
```

## 黑科技

1、块级索引（BRIN），在时序属性字段上，建立块级索引，既能达到高效检索目的，又能节约索引空间，还能加速写入。

[《PostgreSQL BRIN索引的pages_per_range选项优化与内核代码优化思考》](https://github.com/digoal/blog/blob/master/201708/20170824_01.md)

[《万亿级电商广告 - brin黑科技带你(最低成本)玩转毫秒级圈人(视觉挖掘姊妹篇) - 阿里云RDS PostgreSQL, HybridDB for PostgreSQL最佳实践》](https://github.com/digoal/blog/blob/master/201708/20170823_01.md)

[《PostGIS空间索引(GiST、BRIN、R-Tree)选择、优化 - 阿里云RDS PostgreSQL最佳实践》](https://github.com/digoal/blog/blob/master/201708/20170820_01.md)

[《自动选择正确索引访问接口(btree,hash,gin,gist,sp-gist,brin,bitmap...)的方法》](https://github.com/digoal/blog/blob/master/201706/20170617_01.md)

[《PostgreSQL 并行写入堆表，如何保证时序线性存储 - BRIN索引优化》](https://github.com/digoal/blog/blob/master/201706/20170611_02.md)

[《PostgreSQL 9种索引的原理和应用场景》](https://github.com/digoal/blog/blob/master/201706/20170627_01.md)

2、阿里云HDB PG特性：sort key , metascan

与BRIN类似，适合线性数据，自动建立块级元数据(取值范围、平均值、CNT、SUM等)进行过滤。

3、空间索引

GiST, SP-GiST空间索引，适合空间数据、以及其他异构数据。

4、动态合并写，根据位置变化量，自动判断是否需要合并更新。

insert on conflict语法，在do update里面，可以进行条件过滤，当位置变化超过N米时，才进行更新。

5、数组、JSON、KV等多值类型。

特别适合多值属性，例如批量上传的轨迹，通常GPS终端上报位置并不是实时的，可能存在一定的 延迟（例如批量上报）。使用数组、JSON都可以存储。

如果使用数组存储，将来分析轨迹时，依旧可以unnest解开，绘制轨迹。

## 性能

1、动态位置变更：1亿被跟踪对象，TPS：21.6万，动态更新21.6万点/s，187亿点/天。

2、轨迹写入：tps约10万，写入467.5万点/s，4039亿点/天。