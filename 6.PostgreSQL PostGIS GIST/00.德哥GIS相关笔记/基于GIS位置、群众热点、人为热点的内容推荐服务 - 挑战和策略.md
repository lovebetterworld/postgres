## 基于GIS位置、群众热点、人为热点的内容推荐服务 - 挑战和策略  

### 作者                                                                        
digoal                                                                                                                 
                          
### 日期                                                                                                                 
2020-02-26                                                                                                             
                                                                                                                 
### 标签                                                                                                                 
PostgreSQL , gis , 表达式索引 , 空间索引 , 只读实例 , copy    
                     
----

## 背景    
带有位置、群众热点、人为热点属性的内容推荐服务.   
内容数量可控, 有淘汰机制, 数据量不会太大.   
最终用户上报位置, 推荐服务则结合用户的位置、内容的位置、内容的时效(付费权重、阶段性热议标签权重)|动态权重等条件推荐相应的内容给用户.   
让所有的内容都有机会在全国曝光, 同时让平台可以基于流量变现.   

## 推荐内容  
呈现给最终用户的内容分为两部分:  
- 本地属性内容推荐 占比xxx  
- 全国属性内容推荐 占比xxx  
  
### 1本地推荐条件   
地域属性: 可以用来查询在用户附近的内容  

时效权重: 付费时效权重、标签时效权重. 例如:   
- 由运营负责维护的标签权重, 近期热点话题标签, 热度随时可能发生变化.   
- 付费则是为流量变现准备的权重, 付费的内容权重提升. 这些都是有时效的.   
  

动态权重: 内容的下载、点击、赞等计数, 越热的内容越容易被推荐   

### 2全国推荐条件  
时效权重: 付费时效权重、标签时效权重. 例如:   
- 由运营负责维护的标签权重, 近期热点话题标签, 热度随时可能发生变化.   
- 付费则是为流量变现准备的权重, 付费的内容权重提升. 这些都是有时效的.   
  

动态权重: 内容的下载、点击、赞等计数  

## 核心数据结构指标  
内容属性表:  
- 内容id, 付费时效权重, 标签id, 经纬度, 赞, 点击, 下载计数  
  

标签属性(运营调整)表:  
- 标签id, 标签权重   
  
## 业务指标  
xxx亿用户数级别  
xxx亿级别内容数  

## 挑战点和应对策略   
1、作为群众热点的内容有大量更新(动态权重更新:赞, 点击, 下载计数)   
策略: 合并update   

2、本地内容推荐算法: 地域 + (时效权重 + 动态权重)排序 , 挑战: 无法完全用索引筛选.      
策略:  

分级过滤, 数据库过滤第一道, 应用过滤第二道   

batch get: 使用postgis, 采用空间索引, 仅按空间排序get 指定记录数(走空间索引), 返回记录数万级别.   

(根据数据库压力, 选择是否将 时效+动态权重排序 下移到应用层计算.)    

缓存, 应用层根据时效+动态权重排序后筛选并写入缓存, 缓存分批次老化,    

3、全国内容推荐算法: (时效权重 + 动态权重)排序 , 时效权重在另一张表, 挑战: 无法完全用索引筛选.   
策略1:   

batch get1:   
按动态权重排序取出第1批结果(采用表达式索引, 无需额外计算), 输出时同时关联出标签表的权重, 记录数万级别.    

batch get2:   
从标签表按权重取出top n时效标签, 基于这些时效标签查询内容表, 按动态权重排序取出第2批结果(采用表达式索引, 无需额外计算), 输出时同时关联出标签表的权重, 记录数万级别.   

缓存, 应用层根据时效+动态权重排序后筛选并写入缓存, 缓存分批次老化,   

缺陷: 结果可能不精确, 某些(时效权重 + 动态权重高)的内容可能永远取不出来.    

策略2:  

硬算: JOIN后计算综合权重排名, 并写入缓存.  由于是全国热点, 全国共享, 查询量不大, 可以定期更新缓存.   

4、人口密集处可能有大量缓存重复    

缓存:  按geohash box(可根据实际情况缩放hash精度) 映射到 固定内容集ID , 落在同一个geohash的用户, 都取同一个映射的内容集.    

取的时候: 取1个格子的映射内容集, 或取相邻的9个格子的映射内容集, 或取若干相邻格子的映射内容集.    

5、单次数据库batch查询返回记录量大    

使用PG只读实例   

用copy接口返回记录, 并发情况下每秒能返回数千万行记录  

## 结论   
数据库挑战: 大量大结果集的JOIN查询查询|GIS空间排序查询、网络带宽、频繁更新.     

策略: 空间索引、表达式索引、合并更新、加只读节点、copy接口.    

应用挑战: 排序计算(第二道过滤).         

策略: 横向扩容    

缓存挑战: 缓存重复、读、更新.                   

策略: geohash 映射降低重复、横向扩容   