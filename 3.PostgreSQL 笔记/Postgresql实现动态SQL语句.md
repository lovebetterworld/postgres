- [Postgresql实现动态SQL语句](https://blog.csdn.net/neweastsun/article/details/113773212)

本文介绍Postgresql如何实现动态SQL语句。

## 1. 动态SQL

动态SQL在程序启动时会根据输入参数替换相应变量。使用动态SQL可以创建更强大和灵活的应用程序，但在编译时SQL语句的全文不确定，因此运行时编译会牺牲一些性能。动态SQL可以是代码或SQL语句的一部分，动态部分要么由开发人员输入，要么由程序本身创建。

### 1.1 动态SQL使用场景

在PL/pgSQL函数或过程中有时需要生成动态命令，因为命令涉及不同表或数据类型，仅在运行时才能确定具体对象或值。这时比较适合使用动态SQL。

另外，在特定情况下，如果静态SQL语句无法执行，或者您真的不知道函数或过程要执行的确切SQL语句，那么您必须使用动态SQL。

### 1.2 动态SQL VS 静态SQL

| 动态SQL                                                      | 静态SQL                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| SQL语句在编译时编译                                          | SQL语句在运行时编译                                          |
| SQL语句在编译时进行解析，有效性检查表、视图和过程。编译时优化并生成应用程序执行计划 | SQL语句在运行时进行编译解析SQL语句，有效性检查表、视图和过程，优化并生成应用程序执行计划 |
| 性能好，速度快                                               | 与静态SQL相比较性能不好                                      |
| 不够灵活                                                     | 非常灵活                                                     |

## 2. 生成动态SQL

共有三种方式实现动态SQL。使用format函数，使用quote_ident 和 quote_literal函数，使用连接操作符`||`(发音pipe)，我最喜欢使用第一种，下面分别进行说明。

注意：构建查询时插入的动态值需要仔细处理，因为可能需要包含引号。

### 2.1 使用format函数

首先我们介绍下format函数的形式参数：

%s s格式化参数值作为简单字符串.
 %I I 处理参数值作为SQL 标识符，有必要增加双引号.
 %L L 引用参数作为SQL字面值.

这里先要区分两个概念：SQL标识符和SQL字面值。

SQL标识符表示数据库名称、表名、索引名、schema名、约束名、游标名、触发器、列、视图名称。在动态SQL中，I%会按照SQL标识符进行解析。

SQL字面值表示显示值，数值、字符、字符串、布尔值，不代表SQL标识符。有不同类型的字面值：

| 类型           | 举例                                           |
| -------------- | ---------------------------------------------- |
| 字符串 String  | ‘Hello! everyone’                              |
| 整数 Integer   | 45, 78, +89 , -465,6E5                         |
| 数值 Decimal   | 45.56                                          |
| 日期 DateTime  | ‘5/20/2020’ , TIMESTAMP ‘2020-05-20 12:01:01’; |
| 字符 Character | A’ ‘%’ ‘9’ ’ ’ ‘z’ ‘(’                         |
| 布尔 Boolean   | true, false, null                              |

下面举例说明：

```
SELECT format('Hello! Welcome to my %s!', 'Blog') as msg;
```

返回：`Hello! Welcome to my Blog!`

```
SELECT format('%s! Welcome to my %s! - %s','Hi','Blog','Ourtechroom') as msg;
```

返回 ：`Hi! Welcome to my Blog! - Ourtechroom`

```
SELECT format('INSERT INTO %I VALUES(%L)', 'tbl_test', 'test');
```

返回：`INSERT INTO tbl_test VALUES('test')`

这里%I 被替换为`tbl_test`，%L被替换为`'test'`

### 2.2 使用quote_indent 函数

PostgreSQL 的quote_indent 函数

```
quote_ident('Hello World');  // "Hello World" 字面量增加引号

quote_ident('mytable');  // mytable 表名称自动去掉引号

quote_ident('MyTable');  // MyTable 区分大小写，表名称自动去掉引号
```

除此之外还有几个类似函数。 QUOTE_LITERAL(string text), QUOTE_LITERAL(value anyelement), QUOTE_NULLABLE(value anyelement)；

QUOTE_LITERAL函数返回值自动增加引号。QUOTE_NULLABLE对于非空参数增加引号，否则返回null。

### 2.3 使用连接操作符`||`

举例：

```
 select ' be.id from ' || tbl_name || ' t where m.commodity_id = ' || sorttype  || ' order by t.amount ' ||  test_id  ||;
```

这种方式对于需要增加单引号比较麻烦，且容易造成SQL注入。

## 3. 总结

本文介绍Postgresql三种方式实现动态SQL语句，并通过示例对比不同方式的差异。相比使用format方式更简单高效。