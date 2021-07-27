# PostGresql 实现四舍五入、小数转换、百分比的用法说明

需求：两个整数相除，保留两位小数并四舍五入，完了转成百分比形式，即4/5=0.80=80%

### 1.两个整数相除：

```plsql
idn_dw=> select 4/5;
 ?column?
----------
  0
(1 row)
```

在sql运算中，"/"意思是相除取整，这样小数部分就会被舍去。

### 2.用cast将被除数转成小数

```plsql
idn_dw=> select cast(4 as numeric)/5;
  ?column?
------------------------
 0.80000000000000000000
(1 row)
```

也可以简化：pg中"::"是转换的意思

```plsql
idn_dw=> select 4::numeric/5;
  ?column?
------------------------
 0.80000000000000000000
(1 row)
```

### 3.四舍五入，保留两位小数

```plsql
idn_dw=> select round(cast(4 as numeric)/5,2);
 round
-------
 0.80
(1 row)
```

### 4.放大100，转成百分比形式

```plsql
idn_dw=> select concat(round(4::numeric/5,2)*100,'%');
 concat
--------
 80.00%
(1 row)
```

但是，小数部分不需要，调整一下顺序

```plsql
idn_dw=> select concat(round(4::numeric/5*100),'%');
 concat
--------
 80%
(1 row)
```

完事。

**补充：使用postgresql的round()四舍五入函数报错**

### 需求：

使用postgresql的round()四舍五入保留两位小数

报错：

```plsql
HINT: No function matches the given name and argument types. You might
```

### 解决方案：

使用cast函数将需要四舍五入的值转为 numeric，转为其他的类型可能会报错

示例：

```plsql
round(cast(计算结果) as numeric), ``2``)
```