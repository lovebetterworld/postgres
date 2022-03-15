- [sql语句中“!=”与“<>”的区别 - 奋斗中的菜鸟程序猿 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wushuo-1992/p/8364828.html)

ANSI标准中是用<>(所以建议用<>)，但为了跟大部分数据库保持一致，数据库中一般都提供了 !=(高级语言一般用来表示不等于) 与 <> 来表示不等于：

- MySQL 5.1: 支持 `!=` 和 `<>`
- PostgreSQL 8.3: 支持 `!=` 和 `<>`
- SQLite: 支持 `!=` 和 `<>`
- Oracle 10g: 支持 `!=` 和 `<>`
- Microsoft SQL Server 2000/2005/2008: 支持 `!=` 和 `<>`
- IBM Informix Dynamic Server 10: 支持 `!=` 和 `<>`
- InterBase/Firebird: 支持 `!=` 和 `<>`

最后两个只支持ANSI标准的数据库：

- IBM DB2 UDB 9.5:仅支持 `<>`
- Apache Derby:仅支持 `<>`