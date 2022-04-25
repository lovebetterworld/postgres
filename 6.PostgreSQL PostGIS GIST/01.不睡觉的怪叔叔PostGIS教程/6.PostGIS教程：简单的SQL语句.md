# PostGIS教程六：简单的SQL语句

  SQL，或"Structured Query Language-**结构化查询语言**"，是对关系数据库进行查询数据和更新数据的一种方法。

  当我们创建第一个数据库时，你已经看到了SQL：

```sql
SELECT postgis_full_version();
```

  查看PostGIS的版本信息。

  在前面的章节中，我们已经将数据加载到数据库中，现在让我们使用SQL来查询数据！例如：

​    "查看**纽约市**所有社区的名字？"

  通过单击SQL按钮在pgAdmin中打开SQL查询窗口：

![img](https://img-blog.csdnimg.cn/20181225103535963.png)

  然后在查询窗口中输入以下查询语句：

```sql
SELECT name FROM nyc_neighborhoods;
```

  并点击**执行查询**按钮：

 ![img](https://img-blog.csdnimg.cn/20181225103746180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  查询将运行几毫秒并返回129个结果：

![img](https://img-blog.csdnimg.cn/2018122510430658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  但这个过程中到底发生了什么？为了理解，让我们从SQL的四个”动词“开始：

- **SELECT**  ——  返回查询的行记录
- **INSERT**  ——  向表中添加新行记录
- **UPDATE**  ——  更改表中的现有行记录
- **DELETE**  ——  从表中删除行记录

  我们几乎全部使用SELECT语句来使用空间函数，所以着重介绍SELECT语句。

# 一、SELECT查询

   SELECT查询通常采用以下形式：

```sql
SELECT some_columns FROM some_data_source WHERE some_condition;
```

  **注意**：有关所有SELECT语句参数的概要，请参阅[PostgreSQL文档](http://www.postgresql.org/docs/current/interactive/sql-select.html)

  some_columns既可以是**列名**也可以是**列值**的函数，some_data_source既可以是单个表，也可以是通过连接两个表而创建的**组合表**。some_condition相当于一个过滤器，它限制要返回的行数。

​     ”查看**布鲁克林**所有社区的名字？“

  我们使用一个**过滤器**来查询nyc_neighborhoods表，这张表内包含了纽约所有的街区信息，但我们只想要查看属于**布鲁克林（行政区）**的那些社区：

```sql
SELECT name
FROM nyc_neighborhoods
WHERE boroname = 'Brooklyn';
```

  查询将只花费比上次查询更少的时间，并返回23个结果：

![img](https://img-blog.csdnimg.cn/20181225112811558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  有时，我们需要对查询的结果应用一个函数，例如：

​    "**布鲁克林**所有社区的名字里各有多少个字母?"

  幸运的是，PostgreSQL有一个**字符串长度函数**，char_length(string)：

```sql
SELECT char_length(name)
  FROM nyc_neighborhoods
  WHERE boroname = 'Brooklyn';
```

![img](https://img-blog.csdnimg.cn/2018122511400368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  通常，我们对单个行记录数据不感兴趣，而对基于所有行记录数据的统计数据更感兴趣。

  因此，知道各个社区名字的长度可能不如知道所有社区名字的长度的平均值有趣。

  接受多行记录并返回单个结果的函数称为“**聚合**（aggregate）函数"。

  PostgreSQL有一系列内置的**聚合函数**，包括求**平均值**的avg()函数和求**标准差**的stddev()函数。

​    “**布鲁克林**所有社区名字的**平均字母数**和字母数的**标准差**是多少？”

```sql
SELECT avg(char_length(name)), stddev(char_length(name))
FROM nyc_neighborhoods
WHERE boroname = 'Brooklyn';
```

![img](https://img-blog.csdnimg.cn/20181225114113247.png)

![img](https://img-blog.csdnimg.cn/2018122511422189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  上面示例中的**聚合函数**应用于结果记录集中的每一行。

  如果我们希望在整个结果集中对各个**子数据集**分组进行处理，该怎么办？

  我们可以使用**GROUP BY**子句。

  **聚合函数**通常需要添加GROUP BY语句，以便基于一个或多个列对结果记录集进行分组。

​    “基于各个**行政区**进行分组，**纽约市**各个**行政区**的所有**社区**名字的平均字母数是多少？”

```sql
SELECT boroname, avg(char_length(name)), stddev(char_length(name))
FROM nyc_neighborhoods
GROUP BY boroname;
```

  我们将boroname列包含在输出结果中，以便确定哪个统计数据对应于哪个**行政区**。

  在**聚合查询**中，只能输出GROUP BY子句对应的列或**聚合函数**对应的列。

![img](https://img-blog.csdnimg.cn/20181225115122936.png)

![img](https://img-blog.csdnimg.cn/20181225115316821.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 二、简单SQL语句的练习

  使用nyc_census_blocks表，回答以下问题（不要急着回答！）。

  下面是一些有用的信息，回想一下上一章节中的nyc_census_blocks表的定义。

![img](https://img-blog.csdnimg.cn/2018122609114872.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  下面是一些常见的**SQL聚合函数**，你可能会发现它们很有用：

- avg()  ——  返回一个数值列的平均值
- sum()  ——  返回一个数值列的和
- count()  ——  返回一个列的记录数

  下面是问题：

​    第一个问题："纽约市的总人口是多少？"

​    答案：

```sql
SELECT Sum(popn_total) AS population
  FROM nyc_census_blocks;
```

  ![img](https://img-blog.csdnimg.cn/2018122609184233.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  **注意**：关键字**AS**是什么意思？AS用于为**表**或**列**指定**别名**。设置合适的别名可以使查询结果更容易理解。与本来要输出的列名sum不同，我们使用AS关键字将其改为population。

  第二个问题："**布朗克斯**（Bronx）行政区的总人口是多少？"

  答案：

```sql
SELECT Sum(popn_total) AS population
  FROM nyc_census_blocks
  WHERE boroname = 'The Bronx';
```

![img](https://img-blog.csdnimg.cn/20181226092626829.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

  第三个问题："对每个行政区来说，白人占该行政区总人口的百分比是多少？"

  答案：

```sql
SELECT
  boroname,
  100 * Sum(popn_white)/Sum(popn_total) AS white_pct
FROM nyc_census_blocks
GROUP BY boroname;
```

![img](https://img-blog.csdnimg.cn/20181226093130545.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NzMyMTQ3,size_16,color_FFFFFF,t_70)

# 三、本文涉及函数的列表

- [avg(expression) ](http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE)  ——   返回一个数值列的**平均值**的PostgreSQL聚合函数
- [char_length(string)](http://www.postgresql.org/docs/current/static/functions-string.html)  ——  返回字符串中的**字符数**
- [stddev(expression)](http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE)  ——  返回输入值的**标准差**的PostgreSQL聚合函数 
- [count(expression)](http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE)  ——  返回一个列的**记录数**的PostgreSQL聚合函数
- [sum(expression)](http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE)  ——  返回一个数值列的和的PostgreSQL聚合函数