---
title: MySQL 命令
date: 2019-01-03 23:57:41
categories:
- database
tags:
- database
- mysql
---

# MySQL 命令

## 函数

**查看数据库版本**

- select version();

**查表是否存在**

- show tables like 'user_record'
<!--more-->
## 增删改查

**插入**

- INSERT INTO user_record (name, address, phone) VALUES ('a', 'b', 3)

- INSERT INTO user_record (name ,address) VALUES ('a', 'c') ON DUPLICATE KEY UPDATE address = 'c'

    name ,address 中必须存在primary key 或 unique index，触发duplicate时使用update操作

- REPLACE INTO user_record(name, address) VALUES('a', 'c')

    先执行insert操作，当遇到duplicate时，将原有记录删除，然后再insert

**更新**

- UPDATE user_record SET phone = 5 WHERE name = 'a'

**删除**

- TRUNCATE user_record

TRUNCATE语句和DELETE语句的区别:
1. delete语句，是DML语句，truncate语句通常被认为是DDL语句。

2. delete语句，后面可以跟where子句，通常指定where子句中的条件表达式，只删除满足条件的部分记录，而truncate语句，只能用于删除表中的所有记录。

3. truncate语句，删除表中的数据后，向表中添加记录时，自动增加字段的默认初始值重新从1开始，而使用delete语句，删除表中所有记录后，向表中添加记录时，自动增加字段的值，为删除时该字段的最大值加1，也就是在原来的基础上递增。

4. delete语句，每删除一条记录，都会在日志中记录，而使用truncate语句，不会在日志中记录删除的内容，因此，truncate语句的执行效率比delete语句高。

## JSON

- SELECT json_extract(json_data,'$.arr[0].desk') AS desk, json_extract(json_data,'$.arr[0].chair') AS chair FROM house_items

- SELECT * FROM  house_items WHERE json_contains_path(json_data,'one','$.arr[0].desk','$.arr[0].chair') > 0

**JSON_CONTAINS 指定数据是否存在**
```sql
set @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
-- JSON_CONTAINS(json_doc, val[, path])
-- 查询json文档是否在指定path包含指定的数据，包含则返回1，否则返回0。如果有参数为NULL或path不存在，则返回NULL。
SELECT JSON_CONTAINS(@j, '4', '$.c.d'); -- 1
```

**JSON_CONTAINS_PATH 指定路径是否存在**
```sql
-- JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)
-- 查询是否存在指定路径，存在则返回1，否则返回0。如果有参数为NULL，则返回NULL。
-- one_or_all只能取值"one"或"all"，one表示只要有一个存在即可；all表示所有的都存在才行。
SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e'); -- 1
SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.c.d'); -- 1
```

**JSON_EXTRACT 查找所有指定数据**

```sql
-- JSON_EXTRACT(json_doc, path[, path] ...)
-- 从json文档里抽取数据。如果有参数有NULL或path不存在，则返回NULL。如果抽取出多个path，则返回的数据封闭在一个json array里。
set @j2 = '[10, 20, [30, 40]]';
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]'); -- 20
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]', '$[0]'); -- [20, 10]
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[2][*]'); -- [30, 40]
```
 

**JSON_KEYS 查找所有指定键值**
```sql
-- JSON_KEYS(json_doc[, path])
-- 获取json文档在指定路径下的所有键值，返回一个json array。如果有参数为NULL或path不存在，则返回NULL。
SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}'); -- ["a", "b"]
SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}', '$.b'); -- ["c"]
SELECT id,json_keys(info) FROM t_json;
 ```

**JSON_SEARCH 查找所有指定值的位置**
```sql
-- JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])
-- 查询包含指定字符串的paths，并作为一个json array返回。如果有参数为NUL或path不存在，则返回NULL。
-- one_or_all："one"表示查询到一个即返回；"all"表示查询所有。
-- search_str：要查询的字符串。 可以用LIKE里的'%'或‘_’匹配。
-- path：在指定path下查。
SET @j3 = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
SELECT JSON_SEARCH(@j3, 'one', 'abc'); -- "$[0]"
SELECT JSON_SEARCH(@j3, 'all', 'abc'); -- ["$[0]", "$[2].x"]
SELECT JSON_SEARCH(@j3, 'all', 'abc', NULL, '$[2]'); -- "$[2].x"
SELECT JSON_SEARCH(@j3, 'all', '10'); -- "$[1][0].k"
SELECT JSON_SEARCH(@j3, 'all', '%b%'); -- ["$[0]", "$[2].x", "$[3].y"]
SELECT JSON_SEARCH(@j3, 'all', '%b%', NULL, '$[2]'); -- "$[2].x"
```

## 参考

* [mysql json 使用 类型 查询 函数](https://www.cnblogs.com/ooo0/p/9309277.html)
