
[主从复制](#主从复制)
[Sentinel（哨兵模式）](#Sentinel（哨兵模式）)
[Redis Cluster](#Redis%20Cluster)

### 为什么需要Redis集群

单机Redis存在瓶颈
1. 内存受限，单机内存有限
2. QPS受限
3. 单点故障风险
4. 水平扩展困难

### Redis高可用集群方案

#### 主从复制

一个Master，多个Slave，Slave同步Master数据，但是Master宕机需要人工切换，没有自动故障转移

#### Sentinel（哨兵模式）

在主从基础上增加了自动HA组件，监控Redis，故障时自动主从切换。

####  Redis Cluster

1. 数据分片（水平扩展）
2. 高可用（故障自动转移）


