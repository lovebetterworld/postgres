PostgreSQL学习仓库：https://gitee.com/AiShiYuShiJiePingXing/postgres



# 一、仓库说明

本来是想直接同步德哥的pg仓库，但是奈何德哥的仓库太大了，都无法直接从GitHub迁移到gitee。但是看到德哥的许多笔记十分有用，于是花了很久时间，把德哥的笔记都过了两遍。

第一遍，发现德哥有很多私人笔记在里面，并不是有关于PostgreSQL相关的，手动做了剔除。

第二遍，发现笔记太过于混乱，不太好明确的找到相应的内容，遂自己做了一个简单的目录。

同时，也将该仓库作为自己学习和研究PostgreSQL的仓库，会不断积累和分享有关PostgreSQL的内容。

德哥PostgreSQL仓库地址：https://github.com/digoal/blog



## 1.1 德哥是谁？

digoal(德哥)   

- PostgreSQL 中国社区发起人之一。负责PostgreSQL数据库在中国的技术落地与推广、人才培养。          
- 中国开源软件推进联盟PostgreSQL分会，特聘资深领域专家。         
- 中国信息通信研究院主办、中国通信标准化协会支持的"OSCAR云计算开源产业大会"评选：2018届OSCAR开源尖峰人物之一
- 阿里云数据库首席专家团队成员，提供[数据库首席专家服务](https://www.aliyun.com/service/chiefexpert/database)。    
- 阿里巴巴钻石布道师
- 42项数据库专利

### 1.1.1 贡献

专利（截至2020-01）:

```
201210121337.9;102708158B - 一种PostgreSQL云存储归档调度系统 
201410207285.6;104166666B - PostgreSQL高并发流式大数据多维度准实时统计的方法 
201410548447.2;104503965B - PostgreSQL高弹性的高可用及负载均衡实现方法 
201410550641.4;104503966B - PostgreSQL大数据高效免维护自动分区方法 
201410652026.4;104572809B - 一种分布式关系数据库自由扩展方法 
201410652028.3;104503974B - 一种基于云平台的关系数据库自动优化方法 
201410751853.9;104536988B - MonetDB分布式计算存储方法 
201410754052.8;104503865B - PostgreSQL快速恢复到任意时间点的方法 
201410780380.5;104750573B - 分布式数据系统数据节点的全局一致性备份和还原方法 
201510078225.3;104731863B - 简化PostgreSQL分区代码的方法 
201510083556.6 - 一种使用keepalived软件实现数据库HA应用的方法[驳回] 
201510107904.9;104809152B - 一种节约PostgreSQL共享内存的方法及系统 
201510724098.X - 用于资源调度的方法和设备 
201510835059.7 - 用于主备数据一致性校验的方法和设备 
201510875827.1 - 一种用于实现主备同步模式下事务提交的方法与设备 
201510900751.3 - 用于创建主备数据库的方法和设备 
201510904243.2 - 一种提高数据安全的方法与设备 
201511001289.X - 用于数据发送、限制发送进程占用带宽的方法和装置 
201511019216.3 - 数据库心跳检测方法以及装置 
201511026756.4 - 一种用于更改秘钥的方法、装置及设置秘钥的方法、装置 
201610101944.7 - 数据库读写分离方法、装置和系统 
201611183503.2 - 一种数据库死锁的处理方法、装置和数据库系统 
201710093376.5 - 数据同步方法及设备 
201710093382.0 - 一种查询唯一值的方法及设备 
201710488500.8 - 一种数据容错的方法及设备 
201710492419.7 - 一种重做日志持久化方法及设备 
201710581818.0 - 数据库宕机后的访问方法、装置和系统 
201710643448.9 - 数据的存储、处理及读取方法、数据存储设备和系统 
201710813987.2 - 一种数据操作方法及设备 
201710860627.8 - 创建索引的方法和装置 
201710860628.2 - 数据获取模型建立的方法和数据获取的方法及装置 
201710862401.1 - 一种索引创建方法、装置及数据库系统 
201710862402.6 - 数据库的检索方法及装置 
201710862403.0 - 数据库处理方法及装置、系统 
201711105172.5 - 数据查询方法和装置 
201711117230.6 - 数据处理方法、装置及设备 
201711195546.7 - 数据写入方法及设备 
201711239837.1 - 一种用于实现数据库高可用性的方法及装置 
201711275922.3 - 一种数据库更新的方法及装置 
201711482536.1 - 一种数据库系统以及查询数据库的方法和装置 
201810182191.6 - 数据处理方法和数据处理装置 
201810982558.2 - 一种数据跟踪处理方法及装置
```

### 1.1.2 社区贡献   

- 自2008年开始，坚持开源PostgreSQL数据库的布道，负责PostgreSQL数据库在中国的技术落地与推广、人才培养。      

```
加入阿里后的一些布道经历

2015-09-12 PG象行中国上海站-PG回归测试       
2015-10-23 开源中国广州站技术分享 - PG企业特性       
2015-11-15 开源中国源创会广州站2015-PG企业特性       
2015-11-20 2015 PG峰会分享-一位PGer的安全修养        
2016-05-13 2016dtcc- 从Oracle DBA到PostgreSQL布道者  
2016-05-17 云栖大会-武汉-PostgreSQL物联网独孤九式    
2016-09-24 全球敏捷运维峰会-阿里云PostgreSQL介绍     
2016-10-27 PG 2016全国峰会-PostgreSQL 数据库前世今生 
2016-10-27 PG2016全国峰会-PostgreSQL应用开发最佳实践 
2016-10-27 PG2016全国峰会-sharding单元化（based on postgres_fdw)最佳实践     
2017-03-18 SDCC 数据库核心技术与应用实战峰会-数据库超体      
2017-05-27 2017 PG 象行中国社区会议-武汉站-PostgreSQL在阿里的应用    
2017-06-24 开源中国源创会-杭州-自动驾驶背后到数据库  
2017-08-23 2017 ODF会议，分享PG时空数据管理案例      
2017-10-20 开源中国PG分会场，分享，阿里云RDS PG多维存储与流计算实践  
2017-10-21 2017 PG峰会-阿里云RDS、HDB PG 多维存储特性与案例  
2018-03-17 空间数据库应用峰会-北京师范 - PostgreSQL空间数据业务 优化实践     
2018-07-01 开源联盟 - PG HTAP展望    
2018-09-01 云栖TechDay - PG天天象上活动 - 杭州站第一期组织+分享      
2018-09-07 ODF - 数据库华山论剑 PG vs MySQL  
2018-09-09 云栖TechDay - PG天天象上活动 - 北京站组织+分享    
2018-10-13 云栖techday - 全栈数据库PG天天象上活动 - 郑州站组织+分享  
2018-10-14 ruby summit 2018 郑州- PG开发者特性分享   
2018-10-14 中国开发者大会-郑州-高并发和大数据下的PG实战      
2018-10-27 云栖TechDay - PG天天象上活动 - 广州站组织+分享    
2018-10-28 云栖TechDay - PG天天象上活动 - 深圳站组织+分享    
2018-11-24 云栖TechDay - PG天天象上活动 - 上海站组织+分享    
2018-12-22 云栖TechDay - PG天天象上活动 - 南京站组织+分享    
2019-01-12 云栖TechDay - PG天天象上活动 - 合肥站组织+分享    
2019-03-23 云栖TechDay - PG天天象上活动 - 长沙站组织+分享    
2019-04-19 云栖TechDay - PG天天象上数据库沙龙 - 成都站组织+分享      
2019-05-11 dtcc 2019-PG数据库实战深度培训    
2019-05-25 云栖TechDay - PG天天象上活动 - 杭州站组织+分享    
2019-06-01 PG象行中国杭州站-PG11,12新特性分享        
2019-06-22 阿里云开发者技术沙龙 - PG天天象上活动 - 武汉站组织+分享   
2019-07-03 PG社区走进名企-平安集团-分享PG使用的正确姿势      
2019-07-07 PG CONF 2019中国峰会，出席分享、培训      
2019-07-13 PG天天象上-济南站-1天PG技术分享   
2019-07-20 DAMS中国数据库智能管理峰会-PG去O非你莫属  
2019-08-17 福州，象行中国- PG的生态、新特性与Oracle迁移介绍  
2019-08-24 PostgreSQL 11 与 12 用户最关心的特性解读  
2019-09-27 云栖大会2019-开发者进阶-技术影响力 -从技术深度到产品广度的裂变    
2019-11-16 数据库嘉年华-PG为什么这么火       
2019-11-23 第十八届全国软件与应用学术会议(NASAC 2019)-PostgreSQL的社会价值   
2019-11-29 PG12 新特性与路标    
2019-11-30 PG峰会-PG 诊断方法
2019-12-07 PG社区走进中兴企业交流-分享《PG的社会价值&企业如何融入PG社区》    
2020-01-04 pg+mongo助力企业去O       
2020-01-11 PG天天象上2020.1.11杭州站-实时精准营销系统设计
2020-01-18 PostgreSQL+MySQL 联合解决方案课程14讲
2020-07-18 PG SSL安全链路分析  
2020-08-15 CUUG PG数据库高校行启动会, 分享AliPG
2020-09-10 亿级用户量的实时推荐数据库到底要几毛钱?
2020-09-13 在数据库中跑全文检索、模糊查询SQL会不会被开除?
2020-09-17 百城汇：云栖线下高校合作专场 , 宁夏大学
2020-09-19 刷脸支付会不会刷到别人的钱包?
2020-09-26 为什么打车和宇宙大爆炸有关?
2020-09-26 大话数据库终局之战
2020-10-18 为什么饿了么网上订餐不会凉凉 & 牛顿发现万有引力有关?
2020-11-01 PG中文社区 & Hello bike & 阿里云 联合沙龙活动
2020-11-17 PG分会-解密下一代云数据库mybase
2020-11-22 数据库嘉年华分享-解密下一代云数据库mybase
2020-12-18 大型企业数据库与应用实践-dsg,阿里云,pg社区,上海开源信息技术协会 联合主办
2021-01-15 PG中文社区2020年度峰会-2021,别让你的企业输在起跑线
2021-01-18 阿里云PG创新训练营
```



## 1.2 最佳阅读方式

因为文章全部是md格式的，所以建议采用Typora阅读。

1. `git clone https://gitee.com/AiShiYuShiJiePingXing/postgres.git`
2. 使用Typora打开克隆后的项目

![image-20210621165918849](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210621165918849.png)

3. 阅读学习

![image-20210621165952329](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210621165952329.png)

# 二、德哥的PostgreSQL, Greenplum 学习视频  

1、视频下载链接： https://pan.baidu.com/s/1Q5u5NSrb0gL5-psA9DCBUQ   (提取码：5nox   如果链接失效请通知我, 谢谢)  
- PostgreSQL 9.3 数据库管理与优化 4天  
- PostgreSQL 9.3 数据库管理与优化 5天  
- PostgreSQL 9.1 数据库管理与开发 1天  
- PostgreSQL 9.3 数据库优化 3天  
- PostgreSQL 专题讲座  

PostgreSQL Greenplum 培训视频分享：http://pan.baidu.com/s/1pKVCgHX   

# 三、PostgreSQL资料(持续更新)

## 3.1 PostgreSQL学习系列

### 3.1.1 不睡觉的怪叔叔的PostGIS教程

1. [PostGIS介绍](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/1.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9APostGIS%E4%BB%8B%E7%BB%8D.md)

2. [PostGIS安装](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/2.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9APostGIS%E7%9A%84%E5%AE%89%E8%A3%85.md)

3. [创建空间数据库](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/3.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%88%9B%E5%BB%BA%E7%A9%BA%E9%97%B4%E6%95%B0%E6%8D%AE%E5%BA%93.md)

4. [加载空间数据](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/4.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%8A%A0%E8%BD%BD%E7%A9%BA%E9%97%B4%E6%95%B0%E6%8D%AE.md)

5. [数据](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/5.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E6%95%B0%E6%8D%AE.md)

6. [简单的SQL语句](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/6.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%AE%80%E5%8D%95%E7%9A%84SQL%E8%AF%AD%E5%8F%A5.md)

7. [几何图形（Geometry）](s)

8. [关于几何图形的练习](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/8.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%85%B3%E4%BA%8E%E5%87%A0%E4%BD%95%E5%9B%BE%E5%BD%A2%E7%9A%84%E7%BB%83%E4%B9%A0.md)

9. [空间关系](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/9.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%A9%BA%E9%97%B4%E5%85%B3%E7%B3%BB.md)

10. [空间连接](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/10.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%A9%BA%E9%97%B4%E8%BF%9E%E6%8E%A5.md)

11. [空间索引](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/11.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%A9%BA%E9%97%B4%E7%B4%A2%E5%BC%95.md)

12. [投影数据](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/12.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E6%8A%95%E5%BD%B1%E6%95%B0%E6%8D%AE.md)

13. [地理](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/13.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%9C%B0%E7%90%86.md)

14. [几何图形创建函数](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/14.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%87%A0%E4%BD%95%E5%9B%BE%E5%BD%A2%E5%88%9B%E5%BB%BA%E5%87%BD%E6%95%B0.md)

15. [更多的空间连接](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/15.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E6%9B%B4%E5%A4%9A%E7%9A%84%E7%A9%BA%E9%97%B4%E8%BF%9E%E6%8E%A5.md)

16. [有效性](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/16.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E5%87%A0%E4%BD%95%E5%9B%BE%E5%BD%A2%E7%9A%84%E6%9C%89%E6%95%88%E6%80%A7.md)

17. [相等](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/17.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%9B%B8%E7%AD%89.md)

18. [线性参考](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/18.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%BA%BF%E6%80%A7%E5%8F%82%E8%80%83.md)

19. [维数扩展的9交集模型](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/19.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%BB%B4%E6%95%B0%E6%89%A9%E5%B1%95%E7%9A%849%E4%BA%A4%E9%9B%86%E6%A8%A1%E5%9E%8B.md)

20. [索引集群](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/20.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E7%B4%A2%E5%BC%95%E9%9B%86%E7%BE%A4.md)

21. [3-D](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/21.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A3-D.md)

22. [最近邻域搜索](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E4%B8%8D%E7%9D%A1%E8%A7%89%E7%9A%84%E6%80%AA%E5%8F%94%E5%8F%94PostGIS%E6%95%99%E7%A8%8B/22.PostGIS%E6%95%99%E7%A8%8B%EF%BC%9A%E6%9C%80%E8%BF%91%E9%82%BB%E5%9F%9F%E6%90%9C%E7%B4%A2.md)

### 3.1.2 菜鸟教程-PostgreSQL系列

- [1.PostgreSQL 学习系列](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97)

- [2.PostgreSQL 优势](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF)

- [3.PostgreSQL 笔记](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/3.PostgreSQL%20%E7%AC%94%E8%AE%B0)

- [4.PostgreSQL 课程](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/4.PostgreSQL%20%E8%AF%BE%E7%A8%8B)

- [5.PostgreSQL 案例](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B)

- [6.PostgreSQL PostGIS GIST](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/6.PostgreSQL%20PostGIS%20GIST)

- [7.PostgreSQL 推荐系统](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F)

- [8.PostgreSQL 应用开发解决方案](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

- [9.PostgreSQL 插件](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6)

- [10.PostgreSQL 配置文件](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/10.PostgreSQL%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

- [11.PostgreSQL PDF资料文档](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/11.PostgreSQL%20PDF%E8%B5%84%E6%96%99%E6%96%87%E6%A1%A3)

- [12.PostgreSQL 安装与部署](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/12.PostgreSQL%20%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2)

- [13.PostgreSQL Alter,Truncate Table](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/13.PostgreSQL%20Alter,Truncate%20Table.md)

- [14.PostgreSQL视图、事务、锁](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/14.PostgreSQL%E8%A7%86%E5%9B%BE%E3%80%81%E4%BA%8B%E5%8A%A1%E3%80%81%E9%94%81.md)

- [15.PostgreSQL子查询，Auto Increment](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/15.PostgreSQL%E5%AD%90%E6%9F%A5%E8%AF%A2%EF%BC%8CAuto%20Increment.md)

- [16.PostgreSQL权限Privileges](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/16.PostgreSQL%E6%9D%83%E9%99%90Privileges.md)

- [17.PostgreSQL时间日期函数和操作符 ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/17.PostgreSQL%E6%97%B6%E9%97%B4%E6%97%A5%E6%9C%9F%E5%87%BD%E6%95%B0%E5%92%8C%E6%93%8D%E4%BD%9C%E7%AC%A6%20.md)

- [18.PostgreSQL常用函数](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/18.PostgreSQL%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0.md)

- [19.PostgreSQL的模式、表、空间、用户间的关系](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97/%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/19.PostgreSQL%E7%9A%84%E6%A8%A1%E5%BC%8F%E3%80%81%E8%A1%A8%E3%80%81%E7%A9%BA%E9%97%B4%E3%80%81%E7%94%A8%E6%88%B7%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.md)

## 3.2 PostgreSQL优势

- [企业数据库选型规则](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF/%E4%BC%81%E4%B8%9A%E6%95%B0%E6%8D%AE%E5%BA%93%E9%80%89%E5%9E%8B%E8%A7%84%E5%88%99%20%20%20%20.md)

- [为什么数据库选型和找对象一样重要](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF/%E4%B8%BA%E4%BB%80%E4%B9%88%E6%95%B0%E6%8D%AE%E5%BA%93%E9%80%89%E5%9E%8B%E5%92%8C%E6%89%BE%E5%AF%B9%E8%B1%A1%E4%B8%80%E6%A0%B7%E9%87%8D%E8%A6%81.md)
- [为什么选择开源数据库、如何选择、需要做哪些准备](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF/%E4%B8%BA%E4%BB%80%E4%B9%88%E9%80%89%E6%8B%A9%E5%BC%80%E6%BA%90%E6%95%B0%E6%8D%AE%E5%BA%93%E3%80%81%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9%E3%80%81%E9%9C%80%E8%A6%81%E5%81%9A%E5%93%AA%E4%BA%9B%E5%87%86%E5%A4%87.md)
- [学生为什么应该学PG, PG与其他数据库有哪些独特性, 为什么PG是数据库的未来](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF/%E5%AD%A6%E7%94%9F%E4%B8%BA%E4%BB%80%E4%B9%88%E5%BA%94%E8%AF%A5%E5%AD%A6PG,%20PG%E4%B8%8E%E5%85%B6%E4%BB%96%E6%95%B0%E6%8D%AE%E5%BA%93%E6%9C%89%E5%93%AA%E4%BA%9B%E7%8B%AC%E7%89%B9%E6%80%A7,%20%E4%B8%BA%E4%BB%80%E4%B9%88PG%E6%98%AF%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E6%9C%AA%E6%9D%A5%20%20.md)



## 3.3 PostgreSQL笔记

![image-20210622142532687](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622142532687.png)

## 3.4 PostgreSQL课程

![image-20210622142624449](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622142624449.png)

## 3.5 PostgreSQL案例

- [阿里云PostgreSQL案例精选1 - 实时精准营销、人群圈选 ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B/%E9%98%BF%E9%87%8C%E4%BA%91PostgreSQL%E6%A1%88%E4%BE%8B%E7%B2%BE%E9%80%891%20-%20%E5%AE%9E%E6%97%B6%E7%B2%BE%E5%87%86%E8%90%A5%E9%94%80%E3%80%81%E4%BA%BA%E7%BE%A4%E5%9C%88%E9%80%89%20.md)
- [阿里云PostgreSQL案例精选2 - 图像识别、人脸识别、相似特征检索、相似人群圈选](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B/%E9%98%BF%E9%87%8C%E4%BA%91PostgreSQL%E6%A1%88%E4%BE%8B%E7%B2%BE%E9%80%892%20-%20%E5%9B%BE%E5%83%8F%E8%AF%86%E5%88%AB%E3%80%81%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB%E3%80%81%E7%9B%B8%E4%BC%BC%E7%89%B9%E5%BE%81%E6%A3%80%E7%B4%A2%E3%80%81%E7%9B%B8%E4%BC%BC%E4%BA%BA%E7%BE%A4%E5%9C%88%E9%80%89%20%20%20.md)
- [PostgreSQL 物流轨迹系统数据库需求分析与设计 - 包裹侠实时跟踪与召回 ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B/PostgreSQL%20%E7%89%A9%E6%B5%81%E8%BD%A8%E8%BF%B9%E7%B3%BB%E7%BB%9F%E6%95%B0%E6%8D%AE%E5%BA%93%E9%9C%80%E6%B1%82%E5%88%86%E6%9E%90%E4%B8%8E%E8%AE%BE%E8%AE%A1%20-%20%E5%8C%85%E8%A3%B9%E4%BE%A0%E5%AE%9E%E6%97%B6%E8%B7%9F%E8%B8%AA%E4%B8%8E%E5%8F%AC%E5%9B%9E%20.md)
- [PostgreSQL数据库应用：基于GIS的实时车辆位置查询](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B/PostgreSQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%BA%94%E7%94%A8%EF%BC%9A%E5%9F%BA%E4%BA%8EGIS%E7%9A%84%E5%AE%9E%E6%97%B6%E8%BD%A6%E8%BE%86%E4%BD%8D%E7%BD%AE%E6%9F%A5%E8%AF%A2.md)

## 3.6 PostgreSQL PostGIS GIST

### 3.6.1 德哥PostGIS GIST相关笔记

![image-20210622142902120](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210622142902120.png)

- [基于PG与PostGIS搭建实时矢量瓦片服务](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/6.PostgreSQL%20PostGIS%20GIST/%E5%9F%BA%E4%BA%8EPG%E4%B8%8EPostGIS%E6%90%AD%E5%BB%BA%E5%AE%9E%E6%97%B6%E7%9F%A2%E9%87%8F%E7%93%A6%E7%89%87%E6%9C%8D%E5%8A%A1.md)
- [PostGIS总结](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/6.PostgreSQL%20PostGIS%20GIST/PostGIS.md)
- [PostgreSQL存储地理信息数据的注意点](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/6.PostgreSQL%20PostGIS%20GIST/PostgreSQL%E5%AD%98%E5%82%A8%E5%9C%B0%E7%90%86%E4%BF%A1%E6%81%AF%E6%95%B0%E6%8D%AE%E7%9A%84%E6%B3%A8%E6%84%8F%E7%82%B9.md)
- [postgresql 创建gis空间数据库，shp数据入库](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/6.PostgreSQL%20PostGIS%20GIST/postgresql%20%E5%88%9B%E5%BB%BAgis%E7%A9%BA%E9%97%B4%E6%95%B0%E6%8D%AE%E5%BA%93%EF%BC%8Cshp%E6%95%B0%E6%8D%AE%E5%85%A5%E5%BA%93.md)
- [PostGIS基本使用](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/6.PostgreSQL%20PostGIS%20GIST/PostGIS%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8.md)

## 3.7 PostgreSQL推荐系统

- [社交、电商、游戏等 推荐系统 (相似推荐)](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F/%E7%A4%BE%E4%BA%A4%E3%80%81%E7%94%B5%E5%95%86%E3%80%81%E6%B8%B8%E6%88%8F%E7%AD%89%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F%20(%E7%9B%B8%E4%BC%BC%E6%8E%A8%E8%8D%90).md)
- [推荐系统, 已阅读过滤, 大量CPU和IO浪费的优化思路2](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F/%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F,%20%E5%B7%B2%E9%98%85%E8%AF%BB%E8%BF%87%E6%BB%A4,%20%E5%A4%A7%E9%87%8FCPU%E5%92%8CIO%E6%B5%AA%E8%B4%B9%E7%9A%84%E4%BC%98%E5%8C%96%E6%80%9D%E8%B7%AF2.md)
- [用户喜好推荐系统 - PostgreSQL 近似计算应用](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F/%E7%94%A8%E6%88%B7%E5%96%9C%E5%A5%BD%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F%20-%20PostgreSQL%20%E8%BF%91%E4%BC%BC%E8%AE%A1%E7%AE%97%E5%BA%94%E7%94%A8.md)
- [PostgreSQL 推荐系统优化总计](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F/PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96%E6%80%BB%E8%AE%A1.md)
- [PostgreSQL 相似人群圈选，人群扩选，向量相似 使用实践 - cube  ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F/PostgreSQL%20%E7%9B%B8%E4%BC%BC%E4%BA%BA%E7%BE%A4%E5%9C%88%E9%80%89%EF%BC%8C%E4%BA%BA%E7%BE%A4%E6%89%A9%E9%80%89%EF%BC%8C%E5%90%91%E9%87%8F%E7%9B%B8%E4%BC%BC%20%E4%BD%BF%E7%94%A8%E5%AE%9E%E8%B7%B5%20-%20cube%20%20.md)

## 3.8 PostgreSQL应用开发解决方案

- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 1. 中文分词与模糊查询  ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%201.%20%E4%B8%AD%E6%96%87%E5%88%86%E8%AF%8D%E4%B8%8E%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2%20%20.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 2. 短视频业务实时推荐    ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%202.%20%E7%9F%AD%E8%A7%86%E9%A2%91%E4%B8%9A%E5%8A%A1%E5%AE%9E%E6%97%B6%E6%8E%A8%E8%8D%90%20%20%20%20.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 3. 人脸识别和向量相似搜索      ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%203.%20%E4%BA%BA%E8%84%B8%E8%AF%86%E5%88%AB%E5%92%8C%E5%90%91%E9%87%8F%E7%9B%B8%E4%BC%BC%E6%90%9C%E7%B4%A2%20%20%20%20%20%20.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 4. 出行相关调度系统     ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%204.%20%E5%87%BA%E8%A1%8C%E7%9B%B8%E5%85%B3%E8%B0%83%E5%BA%A6%E7%B3%BB%E7%BB%9F%20%20%20%20%20.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 5. 配送相关调度系统    ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%205.%20%E9%85%8D%E9%80%81%E7%9B%B8%E5%85%B3%E8%B0%83%E5%BA%A6%E7%B3%BB%E7%BB%9F%20%20%20%20.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 6. 时空、时态、时序、日志等轨迹系统](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%206.%20%E6%97%B6%E7%A9%BA%E3%80%81%E6%97%B6%E6%80%81%E3%80%81%E6%97%B6%E5%BA%8F%E3%80%81%E6%97%A5%E5%BF%97%E7%AD%89%E8%BD%A8%E8%BF%B9%E7%B3%BB%E7%BB%9F.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 8. 树状图谱关系系统](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%208.%20%E6%A0%91%E7%8A%B6%E5%9B%BE%E8%B0%B1%E5%85%B3%E7%B3%BB%E7%B3%BB%E7%BB%9F.md)
- [PostgreSQL 应用开发解决方案最佳实践系列课程 - 9. 数据存储冷热分离   ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97%E8%AF%BE%E7%A8%8B%20-%209.%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%86%B7%E7%83%AD%E5%88%86%E7%A6%BB%20%20%20.md)



## 3.9 PostgreSQL插件与工具

- [PostgreSQL 用户最喜爱的扩展插件功能  ](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6%E4%B8%8E%E5%B7%A5%E5%85%B7/PostgreSQL%20%E7%94%A8%E6%88%B7%E6%9C%80%E5%96%9C%E7%88%B1%E7%9A%84%E6%89%A9%E5%B1%95%E6%8F%92%E4%BB%B6%E5%8A%9F%E8%83%BD%20%20.md)
- [PostgreSQL 有价值的插件、可改进功能](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6%E4%B8%8E%E5%B7%A5%E5%85%B7/PostgreSQL%20%E6%9C%89%E4%BB%B7%E5%80%BC%E7%9A%84%E6%8F%92%E4%BB%B6%E3%80%81%E5%8F%AF%E6%94%B9%E8%BF%9B%E5%8A%9F%E8%83%BD.md)
- [PostgreSQL 最常用的插件](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6%E4%B8%8E%E5%B7%A5%E5%85%B7/PostgreSQL%20%E6%9C%80%E5%B8%B8%E7%94%A8%E7%9A%84%E6%8F%92%E4%BB%B6.md)
- [PostgreSQL工具](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6%E4%B8%8E%E5%B7%A5%E5%85%B7/PostgreSQL%E5%B7%A5%E5%85%B7.md)

## 3.10 PostgreSQL配置文件

- [PostgreSQL 11 postgresql.conf 参数模板](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/10.PostgreSQL%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6/PostgreSQL%2011%20postgresql.conf%20%E5%8F%82%E6%95%B0%E6%A8%A1%E6%9D%BF.md)



## 3.11 PostgreSQL PDF资料及文档

- PG的社会价值.pdf
- PostgreSQL思维导图
- 开发者PG TOP18问.pdf



## 3.12 PostgreSQL安装与部署

- [Linux中PostgreSQL和PostGIS的安装和使用](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/12.PostgreSQL%20%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2/Linux%E4%B8%ADPostgreSQL%E5%92%8CPostGIS%E7%9A%84%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8.md)
- [PostgreSQL+PostGIS安装部署](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/12.PostgreSQL%20%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2/PostgreSQL+PostGIS%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2.md)
- [PostgreSQL的Docker安装与部署](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/12.PostgreSQL%20%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2/PostgreSQL%E7%9A%84Docker%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2.md)



## 3.13 PostgreSQL开发与使用

### 3.13.1 PostgreSQL规范

- [1.PostgreSQL 命名规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/1.PostgreSQL%20%E5%91%BD%E5%90%8D%E8%A7%84%E8%8C%83.md)
- [2.PostgreSQL 设计规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/2.PostgreSQL%20%E8%AE%BE%E8%AE%A1%E8%A7%84%E8%8C%83.md)
- [3.PostgreSQL QUERY规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/3.PostgreSQL%20QUERY%E8%A7%84%E8%8C%83.md)
- [4.PostgreSQL 管理规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/4.PostgreSQL%20%E7%AE%A1%E7%90%86%E8%A7%84%E8%8C%83.md)
- [5.PostgreSQL 稳定性与性能规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/5.PostgreSQL%20%E7%A8%B3%E5%AE%9A%E6%80%A7%E4%B8%8E%E6%80%A7%E8%83%BD%E8%A7%84%E8%8C%83.md)
- [6.PostgreSQL 阿里云RDS PostgreSQL 使用规范](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E8%A7%84%E8%8C%83/6.PostgreSQL%20%E9%98%BF%E9%87%8C%E4%BA%91RDS%20PostgreSQL%20%E4%BD%BF%E7%94%A8%E8%A7%84%E8%8C%83.md)



- [Geotools连接PostgreSQL数据库](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/Geotools%E8%BF%9E%E6%8E%A5PostgreSQL%E6%95%B0%E6%8D%AE%E5%BA%93.md)
- [PostgreSQL常用SQL](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/PostgreSQL%E5%B8%B8%E7%94%A8SQL.md)
- [JDBC与PostgreSQL（一）](https://gitee.com/AiShiYuShiJiePingXing/postgres/blob/master/13.PostgreSQL%20%E5%BC%80%E5%8F%91%E4%B8%8E%E4%BD%BF%E7%94%A8/JDBC%E4%B8%8EPostgreSQL%EF%BC%88%E4%B8%80%EF%BC%89.md)



# 四、已归类文档 ——此部分请访问德哥github查阅

因为此部分，我看了有些内容没有，遂直接做了删减。

digoal's|PostgreSQL|文章|归类
---|---|---|---
**[1 应用开发](class/1.md)** | **[2 日常维护](class/2.md)** | **[3 监控](class/3.md)** | **[4 备份,恢复,容灾](class/4.md)**    
**[5 高可用](class/5.md)** | **[6 安全与审计](class/6.md)** | **[7 问题诊断与性能优化](class/7.md)** | **[8 流式复制](class/8.md)**    
**[9 读写分离](class/9.md)** | **[10 水平分库](class/10.md)** | **[11 OLAP(MPP...)](class/11.md)** | **[12 数据库扩展插件](class/12.md)**    
**[13 版本新特性](class/13.md)** | **[14 内核原理与开发](class/14.md)** | **[15 经典案例](class/15.md)** | **[16 HTAP](class/16.md)**    
**[17 流式计算](class/17.md)** | **[18 时序、时空、对象多维处理](class/18.md)** | **[19 图式搜索](class/19.md)** | **[20 GIS](class/20.md)**    
**[21 Oracle兼容性](class/21.md)** | **[22 数据库选型](class/22.md)** | **[23 Benchmark](class/23.md)** | **[24 最佳实践](class/24.md)**       
**[25 DaaS](class/25.md)** | **[26 垂直行业应用](class/26.md)** | **[27 标准化(规约、制度、流程)](class/27.md)** | **[28 版本升级](class/28.md)**    
**[29 同、异构数据同步](class/29.md)** | **[30 数据分析](class/30.md)** | **[31 系列课程](class/31.md)** | **[32 其他](class/32.md)**    
**[33 招聘与求职信息](class/33.md)** | **[34 沙龙、会议、培训](class/34.md)** | **[35 思维精进](class/35.md)** | **[36 视频回放](class/36.md)**    

# 五、批评指正

因大部分资料都是互联网资源，如有侵权，请联系删除。

对于错误或者不当之处，劳烦及时指正，会尽快更改。

再次多谢PG大佬们的辛勤付出！！！