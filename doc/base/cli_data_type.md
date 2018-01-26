# redis命令简介及数据类型简介

- [redis命令简介](##-redis命令简介)

## redis命令介绍

```
└──bin
├── redis-benchmark #redis性能测试工具，可以测试在本系统本配置下的读写性能
├── redis-check-aof #对更新日志appendonly.aof检查，是否可用
├── redis-check-dump #用于检查本地数据库的rdb文件
├── redis-cli #redis命令行操作工具，也可以用telnet根据其纯文本协议来操作
├── redis-sentinel Redis-sentinel #是Redis实例的监控管理、通知和实例失效备援服务，是Redis集群的管理工具
└──redis-server #redis服务器的daemon启动程序
```


### redis-cli命令介绍

默认选择 db库是 0
```
redis-cli -p 6379
```
查看当前所在“db库”所有的缓存key
```
redis 127.0.0.1:6379> keys *
```
选择 db库
```
redis 127.0.0.1:6379> select 8
```
清除所有的缓存key
```
redis 127.0.0.1:6379> FLUSHALL
```
清除当前“db库”所有的缓存key
```
redis 127.0.0.1:6379[8]> FLUSHDB
```
设置缓存值
```
redis 127.0.0.1:6379> set keyname keyvalue
```
获取缓存值
```
redis 127.0.0.1:6379> get keyname
```
删除缓存值：返回删除数量（0代表没删除）
```
redis 127.0.0.1:6379> del keyname
```

### 服务端相关命令
`time`：返回当前服务器时间  
`client list`: 返回所有连接到服务器的客户端信息和统计数据 参见[客户端列表](http://redisdoc.com/server/client_list.html )   
`client kill ip:port`：关闭地址为 ip:port 的客户端  
`save`：将数据同步保存到磁盘  
`bgsave`：将数据异步保存到磁盘  
`lastsave`：返回上次成功将数据保存到磁盘的Unix时戳  
`shundown`：将数据同步保存到磁盘，然后关闭服务  
`info`：提供服务器的信息和统计  
`config resetstat`：重置info命令中的某些统计数据  
`config get`：获取配置文件信息  
`config set`：动态地调整 Redis 服务器的配置(configuration)而无须重启，可以修改的配置参数可以使用命令 CONFIG GET* 来列出  
`config rewrite`：Redis 服务器时所指定的 redis.conf 文件进行改写  
`monitor`：实时转储收到的请求  
`slaveof`：改变复制策略设置  
`debug`： sleep segfault  
`slowlog get `获取慢查询日志  
`slowlog len` 获取慢查询日志条数  
`slowlog reset` 清空慢查询  

## redis数据类型及其操作


Redis内部使用一个redisObject对象来表示所有的key和value  
**type代表一个value对象具体是何种数据类型**  
**encoding是不同数据类型在redis内部的存储方式**  
**比如**：type=string代表value存储的是一个普通字符串，那么对应的encoding可以是raw或者是int,如果是int则代表实际

redis内部是按数值型类存储和表示这个字符串的，当然前提是这个字符串本身可以用数值表示，比如:"123" "456"这样的字符串。

Redis的键值可以使用物种数据类型：`字符串`，`散列表`，`列表`，`集合`，`有序集合`。

### String（字符串）　　　　　

string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。
string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
string类型是Redis最基本的数据类型，一个键最大能存储512MB。

例子：
```
redis 127.0.0.1:6379> SET name "duzhaoqi"
OK
redis 127.0.0.1:6379> GET name
"duzhaoqi"
```
在以上实例中我们使用了 Redis 的 SET 和 GET 命令。键为 name，对应的值为 `duzhaoqi`。

注意：一个键最大能存储`512MB`。

### Hash（哈希）
Redis hash 是一个键值(key=>value)对集合。
Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

例子：
```
redis> HMSET myhash field1 "Hello" field2 "World"
"OK"
redis> HGET myhash field1
"Hello"
redis> HGET myhash field2
"World"
```
以上实例中 hash 数据类型存储了包含用户脚本信息的用户对象。 实例中我们使用了 Redis HMSET, HGETALL 命令，user:1 为键值。

每个 hash 可以存储 232 -1 键值对（40多亿）。

### List（列表）
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

例子：
```
redis 127.0.0.1:6379> lpush runoob redis
(integer) 1
redis 127.0.0.1:6379> lpush runoob mongodb
(integer) 2
redis 127.0.0.1:6379> lpush runoob rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange runoob 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
redis 127.0.0.1:6379>
```
列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。

### Set（集合）
Redis的Set是string类型的无序集合。  
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

**sadd 命令**  
添加一个string元素到,key对应的set集合中，成功返回1,如果元素已经在集合中返回0,key对应的set不存在返回错误。  
`sadd key member`  
例子：
```
redis 127.0.0.1:6379> sadd runoob redis
(integer) 1
redis 127.0.0.1:6379> sadd runoob mongodb
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd runoob rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers runoob

1) "rabitmq"
2) "mongodb"
3) "redis"
```
注意：以上实例中 rabitmq 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。

### zset(sorted set：有序集合)
Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。  
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。  
zset的成员是唯一的,但分数(score)却可以重复。

**zadd 命令**  
添加元素到集合，元素在集合中存在则更新对应score  
`zadd key score member `  
例子：
```
redis 127.0.0.1:6379> zadd runoob 0 redis
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 mongodb
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 1
redis 127.0.0.1:6379> zadd runoob 0 rabitmq
(integer) 0
redis 127.0.0.1:6379> ZRANGEBYSCORE runoob 0 1000

1) "redis"
2) "mongodb"
3) "rabitmq"
```