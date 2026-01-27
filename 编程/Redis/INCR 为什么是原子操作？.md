结论

 **INCR 是原子操作，因为 Redis 是单线程模型，命令在服务端串行执行，中途不会被打断。**

## 1. 为什么单线程等于原子

### Redis核心模型
``` vbnet

Client1  ─┐
Client2  ─┼─> 事件循环（Event Loop）─> 执行命令（一个一个）
Client3  ─┘

```

- Redis **只有一个线程执行命令**
- 同一时间 **只会处理一条命令**
- **一条命令从开始到结束不会被打断**

```bash
INCR counter
```

``` text
读取 counter
+1
写回 counter

```
**这三步在一个线程里一次性完成**
