---
title: Redis 设计与实现：核心数据结构详解
date: 2024-02-15
tags:
  - Redis
  - 缓存
  - 后端
cover: /images/redis.jpg
---

Redis 为什么这么快？除了内存操作外，更重要的是它精心设计的数据结构。本文深入解析 Redis 的核心数据结构，帮你理解 Redis 高性能的秘密。

## 一、Redis 为什么快？

1. **内存操作**：数据存在内存中，访问速度比磁盘快几个数量级
2. **单线程**：避免了锁竞争和上下文切换
3. **IO 多路复用**：使用 epoll 高效处理并发连接
4. **精心设计的数据结构**：不同场景使用最合适的数据结构

## 二、五种基本数据结构

### 1. String（字符串）

**底层实现**：动态字符串（SDS）

```c
struct sdshdr {
    int len;    // 已使用长度
    int free;   // 未使用长度
    char buf[]; // 实际存储
};
```

**优点**：
- O(1) 获取字符串长度
- 空间预分配，减少内存分配次数
- 二进制安全（可以存储任意二进制数据）

**常用命令**：
```bash
SET key value
GET key
INCR counter      # 原子递增
SETEX key 3600 value  # 设置过期时间
```

### 2. List（列表）

**底层实现**：双向链表 + 压缩列表（ziplist）

- 元素少时用 ziplist（内存紧凑）
- 元素多时用 quicklist（链表 + ziplist）

**常用命令**：
```bash
LPUSH list item    # 左边插入
RPUSH list item    # 右边插入
LPOP list         # 左边弹出
LRANGE list 0 -1  # 获取所有元素
```

**应用场景**：消息队列、任务列表、热点数据

### 3. Hash（哈希）

**底层实现**：哈希表 + 压缩列表

```c
typedef struct dict {
    dictEntry **table;
    unsigned long size;
    unsigned long used;
    // ...
} dict;
```

**常用命令**：
```bash
HSET user name "Tom"
HSET user age 25
HGET user name
HGETALL user
```

**应用场景**：对象存储、购物车、配置信息

### 4. Set（集合）

**底层实现**：哈希表 / 整数集合（intset）

- 全是整数用 intset（更节省内存）
- 其他用哈希表

**常用命令**：
```bash
SADD tags "java" "python"
SREM tags "python"
SMEMBERS tags
SISMEMBER tags "java"  # 是否存在
```

**应用场景**：标签系统、好友关系、去重

### 5. Sorted Set（有序集合）

**底层实现**：跳表（skiplist） + 压缩列表

**跳表结构**：
```
Level 3: ----> A ---------------------->
Level 2: --> B ----> D --------------->
Level 1: --> C --> D --> E --> F --> G
```

**常用命令**：
```bash
ZADD leaderboard 100 "Tom"
ZADD leaderboard 90 "John"
ZADD leaderboard 80 "Mary"
ZRANGE leaderboard 0 -1 WITHSCORES  # 获取排名
ZREVRANGE leaderboard 0 9           # 倒序前10
```

**应用场景**：排行榜、延迟任务、权重队列

## 三、特殊数据结构

### 1. Bitmaps（位图）

```bash
SETBIT user:login:2024 100 1  # 设置第100号用户2024年已登录
GETBIT user:login:2024 100   # 检查是否登录
BITCOUNT user:login:2024     # 统计登录用户数
```

### 2. HyperLogLog

用于基数统计（不重复元素数量）：

```bash
PFADD hll ip1 ip2 ip3
PFCOUNT hll
```

**优点**：12KB 内存可以统计 2^64 个元素

### 3. Geospatial（地理坐标）

```bash
GEOADD cities 116.40 39.90 "Beijing"
GEODIST cities Beijing Shanghai km  # 计算距离
GEORADIUS cities 116 39 100 km     # 附近的城市
```

### 4. Stream（消息队列）

Redis 5.0+ 引入的持久化消息队列：

```bash
XADD mystream * field value
XRANGE mystream - +
XREAD COUNT 2 STREAMS mystream 0
```

## 四、数据结构选择指南

| 场景 | 推荐数据结构 |
|------|-------------|
| 字符串 | String |
| 列表、队列 | List |
| 对象、Hash | Hash |
| 去重、标签 | Set |
| 排行榜 | Sorted Set |
| 位操作 | Bitmap |
| 统计 | HyperLogLog |
| 地理位置 | Geospatial |
| 消息队列 | Stream |

## 五、总结

Redis 的高性能源于：
1. 内存存储
2. 单线程模型
3. IO 多路复用
4. **精心设计的数据结构**

理解数据结构，才能更好地使用 Redis。

---

*Redis 的设计思想值得所有后端开发者学习：选择最合适的数据结构，而不是最复杂的。*
