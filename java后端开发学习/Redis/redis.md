# Redis

> Redis是一个基于**内存**的**key-value**结构数据库

- `redis-server`启动redis服务器，`redis-cli`连接redis服务器
- `redis-cli -h localhost -p 6379` 指定连接的redis服务器和端口号
- `KEYS pattern` 查找所有符合给定模式的key
- `EXISTS key` 检查给定key是否存在
- `TYPE key` 返回key所储存的值的类型
- `DEL key` 用于在key存在时删除key

## 常用数据类型

### 字符串（string）

普通字符串，redis中最简单的数据

- `SET key value` 设置指定key的值
- `GET key` 获取指定key的值
- `SETEX key seconds value` 设置指定key的值，并将key的国企时间设为seconds秒
- `SETNX key value` 只有在key不存在时设置key的值

### 哈希（hash）

也叫散列，类似java中的HashMap结构。**它是一个string类型的field和value的映射表**，hash适合用于存储对象。

- `HSET key field value` 将哈希表中的字段field的值设为value
- `HGET key field` 获取存储在哈希表中指定字段的值
- `HDEL key field` 删除存储在哈希表中指定字段
- `HKEYS key` 获取哈希表中所有字段
- `HVALS key` 获取哈希表中所有值

### 列表（list）

简单的字符串列表，按照插入顺序排序，可以有重复元素，类似java中的LinkedList

- `LPUSH key value1 [value2]` 将一个或多个值插入到列表头部
- `LRANGE key start stop` 获取列表指定范围内的元素
- `RPOP key` 移除并获取列表最后一个元素
- `LLEN key` 获取列表长度

### 集合（set）

string类型的无序集合，没有重复元素，类似于java中的HashSet

- `SADD key member1 [member2]` 向集合添加一个或多个成员
- `SMEMBERS key` 返回集合中所有成员
- `SCARD key` 获取集合的成员数
- `SINTER key1 [key2]` 返回给定所有集合的交集
- `SUNION key1 [key2]` 返回给定所有集合的并集
- `SREM key member1 [member2]` 删除集合中一个或多个成员

### 有序集合（sorted set/zset）

集合中每个元素关联一个分数，根据分数升序排序，没有重复元素

- `ZADD key score1 member1 [score2 member2]` 向有序集合添加一个或多个成员
- `ZRANGE key start stop [WITHSCORES]` 通过索引区间返回有序集合中指定区间内的成员
- `ZINCRBY key increment member` 有序集合中对指定成员的分数加上增量increment
- `ZREM key member [member ...]` 移除有序集合的一个或多个成员

## Spring Data Redis

maven坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置Redis数据源
