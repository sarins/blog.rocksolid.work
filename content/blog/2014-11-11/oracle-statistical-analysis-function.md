+++
title = 'Oracle统计分析函数'
subtitle = 'Oracle statistical analysis function'
brief = ''
date = 2014-11-11
categories = ['Database', 'Oracle', 'Statistical', 'Analysis']
series = []
tags = ['Database', 'Oracle', 'Statistical', 'Analysis', 'Window function', 'RegExp']
type = 'blog'
+++

## 1. 环境与参考

### 1.1. 引用文档

> Oracle sql language reference 11.2 (E41084) - (SQL_Language_Reference_11.2_e41084.pdf)

### 1.2. 样例数据


>Table name: XXX
| col_1
| -------------------------
| H3.1-K27(me2)K36(me0)
| H3.1-K27(me3)K36(me0)
| H3-LVR
| H3K9(me1)K14(ac1)
| H3.1-K27(me2)K36(me1)
| H3.1-K27(me3)K36(me1)
| H3.1-K27(me0)K36(me2)
| H3.1-K27(me0)K36(me3)
| H3K9(me1)K14(ac0)
| H3.1-K27(ac1)K36(me1)
| H3.1-K27(ac1)K36(me0)
| H3.1-K27(me1)K36(me2)
| H3.1-K27(me1)K36(me3)
| H3.1-K27(me0)K36(me0)
| H3-YRPGTVALR
| H3.1-K27(me1)K36(me1)
| H3.1-K27(me1)K36(me0)
| H3.1-K27(me0)K36(me1)
| H3K79(me2)
| H3K79(me3)
| H3K79(me1)

- - -

## 2. 正则表达式解析函数

### 2.1. 解析规则需求
```properties
# The PTM label format in general is:
Ha.b-Ac(Lx)Ay(Lz)

# Where:
# a, b, c, x, y, z are all numbers, L in braces can be ‘ac’ or ‘me’, A can be K, R, or S

----------------------------------------
|a: 3                |y: 0, 14, 36, 23 |
----------------------------------------
|b: 0, 1, 3	         |z: 0, 1, 2, 3    |
----------------------------------------
|c: 4, 9 ,27, 79, 18 |L: ac, me        |
----------------------------------------
|x: 0, 1, 2, 3	     |A: K, R, S       |
----------------------------------------

# Constrains
--------------------------------------------
|c = 9, y = 14  |L = ac, x, z = 0, 1       |
--------------------------------------------
|c = 27, y = 36 |L = me, x, z = 0, 1, 2, 3 |
--------------------------------------------
|c = 18, y = 23 |                          |
--------------------------------------------
```

- - -

### 2.2. SQL语句编写

```sql
# Oracle PL/SQL

select REGEXP_SUBSTR(x.col_1, '^H3\.[0-3]{1}|^H3') as P1,
       REGEXP_SUBSTR(x.col_1, '(K|R|S)(4|9|27|79|18)|EIR|LVR') as P2,
       REGEXP_SUBSTR(x.col_1, '(\((ac|me))', 1, 1, 'i', 2) as P3,
       REGEXP_SUBSTR(x.col_1, '(\((ac|me)(\d*))', 1, 1, 'i', 3) as P4,
       REGEXP_SUBSTR(x.col_1, '(K|R|S)(0|14|36|23)') as P5,
       REGEXP_SUBSTR(x.col_1, '(\((ac|me))', 1, 2, 'i', 2) as P6,
       REGEXP_SUBSTR(x.col_1, '(\((ac|me)(\d*))', 1, 2, 'i', 3) as P7,
       x.col_1 as type_name
  from XXX x

```

### 2.3. 语法说明

> REGEXP_SUBSTR（source_char, pattern, position, occurrence, match_parameter, subexpr)

（详细语法参见 E4108, 5-224页）.

* 基本语法

```sql
REGEXP_SUBSTR(x.col_1, '^H3\.[0-3]{1}|^H3')
REGEXP_SUBSTR(x.col_1, '(K|R|S)(4|9|27|79|18)|EIR|LVR')
REGEXP_SUBSTR(x.col_1, '(K|R|S)(0|14|36|23)')
```

使用一般语法正则表达式匹配并提取字符串。

* 进阶语法

```sql
REGEXP_SUBSTR(x.col_1, '(\((ac|me))', 1, 1, 'i', 2)
REGEXP_SUBSTR(x.col_1, '(\((ac|me)(\d*))', 1, 1, 'i', 3)
REGEXP_SUBSTR(x.col_1, '(\((ac|me))', 1, 2, 'i', 2)
REGEXP_SUBSTR(x.col_1, '(\((ac|me)(\d*))', 1, 2, 'i', 3)
```

使用一般语法正则表达式匹配，通过后续参数提取字符串。

1. 参数position： 指定被匹配字符串的开始下标。
2. 参数occurrence： 指定正则表达式匹配的次序，即第occurrence次被匹配后才认为是有效匹配。
3. 参数match_parameter: 命中细则，本例中未使用该特性，均为默认的"i"，即case-insensitive（大小写不敏感）。
4. 参数subexpr： 指定使用正则表达式子查询的匹配次序来提取字符串，即第subexpr个子查询来提取字符串，如(\((ac|me))', 1, 1, 'i', 2)则表示，匹配后通过(ac|me)部分来提取字符串。

- - -

## 3. 窗口统计函数

### 3.1. 样例数据与SQL

```sql
 select distinct AA,
                  count(x.aa) OVER(PARTITION BY x.aa ORDER BY x.aa RANGE UNBOUNDED PRECEDING) AS P1C,
                  sum(x.aa) OVER(PARTITION BY x.aa ORDER BY x.aa RANGE UNBOUNDED PRECEDING) AS P1S,
                  BB,
                  count(x.bb) OVER(PARTITION BY x.bb ORDER BY x.bb RANGE UNBOUNDED PRECEDING) AS P2C,
                  sum(x.bb) OVER(PARTITION BY x.bb ORDER BY x.bb RANGE UNBOUNDED PRECEDING) AS P2S,
                  CC,
                  count(x.cc) OVER(PARTITION BY x.cc ORDER BY x.cc RANGE UNBOUNDED PRECEDING) AS P3C,
                  sum(x.cc) OVER(PARTITION BY x.cc ORDER BY x.cc RANGE UNBOUNDED PRECEDING) AS P3S,
                  sum(x.cc) OVER(PARTITION BY x.aa ORDER BY x.aa RANGE UNBOUNDED PRECEDING) AS P3S_BY_AA,
                  avg(x.cc) OVER(PARTITION BY x.aa ORDER BY x.aa RANGE UNBOUNDED PRECEDING) AS P3S_BY_AA
    from (select 1 as aa, 10 as bb, 100 as cc from dual
          union all
          select 1 as aa, 10 as bb, 200 as cc from dual
          union all
          select 1 as aa, 20 as bb, 300 as cc from dual
          union all
          select 1 as aa, 20 as bb, 400 as cc from dual) x
   order by bb, cc
```

> Results:
| AA                                                                 | P1C | P1S | BB  | P2C | P2S | CC  | P3C | P3S | P3S_BY_AA | P3A_BY_AA |
| ------------------------------------------------------------------ | --- | --- | --- | --- | --- | --- | --- | --- | --------- | --------- |
| 1                                                                  | 4   | 4   | 10  | 2   | 20  | 100 | 1   | 100 | 1000      | 250       |
| 1                                                                  | 4   | 4   | 10  | 2   | 20  | 200 | 1   | 200 | 1000      | 250       |
| 1                                                                  | 4   | 4   | 20  | 2   | 40  | 300 | 1   | 300 | 1000      | 250       |
| 1                                                                  | 4   | 4   | 20  | 2   | 40  | 400 | 1   | 400 | 1000      | 250       |

### 3.2. 语法说明

> analytic_function OVER(PARTITION BY "Column define" ORDER BY "Column define" RANGE UNBOUNDED PRECEDING)

（详细语法参见 E4108, 5-11页）.

* 使用说明

1. analytic_function： 一般统计函数皆可以使用，诸如SUM、AVG、COUNT等等。
2. OVER()子句： 声明该函数为窗口调用函数，即该函数作用于fetch动作之后，因此若数据量巨大时应有选择的使用。
3. PARTITION BY "Column define" 子句： 该子句定义统计分析函数将根据参数“Column define”指定的列来筛选数据。
4. ORDER BY "Column define" 子句： 该子句定义统计分析函数执行时将根据参数“Column define”指定的列来排列数据。
5. RANGE UNBOUNDED PRECEDING 子句： 该子句定义统计分析函数统计时对数据的边界认定原则。（本例中的UNBOUNDED PRECEDING表明，如统计数据变化将重新开始统计）
6. **本例具有特定数据特定场景，统计函数详细语法与功能应参考E4108, 5-11页的语法说明。**

