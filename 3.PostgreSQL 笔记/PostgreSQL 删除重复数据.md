- [PostgreSQL 删除重复数据](https://blog.csdn.net/neweastsun/article/details/119566977)

## 准备数据

创建 `basket` 表：

```sql
CREATE TABLE basket(
    id SERIAL PRIMARY KEY,
    fruit VARCHAR(50) NOT NULL
);
```

插入示例数据：

```sql
INSERT INTO basket(fruit) values('apple');
INSERT INTO basket(fruit) values('apple');

INSERT INTO basket(fruit) values('orange');
INSERT INTO basket(fruit) values('orange');
INSERT INTO basket(fruit) values('orange');

INSERT INTO basket(fruit) values('banana');

select * from basket; -- 查询示例数据
```

返回结果：

| id   | fruit  |
| ---- | ------ |
| 1    | apple  |
| 2    | apple  |
| 3    | orange |
| 4    | orange |
| 5    | orange |
| 6    | banana |

我们看到有两条 `apple` 和 三条 `orange` 记录，下面我们的目标是删除重复数据。

### 查询重复数据

```sql
SELECT   fruit,  COUNT(fruit)
FROM basket
GROUP BY fruit
HAVING COUNT(fruit) > 1
ORDER BY fruit;
```

返回结果：

| fruit  | count |
| ------ | ----- |
| apple  | 2     |
| orange | 3     |

## 删除重复数据

### 使用DELETE USING 语句

```sql
DELETE FROM basket a
       USING basket b
WHERE a.id < b.id
  AND a.fruit = b.fruit;
```

我们连接 basket 表 和 它自身，然后检查不同行满足条件为 (a.id < b.id) 和 fruit 相同(a.fruit = b.fruit); 结果符合我们预期，重复数据被删除。

我们发现最小ID 值对应记录被删除而保留了ID值最大的记录。如果我们选哟保留最小的记录：

```sql
DELETE FROM basket a
       USING basket b
WHERE a.id > b.id
  AND a.fruit = b.fruit;
```

结果是最小ID值记录保留。

### 使用子查询删除重复记录

```sql
DELETE FROM basket
WHERE id IN
   (SELECT id
    FROM (
        SELECT id, ROW_NUMBER() OVER( PARTITION BY fruit ORDER BY id ) AS row_num
        FROM basket 
    ) t
    WHERE t.row_num > 1 );
```

子查询返回重复行，除了重复记录的第一行；然后外部DELETE 根据子查询删除对应记录。

如果你想保留最高ID记录，需要修改排序条件：

```sql
DELETE FROM basket
WHERE id IN
   (SELECT id
    FROM (
        SELECT id, ROW_NUMBER() OVER( PARTITION BY fruit ORDER BY  id DESC ) AS row_num
        FROM basket 
    ) t
    WHERE t.row_num > 1 );
```

如果基于多个列确定重复记录，则查询模板为：

```sql
DELETE FROM table_name
WHERE id IN
    (SELECT id
    FROM (
        SELECT id,
         ROW_NUMBER() OVER( PARTITION BY column_1, column_2 ORDER BY  id ) AS row_num
        FROM table_name 
    ) t
    WHERE t.row_num > 1 );
```

如果数据量较大情况下，利用窗口函数的子查询方式效率应该会更优。因为不是每条记录和全部进行比较，而是在窗口内比较。

### 使用中间表删除数据

利用中间表需要下面几个步骤：

1. 创建相同表结构的新表，用于存储无重复记录
2. 插入无重复记录至新表
3. 删除原表
4. 重命名新表

实现如下：

```sql
-- step 1
CREATE TABLE basket_temp (LIKE basket);

-- step 2
INSERT INTO basket_temp(fruit, id)
SELECT 
    DISTINCT ON (fruit) fruit,
    id
FROM basket; 

-- step 3
DROP TABLE basket;

-- step 4
ALTER TABLE basket_temp 
RENAME TO basket;            
```

## 总结

本文介绍不同方式删除重复记录，使用 `delete using` 、利用窗口函数的子查询方式，最后是利用中间表方式。