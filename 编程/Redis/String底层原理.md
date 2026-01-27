[1. 核心特性](#1.%20核心特性)
[2. 内部实现原理：SDS](#2.%20内部实现原理：SDS)
[3. 典型应用场景](#3.%20典型应用场景)
[4. 存储方式](#4.%20存储方式)
[5. 性能调优启示](#5.%20性能调优启示)

在 Redis 中，**String（字符串）** 是最基础、最常用的数据类型。虽然名叫 String，但它其实是“二进制安全”的，这意味着它可以存储任何形式的数据，比如文本、序列化的 JSON 对象、甚至是图片或视频的二进制码。

## 1. 核心特性

1. **二进制安全**：不仅能存文本，还能存整数、浮点数或二进制流。
2.  **最大容量**：单个 String 值的最大上限是 **512 MB**。
3.  **原子性**：针对 String 的数值操作（如自增）是原子的，非常适合计数场景。

## 2. 内部实现原理：SDS

Redis 并没有直接使用 C 语言传统的字符串表示（以 `\0` 结尾的字符数组），而是构建了一种名为 **SDS (Simple Dynamic String，简单动态字符串)** 的结构。

### 为什么用 SDS？

1. **常数复杂度获取长度**：C 字符串需要 $O(N)$ 遍历，SDS 只需 $O(1)$ 读取长度字段。
2. **杜绝缓冲区溢出**：修改前会检查空间是否足够，不足则自动扩容。
3.  **减少内存重分配**：通过“空间预分配”和“惰性空间释放”策略，减少系统调用。

```c
struct sdshdr {
    int len;    // 已使用长度
    int alloc;  // 分配的总长度
    char buf[]; // 实际数据
}

```

## 3. 典型应用场景

### 缓存功能

最经典的应用。将数据库查询结果、会话信息（Session）序列化后存入 Redis，减轻后端数据库压力。

### 计数器 / 限流

利用 `INCR` 命令实现点赞数、访问量统计，或者通过结合过期时间来实现“每分钟限制访问 10 次”的限流功能。
[INCR 为什么是原子操作？](INCR%20为什么是原子操作？.md)

## 4. 存储方式

Redis 会根据你存入的内容自动选择存储方式来节省内存：

- **int**：存储 8 字节的长整型。
- **embstr**：存储短字符串（小于等于 44 字节），内存分配更连续。
- **raw**：存储大于 44 字节的字符串。

> **小贴士**：在设计 Key 的时候，建议采用 `业务名:对象名:ID` 的格式（如 `user:profile:1001`），这样既清晰又方便在图形化工具中分类查看。

```bash
> set key1 123
OK
> OBJECT ENCODING key1
int
> set key2 hello
OK
> OBJECT ENCODING key2
embstr
> set key3 aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
OK
> OBJECT ENCODING key3
raw

```

## 5. 性能调优启示
由于 `embstr` 是一次性分配内存且连续，效率最高。在设计 Key 或 Value 时，如果能通过缩写等方式将长度控制在 **44 字节以内**，能显著提升高并发下的内存访问速度。