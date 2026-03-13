---
title: MySQL 索引原理与优化实战
date: 2024-02-01
tags:
  - MySQL
  - 数据库
  - 性能优化
cover: https://images.unsplash.com/photo-1544383835-bda2bc66a55d?w=800
---

数据库查询性能一直是后端开发的核心问题，而索引是提升查询性能的最有效手段。本文深入探讨 MySQL 索引的底层原理，帮助你从根子上理解如何高效使用索引。

## 一、索引的本质

索引的本质是 **数据结构**，用于快速定位数据。MySQL 中最常用的索引结构是 **B+ Tree**（平衡多路查找树）。

### 为什么不用二叉树？

二叉树查找效率高，但深度太深，IO次数太多。假设 100 万条数据，二叉树深度约 20 层，每层一次 IO 就需要 20 次磁盘读取。

### 为什么是 B+ Tree？

B+ Tree 是一种多叉树，每个节点可以存储多个键值。相同数据量下，B+ Tree 的深度远小于二叉树。

**B+ Tree 的特点：**
- 非叶子节点不存储数据，只存储索引
- 叶子节点存储所有数据
- 叶子节点之间用链表相连（便于范围查询）

## 二、索引分类

### 1. 主键索引（Primary Key）

```sql
CREATE TABLE user (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

主键索引叶子节点存储完整数据行，一个表只能有一个。

### 2. 唯一索引（Unique）

```sql
CREATE UNIQUE INDEX idx_email ON user(email);
```

保证索引列的值唯一。

### 3. 普通索引（Index）

```sql
CREATE INDEX idx_name ON user(name);
```

最基本的索引类型。

### 4. 组合索引

```sql
CREATE INDEX idx_age_name ON user(age, name);
```

多个字段组合的索引，遵循 **最左前缀原则**。

## 三、最左前缀原则

组合索引 `idx_age_name(age, name)` 相当于创建了三个索引：
- (age)
- (age, name)

但 **不能** 用 (name) 查询，因为 name 不是最左边的列。

## 四、Explain 分析执行计划

使用 `EXPLAIN` 关键字可以查看查询是否使用索引：

```sql
EXPLAIN SELECT * FROM user WHERE age = 25;
```

关键字段：
- **type**：查询类型（const > eq_ref > ref > range > index > all）
- **key**：实际使用的索引
- **rows**：扫描的行数
- **Extra**：额外信息（Using index、Using filesort 等）

## 五、实战优化技巧

### 1. 避免索引失效

```sql
-- ❌ 索引失效
SELECT * FROM user WHERE age + 1 = 26;
SELECT * FROM user WHERE LEFT(name, 1) = '张';

-- ✅ 索引有效
SELECT * FROM user WHERE age = 25;
SELECT * FROM user WHERE name LIKE '张%';
```

### 2. 覆盖索引

如果查询的列都在索引中，就不需要回表查询：

```sql
-- ✅ 覆盖索引，不需要回表
SELECT age, name FROM user WHERE age = 25;

-- ❌ 需要回表查询
SELECT * FROM user WHERE age = 25;
```

### 3. 索引下推

MySQL 5.6+ 自动使用索引下推，在索引遍历时过滤数据：

```sql
SELECT * FROM user WHERE age > 20 AND name LIKE '张%';
```

## 六、总结

| 场景 | 推荐索引 |
|------|----------|
| 等值查询 | 主键索引、唯一索引 |
| 范围查询 | 普通索引 |
| 排序 | 索引字段与 ORDER BY 一致 |
| 组合查询 | 组合索引，遵循最左前缀 |

**核心原则**：让查询尽量走索引，减少回表，减少排序。

---

*索引优化是门艺术，需要结合具体业务场景和实际数据特点来设计。*
