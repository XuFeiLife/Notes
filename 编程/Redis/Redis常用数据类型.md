###  String

存储文本，JSON以及数字（支持自增自减）

文本操作
```
127.0.0.1:6379> SET name zhangsan
OK
127.0.0.1:6379> GET name
"zhangsan"
127.0.0.1:6379>
```

数值操作
```
127.0.0.1:6379> INCR age
(integer) 1
127.0.0.1:6379> INCR age
(integer) 2
127.0.0.1:6379> DECR age
(integer) 1
127.0.0.1:6379> DECR age
(integer) 0
127.0.0.1:6379> 
```

**常见用途**
1.  缓存对象（JSON 字符串）  
2.  计数器（访问量、点赞数）  
3. 分布式锁（SETNX）

### Hash

类似对象/map/字典
适合存储用户信息、配置信息
```
127.0.0.1:6379> HSET user:1 name lisi
(integer) 0
127.0.0.1:6379> HSET user:1 age 20
(integer) 0
127.0.0.1:6379> HGET user:1 name
"lisi"
127.0.0.1:6379> HGETALL user:1
1) "name"
2) "lisi"
3) "age"
4) "20"
127.0.0.1:6379> 
```

**常见用途**  
1.  存储对象（如 User、Product）  
2.  结构化数据  
3.  更节省内存（比 JSON String 好）

### List
有序，可当队列、栈
```
127.0.0.1:6379> LPUSH queue a
(integer) 1
127.0.0.1:6379> LPUSH queue b
(integer) 2
127.0.0.1:6379> RPUSH queue c
(integer) 3
127.0.0.1:6379> LRANGE queue 0 -1
1) "b"
2) "a"
3) "c"
127.0.0.1:6379> LPOP queue
"b"
127.0.0.1:6379> LRANGE queue 0 -1
4) "a"
5) "c"
127.0.0.1:6379> RPOP queue
"c"
127.0.0.1:6379> LRANGE queue 0 -1
6) "a"
127.0.0.1:6379> 
```

**常见用途**  
1.  消息队列（简单 MQ）  
2. 时间线  
3.  任务队列

### Set

无序集合，不重复，自动去重
```
127.0.0.1:6379> SADD tags java
(integer) 1
127.0.0.1:6379> SADD tags redis
(integer) 1
127.0.0.1:6379> SADD tags redis
(integer) 0
127.0.0.1:6379> SMEMBERS tags
1) "java"
2) "redis"
127.0.0.1:6379> SISMEMBER tags redis
(integer) 1
127.0.0.1:6379> SREM tags java
(integer) 1
127.0.0.1:6379> SMEMBERS tags
3) "redis"
127.0.0.1:6379>
```

**常见用途**  
1. 去重数据  
2. 共同好友 / 共同兴趣（交集）  
3. 黑名单、白名单

### Sorted Set

有序集合，不重复，按score排序，常见排行榜