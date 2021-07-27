# PostGreSql 判断字符串中是否有中文的案例

## 实例

```plsql
imos=# select 'hello' ~ '[\u2e80-\ua4cf]|[\uf900-\ufaff]|[\ufe30-\ufe4f]';
 ?column?
----------
 f
(1 row)
imos=#
imos=# select 'hello中国' ~ '[\u2e80-\ua4cf]|[\uf900-\ufaff]|[\ufe30-\ufe4f]';
 ?column?
----------
 t
(1 row)
```

**补充：PostgreSQL 判断字符串包含的几种方法**

## 判断字符串包含的几种方法：

### 1. position(substring in string):

```plsql
postgres=# select position('aa' in 'abcd');
 position 
----------
 0
(1 row)
postgres=# select position('ab' in 'abcd');
 position 
----------
 1
(1 row)
postgres=# select position('ab' in 'abcdab');
 position 
----------
 1
(1 row)
```

可以看出，如果包含目标字符串，会返回目标字符串笫一次出现的位置，可以根据返回值是否大于0来判断是否包含目标字符串。

### 2. strpos(string, substring):

该函数的作用是声明子串的位置。

```plsql
postgres=# select strpos('abcd','aa');
 strpos 
--------
 0
(1 row)
postgres=# select strpos('abcd','ab');
 strpos 
--------
 1
(1 row)
postgres=# select strpos('abcdab','ab');
 strpos 
--------
 1
(1 row)
```

作用与position函数一致。

### 3. 使用正则表达式：

```plsql
postgres=# select 'abcd' ~ 'aa';
 ?column? 
----------
 f
(1 row)
postgres=# select 'abcd' ~ 'ab';
 ?column? 
----------
 t
(1 row)
postgres=# select 'abcdab' ~ 'ab';
 ?column? 
----------
 t
(1 row)
```

### 4. 使用数组的@>操作符（不能准确判断是否包含）：

```plsql
postgres=# select regexp_split_to_array('abcd','') @> array['b','e'];
 ?column? 
----------
 f
(1 row)
postgres=# select regexp_split_to_array('abcd','') @> array['a','b'];
 ?column? 
----------
 t
(1 row)
```

## 注意下面这些例子：

```plsql
postgres=# select regexp_split_to_array('abcd','') @> array['a','a'];
 ?column? 
----------
 t
(1 row)
postgres=# select regexp_split_to_array('abcd','') @> array['a','c'];
 ?column? 
----------
 t
(1 row)
postgres=# select regexp_split_to_array('abcd','') @> array['a','c','a','c'];
 ?column? 
----------
 t
(1 row)
```

可以看出，数组的包含操作符判断的时候不管顺序、重复，只要包含了就返回true，在真正使用的时候注意。