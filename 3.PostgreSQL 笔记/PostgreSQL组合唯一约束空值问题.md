- [PostgreSQL组合唯一约束空值问题](https://blog.csdn.net/neweastsun/article/details/119649930)

> 因为 `PostgreSQL`唯一约束并考虑空值的唯一性，我们虽然在列上定义了唯一约束，但仍然会存在重复数据。`PostgreSQL`唯一约束的规则是，唯一键的列值可以为NULL。

## 问题描述

在多个列上定义组合唯一键，那么当其中一个值为空而其他值不为空时约束不起作用，下面看详细过程。

- 创建表

```sql
CREATE TABLE TestUniqueNull
(
    ID INTEGER 
    ,NoA INTEGER 
    ,NoB INTEGER 
    ,NoC INTEGER 
    ,CONSTRAINT pk_tbl_TestUniqueNull_ID PRIMARY KEY(ID)
    ,CONSTRAINT uk_tbl_TestUniqueNull_NoA_NoB_NoC unique (NoA,NoB,NoC)
);
```

- 插入示例数据

```sql
INSERT INTO TestUniqueNull VALUES (1,1,2,NULL);
INSERT INTO TestUniqueNull VALUES (2,1,2,NULL);
INSERT INTO TestUniqueNull VALUES (3,1,5,NULL);
INSERT INTO TestUniqueNull VALUES (4,3,NULL,1);
INSERT INTO TestUniqueNull VALUES (5,3,NULL,1);
```

- 查看数据

| id   | noa  | nob  | noc  |
| ---- | ---- | ---- | ---- |
| 1    | 1    | 2    |      |
| 2    | 1    | 2    |      |
| 3    | 1    | 5    |      |
| 4    | 3    |      | 1    |
| 5    | 3    |      | 1    |

我们看到当列值为空时，存在重复记录，违背了定义唯一约束的需求。

## 使用唯一索引

针对上面的问题，我们使用唯一索引代替组合唯一约束：

```sql
CREATE TABLE TestUniqueNull
(
    ID INTEGER 
    ,NoA INTEGER 
    ,NoB INTEGER 
    ,NoC INTEGER 
    ,CONSTRAINT pk_tbl_TestUniqueNull_ID PRIMARY KEY(ID)
);

CREATE UNIQUE INDEX UIdx_NoA_NoB_NoC 
                    ON TestUniqueNull(coalesce(NoA,-1),coalesce(NoB,-1),coalesce(NoC,-1));
```

再次插入示例会报错：

> SQL 错误 [23505]: 错误: 重复键违反唯一约束 “uidx_noa_nob_noc”
>  Detail: 键值"(COALESCE(noa, ‘-1’::integer), COALESCE(nob, ‘-1’::integer), COALESCE(noc, ‘-1’::integer))=(1, 2, -1)" 已经存在

## 总结

本文描述`PostgreSQL` 组合唯一约束空值问题，并给出使用唯一索引代替约束的解决方法。