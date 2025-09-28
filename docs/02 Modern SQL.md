# **02: Modern SQL**

## 一、SQL 历史与发展

### 1. SQL 发展历程

- **1971年**：IBM创建 SQUARE（第一个关系查询语言）
- **1972年**：IBM创建 SEQUEL（Structured English Query Language）
- **1979-1983年**：IBM发布商业 SQL DBMS（System/38、SQL/DS、DB2）
- **1986/1987年**：ANSI/ISO 标准化
- **当前标准**：SQL:2023（支持属性图查询、多维数组）

### 2. SQL 标准演进

- **SQL:2023**：属性图查询、多维数组
- **SQL:2016**：JSON、多态表
- **SQL:2011**：时态数据库、流水线 DML
- **SQL:2008**：截断、高级排序
- **SQL:2003**：XML、窗口函数、序列、自动生成 ID
- **SQL:1999**：正则表达式、触发器、面向对象

### 3. SQL 现状与未来

- **就业市场需求旺盛**：成为第二大必备编程语言
- **AI/NLP挑战**：自然语言查询可能替代部分 SQL 使用场景
- **向量数据库兴起**：但 SQL 仍是关系数据库的基石

---

## 二、SQL 语言组成

### 1. 主要组成部分
- **DML**（数据操作语言）：SELECT、INSERT、UPDATE、DELETE
- **DDL**（数据定义语言）：CREATE、ALTER、DROP
- **DCL**（数据控制语言）：GRANT、REVOKE
- 其他：视图定义、完整性约束、参照约束、事务

### 2. 重要特性
- 基于**包**（允许重复）而非集合（不允许重复）

---

## 三、聚合与分组

### 1. 聚合函数
- `AVG(col)`、`MIN(col)`、`MAX(col)`、`SUM(col)`、`COUNT(col)`
- 几乎只能在 SELECT 输出列表中使用

### 2. COUNT 的不同写法

```sql
COUNT(login)    -- 计算非空值
COUNT(*)        -- 计算所有行
COUNT(1)        -- 计算所有行
COUNT(1+1+1)    -- 计算所有行
```

### 3. GROUP BY
- 将元组投影到子集，对每个子集计算聚合
- **重要规则**：SELECT 中非聚合列必须出现在 GROUP BY 中

### 4. GROUPING SETS
- 在单个查询中指定多个分组
- 替代多个 UNION ALL 查询
```sql
GROUP BY GROUPING SETS (
    (c.name, e.grade),  -- 按课程和成绩分组
    (c.name),           -- 仅按课程分组
    ()                  -- 总体总计
)
```

### 5. HAVING

- 基于聚合计算过滤结果
- 类似于 GROUP BY 的 WHERE 子句
```sql
SELECT AVG(s.gpa), e.cid
FROM enrolled e, student s
WHERE e.sid = s.sid
GROUP BY e.cid
HAVING AVG(s.gpa) > 3.9;
```

---

## 四、字符串与日期操作

### 1. 字符串操作
- **大小写敏感性**：各 DBMS 实现不同（SQL-92 敏感，MySQL 不敏感）
- **LIKE操作符**：
  - `%`：匹配任意子串（包括空串）
  - `_`：匹配任意单个字符
- **SIMILAR TO**：正则表达式匹配（SQL 标准，但非所有系统支持）
- **字符串函数**：`SUBSTRING`、`UPPER`、`LOWER`等
- **字符串连接**：
  - SQL-92：`||`
  - MSSQL：`+`
  - MySQL：`CONCAT()`

### 2. 日期/时间操作
- 操作 DATE/TIME 属性
- 支持和语法因 DBMS 而异

---

## 五、输出控制与重定向

### 1. 输出控制
- **ORDER BY**：按列值排序（ASC|DESC）
- **FETCH**：限制返回的元组数量
- **OFFSET**：设置偏移量返回范围
```sql
SELECT sid, name FROM student 
WHERE login LIKE '%@cs'
ORDER BY gpa 
OFFSET 5 ROWS 
FETCH FIRST 5 ROWS WITH TIES;
```

### 2. 输出重定向
- 将查询结果存储到另一个表
- 表必须不存在，列数与类型与输入相同
```sql
-- SQL-92
SELECT DISTINCT cid INTO CourseIds FROM enrolled;

-- MySQL
CREATE TABLE CourseIds (SELECT DISTINCT cid FROM enrolled);
```

---

## 六、高级查询技术

### 1. 嵌套查询
- 在查询内部调用另一个查询
- 可以出现在几乎任何位置
- **操作符**：
  - `ALL`：对所有行必须为真
  - `ANY`：至少对一行必须为真
  - `IN`：等价于`=ANY()`
  - `EXISTS`：至少返回一行

### 2. LATERAL JOIN

- 允许嵌套查询引用前面嵌套查询中的属性
- 类似于 for 循环，为表中每个元组调用另一个查询
```sql
SELECT * FROM course AS c,
LATERAL (SELECT COUNT(*) AS cnt FROM enrolled WHERE enrolled.cid = c.cid) AS t1,
LATERAL (SELECT AVG(gpa) AS avg FROM student s JOIN enrolled e ON s.sid = e.sid WHERE e.cid = c.cid) AS t2
ORDER BY cnt ASC;
```

### 3. 公共表表达式（CTE）
- 指定临时结果集，可在查询其他部分引用
- 替代嵌套查询、视图和显式临时表
```sql
WITH maxCTE (maxId) AS (
    SELECT MAX(sid) FROM enrolled
)
SELECT name FROM student s
JOIN maxCTE ON s.sid = maxCTE.maxId;
```

---

## 七、窗口函数

### 1. 基本概念
- 跨一组相关元组执行计算，不将它们折叠为单个输出元组
- 支持运行总计、排名和移动平均
```sql
SELECT FUNC-NAME(...) OVER (...) FROM tableName;
```

### 2. 函数类型
- **聚合函数**：AVG、SUM、COUNT 等
- **特殊窗口函数**：
  - `ROW_NUMBER()`：当前行号
  - `RANK()`：当前行的排序位置

### 3. OVER 子句

- **PARTITION BY**：指定分组
- **ORDER BY**：在组内排序
```sql
SELECT cid, sid, ROW_NUMBER() OVER (PARTITION BY cid ORDER BY grade ASC) AS rank
FROM enrolled;
```

### 4. 应用示例
查找每门课程成绩第二高的学生：
```sql
SELECT * FROM (
    SELECT *, RANK() OVER (PARTITION BY cid ORDER BY grade ASC) AS rank
    FROM enrolled
) AS ranking
WHERE ranking.rank = 2;
```

---

## 八、总结与建议

### 1. SQL 的重要性

- 仍然是数据处理的核心语言
- 就业市场需求持续旺盛
- 尽管有 NL2SQL 工具，但手写 SQL 技能仍然重要

### 2. 最佳实践
- 尽可能将计算作为单个 SQL 语句完成
- 理解不同 DBMS 的方言差异
- 掌握现代 SQL 特性（窗口函数、CTE、LATERAL JOIN 等）

### 3. 学习建议

- 实践导向：通过实际查询加深理解
- 理解原理：掌握关系代数和 SQL 执行机制
- 关注发展：跟踪 SQL 标准和新特性演进

---

这个讲座全面覆盖了现代 SQL 的核心概念和高级特性，从基础聚合到复杂的窗口函数和横向连接，为数据库系统的高效使用提供了坚实的理论基础。