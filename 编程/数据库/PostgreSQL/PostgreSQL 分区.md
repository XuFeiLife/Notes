[[1. 什么是 PostgreSQL 分区？](#1.%20什么是%20PostgreSQL%20分区？)
[2. 为什么要分区](#2.%20为什么要分区)
[3. 分区类型](#3.%20分区类型)
[4. 分区核心机制（分区裁剪）](#4.%20分区核心机制（分区裁剪）)
[5. 索引与分区的关系](#5.%20索引与分区的关系)
[6. 全局索引？](#6.%20全局索引？)
[7. 分区的限制与注意事项](#7.%20分区的限制与注意事项)

# 1. 什么是 PostgreSQL 分区？

**PostgreSQL 分区（Table Partitioning）** 是一种将**一张逻辑表**的数据，按照某个规则拆分到**多张物理子表**中的技术。

- 对应用层来说：**仍然是一张表**
- 对数据库来说：数据分散存储在多个 **分区表（Partition）** 中
```text
orders  （逻辑表）
 ├── orders_2024
 ├── orders_2025
 └── orders_2026
```

主要目的：
- 提升查询效率
- 提升写入和维护效率
- 降低单表数据量过大问题

# 2. 为什么要分区

## 2.1 单表数据量过大问题

当一张表数据量达到千万/亿，会遇到查询效率低，索引膨胀，冷热数据混在一起

## 2.2 分区带来的好处
| 好处       | 说明                          |
| -------- | --------------------------- |
| 查询性能提升   | **分区裁剪（Partition Pruning）** |
| 维护成本降低   | 可以单独 DROP / TRUNCATE 分区     |
| 写入压力分散   | 插入数据分散到不同分区                 |
| 历史数据管理简单 | 直接删除整分区                     |
# 3. 分区类型

## 3.1 RANGE分区（最常用）

按范围分区，通常用于时间字段
适用场景：
- 按时间，日期查询
- 日志表，订单表，流水表
```sql
CREATE TABLE orders (
  id BIGSERIAL,
  order_time DATE NOT NULL,
  amount NUMERIC
) PARTITION BY RANGE (order_time);

```
创建分区
```sql
CREATE TABLE orders_2024
PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE orders_2025
PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

```

## 3.2 LIST分区

按固定枚举值分区
适用场景：
- 状态字段
- 地区/类型
```sql
CREATE TABLE users (
  id BIGINT,
  region TEXT
) PARTITION BY LIST (region);

```

```sql
CREATE TABLE users_cn
PARTITION OF users
FOR VALUES IN ('CN');

CREATE TABLE users_us
PARTITION OF users
FOR VALUES IN ('US');

```

## 3.3 HASH分区

按哈希取模分区
使用场景：
- 数据分布不均
- 无明显范围规则
- 高并发写入
```sql
CREATE TABLE logs (
  id BIGINT,
  user_id BIGINT
) PARTITION BY HASH (user_id);

```

```sql
CREATE TABLE logs_p0 PARTITION OF logs
FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE logs_p1 PARTITION OF logs
FOR VALUES WITH (MODULUS 4, REMAINDER 1);

```

# 4. 分区核心机制（分区裁剪）

## 4.1 什么是分区裁剪

当 SQL **WHERE 条件命中分区键**时：
PostgreSQL **只扫描相关分区**，不会扫描全部表
```sql
SELECT *
FROM orders
WHERE order_time >= '2025-01-01'
  AND order_time < '2025-02-01';

```

# 5. 索引与分区的关系

### 5.1 分区表索引特点

- **父表上的索引 ≠ 实际索引**
- 实际索引是建在 **每个分区表上**
```sql
CREATE INDEX idx_orders_time ON orders(order_time);

```

等价
```sql
orders_2024.idx_orders_time
orders_2025.idx_orders_time
```
# 6. 全局索引？

PostgreSQL **不支持真正的全局索引**

这意味着：
- 唯一约束 **必须包含分区键**
- 否则无法保证全局唯一性
```sql
-- 正确
UNIQUE (id, order_time)

-- 错误
UNIQUE (id)

```

# 7. 分区的限制与注意事项

### 7.1 不是所有表都适合分区

  不适合：
- 小表
- 高频 JOIN 维度表
- 分区键无法用于查询条件

### 7.2 分区 ≠ 自动提升性能

性能提升前提：
- **查询条件必须命中分区键**
- 分区粒度合理（不要过细）

### 7.3 分区数量控制

经验建议：

- 单表分区数：**几十 ~ 几百**
- 上千分区会影响：
    - 规划器性能
    - 元数据管理