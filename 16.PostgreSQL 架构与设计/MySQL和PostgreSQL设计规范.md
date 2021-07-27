- [MySQL和PostgreSQL设计规范](https://blog.csdn.net/Vincent2014Linux/article/details/105792127)



# 1.设计工具

使用Navicat Data Modeler进行数据库设计，使用*.ndml文件交流设计细节，不允许直接操作数据库进行修改

修改数据库一定要同步更新本地的.ndml文件，避免因开发环境异常而导致数据库丢失，demo测试数据或初始化数据要以sql脚本协议代码中*

# 2.命名规范

## 2.1 数据库名

- 使用蛇底式命名法(snake_case)
- [部门名]_项目名, 部门名可省略

## 2.2 表名

- 使用蛇底式命名法(snake_case)
- 名词表名应使用单数形式 (辨认、书写一个单词的复数形式，更困难)
- 表名尽量简短 (例如用户表，表名应为user，而非user_info, 当伴随多个附属表时便于定位主表)
- 表名通常使用名词

## 2.3 字段名

- 使用蛇底式命名法(snake_case)
- 字段名尽量简短
- 可使用众所周知的单词简写(例如 desc)
- 不可使用保留字
- class, desc等

## 2.4 通用字段

特殊含义字段，除非特别需要，请使用约定名称

- id 表的主键为id, 不需要增加类似user，device之类的前缀，例如userid
- 表名_id 逻辑上的外键id，不建议使用外键定义
- name 记录的名称，是id的别名，一般用于友好显示
- create_time 记录的创建时间
- update_time 记录的更新时间
- expire_time 记录的过期时间
- is_deleted 记录的删除标记
- desc 记录的描述，描述通常有长度限制，是对name的进一步说明
- comment 记录的备注，通常是备忘信息，长度较长



# 3.数据类型规范

- 优先选用数值类型
- 如果可以尽量不允许输入null

## 3.1 数字类型选择

- 估算范围优先选择存储空间小的类型 (降低数据存储，降低索引存储)
- id应使用自增类型
- 对与金融货币应采用准确的定点型 (防止计算误差)
- 用整型代替boolean类型 (mysql boolean实际实现为tinyint(1),不会节省空间，但会降低扩展性)
- 需要进行模糊查询时不应使用数字类型, 而应使用字符串(例如，身份证号，电话号)

## 3.2 字符串与字节流

### MYSQL

- 长度变化不大的字符串用char
- 长度变化较大的字符串用varchar
- 需要有默认值使用varchar，不选用text
- 能用varchar，不用text
- 非可视字符数据应使用字节流类型

### POSTGRESQL

- 尽量使用text
- 需要限定最大长度时使用varchar

### 时间时期类型

- 只需要时分秒或年月日的情况下可以选择date
- 需要秒级精度时尽量选用timestamp类型, 不建议使用time(可使用数据库的相关时间函数，但需注意对数据库服务器的时区依赖)
- 可使用bigint
- 不允许使用varchar

```plsql
## 整型
SQL标准             -                             smallint                        -            integer                   bigint  
postgresql 有符号  char(1)[1B,charactor(1)]替代   smallint[2B,int2]         char(3)替代        integer[4B,int,int4]      bigint[8B,int8]
           自增                                   smallserial[2B,serial2]                      serial[4B,serial4]        bigserial[8B,serial8]  
           postgreql serial并非真实类型, 内部实现为一个integet+一个自增serquence实现，可用\d 查看表结构检查
mysql              tinyint(1字节)                 smallint(2字节)           mediumint(3字节)   int(4字节)                bigint(8字节)  

## 浮点型  
postgresql         real,float4(不准确,4字节,6位)     double,float8(不准确,8字节,15位)     decimal,numeric(准确，变长，任意精度，最多1000)
mysql              float(4字节,6位)                  double(8字节,15位)                   decimal(16字节，准确，定点)

## 字符型
postgresql         char(n),character(n)(定长,不足补白,最大1GB)       varchar(n),character varying(n)(变长,最大1GB)         text(变长, 无长度限制)
                   注：postgres中char与varchar无性能差别，但char可能比varchar多占用空间
mysql              char(n)(定长,不足补白,最大255字节)                varchar(n)(变长,v5.0.3最大255之后最大65535)           tinytext(最大255)     text(最大65535)   mediumtext(最大16777215)   longtext(最大429496729)
                   注：mysql中char比varchar性能更高     
                   
## 布尔型
postgresql         bool,boolean(从扩展性角度，不建议使用)  
mysql              bool,boolean(从扩展性角度，不建议使用)
                   
## 时间日期  
postgresql                            date(4字节, 年月日)    time(8字节，时分秒毫秒)                         timestamp(8字节)                      timestamptz(8字节, 时间戳带时区)         timetz(12字节, 时间戳带时区)    interval(12字节, 时间间隔)
mysql              year(1字节, 年)    date(3字节, 年月日)    time(3字节, 时分秒)         datetime(8字节)     timestamp(8字节, 有自动更新特性)

## 字节流
postgresql         bytea(变长，字节流) 
mysql              tinyblob   blob      mediumblob  longBlob

## 特殊数据类型  
postgres  
  money 数据库级别提供并发机制，准确精确计算，但输出与币种有关，且数据库全局设置，不建议使用
  bit  bitvar,bit varying 位串类型，可在位级别进行长度控制
```



# 4.索引选择

## 4.1 索引添加

- 数据表id必须加索引
- 表示唯一数据的字段应该加索引(除非确定肯定不会在查询中用到)
- 可能作用筛选where语句，排序order by的部分必须加索引

## 4.2 索引类型

Hash索引: 内存占用多, 但单条记录查询效率最高，不支持排序
B-Tree索引: 查询效率较高，支持范围查询及排序

- 需要支持排序的索引须使用B-Tree索引，根据场景需注意B-Tree的排序设置
- 需要声明唯一性的索引须使用B-Tree索引
- 需要比较，判等操作时使用B-Tree索引
- 字段值特别长，且只需要等值查询时使用Hash索引
- 按时序插入的数据，且只需要等值及范围查询时使用BRIN索引
- 空间几何类型数据使用GiST或SP-GiST索引
- 全文检索使用BRIN索引



# 5.备注规范

## 5.1 表备注

- 表名不能准确表达表用途时应备注说明

## 5.2 字段备注

- 具有量纲单位的字段，应在备注中给出字段对应的单位
- 枚举或编码数据给出常见值及对应的含义
- 字段名如果使用缩写，备注应给出缩写全称



# 6.数据引擎选择(PostgreSQL不需考虑)

不同的数据库对数据库引擎的支持不同，因此考虑数据库引擎选择的优先级别规范

## 6.1 常见MySQL数据库

MySQL(闭源风险) ==> Infobright (mysql的数据仓库方案)
MariaDB
Percona
AliSQL
postgres ==> Greenplum(postgres的数据仓库方案)

## 6.2 常见数据库引擎使用优先级

Xtradb=>InnoDB (全事务型存储引擎，事务方面支持较好)
TokuDB (新型事务型存储引擎，写入性能高，存储压缩比高，适合写多读少的场景)
Archive (适合归档类数据，读少，写多，可考虑用数据仓库类方案)
MEMORY (适合可忍受丢失，且访问频繁的数据，可用redis，memcached等替代方案)
Aria(原名Maria引擎)=>MyISAM (MYSQL8.0开始不再维护，不推荐使用)
SphinxSE引擎 (是和全文搜索)
# 7.开发测试环境与部署环境分离

- 开发测试环境
  - 服务器：测试用例统一使用数据库，如 10.0.30.201
  - 注意：数据随时删除不应影响测试，测试用例代码包含所需测试数据
- 演示与生产环境
  - 服务器：如默认使用10.0.30.209 ，具体视实际情况而定
  - 注意：应有可靠备份，数据不因误删除而影响