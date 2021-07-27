# PostgreSQL 字符串处理与日期处理操作

### 字符串长度、大小写

```plsql
SELECT CHAR_LENGTH('test') -- 字符串长度
SELECT LENGTH('test') 
LENGTH(string,encoding name)
SELECT LENGTH('测试','UTF-8');
LOWER(string) 或者 UPPER(string) -- 大小写
ASCII(string)
SELECT ASCII('abc') -- 结果是'a'的ascii码
```

### 字符串格式化

```plsql
FORMAT(formatstr text [,formatarg "any" [, ...] ]) -- 类似于printf
```

### 字符串拼接

```plsql
SELECT 'number' || 123 --字符串连接
CONCAT(str "any" [, str "any" [, ...] ])
CONCAT_WS(sep text, str "any" [,str "any" [, ...] ])
SELECT * FROM CONCAT_WS('#','hello','world')
```

### 字符串剪切与截取

```plsql
LPAD(string text, length int [,fill text])
RPAD(string text, length int [,fill text])
SELECT LPAD('12345', 10,'0') -- 结果 "0000012345"
TRIM([leading | trailing | both] [characters] from string)
SELECT TRIM(both ' ' from ' hello world') -- 结果是'hello world'
BTRIM(string text [, characters text])
RTRIM(string text [, characterstext])
LTRIM(string text [, characterstext])
SELECT BTRIM('yyhello worldyyyy','y') -- 结果是'hello world'
LEFT(str text, n int) -- 返回字符串前n个字符，n为负数时返回除最后|n|个字符以外的所有字符
RIGHT(str text, n int)
SUBSTRING(string from int [for int]) 
SELECT SUBSTRING('hello world' from 7 for 5) -- 结果是'world'
```

### 字符串加引号

```plsql
QUOTE_IDENT(string text)
QUOTE_LITERAL(STRING TEXT)
QUOTE_LITERAL(value anyelement)
SELECT 'l''host"' -- 结果是'l'host"'
SELECT QUOTE_LITERAL('l''host"') -- 结果是'l''host"'
```

### 字符串分割

```plsql
SPLIT_PART(string text,delimiter text, field int)
REGEXP_SPLIT_TO_ARRAY(stringtext, pattern text [, flags text])
REGEXP_SPLIT_TO_TABLE(stringtext, pattern text [, flagstext])
SELECT SPLIT_PART('hello#world','#',2) -- 结果是'world'
SELECT REGEXP_SPLIT_TO_ARRAY('hello#world','#') -- 结果是{hello,world}
SELECT REGEXP_SPLIT_TO_TABLE('hello#world','#') as split_res -- 结果是两行,第一行hello,第二行world
```

### 字符串查找、反转与替换

```plsql
POSITION(substring in string) -- 查找
SELECT POSITION('h' in 'hello world') -- 结果是1，这里从1开始计数
REVERSE(str)
REPEAT(string text, number int)
REPLACE(string,string,string)
SELECT REPLACE('hello world',' ','#')
REGEXP_MATCHES(string text,pattern text [, flags text])
REGEXP_REPLACE(string text,pattern text,replacement text[, flags text])
SELECT REGEXP_MATCHES('hello world','.o.','g') -- 返回两行，第一行是'lo ',第二行是'wor'
SELECT REGEXP_MATCHES('hello world','.o.') -- 返回第一个匹配，'lo '
```

### 时间处理

```plsql
SELECT TO_CHAR(TO_TIMESTAMP(CREATE_TIME),'YYYY-MM-DD HH24:MI:SS')
SELECT EXTRACT(YEAR FROM NOW());
```

**补充：postgresql处理时间函数 截取hh:mm/yyyy-mm-dd**

### 1.to_timestamp：

```plsql
AND to_timestamp(a.upload_time,'yyyy-MM-dd')>='"+startTime+"' and to_timestamp(a.upload_time,'yyyy-MM-dd') <= '"+endTime+"' 
```

### 2.substring:

```plsql
substring('2019-04-08 14:18:09',index,k):
```

数值代表含义 index：代表从index开始截取数据，k代表从index开始截取到第k个数据

处理对象：时间为字符串格式的数据

**eg：**

### 截取时间到 年-月-日：

```plsql
SELECT substring(upload_time,1,10) from table WHERE upload_time='2019-04-08 14:18:09'
```

结果：2019-04-08

### 截取时间到 时:分：

```plsql
SELECT substring(upload_time,12,5) from table WHERE upload_time='2019-04-08 14:18:09'
```

结果：14:18