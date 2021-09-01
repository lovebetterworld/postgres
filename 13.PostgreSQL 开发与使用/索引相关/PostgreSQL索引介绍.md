# INDEX

索引是增强数据库性能的常用方法。索引使得数据库在查找和检索数据库的特定行的时候比没有索引快的多。但索引也增加了整个数据库系统的开销，所以应该合理使用。

## 介绍

假设我们有一个类似这样的表：

```
CREATE TABLE test1 (
    id integer,
    content varchar
);
```

应用程序发出许多类似以下的这种查询：

```
SELECT content FROM test1 WHERE id = constant;
```

没有提前的准备，系统将不得不逐行扫描整个test1表，以查找所有匹配的条目。如果test1中有很多行，并且这样的查询仅仅返回几行（可能是零或一行），这显然是一种低效的方法。但是如果系统已被指示在id列上维护索引，则可以使用更有效的方法来定位匹配的行。例如，它可能只需要在搜索树中深入几层。

大多数非小说类书中都使用类似的方法：读者经常查阅的术语和概念在本书末尾以字母索引的形式收集。感兴趣的读者可以相对快速地扫描索引并翻转到适当的页面，而不必阅读整本书以找到感兴趣的材料。正如作者的任务是预测读者可能会查找的项目，数据库程序员的任务是预见哪些索引将是有用的。

可以使用下边的命令可以在id列上创建一个索引：

```
CREATE INDEX test1_id_index ON test1 (id);
```

名称test1_id_index可以自由选择，但您应该选择一些可以让您记住索引的内容的名称。

要删除索引，请使用DROP INDEX命令。索引可以随时添加到表中并从表中删除。

一旦创建了索引，就不需要进一步的干预：当表被修改时，系统将更新索引，当planner认为使用索引比顺序的表扫描更有效的时候，它会使用索引。但是，您可能需要定期运行ANALYZE命令来更新统计信息，以允许查询计划者做出有根据的决策。有关如何确定索引是否被使用以及计划者何时以及为什么选择不使用索引的信息，请参阅[第14章](https://www.postgresql.org/docs/9.6/static/performance-tips.html)。

具有搜索条件的UPDATE和DELETE命令的查询条件也可以使用索引来优化。索引也可以用于连接搜索。因此，在作为连接条件一部分的列上定义的索引也可以显著加快使用连接的查询。

在大型表上创建索引可能需要很长时间。默认情况下，PostgreSQL允许读表（SELECT语句）与索引创建并行发生，但写入（INSERT，UPDATE，DELETE）将被阻止，直到索引生成完成。在生产环境中，这通常是不能接受的。可以设置允许写入与索引创建并行发生，但有几个需要注意的注意事项 - 有关更多信息，请参阅[并发构建索引](https://www.postgresql.org/docs/9.6/static/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY)。

创建索引后，系统必须保持索引与数据表的同步。这增加了数据操作操作的开销。因此，查询中很少或从未使用的索引应该被删除。

## 索引类型

PostgreSQL提供了几种索引类型：B-tree，Hash，GiST，SP-GiST，GIN和BRIN。每个索引类型使用不同的算法，适合不同种类的查询。默认情况下，CREATE INDEX命令创建B-tree索引，这符合最常见的情况。

B-tree可以处理对可以排序成某些顺序的数据的等式和范围查询。特别地，当索引列参与使用以下运算符之一的比较时， PostgreSQL查询计划器将考虑使用B-tree索引：

```
<
<=
=
>=
>
```

也可以使用B-tree索引搜索来实现与这些运算符的组合相同的构造，如BETWEEN和IN。此外，索引列上的IS NULL或IS NOT NULL条件可以与B-tree索引一起使用。

对于涉及模式匹配运算符LIKE的查询，优化器还可以使用B-tree索引，如果模式是常量，并且锚定到字符串的开头，例如col LIKE  'foo％'或 col〜'^ foo'，但不能是col  LIKE'％bar'。但是，如果您的数据库不使用C语言环境，则需要使用特殊的运算符类创建索引，以支持对模式匹配查询的索引;见下文[第11.9节](https://www.postgresql.org/docs/9.6/static/indexes-opclass.html)。也可以对 ILIKE和〜*使用B-tree索引，但只有当模式以非字母字符（即不受大小写转换影响的字符）开始时才可以。

B-tree索引也可用于按排序顺序检索数据。这并不总是比简单的扫描和排序更快，但它通常是有益的。

Hash索引只能处理简单的等式比较。当使用=运算符进行比较时，查询计划器将考虑使用Hash索引。以下命令用于创建Hash索引：

```
CREATE INDEX name ON table USING HASH (column);
```

Hash索引操作目前不记录WAL-log，所以如果有没有写入的更改，Hash索引可能需要在数据库崩溃后用REINDEX重建。此外，在初始基本备份之后，不会通过流式或基于文件的复制来复制Hash索引的更改，因此它们对随后使用它们的查询给出错误的答案。由于这些原因，目前不鼓励使用Hash索引。

GiST索引不是一种单一的索引，而是可以实现许多不同索引策略的基础设施。因此，可以使用GiST索引的特定运算符根据索引策略（运算符类）而变化。例如，PostgreSQL的标准发布版包括几个二维几何数据类型的GiST运算符类，它们支持使用这些运算符的索引查询：

```
<<
&<
&>
>>
<<|
&<|
|&>
|>>
@>
<@
~=
&&
```

（有关这些操作符的含义，请参见[第9.11节](https://www.postgresql.org/docs/9.6/static/functions-geometry.html)。）标准发布版中包含的GiST操作员类别见[表61-1](https://www.postgresql.org/docs/9.6/static/gist-builtin-opclasses.html#GIST-BUILTIN-OPCLASSES-TABLE)。许多其他GiST操作员类别可以在contrib集合中或单独的项目中使用。有关更多信息，请参阅[第61章](https://www.postgresql.org/docs/9.6/static/gist.html)。

GiST索引还能够优化“nearest-neighbor”搜索，例如:

```
SELECT * FROM places ORDER BY location <-> point '(101,456)' LIMIT 10;
```

它找到最接近给定目标点的十个位置。这样做的能力又取决于所使用的特定操作符类。在[表61-1](https://www.postgresql.org/docs/9.6/static/gist-builtin-opclasses.html#GIST-BUILTIN-OPCLASSES-TABLE)中，可以以这种方式使用的操作员列在“Ordering Operators”一栏中。

SP-GiST索引（类似GiST索引）提供支持各种搜索的基础架构。  SP-GiST允许实现各种不同的不平衡的基于磁盘的数据结构，例如四叉树，k-d树和基数树（尝试）。例如，PostgreSQL的标准发布版包括用于二维点的SP-GiST运算符类，它们支持使用这些运算符的索引查询：

```
<<
>>
~=
<@
<^
>^
```

（有关这些操作符的含义，请参见[9.11节](https://www.postgresql.org/docs/9.6/static/functions-geometry.html)。）标准配置中包含的SP-GiST操作员类别见[表62-1](https://www.postgresql.org/docs/9.6/static/spgist-builtin-opclasses.html#SPGIST-BUILTIN-OPCLASSES-TABLE)。有关更多信息，请参见[第62章](https://www.postgresql.org/docs/9.6/static/spgist.html)。

GIN索引是适用于包含多个组件值的数据值（如数组）的“反向索引”。反向索引包含每个组件值的单独条目，并且可以有效地处理测试特定组件值的存在的查询。

像GiST和SP-GiST一样，GIN可以支持许多不同的用户定义的索引策略，并且可以使用GIN索引的特定运算符根据索引策略而变化。例如，PostgreSQL的标准发布版包括一维数组的GIN运算符类，它们支持使用这些运算符的索引查询：

```
<@
@>
=
&&
```

（有关这些操作符的含义，请参见[第9.18节](https://www.postgresql.org/docs/9.6/static/functions-array.html)。）标准分配中包含的GIN运算符类别见[表63-1](https://www.postgresql.org/docs/9.6/static/gin-builtin-opclasses.html#GIN-BUILTIN-OPCLASSES-TABLE)。许多其他GIN操作员类可以在contrib集合中或单独的项目中使用。更多信息，请参见[第63章](https://www.postgresql.org/docs/9.6/static/gin.html)。

BRIN索引（Block Range  INdexes的缩写）存储关于存储在表的连续物理块范围内的值的摘要。像GiST，SP-GiST和GIN一样，BRIN可以支持许多不同的索引策略，并且可以使用BRIN索引的特定操作符根据索引策略而变化。对于具有线性排序顺序的数据类型，索引数据对应于每个块范围的列中值的最小值和最大值。这支持使用这些运算符的索引查询：

```
<
<=
=
>=
```

[表64-1](https://www.postgresql.org/docs/9.6/static/brin-builtin-opclasses.html#BRIN-BUILTIN-OPCLASSES-TABLE)列出了标准配置中包含的BRIN运算符类。有关更多信息，请参见[第64章](https://www.postgresql.org/docs/9.6/static/brin.html)。

## 多列索引

可以在表的多个列上定义索引。例如，如果您有一张类似的数据表：

```
CREATE TABLE test2 (
  major int,
  minor int,
  name varchar
);
```

（比如，你的/ dev目录保存在数据库中...），你经常发出如下的查询：

```
SELECT name FROM test2 WHERE major = constant AND minor = constant;
```

那么在major和minor列上定义一个索引可能是合适的：

```
CREATE INDEX test2_mm_idx ON test2 (major, minor);
```

目前，只有B-tree，GiST，GIN和BRIN索引类型支持多列索引。最多可以指定32列。 （构建PostgreSQL时可以更改此限制;请参阅pg_config_manual.h文件。）

多列B树索引可以用于涉及索引列的任何子集的查询条件，但是当前导（最左侧）列存在约束时，索引效率最高。确切的规则是，前导列上的等式约束以及不具有相等约束的第一列上的任何不等式约束将用于限制扫描的索引部分。在索引中检查这些列右侧的列的约束，因此它们保存对表的访问，但它们不会减少必须扫描的索引部分。例如，给定（a，b，c）上的索引和查询条件WHERE a = 5 AND b> = 42 AND c <77，必须从a = 5和b = 42通过最后一个条目，a = 5，c> =  77的索引条目将被跳过，但仍需扫描。这个索引原则上可以用于对b和/或c有约束的查询，而对b没有约束，但是整个索引必须被扫描，所以在大多数情况下，计划员会喜欢使用索引进行顺序表扫描。

多列GiST索引可用于涉及索引列的任何子集的查询条件。附加列上的条件限制索引返回的条目，但第一列中的条件是确定需要扫描多少索引的最重要的条件。如果GiST索引的第一列只有几个不同的值，即使附加列中有很多不同的值，GiST索引将相对无效。

多列GIN索引可用于涉及索引列的任何子集的查询条件。与B-tree或GiST不同，索引搜索有效性是相同的，无论查询条件使用哪个索引列。

多列BRIN索引可用于涉及索引列的任何子集的查询条件。像GIN一样，与B-tree或GiST不同，索引搜索的有效性是一样的，无论查询条件使用哪个索引列。在单个表上具有多个BRIN索引而不是一个多列BRIN索引的唯一原因是具有不同的pages_per_range存储参数。

当然，每列必须与适用于索引类型的操作符一起使用;涉及其他操作符的情况将不会考虑使用索引。

多列索引应谨慎使用。在大多数情况下，单列上的索引就足以节省空间和时间。具有三列以上的索引不太可能是有用的，除非表的使用非常风格化。有关不同索引配置的优点的一些讨论，请参见[第11.5节](https://www.postgresql.org/docs/9.6/static/indexes-bitmap-scans.html)和[第11.11节](https://www.postgresql.org/docs/9.6/static/indexes-index-only-scans.html)。

## 索引和ORDER BY

除了简单地查找要由查询返回的行之外，索引可能能够以特定的排序顺序传递它们。这允许在没有单独的排序步骤的情况下履行查询的ORDER  BY规范。在PostgreSQL当前支持的索引类型中，只有B-tree可以产生排序的输出 - 其他索引类型以未指定的实现依赖顺序返回匹配的行。

Planner将通过扫描与规范相匹配的可用索引，或通过以物理顺序扫描表并进行明确排序来考虑满足ORDER  BY规范。对于需要扫描表的大部分的查询，显式排序可能比使用索引更快，因为遵循顺序访问模式需要更少的磁盘I /  O。当只需要读取几行时，索引更有用。一个重要的特殊情况是ORDER BY与LIMIT  n组合：显式排序将必须处理所有数据以识别前n行，但如果存在与ORDER BY匹配的索引，则可以直接检索前n行，而不扫描其余部分。

默认情况下，B-tree索引按照升序存储其条目，最后为null。这意味着对列x上的索引的正向扫描产生满足ORDER BY  x（或更详细地，ORDER BY x ASC NULLS LAST）的输出。索引也可以向后扫描，产生满足ORDER BY x  DESC的输出（或更详细地，ORDER BY x DESC NULLS FIRST，因为NULLS FIRST是ORDER BY  DESC的默认值）。

您可以通过在创建索引时包含ASC，DESC，NULLS FIRST和/或NULLS LAST选项来调整B树索引的排序;例如：

```
CREATE INDEX test2_info_nulls_low ON test2 (info NULLS FIRST);
CREATE INDEX test3_desc_index ON test3 (id DESC NULLS LAST);
```

以null first存储的索引首先可以满足ORDER BY x ASC NULLS FIRST或ORDER BY x DESC NULLS LAST，具体取决于扫描的方向。

您可能会想，为什么要提供所有四个选项，当两个选项以及反向扫描的可能性将涵盖ORDER  BY的所有变体。在单列索引中，这些选项确实是多余的，但是在多列索引中它们可能很有用。考虑（x，y）上的两列索引：如果我们向前扫描，则可以满足ORDER BY x，y，或者如果我们向后扫描，则可以满足ORDER BY x DESC，y DESC。但是可能应用程序经常需要使用ORDER BY x  ASC，y DESC。没有办法从一个简单的索引中获得这个排序，但如果索引被定义为（x ASC，y DESC）或（x DESC，y  ASC），这是可能的。

显然，具有非默认排序顺序的索引是一个相当专门的功能，但有时它们可以为某些查询产生巨大的加速。是否值得维护这样的索引取决于使用需要特殊排序顺序的查询的频率。

## 组合多个索引

单个索引扫描只能使用使用索引列的查询子句和运算符类的运算符，并且与AND结合。例如，给定（a，b）上的索引，如WHERE a = 5 AND b = 6的查询条件可以使用索引，但是像WHERE a = 5 OR b = 6这样的查询无法直接使用索引。

幸运的是，PostgreSQL能够组合多个索引（包括相同索引的多个使用）来处理单次索引扫描无法实现的情况。系统可以跨多个索引扫描形成AND和OR条件。例如，像WHERE x = 42 OR x = 47 OR x = 53 OR x =  99这样的查询可以分解为x上的索引的四个独立扫描，每次使用查询子句之一进行扫描。然后将这些扫描的结果OR化在一起以产生结果。另一个例子是，如果我们在x和y上有单独的索引，那么WHERE x = 5 AND y = 6之类的查询的一个可能的实现是将每个索引与适当的查询子句一起使用，然后将索引结果AND结合起来，以识别结果行。

要组合多个索引，系统将扫描每个所需的索引，并准备内存中的位图，为匹配该索引条件的表行的位置提供报告。然后根据查询的需要将位图进行AND和OR操作。最后，访问并返回实际的表行。按照物理顺序访问表行，因为这是位图的布局方式;这意味着原始索引的任何排序都将丢失，因此如果查询具有ORDER  BY子句，则将需要单独的排序步骤。因为这个原因，并且因为每个额外的索引扫描增加额外的时间，Planner有时会选择使用简单的索引扫描，即使可用的附加索引也可以使用。

除了最简单的应用程序之外，索引的各种组合可能是有用的，数据库开发人员必须进行权衡来决定要提供哪些索引。有时，多列索引是最好的，但有时最好创建单独的索引并依赖索引组合功能。例如，如果您的工作负载包含有时仅涉及列x的混合查询，则有时仅列Y列，有时两列，您可以选择在x和y上创建两个单独的索引，依赖于索引组合来处理查询使用两列。您还可以在（x，y）上创建多列索引。该索引通常比涉及两列的查询的索引组合更有效，但如[第11.3节](https://www.postgresql.org/docs/9.6/static/indexes-multicolumn.html)所述，对于仅涉及y的查询几乎无效，因此不应该是唯一的索引。多列索引和y上的单独索引的组合可以很好地起作用。对于仅涉及x的查询，可以使用多列索引，尽管它可能会较大，因此比x上的索引更慢。最后一个选择是创建所有三个索引，但是如果表的搜索比更新更频繁，并且所有三种类型的查询都是常见的，那么这可能是合理的。如果其中一种查询类型比其他类型少得多，那么您可能会建立只创建最符合常见类型的两个索引。

## 唯一索引

索引也可用于强制列的值的唯一性，或多个列的组合值的唯一性。

```
CREATE UNIQUE INDEX name ON table (column [, ...]);
```

目前，只有B-tree索引可以被声明为唯一的。

当索引声明为唯一时，不允许具有相等索引值的多个表行。空值不被认为是相等的。多列唯一索引将仅拒绝所有索引列在多行中相等的情况。

当为表定义唯一的约束或主键时，PostgreSQL会自动创建唯一的索引。该索引涵盖构成主键或唯一约束的列（如果适用，则为多列索引），并且是强制约束的机制。

注意：不需要在唯一列上手动创建索引;这样做只会重复自动创建的索引。

## 表达式上的索引

索引列不必仅仅是基础表的列，也可以是从表的一个或多个列计算的函数或标量表达式。此功能对于根据计算结果快速访问表是非常有用的。

例如，进行区分大小写比较的常见方法是使用lower()函数：

```
SELECT * FROM test1 WHERE lower(col1) = 'value';
```

如果在lower（col1）函数的结果中定义了一个索引，则该查询可以使用索引：

```
CREATE INDEX test1_lower_col1_idx ON test1 (lower(col1));
```

如果我们要声明这个索引UNIQUE，它将阻止创建其col1值的小写相同的行以及col1值实际上相同的行。因此，表达式上的索引可以用于强制不能被定义为简单的唯一约束的约束。

另一个例子，如果一个人经常进行如下查询：

```
SELECT * FROM people WHERE (first_name || ' ' || last_name) = 'John Smith';
```

那么，可以创建一个这样的索引：

```
CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
```

CREATE INDEX命令的语法通常需要在索引表达式周围写圆括号，如第二个示例所示。 当表达式只是一个函数调用时，可以省略括号，如第一个示例所示。

索引表达式维护相对昂贵，因为必须为插入时的每一行计算导出的表达式，并且每当更新它时。  但是，索引表达式在索引搜索期间不重新计算，因为它们已经存储在索引中。 在上述两个示例中，系统将查询视为WHERE indexedcolumn  =“constant”，因此搜索速度等同于任何其他简单的索引查询。 因此，当检索速度比插入和更新速度更重要时，表达式索引是有用的。

## 部分索引

部分索引是在表的子集上构建的索引; 该子集由条件表达式（称为部分索引的谓词）定义。 该索引仅包含满足谓词的那些表行的条目。 部分索引是一个专门的功能，但有几种情况使用部分索引是非常有用的

使用部分索引的一个主要原因是避免索引常见值。  由于搜索公共值（一个占所有表行的百分之几）的查询将不会使用索引，所以根本没有必要在索引中保留这些行。  这减少了索引的大小，这将加快使用索引的查询。 它也将加快许多表更新操作，因为索引在所有情况下都不需要更新。 例11-1显示了这一想法的可能应用。

Example 11-1。 设置部分索引以排除常见值

假设您将Web服务器访问日志存储在数据库中。 大多数访问源自您组织的IP地址范围，但有些来自其他地方（例如拨号连接上的员工）。 如果您的IP搜索主要用于外部访问，则可能不需要对与组织的子网对应的IP范围进行索引。

假设有像这样的一张表：

```
CREATE TABLE access_log (
    url varchar,
    client_ip inet,
    ...
);
```

使用下面的命令来建立一个符合我们情况的部分索引

```
CREATE INDEX access_log_client_ip_ix ON access_log (client_ip)
WHERE NOT (client_ip > inet '192.168.100.0' AND
           client_ip < inet '192.168.100.255');
```

一个典型的使用这个索引的查询可能是这样的：
 SELECT *
 FROM access_log
 WHERE url = '/index.html' AND client_ip = inet '212.78.10.32';

下边的这个查询不能用到这个索引：
 SELECT *
 FROM access_log
 WHERE client_ip = inet '192.168.100.23';

很明显这种部分索引要求公共值是预先确定的，所以这种部分索引最好用于不改变的数据分布。 可以偶尔重新创建索引以调整新的数据分布，但这增加了维护工作。

部分索引的另一个可能用途是从典型查询工作负载不感兴趣的索引中排除值; 这在例11-2中展示。 这导致与上述相同的优点，但是它防止通过该索引访问“不感兴趣”的值，即使索引扫描在这种情况下可能是有利的。 显然，为这种场景设置部分索引需要大量的关注和试验。

Example 11-2。 设置部分索引以排除不感兴趣的值

如果您有一个包含已计费和未计费订单的表，其中未计费订单占总表的一小部分，而那些是最常访问的行，则可以通过仅在未计费的行上创建索引来提高性能。 创建索引的命令如下所示：

```
CREATE INDEX orders_unbilled_index ON orders (order_nr)
    WHERE billed is not true;
```

一个用到这个索引的可能的查询是：

```
SELECT * FROM orders WHERE billed is not true AND order_nr < 10000;
```

然而，在哪些不包含order_nr的查询上也可以使用到这个索引：

```
SELECT * FROM orders WHERE billed is not true AND amount > 5000.00;
```

由于系统必须扫描整个索引，因此这不会比amount列上的局部索引更高效。 然而，如果有相对较少的未结算订单，使用这个部分索引只是找到未结算的订单可能是比较好的。

注意，下边的查询用不到这一索引：

```
SELECT * FROM orders WHERE order_nr = 3501;
```

3501对应的订单可能是已结算的，也可能是未结算的。

示例11-2还说明了索引列和谓词中使用的列不需要匹配。  PostgreSQL支持具有任意谓词的部分索引，只要包含索引的列。但是，谓词必须匹配在索引中受益的查询中使用的条件。准确地说，只有系统可以识别查询的WHERE条件在数学上意味着索引的谓词时，才能在查询中使用部分索引。 PostgreSQL没有一个复杂的定理证明器，可以识别以不同形式编写的数学等效表达式。  （这样一个一般的定理证明者不但难以产生，所以实际使用太慢）。系统可以识别简单的不平等含义，例如“x <1”表示“x  <2”;否则谓词条件必须与查询的WHERE条件完全匹配，否则索引将不被识别为可用。匹配发生在查询计划期间，而不是运行时。因此，参数化查询子句不适用于部分索引。例如，具有参数的准备好的查询可能指定“x <？”对于参数的所有可能值，这不会暗示“x <2”。

部分索引的第三个可能用途不需要在查询中使用索引。 这里的想法是在表的子集上创建唯一的索引，如例11-3所示。 这会在满足索引谓词的行中强制执行唯一性，而不会限制那些不符合索引谓词的行。

Example 11-3. 设置一个部分唯一索引

假设我们有一个描述测试结果的表格。 我们希望确保给定的主题和目标组合只有一个“成功”条目，但可能会有任何数量的“不成功”条目。 这里有一种方法：

```
CREATE TABLE tests (
    subject text,
    target text,
    success boolean,
    ...
);

CREATE UNIQUE INDEX tests_success_constraint ON tests (subject, target)
    WHERE success;
```

当有很少的成功测试和许多不成功的测试时，这是一种特别有效的方法

最后，部分索引还可以用来覆盖系统的查询计划选择。另外，具有特殊分布的数据集可能导致系统在不应该使用索引时使用索引。在这种情况下，可以设置索引，使其不适用于有问题的查询。通常，PostgreSQL对索引的使用合理的选择（例如，避免他们在检索常用值，所以早期的例子真的节省了索引的大小，它不需要避免索引的使用），错误的计划是导致错误报告的原因

请记住，设置一个分部索引表示您至少知道Planner所知道的情况，特别是知道索引何时可能是有利的。形成这种知识需要经验和理解Postgresql的索引如何工作。在大多数情况下，部分索引相对于常规索引的优势是比较小的。

更多的关于部分索引的信息可以在这里找到：[The case for partial indexes](http://db.cs.berkeley.edu/papers/ERL-M89-17.pdf) , [Partial indexing in POSTGRES: research project](https://www.postgresql.org/docs/9.6/static/biblio.html#OLSON93), 和 [Generalized Partial Indexes (cached version)](https://www.postgresql.org/docs/9.6/static/biblio.html#SESHADRI95) .

## 操作员类和操作员组

索引定义可以为索引的每个列指定一个运算符类。

```
CREATE INDEX name ON table (column opclass [sort options] [, ...]);
```

运算符类标识要由该列的索引使用的运算符。例如，类型为int4的B-tree索引将使用int4_ops类;此运算符类包含类型为int4的值的比较函数。实际上，列的数据类型的默认操作符类通常就足够了。使用运算符类的主要原因是对于某些数据类型，可能有多个有意义的索引行为。例如，我们可能希望按绝对值或实数对复数数据类型进行排序。我们可以通过为数据类型定义两个运算符类，然后在进行索引时选择适当的类来实现。操作员类确定基本排序顺序（可以通过添加排序选项COLLATE，ASC / DESC和/或NULLS FIRST / NULLS LAST）进行修改。

除了默认值之外，还有一些内置的运算符类：

运算符类text_pattern_ops，varchar_pattern_ops和bpchar_pattern_ops分别支持类型为text，varchar和char的B-tree索引。与默认操作符类的区别在于，这些值是严格按字符比较而不是根据特定于区域设置的排序规则进行比较的。当数据库不使用标准“C”语言环境时，这使得这些操作符类适用于涉及模式匹配表达式（LIKE或POSIX正则表达式）的查询。举个例子，你可能像这样索引varchar列：

```
CREATE INDEX test_index ON test_table (col varchar_pattern_ops);
```

请注意，如果您希望涉及普通<，<=，> 或 >=  比较的查询使用索引，您还应该使用默认运算符类创建索引。这样的查询不能使用xxx_pattern_ops运算符类。  （但是普通的等式比较可以使用这些运算符类。）可以在不同的运算符类的同一列上创建多个索引。如果您使用C语言环境，则不需要xxx_pattern_ops运算符类，因为具有默认运算符类的索引可用于C语言环境中的模式匹配查询。

以下查询显示所有定义的运算符类：

```
SELECT am.amname AS index_method,
       opc.opcname AS opclass_name,
       opf.opfname AS opfamily_name,
       opc.opcintype::regtype AS indexed_type,
       opc.opcdefault AS is_default
    FROM pg_am am, pg_opclass opc, pg_opfamily opf
    WHERE opc.opcmethod = am.oid AND
          opc.opcfamily = opf.oid
    ORDER BY index_method, opclass_name;
```

此查询显示所有定义的操作员族和每个系列中包含的所有操作符：

```
SELECT am.amname AS index_method,
       opf.opfname AS opfamily_name,
       amop.amopopr::regoperator AS opfamily_operator
    FROM pg_am am, pg_opfamily opf, pg_amop amop
    WHERE opf.opfmethod = am.oid AND
          amop.amopfamily = opf.oid
    ORDER BY index_method, opfamily_name, opfamily_operator;
```

## 索引和排序规则

索引只能支持每个索引列的一个排序规则。如果感兴趣的是多个排序规则，则可能需要多个索引。

考虑以下的SQL:
 CREATE TABLE test1c (
 id integer,
 content varchar COLLATE "x"
 );

```
CREATE INDEX test1c_content_index ON test1c (content);
```

索引自动使用底层列的排序规则，所以，对于以下的查询，

```
SELECT * FROM test1c WHERE content > constant;
```

可以使用索引，因为默认比较将使用列的排序规则。但是，此索引无法加速涉及其他排序规则的查询。所以如果查询是以下的形式

```
SELECT * FROM test1c WHERE content > constant COLLATE "y";
```

可以创建一个支持“y”排序规则的附加索引，如下所示：

```
CREATE INDEX test1c_content_y_index ON test1c (content COLLATE "y");
```

## 仅索引扫描(就是所谓的覆盖索引)

PostgreSQL中的所有索引都是辅助索引，这意味着每个索引都与表的主数据区域（PostgreSQL术语中称为表的堆）分开存储。这意味着在普通的索引扫描中，每行检索都需要从索引和堆中获取数据。此外，虽然与索引条件匹配的索引条目通常在索引中靠近在一起，但是它们引用的表行可能位于堆中的任何位置。因此，索引扫描的堆访问部分涉及到堆中的大量随机访问，这可能很慢，特别是在传统的旋转介质上。 （如[第11.5节](https://www.postgresql.org/docs/9.6/static/indexes-bitmap-scans.html)所述，位图扫描尝试通过以排序顺序执行堆访问来减轻此成本，但也仅仅做了这些）

为了解决这个性能问题，PostgreSQL支持仅索引扫描(就类似其他数据库中的覆盖索引)，可以单独从索引中响应查询，而无需任何堆访问。基本思想是直接从每个索引条目返回值，而不是查询关联的堆条目。这种方法可以使用两个基本限制：

- 索引类型必须支持仅索引扫描。 B-tree索引总是支持的。  GiST和SP-GiST索引支持一些操作符类的仅索引扫描，而不支持其他操作符。其他索引类型不支持。基本要求是索引必须物理存储或者能够重建每个索引条目的原始数据值。作为反例，GIN索引不能支持仅索引扫描，因为每个索引条目通常只保留原始数据值的一部分。

- 查询必须仅引用索引中存储的列。例如，给定一个也有列z的表的x和y列的索引，这些查询可以使用仅索引扫描：
   SELECT x, y FROM tab WHERE x = 'key';
   SELECT x FROM tab WHERE x = 'key' AND y < 42;
   但是另外的这些不能使用：

  SELECT x, z FROM tab WHERE x = 'key';
   SELECT x FROM tab WHERE x = 'key' AND z < 42;

（表达式索引和部分索引会使此规则复杂化，如下所述）

如果满足这两个基本要求，那么查询所需的所有数据值都可以从索引中获得，因此仅索引扫描在物理上是可行的。但PostgreSQL中的任何表扫描还需要额外的要求：它必须验证每个检索到的行对查询的MVCC快照是“可见的”，如[第13章](https://www.postgresql.org/docs/9.6/static/mvcc.html)所述。可见性信息不存储在索引条目中，仅存储在堆条目中;所以乍一看，似乎每行检索都需要堆访问。如果表行最近被修改的话，的确如此。然而，对于很少变化的数据，有一个方法来解决这个问题。  PostgreSQL跟踪表中堆栈中的每个页面，该页面中存储的所有行是否足够旧，以便对当前和未来的所有事务都可见。该信息存储在表的可见性映射中。仅索引扫描在找到候选索引条目后，检查相应堆页的可见性映射位。如果它被设置，该行是已知可见的，因此可以返回数据，而无需进一步的工作。如果未设置，则必须访问堆条目以确定它是否可见，因此在标准索引扫描中不会获得性能优势。即使在成功的情况下，这种方法交换了对堆访问的可见性映射访问;但是由于可见性映射比它描述的堆小四个数量级，所以需要远远少于物理I / O来访问它。在大多数情况下，可见性映射仍然始终缓存在内存中。

简而言之，在给定两个基本要求的情况下，仅索引扫描是可行的，只有当表的堆页的很大一部分设置了它们的全可见映射位时，才能非常有效。但是大部分行不变的表格通常足以使这种类型的扫描在实践中非常有用。

为了有效利用仅索引扫描功能，您可以选择创建这样的索引，其中只有前导列旨在匹配WHERE子句，而后续列保存要由查询返回的“有效负载”数据。例如，如果您通常会运行这样的查询：

```
SELECT y FROM tab WHERE x = 'key';
```

加速这种查询的传统方法将是仅在x上创建索引。但是，（x，y）上的索引将提供将此查询实现为仅索引扫描的可能性。如前所述，这样的索引将更大，因此比单独的x上的索引更昂贵，因此只有当表已知大部分是静态时才是一个比较好的选择。请注意，索引在（x，y）不是（y，x）上声明很重要，因为大多数索引类型(的复合索引)（特别是B-tree）的搜索在前导搜索列不匹配的时候效率不高。

原则上，仅索引扫描可以与表达式索引一起使用。例如，给定f（x）上的索引，其中x是表列，应该可以执行
 SELECT f(x) FROM tab WHERE f(x) < 1;

作为仅索引扫描;如果f（）是一个昂贵的计算函数，这是非常有吸引力的。然而，PostgreSQL的计划者目前对这种情况做的并不是很好。只有查询所需的所有列都可以从索引中获得时，才会将查询视为可执行的索引扫描。在这个例子中，除了在上下文f（x）中，不需要x，但是规划者并没有注意到，并且认为只有索引扫描是不可能的。如果仅索引扫描似乎是足够有价值的，那么可以通过将索引声明为on（f（x），x）来解决这个问题，其中第二列不期望在实践中使用，但只是在说服计划人员可以进行仅索引扫描。如果目标是避免重新计算f（x），则另外需要注意的是，规划者不一定会与索引列中不能在可索引的WHERE子句中使用f（x）。通常会在如上所示的简单查询中获取此权限，但不包括涉及连接的查询。这些缺陷可能会在以后版本的PostgreSQL中得到补救。

部分索引也与仅索引扫描有趣的交互。考虑[示例11-3](https://www.postgresql.org/docs/9.6/static/indexes-partial.html#INDEXES-PARTIAL-EX3)中所示的部分索引：

```
CREATE UNIQUE INDEX tests_success_constraint ON tests (subject, target)
    WHERE success;
```

原则上，我们可以对此索引进行仅索引扫描，以满足查询:

```
SELECT target FROM tests WHERE subject = 'some-subject' AND success;
```

但是有一个问题：WHERE子句依赖了没有作为索引的结果列的success列。尽管如此，仅索引扫描是可能的，因为计划不需要在运行时重新检查WHERE子句的那部分：索引中找到的所有条目必须具有success = true，因此不需要在计划中显式检查。 PostgreSQL  9.6及更高版本将会识别这种情况，并允许生成只针对索引的扫描，但较旧的版本不会生成。

## 检查索引的使用情况

虽然PostgreSQL中的索引不需要维护或调优，但检查实际查询工作负载实际使用哪些索引仍然十分重要。使用EXPLAIN命令检查单个查询的索引使用情况。其第[14.1节](https://www.postgresql.org/docs/9.6/static/using-explain.html)介绍了EXPLAIN的用法。也可以在运行的服务器中收集有关索引使用情况的统计信息，如[第28.2节](https://www.postgresql.org/docs/9.6/static/monitoring-stats.html)所述。

确定要创建一个什么类型的索引的过程一般比较难。在上一节中，示例中已经显示了一些典型的情况。经常需要进行大量实验。本节的其余部分提供了一些技巧：

- 始终先运行ANALYZE。此命令收集表中值的分布统计信息。需要此信息来估计查询返回的行数，Planner需要为每个可能的查询计划分配实际成本。在没有任何真实的统计数据的情况下，假定一些默认值，这几乎是不准确的。因此，检查应用程序的索引使用而不运行ANALYZE是不可行的。有关详细信息，请参见[第24.1.3节](https://www.postgresql.org/docs/9.6/static/routine-vacuuming.html#VACUUM-FOR-STATISTICS)和[第24.1.6节](https://www.postgresql.org/docs/9.6/static/routine-vacuuming.html#AUTOVACUUM)。

- 使用实际数据进行实验。使用测试数据设置索引会告诉您测试数据需要哪些索引，但仅仅是对测试数据有效。

  使用非常小的测试数据集尤其致命。当选择100000行中的1000个可能是索引的候选者时，选择100行中的1个几乎不会是这样的，因为100行可能适合单个磁盘页面，并且没有计划可以顺序地获取1个磁盘页面。

  在编写测试数据时也要小心，当应用程序尚未投入生产时，这通常是不可避免的。非常相似，完全随机或以排序顺序插入的值将使统计数据偏离实际数据所具有的分布。

- 当不使用索引时，测试可能有助于强制使用它们。运行时参数可以关闭各种计划类型（参见[第19.7.1节](https://www.postgresql.org/docs/9.6/static/runtime-config-query.html#RUNTIME-CONFIG-QUERY-ENABLE)）。例如，关闭最基本的计划的顺序扫描（enable_seqscan）和嵌套循环连接（enable_nestloop）将强制系统使用不同的计划。如果系统仍然选择顺序扫描或嵌套循环连接，那么可能还有一个更根本的原因是为什么索引没有被使用;例如，查询条件与索引不匹配。 （什么样的查询可以使用上一节中介绍的类型的索引。）

- 如果强制索引使用确实使用索引，那么有两种可能性：系统是正确的，使用索引确实不合适，或者查询计划的成本估计并不反映现实情况。所以你应该检查使用和不使用索引进行查询时的时间消耗。 EXPLAIN ANALYZE命令在这里非常有用。

- 如果事实证明成本估计是错误的，那么再有两种可能性。总成本由每个计划节点的每行成本乘以计划节点的选择性估计值计算。可以通过运行时参数来调整计划节点的成本（在[第19.7.2节](https://www.postgresql.org/docs/9.6/static/runtime-config-query.html#RUNTIME-CONFIG-QUERY-CONSTANTS)中描述）。选择性估算不准确是由于统计数据不足。通过调整统计信息收集参数可以改善这一点（参见[ALTER TABLE](https://www.postgresql.org/docs/9.6/static/sql-altertable.html)）。