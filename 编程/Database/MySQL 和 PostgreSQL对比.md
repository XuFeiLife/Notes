


## 一句总结

MySQL 更强调易用、部署和读多场景的高性能；PostgreSQL 更偏向功能完整性、标准兼容性、复杂查询与扩展能力（如 GIS、时序、全文、分布式扩展）。选择时要根据业务特性（读写比例、复杂度、扩展需求、运维能力）来权衡。


## 对比

| 维度         |                                    MySQL | PostgreSQL                               |
| ---------- | ---------------------------------------: | ---------------------------------------- |
| 设计目标       |                        简单、易用、Web/OLTP 优化 | 功能全面、SQL 标准、可扩展                          |
| 存储引擎       |                           多引擎（InnoDB 常用） | 单一内置引擎，功能丰富                              |
| 并发控制       |                InnoDB 行级锁 + 间隙锁（存在锁争用概念） | MVCC（快照隔离），避免大量锁竞争                       |
| 扩展/插件      |                         插件/引擎生态（例如第三方引擎） | 丰富扩展（PostGIS、Citus、TimescaleDB 等）        |
| JSON / 文档  |                  JSON 类型（binary JSON）与函数 | json / jsonb，强大的操作符与索引（GIN）              |
| 索引类型       |    B-tree, Hash（有限）, Fulltext（InnoDB 支持） | B-tree, GIN, GiST, SP-GiST, BRIN, Hash 等 |
| 复制 & HA    |         binlog、主从、Group Replication、GTID | 流复制、逻辑复制、复制槽、Patroni/rewind 等 HA 方案      |
| 事务隔离 & 串行化 |                    依赖引擎（InnoDB 支持多种隔离级别） | MVCC + 可选 SERIALIZABLE，真实的可串行化隔离实现       |
| 备份         | mysqldump, mysqlpump, Percona XtraBackup | pg_dump, pg_basebackup, WAL 恢复           |
| 社区 & 生态    |  较大，商业化产品多（Oracle/MySQL、MariaDB、Percona） | 社区活跃，学术与企业扩展丰富                           |


## 设计与内部机制

- 存储引擎：MySQL 的一大特色是可选存储引擎（InnoDB、MyISAM、Memory、NDB 等）。不同引擎之间实现差异较大（事务、锁策略、全文索引等）。PostgreSQL 只有单一内核实现，功能更统一。

- 写前日志与持久化（WAL / redo log）：
  - PostgreSQL 使用 WAL（Write-Ahead Logging）实现持久性，并通过 WAL 来实现流复制（streaming replication）与 PITR（基于 WAL 的时间点恢复）。
  - MySQL 的 InnoDB 有 redo log / undo log 与 binlog（二进制日志）用于复制与恢复；binlog 默认以 statement 或 row 模式记录。

- 并发控制（锁 vs MVCC）：
  - PostgreSQL 采用 MVCC：每个事务看到的是某个快照，写入通过版本链与 VACUUM 回收老版本。MVCC 在读多写少场景能显著降低读锁竞争，但需要定期 vacuum。
  - InnoDB 也实现了 MVCC，但会配合 gap lock（间隙锁）、next-key lock 等策略以确保某些隔离级别下的一致性，这会产生锁等待和死锁的可能性。


## 事务与隔离级别

- PostgreSQL：默认 Read Committed；实现了 Repeatable Read 的快照隔离语义。PostgreSQL 的 SERIALIZABLE 使用可序列化快照隔离实现（SSI），能够捕获冲突并将相应事务回滚以保持并发可序列化语义。

- MySQL（InnoDB）：支持多种隔离级别（READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE）。MySQL 的 REPEATABLE READ 在 InnoDB 中包含了 gap lock 行为以避免幻读（不同于 PG 的实现），但有可能带来锁争用。

实务要点：
- 需要长事务时，PostgreSQL 的 MVCC 版本增长问题需靠 VACUUM（或 autovacuum）治理；长事务会阻止 VACUUM 回收。
- 在高并发写场景，谨慎使用 MySQL 的 REPEATABLE READ + gap lock（可能出现写争用）；也可以调整为 READ COMMITTED 减少锁范围。


## 索引与查询优化器

- 索引支持：PostgreSQL 提供多种索引类型（B-tree、GIN、GiST、BRIN 等），并支持表达式索引（functional index）和部分索引（partial index）。这些强力特性适合复杂查询与大数据量场景。

- MySQL 从 5.7/8.0 起也增加了表达式索引、持久化生成列等能力，但总体索引玩法和灵活性仍不如 PostgreSQL 丰富。

- 查询优化器：两者均为成本模型优化器（CBO），实现差异意味着同一 SQL 在两者上可能完全不同的执行计划。PostgreSQL 的查询规划与子查询/聚合优化通常更成熟，MySQL 在简单读写查询上的基线性能优化更专注。


## JSON / 文档 支持对比（实战）

- PostgreSQL：支持 json（保留原始文本）与 jsonb（二进制、可索引、查询更快）。常见优势：
  - 支持 GIN 索引（快速查询 jsonb 的键/路径/存在性）
  - 丰富操作符（->, ->>, #>, @>, ? 等）
  - 可以构建 expression index 在 jsonb 上

示例：在 `data jsonb` 的列上创建 GIN 索引：

```sql
CREATE INDEX ON events USING GIN (data jsonb_path_ops);
-- 查询某个路径存在
SELECT * FROM events WHERE data @> '{"user": {"id": 123}}';
```

- MySQL：提供 JSON 类型并以二进制存储（compact binary format），函数丰富（JSON_EXTRACT, JSON_UNQUOTE, JSON_CONTAINS 等），并支持在生成列上建立普通索引以实现 json 查询加速。

示例：生成列 + 索引：

```sql
ALTER TABLE events ADD COLUMN user_id INT GENERATED ALWAYS AS (JSON_EXTRACT(data, '$.user.id')) STORED;
CREATE INDEX idx_events_user_id ON events(user_id);
```

实务比较要点：PostgreSQL 的 jsonb + GIN 在复杂 JSON 查询与索引覆盖上更强大；MySQL 在以生成列 + 常规索引绕过限制时也能达到良好效果，但查询表达力与索引灵活性不如 PG。


## Full-Text Search

- PostgreSQL 的内置全文搜索（tsvector / tsquery）与语言处理（词干、停用词）结合良好，并可通过 GIN 索引优化；对于更复杂需求可直接使用 pg_trgm、外部 Elasticsearch 或内嵌扩展。

- MySQL 提供 FULLTEXT 索引（InnoDB 从 5.6 开始支持），能满足常规场景，但在语言处理、可扩展性与搜索质量上通常不及 PostgreSQL + 专门搜索引擎的组合。


## 分区与分表

- PostgreSQL：提供声明式分区（declarative partitioning），支持范围、列表与哈希分区，内部路由（partition pruning）能力较强。对于复杂约束（如唯一约束跨分区）有特定要求（通常要在分区键上保证唯一性）。

- MySQL：也支持分区（range/list/hash/key），但早期版本的分区功能有限，某些约束和外键与分区结合时受限。MySQL 的水平扩展常结合分片中间件或 ProxySQL 等方案。


## 复制、逻辑复制与分布式扩展

- MySQL：binlog 为主的复制体系，支持基于 GTID 的 failover、主从（异步/半同步）、组复制（Group Replication）与 InnoDB Cluster 等。生态中有很多运维工具（Ansible 脚本、Orchestrator、MHA、ProxySQL、MaxScale）。

- PostgreSQL：流复制（physical replication）提供主备热备，逻辑复制允许复制特定表或做双向复制（但更复杂）。常见 HA 方案：Patroni（基于 Etcd/Consul/ZK 实现 leader election）、pg_auto_failover、repmgr 等。分布式扩展可用 Citus 等扩展。

实务要点：
- MySQL 的复制在多年成熟套件与工具支持下易于部署；MySQL 的 binlog row-based replication 在某些场景下复制数据更精确（兼容性）。
- PostgreSQL 的逻辑复制更灵活但细节（DDL 同步、序列、依赖）需要额外处理；流复制更适合物理备份恢复与低延迟备份。


## 备份与恢复

- MySQL 常用：mysqldump（逻辑备份）、mysqlpump、Percona XtraBackup（热备，物理）等。恢复粒度根据工具与 binlog 可实现 point-in-time 恢复。

- PostgreSQL 常用：pg_dump/pg_dumpall（逻辑）、pg_basebackup（物理）、WAL + basebackup 实现 PITR。Postgres 的 WAL 恢复机制成熟且灵活。


## 安全性与权限

- PostgreSQL：支持角色（roles）、基于列/行的权限、行级安全（Row-Level Security, RLS），适合做细粒度安全控制。

- MySQL：用户/权限模型成熟，支持多种认证插件（例如 PAM、LDAP、SHA256、caching_sha2_password），但行级安全是较新的能力，功能上整体不如 PG 细粒度。


## 扩展生态（典型扩展）

- PostgreSQL：PostGIS（GIS）、Citus（分布式扩展）、TimescaleDB（时序数据库）、pg_trgm、pg_cron 等。
- MySQL：常见的是分支/增强版（MariaDB、Percona Server），以及各类存储引擎或插件；生态偏向运维工具和商业产品。


## SQL 语法差异与迁移注意（实用示例）

- Upsert（冲突时更新）：
  - MySQL:

```sql
INSERT INTO t (id, v) VALUES (1, 'a')
ON DUPLICATE KEY UPDATE v = VALUES(v);
```

  - PostgreSQL:

```sql
INSERT INTO t (id, v) VALUES (1, 'a')
ON CONFLICT (id) DO UPDATE SET v = EXCLUDED.v;
```

- 返回受影响行（RETURNING）：
  - PostgreSQL 广泛支持 `RETURNING`，可以直接在 INSERT/UPDATE/DELETE 后得到行数据。
  - MySQL 传统上没有等价通用机制（可用 LAST_INSERT_ID()、SELECT），某些版本在特定 DML 场景增加了返回增强，请以版本文档为准。

- 序列与自增：
  - PostgreSQL 使用 sequence（nextval, currval）；适合跨语句获取 id。
  - MySQL 的 AUTO_INCREMENT 更简单，但跨会话获取最后 id 需要 LAST_INSERT_ID()

- 日期 / 时区：PostgreSQL 在 timestamp with time zone 等语义上更明确，适合严格的时间管理；MySQL 的时间类型有些实现细节需注意（例如时区处理和函数行为）。


## 性能调优要点（实务）

- EXPLAIN / EXPLAIN ANALYZE：PG 推荐使用 `EXPLAIN (ANALYZE, BUFFERS)` 获取计划与实际执行时间；MySQL 的 `EXPLAIN` 输出不同，MySQL 8 的 EXPLAIN JSON 格式也更丰富。

- 参数调优：PostgreSQL 关注 `shared_buffers`, `work_mem`, `maintenance_work_mem`, `effective_cache_size`, `max_wal_size` 等；MySQL 则关注 `innodb_buffer_pool_size`, `innodb_log_file_size`, `query_cache`（已废弃/禁用）等。

- 慎用长事务与未提交事务：都会阻碍 VACUUM / undo 回收，导致磁盘膨胀或 undo log 增大。

- 建表与索引策略：在写密集型场景下，索引越多写越慢；PostgreSQL 的索引类型更容易为查询优化提供多样选择（GIN/BRIN 对特定场景能显著提升性能）。


## 迁移与兼容性建议

- 迁移前做事项清单：
  - 审计 SQL：识别不兼容语法（LIMIT/OFFSET、UPSERT、函数名、系统变量）
  - 数据类型对齐：ENUM、BOOL、TEXT、JSON、BLOB 等
  - 约束与触发器逻辑重写（触发器行为、事务内外差异）
  - 索引与查询计划重写（利用 PG 的表达式索引/部分索引等）

- 常见工具：pgloader（MySQL -> PostgreSQL 自动化迁移工具）、AWS DMS 等，但复杂业务仍需人工校验 SQL 行为与性能。


## 实务选择建议（决策矩阵）

- 如果你需要：
  - 简单、快速上线的 Web 应用，且团队熟悉 LAMP/LEMP：MySQL 是稳妥选择。
  - 复杂关系模型、大量分析查询、高度自定义索引或 GIS 支持：PostgreSQL 更合适。
  - 需要水平扩展并具备成熟商业支持：可综合考虑 MySQL 生态（MariaDB/Percona）或使用 PostgreSQL + Citus/外部 sharding 层。


## 常见误区与注意点

- "MySQL 比 PostgreSQL 快" 不是普适结论：简单读写在 MySQL 上可能延迟更低，但复杂查询、聚合、并发写入、索引选择等场景 PostgreSQL 往往更胜一筹。
- 生产监控与运维同样重要：无论哪种 DB，监控（慢查询、锁等待、IO、replication lag）能显著提升稳定性与性能。
