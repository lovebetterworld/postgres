# 一、仓库说明

本来是想直接同步德哥的pg仓库，但是奈何德哥的仓库太大了，都无法直接从GitHub迁移到gitee。但是看到德哥的许多笔记十分有用，于是花了很久时间，把德哥的笔记都过了两遍。

第一遍，发现德哥有很多私人笔记在里面，并不是有关于PostgreSQL相关的，手动做了剔除。

第二遍，发现笔记太过于混乱，不太好明确的找到相应的内容，遂自己做了一个简单的目录。

同时，也将该仓库作为自己学习和研究PostgreSQL的仓库，会不断积累和分享有关PostgreSQL的内容。



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

# 三、学习资料

[1.PostgreSQL 学习系列](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/1.PostgreSQL%20%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97)

[2.PostgreSQL 优势](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/2.PostgreSQL%20%E4%BC%98%E5%8A%BF)

[3.PostgreSQL 笔记](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/3.PostgreSQL%20%E7%AC%94%E8%AE%B0)

[4.PostgreSQL 课程](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/4.PostgreSQL%20%E8%AF%BE%E7%A8%8B)

[5.PostgreSQL 案例](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/5.PostgreSQL%20%E6%A1%88%E4%BE%8B)

[6.PostgreSQL PostGIS GIST](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/6.PostgreSQL%20PostGIS%20GIST)

[7.PostgreSQL 推荐系统](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/7.PostgreSQL%20%E6%8E%A8%E8%8D%90%E7%B3%BB%E7%BB%9F)

[8.PostgreSQL 应用开发解决方案](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/8.PostgreSQL%20%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)

[9.PostgreSQL 插件](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/9.PostgreSQL%20%E6%8F%92%E4%BB%B6)

[10.PostgreSQL 配置文件](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/10.PostgreSQL%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)

[11.PostgreSQL PDF资料文档](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/11.PostgreSQL%20PDF%E8%B5%84%E6%96%99%E6%96%87%E6%A1%A3)

[12.PostgreSQL 安装与部署](https://gitee.com/AiShiYuShiJiePingXing/postgres/tree/master/12.PostgreSQL%20%E5%AE%89%E8%A3%85%E4%B8%8E%E9%83%A8%E7%BD%B2)



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